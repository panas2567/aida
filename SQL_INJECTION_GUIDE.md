# SQL Injection Attack Types and Prevention Guide

## Overview

SQL injection is a code injection technique that exploits security vulnerabilities in an application's database layer. It occurs when user input is improperly sanitized before being included in SQL queries, allowing attackers to manipulate database operations. This document covers the 5 most common types of SQL injection attacks and their prevention methods.

## 1. Classic SQL Injection (In-band)

### Description
Classic SQL injection is the most straightforward type where the attacker uses the same communication channel to both launch the attack and gather results. The attacker can see the direct response from the database in the application's response.

### Example Vulnerable Code
```sql
-- Vulnerable query construction
SELECT * FROM users WHERE username = '$username' AND password = '$password'
```

### Attack Example
```sql
-- Attacker input for username: admin' OR '1'='1' --
-- Resulting query:
SELECT * FROM users WHERE username = 'admin' OR '1'='1' --' AND password = '$password'
```

### Prevention Methods
- **Use Parameterized Queries/Prepared Statements**: Always use bound parameters instead of string concatenation
- **Input Validation**: Validate all user inputs against expected patterns
- **Escape Special Characters**: Properly escape SQL special characters in user input

```sql
-- Safe parameterized query example
SELECT * FROM users WHERE username = ? AND password = ?
```

## 2. Blind SQL Injection

### Description
Blind SQL injection occurs when an application is vulnerable to SQL injection but the results of the query are not visible to the attacker. The attacker must infer information based on the application's behavior, such as response times or error messages.

### Types of Blind Injection

#### Boolean-based Blind SQL Injection
The attacker sends SQL queries that return true/false results and determines the answer based on the application's response.

#### Time-based Blind SQL Injection
The attacker uses database commands that cause intentional delays to infer whether the query executed successfully.

### Attack Example
```sql
-- Boolean-based example
-- Attacker tests: admin' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a' --

-- Time-based example  
-- Attacker input: admin'; IF (1=1) WAITFOR DELAY '00:00:05' --
```

### Prevention Methods
- **Parameterized Queries**: Same as classic injection prevention
- **Error Handling**: Implement generic error messages that don't reveal database structure
- **Response Time Normalization**: Ensure consistent response times regardless of query results

## 3. Union-based SQL Injection

### Description
Union-based SQL injection leverages the UNION SQL operator to combine results from multiple SELECT statements. Attackers use this technique to extract data from other tables in the database by appending additional SELECT statements to the original query.

### Attack Example
```sql
-- Original query
SELECT product_name, price FROM products WHERE category = '$category'

-- Attacker input: electronics' UNION SELECT username, password FROM users --
-- Resulting query:
SELECT product_name, price FROM products WHERE category = 'electronics' UNION SELECT username, password FROM users --'
```

### Requirements for Success
- The number of columns in both SELECT statements must be identical
- Data types must be compatible
- The attacker needs to determine the number of columns in the original query

### Prevention Methods
- **Parameterized Queries**: Prevent injection at the source
- **Least Privilege Principle**: Database users should only have necessary permissions
- **Column Count Protection**: Validate expected result structure

## 4. Error-based SQL Injection

### Description
Error-based SQL injection relies on database error messages to extract information about the database structure and data. Attackers intentionally cause database errors that reveal sensitive information in the error messages.

### Attack Example
```sql
-- Attacker input designed to cause an error
admin' AND (SELECT COUNT(*) FROM information_schema.tables)=1 AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT version()), 0x7e)) --

-- This might return an error message like:
-- "XPATH syntax error: '~5.7.29-0ubuntu0.18.04.1~'"
```

### Common Error-based Techniques
- **EXTRACTVALUE()**: MySQL function that can be exploited to extract data
- **UpdateXML()**: Another MySQL function vulnerable to exploitation
- **Conversion Errors**: Forcing type conversion errors to reveal data

### Prevention Methods
- **Custom Error Pages**: Never display detailed database errors to users
- **Error Logging**: Log errors securely for debugging without exposing them
- **Input Sanitization**: Prevent malicious input from reaching the database
- **Database Security**: Configure database to minimize information leakage in errors

## 5. Second-order SQL Injection (Stored)

### Description
Second-order SQL injection occurs when malicious input is stored in the database without causing immediate harm, but later triggers an injection attack when the stored data is retrieved and used in another SQL query without proper sanitization.

### Attack Flow
1. Attacker submits malicious input that gets stored in the database
2. The application later retrieves this data and uses it in a new SQL query
3. The stored malicious input executes as part of the new query

### Example Scenario
```sql
-- Step 1: Attacker registers with username
Username: admin'--

-- Step 2: Application stores this in database safely
INSERT INTO users (username, email) VALUES ('admin\'--', 'user@email.com')

-- Step 3: Later, application retrieves and uses this data unsafely
SELECT * FROM user_profiles WHERE username = 'admin'--'
-- The comment (--) causes the rest of the query to be ignored
```

### Prevention Methods
- **Consistent Input Sanitization**: Sanitize data both when storing and retrieving from database
- **Output Encoding**: Encode data when displaying it
- **Parameterized Queries**: Use for all database operations, not just user input
- **Data Validation**: Validate data integrity at multiple application layers

## General Prevention Best Practices

### 1. Input Validation and Sanitization
- Validate all input against expected patterns
- Use whitelist validation when possible
- Sanitize special characters properly

### 2. Parameterized Queries and Prepared Statements
```sql
-- Instead of string concatenation
SELECT * FROM users WHERE id = '$user_id'

-- Use parameterized queries
SELECT * FROM users WHERE id = ?
```

### 3. Least Privilege Principle
- Database accounts should have minimal necessary permissions
- Use separate accounts for different application functions
- Never use administrative accounts for application connections

### 4. Regular Security Testing
- Conduct regular penetration testing
- Use automated vulnerability scanners
- Implement code review processes

### 5. Web Application Firewalls (WAF)
- Deploy WAF solutions to filter malicious requests
- Configure rules to detect common injection patterns
- Monitor and log suspicious activities

### 6. Database Security Configuration
- Keep database software updated
- Disable unnecessary database features
- Use database-specific security features

## Detection and Monitoring

### Log Analysis
Monitor application and database logs for:
- Unusual SQL query patterns
- Multiple failed login attempts
- Unexpected database errors
- Queries with SQL keywords in user input fields

### Automated Detection Tools
- **Static Application Security Testing (SAST)**: Analyze source code for vulnerabilities
- **Dynamic Application Security Testing (DAST)**: Test running applications
- **Interactive Application Security Testing (IAST)**: Real-time application monitoring

## Conclusion

SQL injection attacks remain one of the most critical web application security risks. Understanding these five common types and implementing comprehensive prevention strategies is essential for maintaining application security. The key to prevention lies in treating all user input as potentially malicious and implementing multiple layers of defense including input validation, parameterized queries, proper error handling, and regular security assessments.

Remember: Security is not a one-time implementation but an ongoing process that requires constant vigilance and updates as new attack vectors emerge.

---
*This document serves as a defensive security guide. For more information on web application security, consult OWASP guidelines and security best practices.*