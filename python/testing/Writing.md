## Let's start writing some tests

**Based on materials by Mike Jackson, Katy Huff, Paul Ivanov, Rachel Slaybaugh, and Anthony Scopatz.**

### Exceptions

In the file [tryexcept0.py](python-code/tryexcept0.py) we have a simple function:

    def divide_it(x, y):
       try:
           out = x / y
       except:
           print '   Divide by zero!'
           out = None
       return out

    divide_it(4,2)
    divide_it(4,0)

The `except` block provides a mechanism for handling exceptions which may be raised by the statements in the code. 


    try:
    ....
    except:
    ....

If we didn't use the `except` block, a `ZeroDivisionError` would be raised and the program would terminate. But since we specified what to do in such case (`print ' Divide by zero!' ` and return `None`), the program will carry on  its execution (if there were any further instructions to be exectuted). We used a generic `except` block, that is, it will handle any exception raised.

__Exercise__   
 It is a good practice to be more specific and handle different type of errors which may be raised in a relevant way. How could we modify our function so that when we call `divide_it('a','b')` it displays with a different custom message (eg. "Provide a number")?

 
Now, the exception is *caught* by the `except` block. This is a *runtime test*. It alerts the user to exceptional behavior in the code. Often, exceptions are related to functions that depend on input that is unknown at compile time. Such tests make our code robust and allows our code to behave gracefully - they anticipate problematic values and handle them.

Often, we want to pass such errors to other points in our program rather than just print a message and continue. So, for example we could do,

    except TypeError:
        raise ValueError('The input is not a sequence e.g. a string or list')

which raises a new exception (`ValueError`), with a more meaningful message. If writing a complex application, our user interface could then present this to the user e.g. as a dialog box.

Runtime tests don't test our functions behaviour or whether it's implemented correctly. So, we can add some tests using a print statement:

    def divide_it(x, y):
       try:
           out = x / y
       except:
           print '   Divide by zero!'
           out = None
       return out


    print "4 divided by 2 is ", divide_it(4,2)
    print "6 divided by 3 is ", divide_it(6,3)

But we'd have to visually inspect the results to see they are as expected. So, let's have the computer do that for us and make our lives easier, and save us time in checking. We can use a Python statement `assert`: 

    assert divide_it(4,2) == 2
    assert divide_it(6,3) == 2
    assert divide_it(10,2) == 5

`assert` checks whether a condition is true and, if not, raises an exception. An example of very simple set of tests is:

    def testTrue():
        "Thruth-iness test"
         assert True == 1

    def testFalse():
        "Fact-brication test"
        assert False == 0

We explicitly list the expected result in each statement. But, by doing this there is a risk that we mistype one. A good design principle is to define constant values in one place only. 

For example, for testing our function `divide_it(x,y)` we could create a dictionary with the results:

    DIVISION_RESULTS = {'4/2':, 2, '6/3': 2, '10/2': 5}

Now we can just refer to that,

    assert divide_it(4,2) ==  DIVISION_RESULTS['4/2']
    assert divide_it(6,3) == DIVISION_RESULTS['6/3']
    assert divide_it(10,2) == DIVISION_RESULTS['10/2']
    
But this isn't very modular, and modularity is a good design principle, so let's define some test functions,

    def test_4_2():
        assert divide_it(4,2) ==  DIVISION_RESULTS['4/2']
    def test_6_3():
        assert divide_it(6,3) == DIVISION_RESULTS['6/3']
    def test_10_2():
        assert divide_it(10,2) == DIVISION_RESULTS['10/2']
        
    test_4_2()
    test_6_3()
    test_10_2()

And, rather than have our tests and code in the same file, let's separate them out. So, let's create

    $ nano test_divide_it.py

