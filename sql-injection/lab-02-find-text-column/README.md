# Lab: SQL injection UNION attack, finding a column containing text

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Perform a UNION attack that returns an additional row containing the random value provided by the lab.

## 0. Attacker mindset

After finding the number of columns, I need to know which columns can hold string data. Only string-compatible columns can display the extracted information in the application's response. A single misaligned data type can break the whole query, so I probe each column witha test string until one sticks.

## 1. What is the vulnerability?

The product category filter is vulnerable to SQL injection. The application takes the user-supplied `category` parameter and concatenates it directly into a `SELECT` query. An attacker can inject a `UNION SELECT` statement to append a new row to the results. For the injected row to appear, the values must be compatible with the original column types.

## 2. How did I exploit it?

1. Navigated to a product category (`filter?category...`)
2. Used DevTools -> Network -> **Edit and Resend** to test `ORDER BY` clauses:
    - `'ORDER BY 1--`, `2--`, `3--` worked; `4--` caused an error.
    - Confirmed the query returns **3 columns**
3. Probed each column for string compatibility by injecting the lab-provided value (`yQYpxG`) in each position:
    - `' UNION SELECT 'yQYpxG',NULL,NULL--` -> error
    - `' UNION SELECT NULL,'yQYpxG',NULL--` → success
4. The application displayed `yQYpxG` in the product table, confirming the second column accepts strings and solving the lab

## 3. Impact

- Knowing which columns support string data prepares the ground for extracting sensitive information (usernames, passwords, credit card numbers) with subsequent UNION attacks
- Even extracting a simple string proves that the injection point allows reading data from arbitrary database tables
- Highlights the fact that applications often assume rather than verify the types of data they are handling

## 4. How can it be fixed?

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)