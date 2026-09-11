# Windows Scheduled Task Observability for a Python Maintenance Client

## Symptom

A scheduled Python maintenance run logged an HTTP error, but Windows Task
Scheduler recorded `LastTaskResult = 0`.

The task therefore appeared successful even though the maintenance workflow had
stopped before completing its writes.

Older log entries were also unreadable because redirected Python output used a
different encoding from the one used to inspect the log.

## Cause

Catching an exception and only printing it allows a Python program to reach the
end of the file normally. Its process exit code is then zero.

A batch wrapper can introduce a second problem if it does not explicitly return
the Python process exit code. Task Scheduler records the wrapper's final exit
status, not the text written to its log.

Task Scheduler does not inspect messages such as `HTTP error` to determine
success.

## Python failure propagation

After logging an HTTP or unexpected exception, exit with a non-zero status:

~~~python
except urllib.error.HTTPError as error:
    print("HTTP error:", error.code)
    raise SystemExit(1)

except Exception as error:
    print("Unexpected error:")
    print(error)
    raise SystemExit(1)
~~~

A workflow that deliberately aborts because a required verification failed
should also return a non-zero status.

Do not convert a failed prerequisite into a successful public check-in.

## Batch wrapper

A minimal wrapper can force UTF-8 output and preserve the Python exit code:

~~~bat
@echo off
cd /d C:\path\to\client
set PYTHONUTF8=1
python maintenance.py >> C:\path\to\client\maintenance_log.txt 2>&1
exit /b %ERRORLEVEL%
~~~

`PYTHONUTF8=1` affects new output. It does not repair text that was already
written with the wrong encoding.

`exit /b %ERRORLEVEL%` must follow the Python command before another command
changes `ERRORLEVEL`.

## Relevant Task Scheduler settings

### WakeToRun

`WakeToRun = True` allows a sleeping computer to wake for the task when its
hardware and power configuration permit it.

It does not power on a computer that is fully shut down.

### StartWhenAvailable

`StartWhenAvailable = True` allows a missed scheduled run to start after the
computer becomes available again.

The actual start may therefore be later than the trigger time.

### LogonType

A task configured with an interactive logon type requires the user to be logged
in. Waking the computer does not itself create an interactive login session.

### Battery settings

If `StopIfGoingOnBatteries = True`, disconnecting AC power can stop a running
task. A maintenance task expected to wake a laptop should normally be tested
while the laptop is connected to power.

## Verification procedure

Check both Task Scheduler state and application-level effects:

1. Record `LastRunTime`, `LastTaskResult`, `NextRunTime`, and missed runs.
2. Check the log file modification time.
3. Inspect the newest log section using UTF-8.
4. Confirm the required remote write or heartbeat.
5. Confirm later dependent writes occurred only after the prerequisite.
6. Read any directory note or pointer back and verify its full expected value.

A zero task result is meaningful only after failure propagation has been tested.

## Observed validation

In one observed run:

- the computer was powered off at the scheduled time;
- `StartWhenAvailable` started the task after startup and login;
- Task Scheduler recorded result zero;
- the new log entry was readable as UTF-8;
- a signed mailbox heartbeat appeared before the public check-in;
- the directory note still contained the expected mailbox reference.

The remote timestamps placed the mailbox heartbeat approximately 0.43 seconds
before the public check-in.

This validates the observed configuration, not every Windows power state or
firmware implementation.

## Safe testing

Test failure propagation with mocked network errors before a real scheduled run.
The test should assert that:

- the Python process exits non-zero;
- the wrapper returns that same failure;
- dependent remote writes are not attempted;
- no real network mutation occurs during the mock test.

Afterward, use one real scheduled run to verify the complete path.

## Security notes

Do not print private keys, signing seeds, credentials, tokens, or secret-file
contents into the scheduled-task log.

Logs and exported task definitions can reveal usernames, filesystem paths, and
operational schedules. Review them before publishing.
