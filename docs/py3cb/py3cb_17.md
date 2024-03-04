# 第十五章：C 语言扩展

This chapter looks at the problem of accessing C code from Python. Many of Python’sbuilt-in libraries are written in C, and accessing C is an important part of making Pythontalk to existing libraries. It’s also an area that might require the most study if you’re facedwith the problem of porting extension code from Python 2 to 3.

Although Python provides an extensive C programming API, there are actually manydifferent approaches for dealing with C. Rather than trying to give an exhaustive referencefor every possible tool or technique, the approach is to focus on a small fragmentof C code along with some representative examples of how to work with the code. Thegoal is to provide a series of programming templates that experienced programmerscan expand upon for their own use.

Here is the C code we will work with in most of the recipes:

```py
      /* sample.c */_method
#include <math.h>

/* Compute the greatest common divisor */
int gcd(int x, int y) {
    int g = y;
    while (x > 0) {
        g = x;
        x = y % x;
        y = g;
    }
    return g;
}

/* Test if (x0,y0) is in the Mandelbrot set or not */
int in_mandel(double x0, double y0, int n) {
    double x=0,y=0,xtemp;
    while (n > 0) {
        xtemp = x*x - y*y + x0;
        y = 2*x*y + y0;
        x = xtemp;
        n -= 1;
        if (x*x + y*y > 4) return 0;
    }
    return 1;
}

/* Divide two numbers */
int divide(int a, int b, int *remainder) {
    int quot = a / b;
    *remainder = a % b;
    return quot;
}

/* Average values in an array */
double avg(double *a, int n) {
    int i;
    double total = 0.0;
    for (i = 0; i < n; i++) {
        total += a[i];
    }
    return total / n;
}

/* A C data structure */
typedef struct Point {
    double x,y;
} Point;

/* Function involving a C data structure */
double distance(Point *p1, Point *p2) {
    return hypot(p1->x - p2->x, p1->y - p2->y);
}

```

This code contains a number of different C programming features. First, there are a fewsimple functions such as gcd() and is_mandel(). The divide() function is an exampleof a C function returning multiple values, one through a pointer argument. The avg()function performs a data reduction across a C array. The Point and distance() functioninvolve C structures.

For all of the recipes that follow, assume that the preceding code is found in a file namedsample.c, that definitions are found in a file named sample.h and that it has been compiledinto a library libsample that can be linked to other C code. The exact details ofcompilation and linking vary from system to system, but that is not the primary focus.It is assumed that if you’re working with C code, you’ve already figured that out.

# 15.1 使用 ctypes 访问 C 代码

## 问题

You have a small number of C functions that have been compiled into a shared libraryor DLL. You would like to call these functions purely from Python without having towrite additional C code or using a third-party extension tool.

## 解决方案

For small problems involving C code, it is often easy enough to use the ctypes modulethat is part of Python’s standard library. In order to use ctypes, you must first makesure the C code you want to access has been compiled into a shared library that iscompatible with the Python interpreter (e.g., same architecture, word size, compiler,etc.). For the purposes of this recipe, assume that a shared library, libsample.so, hasbeen created and that it contains nothing more than the code shown in the chapterintroduction. Further assume that the libsample.so file has been placed in the samedirectory as the sample.py file shown next.To access the resulting library, you make a Python module that wraps around it, suchas the following:# sample.pyimport ctypesimport os

# Try to locate the .so file in the same directory as this file_file = ‘libsample.so'_path = os.path.join(*(os.path.split(**file**)[:-1] + (_file,)))_mod = ctypes.cdll.LoadLibrary(_path)

# int gcd(int, int)gcd = _mod.gcdgcd.argtypes = (ctypes.c_int, ctypes.c_int)gcd.restype = ctypes.c_int

# int in_mandel(double, double, int)in_mandel = _mod.in_mandelin_mandel.argtypes = (ctypes.c_double, ctypes.c_double, ctypes.c_int)in_mandel.restype = ctypes.c_int

# int divide(int, int, int *)_divide = _mod.divide_divide.argtypes = (ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_int))_divide.restype = ctypes.c_int

def divide(x, y):
rem = ctypes.c_int()quot = _divide(x, y, rem)

return quot,rem.value

# void avg(double *, int n)# Define a special type for the ‘double *‘ argumentclass DoubleArrayType:

> def from*param(self, param):> typename = type(param).**name**if hasattr(self, ‘from*‘ + typename):
> 
> > return getattr(self, ‘from_‘ + typename)(param)

elif isinstance(param, ctypes.Array):return paramelse:raise TypeError(“Can't convert %s” % typename)> # Cast from array.array objectsdef from_array(self, param):

> > if param.typecode != ‘d':raise TypeError(‘must be an array of doubles')> > ptr, _ = param.buffer_info()return ctypes.cast(ptr, ctypes.POINTER(ctypes.c_double))
> 
> # Cast from lists/tuplesdef from_list(self, param):
> 
> > val = ((ctypes.c_double)*len(param))(*param)return val
> 
> from_tuple = from_list
> 
> # Cast from a numpy arraydef from_ndarray(self, param):
> 
> > return param.ctypes.data_as(ctypes.POINTER(ctypes.c_double))

DoubleArray = DoubleArrayType()_avg = _mod.avg_avg.argtypes = (DoubleArray, ctypes.c_int)_avg.restype = ctypes.c_double

def avg(values):return _avg(values, len(values))

# struct Point { }class Point(ctypes.Structure):

> *fields* = [(‘x', ctypes.c_double),(‘y', ctypes.c_double)]

# double distance(Point *, Point *)distance = _mod.distancedistance.argtypes = (ctypes.POINTER(Point), ctypes.POINTER(Point))distance.restype = ctypes.c_double

If all goes well, you should be able to load the module and use the resulting C functions.For example:

```py
      >>> import sample
>>> sample.gcd(35,42)
7
>>> sample.in_mandel(0,0,500)
1
>>> sample.in_mandel(2.0,1.0,500)
0
>>> sample.divide(42,8)
(5, 2)
>>> sample.avg([1,2,3])
2.0
>>> p1 = sample.Point(1,2)
>>> p2 = sample.Point(4,5)
>>> sample.distance(p1,p2)
4.242640687119285
>>>

```

## 讨论

There are several aspects of this recipe that warrant some discussion. The first issueconcerns the overall packaging of C and Python code together. If you are using ctypesto access C code that you have compiled yourself, you will need to make sure that theshared library gets placed in a location where the sample.py module can find it. Onepossibility is to put the resulting .so file in the same directory as the supporting Pythoncode. This is what’s shown at the first part of this recipe—sample.py looks at the **file**variable to see where it has been installed, and then constructs a path that points to alibsample.so file in the same directory.If the C library is going to be installed elsewhere, then you’ll have to adjust the pathaccordingly. If the C library is installed as a standard library on your machine, you mightbe able to use the ctypes.util.find_library() function. For example:

```py
      >>> from ctypes.util import find_library
>>> find_library('m')
'/usr/lib/libm.dylib'
>>> find_library('pthread')
'/usr/lib/libpthread.dylib'
>>> find_library('sample')
'/usr/local/lib/libsample.so'
>>>

```

Again, ctypes won’t work at all if it can’t locate the library with the C code. Thus, you’llneed to spend a few minutes thinking about how you want to install things.Once you know where the C library is located, you use ctypes.cdll.LoadLibrary()to load it. The following statement in the solution does this where _path is the fullpathname to the shared library:

_mod = ctypes.cdll.LoadLibrary(_path)

Once a library has been loaded, you need to write statements that extract specific sym‐bols and put type signatures on them. This is what’s happening in code fragments suchas this:

# int in_mandel(double, double, int)in_mandel = _mod.in_mandelin_mandel.argtypes = (ctypes.c_double, ctypes.c_double, ctypes.c_int)in_mandel.restype = ctypes.c_int

In this code, the .argtypes attribute is a tuple containing the input arguments to afunction, and .restype is the return type. ctypes defines a variety of type objects (e.g.,c_double, c_int, c_short, c_float, etc.) that represent common C data types. Attach‐ing the type signatures is critical if you want to make Python pass the right kinds ofarguments and convert data correctly (if you don’t do this, not only will the code notwork, but you might cause the entire interpreter process to crash).A somewhat tricky part of using ctypes is that the original C code may use idioms thatdon’t map cleanly to Python. The divide() function is a good example because it returnsa value through one of its arguments. Although that’s a common C technique, it’s oftennot clear how it’s supposed to work in Python. For example, you can’t do anythingstraightforward like this:

```py
      >>> divide = _mod.divide
>>> divide.argtypes = (ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_int))
>>> x = 0
>>> divide(10, 3, x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ctypes.ArgumentError: argument 3: <class 'TypeError'>: expected LP_c_int
instance instead of int
>>>

```

Even if this did work, it would violate Python’s immutability of integers and probablycause the entire interpreter to be sucked into a black hole. For arguments involvingpointers, you usually have to construct a compatible ctypes object and pass it in likethis:

```py
      >>> x = ctypes.c_int()
>>> divide(10, 3, x)
3
>>> x.value
1
>>>

```

Here an instance of a ctypes.c_int is created and passed in as the pointer object. Unlikea normal Python integer, a c_int object can be mutated. The .value attribute can beused to either retrieve or change the value as desired.

For cases where the C calling convention is “un-Pythonic,” it is common to write a smallwrapper function. In the solution, this code makes the divide() function return thetwo results using a tuple instead:# int divide(int, int, int *)_divide = _mod.divide_divide.argtypes = (ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_int))_divide.restype = ctypes.c_int

def divide(x, y):rem = ctypes.c_int()quot = _divide(x,y,rem)return quot, rem.value
The avg() function presents a new kind of challenge. The underlying C code expectsto receive a pointer and a length representing an array. However, from the Python side,we must consider the following questions: What is an array? Is it a list? A tuple? Anarray from the array module? A numpy array? Is it all of these? In practice, a Python“array” could take many different forms, and maybe you would like to support multiplepossibilities.The DoubleArrayType class shows how to handle this situation. In this class, a singlemethod from_param() is defined. The role of this method is to take a single parameterand narrow it down to a compatible ctypes object (a pointer to a ctypes.c_double, inthe example). Within from_param(), you are free to do anything that you wish. In thesolution, the typename of the parameter is extracted and used to dispatch to a morespecialized method. For example, if a list is passed, the typename is list and a methodfrom_list() is invoked.For lists and tuples, the from_list() method performs a conversion to a ctypes arrayobject. This looks a little weird, but here is an interactive example of converting a list toa ctypes array:

```py
      >>> nums = [1, 2, 3]
>>> a = (ctypes.c_double * len(nums))(*nums)
>>> a
<__main__.c_double_Array_3 object at 0x10069cd40>
>>> a[0]
1.0
>>> a[1]
2.0
>>> a[2]
3.0
>>>

```

For array objects, the from_array() method extracts the underlying memory pointerand casts it to a ctypes pointer object. For example:

```py
      >>> import array
>>> a = array.array('d',[1,2,3])
>>> a
array('d', [1.0, 2.0, 3.0])
>>> ptr_ = a.buffer_info()
>>> ptr
4298687200
>>> ctypes.cast(ptr, ctypes.POINTER(ctypes.c_double))
<__main__.LP_c_double object at 0x10069cd40>
>>>

```

The from_ndarray() shows comparable conversion code for numpy arrays.By defining the DoubleArrayType class and using it in the type signature of avg(), asshown, the function can accept a variety of different array-like inputs:

```py
      >>> import sample
>>> sample.avg([1,2,3])
2.0
>>> sample.avg((1,2,3))
2.0
>>> import array
>>> sample.avg(array.array('d',[1,2,3]))
2.0
>>> import numpy
>>> sample.avg(numpy.array([1.0,2.0,3.0]))
2.0
>>>

```

The last part of this recipe shows how to work with a simple C structure. For structures,you simply define a class that contains the appropriate fields and types like this:

class Point(ctypes.Structure):*fields* = [(‘x', ctypes.c_double),(‘y', ctypes.c_double)]
Once defined, you can use the class in type signatures as well as in code that needs toinstantiate and work with the structures. For example:

```py
      >>> p1 = sample.Point(1,2)
>>> p2 = sample.Point(4,5)
>>> p1.x
1.0
>>> p1.y
2.0
>>> sample.distance(p1,p2)
4.242640687119285
>>>

```

A few final comments: ctypes is a useful library to know about if all you’re doing isaccessing a few C functions from Python. However, if you’re trying to access a largelibrary, you might want to look at alternative approaches, such as Swig (described inRecipe 15.9) or Cython (described in Recipe 15.10).

The main problem with a large library is that since ctypes isn’t entirely automatic, you’llhave to spend a fair bit of time writing out all of the type signatures, as shown in theexample. Depending on the complexity of the library, you might also have to write alarge number of small wrapper functions and supporting classes. Also, unless you fullyunderstand all of the low-level details of the C interface, including memory managementand error handling, it is often quite easy to make Python catastrophically crash with asegmentation fault, access violation, or some similar error.As an alternative to ctypes, you might also look at CFFI. CFFI provides much of thesame functionality, but uses C syntax and supports more advanced kinds of C code. Asof this writing, CFFI is still a relatively new project, but its use has been growing rapidly.There has even been some discussion of including it in the Python standard library insome future release. Thus, it’s definitely something to keep an eye on.

