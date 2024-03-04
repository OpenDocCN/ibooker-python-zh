# 第十四章：测试调试和异常

Testing rocks, but debugging? Not so much. The fact that there’s no compiler to analyzeyour code before Python executes it makes testing a critical part of development. Thegoal of this chapter is to discuss some common problems related to testing, debugging,and exception handling. It is not meant to be a gentle introduction to test-driven developmentor the unittest module. Thus, some familiarity with testing concepts isassumed.

# 14.1 测试输出到标准输出上

## 问题

You have a program that has a method whose output goes to standard Output(sys.stdout). This almost always means that it emits text to the screen. You’d like towrite a test for your code to prove that, given the proper input, the proper output isdisplayed.

## 解决方案

Using the unittest.mock module’s patch() function, it’s pretty simple to mock outsys.stdout for just a single test, and put it back again, without messy temporary vari‐ables or leaking mocked-out state between test cases.Consider, as an example, the following function in a module mymodule:

# mymodule.py

def urlprint(protocol, host, domain):url = ‘{}://{}.{}'.format(protocol, host, domain)print(url)
The built-in print function, by default, sends output to sys.stdout. In order to testthat output is actually getting there, you can mock it out using a stand-in object, andthen make assertions about what happened. Using the unittest.mock module’s patch()method makes it convenient to replace objects only within the context of a running test,returning things to their original state immediately after the test is complete. Here’s thetest code for mymodule:

from io import StringIOfrom unittest import TestCasefrom unittest.mock import patchimport mymodule

class TestURLPrint(TestCase):def test_url_gets_to_stdout(self):
protocol = ‘http'host = ‘www'domain = ‘example.com'expected_url = ‘{}://{}.{}n'.format(protocol, host, domain)

