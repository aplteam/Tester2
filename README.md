# Test framework for Unit tests in Dyalog APL


`Tester2` is a member of the APLTree library. The library is a collection of classes etc. that aim to support the Dyalog APL programmer. Search GitHub for "apltree" and you will find solutions to many every-day problems Dyalog APL programmers might have to solve.

**_Note:_** `Tester2` is the successor of the `Tester` class. Although it has inherited the general concepts from the `Tester` class the new `Test2` class has a very different workflow. It was therefore decided to start with a new class.


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

  If you want to make sure that the GUI survives until the close box is clicked etc. then it is not a bad idea to assign the ref to the GUI to a variable "inside" the instance. (It's not really inside the instance but for our purposes it behaves as if it is)

  That would be something like `T.gui←T.RunGUI ⍬`


## Details 


### Terminology

Note that test cases causing a crash are considered "broken". Test cases that do not return the expected result are considered "failing".


### Assumptions and preconditions

1. The `#.Tester2` class assumes that all your tests are hosted by a namespace. It may be an ordinary or a scripted namespace, but it must not be an unnamed namespace. 

1. You must create an instance of the `Tester2` class in order to do anything useful. This is _the_ major difference to the now deprecated `Tester` class.

   The constructor demands a right argument: that must be a reference to the namespace that hosts all your test cases.

   Example: given that all your test cases live in `Foo` then you could create an instance `T` with: `T←⎕NEW Tester2 Foo`

1. All test functions inside that namespace are expected to start their names with `Test_` followed either by some digits or a group name followed by an underscore followed by some digits.

   It is recommended that in case you use groups you assign _all_ your test cases to a group.

1. The number of digits you use for numbering is not restricted: `Test_foo_1` is fine, and so is `Test_foo_0000001`. However, they should be consistent, at least within a group.

   When starting to implement test cases you are advised to leave gaps: `Test_foo_001`, `Test_foo_010` etc.

1. In each test function the first line after the header must carry a comment telling what is actually tested.

   Keep in mind that later this is the only way to tell one test case from the others without reading the code, so be as clear as you can possibly be. 

   You are restricted to a single line, and you should keep it short enough to be displayed with a reasonably setting of `⎕PW`.


### Executing test cases

There are some typical scenarios when you want to run test cases. All these scenarios are covered by functions that are available as instance methods. Their names start with `Run`. We wqill discuss them soon.
    
* Run with error trapping and return a code together with the log (a vector of text vectors) reporting details. The code is `0` in case all test cases executed successfully and something else otherwise.

  This is perfect for, say, running all tests as part of an automated build process. See `Run` for details.
   
* Run without error trapping. Use this when you want to investigate why particular test cases crash or fail. See `RunDebug` for details.

  In case a problem is detected the test stops straight away, so you can investigate.
   
* Run in "batch mode". This means that test cases that require a human being in front of the monitor are **not** executed. See `RunBatchTests` for details. 

  This is the equivalent to `Run` but without any test cases that require a human being in front of the monitor.

  Of course it is best not to rely on a human being but this cannot always be avoided, or it would require too much of an effort.

* Run in "batch mode" but without error trapping. This is the equivalent of running `RunDebug`. See `RunBatchTestsInDebugMode`.

* Run only selected test cases, either by specifying a group name or by specifying test case numbers or both. See `RunThese` for details.

* There is also a method `RunGUI` available which does the same as `RunDebug` but with a GUI. Normally you will prefer `RunDebug` over `RunGUI` but there are circumstances when it is the other way around. See `Tester2`'s own test cases as an example.

### The syntax of test functions

1. Every test function must accept a right argument which is a two-item vector of Booleans:

   1. `debugFlag` (0=use error trapping)
   1. `batchFlag` (0=does not run in batch mode)

1. Every test function must return a result. You are advised to assign one of the read-only fields every instance of `Tester2` comes with. This is much more readable than a simple integer, and it is easier to find as well. See [Constants](#) for details.


## Work flow

No matter which of the `Run*` functions (`Run`, `RunGUI`, `RunBatchTests`, `RunDebug`, `RunBatchTestsInDebugMode`, `RunThese`) you are going to call, the workflow is always the same:


### Create an instance

First you need an instance. Let's assume that you have a project in `#.Foo` and that the project's test cases are hosted by the namespace `#.Foo.TestCases`. You create an instance with:

```
      T←⎕NEW #.Tester2 #.Foo.TestCases
```

From then on all `Run`* functions, all constants and all other helpers are available via the instance `T`. Some examples:

```
      T.RunGUI ⍬
      T.∆OK
      T.PassesIf
      T.⎕nl -3    ⍝ Produces a list of all public instance methods
```

When you call one of the `Run` functions the same steps are executed:


### INI files (optional)

First of all the `Run*` methods checks whether there is a file `testcases.ini` in the current directory. If this is the case that INI file is processed. Use this to specify general stuff that does not depend on a certain computer/environment.

Then it checks for a file `testcases_{computername}.ini`. If this file exists then it is processed. Use this to specify computer-dependent variables.

Note that if one of the two INI files exists there will be a flat namespace `{yourTestNamespace}.INI`, meaning that sections in the INI file are ignored. An example: if your test functions are hosted by a namespace `Foo.TestCases` and your INI file specifies a variable `hello` as holding "world" then:

```
       'world'≡#.Foo.TestCases.INI.hello
1
```


### Initialisation

In the next step the `Run*` method checks whether there is a function `Initial` in the hosting namespace. If that is the case then this function is executed.

Note that the function must be either niladic or monadic, and it may return no result or a Boolean result. A 1 means that function did what it is supposed to do (=same as no result) while a 0 means it could not initialize.

Of course you can simply execute `→` on a single line in your `Initial` function if any requirement is not met but that would also mean that if you run your test cases automatically somehow then this would not work: your workflow would be interrupted by the `→` statement.

In such cases `Initial` should return a 0 indicating failure. Also, part of the initialization might have been carried out, and a function `Cleanup` (discussed in a second) might get rid of any left-overs.

If the function is monadic a reference pointing to the parameter space is passed as the right argument. By checking this `Initial` can, for example, find out whether the cases are running in batch mode or not.

Use this function to create stuff that's needed by **all** test cases, or tell the user something important (only if the batch flag is false of course).

It might  be a good idea for _all_ test functions to tidy up in line 1, just in case a failing test case has left something behind; see [CleanUp](#cleanUp) for details.


### Finally: running the test cases

Now the test cases are executed one by one.


### Tidying up: `CleanUp`{#cleanUp}

After the last test case was executed the `Run*` function checks whether there is a function `Cleanup` in the namespace hosting your test cases. If that's the case then this function is executed. Such a function must be a niladic, no-result returning function.


### INI file again

Finally the namespace `INI` holding variables populated from your INI file(s) is deleted.


## Instance methods offered by `Tester2`

The methods fall into four groups:


### Running test cases

`Run`, `RunGUI`, `RunDebug`, `RunThese`, `RunBatchTest` and `RunBatchTestsInDebugMode` are running all or selected test cases with or without error trapping.


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


You can get a test function template by calling the instance method `GetTestTemplate`. You must provide a number as right argument. The test function will be named `Test_{yourNumber}`. You can also optionally provide a group name via the left argument. See these examples which assume that an instance of `Tester2` is available as `T` and that you are inside the (ordinary, non-scripted) namespace that hosts the test cases:

```
      ≢'T'⎕NL 3
0
      T.GetTestTemplate 3
Test_003
     ≢'T'⎕NL 3
1
      'Misc' T.GetTestTemplate 1
Test_Misc_001
     ≢'T'⎕NL 3
2
```

Note that in case such a test function already exists you will be prompted.

Note that for technical reasons a test function cannot be established by `GetTestTemplate` when the namespace hosting the test cases is scripted. If that is the case you will be prompted for copying the code to the clipboard and then inserting the code into the script yourself.


### Constants

These are the public read-only instance fields that act like constants:


| Name                  | Meaning                                                                     |
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



### Misc

`ListGroups`

: List all groups, if any.

`ListTestFunctions`

: List all test functions --- if the right argument is empty --- together with the first comment line in the function.

: The right argument can be used to restrict output to a group, the optional left argument to restrict output to certain numbers.

: For example, `T.ListTestFunctions 'Misc'` would list all test functions of the group "Misc" while `T.ListTestFunctions 'M*'` would list all test functions that start their names with `Test_M`.

`EditTestFunctions`

: Edit all test functions (empty right argument) or the function(s) identified by the right argument, be it a simple character vector (single name), a vector of character vectors or a matrix.

: Note that `EditTestFunctions` can process the result of `ListTestFunctions`.

## Examples

### `Tester2`'s `Run*` functions

#### `RunDebug`

Usually you will run this function with a 0 as right argument.

However, sometimes you want to trace through a test case, and that is when specifying a 1 as right argument comes in handy: `RunDebug` would then stop just before any of the test cases is actually executed, after having processed any INI file(s) and executed any `Initial` function.

#### RunGUI

With `RunGUI` you can achieve the same as with `RunDebug` but it is a Windows-only feature. It might be easier to start with `RunGUI` but you might switch to `RunDebug` later, if only because it is significantly faster.

However, there are situations when `RunGUI` is indispensable: `Tester2`'s own test cases are almost impossible to follow without it, for example. It's also very useful in order to demonstrate the features of the `Tester2` class.


#### `RunThese`

A particularly helpful method while developing/enhancing stuff is `RunThese`. The function allows you to run just selected test functions rather than a whole test suite.

If you now think, well, why not just call any function `Test_001` myself then imagine a situation when all your test cases depend on an INI file or the execution of `Initial` ot both. That is exactly the advantage of `RunThese`: it carries out all these steps, and also executes the `Cleanup` function in case there is one.

`RunThese` offers the following options:

```
T.RunThese 1 2               ⍝ Execute test cases 1 & 2 that do not belong to any group
1 T.RunThese 1 2             ⍝ Same but stop just before a test case is actually executed
T.RunThese 'Group1'          ⍝ Execute all test cases belonging to group 1
T.RunThese 'Group1' (2 3)    ⍝ Execute test cases 2 & 3 of group 1
T.RunThese 'Group1' 2 3      ⍝ Same as before
T.RunThese 'Misc'            ⍝ Execute all test cases of the group "Misc"
T.RunThese 'L*'              ⍝ Execute all test cases of all groups starting with "L"
T.RunThese 'Test_Group1_001' ⍝ Executes just this test case
```

Sometimes you want to trace through test cases. This can be achieved be specifying a 1 as the (optional) left argument. `RunThese` would then stop just before any test case is actually executed, after processing any INI file(s), executing any `Initial` function before executing the test cases, and any `Cleanup` function after having executed all test functions.

### Managing test cases 

There are a couple of methods available that assist you in managing test cases.

The examples stem from the `Fire` project.

#### Listing groups with `ListGroups`

`ListGroups` lists all groups:

```
      ListGroups
acre           
InternalMethods
List           
Misc           
Replace        
ReportGhosts   
Search 
```

#### Listing test functions with `ListTestFunctions`

`ListTestFunctions` requires a right argument (empty or group name) and accepts an optional left argument (test case numbers).

```
      ⍴ListTestFunctions ''
99 150
        
      ListTestFunctions 'Li*'
Test_List_001             Unnamed NSs & GUI instances are ignored.                                        
Test_List_002             Just GUI instances are ignored                                                  
Test_List_003             Nothing is ignored at all; instances of GUI objects are treaded like namespaces 
Test_List_004             Nothing is ignored but class instances                                          
Test_List_005             Scan all object types but with "Recursive"≡None                                 
Test_List_006             Scan all object types but with "Recursive"≡Just one level                       
Test_List_007             Search for "Hello" with a ref and an unnamed namespace                          
    1 7 99 ListTestFunctions 'List'
Test_List_001             Unnamed NSs & GUI instances are ignored.               
Test_List_007             Search for "Hello" with a ref and an unnamed namespace   
      ⍴ListTestFunctions''
99 2
```

Notes:

* Group names must be specified fully or you must specify an asterisk at the end.
* Test case numbers that do not exist are simply ignored.
* The result is a matrix with two columns, therefore `(ListTestFunctions'')[;1]` returns a vector with test case names.

#### Editing test cases with `EditTestFunction`

Occasionally you might want to edit some or even all test case functions. That can be easily achieved 

```
EditTestFunction''                             ⍝ Edit all test cases
EditTestFunction (ListTestFunctions'ZZZ')[;1]  ⍝ Edit all test cases of the group "ZZZ"
      
```

#### Rename a test function with `RenameTestFnsTo`

Renaming a test function is actually harder --- and more dangerous --- than first meets the eye, hence the method `RenameTestFnsTo` is there to assist you.

The syntax is easy:

```
'oldname' T.RenameTestFnsTo 'newname'
```

In a real example lets assume that at first you started numbering your test functions. Soon you came to realize that groups would be helpfull. But it is awkward to have some test functions carrying just numbers and others group names _and_ numbers. Therefore it is the good idea to rename those with just a number into `Test_Misc_` followed by the number.

Therefore this would make sense:

```
'Test_001' T.RenameTestFnsTo 'Test_Misc_001'
```

`RenameTestFnsTo` does a couple of things for you:

1. It copies `Test_001` to `Test_Misc_001`
1. It tells the project management system acre[^acre] (if it is around) about the introduction of `Test_Misc_001`
1. It deletes the function `Test_001`
1. It tells acre about the deletion of `Test_001`


## Best Practices

* Try to keep your test cases simple and test just one thing at a time, for example just one method at a time. 

* For more complex methods (for example when the method accepts different kinds of arguments) create a group with the method name as group name.

* Create everything you need on the fly and tidy up afterwards. Or more precisely, tidy up (leftovers!), prepare, test, tidy up again. In other words, make the test case "stand-alone".

  The exception from this rule is when _all_ test cases require the same pre-condition like, say, a database connection. In that case establish what's needed in a function [`Initial`](#Initialisation) and use a function [`CleanUp`](#cleanUp) to get rid of it.

* Avoid a test case relying on changes made by an earlier test case. It's a tempting thing to do but you will almost certainly regret this later.

* Notice that the DRY principle (don't repeat yourself) can and should be ignored when it comes to test cases: any test case should read from top to bottom like an independent story that can be understood by itself.

[^acre]: If you do not know what acre is see <https://github.com/the-carlisle-group/Acre-Desktop>