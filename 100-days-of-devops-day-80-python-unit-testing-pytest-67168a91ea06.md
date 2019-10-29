
# 100 Days of DevOps — Day 80-Python Unit Testing(Pytest)

Welcome to Day 80 of 100 Days of DevOps, Focus for today is Python Unit Testing(Pytest)

*A good programmer always tests his code before merging moving his code to production. The way to do it is by writing an automated test.*

*Pytest is a full-featured Python testing tool which means it does everything from test collection to run the test, to give you the output on whether which test failed or passed.*

*Let’s take an example of a simple function(**mathexample.py**), which takes two arguments and return the sum/difference of two number*

    ***def add_two(**num1,num2**):
        return **num1 **+ **num2*

    ***def sub_two(**num1,num2**):
        return **num1 **- **num2*

*The first step is to install pytest*

    *pip3 install pytest*

*Unittest look like this*

    ***import mathexample**  
    **def test_add_two():
        **add **= **mathexample.add_two**(**1,2**)
        assert **add **== **3*

    ***def test_sub_two():
        **substract **= **mathexample.sub_two**(**1,2**)
        assert **substract **== -**1*

*Let take a look what’s going on here*

* *First, we import the module(or the script we wrote above)*

* *We appended test in front of the function we defined above and the reason for that pytest using some discovery rules to find out these test cases and one of the discovery rule is to look for a prefix which has test underscore*

* *To verify your output we are going to use assert*

*Now there are two ways to execute these tests*

* ***python -m pytest **if we just run to run this command, it will do it will recursively goes into the directory and look for any file with **test_ **and execute any method with** test_ prefix***

***Output***

    *python -m pytest
    =========================================== test session starts ============================================
    platform darwin -- Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 2 items *

    *test_mathexample.py ..*

    *========================================= 2 passed in 0.01 seconds =========================================*

* *The second way to running the same test is via py.test(with verbose output)*

    *py.test -v
    =========================================================================================== test session starts ============================================================================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0 — /Users/plakhera/anaconda/bin/python
    cachedir: .cache
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 2 items*

    *test_mathexample.py::test_add_two PASSED
    test_mathexample.py::test_sub_two PASSED*

    *========================================================================================= 2 passed in 0.01 seconds =========================================================================================*

*Failure output will look like this*

    *py.test -v
    =========================================================================================== test session starts ============================================================================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0 — /Users/plakhera/anaconda/bin/python
    cachedir: .cache
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 2 items*

    *test_mathexample.py::test_add_two FAILED
    test_mathexample.py::test_sub_two PASSED*

    *================================================================================================= FAILURES =================================================================================================
    _______________________________________________________________________________________________ test_add_two _______________________________________________________________________________________________*

    *def test_add_two():
     add = mathexample.add_two(1,2)
    > assert add == 4
    E assert 3 == 4*

    *test_mathexample.py:4: AssertionError
    ==================================================================================== 1 failed, 1 passed in 0.03 seconds ====================================================================================*

*Generally, we don’t write a single test case for testing purpose but we wrote multiple test cases*

    ***import **mathexample
    **def test_add_two_1():
        **add **= **mathexample.add_two**(**1,2**)
        assert **add **== **3*

    ***def test_add_two_2():
        **add **= **mathexample.add_two**(**1,6**)
        assert **add **== **7*

    ***def test_sub_two():
        **substract **= **mathexample.sub_two**(**1,2**)
        assert **substract **== -**1*

*Output*

    *py.test -v
    =========================================================================================== test session starts ============================================================================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0 — /Users/plakhera/anaconda/bin/python
    cachedir: .cache
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 3 items*

    *test_mathexample.py::test_add_two_1 PASSED
    test_mathexample.py::test_add_two_2 PASSED
    test_mathexample.py::test_sub_two PASSED*

    *========================================================================================= 3 passed in 0.01 seconds =========================================================================================*

*But as you can see we have a problem in the above code, we are defining two functions to perform the same task **test_add_two_1 and test_add_two_2, **so there is a duplication of code here.*

*The much better way to write the same code is to parametrize the same code*

    ***import **mathexample
    **import **pytest*

    *# We need to use this special decorator when we need to parametrize our code(We are passing input/output as tuple)
    **@**pytest.mark.parametrize**("input1, input2, output"**,
                             **[
                                 (**1,2,3**)**,
                                 **(**1,6,7**)
    ***

    ***                         ]
                             )
    def test_add_two(**input1, input2,output**):
        **add **= **mathexample.add_two**(**input1, input2**)
        assert **add **== **output*

    ***@**pytest.mark.parametrize**("input1, input2, output"**,
                             **[
                                 (**1,2,**-**1**)***

    ***                         ]
                             )***

    ***def test_sub_two(**input1, input2,output**):
        **substract **= **mathexample.sub_two**(**input1, input2**)
        assert **substract **== **output*

*The output will look like this*

    *py.test -v
    =========================================================================================== test session starts ============================================================================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0 — /Users/plakhera/anaconda/bin/python
    cachedir: .cache
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 3 items*

    *test_mathexample.py::test_add_two[1–2–3] PASSED
    test_mathexample.py::test_add_two[1–6–7] PASSED
    test_mathexample.py::test_sub_two[1–2–1] PASSED*

    *========================================================================================= 3 passed in 0.01 seconds =========================================================================================*

