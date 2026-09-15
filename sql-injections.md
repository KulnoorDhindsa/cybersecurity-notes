# SQLi (SQL injections)
---
*Resource: CTFs, PortSwigger's SQL Injection*

The core exploitbale flaw of queries is when user-input is put **directly** into SQL queries instead of being treated as simple data.

## Blind SQLi
Regular SQLi has *feedback*, Blind SQLi is when there is clear *injection point* (query is vulnerable), but no *error* or *data* shows as output on the webpage.

1. **Boolean-based Blind SQLi**: Boolean as in *conditional*, resulting in `True` or `False`
    - An attacker only needs to *observe* - difference in loading, slower image loading, different error page etc
    - When user data is directly inputed into the query, by **closing the `"` early**, query logic can be altered.
    - **Exploit**:  `AND`, `OR` can be used to re-write the `WHERE` and based off of the `True` or `False` condition, a command can be run

2. **Time-Based Blind SQLi**: No visible factor differentiates the outputs, so a *time* becomes a new factor.
    - Measuring how long the server takes to respond - longer or slower, depending on a query can indicate required responses
    - `SLEEP()`, `WAITFOR DELAY()` Mostly, slow loading means `True` and a faster / normal loading means `False`
    - **Why its dangerous**: With **zero feedback** required, an attacker can exfiltrate 

















## Blind SQLi
**Blind SQL Injections (Blind SQLi)** is security vulnerability where malicious SQL commands are executed *without* showing *error* or *any message* on the webpage.

### 1. Time-Based Blind SQLi 
`sleep()` command allows delay of query to be executed by *N seconds*, allowing a factor **outside of HTML response** to indicate certain outputs.

If website takes *longer to load*, then the IF condition is TRUE, and `sleep(N)` is running.
```SQL
SELECT * FROM users WHERE username="natas18" AND IF(SUBSTRING(password,1,1)="a", sleep(5), 0); --Pauses terminal for 5 seconds if password first letter is "a"
```
### 2. Boolean-based / Conditional Response Blind SQLi
When a *true-false* query is injected, and based on the response, certain commands are eecuted.