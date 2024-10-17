---
layout: post  
title: SQL Queries and their Rails Equivalents
subtitle: Key SQL Query types(DDL, DML, DQL, DCL & TCL) and their Rails Active Record handling.
tags: [SQL, Active Record, interview questions, ruby on rails]  
comments: true  
thumbnail-img: /assets/img/data-code.gif
---


In SQL, queries can be categorized into five types based on their purpose: **DDL (Data Definition Language)**, **DML (Data Manipulation Language)**, **DQL (Data Query Language)**, **DCL (Data Control Language)**, and **TCL (Transaction Control Language)**. Let's explore each with its corresponding operations and their equivalents in Ruby on Rails using Active Record.

<p align="center">
  <img src="/assets/img/sql-query-types.png" alt="sql-query-types" />
</p>

### 1. **DDL (Data Definition Language)**
DDL is used to define and manage database schema, such as creating or modifying tables, indexes, and constraints.

#### Common SQL Commands:
- `CREATE`: Create new database objects (e.g., tables, indexes).
- `ALTER`: Modify existing database objects (e.g., adding a new column).
- `DROP`: Delete database objects (e.g., tables, indexes).

#### Rails Equivalents:
In Rails, these operations are typically handled using **migrations**.

**Example of DDL in SQL:**
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);
```

**Rails Migration for Creating a Table:**
```ruby
class CreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      t.string :name
      t.string :email
      t.timestamps
    end
  end
end
```

**Example of modifying a table (SQL `ALTER`):**
```sql
ALTER TABLE users ADD COLUMN age INT;
```

**Rails Migration for Modifying a Table:**
```ruby
class AddAgeToUsers < ActiveRecord::Migration[6.0]
  def change
    add_column :users, :age, :integer
  end
end
```


### 2. **DML (Data Manipulation Language)**
DML is used to manipulate the data inside the tables, including inserting, updating, and deleting records.

#### Common SQL Commands:
- `INSERT`: Add new records.
- `UPDATE`: Modify existing records.
- `DELETE`: Remove records.

#### Rails Equivalents:
In Rails, these operations are handled by Active Record methods like `create`, `update`, and `destroy`.

**Example of DML in SQL (`INSERT`):**
```sql
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
```

**Rails Active Record Equivalent for Creating a Record:**
```ruby
User.create(name: 'John Doe', email: 'john@example.com')
```

**Example of DML in SQL (`UPDATE`):**
```sql
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;
```

**Rails Active Record Equivalent for Updating a Record:**
```ruby
user = User.find(1)
user.update(email: 'newemail@example.com')
```

**Example of DML in SQL (`DELETE`):**
```sql
DELETE FROM users WHERE id = 1;
```

**Rails Active Record Equivalent for Deleting a Record:**
```ruby
User.find(1).destroy
```


### 3. **DQL (Data Query Language)**
DQL is primarily used for querying data from the database. The most common DQL command is `SELECT`.

#### Common SQL Commands:
- `SELECT`: Retrieve data from one or more tables.

#### Rails Equivalents:
In Rails, querying data is handled using Active Record methods like `where`, `select`, `find`, etc.

**Example of DQL in SQL (`SELECT`):**
```sql
SELECT * FROM users WHERE name = 'John Doe';
```

**Rails Active Record Equivalent for Querying Data:**
```ruby
User.where(name: 'John Doe')
```

**Example of DQL in SQL (`SELECT` with specific columns):**
```sql
SELECT name, email FROM users WHERE age > 25;
```

**Rails Active Record Equivalent for Selecting Specific Columns:**
```ruby
User.where('age > ?', 25).select(:name, :email)
```
### SQL JOINS

<p align="center">
  <img src="/assets/img/joins.jpg" alt="sql-joins" />
</p>

In SQL, **joins** are used to combine rows from two or more tables based on a related column between them. There are different types of joins, and they are all part of **Data Query Language (DQL)** as they are used for querying data. Let's explore the most common types of joins and their equivalents in Rails using **Active Record**.

### A. **INNER JOIN**
An **INNER JOIN** returns records that have matching values in both tables. It filters out rows that do not match the condition.

#### SQL Query:
```sql
SELECT users.name, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

#### Rails Active Record Equivalent:
```ruby
User.joins(:orders).select('users.name, orders.amount')
```

#### Explanation:
- The SQL query joins the `users` and `orders` tables based on the `user_id` column in the `orders` table.
- In Rails, the `joins` method is used to create an inner join. You specify the associated table (`:orders`), and you can use `select` to fetch specific columns.

### B. **LEFT OUTER JOIN (or LEFT JOIN)**
A **LEFT JOIN** returns all records from the left table (`users`), and the matched records from the right table (`orders`). If there is no match, NULL values are returned for columns from the right table.