# 15.2 简单的 C 扩展模块

## 问题

You want to write a simple C extension module directly using Python’s extension APIand no other tools.

## 解决方案

For simple C code, it is straightforward to make a handcrafted extension module. As apreliminary step, you probably want to make sure your C code has a proper header file.For example,

/ *sample.h */

# include<math.h></math.h>

extern int gcd(int, int);extern int in_mandel(double x0, double y0, int n);extern int divide(int a, int b, int *remainder);extern double avg(double *a, int n);

typedef struct Point {double x,y;
} Point;

extern double distance(Point *p1, Point *p2);

Typically, this header would correspond to a library that has been compiled separately.With that assumption, here is a sample extension module that illustrates the basics ofwriting extension functions:

# include “Python.h”#include “sample.h”

/ *int gcd(int, int) */static PyObject *py_gcd(PyObject *self, PyObject *args) {

> > int x, y, result;

if (!PyArg_ParseTuple(args,”ii”, &x, &y)) {return NULL;> }result = gcd(x,y);return Py_BuildValue(“i”, result);

}

/ *int in_mandel(double, double, int) */static PyObject *py_in_mandel(PyObject *self, PyObject *args) {

> > double x0, y0;int n;int result;

if (!PyArg_ParseTuple(args, “ddi”, &x0, &y0, &n)) {return NULL;> }result = in_mandel(x0,y0,n);return Py_BuildValue(“i”, result);

}

/ *int divide(int, int, int *) */static PyObject *py_divide(PyObject *self, PyObject *args) {

> > int a, b, quotient, remainder;if (!PyArg_ParseTuple(args, “ii”, &a, &b)) {
> > 
> > return NULL;
> 
> }quotient = divide(a,b, &remainder);return Py_BuildValue(“(ii)”, quotient, remainder);

}

/ *Module method table */static PyMethodDef SampleMethods[] = {

> {“gcd”, py_gcd, METH_VARARGS, “Greatest common divisor”},{“in_mandel”, py_in_mandel, METH_VARARGS, “Mandelbrot test”},{“divide”, py_divide, METH_VARARGS, “Integer division”},{ NULL, NULL, 0, NULL}

};

/ *Module structure */static struct PyModuleDef samplemodule = {

> > PyModuleDef_HEAD_INIT,
> 
> “sample”, / *name of module */“A sample module”, /* Doc string (may be NULL) */-1, /* Size of per-interpreter state or -1 */SampleMethods /* Method table */

};

/ *Module initialization function */PyMODINIT_FUNCPyInit_sample(void) {

> return PyModule_Create(&samplemodule);

}

For building the extension module, create a setup.py file that looks like this:

# setup.pyfrom distutils.core import setup, Extension

setup(name='sample',ext_modules=[Extension(‘sample',[‘pysample.c'],include_dirs = [‘/some/dir'],define_macros = [(‘FOO',‘1')],undef_macros = [‘BAR'],library_dirs = [‘/usr/local/lib'],libraries = [‘sample'])
]

)

Now, to build the resulting library, simply use python3 buildlib.py build_ext –inplace. For example:

bash % python3 setup.py build_ext –inplacerunning build_extbuilding ‘sample' extensiongcc -fno-strict-aliasing -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes

> -I/usr/local/include/python3.3m -c pysample.c-o build/temp.macosx-10.6-x86_64-3.3/pysample.o

gcc -bundle -undefined dynamic_lookupbuild/temp.macosx-10.6-x86_64-3.3/pysample.o

> -L/usr/local/lib -lsample -o sample.so

bash %

As shown, this creates a shared library called sample.so. When compiled, you shouldbe able to start importing it as a module:

```py
      >>> import sample
>>> sample.gcd(35, 42)
7
>>> sample.in_mandel(0, 0, 500)
1
>>> sample.in_mandel(2.0, 1.0, 500)

```

0>>> sample.divide(42, 8)(5, 2)>>>

If you are attempting these steps on Windows, you may need to spend some time fiddlingwith your environment and the build environment to get extension modules to buildcorrectly. Binary distributions of Python are typically built using Microsoft VisualStudio. To get extensions to work, you may have to compile them using the same orcompatible tools. See the Python documentation.

## 讨论

Before attempting any kind of handwritten extension, it is absolutely critical that youconsult Python’s documentation on “Extending and Embedding the Python Interpret‐er”. Python’s C extension API is large, and repeating all of it here is simply not practical.However, the most important parts can be easily discussed.First, in extension modules, functions that you write are all typically written with acommon prototype such as this:

static PyObject *py_func(PyObject *self, PyObject *args) {...
}

PyObject is the C data type that represents any Python object. At a very high level, anextension function is a C function that receives a tuple of Python objects (in PyObject*args) and returns a new Python object as a result. The self argument to the functionis unused for simple extension functions, but comes into play should you want to definenew classes or object types in C (e.g., if the extension function were a method of a class,then self would hold the instance).The PyArg_ParseTuple() function is used to convert values from Python to a C rep‐resentation. As input, it takes a format string that indicates the required values, such as“i” for integer and “d” for double, as well as the addresses of C variables in which to placethe converted results. PyArg_ParseTuple() performs a variety of checks on the numberand type of arguments. If there is any mismatch with the format string, an exception israised and NULL is returned. By checking for this and simply returning NULL, an ap‐propriate exception will have been raised in the calling code.The Py_BuildValue() function is used to create Python objects from C data types. Italso accepts a format code to indicate the desired type. In the extension functions, it isused to return results back to Python. One feature of Py_BuildValue() is that it canbuild more complicated kinds of objects, such as tuples and dictionaries. In the codefor py_divide(), an example showing the return of a tuple is shown. However, here area few more examples:

return Py_BuildValue(“i”, 34); // Return an integerreturn Py_BuildValue(“d”, 3.4); // Return a doublereturn Py_BuildValue(“s”, “Hello”); // Null-terminated UTF-8 stringreturn Py_BuildValue(“(ii)”, 3, 4); // Tuple (3, 4)

Near the bottom of any extension module, you will find a function table such as theSampleMethods table shown in this recipe. This table lists C functions, the names to usein Python, as well as doc strings. All modules are required to specify such a table, as itgets used in the initialization of the module.The final function PyInit_sample() is the module initialization function that executeswhen the module is first imported. The primary job of this function is to register themodule object with the interpreter.As a final note, it must be stressed that there is considerably more to extending Pythonwith C functions than what is shown here (in fact, the C API contains well over 500functions in it). You should view this recipe simply as a stepping stone for getting started.To do more, start with the documentation on the PyArg_ParseTuple() and Py_BuildValue() functions, and expand from there.

# 15.3 一个操作数组的扩展函数

## 问题

You want to write a C extension function that operates on contiguous arrays of data, asmight be created by the array module or libraries like NumPy. However, you would likeyour function to be general purpose and not specific to any one array library.

## 解决方案

To receive and process arrays in a portable manner, you should write code that uses theBuffer Protocol. Here is an example of a handwritten C extension function that receivesarray data and calls the avg(double *buf, int len) function from this chapter’s in‐troduction:

/ *Call double avg(double *, int) */static PyObject *py_avg(PyObject *self, PyObject *args) {

> > PyObject _bufobj;Py*buffer view;double result;/* Get the passed Python object */if (!PyArg_ParseTuple(args, “O”, &bufobj)) {
> > 
> > return NULL;
> 
> }
> 
> / *Attempt to extract buffer information from it */

if (PyObject_GetBuffer(bufobj, &view,> > PyBUF_ANY_CONTIGUOUS | PyBUF_FORMAT) == -1) {

> return NULL;
> 
> }

if (view.ndim != 1) {PyErr_SetString(PyExc_TypeError, “Expected a 1-dimensional array”);PyBuffer_Release(&view);return NULL;> }

> / *Check the type of items in the array */if (strcmp(view.format,”d”) != 0) {
> 
> > PyErr_SetString(PyExc_TypeError, “Expected an array of doubles”);PyBuffer_Release(&view);return NULL;
> 
> }
> 
> / *Pass the raw buffer and size to the C function */result = avg(view.buf, view.shape[0]);
> 
> / *Indicate we're done working with the buffer */PyBuffer_Release(&view);return Py_BuildValue(“d”, result);

}

Here is an example that shows how this extension function works:

```py
      >>> import array
>>> avg(array.array('d',[1,2,3]))
2.0
>>> import numpy
>>> avg(numpy.array([1.0,2.0,3.0]))
2.0
>>> avg([1,2,3])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'list' does not support the buffer interface
>>> avg(b'Hello')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Expected an array of doubles
>>> a = numpy.array([[1.,2.,3.],[4.,5.,6.]])
>>> avg(a[:,2])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: ndarray is not contiguous
>>> sample.avg(a)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Expected a 1-dimensional array
>>> sample.avg(a[0])

```

### 2.0

## 讨论

Passing array objects to C functions might be one of the most common things you wouldwant to do with a extension function. A large number of Python applications, rangingfrom image processing to scientific computing, are based on high-performance arrayprocessing. By writing code that can accept and operate on arrays, you can write cus‐tomized code that plays nicely with those applications as opposed to having some sortof custom solution that only works with your own code.The key to this code is the PyBuffer_GetBuffer() function. Given an arbitrary Pythonobject, it tries to obtain information about the underlying memory representation. Ifit’s not possible, as is the case with most normal Python objects, it simply raises anexception and returns -1\. The special flags passed to PyBuffer_GetBuffer() giveadditional hints about the kind of memory buffer that is requested. For example,PyBUF_ANY_CONTIGUOUS specifies that a contiguous region of memory is required.For arrays, byte strings, and other similar objects, a Py_buffer structure is filled withinformation about the underlying memory. This includes a pointer to the memory, size,itemsize, format, and other details. Here is the definition of this structure:

typedef struct bufferinfo {void *buf; /* Pointer to buffer memory */PyObject *obj; /* Python object that is the owner _/Py_ssize*t len; /* Total size in bytes _/Py_ssize*t itemsize; /* Size in bytes of a single item */int readonly; /* Read-only access flag */int ndim; /* Number of dimensions */char *format; /* struct code of a single item _/Py_ssize*t *shape; /* Array containing dimensions _/Py_ssize*t *strides; /* Array containing strides _/Py_ssize*t *suboffsets; /* Array containing suboffsets */
} Py_buffer;

In this recipe, we are simply concerned with receiving a contiguous array of doubles.To check if items are a double, the format attribute is checked to see if the string is“d”. This is the same code that the struct module uses when encoding binary values.As a general rule, format could be any format string that’s compatible with the structmodule and might include multiple items in the case of arrays containing C structures.Once we have verified the underlying buffer information, we simply pass it to the Cfunction, which treats it as a normal C array. For all practical purposes, it is not con‐cerned with what kind of array it is or what library created it. This is how the functionis able to work with arrays created by the array module or by numpy.

Before returning a final result, the underlying buffer view must be released usingPyBuffer_Release(). This step is required to properly manage reference counts ofobjects.Again, this recipe only shows a tiny fragment of code that receives an array. If workingwith arrays, you might run into issues with multidimensional data, strided data, differentdata types, and more that will require study. Make sure you consult the official docu‐mentation to get more details.If you need to write many extensions involving array handling, you may find it easierto implement the code in Cython. See Recipe 15.11.

# 15.4 在 C 扩展模块中操作隐形指针

## 问题

You have an extension module that needs to handle a pointer to a C data structure, butyou don’t want to expose any internal details of the structure to Python.

## 解决方案

Opaque data structures are easily handled by wrapping them inside capsule objects.Consider this fragment of C code from our sample code:

typedef struct Point {double x,y;
} Point;

extern double distance(Point *p1, Point *p2);

Here is an example of extension code that wraps the Point structure and distance()function using capsules:

/ *Destructor function for points */static void del_Point(PyObject *obj) {

> free(PyCapsule_GetPointer(obj,”Point”));

}

/ *Utility functions */static Point *PyPoint_AsPoint(PyObject *obj) {

> return (Point *) PyCapsule_GetPointer(obj, “Point”);

}

static PyObject *PyPoint_FromPoint(Point *p, int must_free) {return PyCapsule_New(p, “Point”, must_free ? del_Point : NULL);
}

/ *Create a new Point object */static PyObject *py_Point(PyObject *self, PyObject *args) {

> > Point *p;double x,y;if (!PyArg_ParseTuple(args,”dd”,&x,&y)) {
> > 
> > return NULL;
> 
> }p = (Point *) malloc(sizeof(Point));p->x = x;p->y = y;return PyPoint_FromPoint(p, 1);

}

static PyObject *py_distance(PyObject *self, PyObject *args) {
Point *p1, *p2;PyObject *py_p1, *py_p2;double result;

if (!PyArg_ParseTuple(args,”OO”,&py_p1, &py_p2)) {return NULL;
}if (!(p1 = PyPoint_AsPoint(py_p1))) {

> return NULL;

}if (!(p2 = PyPoint_AsPoint(py_p2))) {

> return NULL;

}result = distance(p1,p2);return Py_BuildValue(“d”, result);

}

Using these functions from Python looks like this:

