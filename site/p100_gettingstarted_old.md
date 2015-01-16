---
layout: page
title: Getting Started (OBSOLETE)
permalink: gettingstarted_obsolete.html
---

# Getting Started with fips

### You need:

* Python 2.7.x
* CMake (at least version 2.8.11)
* a working C/C++ development environment for your OS (e.g. Xcode on OSX,
make/gcc on Linux, Visual Studio on Windows)

### TL;DR:

To fetch, build and run a simple hello-world project:

>NOTE: on Windows, run 'fips' instead of './fips'!

{% highlight bash %}
> git clone https://github.com/floooh/fips-hello-world.git
> cd fips-hello-world
> ./fips build
...
> ./fips run hello
=== run 'hello' (config: osx-xcode-debug, project: fips-hello-world):
Hello World!
...
> _
{% endhighlight %}

**What happened**:

1. each 'fipsified' project has a small python script in its root directory called
_fips_ which invokes one of several actions
2. if fips is installed, the 'proxy' fips script calls the actual fips main
script to perform an action (in this case the _build_ action)
3. _./fips build_ performs a complete build of project in the host-platform's
default build configuration, this means:
    - pull in all external dependencies of the project through git (in this case:
      _fips-hello-dep1_ and _fips-hello-dep2_)
    - run cmake to generate build files
    - compile and link the build targets
4. finally, _./fips run hello_ runs the small hello-world program that was built

Such a simple default build is nothing special of course, so let's have 
a look at what else fips can do:

### Showing help:

If you don't know what to do next, run _fips help_ to get an overview of 
available commands:

{% highlight bash %}
> ./fips help
fips: the high-level, multi-platform build system wrapper
v0.0.1
https://www.github.com/floooh/fips

fips build
fips build [config]
   perform a build for current or named config
...
fips unset config
fips unset target
    unset currently active config or make-target

> _
{% endhighlight %}

You can also show help for a specific command:

{% highlight bash %}
> ./fips help diag
fips diag
fips diag all
fips diag fips
fips diag tools
fips diag configs
fips diag imports
    run diagnostics and check for errors
{% endhighlight %}

### Troubleshooting

If something goes wrong, _./fips clean all_ and _./fips diag_ are useful
commands.

_./fips clean all_ deletes all files generated by fips and _./fips diag_ runs
a few diagnostics.

First let's check whether all required tools are installed, the actual
output depends on your host operating system:

{% highlight bash %}
> cd fips-hello-world
> ./fips diag tools
=== tools:
git:	found
cmake:	found
ccmake:	found
make:	found
ninja:	OPTIONAL, NOT FOUND (required for building '*-ninja-*' configs)
xcodebuild:	found
ant:	found
java:	found
node:	found
{% endhighlight %}

In this case, all tools are found except the _ninja_ build tool, which is 
optional and only needed for some build configurations. Usually you want
to make sure that all missing tools are installed and in the path.

Run _./fips diag fips_ to check whether fips itself is uptodate:

{% highlight bash %}
> ./fips diag fips
=== fips:
  uptodate
{% endhighlight %}

If fips is not uptodate, do a _git pull_ from inside the fips directory to 
update to the latest version.

### IDE support

On OSX and Windows, there's a handy shortcut to open the project in Xcode
or Visual Studio:

{% highlight bash %}
# first generate project files via cmake:
> ./fips gen
...
# then open the project in Xcode or Visual Studio:
> ./fips open
...
> _
{% endhighlight %}

### Build configurations

Managing different build configurations is fips' core feature. A build
configuration tells cmake how and where to create the intermediate 
build files.

Run _./fips list configs_ to see a list of all existing build configurations:

{% highlight bash %}
> ./fips list configs
=== configs:
from /Users/floh/fips-workspace/fips/configs:
  android-make-debug
  android-make-release
  android-ninja-debug
...
  osx-xcode-debug
  osx-xcode-release
  osx-xcode-unittest
...
{% endhighlight %}

As you can see there is lots of them, for different target platforms 
(like Android, OSX, Linux, Emscripten, ...), build tools (make, ninja,
Xcode, Visual Studio, ...) and build types (Debug, Release, ...).

Many fips commands need to know which build configuration to use and
accept an explicit build config name, for instance:

{% highlight bash %}
# on OSX, build with Xcode in Debug mode:
> ./fips build osx-xcode-debug
...
# ...or in Release mode with make:
> ./fips build osx-make-release
...
{% endhighlight %}

To save typing it is possible to omit the config name, in this case
an _active config_ will be used. To see which config is the current
_active config_ run _./fips list settings_:

{% highlight bash %}
> ./fips list settings
=== settings:
  config: osx-xcode-debug (default value)
  target: None (default value)
{% endhighlight %}

Each host platform has a default config (on OSX this is _osx-xcode-debug_).

To change the _active config_ use _./fips set config [config]_:

{% highlight bash %}
> ./fips set config osx-make-release
'config' set to 'osx-make-release' in project 'fips-hello-world'
> ./fips list settings
=== settings:
  config: osx-make-release
  target: None (default value)
{% endhighlight %}

To revert back to the default config, use _./fips unset config_:

{% highlight bash %}
> ./fips unset config
'config' unset in project 'fips-hello-world'
{% endhighlight %}

### Working with Build Targets

You can get a list of all build targets in a project, but only
after build files have been generated by cmake:

{% highlight bash %}
> ./fips gen
...
> ./fips list targets
=== targets:
  config: osx-xcode-debug
  lib:
  module:
    dep1
    dep2
  app:
    hello
> _
{% endhighlight %}

You can build a single target with 'fips make':

{% highlight bash %}
> ./fips build hello
...
> _
{% endhighlight %}

Finally it is possible to run a built app target:

{% highlight bash %}
> ./fips run hello
=== run 'hello' (config: osx-xcode-debug, project: fips-hello-world):
Hello World!
Hello from dep1
Hello from dep2!
Imported string define: Bla
Imported int define: 1
> _
{% endhighlight %}

What happens exactly on 'fips run' depends on the target platform. For instance
when building for a web platform like emscripten or PNaCl, a local web
server and browser will be started.
