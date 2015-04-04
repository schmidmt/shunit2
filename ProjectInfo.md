# Project Info #


## Overview ##
[shUnit2](http://sourceforge.net/projects/shunit2) is a [xUnit](http://en.wikipedia.org/wiki/XUnit) unit test framework for Bourne based shell scripts, and it is designed to work in a similar manner to [JUnit](http://www.junit.org/), [PyUnit](http://pyunit.sourceforge.net/), etc. If you have ever had the desire to write a unit test for a shell script, shUnit2 can do the job.

shUnit2 was originally developed to provide a consistent testing solution for [log4sh](http://log4sh.sourceforge.net/), a shell based logging framework similar to [log4j](http://logging.apache.org/log4j/). During the development of that product, I repeatedly ran into the problem of having things work just fine under one shell (`/bin/bash` on Linux to be specific), and then not working under another shell (`/bin/sh` on Solaris). Although I had some simple tests that I ran, I finally decided to develop a more robust unit test framework after releasing multiple brown-bag releases.

Rather than reinvent the wheel again, I chose to base the shUnit2 framework as much as possible on JUnit. I wanted something that was both incredibly easy to use, but that was as well incredibly powerful. I knew that the code would be built only for log4sh at the start, but hoped that someday I could release the code as a separate project. Now that the code is stable and mature enough, I have done just that.

You might ask why I didn't just use the already existing [ShUnit](http://shunit.sourceforge.net/) product. I took a look at it, but wasn't really happy with how it worked or how little it reflected the way JUnit and other unit test frameworks function. As such, I figured that it was probably better just to write my own.

shUnit2 has been developed under the Bourne Again Shell (`/bin/bash`) on Linux, but _great care_ has been taken to make sure it works under the other Unix shells to insure maximum compatibility. If you like what you see, or have any suggestions on improvements, please feel free to drop me an email.

_Supported Operating Systems_
  * [Cygwin](http://www.cygwin.com/) (under Microsoft Windows XP)
  * [FreeBSD](http://www.freebsd.org/) (user supported)
  * Linux ([Ubuntu](http://www.ubuntu.com/))
  * [Mac OS X](http://www.apple.com/macosx/)
  * [Solaris](http://www.sun.com/software/solaris/) 8-10 and [OpenSolaris](http://www.opensolaris.org/)

_Tested Shells_
  * Bourne Shell (**`sh`**)
  * [BASH](http://www.gnu.org/software/bash/) – GNU Bourne Again SHell (**`bash`**)
  * [DASH](http://gondor.apana.org.au/~herbert/dash/) (**`dash`**)
  * [Korn Shell](http://www.kornshell.com/) (**`ksh`**)
  * [pdksh](http://web.cs.mun.ca/~michael/pdksh/) – the Public Domain Korn Shell (**`pdksh`**)
  * [Zsh](http://www.zsh.org/) - (**`zsh`**)

## Downloading ##
There are two current release series of shUnit2 available. The 2.0 series is the stable release, and it contains only bug fixes. The 2.1 series is meant for new features and is still under active development. The 2.1 series of shUnit2 has its own set of unit tests, and should be considered fine for daily use.

If you would like to verify the GPG signatures of the releases, please take a look at my [GnuPG Info](http://forestent.com/wiki/Info:GnuPG) page.

### 2.0.x - Stable Releases ###
| **Release** | **Date** |
|:------------|:---------|
| [2.0.3](http://downloads.sourceforge.net/shunit2/shunit2-2.0.3.tgz) | Thu Jul 12 2007 |
| [2.0.2](http://downloads.sourceforge.net/shunit2/shunit2-2.0.2.tgz) | Mon Apr 23 2007 |
| [2.0.1](http://downloads.sourceforge.net/shunit2/shunit2-2.0.1.tgz) | Wed Feb 21 2007 |
| [2.0.0](http://downloads.sourceforge.net/shunit2/shunit2-2.0.0.tgz) | Mon Feb 19 2007 |

### 2.1.x - Development Releases ###
| **Release** | **Date** | **Known Issues** |
|:------------|:---------|:-----------------|
| [2.1.6](http://shunit2.googlecode.com/files/shunit2-2.1.6.tgz) | Sun May 1 2011 |  |
| [2.1.5](http://shunit2.googlecode.com/files/shunit2-2.1.5.tgz) | Wed Oct 29 2008 |  |
| [2.1.4](http://shunit2.googlecode.com/files/shunit2-2.1.4.tgz) | Fri Jul 11 2008 |  |
| [2.1.3](http://downloads.sourceforge.net/shunit2/shunit2-2.1.3.tgz) | Sun May 10 2008 | Doesn't work with **`zsh`** versions prior to 4.0. |
| [2.1.2](http://downloads.sourceforge.net/shunit2/shunit2-2.1.2.tgz) | Mon Dec 31 2007 |  |
| [2.1.1](http://downloads.sourceforge.net/shunit2/shunit2-2.1.1.tgz) | Fri Jul 13 2007 |  |
| [2.1.0](http://downloads.sourceforge.net/shunit2/shunit2-2.1.0.tgz) | Mon Apr 23 2007 |  |

## Support ##

### Documentation ###
HTML documentation is also included in the `doc` directory of each distribution. Links to the latest version in Subversion are provided below.

[2.0](http://shunit2.googlecode.com/svn/trunk/source/2.0/doc/shunit2.html) ·
[2.1](http://shunit2.googlecode.com/svn/trunk/source/2.1/doc/shunit2.html)

You might also be interested in our GeneralFaq page.

### Examples ###
One case study for writing a function to convert a relative path to an absolute path ([HOWTO\_TestAbsolutePathFn](http://code.google.com/p/shunit2/wiki/HOWTO_TestAbsolutePathFn)).

Starting with the 2.1.x series, additional examples are included with the source code in the `examples` directory.
  * `mkdir_test.sh` – demonstration of testing an existing Unix command (e.g. an already written shell script)

### Mailing Lists ###
A [shunit2-users@googlegroups.com](mailto:shunit2-users@googlegroups.com) mailing list now exists. Everyone is free to [view the discussions](http://groups.google.com/group/shunit2-users/topics) or to subscribe.

## Miscellaneous ##

### Licensing ###
shUnit2 is licensed under the GNU [Lesser General Public License](http://www.gnu.org/licenses/lgpl.html) (LGPL). The contents and copyright of this site and all provided source code are owned by [Kate Ward](http://www.linkedin.com/pub/0/9b9/111).

### Thanks ###
A list of contributors is provided with each release (found as `doc/contributors.txt`). Many thanks go out to each of these individuals for finding bugs and/or providing patches.

Hosting resources are gracefully provided by [Google Code](http://code.google.com/).