#### SQL Query:
```sql
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

#### Rails Active Record Equivalent:
```ruby
User.left_joins(:orders).select('users.name, orders.amount')
```

#### Explanation:
- The SQL query returns all users, including those who do not have any associated orders (with `NULL` values for `orders.amount`).
- In Rails, `left_joins` is used to create a left join.

### C. **RIGHT OUTER JOIN (or RIGHT JOIN)**
A **RIGHT JOIN** is similar to a **LEFT JOIN**, but it returns all records from the right table (`orders`), and the matched records from the left table (`users`). If there is no match, NULL values are returned for columns from the left table.

#### SQL Query:
```sql
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```

#### Rails Active Record Equivalent:
Active Record does not directly support `RIGHT JOIN`, but you can use raw SQL for this.

```ruby
User.joins('RIGHT JOIN orders ON users.id = orders.user_id').select('users.name, orders.amount')
```

### D. **FULL OUTER JOIN**
A **FULL OUTER JOIN** returns all records when there is a match in either the left (`users`) or right (`orders`) table. Rows without a match in one of the tables will return `NULL` for the columns from the other table.

#### SQL Query:
```sql
SELECT users.name, orders.amount
FROM users
FULL OUTER JOIN orders ON users.id = orders.user_id;
```

#### Rails Active Record Equivalent:
Active Record doesn't support `FULL OUTER JOIN` out of the box, so you can use raw SQL.

```ruby
User.joins('FULL OUTER JOIN orders ON users.id = orders.user_id').select('users.name, orders.amount')
```

### E. **CROSS JOIN**
A **CROSS JOIN** returns the Cartesian product of the two tables, i.e., all possible combinations of rows between the two tables.

#### SQL Query:
```sql
SELECT users.name, orders.amount
FROM users
CROSS JOIN orders;
```

#### Rails Active Record Equivalent:
For cross joins, you can use raw SQL in Rails as well.

```ruby
User.joins('CROSS JOIN orders').select('users.name, orders.amount')
```

### F. **SELF JOIN**
A **SELF JOIN** is used to join a table to itself. This is useful in hierarchical relationships, such as when employees have managers who are also employees.

#### SQL Query:
```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;
```

#### Rails Active Record Equivalent:
```ruby
Employee.joins("INNER JOIN employees managers ON employees.manager_id = managers.id")
        .select('employees.name AS employee, managers.name AS manager')
```

### G. **NATURAL JOIN**
A **NATURAL JOIN** automatically joins tables based on columns with the same name and data type in both tables. It avoids specifying a specific condition but joins the tables using the natural relationship between them.

#### SQL Query:
```sql
SELECT users.name, orders.amount
FROM users
NATURAL JOIN orders;
```

#### Rails Active Record Equivalent:
Rails does not support **NATURAL JOIN** natively, but you can use raw SQL.

```ruby
User.joins('NATURAL JOIN orders').select('users.name, orders.amount')
```

In Rails, Active Record provides a convenient interface for most common SQL joins, and for more complex queries like **FULL JOIN**, **RIGHT JOIN**, and **CROSS JOIN**, raw SQL can be used within Active Record methods.

### 4. **DCL (Data Control Language)**
DCL is used to control access to the data in the database. It is concerned with permissions and access control.

#### Common SQL Commands:
- `GRANT`: Grant access rights.
- `REVOKE`: Remove access rights.

#### Rails Equivalents:
Rails does not directly handle database-level permissions; this is typically done at the database level using SQL. However, you can control user permissions and authorization using gems like **Pundit** or **CanCanCan** in Rails for application-level access control.

**Example of DCL in SQL (`GRANT`):**
```sql
GRANT SELECT, INSERT ON users TO 'someuser';
```

**Example of DCL in SQL (`REVOKE`):**
```sql
REVOKE INSERT ON users FROM 'someuser';
```

<p align="center">
  <img src="/assets/img/hacker-cat.gif" alt="hacker-cat" />
</p>

### 5. **TCL (Transaction Control Language)**
TCL is used to control the execution of transactions. It ensures that a series of operations either complete successfully or fail together.

#### Common SQL Commands:
- `BEGIN`: Start a transaction.
- `COMMIT`: Save the changes made in the transaction.
- `ROLLBACK`: Undo the changes if there’s an error in the transaction.

#### Rails Equivalents:
In Rails, you can use Active Record's `transaction` method to manage transactions.

**Example of TCL in SQL (`BEGIN`, `COMMIT`, `ROLLBACK`):**
```sql
BEGIN;
UPDATE users SET name = 'Jane Doe' WHERE id = 1;
DELETE FROM orders WHERE user_id = 1;
COMMIT;
```

**Rails Active Record Equivalent for Transaction Handling:**
```ruby
User.transaction do
  user = User.find(1)
  user.update(name: 'Jane Doe')
  Order.where(user_id: user.id).destroy_all
end
```

If an exception occurs, the transaction will be rolled back automatically.
---

### Summary

- **DDL (Data Definition Language)** handles schema changes like creating, altering, or dropping tables. In Rails, this is done using migrations.
- **DML (Data Manipulation Language)** is for inserting, updating, and deleting records. Rails provides methods like `create`, `update`, and `destroy`.
- **DQL (Data Query Language)** is for querying data. Active Record’s `where`, `find`, and `select` methods are the Rails equivalents.
- **DCL (Data Control Language)** controls permissions, though in Rails, this is handled by gems for authorization like Pundit or CanCanCan.
- **TCL (Transaction Control Language)** controls transactions, and in Rails, Active Record's `transaction` method manages these operations.

This classification helps you understand the roles of various SQL queries and how they map to Rails’ Active Record, making your database operations in Rails seamless and easy to manage.