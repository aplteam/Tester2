# Test framework for Unit tests in Dyalog APL


`Tester2` is a member of the APLTree library. The library is a collection of classes etc. that aim to support the Dyalog APL programmer. Search GitHub for "apltree" and you will find solutions to many every-day problems Dyalog APL programmers might have to solve.


## Overview 

The purpose of this class is to provide a framework for testing all the projects of the APLTree library. Only with such a framework is it possible to make changes to any APLTree project with confidence.

You might find the framework flexible enough to suit your own needs when it comes to implementing tests, even independently from the APLTree project.

Depending on how the Test framework is used it might or might not present a GUI. Note that the GUI is a Windows-only feature although `Tester2` works on all platforms; without a GUI messages of any kind are printed to the Dyalog session window.

If the GUI shows that's how it looks like:


![The messages](gui_1.png "The messages")

The "Message" tab shows all the messages the `Tester2` class provides, and prints to the Dyalog session window on non-Windows platforms:

* Whether there were any INI files instantiated
* Whether a function  `Initial` was found and executed
* Which test cases got examinated and possibly executed
* What the overall picture is:
  * Number of test cases executed
  * Number of test cases that crashed
  * Number of test cases that failed
  * Number of test cases that were inactive
  * Number of test cases that were not applicable
  * Number of test cases that are designed to run on different platforms
* Whether  a function  `Cleanup` was found and executed

The "List" tab shows a list of all test cases with the status in the first column, the name in the second column and the first comment line in the third column:

![Ths list](gui_2.png "The list")

The status column shows:

* For okay a `✓`
* For a failed one a `*`
* For a crashed one a `#`
* For an inactive one a `⍝`

The same information is available by colour:

* Green for okay
* Black for failed
* Red for crashed
* Grey for inactive.

After all test cases are processed the user might interact with the GUI. These are the commands presented by the context menu:

![Context menu](gui_3.png "Context menu")

Notes:

* All `Run*` functions excepts `RunGUI` return  a two-element vector:

  [1] is a return code (`rc`) with 0 for "okay".
  [2] is an empty vector in case `rc` is 0 and might contain additional information in case `rc` is not 0.
 
* The `RunGUI` function returns a three-element vector. The third element is a reference pointing to the GUI (Form).

  Without catching that reference the next memory compaction will get rid of the GUI. You can enforce a memory compaction by executing `⎕WA`.


## Details 


### Terminology

Note that test cases causing a crash are considered "broken". Test cases that do not return the expected result are considered "failing".


### Assumptions and preconditions

1. The `#.Tester2` class assumes that all your tests are hosted by a namespace. It must not be an unnamed namespace. All methods of the `Tester2` class running test cases take a ref to that namespace as an argument.

1. All test functions inside that namespace are expected to start their names with `Test_` followed by digits. 

   **Update**: since version 1.1 test functions can be grouped. For example, in order to group all functions testing a method `foo` the test functions can be named `Test_foo_001`, `Test_foo_002`, `Test_foo_003` and so on.
   
   It is recommended that in case you use groups you assign _all_ your test cases to a group.

1. The number of digits you use for numbering is not restricted: `Test_foo_1` is as fine as is `Test_foo_0000001`. However, they should be consistent. 

   When starting to implement test cases you are advised to leave gaps: `Test_foo_001`, `Test_foo_010` etc.

1. The first line after the header must carry a comment telling what is actually tested.

   Keep in mind that later this is the only way to tell one test case from the others without reading the code, so be as clear as you can possibly be. 

   You are restricted to a single line, and you should keep it short enougn to be displayed with a reasonably setting of `⎕PW`.


### Executing test cases

