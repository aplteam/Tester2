# Tester2: Test Framework for Unit Tests in Dyalog APL


`Tester2` offers a comprehensive suite of features for building and managing test cases in complex scenarios. Here's a brief overview of its capabilities:

* **Automatic Test Execution**: Tests can be executed automatically based on naming conventions.

* **Test Organisation**: Tests can be organised into groups for better management.

* **Flexible Execution Options**: You can execute all tests, specific groups of tests, all groups except certain ones, or individual tests.

* **Flow Control with Helper Functions**: These functions allow for the continuation or immediate stop of test execution upon failure, facilitating in-depth investigation when needed.

* **INI File Integration**: You can include an INI file for, say, machine-dependent parameters, making your tests more adaptable to different environments.

* **Initialisation and Clean-Up Functions**: You can define functions that run before any tests are executed and after all tests are completed. These can include group-specific initialisation and clean-up functions.

* **Test Function Return Values**: Test functions return integers that are translated into symbolic names like “OK”, “Failure”, “MacOnly”, etc. You can also define additional symbolic names for custom purposes.

* **Execution Control**: The framework can be set to pause before executing an initialisation, test or clean-up function, or any combination of them.

* **Batch mode**: The tests can be executed in batch mode, resulting in a Boolean indicating success or failure and a detailed report.

* **Helper Functions for Management**: These include functions for listing groups and tests and renaming test functions or groups.

* **Code Coverage Analysis**: If the optional `CodeCoverage` package is available, it provides a report on the code coverage of your tests, helping to identify untested parts of your code.

This feature set makes `Tester2` an effective tool for testing in various scenarios, especially in projects where test case management and execution flexibility are crucial. 

An [exhaustive documentation](https://aplteam.github.io/Tester2/Tester2-Reference.html) is available.
