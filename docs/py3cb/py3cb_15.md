# 第十三章：脚本编程与系统管理

A lot of people use Python as a replacement for shell scripts, using it to automate commonsystem tasks, such as manipulating files, configuring systems, and so forth. Themain goal of this chapter is to describe features related to common tasks encounteredwhen writing scripts. For example, parsing command-line options, manipulating fileson the filesystem, getting useful system configuration data, and so forth. Chapter 5 alsocontains general information related to files and directories.

# 13.1 通过重定向/管道/文件接受输入

## 问题

You want a script you’ve written to be able to accept input using whatever mechanismis easiest for the user. This should include piping output from a command to the script,redirecting a file into the script, or just passing a filename, or list of filenames, to thescript on the command line.

## 解决方案

Python’s built-in fileinput module makes this very simple and concise. If you have ascript that looks like this:#!/usr/bin/env python3import fileinput

with fileinput.input() as f_input:for line in f_input:print(line, end='‘)
Then you can already accept input to the script in all of the previously mentioned ways.If you save this script as filein.py and make it executable, you can do all of the followingand get the expected output:

$ ls | ./filein.py # Prints a directory listing to stdout.$ ./filein.py /etc/passwd # Reads /etc/passwd to stdout.$ ./filein.py < /etc/passwd # Reads /etc/passwd to stdout.

## 讨论

The fileinput.input() function creates and returns an instance of the FileInputclass. In addition to containing a few handy helper methods, the instance can also beused as a context manager. So, to put all of this together, if we wrote a script that expectedto be printing output from several files at once, we might have it include the filenameand line number in the output, like this:

```py
>>> import fileinput
>>> with fileinput.input('/etc/passwd') as f:
>>>     for line in f:
...         print(f.filename(), f.lineno(), line, end='')
...
/etc/passwd 1 ##
/etc/passwd 2 # User Database
/etc/passwd 3 #

```

<other output="" omitted="">Using it as a context manager ensures that the file is closed when it’s no longer beingused, and we leveraged a few handy FileInput helper methods here to get some extrainformation in the output.</other>

# 13.2 终止程序并给出错误信息

## 问题

You want your program to terminate by printing a message to standard error and re‐turning a nonzero status code.

## 解决方案

To have a program terminate in this manner, raise a SystemExit exception, but supplythe error message as an argument. For example:

raise SystemExit(‘It failed!')

This will cause the supplied message to be printed to sys.stderr and the program toexit with a status code of 1.

## 讨论

This is a small recipe, but it solves a common problem that arises when writing scripts.Namely, to terminate a program, you might be inclined to write code like this:

import syssys.stderr.write(‘It failed!n')raise SystemExit(1)

None of the extra steps involving import or writing to sys.stderr are neccessary if yousimply supply the message to SystemExit() instead.

# 13.3 解析命令行选项

## 问题

You want to write a program that parses options supplied on the command line (foundin sys.argv).

## 解决方案

The argparse module can be used to parse command-line options. A simple examplewill help to illustrate the essential features:

# search.py‘''Hypothetical command-line tool for searching a collection offiles for one or more text patterns.‘''import argparseparser = argparse.ArgumentParser(description='Search some files')

parser.add_argument(dest='filenames',metavar='filename', nargs='*')

parser.add_argument(‘-p', ‘–pat',metavar='pattern', required=True,dest='patterns', action='append',help='text pattern to search for')parser.add_argument(‘-v', dest='verbose', action='store_true',help='verbose mode')parser.add_argument(‘-o', dest='outfile', action='store',help='output file')parser.add_argument(‘–speed', dest='speed', action='store',choices={‘slow','fast'}, default='slow',help='search speed')
args = parser.parse_args()

# Output the collected argumentsprint(args.filenames)print(args.patterns)print(args.verbose)print(args.outfile)print(args.speed)

This program defines a command-line parser with the following usage:

bash % python3 search.py -husage: search.py [-h] [-p pattern] [-v] [-o OUTFILE] [–speed {slow,fast}]

> [filename [filename ...]]

Search some files

positional arguments:filenameoptional arguments:

| `-h, --help` | show this help message and exit |
| `-p `pattern`, --pat `pattern`` |
|   | text pattern to search for |
| `-v` | verbose mode |
| `-o `OUTFILE`` | output file |

–speed {slow,fast} search speed

The following session shows how data shows up in the program. Carefully observe theoutput of the print() statements.

bash % python3 search.py foo.txt bar.txtusage: search.py [-h] -p pattern [-v] [-o OUTFILE] [–speed {fast,slow}]

> [filename [filename ...]]

search.py: error: the following arguments are required: -p/–pat

bash % python3 search.py -v -p spam –pat=eggs foo.txt bar.txtfilenames = [‘foo.txt', ‘bar.txt']patterns = [‘spam', ‘eggs']verbose = Trueoutfile = Nonespeed = slow

bash % python3 search.py -v -p spam –pat=eggs foo.txt bar.txt -o resultsfilenames = [‘foo.txt', ‘bar.txt']patterns = [‘spam', ‘eggs']verbose = Trueoutfile = resultsspeed = slow

bash % python3 search.py -v -p spam –pat=eggs foo.txt bar.txt -o results –speed=fast
filenames = [‘foo.txt', ‘bar.txt']patterns = [‘spam', ‘eggs']verbose = Trueoutfile = resultsspeed = fast

Further processing of the options is up to the program. Replace the print() functionswith something more interesting.

## 讨论

The argparse module is one of the largest modules in the standard library, and has ahuge number of configuration options. This recipe shows an essential subset that canbe used and extended to get started.To parse options, you first create an ArgumentParser instance and add declarations forthe options you want to support it using the add_argument() method. In each add_argument() call, the dest argument specifies the name of an attribute where the result ofparsing will be placed. The metavar argument is used when generating help messages.The action argument specifies the processing associated with the argument and is oftenstore for storing a value or append for collecting multiple argument values into a list.The following argument collects all of the extra command-line arguments into a list. It’sbeing used to make a list of filenames in the example:

parser.add_argument(dest='filenames',metavar='filename', nargs='*')

The following argument sets a Boolean flag depending on whether or not the argumentwas provided:

parser.add_argument(‘-v', dest='verbose', action='store_true',help='verbose mode')
The following argument takes a single value and stores it as a string:

parser.add_argument(‘-o', dest='outfile', action='store',help='output file')
The following argument specification allows an argument to be repeated multiple timesand all of the values append into a list. The required flag means that the argument mustbe supplied at least once. The use of -p and –pat mean that either argument name isacceptable.

parser.add_argument(‘-p', ‘–pat',metavar='pattern', required=True,dest='patterns', action='append',help='text pattern to search for')
Finally, the following argument specification takes a value, but checks it against a set ofpossible choices.

parser.add_argument(‘–speed', dest='speed', action='store',choices={‘slow','fast'}, default='slow',help='search speed')
Once the options have been given, you simply execute the parser.parse() method.This will process the sys.argv value and return an instance with the results. The results

for each argument are placed into an attribute with the name given in the dest parameterto add_argument().There are several other approaches for parsing command-line options. For example,you might be inclined to manually process sys.argv yourself or use the getopt module(which is modeled after a similarly named C library). However, if you take this approach,you’ll simply end up replicating much of the code that argparse already provides. Youmay also encounter code that uses the optparse library to parse options. Althoughoptparse is very similar to argparse, the latter is more modern and should be preferredin new projects.

# 13.4 运行时弹出密码输入提示

## 问题

You’ve written a script that requires a password, but since the script is meant for inter‐active use, you’d like to prompt the user for a password rather than hardcode it into thescript.

## 解决方案

Python’s getpass module is precisely what you need in this situation. It will allow youto very easily prompt for a password without having the keyed-in password displayedon the user’s terminal. Here’s how it’s done:

import getpass

user = getpass.getuser()passwd = getpass.getpass()

if svc_login(user, passwd): # You must write svc_login()print(‘Yay!')else:print(‘Boo!')
In this code, the svc_login() function is code that you must write to further processthe password entry. Obviously, the exact handling is application-specific.

## 讨论

Note in the preceding code that getpass.getuser() doesn’t prompt the user for theirusername. Instead, it uses the current user’s login name, according to the user’s shellenvironment, or as a last resort, according to the local system’s password database (onplatforms that support the pwd module).

If you want to explicitly prompt the user for their username, which can be more reliable,use the built-in input function:

user = input(‘Enter your username: ‘)

It’s also important to remember that some systems may not support the hiding of thetyped password input to the getpass() method. In this case, Python does all it can toforewarn you of problems (i.e., it alerts you that passwords will be shown in cleartext)before moving on.

# 13.5 获取终端的大小

## 问题

You need to get the terminal size in order to properly format the output of your program.

## 解决方案

Use the os.get_terminal_size() function to do this:

```py
>>> import os
>>> sz = os.get_terminal_size()
>>> sz
os.terminal_size(columns=80, lines=24)
>>> sz.columns
80
>>> sz.lines
24
>>>

```

## 讨论

There are many other possible approaches for obtaining the terminal size, ranging fromreading environment variables to executing low-level system calls involving ioctl()and TTYs. Frankly, why would you bother with that when this one simple call willsuffice?

# 13.6 执行外部命令并获取它的输出

## 问题

You want to execute an external command and collect its output as a Python string.

## 解决方案

Use the subprocess.check_output() function. For example:

import subprocessout_bytes = subprocess.check_output([‘netstat','-a'])

This runs the specified command and returns its output as a byte string. If you need tointerpret the resulting bytes as text, add a further decoding step. For example:

out_text = out_bytes.decode(‘utf-8')

If the executed command returns a nonzero exit code, an exception is raised. Here isan example of catching errors and getting the output created along with the exit code:

try:out_bytes = subprocess.check_output([‘cmd','arg1','arg2'])except subprocess.CalledProcessError as e:out_bytes = e.output # Output generated before errorcode = e.returncode # Return code
By default, check_output() only returns output written to standard output. If you wantboth standard output and error collected, use the stderr argument:

out_bytes = subprocess.check_output([‘cmd','arg1','arg2'],stderr=subprocess.STDOUT)
If you need to execute a command with a timeout, use the timeout argument:

try:out_bytes = subprocess.check_output([‘cmd','arg1','arg2'], timeout=5)except subprocess.TimeoutExpired as e:...
Normally, commands are executed without the assistance of an underlying shell (e.g.,sh, bash, etc.). Instead, the list of strings supplied are given to a low-level system com‐mand, such as os.execve(). If you want the command to be interpreted by a shell,supply it using a simple string and give the shell=True argument. This is sometimesuseful if you’re trying to get Python to execute a complicated shell command involvingpipes, I/O redirection, and other features. For example:

out_bytes = subprocess.check_output(‘grep python | wc > out', shell=True)

Be aware that executing commands under the shell is a potential security risk if argu‐ments are derived from user input. The shlex.quote() function can be used to properlyquote arguments for inclusion in shell commands in this case.

## 讨论

The check_output() function is the easiest way to execute an external command andget its output. However, if you need to perform more advanced communication with a

subprocess, such as sending it input, you’ll need to take a difference approach. For that,use the subprocess.Popen class directly. For example:

import subprocess

# Some text to sendtext = b'‘'hello worldthis is a testgoodbye‘'‘

# Launch a command with pipesp = subprocess.Popen([‘wc'],

> stdout = subprocess.PIPE,stdin = subprocess.PIPE)

# Send the data and get the outputstdout, stderr = p.communicate(text)

# To interpret as text, decodeout = stdout.decode(‘utf-8')err = stderr.decode(‘utf-8')

The subprocess module is not suitable for communicating with external commandsthat expect to interact with a proper TTY. For example, you can’t use it to automate tasksthat ask the user to enter a password (e.g., a ssh session). For that, you would need toturn to a third-party module, such as those based on the popular “expect” family of tools(e.g., pexpect or similar).

# 13.7 复制或者移动文件和目录

## 问题

You need to copy or move files and directories around, but you don’t want to do it bycalling out to shell commands.

## 解决方案

The shutil module has portable implementations of functions for copying files anddirectories. The usage is extremely straightforward. For example:

import shutil

# Copy src to dst. (cp src dst)shutil.copy(src, dst)

# Copy files, but preserve metadata (cp -p src dst)shutil.copy2(src, dst)

# Copy directory tree (cp -R src dst)shutil.copytree(src, dst)

# Move src to dst (mv src dst)shutil.move(src, dst)

The arguments to these functions are all strings supplying file or directory names. Theunderlying semantics try to emulate that of similar Unix commands, as shown in thecomments.By default, symbolic links are followed by these commands. For example, if the sourcefile is a symbolic link, then the destination file will be a copy of the file the link pointsto. If you want to copy the symbolic link instead, supply the follow_symlinks keywordargument like this:

shutil.copy2(src, dst, follow_symlinks=False)

If you want to preserve symbolic links in copied directories, do this:

shutil.copytree(src, dst, symlinks=True)

The copytree() optionally allows you to ignore certain files and directories during thecopy process. To do this, you supply an ignore function that takes a directory nameand filename listing as input, and returns a list of names to ignore as a result. For ex‐ample:

def ignore_pyc_files(dirname, filenames):return [name in filenames if name.endswith(‘.pyc')]
shutil.copytree(src, dst, ignore=ignore_pyc_files)

Since ignoring filename patterns is common, a utility function ignore_patterns() hasalready been provided to do it. For example:

shutil.copytree(src, dst, ignore=shutil.ignore*patterns(‘*~','_.pyc'))

## 讨论

Using shutil to copy files and directories is mostly straightforward. However, onecaution concerning file metadata is that functions such as copy2() only make a besteffort in preserving this data. Basic information, such as access times, creation times,and permissions, will always be preserved, but preservation of owners, ACLs, resourceforks, and other extended file metadata may or may not work depending on the un‐derlying operating system and the user’s own access permissions. You probably wouldn’twant to use a function like shutil.copytree() to perform system backups.When working with filenames, make sure you use the functions in os.path for thegreatest portability (especially if working with both Unix and Windows). For example:

```py
>>> filename = '/Users/guido/programs/spam.py'
>>> import os.path
>>> os.path.basename(filename)
'spam.py'
>>> os.path.dirname(filename)
'/Users/guido/programs'
>>> os.path.split(filename)
('/Users/guido/programs', 'spam.py')
>>> os.path.join('/new/dir', os.path.basename(filename))
'/new/dir/spam.py'
>>> os.path.expanduser('~/guido/programs/spam.py')
'/Users/guido/programs/spam.py'
>>>

```

One tricky bit about copying directories with copytree() is the handling of errors. Forexample, in the process of copying, the function might encounter broken symbolic links,files that can’t be accessed due to permission problems, and so on. To deal with this, allexceptions encountered are collected into a list and grouped into a single exception thatgets raised at the end of the operation. Here is how you would handle it:

try:shutil.copytree(src, dst)except shutil.Error as e:for src, dst, msg in e.args[0]:# src is source name# dst is destination name# msg is error message from exceptionprint(dst, src, msg)
If you supply the ignore_dangling_symlinks=True keyword argument, then copytree() will ignore dangling symlinks.The functions shown in this recipe are probably the most commonly used. However,shutil has many more operations related to copying data. The documentation is def‐initely worth a further look. See the Python documentation.

# 13.8 创建和解压压缩文件

## 问题

You need to create or unpack archives in common formats (e.g., .tar, .tgz, or .zip).

## 解决方案

The shutil module has two functions—make_archive() and unpack_archive()—thatdo exactly what you want. For example:

```py
>>> import shutil
>>> shutil.unpack_archive('Python-3.3.0.tgz')

>>> shutil.make_archive('py33','zip','Python-3.3.0')
'/Users/beazley/Downloads/py33.zip'
>>>

```

The second argument to make_archive() is the desired output format. To get a list ofsupported archive formats, use get_archive_formats(). For example:

```py
>>> shutil.get_archive_formats()
[('bztar', "bzip2'ed tar-file"), ('gztar', "gzip'ed tar-file"),
 ('tar', 'uncompressed tar file'), ('zip', 'ZIP file')]
>>>

```

## 讨论

Python has other library modules for dealing with the low-level details of various archiveformats (e.g., tarfile, zipfile, gzip, bz2, etc.). However, if all you’re trying to do ismake or extract an archive, there’s really no need to go so low level. You can just usethese high-level functions in shutil instead.The functions have a variety of additional options for logging, dryruns, file permissions,and so forth. Consult the shutil library documentation for further details.

# 13.9 通过文件名查找文件

## 问题

You need to write a script that involves finding files, like a file renaming script or a logarchiver utility, but you’d rather not have to call shell utilities from within your Pythonscript, or you want to provide specialized behavior not easily available by “shelling out.”

## 解决方案

To search for files, use the os.walk() function, supplying it with the top-level directory.Here is an example of a function that finds a specific filename and prints out the fullpath of all matches:

# !/usr/bin/env python3.3import os

def findfile(start, name):for relpath, dirs, files in os.walk(start):if name in files:full_path = os.path.join(start, relpath, name)print(os.path.normpath(os.path.abspath(full_path)))if **name** == ‘**main**':findfile(sys.argv[1], sys.argv[2])
Save this script as findfile.py and run it from the command line, feeding in the startingpoint and the name as positional arguments, like this:

bash % ./findfile.py . myfile.txt

## 讨论

The os.walk() method traverses the directory hierarchy for us, and for each directoryit enters, it returns a 3-tuple, containing the relative path to the directory it’s inspecting,a list containing all of the directory names in that directory, and a list of filenames inthat directory.For each tuple, you simply check if the target filename is in the files list. If it is,os.path.join() is used to put together a path. To avoid the possibility of weird lookingpaths like ././foo//bar, two additional functions are used to fix the result. The first isos.path.abspath(), which takes a path that might be relative and forms the absolutepath, and the second is os.path.normpath(), which will normalize the path, therebyresolving issues with double slashes, multiple references to the current directory, andso on.Although this script is pretty simple compared to the features of the find utility foundon UNIX platforms, it has the benefit of being cross-platform. Furthermore, a lot ofadditional functionality can be added in a portable manner without much more work.To illustrate, here is a function that prints out all of the files that have a recent modifi‐cation time:

# !/usr/bin/env python3.3

import osimport time

def modified_within(top, seconds):
now = time.time()for path, dirs, files in os.walk(top):

> for name in files:> fullpath = os.path.join(path, name)if os.path.exists(fullpath):
> 
> > > > mtime = os.path.getmtime(fullpath)if mtime > (now - seconds):
> > > 
> > > print(fullpath)

if **name** == ‘**main**':
import sysif len(sys.argv) != 3:

> print(‘Usage: {} dir seconds'.format(sys.argv[0]))raise SystemExit(1)

modified_within(sys.argv[1], float(sys.argv[2]))

It wouldn’t take long for you to build far more complex operations on top of this littlefunction using various features of the os, os.path, glob, and similar modules. See Rec‐ipes 5.11 and 5.13 for related recipes.

# 13.10 读取配置文件

## 问题

You want to read configuration files written in the common .ini configuration fileformat.

## 解决方案

The configparser module can be used to read configuration files. For example, supposeyou have this configuration file:

; config.ini; Sample configuration file

[installation]library=%(prefix)s/libinclude=%(prefix)s/includebin=%(prefix)s/binprefix=/usr/local

# Setting related to debug configuration[debug]log_errors=trueshow_warnings=False

[server]port: 8080nworkers: 32pid-file=/tmp/spam.pidroot=/www/rootsignature:

Here is an example of how to read it and extract values:

```py
>>> from configparser import ConfigParser
>>> cfg = ConfigParser()
>>> cfg.read('config.ini')
['config.ini']
>>> cfg.sections()
['installation', 'debug', 'server']
>>> cfg.get('installation','library')
'/usr/local/lib'
>>> cfg.getboolean('debug','log_errors')

```

True>>> cfg.getint(‘server','port')8080>>> cfg.getint(‘server','nworkers')32>>> print(cfg.get(‘server','signature'))

# Brought to you by the Python Cookbook

```py
>>>

```

If desired, you can also modify the configuration and write it back to a file using thecfg.write() method. For example:

```py
>>> cfg.set('server','port','9000')
>>> cfg.set('debug','log_errors','False')
>>> import sys
>>> cfg.write(sys.stdout)
[installation]
library = %(prefix)s/lib
include = %(prefix)s/include
bin = %(prefix)s/bin
prefix = /usr/local

```

[debug]log_errors = Falseshow_warnings = False

[server]port = 9000nworkers = 32pid-file = /tmp/spam.pidroot = /www/rootsignature =

## 讨论

Configuration files are well suited as a human-readable format for specifying configu‐ration data to your program. Within each config file, values are grouped into differentsections (e.g., “installation,” “debug,” and “server,” in the example). Each section thenspecifies values for various variables in that section.There are several notable differences between a config file and using a Python sourcefile for the same purpose. First, the syntax is much more permissive and “sloppy.” Forexample, both of these assignments are equivalent:

prefix=/usr/localprefix: /usr/local

The names used in a config file are also assumed to be case-insensitive. For example:

```py
>>> cfg.get('installation','PREFIX')
'/usr/local'
>>> cfg.get('installation','prefix')
'/usr/local'
>>>

```

When parsing values, methods such as getboolean() look for any reasonable value.For example, these are all equivalent:

> log_errors = truelog_errors = TRUElog_errors = Yeslog_errors = 1

Perhaps the most significant difference between a config file and Python code is that,unlike scripts, configuration files are not executed in a top-down manner. Instead, thefile is read in its entirety. If variable substitutions are made, they are done after the fact.For example, in this part of the config file, it doesn’t matter that the prefix variable isassigned after other variables that happen to use it:

> [installation]library=%(prefix)s/libinclude=%(prefix)s/includebin=%(prefix)s/binprefix=/usr/local

An easily overlooked feature of ConfigParser is that it can read multiple configurationfiles together and merge their results into a single configuration. For example, supposea user made their own configuration file that looked like this:

> > ; ~/.config.ini[installation]prefix=/Users/beazley/test
> 
> [debug]log_errors=False

This file can be merged with the previous configuration by reading it separately. Forexample:

```py
>>> # Previously read configuration
>>> cfg.get('installation', 'prefix')
'/usr/local'

>>> # Merge in user-specific configuration
>>> import os
>>> cfg.read(os.path.expanduser('~/.config.ini'))
['/Users/beazley/.config.ini']

>>> cfg.get('installation', 'prefix')
'/Users/beazley/test'
>>> cfg.get('installation', 'library')
'/Users/beazley/test/lib'
>>> cfg.getboolean('debug', 'log_errors')
False
>>>

```

Observe how the override of the prefix variable affects other related variables, such asthe setting of library. This works because variable interpolation is performed as lateas possible. You can see this by trying the following experiment:

```py
>>> cfg.get('installation','library')
'/Users/beazley/test/lib'
>>> cfg.set('installation','prefix','/tmp/dir')
>>> cfg.get('installation','library')
'/tmp/dir/lib'
>>>

```

Finally, it’s important to note that Python does not support the full range of features youmight find in an .ini file used by other programs (e.g., applications on Windows). Makesure you consult the configparser documentation for the finer details of the syntaxand supported features.

# 13.11 给简单脚本增加日志功能

## 问题

You want scripts and simple programs to write diagnostic information to log files.

## 解决方案

The easiest way to add logging to simple programs is to use the logging module. Forexample:

import logging

def main():

# Configure the logging systemlogging.basicConfig(

> filename='app.log',level=logging.ERROR

)

# Variables (to make the calls that follow work)hostname = ‘www.python.org'item = ‘spam'filename = ‘data.csv'mode = ‘r'

# Example logging calls (insert into your program)logging.critical(‘Host %s unknown', hostname)logging.error(“Couldn't find %r”, item)logging.warning(‘Feature is deprecated')logging.info(‘Opening file %r, mode=%r', filename, mode)logging.debug(‘Got here')

if **name** == ‘**main**':main()
The five logging calls (critical(), error(), warning(), info(), debug()) representdifferent severity levels in decreasing order. The level argument to basicConfig() isa filter. All messages issued at a level lower than this setting will be ignored.The argument to each logging operation is a message string followed by zero or morearguments. When making the final log message, the % operator is used to format themessage string using the supplied arguments.If you run this program, the contents of the file app.log will be as follows:

> CRITICAL:root:Host www.python.org unknownERROR:root:Could not find ‘spam'

If you want to change the output or level of output, you can change the parameters tothe basicConfig() call. For example:

logging.basicConfig(filename='app.log',level=logging.WARNING,format='%(levelname)s:%(asctime)s:%(message)s')
As a result, the output changes to the following:

> CRITICAL:2012-11-20 12:27:13,595:Host www.python.org unknownERROR:2012-11-20 12:27:13,595:Could not find ‘spam'WARNING:2012-11-20 12:27:13,595:Feature is deprecated

As shown, the logging configuration is hardcoded directly into the program. If you wantto configure it from a configuration file, change the basicConfig() call to the following:

import loggingimport logging.config

def main():# Configure the logging systemlogging.config.fileConfig(‘logconfig.ini')...
Now make a configuration file logconfig.ini that looks like this:

> > [loggers]keys=root
> 
> [handlers]keys=defaultHandler
> 
> [formatters]keys=defaultFormatter
> 
> [logger_root]level=INFOhandlers=defaultHandlerqualname=root
> 
> [handler_defaultHandler]class=FileHandlerformatter=defaultFormatterargs=(‘app.log', ‘a')
> 
> [formatter_defaultFormatter]format=%(levelname)s:%(name)s:%(message)s

If you want to make changes to the configuration, you can simply edit the logcon‐fig.ini file as appropriate.

## 讨论

Ignoring for the moment that there are about a million advanced configuration optionsfor the logging module, this solution is quite sufficient for simple programs and scripts.Simply make sure that you execute the basicConfig() call prior to making any loggingcalls, and your program will generate logging output.If you want the logging messages to route to standard error instead of a file, don’t supplyany filename information to basicConfig(). For example, simply do this:

logging.basicConfig(level=logging.INFO)

One subtle aspect of basicConfig() is that it can only be called once in your program.If you later need to change the configuration of the logging module, you need to obtainthe root logger and make changes to it directly. For example:

logging.getLogger().level = logging.DEBUG

It must be emphasized that this recipe only shows a basic use of the logging module.There are significantly more advanced customizations that can be made. An excellentresource for such customization is the “Logging Cookbook”.

# 13.12 给内库增加日志功能

## 问题

You would like to add a logging capability to a library, but don’t want it to interfere withprograms that don’t use logging.

## 解决方案

For libraries that want to perform logging, you should create a dedicated logger object,and initially configure it as follows:

# somelib.py

import logginglog = logging.getLogger(**name**)log.addHandler(logging.NullHandler())

# Example function (for testing)def func():

> log.critical(‘A Critical Error!')log.debug(‘A debug message')

With this configuration, no logging will occur by default. For example:

```py
>>> import somelib
>>> somelib.func()
>>>

```

However, if the logging system gets configured, log messages will start to appear. Forexample:

```py
>>> import logging
>>> logging.basicConfig()
>>> somelib.func()
CRITICAL:somelib:A Critical Error!
>>>

```

## 讨论

Libraries present a special problem for logging, since information about the environ‐ment in which they are used isn’t known. As a general rule, you should never writelibrary code that tries to configure the logging system on its own or which makes as‐sumptions about an already existing logging configuration. Thus, you need to take greatcare to provide isolation.The call to getLogger(**name**) creates a logger module that has the same name asthe calling module. Since all modules are unique, this creates a dedicated logger that islikely to be separate from other loggers.

The log.addHandler(logging.NullHandler()) operation attaches a null handler tothe just created logger object. A null handler ignores all logging messages by default.Thus, if the library is used and logging is never configured, no messages or warningswill appear.One subtle feature of this recipe is that the logging of individual libraries can be inde‐pendently configured, regardless of other logging settings. For example, consider thefollowing code:

```py
>>> import logging
>>> logging.basicConfig(level=logging.ERROR)
>>> import somelib
>>> somelib.func()
CRITICAL:somelib:A Critical Error!

>>> # Change the logging level for 'somelib' only
>>> logging.getLogger('somelib').level=logging.DEBUG
>>> somelib.func()
CRITICAL:somelib:A Critical Error!
DEBUG:somelib:A debug message
>>>

```

Here, the root logger has been configured to only output messages at the ERROR level orhigher. However, the level of the logger for somelib has been separately configured tooutput debugging messages. That setting takes precedence over the global setting.The ability to change the logging settings for a single module like this can be a usefuldebugging tool, since you don’t have to change any of the global logging settings—simplychange the level for the one module where you want more output.The “Logging HOWTO” has more information about configuring the logging moduleand other useful tips.

# 13.13 记录程序执行的时间

## 问题

You want to be able to record the time it takes to perform various tasks.

## 解决方案

The time module contains various functions for performing timing-related functions.However, it’s often useful to put a higher-level interface on them that mimics a stopwatch. For example:

import time

class Timer:def **init**(self, func=time.perf_counter):self.elapsed = 0.0self._func = funcself._start = Nonedef start(self):if self._start is not None:raise RuntimeError(‘Already started')
self._start = self._func()

def stop(self):if self._start is None:raise RuntimeError(‘Not started')
end = self._func()self.elapsed += end - self._startself._start = None

def reset(self):self.elapsed = 0.0
@propertydef running(self):

> return self._start is not None

def **enter**(self):self.start()return selfdef **exit**(self, *args):self.stop()
This class defines a timer that can be started, stopped, and reset as needed by the user.It keeps track of the total elapsed time in the elapsed attribute. Here is an example thatshows how it can be used:

def countdown(n):while n > 0:n -= 1

# Use 1: Explicit start/stopt = Timer()t.start()countdown(1000000)t.stop()print(t.elapsed)

# Use 2: As a context managerwith t:

> countdown(1000000)

print(t.elapsed)

with Timer() as t2:countdown(1000000)
print(t2.elapsed)

## 讨论

This recipe provides a simple yet very useful class for making timing measurements andtracking elapsed time. It’s also a nice illustration of how to support the context-management protocol and the with statement.One issue in making timing measurements concerns the underlying time function usedto do it. As a general rule, the accuracy of timing measurements made with functionssuch as time.time() or time.clock() varies according to the operating system. Incontrast, the time.perf_counter() function always uses the highest-resolution timeravailable on the system.As shown, the time recorded by the Timer class is made according to wall-clock time,and includes all time spent sleeping. If you only want the amount of CPU time used bythe process, use time.process_time() instead. For example:

t = Timer(time.process_time)with t:

> countdown(1000000)

print(t.elapsed)

Both the time.perf_counter() and time.process_time() return a “time” in fractionalseconds. However, the actual value of the time doesn’t have any particular meaning. Tomake sense of the results, you have to call the functions twice and compute a timedifference.More examples of timing and profiling are given in Recipe 14.13.

# 13.14 限制内存和 CPU 的使用量

## 问题

You want to place some limits on the memory or CPU use of a program running onUnix system.

## 解决方案

The resource module can be used to perform both tasks. For example, to restrict CPUtime, do the following:

import signalimport resourceimport os

def time_exceeded(signo, frame):print(“Time's up!”)raise SystemExit(1)def set_max_runtime(seconds):# Install the signal handler and set a resource limitsoft, hard = resource.getrlimit(resource.RLIMIT_CPU)resource.setrlimit(resource.RLIMIT_CPU, (seconds, hard))signal.signal(signal.SIGXCPU, time_exceeded)if **name** == ‘**main**':
set_max_runtime(15)while True:

> pass

When this runs, the SIGXCPU signal is generated when the time expires. The programcan then clean up and exit.To restrict memory use, put a limit on the total address space in use. For example:

import resource

def limit_memory(maxsize):soft, hard = resource.getrlimit(resource.RLIMIT_AS)resource.setrlimit(resource.RLIMIT_AS, (maxsize, hard))
With a memory limit in place, programs will start generating MemoryError exceptionswhen no more memory is available.

## 讨论

In this recipe, the setrlimit() function is used to set a soft and hard limit on a particularresource. The soft limit is a value upon which the operating system will typically restrictor notify the process via a signal. The hard limit represents an upper bound on the valuesthat may be used for the soft limit. Typically, this is controlled by a system-wide pa‐rameter set by the system administrator. Although the hard limit can be lowered, it cannever be raised by user processes (even if the process lowered itself).The setrlimit() function can additionally be used to set limits on things such as thenumber of child processes, number of open files, and similar system resources. Consultthe documentation for the resource module for further details.Be aware that this recipe only works on Unix systems, and that it might not work on allof them. For example, when tested, it works on Linux but not on OS X.

# 13.15 启动一个 WEB 浏览器

## 问题

You want to launch a browser from a script and have it point to some URL that youspecify.

## 解决方案

The webbrowser module can be used to launch a browser in a platform-independentmanner. For example:

```py
>>> import webbrowser
>>> webbrowser.open('http://www.python.org')
True
>>>

```

This opens the requested page using the default browser. If you want a bit more controlover how the page gets opened, you can use one of the following functions:

```py
>>> # Open the page in a new browser window
>>> webbrowser.open_new('http://www.python.org')
True
>>>

>>> # Open the page in a new browser tab
>>> webbrowser.open_new_tab('http://www.python.org')
True
>>>

```

These will try to open the page in a new browser window or tab, if possible and supportedby the browser.If you want to open a page in a specific browser, you can use the webbrowser.get()function to specify a particular browser. For example:

```py
>>> c = webbrowser.get('firefox')
>>> c.open('http://www.python.org')
True
>>> c.open_new_tab('http://docs.python.org')
True
>>>

```

A full list of supported browser names can be found in the Python documentation.

## 讨论

Being able to easily launch a browser can be a useful operation in many scripts. Forexample, maybe a script performs some kind of deployment to a server and you’d liketo have it quickly launch a browser so you can verify that it’s working. Or maybe aprogram writes data out in the form of HTML pages and you’d just like to fire up abrowser to see the result. Either way, the webbrowser module is a simple solution.