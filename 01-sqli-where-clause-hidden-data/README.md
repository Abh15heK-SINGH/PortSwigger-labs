# SQL Injection Vulnerability in WHERE Clause — Retrieving Hidden Data

## Lab Information

| Field               | Details                          |
| ------------------- | -------------------------------- |
| **Platform**        | PortSwigger Web Security Academy |
| **Lab Level**       | Apprentice                       |
| **Vulnerability**   | SQL Injection                    |
| **Injection Point** | `category` parameter             |
| **Attack Type**     | WHERE clause manipulation        |

---

## 1. Objective

The application contains a SQL injection vulnerability in the product category filter.

The objective is to manipulate the SQL query so that the application displays one or more products that have not been released.

---

## 2. Understanding the Application

When a user selects a product category, the application performs a SQL query similar to:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
AND released = 1
```

The `released = 1` condition ensures that only released products are returned.

The `category` value is controlled by the user and is incorporated into the SQL query.

---

## 3. Testing Methodology

### Step 1 — Establish a Baseline

I first accessed the product category normally:

```text
/filter?category=Gifts
```

The application returned products belonging to the `Gifts` category.

<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/8b7da1fd-dd3c-45c0-acdc-7704854de7bb" />

---

### Step 2 — Test the `category` Parameter

I tested whether the `category` parameter could influence the SQL query.

The following SQL injection payload was used:

```sql
Gifts' OR 1=1--
```

<img width="960" height="540" alt="image" src="https://github.com/user-attachments/assets/ce323bac-4471-482f-af96-4914130ea320" />


---

## 4. Payload Analysis

The payload:

```sql
Gifts' OR 1=1--
```

contains three important components.

### `'`

The single quote terminates the original SQL string:

```sql
category = 'Gifts'
```

becomes:

```sql
category = 'Gifts'
```

with the attacker-controlled SQL continuing after the closing quote.

### `OR 1=1`

`1=1` is always true.

Therefore:

```sql
category = 'Gifts' OR 1=1
```

can cause the condition to evaluate as true for additional records.

### `--`

The `--` sequence starts a SQL comment.

This causes the remaining portion of the original query to be ignored.

The resulting query is approximately:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
OR 1=1
-- AND released = 1
```

## 6. Vulnerability Impact

SQL Injection can allow an attacker to manipulate database queries.

Depending on the vulnerable query and database privileges, possible impacts include:

* Bypassing application filters
* Retrieving unauthorized information
* Authentication bypass
* Accessing sensitive database records
* Modifying or deleting database data
* Performing further attacks against the application

The actual impact depends on the database permissions and the vulnerable SQL query.

---

## 7. Remediation

The primary remediation is to use **parameterized queries / prepared statements**.

Instead of constructing SQL queries using user input:

```sql
SELECT * FROM products
WHERE category = 'USER_INPUT'
AND released = 1
```

the application should use a parameterized query:

```sql
SELECT * FROM products
WHERE category = ?
AND released = 1
```

The category value should be supplied separately as a parameter.

Additional security controls include:

* Use parameterized queries throughout the application.
* Apply least-privilege permissions to database accounts.
* Avoid exposing detailed database errors.
* Perform regular SQL Injection testing.
* Validate input where appropriate as a secondary security control.