```py
      >>> import sample
>>> p1 = sample.Point(2,3)
>>> p2 = sample.Point(4,5)
>>> p1
<capsule object "Point" at 0x1004ea330>
>>> p2
<capsule object "Point" at 0x1005d1db0>
>>> sample.distance(p1,p2)
2.8284271247461903
>>>

```

## 讨论

Capsules are similar to a typed C pointer. Internally, they hold a generic pointer alongwith an identifying name and can be easily created using the PyCapsule_New() function.In addition, an optional destructor function can be attached to a capsule to release theunderlying memory when the capsule object is garbage collected.

To extract the pointer contained inside a capsule, use the PyCapsule_GetPointer()function and specify the name. If the supplied name doesn’t match that of the capsuleor some other error occurs, an exception is raised and NULL is returned.In this recipe, a pair of utility functions—PyPoint_FromPoint() and PyPoint_AsPoint()—have been written to deal with the mechanics of creating and unwindingPoint instances from capsule objects. In any extension functions, we’ll use these func‐tions instead of working with capsules directly. This design choice makes it easier todeal with possible changes to the wrapping of Point objects in the future. For example,if you decided to use something other than a capsule later, you would only have to changethese two functions.One tricky part about capsules concerns garbage collection and memory management.The PyPoint_FromPoint() function accepts a must_free argument that indicateswhether the underlying Point * structure is to be collected when the capsule is de‐stroyed. When working with certain kinds of C code, ownership issues can be difficultto handle (e.g., perhaps a Point structure is embedded within a larger data structurethat is managed separately). Rather than making a unilateral decision to garbage collect,this extra argument gives control back to the programmer. It should be noted that thedestructor associated with an existing capsule can also be changed using the PyCapsule_SetDestructor() function.Capsules are a sensible solution to interfacing with certain kinds of C code involvingstructures. For instance, sometimes you just don’t care about exposing the internals ofa structure or turning it into a full-fledged extension type. With a capsule, you can puta lightweight wrapper around it and easily pass it around to other extension functions.

# 15.5 从扩张模块中定义和导出 C 的 API

## 问题

You have a C extension module that internally defines a variety of useful functions thatyou would like to export as a public C API for use elsewhere. You would like to use thesefunctions inside other extension modules, but don’t know how to link them together,and doing it with the C compiler/linker seems excessively complicated (or impossible).

## 解决方案

This recipe focuses on the code written to handle Point objects, which were presentedin Recipe 15.4\. If you recall, that C code included some utility functions like this:

/ *Destructor function for points */static void del_Point(PyObject *obj) {

> free(PyCapsule_GetPointer(obj,”Point”));

}

/ *Utility functions */static Point *PyPoint_AsPoint(PyObject *obj) {

> return (Point *) PyCapsule_GetPointer(obj, “Point”);

}

static PyObject *PyPoint_FromPoint(Point *p, int must_free) {return PyCapsule_New(p, “Point”, must_free ? del_Point : NULL);
}

The problem now addressed is how to export the PyPoint_AsPoint() and PyPoint_FromPoint() functions as an API that other extension modules could use andlink to (e.g., if you have other extensions that also want to use the wrapped Pointobjects).To solve this problem, start by introducing a new header file for the “sample” extensioncalled pysample.h. Put the following code in it:

/ *pysample.h */#include “Python.h”#include “sample.h”#ifdef __cplusplusextern “C” {#endif

/ *Public API Table */typedef struct {

> Point *(*aspoint)(PyObject *);PyObject *(*frompoint)(Point *, int);

} _PointAPIMethods;

# ifndef PYSAMPLE_MODULE/ *Method table in external module */static _PointAPIMethods *_point_api = 0;

/ *Import the API table from sample */static int import_sample(void) {

> _point_api = (_PointAPIMethods *) PyCapsule_Import(“sample._point_api”,0);return (_point_api != NULL) ? 1 : 0;

}

/ *Macros to implement the programming interface */#define PyPoint_AsPoint(obj) (_point_api->aspoint)(obj)#define PyPoint_FromPoint(obj) (_point_api->frompoint)(obj)#endif

# ifdef __cplusplus}#endif

The most important feature here is the _PointAPIMethods table of function pointers. Itwill be initialized in the exporting module and found by importing modules.Change the original extension module to populate the table and export it as follows:

/ *pysample.c */

# include “Python.h”#define PYSAMPLE_MODULE#include “pysample.h”

.../ *Destructor function for points */static void del_Point(PyObject *obj) {

> printf(“Deleting pointn”);free(PyCapsule_GetPointer(obj,”Point”));

}

/ *Utility functions */static Point *PyPoint_AsPoint(PyObject *obj) {

> return (Point *) PyCapsule_GetPointer(obj, “Point”);

}

static PyObject *PyPoint_FromPoint(Point *p, int free) {return PyCapsule_New(p, “Point”, free ? del_Point : NULL);
}