with patch(‘sys.stdout', new=StringIO()) as fake_out:mymodule.urlprint(protocol, host, domain)self.assertEqual(fake_out.getvalue(), expected_url)

## 讨论

The urlprint() function takes three arguments, and the test starts by setting up dummyarguments for each one. The expected_url variable is set to a string containing theexpected output.To run the test, the unittest.mock.patch() function is used as a context manager toreplace the value of sys.stdout with a StringIO object as a substitute. The fake_outvariable is the mock object that’s created in this process. This can be used inside thebody of the with statement to perform various checks. When the with statement com‐pletes, patch conveniently puts everything back the way it was before the test ever ran.It’s worth noting that certain C extensions to Python may write directly to standardoutput, bypassing the setting of sys.stdout. This recipe won’t help with that scenario,but it should work fine with pure Python code (if you need to capture I/O from such Cextensions, you can do it by opening a temporary file and performing various tricksinvolving file descriptors to have standard output temporarily redirected to that file).More information about capturing IO in a string and StringIO objects can be found inRecipe 5.6.

# 14.2 在单元测试中给对象打补丁

## 问题

You’re writing unit tests and need to apply patches to selected objects in order to makeassertions about how they were used in the test (e.g., assertions about being called withcertain parameters, access to selected attributes, etc.).

## 解决方案

The unittest.mock.patch() function can be used to help with this problem. It’s a littleunusual, but patch() can be used as a decorator, a context manager, or stand-alone. Forexample, here’s an example of how it’s used as a decorator:

from unittest.mock import patchimport example

@patch(‘example.func')def test1(x, mock_func):

> example.func(x) # Uses patched example.funcmock_func.assert_called_with(x)

It can also be used as a context manager:

with patch(‘example.func') as mock_func:example.func(x) # Uses patched example.funcmock_func.assert_called_with(x)
Last, but not least, you can use it to patch things manually:

p = patch(‘example.func')mock_func = p.start()example.func(x)mock_func.assert_called_with(x)p.stop()

If necessary, you can stack decorators and context managers to patch multiple objects.For example:

@patch(‘example.func1')@patch(‘example.func2')@patch(‘example.func3')def test1(mock1, mock2, mock3):

> ...

def test2():with patch(‘example.patch1') as mock1, patch(‘example.patch2') as mock2, patch(‘example.patch3') as mock3:
...

## 讨论

patch() works by taking an existing object with the fully qualified name that you pro‐vide and replacing it with a new value. The original value is then restored after thecompletion of the decorated function or context manager. By default, values are replacedwith MagicMock instances. For example:

```py
>>> x = 42
>>> with patch('__main__.x'):
...     print(x)
...
<MagicMock name='x' id='4314230032'>
>>> x
42
>>>

```

However, you can actually replace the value with anything that you wish by supplyingit as a second argument to patch():

```py
>>> x
42
>>> with patch('__main__.x', 'patched_value'):
...     print(x)
...
patched_value
>>> x
42
>>>

```

The MagicMock instances that are normally used as replacement values are meant tomimic callables and instances. They record information about usage and allow you tomake assertions. For example:

```py
>>> from unittest.mock import MagicMock
>>> m = MagicMock(return_value = 10)
>>> m(1, 2, debug=True)
10
>>> m.assert_called_with(1, 2, debug=True)
>>> m.assert_called_with(1, 2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File ".../unittest/mock.py", line 726, in assert_called_with
    raise AssertionError(msg)
AssertionError: Expected call: mock(1, 2)
Actual call: mock(1, 2, debug=True)
>>>

>>> m.upper.return_value = 'HELLO'
>>> m.upper('hello')
'HELLO'
>>> assert m.upper.called

>>> m.split.return_value = ['hello', 'world']
>>> m.split('hello world')
['hello', 'world']
>>> m.split.assert_called_with('hello world')
>>>

>>> m['blah']
<MagicMock name='mock.__getitem__()' id='4314412048'>
>>> m.__getitem__.called
True
>>> m.__getitem__.assert_called_with('blah')
>>>

```

Typically, these kinds of operations are carried out in a unit test. For example, supposeyou have some function like this:

# example.pyfrom urllib.request import urlopenimport csv

def dowprices():u = urlopen(‘[`finance.yahoo.com/d/quotes.csv`](http://finance.yahoo.com/d/quotes.csv)?s=@^DJI&f=sl1‘)lines = (line.decode(‘utf-8') for line in u)rows = (row for row in csv.reader(lines) if len(row) == 2)prices = { name:float(price) for name, price in rows }return prices
Normally, this function uses urlopen() to go fetch data off the Web and parse it. Tounit test it, you might want to give it a more predictable dataset of your own creation,however. Here’s an example using patching:

import unittestfrom unittest.mock import patchimport ioimport example

sample_data = io.BytesIO(b'‘‘“IBM”,91.1r“AA”,13.25r“MSFT”,27.72rr‘'‘)

class Tests(unittest.TestCase):
@patch(‘example.urlopen', return_value=sample_data)def test_dowprices(self, mock_urlopen):

> > p = example.dowprices()self.assertTrue(mock_urlopen.called)self.assertEqual(p,
> > 
> > {‘IBM': 91.1,‘AA': 13.25,‘MSFT' : 27.72})

if **name** == ‘**main**':unittest.main()
In this example, the urlopen() function in the example module is replaced with a mockobject that returns a BytesIO() containing sample data as a substitute.An important but subtle facet of this test is the patching of example.urlopen instead ofurllib.request.urlopen. When you are making patches, you have to use the namesas they are used in the code being tested. Since the example code uses from urllib.request import urlopen, the urlopen() function used by the dowprices() function isactually located in example.This recipe has really only given a very small taste of what’s possible with the unittest.mock module. The official documentation is a must-read for more advancedfeatures.

# 14.3 在单元测试中测试异常情况

## 问题

You want to write a unit test that cleanly tests if an exception is raised.

## 解决方案

To test for exceptions, use the assertRaises() method. For example, if you want to testthat a function raised a ValueError exception, use this code:

import unittest

# A simple function to illustratedef parse_int(s):

> return int(s)

class TestConversion(unittest.TestCase):def test_bad_int(self):self.assertRaises(ValueError, parse_int, ‘N/A')
If you need to test the exception’s value in some way, then a different approach is needed.For example:

import errno

class TestIO(unittest.TestCase):def test_file_not_found(self):try:f = open(‘/file/not/found')except IOError as e:self.assertEqual(e.errno, errno.ENOENT)else:self.fail(‘IOError not raised')

## 讨论

The assertRaises() method provides a convenient way to test for the presence of anexception. A common pitfall is to write tests that manually try to do things with excep‐tions on their own. For instance:

class TestConversion(unittest.TestCase):def test_bad_int(self):try:r = parse_int(‘N/A')except ValueError as e:self.assertEqual(type(e), ValueError)
The problem with such approaches is that it is easy to forget about corner cases, suchas that when no exception is raised at all. To do that, you need to add an extra check forthat situation, as shown here:

class TestConversion(unittest.TestCase):def test_bad_int(self):try:r = parse_int(‘N/A')except ValueError as e:self.assertEqual(type(e), ValueError)else:self.fail(‘ValueError not raised')
The assertRaises() method simply takes care of these details, so you should prefer touse it.The one limitation of assertRaises() is that it doesn’t provide a means for testing thevalue of the exception object that’s created. To do that, you have to manually test it, asshown. Somewhere in between these two extremes, you might consider using the assertRaisesRegex() method, which allows you to test for an exception and perform aregular expression match against the exception’s string representation at the same time.For example:

class TestConversion(unittest.TestCase):def test_bad_int(self):self.assertRaisesRegex(ValueError, ‘invalid literal .*',parse_int, ‘N/A')
A little-known fact about assertRaises() and assertRaisesRegex() is that they canalso be used as context managers:

class TestConversion(unittest.TestCase):def test_bad_int(self):with self.assertRaisesRegex(ValueError, ‘invalid literal .*'):r = parse_int(‘N/A')
This form can be useful if your test involves multiple steps (e.g., setup) besides that ofsimply executing a callable.

# 14.4 将测试输出用日志记录到文件中

## 问题

You want the results of running unit tests written to a file instead of printed to standardoutput.

## 解决方案

A very common technique for running unit tests is to include a small code fragmentlike this at the bottom of your testing file:

import unittest

class MyTest(unittest.TestCase):...if **name** == ‘**main**':unittest.main()
This makes the test file executable, and prints the results of running tests to standardoutput. If you would like to redirect this output, you need to unwind the main() call abit and write your own main() function like this:

import sysdef main(out=sys.stderr, verbosity=2):

> loader = unittest.TestLoader()suite = loader.loadTestsFromModule(sys.modules[**name**])unittest.TextTestRunner(out,verbosity=verbosity).run(suite)

if **name** == ‘**main**':with open(‘testing.out', ‘w') as f:main(f)

## 讨论

The interesting thing about this recipe is not so much the task of getting test resultsredirected to a file, but the fact that doing so exposes some notable inner workings ofthe unittest module.At a basic level, the unittest module works by first assembling a test suite. This testsuite consists of the different testing methods you defined. Once the suite has beenassembled, the tests it contains are executed.

These two parts of unit testing are separate from each other. The unittest.TestLoader instance created in the solution is used to assemble a test suite. The loadTestsFromModule() is one of several methods it defines to gather tests. In this case, it scans amodule for TestCase classes and extracts test methods from them. If you want some‐thing more fine-grained, the loadTestsFromTestCase() method (not shown) can beused to pull test methods from an individual class that inherits from TestCase.The TextTestRunner class is an example of a test runner class. The main purpose ofthis class is to execute the tests contained in a test suite. This class is the same test runnerthat sits behind the unittest.main() function. However, here we’re giving it a bit oflow-level configuration, including an output file and an elevated verbosity level.Although this recipe only consists of a few lines of code, it gives a hint as to how youmight further customize the unittest framework. To customize how test suites areassembled, you would perform various operations using the TestLoader class. To cus‐tomize how tests execute, you could make custom test runner classes that emulate thefunctionality of TextTestRunner. Both topics are beyond the scope of what can be cov‐ered here. However, documentation for the unittest module has extensive coverageof the underlying protocols.

# 14.5 忽略或者期望测试失败

## 问题

You want to skip or mark selected tests as an anticipated failure in your unit tests.

## 解决方案

The unittest module has decorators that can be applied to selected test methods tocontrol their handling. For example:

import unittestimport osimport platform

class Tests(unittest.TestCase):def test_0(self):self.assertTrue(True)
@unittest.skip(‘skipped test')def test_1(self):

> self.fail(‘should have failed!')

@unittest.skipIf(os.name=='posix', ‘Not supported on Unix')def test_2(self):

> import winreg

@unittest.skipUnless(platform.system() == ‘Darwin', ‘Mac specific test')def test_3(self):

> self.assertTrue(True)

@unittest.expectedFailuredef test_4(self):

> self.assertEqual(2+2, 5)

if **name** == ‘**main**':unittest.main()
If you run this code on a Mac, you’ll get this output:

> > bash % python3 testsample.py -vtest_0 (**main**.Tests) ... oktest_1 (**main**.Tests) ... skipped ‘skipped test'test_2 (**main**.Tests) ... skipped ‘Not supported on Unix'test_3 (**main**.Tests) ... oktest_4 (**main**.Tests) ... expected failure
> 
> Ran 5 tests in 0.002s
> 
> OK (skipped=2, expected failures=1)

## 讨论

The skip() decorator can be used to skip over a test that you don’t want to run at all.skipIf() and skipUnless() can be a useful way to write tests that only apply to certainplatforms or Python versions, or which have other dependencies. Use the @expectedFailure decorator to mark tests that are known failures, but for which you don’t wantthe test framework to report more information.The decorators for skipping methods can also be applied to entire testing classes. Forexample:

@unittest.skipUnless(platform.system() == ‘Darwin', ‘Mac specific tests')class DarwinTests(unittest.TestCase):

> ...

# 14.6 处理多个异常

## 问题

You have a piece of code that can throw any of several different exceptions, and youneed to account for all of the potential exceptions that could be raised without creatingduplicate code or long, meandering code passages.

## 解决方案

If you can handle different exceptions all using a single block of code, they can begrouped together in a tuple like this:

try:client_obj.get_url(url)except (URLError, ValueError, SocketTimeout):client_obj.remove_url(url)
In the preceding example, the remove_url() method will be called if any one of thelisted exceptions occurs. If, on the other hand, you need to handle one of the exceptionsdifferently, put it into its own except clause:

try:client_obj.get_url(url)except (URLError, ValueError):client_obj.remove_url(url)except SocketTimeout:client_obj.handle_url_timeout(url)
Many exceptions are grouped into an inheritance hierarchy. For such exceptions, youcan catch all of them by simply specifying a base class. For example, instead of writingcode like this:

try:f = open(filename)except (FileNotFoundError, PermissionError):...
you could rewrite the except statement as:

try:f = open(filename)except OSError:...
This works because OSError is a base class that’s common to both the FileNotFoundErrorand PermissionError exceptions.

## 讨论

Although it’s not specific to handling multiple exceptions per se, it’s worth noting thatyou can get a handle to the thrown exception using the as keyword:

try:f = open(filename)except OSError as e:if e.errno == errno.ENOENT:logger.error(‘File not found')elif e.errno == errno.EACCES:logger.error(‘Permission denied')else:logger.error(‘Unexpected error: %d', e.errno)
In this example, the e variable holds an instance of the raised OSError. This is useful ifyou need to inspect the exception further, such as processing it based on the value of anadditional status code.Be aware that except clauses are checked in the order listed and that the first matchexecutes. It may be a bit pathological, but you can easily create situations where multipleexcept clauses might match. For example:

```py
>>> f = open('missing')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
FileNotFoundError: [Errno 2] No such file or directory: 'missing'
>>> try:
...     f = open('missing')
... except OSError:
...     print('It failed')
... except FileNotFoundError:
...     print('File not found')
...
It failed
>>>

```

Here the except FileNotFoundError clause doesn’t execute because the OSError ismore general, matches the FileNotFoundError exception, and was listed first.As a debugging tip, if you’re not entirely sure about the class hierarchy of a particularexception, you can quickly view it by inspecting the exception’s **mro** attribute. Forexample:

```py
>>> FileNotFoundError.__mro__
(<class 'FileNotFoundError'>, <class 'OSError'>, <class 'Exception'>,
 <class 'BaseException'>, <class 'object'>)
>>>

```

Any one of the listed classes up to BaseException can be used with the except statement.

# 14.7 捕获所有异常

## 问题

You want to write code that catches all exceptions.

## 解决方案

To catch all exceptions, write an exception handler for Exception, as shown here:

try:...except Exception as e:...log(‘Reason:', e) # Important!
This will catch all exceptions save SystemExit, KeyboardInterrupt, and GeneratorExit. If you also want to catch those exceptions, change Exception to BaseException.

## 讨论

Catching all exceptions is sometimes used as a crutch by programmers who can’t re‐member all of the possible exceptions that might occur in complicated operations. Assuch, it is also a very good way to write undebuggable code if you are not careful.Because of this, if you choose to catch all exceptions, it is absolutely critical to log orreport the actual reason for the exception somewhere (e.g., log file, error message print‐ed to screen, etc.). If you don’t do this, your head will likely explode at some point.Consider this example:

def parse_int(s):try:n = int(v)except Exception:print(“Couldn't parse”)
If you try this function, it behaves like this:

```py
>>> parse_int('n/a')
Couldn't parse
>>> parse_int('42')
Couldn't parse
>>>

```

At this point, you might be left scratching your head as to why it doesn’t work. Nowsuppose the function had been written like this:

def parse_int(s):try:n = int(v)except Exception as e:print(“Couldn't parse”)print(‘Reason:', e)
In this case, you get the following output, which indicates that a programming mistakehas been made:

```py
>>> parse_int('42')
Couldn't parse
Reason: global name 'v' is not defined
>>>

```

All things being equal, it’s probably better to be as precise as possible in your exceptionhandling. However, if you must catch all exceptions, just make sure you give good di‐agnostic information or propagate the exception so that cause doesn’t get lost.

# 14.8 创建自定义异常

## 问题

You’re building an application and would like to wrap lower-level exceptions with cus‐tom ones that have more meaning in the context of your application.

## 解决方案

Creating new exceptions is easy—just define them as classes that inherit from Exception (or one of the other existing exception types if it makes more sense). For example,if you are writing code related to network programming, you might define some customexceptions like this:

class NetworkError(Exception):passclass HostnameError(NetworkError):passclass TimeoutError(NetworkError):passclass ProtocolError(NetworkError):pass
Users could then use these exceptions in the normal way. For example:

try:msg = s.recv()except TimeoutError as e:...except ProtocolError as e:...

## 讨论

Custom exception classes should almost always inherit from the built-in Exceptionclass, or inherit from some locally defined base exception that itself inherits from Exception. Although all exceptions also derive from BaseException, you should not usethis as a base class for new exceptions. BaseException is reserved for system-exitingexceptions, such as KeyboardInterrupt or SystemExit, and other exceptions thatshould signal the application to exit. Therefore, catching these exceptions is not the

intended use case. Assuming you follow this convention, it follows that inheriting fromBaseException causes your custom exceptions to not be caught and to signal an im‐minent application shutdown!Having custom exceptions in your application and using them as shown makes yourapplication code tell a more coherent story to whoever may need to read the code. Onedesign consideration involves the grouping of custom exceptions via inheritance. Incomplicated applications, it may make sense to introduce further base classes that groupdifferent classes of exceptions together. This gives the user a choice of catching a nar‐rowly specified error, such as this:

try:s.send(msg)except ProtocolError:...
It also gives the ability to catch a broad range of errors, such as the following:

try:s.send(msg)except NetworkError:...
If you are going to define a new exception that overrides the **init**() method ofException, make sure you always call Exception.**init**() with all of the passedarguments. For example:

class CustomError(Exception):def **init**(self, message, status):super().**init**(message, status)self.message = messageself.status = status
This might look a little weird, but the default behavior of Exception is to accept allarguments passed and to store them in the .args attribute as a tuple. Various otherlibraries and parts of Python expect all exceptions to have the .args attribute, so if youskip this step, you might find that your new exception doesn’t behave quite right incertain contexts. To illustrate the use of .args, consider this interactive session with thebuilt-in RuntimeError exception, and notice how any number of arguments can be usedwith the raise statement:

```py
>>> try:
...     raise RuntimeError('It failed')
... except RuntimeError as e:
...     print(e.args)
...
('It failed',)
>>> try:
...     raise RuntimeError('It failed', 42, 'spam')
... except RuntimeError as e:

```

... print(e.args)...(‘It failed', 42, ‘spam')>>>

For more information on creating your own exceptions, see the Python documentation.

# 14.9 捕获异常后抛出另外的异常

## 问题

You want to raise an exception in response to catching a different exception, but wantto include information about both exceptions in the traceback.

## 解决方案

To chain exceptions, use the raise from statement instead of a simple raise statement.This will give you information about both errors. For example:

```py
>>> def example():
...     try:
...             int('N/A')
...     except ValueError as e:
...             raise RuntimeError('A parsing error occurred') from e...
>>>
example()
Traceback (most recent call last):
  File "<stdin>", line 3, in example
ValueError: invalid literal for int() with base 10: 'N/A'

```

The above exception was the direct cause of the following exception:

Traceback (most recent call last):File “<stdin>”, line 1, in <module>File “<stdin>”, line 5, in example
RuntimeError: A parsing error occurred>>></stdin></module></stdin>

As you can see in the traceback, both exceptions are captured. To catch such an excep‐tion, you would use a normal except statement. However, you can look at the **cause**attribute of the exception object to follow the exception chain should you wish. Forexample:try:

> example()

except RuntimeError as e:
print(“It didn't work:”, e)

if e.**cause**:print(‘Cause:', e.**cause**)
An implicit form of chained exceptions occurs when another exception gets raised in‐side an except block. For example:

```py
>>> def example2():
...     try:
...             int('N/A')
...     except ValueError as e:
...             print("Couldn't parse:", err)
...
>>>
>>> example2()
Traceback (most recent call last):
  File "<stdin>", line 3, in example2
ValueError: invalid literal for int() with base 10: 'N/A'

```

During handling of the above exception, another exception occurred:

Traceback (most recent call last):File “<stdin>”, line 1, in <module>File “<stdin>”, line 5, in example2
NameError: global name ‘err' is not defined>>></stdin></module></stdin>

In this example, you get information about both exceptions, but the interpretation is abit different. In this case, the NameError exception is raised as the result of a program‐ming error, not in direct response to the parsing error. For this case, the **cause**attribute of an exception is not set. Instead, a **context** attribute is set to the priorexception.If, for some reason, you want to suppress chaining, use raise from None:

```py
>>> def example3():
...     try:
...             int('N/A')
...     except ValueError:
...             raise RuntimeError('A parsing error occurred') from None...
>>>
example3()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in example3
RuntimeError: A parsing error occurred
>>>

```

## 讨论

In designing code, you should give careful attention to use of the raise statement insideof other except blocks. In most cases, such raise statements should probably bechanged to raise from statements. That is, you should prefer this style:

try:...except SomeException as e:raise DifferentException() from e
The reason for doing this is that you are explicitly chaining the causes together. That is,the DifferentException is being raised in direct response to getting a SomeException. This relationship will be explicitly stated in the resulting traceback.If you write your code in the following style, you still get a chained exception, but it’soften not clear if the exception chain was intentional or the result of an unforeseenprogramming error:

try:...except SomeException:raise DifferentException()
When you use raise from, you’re making it clear that you meant to raise the secondexception.Resist the urge to suppress exception information, as shown in the last example. Al‐though suppressing exception information can lead to smaller tracebacks, it also dis‐cards information that might be useful for debugging. All things being equal, it’s oftenbest to keep as much information as possible.

# 14.10 重新抛出最后的异常

## 问题

You caught an exception in an except block, but now you want to reraise it.

## 解决方案

Simply use the raise statement all by itself. For example:

```py
>>> def example():
...     try:
...             int('N/A')
...     except ValueError:
...             print("Didn't work")
...             raise
...

>>> example()
Didn't work
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in example
ValueError: invalid literal for int() with base 10: 'N/A'
>>>

```

## 讨论

This problem typically arises when you need to take some kind of action in response toan exception (e.g., logging, cleanup, etc.), but afterward, you simply want to propagatethe exception along. A very common use might be in catch-all exception handlers:

try:...except Exception as e:

# Process exception information in some way...

# Propagate the exceptionraise

# 14.11 输出警告信息

## 问题

You want to have your program issue warning messages (e.g., about deprecated featuresor usage problems).

## 解决方案

To have your program issue a warning message, use the warnings.warn() function. Forexample:

import warnings

def func(x, y, logfile=None, debug=False):if logfile is not None:warnings.warn(‘logfile argument deprecated', DeprecationWarning)
...

The arguments to warn() are a warning message along with a warning class, which istypically one of the following: UserWarning, DeprecationWarning, SyntaxWarning,RuntimeWarning, ResourceWarning, or FutureWarning.The handling of warnings depends on how you have executed the interpreter and otherconfiguration. For example, if you run Python with the -W all option, you’ll get outputsuch as the following:

> > bash % python3 -W all example.pyexample.py:5: DeprecationWarning: logfile argument is deprecated
> > 
> > warnings.warn(‘logfile argument is deprecated', DeprecationWarning)

Normally, warnings just produce output messages on standard error. If you want to turnwarnings into exceptions, use the -W error option:

> > bash % python3 -W error example.pyTraceback (most recent call last):
> > 
> > File “example.py”, line 10, in <module>func(2, 3, logfile='log.txt')File “example.py”, line 5, in funcwarnings.warn(‘logfile argument is deprecated', DeprecationWarning)</module>
> 
> DeprecationWarning: logfile argument is deprecatedbash %

## 讨论

Issuing a warning message is often a useful technique for maintaining software andassisting users with issues that don’t necessarily rise to the level of being a full-fledgedexception. For example, if you’re going to change the behavior of a library or framework,you can start issuing warning messages for the parts that you’re going to change whilestill providing backward compatibility for a time. You can also warn users about prob‐lematic usage issues in their code.As another example of a warning in the built-in library, here is an example of a warningmessage generated by destroying a file without closing it:

```py
>>> import warnings
>>> warnings.simplefilter('always')
>>> f = open('/etc/passwd')
>>> del f
__main__:1: ResourceWarning: unclosed file <_io.TextIOWrapper name='/etc/passwd'
 mode='r' encoding='UTF-8'>
>>>

```

By default, not all warning messages appear. The -W option to Python can control theoutput of warning messages. -W all will output all warning messages, -W ignoreignores all warnings, and -W error turns warnings into exceptions. As an alternative,you can can use the warnings.simplefilter() function to control output, as justshown. An argument of always makes all warning messages appear, ignore ignores allwarnings, and error turns warnings into exceptions.For simple cases, this is all you really need to issue warning messages. The warningsmodule provides a variety of more advanced configuration options related to the fil‐tering and handling of warning messages. See the Python documentation for moreinformation.

# 14.12 调试基本的程序崩溃错误

## 问题

Your program is broken and you’d like some simple strategies for debugging it.

## 解决方案

If your program is crashing with an exception, running your program as python3 -isomeprogram.py can be a useful tool for simply looking around. The -i option startsan interactive shell as soon as a program terminates. From there, you can explore theenvironment. For example, suppose you have this code:

# sample.py

def func(n):return n + 10
func(‘Hello')

Running python3 -i produces the following:

bash % python3 -i sample.pyTraceback (most recent call last):

> File “sample.py”, line 6, in <module>func(‘Hello')File “sample.py”, line 4, in funcreturn n + 10</module>

TypeError: Can't convert ‘int' object to str implicitly>>> func(10)20>>>

If you don’t see anything obvious, a further step is to launch the Python debugger aftera crash. For example:

```py
>>> import pdb
>>> pdb.pm()
> sample.py(4)func()
-> return n + 10
(Pdb) w
  sample.py(6)<module>()
-> func('Hello')
> sample.py(4)func()
-> return n + 10
(Pdb) print n
'Hello'
(Pdb) q
>>>

```

If your code is deeply buried in an environment where it is difficult to obtain an inter‐active shell (e.g., in a server), you can often catch errors and produce tracebacks yourself.For example:

import tracebackimport sys

try:func(arg)except:print(‘** **AN ERROR OCCURRED ****‘)traceback.print_exc(file=sys.stderr)
If your program isn’t crashing, but it’s producing wrong answers or you’re mystified byhow it works, there is often nothing wrong with just injecting a few print() calls inplaces of interest. However, if you’re going to do that, there are a few related techniquesof interest. First, the traceback.print_stack() function will create a stack track ofyour program immediately at that point. For example:

```py
>>> def sample(n):
...     if n > 0:
...             sample(n-1)
...     else:
...             traceback.print_stack(file=sys.stderr)
...
>>> sample(5)
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in sample
  File "<stdin>", line 3, in sample
  File "<stdin>", line 3, in sample
  File "<stdin>", line 3, in sample
  File "<stdin>", line 3, in sample
  File "<stdin>", line 5, in sample
>>>

```

Alternatively, you can also manually launch the debugger at any point in your programusing pdb.set_trace() like this:

import pdb

def func(arg):...pdb.set_trace()...
This can be a useful technique for poking around in the internals of a large programand answering questions about the control flow or arguments to functions. For instance,once the debugger starts, you can inspect variables using print or type a command suchas w to get the stack traceback.

## 讨论

Don’t make debugging more complicated than it needs to be. Simple errors can oftenbe resolved by merely knowing how to read program tracebacks (e.g., the actual erroris usually the last line of the traceback). Inserting a few selected print() functions inyour code can also work well if you’re in the process of developing it and you simplywant some diagnostics (just remember to remove the statements later).A common use of the debugger is to inspect variables inside a function that has crashed.Knowing how to enter the debugger after such a crash has occurred is a useful skill toknow.Inserting statements such as pdb.set_trace() can be useful if you’re trying to unravelan extremely complicated program where the underlying control flow isn’t obvious.Essentially, the program will run until it hits the set_trace() call, at which point it willimmediately enter the debugger. From there, you can try to make more sense of it.If you’re using an IDE for Python development, the IDE will typically provide its owndebugging interface on top of or in place of pdb. Consult the manual for your IDE formore information.

# 14.13 给你的程序做基准测试

## 问题

You would like to find out where your program spends its time and make timingmeasurements.

## 解决方案

If you simply want to time your whole program, it’s usually easy enough to use somethinglike the Unix time command. For example:

bash % time python3 someprogram.pyreal 0m13.937suser 0m12.162ssys 0m0.098sbash %

On the other extreme, if you want a detailed report showing what your program is doing,you can use the cProfile module:

bash % python3 -m cProfile someprogram.py> 859647 function calls in 16.016 CPU seconds

Ordered by: standard name

ncalls tottime percall cumtime percall filename:lineno(function)263169 0.080 0.000 0.080 0.000 someprogram.py:16(frange)

> 513 0.001 0.000 0.002 0.000 someprogram.py:30(generate_mandel)

262656 0.194 0.000 15.295 0.000 someprogram.py:32(<genexpr>)1 0.036 0.036 16.077 16.077 someprogram.py:4(<module>)262144 15.021 0.000 15.021 0.000 someprogram.py:4(in_mandelbrot)> > > 1 0.000 0.000 0.000 0.000 os.py:746(urandom)1 0.000 0.000 0.000 0.000 png.py:1056(_readable)1 0.000 0.000 0.000 0.000 png.py:1073(Reader)1 0.227 0.227 0.438 0.438 png.py:163(<module>)</module></module></genexpr>

> 512 0.010 0.000 0.010 0.000 png.py:200(group)

...

bash %

More often than not, profiling your code lies somewhere in between these two extremes.For example, you may already know that your code spends most of its time in a fewselected functions. For selected profiling of functions, a short decorator can be useful.For example:

# timethis.py

import timefrom functools import wraps

def timethis(func):
@wraps(func)def wrapper(*args, **kwargs):

> start = time.perf_counter()r = func(*args, **kwargs)end = time.perf_counter()print(‘{}.{} : {}'.format(func.**module**, func.**name**, end - start))return r

return wrapper

To use this decorator, you simply place it in front of a function definition to get timingsfrom it. For example:

```py
>>> @timethis
... def countdown(n):
...     while n > 0:
...             n -= 1
...
>>> countdown(10000000)
__main__.countdown : 0.803001880645752
>>>

```

To time a block of statements, you can define a context manager. For example:

from contextlib import contextmanager

@contextmanagerdef timeblock(label):

> > start = time.perf_counter()try:
> > 
> > yield

finally:end = time.perf_counter()print(‘{} : {}'.format(label, end - start))

Here is an example of how the context manager works:

```py
>>> with timeblock('counting'):
...     n = 10000000
...     while n > 0:
...             n -= 1
...
counting : 1.5551159381866455
>>>

```

For studying the performance of small code fragments, the timeit module can be useful.For example:

```py
>>> from timeit import timeit
>>> timeit('math.sqrt(2)', 'import math')
0.1432319980012835
>>> timeit('sqrt(2)', 'from math import sqrt')
0.10836604500218527
>>>

```

timeit works by executing the statement specified in the first argument a million timesand measuring the time. The second argument is a setup string that is executed to setup the environment prior to running the test. If you need to change the number ofiterations, supply a number argument like this:

```py
>>> timeit('math.sqrt(2)', 'import math', number=10000000)
1.434852126003534
>>> timeit('sqrt(2)', 'from math import sqrt', number=10000000)
1.0270336690009572
>>>

```

## 讨论

When making performance measurements, be aware that any results you get are ap‐proximations. The time.perf_counter() function used in the solution provides thehighest-resolution timer possible on a given platform. However, it still measures wall-clock time, and can be impacted by many different factors, such as machine load.If you are interested in process time as opposed to wall-clock time, use time.process_time() instead. For example:

from functools import wrapsdef timethis(func):

> > @wraps(func)def wrapper(*args, **kwargs):
> > 
> > start = time.process_time()r = func(*args, **kwargs)end = time.process_time()print(‘{}.{} : {}'.format(func.**module**, func.**name**, end - start))return r
> 
> return wrapper

Last, but not least, if you’re going to perform detailed timing analysis, make sure to readthe documentation for the time, timeit, and other associated modules, so that you havean understanding of important platform-related differences and other pitfalls.See Recipe 13.13 for a related recipe on creating a stopwatch timer class.

# 14.14 让你的程序跑的更快

## 问题

Your program runs too slow and you’d like to speed it up without the assistance of moreextreme solutions, such as C extensions or a just-in-time (JIT) compiler.

## 解决方案

While the first rule of optimization might be to “not do it,” the second rule is almostcertainly “don’t optimize the unimportant.” To that end, if your program is running slow,you might start by profiling your code as discussed in Recipe 14.13.More often than not, you’ll find that your program spends its time in a few hotspots,such as inner data processing loops. Once you’ve identified those locations, you can usethe no-nonsense techniques presented in the following sections to make your programrun faster.

Use functionsA lot of programmers start using Python as a language for writing simple scripts. Whenwriting scripts, it is easy to fall into a practice of simply writing code with very littlestructure. For example:

# somescript.py

import sysimport csv

with open(sys.argv[1]) as f:
for row in csv.reader(f):

> # Some kind of processing...

A little-known fact is that code defined in the global scope like this runs slower thancode defined in a function. The speed difference has to do with the implementation oflocal versus global variables (operations involving locals are faster). So, if you want tomake the program run faster, simply put the scripting statements in a function:

# somescript.pyimport sysimport csv

def main(filename):with open(filename) as f:for row in csv.reader(f):# Some kind of processing...
main(sys.argv[1])

The speed difference depends heavily on the processing being performed, but in ourexperience, speedups of 15-30% are not uncommon.

Selectively eliminate attribute accessEvery use of the dot (.) operator to access attributes comes with a cost. Under the covers,this triggers special methods, such as **getattribute**() and **getattr**(), whichoften lead to dictionary lookups.You can often avoid attribute lookups by using the from module import name form ofimport as well as making selected use of bound methods. To illustrate, consider thefollowing code fragment:

import math

def compute_roots(nums):
result = []for n in nums:

> result.append(math.sqrt(n))

return result

# Testnums = range(1000000)for n in range(100):

> r = compute_roots(nums)

When tested on our machine, this program runs in about 40 seconds. Now change thecompute_roots() function as follows:

from math import sqrt

def compute_roots(nums):

> > result = []result_append = result.appendfor n in nums:
> > 
> > result_append(sqrt(n))
> 
> return result

This version runs in about 29 seconds. The only difference between the two versions ofcode is the elimination of attribute access. Instead of using math.sqrt(), the code usessqrt(). The result.append() method is additionally placed into a local variable result_append and reused in the inner loop.However, it must be emphasized that these changes only make sense in frequently ex‐ecuted code, such as loops. So, this optimization really only makes sense in carefullyselected places.

Understand locality of variablesAs previously noted, local variables are faster than global variables. For frequently ac‐cessed names, speedups can be obtained by making those names as local as possible.For example, consider this modified version of the compute_roots() function justdiscussed:

import math

def compute_roots(nums):
sqrt = math.sqrtresult = []result_append = result.appendfor n in nums:

> result_append(sqrt(n))

return result

In this version, sqrt has been lifted from the math module and placed into a localvariable. If you run this code, it now runs in about 25 seconds (an improvement overthe previous version, which took 29 seconds). That additional speedup is due to a locallookup of sqrt being a bit faster than a global lookup of sqrt.Locality arguments also apply when working in classes. In general, looking up a valuesuch as self.name will be considerably slower than accessing a local variable. In innerloops, it might pay to lift commonly accessed attributes into a local variable. For example:

# Slowerclass SomeClass:

> > ...def method(self):
> > 
> > for x in s:op(self.value)

# Fasterclass SomeClass:

> > ...def method(self):
> > 
> > > > value = self.valuefor x in s:
> > > 
> > > op(value)

Avoid gratuitous abstractionAny time you wrap up code with extra layers of processing, such as decorators, prop‐erties, or descriptors, you’re going to make it slower. As an example, consider this class:

class A:def **init**(self, x, y):self.x = xself.y = y
@propertydef y(self):

> return self._y

@y.setterdef y(self, value):

> self._y = value

Now, try a simple timing test:

```py
>>> from timeit import timeit
>>> a = A(1,2)
>>> timeit('a.x', 'from __main__ import a')
0.07817923510447145
>>> timeit('a.y', 'from __main__ import a')
0.35766440676525235
>>>

```

As you can observe, accessing the property y is not just slightly slower than a simpleattribute x, it’s about 4.5 times slower. If this difference matters, you should ask yourselfif the definition of y as a property was really necessary. If not, simply get rid of it andgo back to using a simple attribute instead. Just because it might be common for pro‐grams in another programming language to use getter/setter functions, that doesn’tmean you should adopt that programming style for Python.

Use the built-in containersBuilt-in data types such as strings, tuples, lists, sets, and dicts are all implemented in C,and are rather fast. If you’re inclined to make your own data structures as a replacement(e.g., linked lists, balanced trees, etc.), it may be rather difficult if not impossible to matchthe speed of the built-ins. Thus, you’re often better off just using them.

Avoid making unnecessary data structures or copiesSometimes programmers get carried away with making unnecessary data structureswhen they just don’t have to. For example, someone might write code like this:

values = [x for x in sequence]squares = [x*x for x in values]

Perhaps the thinking here is to first collect a bunch of values into a list and then to startapplying operations such as list comprehensions to it. However, the first list is com‐pletely unnecessary. Simply write the code like this:

squares = [x*x for x in sequence]

Related to this, be on the lookout for code written by programmers who are overlyparanoid about Python’s sharing of values. Overuse of functions such as copy.deepcopy() may be a sign of code that’s been written by someone who doesn’t fully under‐stand or trust Python’s memory model. In such code, it may be safe to eliminate manyof the copies.

## 讨论

Before optimizing, it’s usually worthwhile to study the algorithms that you’re using first.You’ll get a much bigger speedup by switching to an O(n log n) algorithm than bytrying to tweak the implementation of an an O(n**2) algorithm.If you’ve decided that you still must optimize, it pays to consider the big picture. As ageneral rule, you don’t want to apply optimizations to every part of your program,because such changes are going to make the code hard to read and understand. Instead,focus only on known performance bottlenecks, such as inner loops.You need to be especially wary interpreting the results of micro-optimizations. Forexample, consider these two techniques for creating a dictionary:

a = {‘name' : ‘AAPL',‘shares' : 100,‘price' : 534.22
}

b = dict(name='AAPL', shares=100, price=534.22)

The latter choice has the benefit of less typing (you don’t need to quote the key names).However, if you put the two code fragments in a head-to-head performance battle, you’llfind that using dict() runs three times slower! With this knowledge, you might beinclined to scan your code and replace every use of dict() with its more verbose al‐ternative. However, a smart programmer will only focus on parts of a program whereit might actually matter, such as an inner loop. In other places, the speed difference justisn’t going to matter at all.If, on the other hand, your performance needs go far beyond the simple techniques inthis recipe, you might investigate the use of tools based on just-in-time (JIT) compilationtechniques. For example, the PyPy project is an alternate implementation of the Python

interpreter that analyzes the execution of your program and generates native machinecode for frequently executed parts. It can sometimes make Python programs run anorder of magnitude faster, often approaching (or even exceeding) the speed of codewritten in C. Unfortunately, as of this writing, PyPy does not yet fully support Python3\. So, that is something to look for in the future. You might also consider the Numbaproject. Numba is a dynamic compiler where you annotate selected Python functionsthat you want to optimize with a decorator. Those functions are then compiled intonative machine code through the use of LLVM. It too can produce signficant perfor‐mance gains. However, like PyPy, support for Python 3 should be viewed as somewhatexperimental.Last, but not least, the words of John Ousterhout come to mind: “The best performanceimprovement is the transition from the nonworking to the working state.” Don’t worryabout optimization until you need to. Making sure your program works correctly isusually more important than making it run fast (at least initially).