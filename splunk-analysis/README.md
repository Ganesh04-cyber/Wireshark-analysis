# Project 2: Splunk Log Analysis

## Data Loaded
I uploaded splunk_practice_round4.csv (21 events) to Splunk index "practice" with source type "csv". The file contains Windows Security-style events with the fields _time, host, user, EventCode, src_ip, and action.

## Search 1: Count events by EventCode
Search: index=practice | stats count by EventCode

Result:
- 4624 (successful login): 4
- 4625 (failed login): 14
- 4634 (logoff): 1
- 4688 (process created): 1
- 4728 (user added to group): 1

![Count by EventCode](01-count-by-eventcode.png)

## Search 2: Failed logins by source IP and user
Search: index=practice EventCode=4625 | stats count by src_ip, user | sort -count

Result (top rows):
- svc_sql from 10.4.5.40: 7 failures
- finance from 172.16.9.4: 5 failures
- rohan from 10.4.5.22: 1 failure
- neha from 10.4.5.11: 1 failure

![Failed logins by IP and user](02-failed-logins-by-ip-user.png)

## Search 3: Failures for svc_sql
Search: index=practice EventCode=4625 user=svc_sql | table _time, user, src_ip, EventCode

Result: 7 failed logins from 10.4.5.40 on DC-01, all between 07:30:00 and 07:30:06 (6 seconds). No successful login for svc_sql appears in the file.

![svc_sql failures](03-svc-sql-failures.png)

## Top Finding
The account svc_sql had 7 failed logins in 6 seconds from 10.4.5.40, an internal IP. The speed and repetition point to an automated process, such as a service with an outdated password, or a brute-force attempt from inside the network. The finance account also had 5 failures from 172.16.9.4 within 40 seconds.

## Recommended Next Actions
1. Confirm with the owner whether svc_sql runs a scheduled task with an outdated password, and update it if so.
2. Check the source host 10.4.5.40 for unauthorized tools or scripts.
3. Investigate the finance failures from 172.16.9.4 and block the IP if it isn't expected.
4. Look for any successful login (EventCode 4624) from these accounts after the failures.
