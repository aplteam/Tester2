# `Tester2` versus `Tester`

## To whom it concerns

This document is of interest only if you have used `Tester` so far, and now want to replace it by `Tester2`, which is strongly recommended.

Note that though `Tester2` implements the same principle ideas as `Tester`, the approach is radically different.

## Overview 

Although `Tester2` has inherited the general ideas and concepts from the `Tester` class, the new `Tester2` class has a very different workflow. Methods have been removed, added and renamed. It was therefore decided to start fresh with a new class.

`Tester` was created around 2010. Although it's solved its purpose technology has advanced. Also, during all these years a couple of design problems `Tester` was suffering from raised their heads. 

`Tester2` is an attempt to integrate (almost) latest technology (version 17.0 of Dyalog APL) and to overcome those aforementioned design obstacles.


## Differences in detail

## `Tester2` needs to be instantiated.

While `Tester` offered a function `EstablishHelpersIn` which injected a significant number of functions into the namespace that hosts test cases `Tester2` _must_ be instantiated.

All methods and all symbolic names are available via the instance, and only via the instance. Therefore:

```
      T←⎕NEW Tester2 HomeOfYourTestCases
      ⊃⍴T.⎕nl -3
24
      ⊃⍴T.⎕nl -2
13
```

The advantage is that your namespace is not cluttered with those methods and symbolic names. The disadvantage is that you must use the instance name now.

## Symbolic names got renamed

In `Tester` all symbolic names started with a `∆` character.

In `Tester2` they start with an underscore (`_`). This is a very common convention for symbolic names (or constants) and was therefore adapted.


## Custom constants added

In addition to the pre-defined symbolic names you can define your own ones; see [Custom constants](#custom-constants) for details.


## Couple of helpers have been renamed


| Old name | New name              |
|----------|-----------------------|
| `E`      | `T.EditTestFunctions` |
| `L`      | `T.ListTestFunctions` |
| `G`      | `T.ListGroups`        |

## INI files

When `Tester` found one or two INI files they were merged, converted into a single flat namespace holding variables, and that namespace was created in the hosting namespace as `INI`.

With `Tester2` you won't be surprised to learn that the INI file is now part of the instance. With an instance `T` you can address a variable `foo` as `T.INI.foo`.


## The `Run*` functions

The number of `Run*` functions has been reduced, but at the same time a general function `Run__` is now available that can be used for all possible scenarios.


## The result of the `Run*` functions

With `Tester2` all `Run*` function return 0, 1 or 2 as return code:

* `0` means that all test cases that got executed returned `_OK`.
* `1` means that at least one test function either crashed or returned `_Failed`.
* `2` means that a function `Initial` was found and executed but failed to return a 0, meaning it could not initialize.


## `Cleanup` function may accept a right argument

In `Tester` the function `Cleanup` was expected to be niladic. With `Tester2` it may be monadic.


## Initialising on a per-group basis

In addition to the [global `Initial` function](#initialisation) you can also have [group-specific `Initial` functions](#initialisation-for-groups), a feature that was not available with `Tester`.

A function is recognized as a group-specific `Initial` function by naming convention: for a group `Foo` the function's name must be `Initial_Foo` for it to be recognized.


## The `List` function got renamed

When a parameter namespace is created by calling the instance method `CreateParms` it comes with a built-in function `List`. In `Tester` the name of that function was `∆List`.


## New method `QuitTests`

`Tester` did not offer a way to quit from a test suite after having executed only a subset of the test functions, for example after a test function breaks and you don't want to continue.

`Tester2` offers the method `QuitTests` for this purpose. When executed it prints a `⎕SIGNAL` command to the session that when executed will make `Tester2` finish the currently running tests in an orderly fashion, including executing any `Cleanup` function(s).