*There might be times where we selectively need to run test cases, to do that we need to use special decorator **pytest.mark.skip***

    ***import **mathexample
    **import **pytest*

    ***@pytest.mark.parametrize("input1, input2, output"**,
                             **[
                                 (**1,2,3**)**,
                                 **(**1,6,7**)
    ***

    ***                         ]
                             )
    @**pytest.mark.skip**(**reason**="Not required for demo purpose")
    def test_add_two(**input1, input2,output**):
        **add **= **mathexample.add_two**(**input1, input2**)
        assert **add **== **output*

    ***@**pytest.mark.parametrize**("input1, input2, output"**,
                             **[
                                 (**1,2,**-**1**)***

    ***                         ]
                             )***

    ***def test_sub_two(**input1, input2,output**):
        **substract **= **mathexample.sub_two**(**input1, input2**)
        assert **substract **== **output*

*Output*

    *py.test -v
    =========================================================================================== test session starts ============================================================================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0 — /Users/plakhera/anaconda/bin/python
    cachedir: .cache
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 3 items*

    *test_mathexample.py::test_add_two[1–2–3] **SKIPPED**
    test_mathexample.py::test_add_two[1–6–7] **SKIPPED**
    test_mathexample.py::test_sub_two[1–2–1] PASSED*

    *=================================================================================== 1 passed, 2 skipped in 0.01 seconds ====================================================================================*

*We can also skip the test based on certain conditions and for that, we need to use **pytest.mark.skipif***

    ***import **mathexample
    **import **pytest
    **import **sys*

    ***@**pytest.mark.parametrize**("input1, input2, output"**,
                             **[
                                 (**1,2,3**)**,
                                 **(**1,6,7**)
    ***

    ***                         ]
                             )
    @pytest.mark.skipif(**sys.version_info **< (**3,5**)**,reason**="Not required for demo purpose")
    def test_add_two(**input1, input2,output**):
        **add **= **mathexample.add_two**(**input1, input2**)
        assert **add **== **output*

    ***@**pytest.mark.parametrize**("input1, input2, output"**,
                             **[
                                 (**1,2,**-**1**)***

    ***                         ]
                             )***

    ***def test_sub_two(**input1, input2,output**):
        **substract **= **mathexample.sub_two**(**input1, input2**)
        assert **substract **== **output*

* *Here we are saying if Python version is < 3.5 please skip these tests*

*Now there is one much easy way to run specific test*

    ***import **mathexample
    *

    ***@**pytest.mark.parametrize**("input1, input2, output"**,
                             **[
                                 (**1,2,3**)**,
                                 **(**1,6,7**)
    ***

    ***                         ]
                             )
    def test_add_two(**input1, input2,output**):
        **add **= **mathexample.add_two**(**input1, input2**)
        assert **add **== **output*

    ***@**pytest.mark.parametrize**("input1, input2, output"**,
                             **[
                                 (**1,2,**-**1**)***

    ***                         ]
                             )***

    ***def test_sub_two(**input1, input2,output**):
        **substract **= **mathexample.sub_two**(**input1, input2**)
        assert **substract **== **output*

*Now here if I want to run only subtract test case, I need to pass -k option*

    *pytest -k sub
    =========================================== test session starts ============================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 3 items*

    *test_mathexample.py .*

    *============================================ 2 tests deselected ============================================
    ================================== 1 passed, 2 deselected in 0.01 seconds ====================*

*where -k*

    *-k EXPRESSION only run tests which match the given substring
     expression. An expression is a python evaluatable
     expression where all names are substring-matched
     against test names and their parent classes. Example:
     -k ‘test_method or test_other’ matches all test
     functions and classes whose name contains
     ‘test_method’ or ‘test_other’. Additionally keywords
     are matched to classes and functions containing extra
     names in their ‘extra_keyword_matches’ set, as well as
     functions which have names assigned directly to them.*

*We can also set our own custom marker*

    ***import **mathexample
    **import **pytest*

    ***@**pytest.mark.add*

    ***def test_add_two():
        **add **= **mathexample.add_two**(**1,2**)
        assert **add **== **3*

    ***@**pytest.mark.sub
    **def test_sub_two():
        **substract **= **mathexample.sub_two**(**1,2**)
        assert **substract **== -**1*

*and the way to run it*

    *pytest -m sub -v
    =========================================================================================== test session starts ============================================================================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0 — /Users/plakhera/anaconda/bin/python
    cachedir: .cache
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 2 items*

    *test_mathexample.py::test_sub_two PASSED*

    *============================================================================================ 1 tests deselected ============================================================================================
    ================================================================================== 1 passed, 1 deselected in 0.01 seconds ==================================================================================*

*The same way we can check for add*

    *pytest -m add -v
    =========================================================================================== test session starts ============================================================================================
    platform darwin — Python 3.6.0, pytest-3.0.5, py-1.4.32, pluggy-0.4.0 — /Users/plakhera/anaconda/bin/python
    cachedir: .cache
    rootdir: /Users/plakhera/Downloads/advanced_python, inifile: 
    collected 2 items*

    *test_mathexample.py::test_add_two PASSED*

    *============================================================================================ 1 tests deselected ============================================================================================
    ================================================================================== 1 passed, 1 deselected in 0.01 seconds ==================================================================================*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
[**100 Days of DevOps — Day 79-Apache Log Parser Using Python**
*Welcome to Day 79 of 100 Days of DevOps, Focus for today is Apache Log Parser Using Python*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-79-apache-log-parser-using-python-849135ed1a08)
