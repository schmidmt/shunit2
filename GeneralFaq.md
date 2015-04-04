# General FAQ #


_Note: All examples are written using portable shell, or rather shell that will run on as many platforms as possible._

# How do I end all testing immediately upon failure? #
Due to the nature of shell scripting, it is not easy to do something like exception handling. As such, exiting from a tree of function calls to the outer-most call is not particularly easy. When writing unit tests, this is no different, which is why shUnit2 will run through each and every assert you have written rather than stopping after the first failure. If this is not the behavior you desire, there is something you can do.

To stop the test after the first assert fails, you will need to add '` || return`' to the end of each `assert*()` call that you make.

Below are two test functions. In the first function – `testAllAsserts()` – both asserts will be executed 100% of the time, even though the first assert will always fail. In the second function – `testUntilFailure()` – the entire test will be halted after the first failure is encountered.

All asserts will be tested
```
testAllAsserts()
{
  assertEqual 1 2
  assertEqual 2 2
}
```

Test will proceed until first failure
```
testUntilFailure()
{
  assertEqual 1 2 || return
  assertEqual 2 2 || return
}
```

# How do I feed data from files as input to tests? #
You have a choice between in-line data that is included as part of the test itself, or data that is stored in an external file. Strict adherents to proper unit testing will point out that unit tests should not touch the disk for any reason, which would indicate that in-line data should always be preferred over external files.

## In-line data ##
There are times when you have a list of various inputs with expected output that you would like to test. The function below provides just one means of doing this fully in-line within the test (i.e. no external input file is required).

This function is good, for example, when you want to pass a bunch of different input into a function and test the output is always the same.
```
testTryInlineFile()
{
  # save stdin, and redirect it from a file
  exec 9<&0 <<EOF
2
1
# this one should fail
0
EOF
  while read try; do
    # ignore comment lines or blank lines
    echo "${try}" |egrep -v '^(#|$)' >/dev/null || continue

    # run the test(s)
    assertTrue "[ ${try} -gt 0 ]"
  done
  # restore stdin
  exec 0<&9 9<&-
}
```

There are other times when different input into a function produces different output. This test function allows for a 'try' and 'expected' value to be used.
```
testTryExpectInlineFile()
{
  # save stdin, and redirect it from a file
  exec 9<&0 <<EOF
1 2
2 3
3 4
EOF
  while read try expected; do
    # ignore comment lines or blank lines
    echo "${try}" |egrep -v '^(#|$)' >/dev/null || continue

    # run the test(s)
    tryPlusOne=`expr ${try} + 1`
    assertEquals ${expected} ${tryPlusOne}
  done
  # restore stdin
  exec 0<&9 9<&-
}
```

## External files ##
There are times when for whatever reason, you need to use an external file as input for running tests, and an in-line file just won't do. The code below demonstrates a test where the input file needs to be read one line at a time so that tests on each line can be evaluated.

```
testExternalFile()
{
  testFile='/path/to/some/test/file'

  while read try; do
    # ignore comment lines or blank lines
    echo "${try}" |egrep -v '^(#|$)' >/dev/null || continue

    # run the test(s)
    assertTrue "[ ${try} -gt 0 ]"
  done <"${testFile}"
}
```

# How do I write my own custom asserts? #
The following code is pulled from the unit tests of [log4sh](http://log4sh.sourceforge.net/). The code demonstrates the testing of various conversion patterns in log4sh. The custom assert was written to reduce code duplication (think the [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) principle) for many tests, although only one test is shown.

```
assertPattern()
{
  msg=''
  if [ $# -eq 3 ]; then
    msg=$1
    shift
  fi
  pattern=$1
  expected=$2

  appender_setPattern ${APP_NAME} "${pattern}"
  appender_activateOptions ${APP_NAME}
  actual=`logger_info 'dummy'`
  msg=`eval "echo \"${msg}\""`
  assertEquals "${msg}" "${expected}" "${actual}"
}

testCategoryPattern()
{
  pattern='%c'
  expected='shell'
  msg="category '%c' pattern failed: '\${expected}' != '\${actual}'"
  assertPattern "${msg}" "${expected}" "${pattern}"
}
```

# How do I write tests for an already existing command? #
Writing unit tests for a pre-existing binary is not much different than writing unit tests for functions. The only real difference is that you do not have the degree of flexibility of being able to rewrite the command to make testing of unique conditions easier.

If you would like to see an example of testing the **mkdir** command, look at the `examples/mkdir_test.sh` file in the 2.1.5 (or later) release, or you can browse the source of [mkdir\_test.sh](http://code.google.com/p/shunit2/source/browse/trunk/source/2.1/examples/mkdir_test.sh) locally.