There are three typical scenarios when you want to run test cases. All these scenarios are covered by functions that are available as [Helpers](#), see there for details.
    
* Run with error trapping and return a code together with the log (a vector of text vectors) reporting details. The code is `0` in case all test cases executed successfully and something else otherwise.

  This is perfect for, say, running all tests as part of an automated build process. See `Run` for details.
   
* Run without error trapping. Use this when you want to investigate why particular test cases crash or fail. See `RunDebug` for details.

  In case a problem is detected the test stops straight away, so you can investigate.
   
* Run "batch mode". This means that test cases that require a human being in front of the monitor are **not** executed. See `unBatchTests` for details. 

  This is the equivalent to `Run` but without any test cases that require a human being in front of the monitor.

  Of course it is best not to rely on a human being but this cannot always be avoided, or it would require too much of an effort.

* Run "batch mode" but without error trapping. This is the equivalent of running `RunDebug`. See `RunBatchTestsInDebugMode`.

* Run only selected test cases, either by specifying a group name or by specifying test case numbers or both. See `RunThese` for details.

### Syntax of test function

1. Every test function must accept a right argument which is a two-item vector of Booleans:

   1. `debugFlag` (0=use error trapping)
   1. `batchFlag` (0=does not run in batch mode)

1. Every test function must return a result. Use one of the following niladic functions (acting like constants) as explicit result; see next topic.

#### Constants

| Constant              | Meaning                                                                     |
|-----------------------|-----------------------------------------------------------------------------|
| `∆OK`                 | Passed |
| `∆Failed`             | Unexpected result|
| `∆NoBatchTest`        | Not executed because `batchFlag` was 1.|
| `∆NotApplicable`      | This test is not applicable here and now. |
| `∆InActive`           | Not executed because the test case is inactive (not ready, buggy, ...) |
| `∆LinuxOnly`          | Not executed because runs under Linux only.|
| `∆LinuxOrMacOnly`     | Not executed because runs under Linux/Mac OS only.|
| `∆LinuxOrWindowsOnly` | Not executed because runs under Linux/Windows only.|
| `∆MacOrWindowsOnly`   | Not executed because runs under Mac OS/Windows only.|
| `∆MacOnly`            | Not executed because runs under Mac OS only.|
| `∆NoAcreTests`        | Not executed because it's acre related.|
| `∆WindowsOnly`        | Not executed because runs under Windows only.|

These niladic functions are made available as [Helpers](#).

Using these niladic functions rather than numeric values increases not only readability, it is also easier to search for, say, all inactive test cases.


## Work flow

No matter which of the `Run*` functions (`Run`, `RunGUI`, `RunBatchTests`, `RunDebug`, `RunBatchTestsInDebugMode`, `RunThese`) you are going to call, the workflow is always the same:


### INI files (optional)

First of all the `Run*` method checks whether there is a file `testcases.ini` in the current directory. If this is the case that INI file is processed. Use this to specify general stuff that does not depend on a certain computer/environment.

Then it checks for a file `testcases_{computername}.ini`. If this exists then it is processed. Use this to specify computer-dependent variables.

Note that if one of the two INI files exists there will be a flat namespace `{yourTestNamespace}.INI`, meaning that sections in the INI file are ignored. An example: if your test functions are hosted by a namespace `Foo` and your INI file specifies a variable `hello` as holding "world" then:

```
       'world'≡#.Foo.INI.hello
1
```


### Initialisation

In the next step the `Run*` method checks whether there is a function `Initial` in the hosting namespace. If that is the case then the function is executed.

Note that the function must be either niladic or monadic, and it may or may not return a Boolean result. A 1 means that function did what it is supposed to do (=same as no result) while a 0 means it could not initialize.

Of course you can simply execute `→` on a single line in your `Initial` function if any requirement is not met but that would also mean that if you run your test cases automatically somehow then this would not work; in such cases `Initial` should return a 0 indicating failure. Also, part of the initialization might have been carried out, and a function `Cleanup` (discussed in a second) might get rid of any left-overs.

If the function is monadic a reference pointing to the parameter space is passed as the right argument. By checking this `Initial` can, for example, find out whether the cases are running in batch mode or not.

Use this function to create stuff that's needed by **all** test cases, or tell the user something important (only if the batch flag is false of course).

It might  be a good idea for _all_ test functions to tidy up in line 1, just in case a failing test case has left something behind; see [CleanUp](#cleanUp) for details.


### Finally: running the test cases

Now the test cases are executed one by one.


### Tidying up: `CleanUp`{#cleanUp}

After the last test case was executed the `Run*` function checks whether there is a function `Cleanup` in the namespace hosting your test cases. If that's the case then this function is executed. Such a function must be a niladic, no-result returning function.


### INI file again

Finally the namespace "INI" holding variables populated from your INI file(s) is deleted.


## Helpers

Over time it emerged that certain helpers are very useful in the namespace hosting the test cases. Therefore with version 1.4.0 a method `EstablishHelpersIn` was introduced that takes a reference as the right argument. 

That ref should point to the namespace hosting the test cases. If you are executing `#.Tester2.EstablishHelpersIn` from within that namespace you can specify an empty vector as the right argument: it then defaults to `⎕IO⊃⎕RSI`.

One of the helpers is the method `ListHelpers` which returns a table with all names in the first column and the first line of the function (which is expected to be a comment telling what the function actually does) in the second column:

```
      ListHelpers
 Run                       Run all test cases                                                           
 RunGUI                    Run all or a subset of test cases with a GUI. (Windows only)
 RunDebug                  Run all test cases with DEBUG flag on                                        
 RunThese                  Run just the specified tests.                                                
 RunBatchTests             Run all batch tests                                                          
 RunBatchTestsInDebugMode  Run all batch tests in debug mode
 E                         Get all functions into the editor starting their names with "Test_".         
 L                         Prints a list with all test cases and the first comment line to the session. 
 G                         Prints all groups to the session.                                            
 FailsIf                   Usage : →FailsIf x, where x is a boolean scalar                              
 PassesIf                  Usage : →PassesIf x, where x is a boolean scalar        
 GetTestFnsTemplate        ⍝ Returns a vector of text vectors with the code of a template test case.                     
 GoToTidyUp                Returns either an empty vector or "Label" which defaults to ∆TidyUp          
 RenameTestFnsTo           Renames a test function and tell acre.                                       
 ListHelpers               Lists all helpers available from the `Tester2` class.                         
 ∆...                      ⍝ Niladic functions returning an integer constant, see table above.
```

The helpers fall into four groups:


### Running test cases

`Run`, `RunGUI`, `RunDebug`, `RunThese`, `RunBatchTest` and `RunBatchTestsInDebugMode` are running all or selected test cases with or without error trapping. They are shortcuts for calling certain methods in `Tester2`.


### Flow control

`FailsIf`, `PassesIf` and `GoToTidyUp` control the program flow in test functions. The test template contains examples for how to use these functions.

The functions return a result (Boolean) in case error trapping is active but make the calling `Test*`-function crash otherwise, allowing you to investigate a failing test case right on the spot.

This is achieved by the functions `FailsIf` and `PassesIf` signalling an error that can be trapped with:

```
      ⎕TRAP←(999 'C' '. ⍝ Deliberate error')(0 'N')
```

That's why the template for a test function (injected by `EstablishHelpersIn` ) carries such a statement _and_ keeps `⎕TRAP` local.

Note that `GoToTidyUp` allows you to jump to a label `∆TidyUp` with a statement like:

```
 →GoToTidyUp ~expected≡result
```

This is useful in case a test case needs to do some cleaning up like deleting a temporary file created by the test case. In case the right argument is 1 (rather than 0) it causes a crash in debug mode and carries out the jump otherwise.


### Test function template

In case the namespace has no test functions in it (yet) the `EstablishHelpersIn` function also creates a test function template named `Test_000`. 

Note that for technical reasons this is _not_ the case when the hosting namespace is scripted.


### Constants

`∆OK` and `∆Failed` and others are niladic functions that behave like constants; they are useful for assigning explicit results in test functions. For a full list see [Constants](#).


### Misc

`G`

: List all groups, if any.

`L`

: List all test functions if the right argument is (empty).

: The right argument can be used to restrict output to a group, the optional left argument to restrict output to certain numbers.

`E`

: Edit all test functions (empty right argument) or the function(s) provided as right argument, be it a simple character vector (single name), a vector of character vectors or a matrix.

: Note that `E` can process the result of `L`.

### Technical documentation

All Helpers are members of the `Tester2.Helpers` sub class. That means you can get a technical documentation with this statement:

```
      ]ADOC #.Tester2.Helpers
```


## Examples

### Running test cases

#### `RunDebug`

Usually you will run this function with a 0 as right argument.

However, sometimes you want to trace through a test case, and that is when specifying a 1 as right argument comes in handy: `RunDebug` would then stop just before the test cases are actually executed, after processing any INI file(s), executing any `Initial` function before executing the test cases, and any `Cleanup` function after having executed all test functions.

#### RunGUI

With `RunGUI` you can achieve the same as with `RunDebug`. It might be easier to start with `RunGUI` but you might switch to `RunDebug` later, if only because it is significantly faster.

However, there are situations when `RunGUI` is indispensable: `Tester2`s own test cases are almost impossible to follow without it, for example. It's also very useful in order to demonstrate the features of the `Tester2` class.


#### `RunThese`

A particularly helpful method while developing/enhancing stuff is `RunThese`. The function allows you to run just selected test functions rather than a whole test suite.

If you now think, well, why not just call any function `Test_001` myself then imagine a situation when all your test cases depend on an INI file or the execution of `Initial` ot both. That is exactly the advantage of `RunThese`: it carries out all these steps, and also executes the `Cleanup` function in case there is one.

`RunThese` offers the following options:

```
      RunThese 1 2    ⍝ Execute test cases 1 & 2 that do not belong to any group.
      RunThese ¯1 ¯2  ⍝ Same but stop just before a test case is actually executed.
      RunThese 'Group1' ⍝ Execute all test cases belonging to group 1.
      RunThese 'Group1' (2 3) ⍝ Execute test cases 2 & 3 of group 1.
      RunThese 'Group1' 2 3   ⍝ Same as before.
      RunThese 'L'    ⍝ Execute all test cases of the group that start with "L" - if there is just one
      RunThese 'L*'   ⍝ Execute all test cases of all groups starting with "L"
      RunThese 'Test_Group1_001'  ⍝ Executes just this test case.
      RunThese 0      ⍝ Run all test cases but stop just before the execution.
```

Sometimes you want to trace through test cases. This can be achieved be specifying the number as negative integers. `RunThese` would then stop just before the test cases are actually executed, after processing any INI file(s), executing any `Initial` function before executing the test cases, and any `Cleanup` function after having executed all test functions.

### Managing test cases 

There are three helper functions available that assist you in managing test cases.

The examples stem from the `Fire` project.

#### Listing groups with `G`

`G` lists all groups:

```
      G
acre           
InternalMethods
List           
Misc           
Replace        
ReportGhosts   
Search 
```

#### Listing test functions with `L`

`L` requires a right argument (empty or group name) and accepts an optional left argument (test case numbers).

```
      ⍴L ''
99 150
        
      L 'Li*'
Test_List_001             Unnamed NSs & GUI instances are ignored.                                        
Test_List_002             Just GUI instances are ignored                                                  
Test_List_003             Nothing is ignored at all; instances of GUI objects are treaded like namespaces 
Test_List_004             Nothing is ignored but class instances                                          
Test_List_005             Scan all object types but with "Recursive"≡None                                 
Test_List_006             Scan all object types but with "Recursive"≡Just one level                       
Test_List_007             Search for "Hello" with a ref and an unnamed namespace                          
    1 7 99 L 'List'
Test_List_001             Unnamed NSs & GUI instances are ignored.               
Test_List_007             Search for "Hello" with a ref and an unnamed namespace   
      ⍴L''
99 2
```

Notes:

* Group names must be specified fully or you must specify an asterisk at the end.
* Test case numbers that do not exist are simply ignored.
* The result is a matrix with two columns, therefore `(L'')[;1]` returns a vector with test case names.

#### Editing test cases with `E`

Occasionally you might want to edit some or even all test case functions. That can be easily achieved 

```
      E''            ⍝ Edit all test cases
      E (L'ZZZ')[;1] ⍝ Edit all test cases of the group "ZZZ"
      
```


## Best Practices

* Try to keep your test cases simple and test just one thing at a time, for example just one method at a time. 

* For more complex methods (for example, excepts different kinds of arguments) create a group with the method name as group name.

* Create everything you need on the fly and tidy up afterwards. Or more precisely, tidy up (leftovers!), prepare, test, tidy up again. In other words, make the test case "stand-alone".

  The exception from this rule is when _all_ test cases require the same pre-condition like, say, a database connection. In that case establish what's needed in a function [`Initial`](#Initialisation) and use a function [`CleanUp`](#cleanUp) to get rid of it.

* Avoid a test case relying on changes made by an earlier test case. It's a tempting thing to do but you will almost certainly regret this later.

* Notice that the DRY principle (don't repeat yourself) can and should be ignored when it comes to test cases: any test case should read from top to bottom like an independent story that can be understood by itself.
