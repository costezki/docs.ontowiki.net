This document describes the use of the PHPCodeSniffer to check if your coding style complies to the Ontowiki/Erfurt Coding standard.

## Requirements ##

To use the PHPCodeSniffer you must install the PHP PEAR Framework

For **Ubuntu** execute:

    sudo apt-get install php-pear


For other systems or more information go to
[[PEAR Framework Installation|http://pear.php.net/manual/en/installation.php]]

## Installation ##

Go to your Ontowiki/Erfurt directory and run:

    make cs-install


This will install the latest version of the CodeSniffer and enables automatically the pre-commit code checking.

## Code Checking ##
### pre-commit ###
For automatically checking your coding style before every commit you can run the following commands in your Ontowiki/Erfurt directory.

    make cs-enable

After that the CodeSniffer will check your code before each commit.

To turn off this functionality run:

    make cs-disable

**Important**:
The CodeSniffer only check the added or changed commit files, not the deleted files and not the whole Ontowiki-Source.
## Summary View ##
If you make a commit and your code not match the coding standard you will get something like that:

    ./application/tests/CodeSniffer/pre-commit
    E.E.EE


    PHP CODE SNIFFER REPORT SUMMARY
    --------------------------------------------------------------------------------
    FILE                                                            ERRORS  WARNINGS
    --------------------------------------------------------------------------------
    /home/lars/Uni/BA/OW_default/TestClass.php                      6       0
    /home/lars/Uni/BA/OW_default/TestClass3.php                     6       0
    /home/lars/Uni/BA/OW_default/TestClass5.php                     6       0
    /home/lars/Uni/BA/OW_default/TestClass6.php                     6       0
    --------------------------------------------------------------------------------
    A TOTAL OF 24 ERROR(S) AND 0 WARNING(S) WERE FOUND IN 4 FILE(S)
    --------------------------------------------------------------------------------

    Time: 0 seconds, Memory: 2.50Mb

    make: *** [cs-check-commit] Fehler 1

This is a summary of all files, that have been checked. You can see how many files were checked and how many errors or warnings every file have caused.

## Detail View ##
To correct your code you need a more detailed view, so you can run:

    make cs-check-commit

and get something like that for every file that have been checked:

    FILE: /home/lars/Uni/BA/OW_default/TestClass.php
    --------------------------------------------------------------------------------
    FOUND 6 ERROR(S) AFFECTING 5 LINE(S)
    --------------------------------------------------------------------------------
      6 | ERROR | Protected member variable "test" must contain a leading
        |       | underscore
      9 | ERROR | Variable "test_variable" is not in valid camel caps format
     11 | ERROR | Variable "Test" is not in valid camel caps format
     13 | ERROR | Opening brace should be on a new line
     14 | ERROR | No space found after comma in function call
     14 | ERROR | Space before opening parenthesis of function call prohibited
    --------------------------------------------------------------------------------

Now you can see the description of the errors and the line where they occur.
So you have the possibility to correct your code and check it again.

**Note:** If there are less than six files to check, you get the detail view automatically and not the summary view.

## Tips ##
For **git**:

The CodeSniffer checks all files or fileparts that has been added to the index (with **git add**). If you fix some issues that were reported, you have to re-add these changes, otherwise CS will complain again and again.