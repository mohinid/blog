---
layout: post
title: Optimistic and Pessimistic Locking in Rails
subtitle:  Locking implementation in rails and why implicit DB locks are not enough
tags: [locking, transactions, db, data consistency, ruby on rails]  
comments: true  
thumbnail-img: /assets/img/lock door.gif
---

In Rails, **optimistic locking** and **pessimistic locking** are mechanisms to handle concurrent updates to database records. These approaches help maintain data integrity when multiple users or processes attempt to update the same record simultaneously. Here's an implementation of each with examples:

<p align="center" >
  <img src="/assets/img/transaction lock.png" alt="transaction lock" />
</p>
---

### **1. Optimistic Locking**

Optimistic locking assumes that conflicts are rare and checks for conflicts when saving a record. It relies on a version column (e.g., `lock_version`) in the database to detect concurrent modifications.

#### **How It Works:**
- Each record has a `lock_version` column.
- When a record is read, the current `lock_version` is noted.
- Before saving, Rails checks if the `lock_version` in the database matches the one read earlier.
- If the versions differ, an exception (`ActiveRecord::StaleObjectError`) is raised.

#### **Setup:**

1. Add a `lock_version` column:
   ```bash
   rails generate migration AddLockVersionToPosts lock_version:integer
   rails db:migrate
   ```

2. Update the model:
   ```ruby
   class Post < ApplicationRecord
     # Optimistic locking is enabled automatically when lock_version exists
   end
   ```

3. Example usage:
   ```ruby
   post1 = Post.find(1)
   post2 = Post.find(1)

   post1.title = "Updated by User 1"
   post1.save # lock_version increments to 1

   post2.title = "Updated by User 2"
   post2.save # Raises ActiveRecord::StaleObjectError
   ```

#### **When to Use:**
- Scenarios where conflicts are rare.
- Better for web applications with occasional concurrent edits.
- Example: Collaborative document editing or user profile updates.

<p align="center" >
  <img src="/assets/img/doc editing.gif" alt="doc editing" />
</p>

---

### **2. Pessimistic Locking**

Pessimistic locking locks a record at the database level to prevent other transactions from modifying it until the lock is released. This approach avoids conflicts by blocking access to the locked record.

#### **How It Works:**
- Locks are applied at the database level (`SELECT ... FOR UPDATE`).
- Other transactions trying to modify the locked record are blocked until the lock is released.

#### **Usage in Rails:**

1. Example with `lock`:
   ```ruby
   post = Post.lock.find(1)
   post.update(title: "Locked and Updated")
   ```

2. Example with `select_for_update`:
   ```ruby
   Post.transaction do
     post = Post.find(1).lock!
     # Perform updates
     post.update!(title: "Pessimistically Locked")
   end
   ```

3. Handling blocking:
   - Use `lock!` or `Post.lock`.
   - The lock is released when the transaction ends.

#### **When to Use:**
- When conflicts are likely, and you need to ensure data consistency by blocking access.
- For critical updates like inventory management, reservations, or financial transactions.

<p align="center" >
  <img src="/assets/img/googlepay.gif" alt="google pay" />
</p>

#### Deadlock situations in Pessimistic locking:
Pessimistic locking in Rails (and any database-level locking) can cause a deadlock under certain conditions. Deadlocks occur when two or more transactions lock resources in different orders and block each other indefinitely while waiting for the other to release the lock.

**To avoid this:**
- Lock rows in a consistent order.
- Keep transactions short.
- Indexes: Indexes can minimize locking contention by reducing the number of rows locked.

---

### **Key Differences**

| **Feature**            | **Optimistic Locking**                              | **Pessimistic Locking**                      |
|-------------------------|----------------------------------------------------|----------------------------------------------|
| **Mechanism**           | Version column (application-level conflict check). | Database-level record locking.              |
| **Conflict Handling**   | Detects conflicts at save time (raises exception). | Blocks other transactions until lock ends.  |
| **Performance**         | No blocking; better for low contention.            | Potentially slower due to transaction locks. |
| **Use Case**            | Non-critical updates with rare conflicts.          | Critical updates or frequent contention.     |

---

<p align="center" >
  <img src="/assets/img/lock door.gif" alt="lock door" />
</p>

### While databases like MySQL provide locking mechanisms implicitly, **Rails-level optimistic and pessimistic locking** are still necessary in many scenarios because they address application-specific requirements that database-level locking alone cannot fully manage. 

### Here's why you might use Rails' locking strategies alongside database locking:

---

### **1. Optimistic Locking**
- **Detect Application-Level Conflicts:**
  Database-level locking doesn’t prevent two users from reading the same record and attempting to update it later. Optimistic locking tracks changes and ensures that the first user to save updates successfully, while others are notified of a conflict.

- **Lightweight Conflict Resolution:**
  Optimistic locking doesn’t block other users or processes from accessing the data. It only detects conflicts during the save operation, making it suitable for high-concurrency environments with rare conflicts.

- **Example Use Case:**
  - User A and User B read the same record simultaneously.
  - User A updates and saves the record.
  - User B tries to save their changes without knowing User A already updated it.
  - Rails detects the conflict via the `lock_version` field and raises an `ActiveRecord::StaleObjectError`.

#### **Why the Database Lock Alone Isn't Enough:**
The database can lock rows but cannot automatically resolve or notify users about conflicts in a user-friendly way. Rails’ optimistic locking handles this elegantly with minimal performance overhead.

---

### **2. Pessimistic Locking**
- **Easier API for Application Developers:**
  Pessimistic locking in Rails (via `lock` or `lock!`) provides an abstraction over the database's locking features. Instead of writing raw SQL (`SELECT ... FOR UPDATE`), developers can use Rails' ORM methods.

- **Granular Control Over Transactions:**
  Rails allows you to define locking logic directly within the application code, making it easier to integrate into business logic and workflows.

- **Database Agnostic:**
  Rails provides a consistent API for locking regardless of the underlying database. This abstraction simplifies migration between databases.

#### **Example Use Case:**
  - In an e-commerce system, two users attempt to purchase the last item in stock simultaneously.
  - Rails' pessimistic locking ensures only one user can decrement the stock count successfully, while others are blocked until the transaction completes.

#### **Why the Database Lock Alone Isn't Enough:**
Relying on implicit database locks often requires writing custom SQL and managing transactions manually, which can complicate the codebase. Rails' implementation simplifies these tasks and integrates them into ActiveRecord seamlessly.

---

## Key Benefits of Rails-Level Locking

| **Aspect**               | **Rails-Level Locking**                               | **Database Locking**                          |
|---------------------------|------------------------------------------------------|-----------------------------------------------|
| **Conflict Handling**     | Detects and resolves conflicts at the application level. | Prevents simultaneous updates but does not resolve conflicts. |
| **Ease of Use**           | Built into Rails with high-level APIs.               | Requires raw SQL queries for fine-grained control. |
| **Blocking**              | Optimistic locking doesn’t block, suitable for low contention. | Implicit locks may block rows until the transaction completes. |
| **Portability**           | Abstracted across databases, consistent with Rails ORM. | Database-specific syntax and behavior.        |

---

### **Conclusion**

Rails' optimistic and pessimistic locking complement the database's locking mechanisms by handling application-level concerns like **conflict resolution, user notifications, and consistent APIs** across different databases. These features improve code maintainability, scalability, and security. Have a nice week ahead!!


<p align="center" >
  <img src="/assets/img/safety first.gif" alt="safety first"  width="50%" height="50%"/>
</p>