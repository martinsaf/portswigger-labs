# Lab: SQL injection Vulnerability in WHERE clause allowing retrieval of hidden data

**Category:** SQL Injection
**Difficulty:** Apprentice
**Objective:** Perform a SQL injection to display unreleased products

## 0. Attacker mindset

When a URL parameter is used directly in a SQL query without sanitization, I can inject SQL syntax to alter the query's logic. The goal is to make the `WHERE` clause always true or to comment out restrictive conditions, bypassing filters like `released = 1`

## 1. What is the vulnerability

The product category filter passes user input directly into a SQL query without any sanitization or parameterization. The original query: `SELECT * FROM products WHERE category = 'Gifts' AND released = 1` restricts results to released products only. By injecting SQL metacharacters, an attacker can break out of the string literal and modify the query logic

## 2. How did I exploit it?

1. Navigated to the Gifts category: `https://[lab]/filter?category=Gifts`
2. Modified the `category` parameter to inject a condition that is always true: `Gifts'+OR+1=1--`
3. The resulting query became: `SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`. The `--` comments out the rest of the line, bypassing the `released` restriction. The `OR 1=1` makes the `WHERE` clause true for all rows
4. The page displayed all products, including unreleased ones, solving the lab

## 3. Impact

- Expore of hidden or unreleased data
- Bypass of access controls and business logic filters
- Potential for further exploitation: extracting sensitive tables, user credentials, or modifying data

## 4. How can it be fixed?

- Use parametrized queries (prepared statements) instead of string concatenation for SQL queries
- Validate and sanitize user input, allowing only expected characters
- Apply the principle of least privilege to the database user
- Use stored procedures to encapsulate query logic, avoiding dynamic SQL

**MITRE ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)