Now, our function and the division results data are in `division_func.py` (let's create a copy of `tryexcept0.py`) and we want to refer to them in `test_divide_it.py` file, we need to *import* them. We can do this as,

    from division_func import divide_it
    from division_func import DIVISION_RESULTS

Then we can add all our test functions and function calls to this file. And run the tests,

    $ python test_divide_it.py

Typically, a test function,

* Sets up some inputs and the associated expected outputs. The expected outputs might be a single number, a range of numbers, some text, a file, a set of files, or whatever.
* Runs the function or component being tested on the inputs to get some actual outputs.
* Checks that the actual outputs match the expected outputs. We use assertions as part of this checking. We can check both that conditions hold and that conditions do not hold.

So, we could rewrite `test_4_2`, as the more, verbose, but equivalent,

    def test_4_2():
        expected = DIVISION_RESULTS['4/2']
        actual = divide_it(4,2)                   
        assert expected == actual

We can get some more meaningful messages when using assert. Let's look at the example below:

    
    def do_string_stuff(val):
        assert type(val) == type("")
        print ">" + val + "< length:", len(val)
        
We could add a custom message which will inform us that what type was entered instead of string:
    
    def do_string_stuff_better(val):
        val_type = type(val)
        assert val_type == type(""), "Given a %s" % (str(val_type))
        print ">" + val + "< length:", len(val)
        


## `nose` - a Python test framework

`nose` is a test framework for Python that will automatically find, run and report on tests written in Python. It is an example of what has been termed an *[xUnit test framework](http://en.wikipedia.org/wiki/XUnit)*, perhaps the most famous being JUnit for Java.

To use `nose`, we write test functions, as we've been doing, with the prefix `test_` and put these in files, likewise prefixed by `test_`. The prefixes `Test-`, `Test_` and `test-` can also be used.

To run `nose` for our tests, we can do,

    $ nosetests test_divide_it.py

Each `.` corresponds to a successful test. And to prove `nose` is finding our tests, let's remove the function calls from `test_divide_it.py` and try again,

    $ nosetests test_divide_it.py

`nose` defines additional functions which can be used to check for a rich range of conditions, for example `assert_equal`, `assert_true` or `assert_false`:

    $ python
    >>> from nose.tools import *

    >>> expected = 123
    >>> actual = 123
    >>> assert_equal(expected, actual)
    >>> actual = 456   
    >>> assert_equal(expected, actual)
    
    >>> expected = "GATTACCA"
    >>> actual = ["GATC", "GATTACCA"]
    >>> assert_true(expected in actual)
    >>> assert_false(expected in actual)

We can add more information to the failure messages by providing additional string arguments e.g.

    >>> assert_true("GTA" in actual, "Expected value was not in the output list")
    
We can also use `assert_raises` :

    assert_raises(exception, func, *args, **kwargs)

`assert raises` is used for where we want to test that an exception is raised, if for example, we give a function a bad input. 

   


## Write some more tests

Let's spend a few minutes coming up with some more tests for `divide_it`. Consider,

* What haven't we tested for so far?  
* Have we covered all the types of arguments (input)) we can expect? 
* In addition to test functions, other types of runtime test could we add to `divide_it`?

For example, we could add a test to deal with:

    divide_it('a','b) 

It requires us to check whether an exception was raised. Let's rewrite our function again:

    def divide_it(x, y):
        try:
            out = x / y
        except ZeroDivisionError:
            print '   Divide by zero!'
            out = None
        except:
    	    raise TypeError (' Provide a number!')
            out = None    
       return out
       
 And now we can write a test which checks if the fuction actually raises the error:
 
    def test_divide_it_ab():
        try:
            divide_it('a','b') 
            assert False
        except TypeError:
            assert True
        
This is like catching a runtime error. If an exception is raised then our test passes (`assert True`), else if no exception is raised, it fails. Alternatively, we can use `assert_raises` from `nose.tools`,

    from nose.tools import assert_raises

    def test_divide_it_ab():
        assert_raises(TypeError, divide_it, 'a','b')

The assert fails if the named exception is *not* raised. If the function raises the right error, then the test passes. We're  we're checking (testing) the behaviour of the function. We're trying to 'force' the function to raise the 'correct' error and see if it really does that.


Previous: [Testing](README.md) Next: [Testing in practice](RealWorld.md)
