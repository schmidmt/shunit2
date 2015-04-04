

This example is based upon a real battle I encountered trying to get a particular regular expression to work. I'm not the best when it comes to writing new regexes off the top of my head, and so I end up doing what one might consider a brute-force attack in getting my regexes to work. You might wonder why I don't just learn them, but I must admit that I don't write them often enough for the nuances to stick in my brain. I'm sure others have a better way of doing what you see here (there always is), but I didn't have one at the time. In any case, maybe some of you will sympathize with my battle, and maybe this will give somebody else some ideas for testing their own code.

# First Go #
_Jan 2007_

## Problem ##
I once had a tiny shell function I needed to write that took a relative path (e.g. `./bin`) and turned it into the absolute path (e.g. `/home/kward/bin`).

## Solution ##
Here was my first go. I wrapped the function in a simple shell script for easier testing and got the following:
```
#! /bin/sh

myBase="`basename $0`"
myDir="`dirname $0`"

relToAbsPath()
{
  path=$1

  dir=`dirname "${path}"`
  base=`basename "${path}"`
  echo "`( cd \"${dir}\" && pwd )`/${base}"
}

myDir=`relToAbsPath "${myDir}"`
echo "myDir='${myDir}' myBase='${myBase}'"
```

I saved the script as `${HOME}/bin/reltoabs` and called it from my home directory like this:
```
[kward@laptop]~$ ./bin/reltoabs
myDir='/home/kward/bin' myBase='reltoabs'
```

Perfect! It did _exactly_ what I wanted it to do, so I stuck the function in another script and started using it.

# Second Go #
_Jan 2007_

## Problem ##
After time, I found a case where my function wasn't doing quite what I wanted. Let's say I had some non-existant relative path `abc/def/ghi` that I wanted to turn into an absolute path. If you noticed, the first version of the function used the **cd** command to change to the **dirname** of the relative directory, and used that information to determine the absolute path. In this case, I had a directory that didn't exist, but I still wanted the function to work.

Using the same test wrapper script, I changed the `myDir=` line to ``myDir="`reltoabs 'abc/def/ghi'`"`` and ran the script just as I did in the first go. Here is what my function gave as output:
```
$ bin/reltoabs.sh 
cd: 15: can't cd to abc/def
myDir='/ghi' myBase='reltoabs.sh'
```

As you can see, it didn't work. I spent the next half-hour fighting with various **sed** regexes on the command-line trying to get things to work. Just as I would get something to work for one case, another case wouldn't work. (I had by this time expanded my expected functionality to several different patterns that my function should work properly for, and testing on the command-line was becoming _very_ tedious.)

After enough time, I decided that I'd better wrap this in some sort of bigger testing script so that I could test the various multiple cases. I thought through it a bit and eventually settled on using shUnit2 as it just made the job very easy.

## Solution ##
Below are two files. The first is a shell script snippit that gets included (a.k.a. "sourced" in shell speak) into the shUnit2 unit test, and the second file is the unit test itself.

By the time I was to this stage, I had already included the `reltoabs()` function into a more general `shlib_base.inc` library and renamed it to `shlib_relToAbsPath()`, so the source provided looks a bit different than before. You will also notice that I keep a copy of `shunit2` in my HOME directory referenced as `${HOME}/lib/sh/shunit2`. The second file expects this, so if you want to duplicate these results, you will need to fix that.

`shlib_base.inc`
```
shlib_relToAbsPath()
{
  _shlib_path=$1

  # deal with paths that start with /
  echo "${_shlib_path}" |grep '^/' >/dev/null 2>&1
  if [ $? -ne 0 ]; then
    _shlib_pwd=`pwd`
    _shlib_path="${_shlib_pwd}/${_shlib_path}"
    unset _shlib_pwd
  fi

  # clean up the path.
  echo "${_shlib_path}" |sed -r 's/[^/]*\/+\.{2}\/*//g;s/\/\.\//\//;s/\/+$//'

  unset _shlib_path
}
```

`shlib_base_test`
```
#! /bin/sh

#-----------------------------------------------------------------------------
# suite tests
#

testRelToAbsPath()
{
  parent=`dirname ${PWD}`
  exec 9<&0 <<EOF
abc                ${PWD}/abc
abc/def            ${PWD}/abc/def
abc/def/ghi        ${PWD}/abc/def/ghi
abc/./def          ${PWD}/abc/def
abc/../def         ${PWD}/def
abc/../def/../ghi  ${PWD}/ghi
/abc               /abc
/abc/def           /abc/def
/abc/def/ghi       /abc/def/ghi
/abc/../def        /def
/abc/../def/../ghi /ghi
./abc              ${PWD}/abc
../abc             ${parent}/abc
../abc/def         ${parent}/abc/def
EOF
  while read relPath absPath; do
    echo "${relPath}" |grep "^#" >/dev/null 2>&1 && continue
    newPath=`shlib_relToAbsPath "${relPath}"`
    assertSame \
        "'${relPath}' -> '${newPath}' != '${absPath}'" \
        "${newPath}" "${absPath}"
  done
  exec 0<&9 9<&-
}

