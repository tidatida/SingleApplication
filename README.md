SingleApplication
=================

This is a replacement of the QSingleApplication for `Qt5`.

Keeps the Primary Instance of your Application and kills each subsequent
instances. It can (if enabled) spawn secondary (non-related to the primary)
instances and can send data to the primary instance from secondary instances.

Usage
-----

The `SingleApplication` class inherits from whatever `Q[Core|Gui]Application`
class you specify via the `QAPPLICATION_CLASS` macro (`QCoreApplication` is the
default). Further usage is similar to the use of the `Q[Core|Gui]Application`
classes.

The library sets up a `QLocalServer` and a `QSharedMemory` block. The first
instance of your Application is your Primary Instance. It would check if the
shared memory block exists and if not it will start a `QLocalServer` and then
listen for connections. Each subsequent instance of your application would
check if the shared memory block exists and if it does, it will connect to the
QLocalServer to notify it that a new instance had been started, after which it
would terminate with status code `0`. The Primary Instance, `SingleApplication`
would emit the `instanceStarted()` signal upon detecting that a new instance
had been started.

The library uses `stdlib` to terminate the program with the `exit()` function.

You can use the library as if you use any other `QCoreApplication` class:

```cpp
#include <QApplication>
#include <SingleApplication.h>

int main( int argc, char* argv[] )
{
    SingleApplication app( argc, argv );

    return app.exec();
}
```

To include the library files I would recommend that you add it as a git
submodule to your project and include it's contents with a `.pri` file. Here is
how:

```bash
git submodule add git@github.com:itay-grudev/SingleApplication.git singleapplication
```

Then include the `singleapplication.pri` file in your `.pro` project file. Also
don't forget to specify which `QCoreApplication` class your app is using if it
is not `QCoreApplication`.

```qmake
include(singleapplication/singleapplication.pri)
DEFINES += QAPPLICATION_CLASS=QApplication
```

The `Instance Started` signal
------------------------

The SingleApplication class implements a `instanceStarted()` signal. You can
bind to that signal to raise your application's window when a new instance had
been started.

```cpp
// window is a QWindow instance
QObject::connect( &app, &SingleApplication::instanceStarted, window, &QWindow::raise );
```

Using `SingleApplication::instance()` is a neat way to get the
`SingleApplication` instance for binding to it's signals anywhere in your
program.

Secondary Instances
-------------------

If you want to be able to launch additional Secondary Instances (not related to
your Primary Instance) you have to enable that with the third parameter of the
`SingleApplication` constructor. The default is `false` meaning no Secondary
Instances. Here is an example of how you would start a Secondary Instance send
a message with the command line arguments to the primary instance and then shut
down.

```cpp
int main(int argc, char *argv[])
{
    SingleApplication app( argc, argv, true );

    if( app.isSecondary() ) {
        app.sendMessage(  app.arguments().join(' ')).toUtf8() );
        app.exit( 0 );
    }

    return app.exec();
}
```

___Note:__ A secondary instance won't cause the emission of the
`instanceStarted()` signal.

You can check whether your instance is a primary or secondary with the following
methods:

```cpp
app.isPrimary();
// or
app.isSecondary();
```

__*Note:*__ If your Primary Instance is terminated a newly launched instance
will replace the Primary one even if the Secondary flag has been set.

Versioning
----------

The current library versions is `3.0a`.

Each major version introduces either very significant changes or is not
backwards compatible with the previous version. Minor versions only add
additional features, bug fixes or performance improvements and are backwards
compatible with the previous release. See `CHANGELOG.md` for more details.

Implementation
--------------

The library is implemented with a QSharedMemory block which is thread safe and
guarantees a race condition will not occur. It also uses a QLocalSocket to
notify the main process that a new instance had been spawned and thus invoke the
`instanceStarted()` signal.

To handle an issue on `*nix` systems, where the operating system owns the shared
memory block and if the program crashes the memory remains untouched, the
library binds to the following signals and closes the program with
`error code = 128 + signum` where signum is the number representation of the
signal listed below. Handling the signal is required in order to safely delete
the `QSharedMemory` block. Each of these signals are potentially lethal and will
results in process termination.

*   `SIGHUP` - `1`, Hangup.
*   `SIGINT` - `2`, Terminal interrupt signal
*   `SIGQUIT` - `3`, Terminal quit signal.
*   `SIGILL` - `4`, Illegal instruction.
*   `SIGABRT` - `6`, Process abort signal.
*   `SIGBUS` - `7`, Access to an undefined portion of a memory object.
*   `SIGFPE` - `8`, Erroneous arithmetic operation (such as division by zero).
*   `SIGSEGV` - `11`, Invalid memory reference.
*   `SIGSYS` - `12`, Bad system call.
*   `SIGPIPE` - `13`, Write on a pipe with no one to read it.
*   `SIGALRM` - `14`, Alarm clock.
*   `SIGTERM` - `15`, Termination signal.
*   `SIGXCPU` - `24`, CPU time limit exceeded.
*   `SIGXFSZ` - `25`, File size limit exceeded.


License
-------
This library and it's supporting documentation are released under
`The MIT License (MIT)` with exception to some of the examples distributed under
the terms of the BSD license.
