_The following code is pulled from the unit tests of [log4sh](http://log4sh.sourceforge.net/). The example is somewhat complex, but it does demonstrate the use of a possibility matrix for test outcomes._

In this example, sending a logging message to a log4sh appender at a given logging level should result in a message at that logging level or above. Basically, what is happening is that a logging message is being sent at all six possible logging levels simultaneously, and the data file contains a list of the logging levels where a message should actually show up. Don't worry if that makes no sense as the example is for log4sh, but some studying of what is happening will help you understand how you might implement a possibility matrix of your own.

The possibility matrix (`priorityMatrix.data`)
```
# column definitions
#  priority
#  which priorities should generate a message
#
# note: the default IFS in pure POSIX shells seems to parse only spaces when
# using the 'read' builtin. as such, only spaces are used as separation
# characters below.
#
# priority TRACE DEBUG INFO WARN ERROR FATAL
TRACE   1 1 1 1 1 1
DEBUG   0 1 1 1 1 1
INFO    0 0 1 1 1 1
WARN    0 0 0 1 1 1
ERROR   0 0 0 0 1 1
FATAL   0 0 0 0 0 1
OFF     0 0 0 0 0 0
```

The test function (actual log4sh-1.4.2 [testPriority](http://log4sh.svn.sourceforge.net/viewvc/log4sh/tags/source/1.4.2/src/test/testPriority?revision=574&view=markup) unit test)
```
testPriorityMatrix()
{
  DEBUG=': '
  PRIORITY_NAMES='TRACE DEBUG INFO WARN ERROR FATAL'
  PRIORITY_POS='1 2 3 4 5 6'
  TEST_DATA='priorityMatrix.data'

  # to enable debugging, uncomment the following line
  #DEBUG=''

  while read priority outputs; do
    # ignore comment lines or blank lines
    echo "${priority}" |egrep -v '^(#|$)' >/dev/null || continue

    ${DEBUG} echo "setting appender priority to '${priority}' priority"
    appender_setLevel ${APP_NAME} ${priority}
    appender_activateOptions ${APP_NAME}

    # the number of outputs must match the number of priority names and
    # positions for this to work
    for pos in ${PRIORITY_POS}; do
      testPriority=`echo ${PRIORITY_NAMES} |cut -d' ' -f${pos}`
      shouldOutput=`echo ${outputs} |cut -d' ' -f${pos}`

      ${DEBUG} echo "generating '${testPriority}' message"
      result=`log ${testPriority} 'dummy'`
      ${DEBUG} echo "result=${result}"

      if [ ${shouldOutput} -eq 1 ]; then
        assertNotNull \
          "'${priority}' priority appender did not emit a '${testPriority}' message" \
          "${result}"
      else
        assertNull \
          "'${priority}' priority appender emitted a '${testPriority}' message" \
          "${result}"
      fi
    done
  done <"${TEST_DATA}"
}
```