#-----------------------------------------------------------------------------
# suite functions
#

oneTimeSetUp()
{
  # load shlib
  . ./shlib_base.inc
}

# load and run shUnit2
. ${HOME}/lib/sh/shunit2
```

After a small chunk of time (something like 15min) I had a working regex. The unit test greatly improved my testing rate. Running the eventual unit test produces the following output:
```
$ ./shlib_base_test 
#
# Performing tests
#
testRelToAbsPath

#
# Test report
#
tests passed: 14
tests failed: 0
tests total:  14
success rate: 100%
```

Perfect! It did _exactly_ what I wanted it to do, so I now had a working function that was usable.

_Based on previous experience, I did not delete the unit test. I figured that if ever the occasion presented itself where my function failed, I would still have the unit test available to work from._

# Third Go #
_Feb 2007_

## Problem ##
The majority of shell scripts that I write are for Linux. There is however the rare occasion where I must write something for Solaris. When I do, I want that all the scripts that I have written be functional under Solaris as well as Linux. I have the added difficulty that I sometimes run on very old Solaris releases (2.5.1 and 2.6), and I want my scripts to function there as well. This presents a problem for the average script writer as they never leave Linux these days, but for me I cope just fine.

Well, as you can imagine, the above `shlib_relToAbsPath()` function did not work under Solaris. I'm not perfect, and I'd forgotten that the nice little `-r` command-line flag to **sed** was not present on Solaris. Argh! Oh, but wait! Learning from previous pain, I had left my **shlib\_base\_test** unit test in the same directory as my `shlib_base.inc` library, so I already had a way to begin my debugging.

## Solution ##
To start with, I ran my unit test to see what kind of output I would get. It failed miserably.

Well, needless to say, I removed the offending `-r` command-line flag from **sed**, and got to work. The process was much easier this time as I already had a working unit test, so all I had to do was tweak my regex until it worked. It didn't take long, maybe 5 minutes, before I had a working test. This time, I tested the regex in both Linux _and_ Solaris, and as such I was confident that I had a working function for both platforms.

Final `shlib_base.inc`
```
shlib_relToAbsPath()
{
  _shlib_path=$1

  # deal with paths that start with /
  echo "${_shlib_path}" |grep '^/' >/dev/null 2>&1
  if [ $? -ne 0 ]; then
    _shlib_pwd=`pwd`
    _shlib_path="${_shlib_pwd}/${_shlib_path}"
    unset _shlib_pwd
  fi

  # clean up the path. if all seds supported true regular expressions, then
  # this is what it would be:
  # echo "${_shlib_path}" |sed -r 's/[^/]*\/+\.{2}\/*//g;s/\/\.\//\//;s/\/+$//'
  echo "${_shlib_path}" |sed 's/[^/]*\/*\.\.\/*//g;s/\/\.\//\//'

  unset _shlib_path
}
```

_The `shlib_base_test` unit test remained unchanged, so it is not re-listed._

# Fourth Go #
_**Update:** Oct 2008_

Believe it or not, more than a year and a half after I wrote the original document, I found more bugs! I bet you're not surprised. Basically, including multiple parent references caused problems. Here is the latest version of my code.

`shlib_base.inc`
```
shlib_relToAbsPath()
{
  shlib_path_=$1

  # prepend current directory to relative paths
  echo "${shlib_path_}" |grep '^/' >/dev/null 2>&1 \
      || shlib_path_="`pwd`/${shlib_path_}"

  # clean up the path. if all seds supported true regular expressions, then
  # this is what it would be:
  shlib_old_=${shlib_path_}
  while true; do
    shlib_new_=`echo "${shlib_old_}" |sed 's/[^/]*\/\.\.\/*//g;s/\/\.\//\//'`
    [ "${shlib_old_}" = "${shlib_new_}" ] && break
    shlib_old_=${shlib_new_}
  done
  echo "${shlib_new_}"

  unset shlib_path_ shlib_old_ shlib_new_
}
```

`shlib_base_test`
```
testRelToAbsPath()
{
  parent=`dirname ${PWD}`

  # save stdin and redirect it from an in-line file
  exec 9<&0 <<EOF
abc                    ${PWD}/abc
abc/def                ${PWD}/abc/def
abc/def/ghi            ${PWD}/abc/def/ghi
abc/./def              ${PWD}/abc/def
abc/../def             ${PWD}/def
abc/../def/../ghi      ${PWD}/ghi
/abc                   /abc
/abc/def               /abc/def
/abc/def/ghi           /abc/def/ghi
/abc/def/../../ghi     /ghi
/abc/def/ghi/../../jkl /abc/jkl
/abc/../def            /def
/abc/../def/../ghi     /ghi
./abc                  ${PWD}/abc
../abc                 ${parent}/abc
../abc/def             ${parent}/abc/def
EOF
  while read relPath absPath; do
    # ignore comment and blank lines
    echo "${relPath}" |egrep -v "^(#|$)" >/dev/null || continue

    # test the function
    newPath=`shlib_relToAbsPath "${relPath}"`
    assertEquals "${relPath}" "${absPath}" "${newPath}"
  done
  exec 0<&9 9<&-
}
```