static _PointAPIMethods _point_api = {PyPoint_AsPoint,PyPoint_FromPoint

### };

/ *Module initialization function */PyMODINIT_FUNCPyInit_sample(void) {

> > PyObject *m;PyObject *py_point_api;
> 
> m = PyModule_Create(&samplemodule);if (m == NULL)
> 
> > return NULL;
> 
> / *Add the Point C API functions */py_point_api = PyCapsule_New((void *) &_point_api, “sample._point_api”, NULL);if (py_point_api) {
> 
> > PyModule_AddObject(m, “_point_api”, py_point_api);
> 
> }return m;

}

Finally, here is an example of a new extension module that loads and uses these APIfunctions:

/ *ptexample.c */

/ *Include the header associated with the other module */#include “pysample.h”

/ *An extension function that uses the exported API */static PyObject *print_point(PyObject *self, PyObject *args) {

> > PyObject *obj;Point *p;if (!PyArg_ParseTuple(args,”O”, &obj)) {
> > 
> > return NULL;
> 
> }
> 
> / *Note: This is defined in a different module */p = PyPoint_AsPoint(obj);if (!p) {
> 
> > return NULL;
> 
> }printf(“%f %fn”, p->x, p->y);return Py_BuildValue(“”);

}

static PyMethodDef PtExampleMethods[] = {{“print_point”, print_point, METH_VARARGS, “output a point”},{ NULL, NULL, 0, NULL}
};

static struct PyModuleDef ptexamplemodule = {PyModuleDef_HEAD*INIT,“ptexample”, /* name of module* /“A module that imports an API”, / *Doc string (may be NULL)* /-1, / *Size of per-interpreter state or -1* /PtExampleMethods /_ Method table */
};

/ *Module initialization function */PyMODINIT_FUNCPyInit_ptexample(void) {

> > PyObject *m;
> 
> m = PyModule_Create(&ptexamplemodule);if (m == NULL)
> 
> > return NULL;
> 
> / *Import sample, loading its API functions */if (!import_sample()) {
> 
> > return NULL;
> 
> }
> 
> return m;

}

When compiling this new module, you don’t even need to bother to link against any ofthe libraries or code from the other module. For example, you can just make a simplesetup.py file like this:

# setup.pyfrom distutils.core import setup, Extension

setup(name='ptexample',ext_modules=[Extension(‘ptexample',[‘ptexample.c'],include_dirs = [], # May need pysample.h directory)
]

)

If it all works, you’ll find that your new extension function works perfectly with the CAPI functions defined in the other module:

```py
      >>> import sample
>>> p1 = sample.Point(2,3)
>>> p1
<capsule object "Point *" at 0x1004ea330>
>>> import ptexample
>>> ptexample.print_point(p1)
2.000000 3.000000
>>>

```

## 讨论

This recipe relies on the fact that capsule objects can hold a pointer to anything youwish. In this case, the defining module populates a structure of function pointers, createsa capsule that points to it, and saves the capsule in a module-level attribute (e.g., sample._point_api).Other modules can be programmed to pick up this attribute when imported and extractthe underlying pointer. In fact, Python provides the PyCapsule_Import() utility func‐tion, which takes care of all the steps for you. You simply give it the name of the attribute(e.g., sample._point_api), and it will find the capsule and extract the pointer all in onestep.There are some C programming tricks involved in making exported functions looknormal in other modules. In the pysample.h file, a pointer _point_api is used to pointto the method table that was initialized in the exporting module. A related functionimport_sample() is used to perform the required capsule import and initialize thispointer. This function must be called before any functions are used. Normally, it would

be called in during module initialization. Finally, a set of C preprocessor macros havebeen defined to transparently dispatch the API functions through the method table.The user just uses the original function names, but doesn’t know about the extra indi‐rection through these macros.Finally, there is another important reason why you might use this technique to linkmodules together—it’s actually easier and it keeps modules more cleanly decoupled. Ifyou didn’t want to use this recipe as shown, you might be able to cross-link modulesusing advanced features of shared libraries and the dynamic loader. For example, puttingcommon API functions into a shared library and making sure that all extension moduleslink against that shared library. Yes, this works, but it can be tremendously messy inlarge systems. Essentially, this recipe cuts out all of that magic and allows modules tolink to one another through Python’s normal import mechanism and just a tiny numberof capsule calls. For compilation of modules, you only need to worry about header files,not the hairy details of shared libraries.Further information about providing C APIs for extension modules can be found in thePython documentation.

# 15.6 从 C 语言中调用 Python 代码

## 问题

You want to safely execute a Python callable from C and return a result back to C. Forexample, perhaps you are writing C code that wants to use a Python function as acallback.

## 解决方案

Calling Python from C is mostly straightforward, but involves a number of tricky parts.The following C code shows an example of how to do it safely:

# include<python.h></python.h>

/ *Execute func(x,y) in the Python interpreter. Thearguments and return result of the function mustbe Python floats */double call_func(PyObject *func, double x, double y) {
PyObject *args;PyObject *kwargs;PyObject *result = 0;double retval;

/ *Make sure we own the GIL */PyGILState_STATE state = PyGILState_Ensure();

/ *Verify that func is a proper callable */if (!PyCallable_Check(func)) {

> fprintf(stderr,”call_func: expected a callablen”);goto fail;

}/ *Build arguments */args = Py_BuildValue(“(dd)”, x, y);kwargs = NULL;

/ *Call the function */result = PyObject_Call(func, args, kwargs);Py_DECREF(args);Py_XDECREF(kwargs);

/ *Check for Python exceptions (if any) */if (PyErr_Occurred()) {

> PyErr_Print();goto fail;

}

/ *Verify the result is a float object */if (!PyFloat_Check(result)) {

> fprintf(stderr,”call_func: callable didn't return a floatn”);goto fail;

}

/ *Create the return value */retval = PyFloat_AsDouble(result);Py_DECREF(result);

/ *Restore previous GIL state and return */PyGILState_Release(state);return retval;

fail:Py_XDECREF(result);PyGILState_Release(state);abort(); // Change to something more appropriate
}

To use this function, you need to have obtained a reference to an existing Python callableto pass in. There are many ways that you can go about doing that, such as having acallable object passed into an extension module or simply writing C code to extract asymbol from an existing module.Here is a simple example that shows calling a function from an embedded Pythoninterpreter:

# include<python.h></python.h>

/ *Definition of call_func() same as above */...

/ *Load a symbol from a module */PyObject *import_name(const char *modname, const char *symbol) {

> PyObject *u_name, *module;u_name = PyUnicode_FromString(modname);module = PyImport_Import(u_name);Py_DECREF(u_name);return PyObject_GetAttrString(module, symbol);

}

/ *Simple embedding example */int main() {

> > PyObject *pow_func;double x;
> 
> Py_Initialize();/ *Get a reference to the math.pow function */pow_func = import_name(“math”,”pow”);
> 
> / *Call it using our call_func() code */for (x = 0.0; x < 10.0; x += 0.1) {
> 
> > printf(“%0.2f %0.2fn”, x, call_func(pow_func,x,2.0));
> 
> }/ *Done */Py_DECREF(pow_func);Py_Finalize();return 0;

}

To build this last example, you’ll need to compile the C and link against the Pythoninterpreter. Here is a Makefile that shows how you might do it (this is something thatmight require some amount of fiddling with on your machine):

all::cc -g embed.c -I/usr/local/include/python3.3m -L/usr/local/lib/python3.3/config-3.3m -lpython3.3m
Compiling and running the resulting executable should produce output similar to this:

0.00 0.000.10 0.010.20 0.040.30 0.090.40 0.16...

Here is a slightly different example that shows an extension function that receives acallable and some arguments and passes them to call_func() for the purposes oftesting:

/ *Extension function for testing the C-Python callback */PyObject *py_call_func(PyObject *self, PyObject *args) {

> > PyObject *func;
> 
> double x, y, result;if (!PyArg_ParseTuple(args,”Odd”, &func,&x,&y)) {
> 
> > return NULL;
> 
> }result = call_func(func, x, y);return Py_BuildValue(“d”, result);

}

Using this extension function, you could test it as follows:

```py
      >>> import sample
>>> def add(x,y):
...     return x+y
...
>>> sample.call_func(add,3,4)
7.0
>>>

```

## 讨论

If you are calling Python from C, the most important thing to keep in mind is that C isgenerally going to be in charge. That is, C has the responsibility of creating the argu‐ments, calling the Python function, checking for exceptions, checking types, extractingreturn values, and more.As a first step, it is critical that you have a Python object representing the callable thatyou’re going to invoke. This could be a function, class, method, built-in method, oranything that implements the **call**() operation. To verify that it’s callable, usePyCallable_Check() as shown in this code fragment:

double call_func(PyObject *func, double x, double y) {
.../ *Verify that func is a proper callable */if (!PyCallable_Check(func)) {

> fprintf(stderr,”call_func: expected a callablen”);goto fail;

As an aside, handling errors in the C code is something that you will need to carefullystudy. As a general rule, you can’t just raise a Python exception. Instead, errors will haveto be handled in some other manner that makes sense to your C code. In the solution,we’re using goto to transfer control to an error handling block that calls abort(). Thiscauses the whole program to die, but in real code you would probably want to do some‐thing more graceful (e.g., return a status code). Keep in mind that C is in charge here,so there isn’t anything comparable to just raising an exception. Error handling is some‐thing you’ll have to engineer into the program somehow.Calling a function is relatively straightforward—simply use PyObject_Call(), supply‐ing it with the callable object, a tuple of arguments, and an optional dictionary of

keyword arguments. To build the argument tuple or dictionary, you can use Py_BuildValue(), as shown.

double call_func(PyObject *func, double x, double y) {
PyObject *args;PyObject *kwargs;

.../ *Build arguments */args = Py_BuildValue(“(dd)”, x, y);kwargs = NULL;

/ *Call the function */result = PyObject_Call(func, args, kwargs);Py_DECREF(args);Py_XDECREF(kwargs);...

If there are no keyword arguments, you can pass NULL, as shown. After making thefunction call, you need to make sure that you clean up the arguments using Py_DECREF() or Py_XDECREF(). The latter function safely allows the NULL pointer to bepassed (which is ignored), which is why we’re using it for cleaning up the optionalkeyword arguments.After calling the Python function, you must check for the presence of exceptions. ThePyErr_Occurred() function can be used to do this. Knowing what to do in response toan exception is tricky. Since you’re working from C, you really don’t have the exceptionmachinery that Python has. Thus, you would have to set an error status code, log theerror, or do some kind of sensible processing. In the solution, abort() is called for lackof a simpler alternative (besides, hardened C programmers will appreciate the abruptcrash):

> > .../ *Check for Python exceptions (if any) */if (PyErr_Occurred()) {
> > 
> > PyErr_Print();goto fail;

fail:PyGILState_Release(state);abort();
Extracting information from the return value of calling a Python function is typicallygoing to involve some kind of type checking and value extraction. To do this, you mayhave to use functions in the Python concrete objects layer. In the solution, the codechecks for and extracts the value of a Python float using PyFloat_Check() and PyFloat_AsDouble().

A final tricky part of calling into Python from C concerns the management of Python’sglobal interpreter lock (GIL). Whenever Python is accessed from C, you need to makesure that the GIL is properly acquired and released. Otherwise, you run the risk of havingthe interpreter corrupt data or crash. The calls to PyGILState_Ensure() and PyGILState_Release() make sure that it’s done correctly:

double call_func(PyObject *func, double x, double y) {
...double retval;

/ *Make sure we own the GIL _/PyGILState_STATE state = PyGILState*Ensure();.../* Code that uses Python C API functions */.../* Restore previous GIL state and return */PyGILState_Release(state);return retval;

fail:PyGILState_Release(state);abort();
}

Upon return, PyGILState_Ensure() always guarantees that the calling thread has ex‐clusive access to the Python interpreter. This is true even if the calling C code is runninga different thread that is unknown to the interpreter. At this point, the C code is free touse any Python C-API functions that it wants. Upon successful completion, PyGILState_Release() is used to restore the interpreter back to its original state.It is critical to note that every PyGILState_Ensure() call must be followed by a matchingPyGILState_Release() call—even in cases where errors have occurred. In the solution,the use of a goto statement might look like a horrible design, but we’re actually using itto transfer control to a common exit block that performs this required step. Think ofthe code after the fail: lable as serving the same purpose as code in a Python finally: block.If you write your C code using all of these conventions including management of theGIL, checking for exceptions, and thorough error checking, you’ll find that you canreliably call into the Python interpreter from C—even in very complicated programsthat utilize advanced programming techniques such as multithreading.

# 15.7 从 C 扩展中释放全局锁

## 问题

You have C extension code in that you want to execute concurrently with other threadsin the Python interpreter. To do this, you need to release and reacquire the global in‐terpreter lock (GIL).

## 解决方案

In C extension code, the GIL can be released and reacquired by inserting the followingmacros in the code:

# include “Python.h”...

PyObject *pyfunc(PyObject *self, PyObject *args) {...Py_BEGIN_ALLOW_THREADS// Threaded C code. Must not use Python API functions...Py_END_ALLOW_THREADS...return result;
}

## 讨论

The GIL can only safely be released if you can guarantee that no Python C API functionswill be executed in the C code. Typical examples where the GIL might be released arein computationally intensive code that performs calculations on C arrays (e.g., in ex‐tensions such as numpy) or in code where blocking I/O operations are going to be per‐formed (e.g., reading or writing on a file descriptor).While the GIL is released, other Python threads are allowed to execute in the interpreter.The Py_END_ALLOW_THREADS macro blocks execution until the calling threads reacquiresthe GIL in the interpreter.

# 15.8 C 和 Python 中的线程混用

## 问题

You have a program that involves a mix of C, Python, and threads, but some of thethreads are created from C outside the control of the Python interpreter. Moreover,certain threads utilize functions in the Python C API.

## 解决方案

If you’re going to mix C, Python, and threads together, you need to make sure youproperly initialize and manage Python’s global interpreter lock (GIL). To do this, includethe following code somewhere in your C code and make sure it’s called prior to creationof any threads:

# include<python.h></python.h>

> > ...if (!PyEval_ThreadsInitialized()) {
> > 
> > PyEval_InitThreads();

For any C code that involves Python objects or the Python C API, make sure you prop‐erly acquire and release the GIL first. This is done using PyGILState_Ensure() andPyGILState_Release(), as shown in the following:

> > .../ *Make sure we own the GIL */PyGILState_STATE state = PyGILState_Ensure();
> 
> / *Use functions in the interpreter */.../* Restore previous GIL state and return */PyGILState_Release(state);...

Every call to PyGILState_Ensure() must have a matching call to PyGILState_Release().

## 讨论

In advanced applications involving C and Python, it is not uncommon to have manythings going on at once—possibly involving a mix of a C code, Python code, C threads,and Python threads. As long as you diligently make sure the interpreter is properlyinitialized and that C code involving the interpreter has the proper GIL managementcalls, it all should work.Be aware that the PyGILState_Ensure() call does not immediately preempt or interruptthe interpreter. If other code is currently executing, this function will block until thatcode decides to release the GIL. Internally, the interpreter performs periodic threadswitching, so even if another thread is executing, the caller will eventually get to run(although it may have to wait for a while first).

# 15.9 用 WSIG 包装 C 代码

## 问题

You have existing C code that you would like to access as a C extension module. Youwould like to do this using the Swig wrapper generator.

## 解决方案

Swig operates by parsing C header files and automatically creating extension code. Touse it, you first need to have a C header file. For example, this header file for our samplecode:

/ *sample.h */

# include <math.h>extern int gcd(int, int);extern int in_mandel(double x0, double y0, int n);extern int divide(int a, int b, int *remainder);extern double avg(double *a, int n);</math.h>

typedef struct Point {double x,y;
} Point;

extern double distance(Point *p1, Point *p2);

Once you have the header files, the next step is to write a Swig “interface” file. By con‐vention, these files have a .i suffix and might look similar to the following:

// sample.i - Swig interface%module sample%{#include “sample.h”%}

/ *Customizations */%extend Point {

> > > > > / *Constructor for Point objects */Point(double x, double y) {
> > > 
> > > Point *p = (Point *) malloc(sizeof(Point));p->x = x;p->y = y;return p;
> 
> };

};

/ *Map int *remainder as an output argument */%include typemaps.i%apply int *OUTPUT { int * remainder };

/ *Map the argument pattern (double *a, int n) to arrays */%typemap(in) (double *a, int n)(Py_buffer view) {

> > view.obj = NULL;if (PyObject_GetBuffer($input, &view, PyBUF_ANY_CONTIGUOUS | PyBUF_FORMAT) == -1) {
> > 
> > SWIG_fail;
> 
> }if (strcmp(view.format,”d”) != 0) {
> 
> > PyErr_SetString(PyExc_TypeError, “Expected an array of doubles”);SWIG_fail;
> 
> }$1 = (double *) view.buf;$2 = view.len / sizeof(double);

}

%typemap(freearg) (double *a, int n) {if (view$argnum.obj) {PyBuffer_Release(&view$argnum);
}

}

/ *C declarations to be included in the extension module */

extern int gcd(int, int);extern int in_mandel(double x0, double y0, int n);extern int divide(int a, int b, int *remainder);extern double avg(double *a, int n);

typedef struct Point {double x,y;
} Point;

extern double distance(Point *p1, Point *p2);

Once you have written the interface file, Swig is invoked as a command-line tool:

bash % swig -python -py3 sample.ibash %

The output of swig is two files, sample_wrap.c and sample.py. The latter file is whatusers import. The sample_wrap.c file is C code that needs to be compiled into a sup‐porting module called _sample. This is done using the same techniques as for normalextension modules. For example, you create a setup.py file like this:

# setup.pyfrom distutils.core import setup, Extension

setup(name='sample',
py_modules=[‘sample.py'],ext_modules=[

> Extension(‘_sample',> [‘sample_wrap.c'],include_dirs = [],define_macros = [],
> 
> undef_macros = [],library_dirs = [],libraries = [‘sample'])
> 
> ]

)

To compile and test, run python3 on the setup.py file like this:

bash % python3 setup.py build_ext –inplacerunning build_extbuilding ‘_sample' extensiongcc -fno-strict-aliasing -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes-I/usr/local/include/python3.3m -c sample_wrap.c

> -o build/temp.macosx-10.6-x86_64-3.3/sample_wrap.o

sample_wrap.c: In function ‘SWIG_InitializeModule’:sample_wrap.c:3589: warning: statement with no effectgcc -bundle -undefined dynamic_lookup build/temp.macosx-10.6-x86_64-3.3/sample.o

> build/temp.macosx-10.6-x86_64-3.3/sample_wrap.o -o _sample.so -lsample

bash %

If all of this works, you’ll find that you can use the resulting C extension module in astraightforward way. For example:

```py
      >>> import sample
>>> sample.gcd(42,8)
2
>>> sample.divide(42,8)
[5, 2]
>>> p1 = sample.Point(2,3)
>>> p2 = sample.Point(4,5)
>>> sample.distance(p1,p2)
2.8284271247461903
>>> p1.x
2.0
>>> p1.y
3.0
>>> import array
>>> a = array.array('d',[1,2,3])
>>> sample.avg(a)
2.0
>>>

```

## 讨论

Swig is one of the oldest tools for building extension modules, dating back to PythonPython. Swig can automate much of the wrapper generation process.

All Swig interfaces tend to start with a short preamble like this:

%module sample%{#include “sample.h”%}

This merely declares the name of the extension module and specifies C header files thatmust be included to make everything compile (the code enclosed in %{ and %} is pasteddirectly into the output code so this is where you put all included files and other defi‐nitions needed for compilation).The bottom part of a Swig interface is a listing of C declarations that you want to beincluded in the extension. This is often just copied from the header files. In our example,we just pasted in the header file directly like this:

%module sample%{#include “sample.h”%}...extern int gcd(int, int);extern int in_mandel(double x0, double y0, int n);extern int divide(int a, int b, int *remainder);extern double avg(double *a, int n);

typedef struct Point {double x,y;
} Point;

extern double distance(Point *p1, Point *p2);

It is important to stress that these declarations are telling Swig what you want to includein the Python module. It is quite common to edit the list of declarations or to makemodifications as appropriate. For example, if you didn’t want certain declarations to beincluded, you would remove them from the declaration list.The most complicated part of using Swig is the various customizations that it can applyto the C code. This is a huge topic that can’t be covered in great detail here, but a numberof such customizations are shown in this recipe.The first customization involving the %extend directive allows methods to be attachedto existing structure and class definitions. In the example, this is used to add a con‐structor method to the Point structure. This customization makes it possible to use thestructure like this:

```py
      >>> p1 = sample.Point(2,3)
>>>

```

If omitted, then Point objects would have to be created in a much more clumsy mannerlike this:

```py
      >>> # Usage if %extend Point is omitted
>>> p1 = sample.Point()
>>> p1.x = 2.0
>>> p1.y = 3

```

The second customization involving the inclusion of the typemaps.i library and the%apply directive is instructing Swig that the argument signature int *remainder is tobe treated as an output value. This is actually a pattern matching rule. In all declarationsthat follow, any time int *remainder is encountered, it is handled as output. Thiscustomization is what makes the divide() function return two values:

```py
      >>> sample.divide(42,8)
[5, 2]
>>>

```

The last customization involving the %typemap directive is probably the most advancedfeature shown here. A typemap is a rule that gets applied to specific argument patternsin the input. In this recipe, a typemap has been written to match the argument pattern(double *a, int n). Inside the typemap is a fragment of C code that tells Swig howto convert a Python object into the associated C arguments. The code in this recipe hasbeen written using Python’s buffer protocol in an attempt to match any input argumentthat looks like an array of doubles (e.g., NumPy arrays, arrays created by the arraymodule, etc.). See Recipe 15.3.Within the typemap code, substitutions such as $1 and $2 refer to variables that holdthe converted values of the C arguments in the typemap pattern (e.g., $1 maps to double*a and $2 maps to int n). $input refers to a PyObject * argument that was suppliedas an input argument. $argnum is the argument number.Writing and understanding typemaps is often the bane of programmers using Swig. Notonly is the code rather cryptic, but you need to understand the intricate details of boththe Python C API and the way in which Swig interacts with it. The Swig documentationhas many more examples and detailed information.Nevertheless, if you have a lot of a C code to expose as an extension module, Swig canbe a very powerful tool for doing it. The key thing to keep in mind is that Swig is basicallya compiler that processes C declarations, but with a powerful pattern matching andcustomization component that lets you change the way in which specific declarationsand types get processed. More information can be found at Swig’s website, includingPython-specific documentation.

# 15.10 用 Cython 包装 C 代码

## 问题

You want to use Cython to make a Python extension module that wraps around anexisting C library.

## 解决方案

Making an extension module with Cython looks somewhat similar to writing a hand‐written extension, in that you will be creating a collection of wrapper functions. How‐ever, unlike previous recipes, you won’t be doing this in C—the code will look a lot morelike Python.As preliminaries, assume that the sample code shown in the introduction to this chapterhas been compiled into a C library called libsample. Start by creating a file namedcsample.pxd that looks like this:

# csample.pxd## Declarations of “external” C functions and structures

cdef extern from “sample.h”:
int gcd(int, int)bint in_mandel(double, double, int)int divide(int, int, int *)double avg(double *, int) nogil

ctypedef struct Point:double xdouble y
double distance(Point *, Point *)

This file serves the same purpose in Cython as a C header file. The initial declarationcdef extern from “sample.h” declares the required C header file. Declarationsthat follow are taken from that header. The name of this file is csample.pxd, not sam‐ple.pxd—this is important.Next, create a file named sample.pyx. This file will define wrappers that bridge thePython interpreter to the underlying C code declared in the csample.pxd file:

# sample.pyx

# Import the low-level C declarationscimport csample

# Import some functionality from Python and the C stdlibfrom cpython.pycapsule cimport *

from libc.stdlib cimport malloc, free

# Wrappersdef gcd(unsigned int x, unsigned int y):

> return csample.gcd(x, y)

def in_mandel(x, y, unsigned int n):return csample.in_mandel(x, y, n)def divide(x, y):cdef int remquot = csample.divide(x, y, &rem)return quot, remdef avg(double[:] a):cdef:int szdouble result
sz = a.sizewith nogil:

> result = csample.avg(<double *> &a[0], sz)

return result

# Destructor for cleaning up Point objectscdef del_Point(object obj):

> pt = <csample.Point *> PyCapsule_GetPointer(obj,”Point”)free(<void *> pt)

# Create a Point object and return as a capsuledef Point(double x,double y):

> > cdef csample.Point *pp = <csample.Point *> malloc(sizeof(csample.Point))if p == NULL:
> > 
> > raise MemoryError(“No memory to make a Point”)
> 
> p.x = xp.y = yreturn PyCapsule_New(<void *>p,”Point”,<pycapsule_destructor>del_Point)</pycapsule_destructor>

def distance(p1, p2):pt1 = <csample.Point *> PyCapsule_GetPointer(p1,”Point”)pt2 = <csample.Point *> PyCapsule_GetPointer(p2,”Point”)return csample.distance(pt1,pt2)
Various details of this file will be covered further in the discussion section. Finally, tobuild the extension module, create a setup.py file that looks like this:

from distutils.core import setupfrom distutils.extension import Extensionfrom Cython.Distutils import build_ext

ext_modules = [
Extension(‘sample',

> [‘sample.pyx'],libraries=[‘sample'],library_dirs=[‘.'])]

setup(name = ‘Sample extension module',cmdclass = {‘build_ext': build_ext},ext_modules = ext_modules
)

To build the resulting module for experimentation, type this:

bash % python3 setup.py build_ext –inplacerunning build_extcythoning sample.pyx to sample.cbuilding ‘sample' extensiongcc -fno-strict-aliasing -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes

> -I/usr/local/include/python3.3m -c sample.c-o build/temp.macosx-10.6-x86_64-3.3/sample.o

gcc -bundle -undefined dynamic_lookup build/temp.macosx-10.6-x86_64-3.3/sample.o-L. -lsample -o sample.so
bash %

If it works, you should have an extension module sample.so that can be used as shownin the following example:

```py
      >>> import sample
>>> sample.gcd(42,10)
2
>>> sample.in_mandel(1,1,400)
False
>>> sample.in_mandel(0,0,400)
True
>>> sample.divide(42,10)
(4, 2)
>>> import array
>>> a = array.array('d',[1,2,3])
>>> sample.avg(a)
2.0
>>> p1 = sample.Point(2,3)
>>> p2 = sample.Point(4,5)
>>> p1
<capsule object "Point" at 0x1005d1e70>
>>> p2
<capsule object "Point" at 0x1005d1ea0>
>>> sample.distance(p1,p2)
2.8284271247461903
>>>

```

## 讨论

This recipe incorporates a number of advanced features discussed in prior recipes, in‐cluding manipulation of arrays, wrapping opaque pointers, and releasing the GIL. Eachof these parts will be discussed in turn, but it may help to review earlier recipes first.At a high level, using Cython is modeled after C. The .pxd files merely contain C defi‐nitions (similar to .h files) and the .pyx files contain implementation (similar to a .c file).The cimport statement is used by Cython to import definitions from a .pxd file. This isdifferent than using a normal Python import statement, which would load a regularPython module.Although .pxd files contain definitions, they are not used for the purpose of automati‐cally creating extension code. Thus, you still have to write simple wrapper functions.For example, even though the csample.pxd file declares int gcd(int, int) as a func‐tion, you still have to write a small wrapper for it in sample.pyx. For instance:

cimport csample

def gcd(unsigned int x, unsigned int y):return csample.gcd(x,y)
For simple functions, you don’t have to do too much. Cython will generate wrapper codethat properly converts the arguments and return value. The C data types attached to thearguments are optional. However, if you include them, you get additional error checkingfor free. For example, if someone calls this function with negative values, an exceptionis generated:

```py
      >>> sample.gcd(-10,2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "sample.pyx", line 7, in sample.gcd (sample.c:1284)
    def gcd(unsigned int x,unsigned int y):
OverflowError: can't convert negative value to unsigned int
>>>

```

If you want to add additional checking to the wrapper, just use additional wrapper code.For example:

def gcd(unsigned int x, unsigned int y):if x <= 0:raise ValueError(“x must be > 0”)if y <= 0:raise ValueError(“y must be > 0”)
return csample.gcd(x,y)

The declaration of in_mandel() in the csample.pxd file has an interesting, but subtledefinition. In that file, the function is declared as returning a bint instead of an int.This causes the function to create a proper Boolean value from the result instead of asimple integer. So, a return value of 0 gets mapped to False and 1 to True.

Within the Cython wrappers, you have the option of declaring C data types in additionto using all of the usual Python objects. The wrapper for divide() shows an exampleof this as well as how to handle a pointer argument.

def divide(x,y):cdef int remquot = csample.divide(x,y,&rem)return quot, rem
Here, the rem variable is explicitly declared as a C int variable. When passed to theunderlying divide() function, &rem makes a pointer to it just as in C.The code for the avg() function illustrates some more advanced features of Cython.First the declaration def avg(double[:] a) declares avg() as taking a one-dimensionalmemoryview of double values. The amazing part about this is that the resulting functionwill accept any compatible array object, including those created by libraries such asnumpy. For example:>>> import array>>> a = array.array(‘d',[1,2,3])>>> import numpy>>> b = numpy.array([1., 2., 3.])>>> import sample>>> sample.avg(a)2.0>>> sample.avg(b)2.0>>>

In the wrapper, a.size and &a[0] refer to the number of array items and underlyingpointer, respectively. The syntax <double *> &a[0] is how you type cast pointers to adifferent type if necessary. This is needed to make sure the C avg() receives a pointerof the correct type. Refer to the next recipe for some more advanced usage of Cythonmemoryviews.In addition to working with general arrays, the avg() example also shows how to workwith the global interpreter lock. The statement with nogil: declares a block of code asexecuting without the GIL. Inside this block, it is illegal to work with any kind of normalPython object—only objects and functions declared as cdef can be used. In addition tothat, external functions must explicitly declare that they can execute without the GIL.Thus, in the csample.pxd file, the avg() is declared as double avg(double *, int)nogil.The handling of the Point structure presents a special challenge. As shown, this recipetreats Point objects as opaque pointers using capsule objects, as described inRecipe 15.4\. However, to do this, the underlying Cython code is a bit more complicated.First, the following imports are being used to bring in definitions of functions from theC library and Python C API:

from cpython.pycapsule cimport *from libc.stdlib cimport malloc, free

The function del_Point() and Point() use this functionality to create a capsule objectthat wraps around a Point * pointer. The declaration cdef del_Point() declaresdel_Point() as a function that is only accessible from Cython and not Python. Thus,this function will not be visible to the outside—instead, it’s used as a callback functionto clean up memory allocated by the capsule. Calls to functions such as PyCapsule_New(), PyCapsule_GetPointer() are directly from the Python C API and are usedin the same way.The distance() function has been written to extract pointers from the capsule objectscreated by Point(). One notable thing here is that you simply don’t have to worry aboutexception handling. If a bad object is passed, PyCapsule_GetPointer() raises an ex‐ception, but Cython already knows to look for it and propagate it out of the distance() function if it occurs.A downside to the handling of Point structures is that they will be completely opaquein this implementation. You won’t be able to peek inside or access any of their attributes.There is an alternative approach to wrapping, which is to define an extension type, asshown in this code:

# sample.pyx

cimport csamplefrom libc.stdlib cimport malloc, free...

cdef class Point:
cdef csample.Point *_c_pointdef **cinit**(self, double x, double y):

> self._c_point = <csample.Point *> malloc(sizeof(csample.Point))self._c_point.x = xself._c_point.y = y

def **dealloc**(self):free(self._c_point)property x:def **get**(self):return self._c_point.xdef **set**(self, value):self._c_point.x = valueproperty y:def **get**(self):return self._c_point.ydef **set**(self, value):self._c_point.y = valuedef distance(Point p1, Point p2):return csample.distance(p1._c_point, p2._c_point)
Here, the cdef class Point is declaring Point as an extension type. The class variablecdef csample.Point *_c_point is declaring an instance variable that holds a pointerto an underlying Point structure in C. The **cinit**() and **dealloc**() methodscreate and destroy the underlying C structure using malloc() and free() calls. Theproperty x and property y declarations give code that gets and sets the underlyingstructure attributes. The wrapper for distance() has also been suitably modified toaccept instances of the Point extension type as arguments, but pass the underlyingpointer to the C function.Making this change, you will find that the code for manipulating Point objects is morenatural:

```py
      >>> import sample
>>> p1 = sample.Point(2,3)
>>> p2 = sample.Point(4,5)
>>> p1
<sample.Point object at 0x100447288>
>>> p2
<sample.Point object at 0x1004472a0>
>>> p1.x
2.0
>>> p1.y
3.0
>>> sample.distance(p1,p2)
2.8284271247461903
>>>

```

This recipe has illustrated many of Cython’s core features that you might be able toextrapolate to more complicated kinds of wrapping. However, you will definitely wantto read more of the official documentation to do more.The next few recipes also illustrate a few additional Cython features.

# 15.11 用 Cython 写高性能的数组操作

## 问题

You would like to write some high-performance array processing functions to operateon arrays from libraries such as NumPy. You’ve heard that tools such as Cython canmake this easier, but aren’t sure how to do it.

## 解决方案

As an example, consider the following code which shows a Cython function for clippingthe values in a simple one-dimensional array of doubles:

# sample.pyx (Cython)

cimport cython

@cython.boundscheck(False)@cython.wraparound(False)cpdef clip(double[:] a, double min, double max, double[:] out):

> > ‘''Clip the values in a to be between min and max. Result in out‘''if min > max:
> > 
> > raise ValueError(“min must be <= max”)

if a.shape[0] != out.shape[0]:raise ValueError(“input and output arrays must be the same size”)for i in range(a.shape[0]):if a[i] < min:out[i] = minelif a[i] > max:out[i] = maxelse:out[i] = a[i]

To compile and build the extension, you’ll need a setup.py file such as the following (usepython3 setup.py build_ext –inplace to build it):

from distutils.core import setupfrom distutils.extension import Extensionfrom Cython.Distutils import build_ext

ext_modules = [Extension(‘sample',[‘sample.pyx'])
]

setup(name = ‘Sample app',cmdclass = {‘build_ext': build_ext},ext_modules = ext_modules
)

You will find that the resulting function clips arrays, and that it works with many dif‐ferent kinds of array objects. For example:

```py
      >>> # array module example
>>> import sample
>>> import array
>>> a = array.array('d',[1,-3,4,7,2,0])
>>> a

```

array(‘d', [1.0, -3.0, 4.0, 7.0, 2.0, 0.0])>>> sample.clip(a,1,4,a)>>> aarray(‘d', [1.0, 1.0, 4.0, 4.0, 2.0, 1.0])

```py
      >>> # numpy example
>>> import numpy
>>> b = numpy.random.uniform(-10,10,size=1000000)
>>> b
array([-9.55546017,  7.45599334,  0.69248932, ...,  0.69583148,
       -3.86290931,  2.37266888])
>>> c = numpy.zeros_like(b)
>>> c
array([ 0.,  0.,  0., ...,  0.,  0.,  0.])
>>> sample.clip(b,-5,5,c)
>>> c
array([-5\.        ,  5\.        ,  0.69248932, ...,  0.69583148,
       -3.86290931,  2.37266888])
>>> min(c)
-5.0
>>> max(c)
5.0
>>>

```

You will also find that the resulting code is fast. The following session puts our imple‐mentation in a head-to-head battle with the clip() function already present in numpy:

```py
      >>> timeit('numpy.clip(b,-5,5,c)','from __main__ import b,c,numpy',number=1000)
8.093049556000551
>>> timeit('sample.clip(b,-5,5,c)','from __main__ import b,c,sample',
...         number=1000)
3.760528204000366
>>>

```

As you can see, it’s quite a bit faster—an interesting result considering the core of theNumPy version is written in C.

## 讨论

This recipe utilizes Cython typed memoryviews, which greatly simplify code that op‐erates on arrays. The declaration cpdef clip() declares clip() as both a C-level andPython-level function. In Cython, this is useful, because it means that the function callis more efficently called by other Cython functions (e.g., if you want to invoke clip()from a different Cython function).The typed parameters double[:] a and double[:] out declare those parameters asone-dimensional arrays of doubles. As input, they will access any array object thatproperly implements the memoryview interface, as described in PEP 3118\. This includesarrays from NumPy and from the built-in array library.

When writing code that produces a result that is also an array, you should follow theconvention shown of having an output parameter as shown. This places the responsi‐bility of creating the output array on the caller and frees the code from having to knowtoo much about the specific details of what kinds of arrays are being manipulated (itjust assumes the arrays are already in-place and only needs to perform a few basic sanitychecks such as making sure their sizes are compatible). In libraries such as NumPy, itis relatively easy to create output arrays using functions such as numpy.zeros() ornumpy.zeros_like(). Alternatively, to create uninitialized arrays, you can use numpy.empty() or numpy.empty_like(). This will be slightly faster if you’re about to over‐write the array contents with a result.In the implementation of your function, you simply write straightforward looking arrayprocessing code using indexing and array lookups (e.g., a[i], out[i], and so forth).Cython will take steps to make sure these produce efficient code.The two decorators that precede the definition of clip() are a few optional performanceoptimizations. @cython.boundscheck(False) eliminates all array bounds checking andcan be used if you know the indexing won’t go out of range. @cython.wraparound(False) eliminates the handling of negative array indices as wrapping aroundto the end of the array (like with Python lists). The inclusion of these decorators canmake the code run substantially faster (almost 2.5 times faster on this example whentested).Whenever working with arrays, careful study and experimentation with the underlyingalgorithm can also yield large speedups. For example, consider this variant of the clip()function that uses conditional expressions:

@cython.boundscheck(False)@cython.wraparound(False)cpdef clip(double[:] a, double min, double max, double[:] out):

> if min > max:raise ValueError(“min must be <= max”)if a.shape[0] != out.shape[0]:raise ValueError(“input and output arrays must be the same size”)for i in range(a.shape[0]):out[i] = (a[i] if a[i] < max else max) if a[i] > min else min

When tested, this version of the code runs over 50% faster (2.44s versus 3.76s on thetimeit() test shown earlier).At this point, you might be wondering how this code would stack up against a hand‐written C version. For example, perhaps you write the following C function and craft ahandwritten extension to using techniques shown in earlier recipes:

void clip(double *a, int n, double min, double max, double *out) {
double x;for (; n >= 0; n–, a++, out++) {

> > x = *a;
> 
> *out = x > max ? max : (x < min ? min : x);

}

}

The extension code for this isn’t shown, but after experimenting, we found that a hand‐crafted C extension ran more than 10% slower than the version created by Cython. Thebottom line is that the code runs a lot faster than you might think.There are several extensions that can be made to the solution code. For certain kinds ofarray operations, it might make sense to release the GIL so that multiple threads canrun in parallel. To do that, modify the code to include the with nogil: statement:

@cython.boundscheck(False)@cython.wraparound(False)cpdef clip(double[:] a, double min, double max, double[:] out):

> if min > max:raise ValueError(“min must be <= max”)if a.shape[0] != out.shape[0]:raise ValueError(“input and output arrays must be the same size”)with nogil:for i in range(a.shape[0]):out[i] = (a[i] if a[i] < max else max) if a[i] > min else min

If you want to write a version of the code that operates on two-dimensional arrays, hereis what it might look like:

@cython.boundscheck(False)@cython.wraparound(False)cpdef clip2d(double[:,:] a, double min, double max, double[:,:] out):

> if min > max:raise ValueError(“min must be <= max”)for n in range(a.ndim):if a.shape[n] != out.shape[n]:raise TypeError(“a and out have different shapes”)for i in range(a.shape[0]):for j in range(a.shape[1]):if a[i,j] < min:out[i,j] = minelif a[i,j] > max:out[i,j] = maxelse:out[i,j] = a[i,j]

Hopefully it’s not lost on the reader that all of the code in this recipe is not tied to anyspecific array library (e.g., NumPy). That gives the code a great deal of flexibility. How‐ever, it’s also worth noting that dealing with arrays can be significantly more complicatedonce multiple dimensions, strides, offsets, and other factors are introduced. Those top‐ics are beyond the scope of this recipe, but more information can be found in PEP3118\. The Cython documentation on “typed memoryviews” is also essential reading.

# 15.12 将函数指针转换为可调用对象

## 问题

You have (somehow) obtained the memory address of a compiled function, but wantto turn it into a Python callable that you can use as an extension function.

## 解决方案

The ctypes module can be used to create Python callables that wrap around arbitrarymemory addresses. The following example shows how to obtain the raw, low-level ad‐dress of a C function and how to turn it back into a callable object:

```py
      >>> import ctypes
>>> lib = ctypes.cdll.LoadLibrary(None)
>>> # Get the address of sin() from the C math library
>>> addr = ctypes.cast(lib.sin, ctypes.c_void_p).value
>>> addr
140735505915760

>>> # Turn the address into a callable function
>>> functype = ctypes.CFUNCTYPE(ctypes.c_double, ctypes.c_double)
>>> func = functype(addr)
>>> func
<CFunctionType object at 0x1006816d0>

>>> # Call the resulting function
>>> func(2)
0.9092974268256817
>>> func(0)
0.0
>>>

```

## 讨论

To make a callable, you must first create a CFUNCTYPE instance. The first argument toCFUNCTYPE() is the return type. Subsequent arguments are the types of the arguments.Once you have defined the function type, you wrap it around an integer memory addressto create a callable object. The resulting object is used like any normal function accessedthrough ctypes.This recipe might look rather cryptic and low level. However, it is becoming increasinglycommon for programs and libraries to utilize advanced code generation techniques likejust in-time compilation, as found in libraries such as LLVM.For example, here is a simple example that uses the llvmpy extension to make a smallassembly function, obtain a function pointer to it, and turn it into a Python callable:

```py
      >>> from llvm.core import Module, Function, Type, Builder
>>> mod = Module.new('example')
>>> f = Function.new(mod,Type.function(Type.double(), \
                     [Type.double(), Type.double()], False), 'foo')
>>> block = f.append_basic_block('entry')
>>> builder = Builder.new(block)
>>> x2 = builder.fmul(f.args[0],f.args[0])
>>> y2 = builder.fmul(f.args[1],f.args[1])
>>> r = builder.fadd(x2,y2)
>>> builder.ret(r)
<llvm.core.Instruction object at 0x10078e990>
>>> from llvm.ee import ExecutionEngine
>>> engine = ExecutionEngine.new(mod)
>>> ptr = engine.get_pointer_to_function(f)
>>> ptr
4325863440
>>> foo = ctypes.CFUNCTYPE(ctypes.c_double, ctypes.c_double, ctypes.c_double)(ptr)

>>> # Call the resulting function
>>> foo(2,3)
13.0
>>> foo(4,5)
41.0
>>> foo(1,2)
5.0
>>>

```

It goes without saying that doing anything wrong at this level will probably cause thePython interpreter to die a horrible death. Keep in mind that you’re directly workingwith machine-level memory addresses and native machine code—not Pythonfunctions.

# 15.13 传递 NULL 结尾的字符串给 C 函数库

## 问题

You are writing an extension module that needs to pass a NULL-terminated string to aC library. However, you’re not entirely sure how to do it with Python’s Unicode stringimplementation.

## 解决方案

Many C libraries include functions that operate on NULL-terminated strings declaredas type char *. Consider the following C function that we will use for the purposes ofillustration and testing:

void print_chars(char *s) {while (*s) {
printf(“%2x ”, (unsigned char) *s);

s++;

}printf(“n”);

}

This function simply prints out the hex representation of individual characters so thatthe passed strings can be easily debugged. For example:print_chars(“Hello”); // Outputs: 48 65 6c 6c 6f

For calling such a C function from Python, you have a few choices. First, you couldrestrict it to only operate on bytes using “y” conversion code to PyArg_ParseTuple()like this:

static PyObject *py_print_chars(PyObject *self, PyObject *args) {
char *s;

if (!PyArg_ParseTuple(args, “y”, &s)) {return NULL;
}print_chars(s);Py_RETURN_NONE;

}

The resulting function operates as follows. Carefully observe how bytes with embeddedNULL bytes and Unicode strings are rejected:

```py
      >>> print_chars(b'Hello World')
48 65 6c 6c 6f 20 57 6f 72 6c 64
>>> print_chars(b'Hello\x00World')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: must be bytes without null bytes, not bytes
>>> print_chars('Hello World')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' does not support the buffer interface
>>>

```

If you want to pass Unicode strings instead, use the “s” format code to PyArg_ParseTuple() such as this:

static PyObject *py_print_chars(PyObject *self, PyObject *args) {
char *s;

if (!PyArg_ParseTuple(args, “s”, &s)) {return NULL;
}print_chars(s);Py_RETURN_NONE;

}

When used, this will automatically convert all strings to a NULL-terminated UTF-8encoding. For example:

```py
      >>> print_chars('Hello World')
48 65 6c 6c 6f 20 57 6f 72 6c 64
>>> print_chars('Spicy Jalape\u00f1o')  # Note: UTF-8 encoding
53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
>>> print_chars('Hello\x00World')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: must be str without null characters, not str
>>> print_chars(b'Hello World')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: must be str, not bytes
>>>

```

If for some reason, you are working directly with a PyObject *and can’t use PyArg_ParseTuple(), the following code samples show how you can check and extract a suitablechar* reference, from both a bytes and string object:

/ *Some Python Object (obtained somehow) */PyObject *obj;

/ *Conversion from bytes */{

> > char *s;s = PyBytes_AsString(o);if (!s) {
> > 
> > return NULL; / *TypeError already raised */
> 
> }print_chars(s);

}

/ *Conversion to UTF-8 bytes from a string */{

> > PyObject *bytes;char *s;if (!PyUnicode_Check(obj)) {
> > 
> > PyErr_SetString(PyExc_TypeError, “Expected string”);return NULL;
> 
> }bytes = PyUnicode_AsUTF8String(obj);s = PyBytes_AsString(bytes);print_chars(s);Py_DECREF(bytes);

}

Both of the preceding conversions guarantee NULL-terminated data, but they do notcheck for embedded NULL bytes elsewhere inside the string. Thus, that’s somethingthat you would need to check yourself if it’s important.

### 讨论

If it all possible, you should try to avoid writing code that relies on NULL-terminatedstrings since Python has no such requirement. It is almost always better to handle stringsusing the combination of a pointer and a size if possible. Nevertheless, sometimes youhave to work with legacy C code that presents no other option.Although it is easy to use, there is a hidden memory overhead associated with using the“s” format code to PyArg_ParseTuple() that is easy to overlook. When you write codethat uses this conversion, a UTF-8 string is created and permanently attached to theoriginal string object. If the original string contains non-ASCII characters, this makesthe size of the string increase until it is garbage collected. For example:

```py
      >>> import sys
>>> s = 'Spicy Jalape\u00f1o'
>>> sys.getsizeof(s)
87
>>> print_chars(s)     # Passing string
53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
>>> sys.getsizeof(s)   # Notice increased size
103
>>>

```

If this growth in memory use is a concern, you should rewrite your C extension codeto use the PyUnicode_AsUTF8String() function like this:

static PyObject *py_print_chars(PyObject *self, PyObject *args) {
PyObject *o, *bytes;char *s;

if (!PyArg_ParseTuple(args, “U”, &o)) {return NULL;
}bytes = PyUnicode_AsUTF8String(o);s = PyBytes_AsString(bytes);print_chars(s);Py_DECREF(bytes);Py_RETURN_NONE;

}

With this modification, a UTF-8 encoded string is created if needed, but then discardedafter use. Here is the modified behavior:

```py
      >>> import sys
>>> s = 'Spicy Jalape\u00f1o'
>>> sys.getsizeof(s)
87
>>> print_chars(s)
53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
>>> sys.getsizeof(s)
87
>>>

```

If you are trying to pass NULL-terminated strings to functions wrapped via ctypes, beaware that ctypes only allows bytes to be passed and that it does not check for embeddedNULL bytes. For example:

```py
      >>> import ctypes
>>> lib = ctypes.cdll.LoadLibrary("./libsample.so")
>>> print_chars = lib.print_chars
>>> print_chars.argtypes = (ctypes.c_char_p,)
>>> print_chars(b'Hello World')
48 65 6c 6c 6f 20 57 6f 72 6c 64
>>> print_chars(b'Hello\x00World')
48 65 6c 6c 6f
>>> print_chars('Hello World')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ctypes.ArgumentError: argument 1: <class 'TypeError'>: wrong type
>>>

```

If you want to pass a string instead of bytes, you need to perform a manual UTF-8encoding first. For example:

```py
      >>> print_chars('Hello World'.encode('utf-8'))
48 65 6c 6c 6f 20 57 6f 72 6c 64
>>>

```

For other extension tools (e.g., Swig, Cython), careful study is probably in order shouldyou decide to use them to pass strings to C code.

# 15.14 传递 Unicode 字符串给 C 函数库

## 问题

You are writing an extension module that needs to pass a Python string to a C libraryfunction that may or may not know how to properly handle Unicode.

## 解决方案

There are many issues to be concerned with here, but the main one is that existing Clibraries won’t understand Python’s native representation of Unicode. Therefore, yourchallenge is to convert the Python string into a form that can be more easily understoodby C libraries.For the purposes of illustration, here are two C functions that operate on string dataand output it for the purposes of debugging and experimentation. One uses bytes pro‐vided in the form char *, int, whereas the other uses wide characters in the formwchar_t *, int:

void print_chars(char *s, int len) {
int n = 0;

while (n < len) {printf(“%2x ”, (unsigned char) s[n]);n++;
}printf(“n”);

}

void print_wchars(wchar_t *s, int len) {
int n = 0;while (n < len) {

> printf(“%x ”, s[n]);n++;

}printf(“n”);

}

For the byte-oriented function print_chars(), you need to convert Python strings intoa suitable byte encoding such as UTF-8\. Here is a sample extension function that doesthis:

static PyObject *py_print_chars(PyObject *self, PyObject *args) {
char *s;Py_ssize_t len;

if (!PyArg_ParseTuple(args, “s#”, &s, &len)) {return NULL;
}print_chars(s, len);Py_RETURN_NONE;

}

For library functions that work with the machine native wchar_t type, you can writeextension code such as this:

static PyObject *py_print_wchars(PyObject *self, PyObject *args) {
wchar_t *s;Py_ssize_t len;

if (!PyArg_ParseTuple(args, “u#”, &s, &len)) {return NULL;
}print_wchars(s,len);Py_RETURN_NONE;

}

Here is an interactive session that illustrates how these functions work:

```py
      >>> s = 'Spicy Jalape\u00f1o'
>>> print_chars(s)
53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
>>> print_wchars(s)
53 70 69 63 79 20 4a 61 6c 61 70 65 f1 6f
>>>

```

Carefully observe how the byte-oriented function print_chars() is receiving UTF-8encoded data, whereas print_wchars() is receiving the Unicode code point values.

## 讨论

Before considering this recipe, you should first study the nature of the C library thatyou’re accessing. For many C libraries, it might make more sense to pass bytes insteadof a string. To do that, use this conversion code instead:

static PyObject *py_print_chars(PyObject *self, PyObject *args) {
char *s;Py_ssize_t len;

/ *accepts bytes, bytearray, or other byte-like object */if (!PyArg_ParseTuple(args, “y#”, &s, &len)) {

> return NULL;

}print_chars(s, len);Py_RETURN_NONE;

}

If you decide that you still want to pass strings, you need to know that Python 3 uses anadaptable string representation that is not entirely straightforward to map directly to Clibraries using the standard types char *or wchar_t* See PEP 393 for details. Thus,to present string data to C, some kind of conversion is almost always necessary. The s#and u# format codes to PyArg_ParseTuple() safely perform such conversions.One potential downside is that such conversions cause the size of the original stringobject to permanently increase. Whenever a conversion is made, a copy of the converteddata is kept and attached to the original string object so that it can be reused later. Youcan observe this effect:

```py
      >>> import sys
>>> s = 'Spicy Jalape\u00f1o'
>>> sys.getsizeof(s)
87
>>> print_chars(s)
53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
>>> sys.getsizeof(s)
103
>>> print_wchars(s)
53 70 69 63 79 20 4a 61 6c 61 70 65 f1 6f
>>> sys.getsizeof(s)
163
>>>

```

For small amounts of string data, this might not matter, but if you’re doing large amountsof text processing in extensions, you may want to avoid the overhead. Here is analternative implementation of the first extension function that avoids these memoryinefficiencies:

static PyObject *py_print_chars(PyObject *self, PyObject *args) {
PyObject *obj, *bytes;char *s;Py_ssize_t len;

if (!PyArg_ParseTuple(args, “U”, &obj)) {return NULL;
}bytes = PyUnicode_AsUTF8String(obj);PyBytes_AsStringAndSize(bytes, &s, &len);print_chars(s, len);Py_DECREF(bytes);Py_RETURN_NONE;

}

Avoiding memory overhead for wchar_t handling is much more tricky. Internally,Python stores strings using the most efficient representation possible. For example,strings containing nothing but ASCII are stored as arrays of bytes, whereas strings con‐taining characters in the range U+0000 to U+FFFF use a two-byte representation. Sincethere isn’t a single representation of the data, you can’t just cast the internal array towchar_t * and hope that it works. Instead, a wchar_t array has to be created and textcopied into it. The “u#” format code to PyArg_ParseTuple() does this for you at thecost of efficiency (it attaches the resulting copy to the string object).If you want to avoid this long-term memory overhead, your only real choice is to copythe Unicode data into a temporary array, pass it to the C library function, and thendeallocate the array. Here is one possible implementation:

static PyObject *py_print_wchars(PyObject *self, PyObject *args) {
PyObject *obj;wchar_t *s;Py_ssize_t len;

if (!PyArg_ParseTuple(args, “U”, &obj)) {return NULL;
}if ((s = PyUnicode_AsWideCharString(obj, &len)) == NULL) {

> return NULL;

}print_wchars(s, len);PyMem_Free(s);Py_RETURN_NONE;

}

In this implementation, PyUnicode_AsWideCharString() creates a temporary buffer ofwchar_t characters and copies data into it. That buffer is passed to C and then releasedafterward. As of this writing, there seems to be a possible bug related to this behavior,as described at the Python issues page.

If, for some reason you know that the C library takes the data in a different byte encodingthan UTF-8, you can force Python to perform an appropriate conversion using exten‐sion code such as the following:

static PyObject *py_print_chars(PyObject *self, PyObject *args) {
char *s = 0;int len;if (!PyArg_ParseTuple(args, “es#”, “encoding-name”, &s, &len)) {

> return NULL;

}print_chars(s, len);PyMem_Free(s);Py_RETURN_NONE;

}

Last, but not least, if you want to work directly with the characters in a Unicode string,here is an example that illustrates low-level access:

static PyObject *py_print_wchars(PyObject *self, PyObject *args) {
PyObject *obj;int n, len;int kind;void *data;

if (!PyArg_ParseTuple(args, “U”, &obj)) {return NULL;
}if (PyUnicode_READY(obj) < 0) {

> return NULL;

}

len = PyUnicode_GET_LENGTH(obj);kind = PyUnicode_KIND(obj);data = PyUnicode_DATA(obj);

for (n = 0; n < len; n++) {Py_UCS4 ch = PyUnicode_READ(kind, data, n);printf(“%x ”, ch);
}printf(“n”);Py_RETURN_NONE;

}

In this code, the PyUnicode_KIND() and PyUnicode_DATA() macros are related to thevariable-width storage of Unicode, as described in PEP 393\. The kind variable encodesinformation about the underlying storage (8-bit, 16-bit, or 32-bit) and data points thebuffer. In reality, you don’t need to do anything with these values as long as you passthem to the PyUnicode_READ() macro when extracting characters.A few final words: when passing Unicode strings from Python to C, you should probablytry to make it as simple as possible. If given the choice between an encoding such as

UTF-8 or wide characters, choose UTF-8\. Support for UTF-8 seems to be much morecommon, less trouble-prone, and better supported by the interpreter. Finally, make sureyour review the documentation on Unicode handling.

# 15.15 C 字符串转换为 Python 字符串

## 问题

You want to convert strings from C to Python bytes or a string object.

## 解决方案

For C strings represented as a pair char *, int, you must decide whether or not youwant the string presented as a raw byte string or as a Unicode string. Byte objects canbe built using Py_BuildValue() as follows:

char *s; /* Pointer to C string data */int len; /* Length of data */

/ *Make a bytes object */PyObject *obj = Py_BuildValue(“y#”, s, len);

If you want to create a Unicode string and you know that s points to data encoded asUTF-8, you can use the following:

PyObject *obj = Py_BuildValue(“s#”, s, len);

If s is encoded in some other known encoding, you can make a string using PyUnicode_Decode() as follows:

PyObject *obj = PyUnicode_Decode(s, len, “encoding”, “errors”);

/ *Examples /*obj = PyUnicode_Decode(s, len, “latin-1”, “strict”);obj = PyUnicode_Decode(s, len, “ascii”, “ignore”);

If you happen to have a wide string represented as a wchar_t *, len pair, there are afew options. First, you could use Py_BuildValue() as follows:

wchar_t *w; /* Wide character string */int len; /* Length */

PyObject *obj = Py_BuildValue(“u#”, w, len);

Alternatively, you can use PyUnicode_FromWideChar():

PyObject *obj = PyUnicode_FromWideChar(w, len);

For wide character strings, no interpretation is made of the character data—it is assumedto be raw Unicode code points which are directly converted to Python.

## 讨论

Conversion of strings from C to Python follow the same principles as I/O. Namely, thedata from C must be explicitly decoded into a string according to some codec. Commonencodings include ASCII, Latin-1, and UTF-8\. If you’re not entirely sure of the encodingor the data is binary, you’re probably best off encoding the string as bytes instead.When making an object, Python always copies the string data you provide. If necessary,it’s up to you to release the C string afterward (if required). Also, for better reliability,you should try to create strings using both a pointer and a size rather than relying onNULL-terminated data.

# 15.16 不确定编码格式的 C 字符串

## 问题

You are converting strings back and forth between C and Python, but the C encodingis of a dubious or unknown nature. For example, perhaps the C data is supposed to beUTF-8, but it’s not being strictly enforced. You would like to write code that can handlemalformed data in a graceful way that doesn’t crash Python or destroy the string datain the process.

## 解决方案

Here is some C data and a function that illustrates the nature of this problem:

/ *Some dubious string data (malformed UTF-8) */const char *sdata = “Spicy Jalapexc3xb1oxae”;int slen = 16;

/ *Output character data */void print_chars(char *s, int len) {

> > int n = 0;while (n < len) {
> > 
> > printf(“%2x ”, (unsigned char) s[n]);n++;
> 
> }printf(“n”);

}

In this code, the string sdata contains a mix of UTF-8 and malformed data. Neverthe‐less, if a user calls print_chars(sdata, slen) in C, it works fine.Now suppose you want to convert the contents of sdata into a Python string. Furthersuppose you want to later pass that string to the print_chars() function through anextension. Here’s how to do it in a way that exactly preserves the original data eventhough there are encoding problems:

/ *Return the C string back to Python */static PyObject *py_retstr(PyObject *self, PyObject *args) {

> if (!PyArg_ParseTuple(args, “”)) {return NULL;> }return PyUnicode_Decode(sdata, slen, “utf-8”, “surrogateescape”);

}

/ *Wrapper for the print_chars() function */static PyObject *py_print_chars(PyObject *self, PyObject *args) {

> > PyObject *obj, *bytes;char *s = 0;Py_ssize_t len;

if (!PyArg_ParseTuple(args, “U”, &obj)) {return NULL;> }

if ((bytes = PyUnicode_AsEncodedString(obj,”utf-8”,”surrogateescape”))> > == NULL) {

> return NULL;
> 
> }PyBytes_AsStringAndSize(bytes, &s, &len);print_chars(s, len);Py_DECREF(bytes);Py_RETURN_NONE;

}

If you try these functions from Python, here’s what happens:

```py
      >>> s = retstr()
>>> s
'Spicy Jalapeño\udcae'
>>> print_chars(s)
53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f ae
>>>

```

Careful observation will reveal that the malformed string got encoded into a Pythonstring without errors, and that when passed back into C, it turned back into a byte stringthat exactly encoded the same bytes as the original C string.

## 讨论

This recipe addresses a subtle, but potentially annoying problem with string handlingin extension modules. Namely, the fact that C strings in extensions might not follow thestrict Unicode encoding/decoding rules that Python normally expects. Thus, it’s possiblethat some malformed C data would pass to Python. A good example might be C stringsassociated with low-level system calls such as filenames. For instance, what happens ifa system call returns a broken string back to the interpreter that can’t be properlydecoded.

Normally, Unicode errors are often handled by specifying some sort of error policy, suchas strict, ignore, replace, or something similar. However, a downside of these policiesis that they irreparably destroy the original string content. For example, if the malformeddata in the example was decoded using one of these polices, you would get results suchas this:

```py
      >>> raw = b'Spicy Jalape\xc3\xb1o\xae'
>>> raw.decode('utf-8','ignore')
'Spicy Jalapeño'
>>> raw.decode('utf-8','replace')
'Spicy Jalapeño?'
>>>

```

The surrogateescape error handling policies takes all nondecodable bytes and turnsthem into the low-half of a surrogate pair (udcXX where XX is the raw byte value). Forexample:

```py
      >>> raw.decode('utf-8','surrogateescape')
'Spicy Jalapeño\udcae'
>>>

```

Isolated low surrogate characters such as udcae never appear in valid Unicode. Thus,this string is technically an illegal representation. In fact, if you ever try to pass it tofunctions that perform output, you’ll get encoding errors:

```py
      >>> s = raw.decode('utf-8', 'surrogateescape')
>>> print(s)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'utf-8' codec can't encode character '\udcae'
in position 14: surrogates not allowed
>>>

```

However, the main point of allowing the surrogate escapes is to allow malformed stringsto pass from C to Python and back into C without any data loss. When the string isencoded using surrogateescape again, the surrogate characters are turned back intotheir original bytes. For example:

```py
      >>> s
'Spicy Jalapeño\udcae'
>>> s.encode('utf-8','surrogateescape')
b'Spicy Jalape\xc3\xb1o\xae'
>>>

```

As a general rule, it’s probably best to avoid surrogate encoding whenever possible—your code will be much more reliable if it uses proper encodings. However, sometimesthere are situations where you simply don’t have control over the data encoding andyou aren’t free to ignore or replace the bad data because other functions may need touse it. This recipe shows how to do it.

As a final note, many of Python’s system-oriented functions, especially those related tofilenames, environment variables, and command-line options, use surrogate encoding.For example, if you use a function such as os.listdir() on a directory containing aundecodable filename, it will be returned as a string with surrogate escapes. SeeRecipe 5.15 for a related recipe.PEP 383 has more information about the problem addressed by this recipe and surrogateescape error handling.

# 15.17 传递文件名给 C 扩展

## 问题

You need to pass filenames to C library functions, but need to make sure the filenamehas been encoded according to the system’s expected filename encoding.

## 解决方案

To write an extension function that receives a filename, use code such as this:

static PyObject *py_get_filename(PyObject *self, PyObject *args) {
PyObject *bytes;char *filename;Py_ssize_t len;if (!PyArg_ParseTuple(args,”O&”, PyUnicode_FSConverter, &bytes)) {

> return NULL;

}PyBytes_AsStringAndSize(bytes, &filename, &len);/ *Use filename */...

/ *Cleanup and return */Py_DECREF(bytes)Py_RETURN_NONE;

}

If you already have a PyObject * that you want to convert as a filename, use code suchas the following:

PyObject *obj; /* Object with the filename */PyObject *bytes;char *filename;Py_ssize_t len;

bytes = PyUnicode_EncodeFSDefault(obj);PyBytes_AsStringAndSize(bytes, &filename, &len);/ *Use filename */...

/ *Cleanup */Py_DECREF(bytes);

If you need to return a filename back to Python, use the following code:

/ *Turn a filename into a Python object */

char *filename; /* Already set _/int filename*len; /* Already set */

PyObject *obj = PyUnicode_DecodeFSDefaultAndSize(filename, filename_len);

## 讨论

Dealing with filenames in a portable way is a tricky problem that is best left to Python.If you use this recipe in your extension code, filenames will be handled in a manner thatis consistent with filename handling in the rest of Python. This includes encoding/decoding of bytes, dealing with bad characters, surrogate escapes, and other complica‐tions.

# 15.18 传递已打开的文件给 C 扩展

## 问题

You have an open file object in Python, but need to pass it to C extension code that willuse the file.

## 解决方案

To convert a file to an integer file descriptor, use PyFile_FromFd(), as shown:

PyObject *fobj; /* File object (already obtained somehow) */int fd = PyObject_AsFileDescriptor(fobj);if (fd < 0) {

> return NULL;

}

The resulting file descriptor is obtained by calling the fileno() method on fobj. Thus,any object that exposes a descriptor in this manner should work (e.g., file, socket, etc.).Once you have the descriptor, it can be passed to various low-level C functions thatexpect to work with files.If you need to convert an integer file descriptor back into a Python object, usePyFile_FromFd() as follows:

int fd; / *Existing file descriptor (already open) */PyObject *fobj = PyFile_FromFd(fd, “filename”,”r”,-1,NULL,NULL,NULL,1);

The arguments to PyFile_FromFd() mirror those of the built-in open() function. NULLvalues simply indicate that the default settings for the encoding, errors, and newlinearguments are being used.

## 讨论

If you are passing file objects from Python to C, there are a few tricky issues to beconcerned about. First, Python performs its own I/O buffering through the io module.Prior to passing any kind of file descriptor to C, you should first flush the I/O bufferson the associated file objects. Otherwise, you could get data appearing out of order onthe file stream.Second, you need to pay careful attention to file ownership and the responsibility ofclosing the file in particular. If a file descriptor is passed to C, but still used in Python,you need to make sure C doesn’t accidentally close the file. Likewise, if a file descriptoris being turned into a Python file object, you need to be clear about who is responsiblefor closing it. The last argument to PyFile_FromFd() is set to 1 to indicate that Pythonshould close the file.If you need to make a different kind of file object such as a FILE * object from the Cstandard I/O library using a function such as fdopen(), you’ll need to be especiallycareful. Doing so would introduce two completely different I/O buffering layers intothe I/O stack (one from Python’s io module and one from C stdio). Operations suchas fclose() in C could also inadvertently close the file for further use in Python. If givena choice, you should probably make extension code work with the low-level integer filedescriptors as opposed to using a higher-level abstraction such as that provided by<stdio.h>.</stdio.h>

# 15.19 从 C 语言中读取类文件对象

## 问题

You want to write C extension code that consumes data from any Python file-like object(e.g., normal files, StringIO objects, etc.).

## 解决方案

To consume data on a file-like object, you need to repeatedly invoke its read() methodand take steps to properly decode the resulting data.Here is a sample C extension function that merely consumes all of the data on a file-likeobject and dumps it to standard output so you can see it:

# define CHUNK_SIZE 8192

/ *Consume a “file-like” object and write bytes to stdout */static PyObject *py_consume_file(PyObject *self, PyObject *args) {

> > > > > PyObject *obj;PyObject *read_meth;PyObject *result = NULL;PyObject *read_args;

if (!PyArg_ParseTuple(args,”O”, &obj)) {return NULL;> > }

> > / *Get the read method of the passed object */if ((read_meth = PyObject_GetAttrString(obj, “read”)) == NULL) {
> > 
> > > return NULL;
> > 
> > }
> > 
> > / *Build the argument list to read() */read_args = Py_BuildValue(“(i)”, CHUNK_SIZE);while (1) {
> > 
> > > > > > PyObject *data;PyObject *enc_data;char *buf;Py_ssize_t len;
> > > 
> > > / *Call read() */if ((data = PyObject_Call(read_meth, read_args, NULL)) == NULL) {
> > > 
> > > > goto final;
> > > 
> > > }
> > > 
> > > / *Check for EOF */if (PySequence_Length(data) == 0) {
> > > 
> > > > Py_DECREF(data);break;
> > > 
> > > }
> > > 
> > > / *Encode Unicode as Bytes for C */if ((enc_data=PyUnicode_AsEncodedString(data,”utf-8”,”strict”))==NULL) {
> > > 
> > > > Py_DECREF(data);goto final;
> > > 
> > > }
> > > 
> > > / *Extract underlying buffer data */PyBytes_AsStringAndSize(enc_data, &buf, &len);
> > > 
> > > / *Write to stdout (replace with something more useful) */write(1, buf, len);
> > > 
> > > / *Cleanup */Py_DECREF(enc_data);Py_DECREF(data);
> > 
> > }result = Py_BuildValue(“”);

final:/ *Cleanup */Py_DECREF(read_meth);Py_DECREF(read_args);return result;

}

To test the code, try making a file-like object such as a StringIO instance and pass it in:

```py
      >>> import io
>>> f = io.StringIO('Hello\nWorld\n')
>>> import sample
>>> sample.consume_file(f)
Hello
World
>>>

```

## 讨论

Unlike a normal system file, a file-like object is not necessarily built around a low-levelfile descriptor. Thus, you can’t use normal C library functions to access it. Instead, youneed to use Python’s C API to manipulate the file-like object much like you would inPython.In the solution, the read() method is extracted from the passed object. An argumentlist is built and then repeatedly passed to PyObject_Call() to invoke the method. Todetect end-of-file (EOF), PySequence_Length() is used to see if the returned result haszero length.For all I/O operations, you’ll need to concern yourself with the underlying encodingand distinction between bytes and Unicode. This recipe shows how to read a file in textmode and decode the resulting text into a bytes encoding that can be used by C. If youwant to read the file in binary mode, only minor changes will be made. For example:

...
/ *Call read() */if ((data = PyObject_Call(read_meth, read_args, NULL)) == NULL) {

> goto final;

}

/ *Check for EOF */if (PySequence_Length(data) == 0) {

> Py_DECREF(data);break;

}if (!PyBytes_Check(data)) {

> Py_DECREF(data);PyErr_SetString(PyExc_IOError, “File must be in binary mode”);goto final;

}

/ *Extract underlying buffer data */PyBytes_AsStringAndSize(data, &buf, &len);...

The trickiest part of this recipe concerns proper memory management. When workingwith PyObject * variables, careful attention needs to be given to managing referencecounts and cleaning up values when no longer needed. The various Py_DECREF() callsare doing this.The recipe is written in a general-purpose manner so that it can be adapted to other fileoperations, such as writing. For example, to write data, merely obtain the write()method of the file-like object, convert data into an appropriate Python object (bytes orUnicode), and invoke the method to have it written to the file.Finally, although file-like objects often provide other methods (e.g., readline(),read_into()), it is probably best to just stick with the basic read() and write() meth‐ods for maximal portability. Keeping things as simple as possible is often a good policyfor C extensions.

# 15.20 处理 C 语言中的可迭代对象

## 问题

You want to write C extension code that consumes items from any iterable object suchas a list, tuple, file, or generator.

## 解决方案

Here is a sample C extension function that shows how to consume the items on aniterable:

static PyObject *py_consume_iterable(PyObject *self, PyObject *args) {
PyObject *obj;PyObject *iter;PyObject *item;

if (!PyArg_ParseTuple(args, “O”, &obj)) {return NULL;
}if ((iter = PyObject_GetIter(obj)) == NULL) {

> return NULL;

}while ((item = PyIter_Next(iter)) != NULL) {

> / *Use item */...Py_DECREF(item);

}

Py_DECREF(iter);return Py_BuildValue(“”);

}

## 讨论

The code in this recipe mirrors similar code in Python. The PyObject_GetIter() callis the same as calling iter() to get an iterator. The PyIter_Next() function invokesthe next method on the iterator returning the next item or NULL if there are no moreitems. Make sure you’re careful with memory management—Py_DECREF() needs to becalled on both the produced items and the iterator object itself to avoid leaking memory.

# 15.21 诊断分析代码错误

## 问题

The interpreter violently crashes with a segmentation fault, bus error, access violation,or other fatal error. You would like to get a Python traceback that shows you where yourprogram was running at the point of failure.

## 解决方案

The faulthandler module can be used to help you solve this problem. Include thefollowing code in your program:

import faulthandlerfaulthandler.enable()

Alternatively, run Python with the -Xfaulthandler option such as this:

bash % python3 -Xfaulthandler program.py

Last, but not least, you can set the PYTHONFAULTHANDLER environment variable.With faulthandler enabled, fatal errors in C extensions will result in a Python trace‐back being printed on failures. For example:

> > Fatal Python error: Segmentation fault

Current thread 0x00007fff71106cc0:File “example.py”, line 6 in fooFile “example.py”, line 10 in barFile “example.py”, line 14 in spamFile “example.py”, line 19 in <module>> Segmentation fault</module>

Although this won’t tell you where in the C code things went awry, at least it can tell youhow it got there from Python.

## 讨论

The faulthandler will show you the stack traceback of the Python code executing atthe time of failure. At the very least, this will show you the top-level extension functionthat was invoked. With the aid of pdb or other Python debugger, you can investigate theflow of the Python code leading to the error.faulthandler will not tell you anything about the failure from C. For that, you willneed to use a traditional C debugger, such as gdb. However, the information from thefaulthandler traceback may give you a better idea of where to direct your attention.It should be noted that certain kinds of errors in C may not be easily recoverable. Forexample, if a C extension trashes the stack or program heap, it may render faulthandler inoperable and you’ll simply get no output at all (other than a crash). Obviously,your mileage may vary.