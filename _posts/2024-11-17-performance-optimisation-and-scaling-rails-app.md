---
layout: post
title: Performance Optimisation and Scaling in Rails app
subtitle:  Performance Optimisation and Scaling strategies
tags: [performance, scaling,  questions, ruby on rails]  
comments: true  
thumbnail-img: /assets/img/performance.gif
---

Along with writing clean and efficient code, optimizing performance and scaling a Rails application require strategies at the **Model**, **View**, and **Controller** levels, as well as broader architectural considerations. Here's a breakdown of key approaches:

---

## **Performance Optimizations in Rails MVC**

### **1. Model Optimizations**

#### **a. Query Optimization**
- **Use ActiveRecord Efficiently**: Avoid N+1 queries by using `includes`, `joins`, or `eager_load`.
  ```ruby
  # Inefficient
  posts.each { |post| post.comments }

  # Optimized
  posts = Post.includes(:comments)
  posts.each { |post| post.comments }
  ```
- **Select Only Necessary Columns**: Use `select` to reduce database load.
  ```ruby
  # Inefficient
  users = User.all 

  # Optimized
  users = User.select(:id, :name)
  ```
- **Indexing**: Add database indexes for frequently queried columns.
  ```bash
  rails generate migration AddIndexToUsersEmail
  ```
  ```ruby
  add_index :users, :email
  ```

#### **b. Caching**
- **Query Caching**: Rails caches query results by default. Use tools like `cache_store` to persist results.
  ```ruby
  Rails.cache.fetch("user_#{user.id}_posts") { user.posts.to_a }
  ```

#### **c. Avoid Callbacks for Heavy Operations**
Move non-critical logic to background jobs using tools like **Sidekiq**.
```ruby
class User < ApplicationRecord
  after_create :send_welcome_email

  private

  def send_welcome_email
    WelcomeEmailJob.perform_later(self.id)
  end
end
```

---

### **2. View Optimizations**
#### **a. Fragment Caching**
Cache parts of views that don’t change often.
```ruby
# In your view
<% cache(@post) do %>
  <%= render @post %>
<% end %>
```

#### **b. Avoid Complex Helpers**
Move complex logic from views to presenters or decorators for better performance and readability.

#### **c. Reduce Asset Loading**
- Use the **asset pipeline** or **Webpacker** to precompile assets.
- Minify JavaScript and CSS files.
- Serve assets via a CDN like **Cloudflare** or **AWS CloudFront**.

#### **d. Lazy Loading**
Load images and other assets lazily to improve page speed.
```html
<img src="placeholder.jpg" data-src="actual-image.jpg" loading="lazy">
```

---

### **3. Controller Optimizations**
#### **a. Reduce Instance Variables**
Pass only the required data to views.
```ruby
# Inefficient
@user = User.find(params[:id])
@posts = @user.posts

# Optimized
@posts = User.find(params[:id]).posts
```

#### **b. Use Concerns for Reusable Logic**
Extract shared logic into **concerns** to improve maintainability and avoid redundant processing.

#### **c. Avoid Overloading Controllers**
Use **service objects** for complex logic.
```ruby
class UserController < ApplicationController
  def create
    result = UserCreationService.new(params).call
    if result.success?
      redirect_to result.user
    else
      render :new
    end
  end
end
```

#### **d. Pagination**
Use gems like **kaminari** or **will_paginate** to paginate large datasets.
```ruby
@posts = Post.page(params[:page]).per(10)
```

---

## **Scaling Strategies in Rails MVC**

<p align="center" >
  <img src="/assets/img/ec2 load balance.png" alt="ec2" />
</p>

### **1. Horizontal Scaling**
#### **a. Load Balancing**
Distribute traffic across multiple application servers using tools like **AWS Elastic Load Balancer** or **NGINX**.

#### **b. Statelessness**
Ensure requests don’t depend on server-specific states by storing sessions in a distributed store like **Redis** or **Memcached**.

---

### **2. Database Scaling**
#### **a. Read Replicas**
Use read replicas to offload read queries from the primary database.

#### **b. Sharding**
Split data across multiple databases based on specific criteria (e.g., user ID ranges).

#### **c. Connection Pooling**
Optimize database connections by tuning `pool` size in `config/database.yml`.
```yaml
production:
  pool: 25
```

#### **d. Background Jobs for Non-Critical Writes**
Offload heavy write operations to a background queue.

---

### **3. Caching Strategies**
#### **a. Full-Page Caching**
Cache entire HTML pages for highly static pages.
- Use **Redis** or **Memcached** to store cached content.
- Expire caches when updates occur:
  ```ruby
  expire_fragment("key")
  ```

#### **b. API Response Caching**
Cache JSON API responses to reduce backend load.

#### **c. HTTP Caching**
Leverage HTTP headers:
```ruby
Cache-Control: public, max-age=3600
```

---

### **4. Asynchronous Processing**
- Use **Sidekiq**, **Resque**, or **Delayed Job** to process non-critical operations asynchronously.
- Examples:
  - Sending emails.
  - Generating reports.
  - Sending push notifications.

---

### **5. Microservices Architecture**
For large applications, consider splitting functionalities into **microservices**.
- Example: Separate services for authentication, payments, and notifications.
- Use **gRPC** or **REST APIs** for communication between services.

---

### **6. Distributed Application Patterns**
- **Event-Driven Architecture**: Use message queues like **RabbitMQ** or **Kafka** to decouple services.
- **CQRS (Command Query Responsibility Segregation)**: Separate write and read concerns for better scalability.

---

### **7. Multiple Data Source Management**
Rails supports multiple databases:
- Configure in `database.yml`:
  ```yaml
  development:
    primary:
      database: primary_db
    analytics:
      database: analytics_db
  ```
- Use ActiveRecord's `connects_to` method:
  ```ruby
  class ApplicationRecord < ActiveRecord::Base
    connects_to database: { writing: :primary, reading: :analytics }
  end
  ```

---

## **Monitoring and Observability**
- Use tools like **New Relic**, **Datadog**, or **Scout APM** for monitoring application performance.
- Implement structured logging with **Sentry**, **Lograge** to centralize and analyze logs.

---

By adopting these strategies along with writing clean and optimised code, you can enhance both the performance and scalability of your Rails application(or any MVC app), ensuring it can handle increased load and complexity efficiently.