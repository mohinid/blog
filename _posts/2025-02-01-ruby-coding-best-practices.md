---
layout: post
title: Ruby Code Best Practices
subtitle: Ruby code best practices with Rubocop
tags: [ruby, best practices, standards, code review]
comments: true  
thumbnail-img: /assets/img/ruby.png
---
# Ruby Best Practices: Writing Clean, Maintainable, and Efficient Code

Ruby is known for its elegant syntax and developer-friendly features. However, writing clean, maintainable, and efficient Ruby code requires adhering to best practices. In this guide, we’ll cover essential Ruby best practices, including naming conventions, method structuring, exception handling, and using tools like **RuboCop** to enforce coding standards.

---
## 1. Follow Proper Naming Conventions
Ruby has clear naming conventions that improve code readability:

- **Classes & Modules:** Use `CamelCase` (e.g., `UserProfile`, `PaymentGateway`)
- **Methods & Variables:** Use `snake_case` (e.g., `calculate_total`, `user_id`)
- **Constants:** Use `SCREAMING_SNAKE_CASE` (e.g., `API_KEY`, `DEFAULT_TIMEOUT`)
- **Booleans:** Use predicate method names ending in `?` (e.g., `valid?`, `empty?`)
- **Bang Methods (`!`)**: Use `!` for methods that modify the object (e.g., `save!`, `update!`)

Example:
```ruby
class OrderProcessor  # CamelCase for class names
  DEFAULT_TAX_RATE = 0.05  # SCREAMING_SNAKE_CASE for constants
  
  def calculate_total(price, tax_rate = DEFAULT_TAX_RATE)  # snake_case for methods
    price + (price * tax_rate)
  end
end
```

---
## 2. Keep Methods Short and Focused
Each method should perform a **single responsibility**. If a method does too much, consider breaking it down into smaller helper methods.

**Bad:**
```ruby
def process_order(order)
  validate_order(order)
  apply_discounts(order)
  charge_payment(order)
  send_confirmation_email(order)
end
```

**Good:**
```ruby
def process_order(order)
  validate(order)
  apply_discounts(order)
  charge(order)
  notify_user(order)
end
```
Each of these smaller methods does one thing, making the code easier to test and maintain.

---
## 3. Use Symbols Instead of Strings for Hash Keys
Symbols are more memory-efficient than strings when used as hash keys.

**Bad:**
```ruby
user = { "name" => "Alice", "age" => 30 }
puts user["name"]
```

**Good:**
```ruby
user = { name: "Alice", age: 30 }
puts user[:name]
```

---
## 4. Use `each` Instead of `for`
Avoid using `for` loops in Ruby; prefer `each`, which is more idiomatic.

**Bad:**
```ruby
for num in [1, 2, 3] do
  puts num
end
```

**Good:**
```ruby
[1, 2, 3].each { |num| puts num }
```

---
## 5. Prefer `map` Over `each` for Transformations
If you are transforming an array, use `map` instead of `each`.

**Bad:**
```ruby
squared_numbers = []
[1, 2, 3].each { |n| squared_numbers << n**2 }
```

**Good:**
```ruby
squared_numbers = [1, 2, 3].map { |n| n**2 }
```

---
## 6. Use Safe Navigation Operator (`&.`) to Avoid `nil` Errors
Instead of checking `nil?` manually, use `&.`.

**Bad:**
```ruby
email = user && user.profile && user.profile.email
```

**Good:**
```ruby
email = user&.profile&.email
```

---
## 7. Use `begin...rescue` for Exception Handling

**Bad:**
```ruby
def fetch_data
  response = external_api_call
  response[:data]
rescue StandardError
  puts "An error occurred"
end
```

**Good:**
```ruby
def fetch_data
  response = external_api_call
  response[:data]
rescue StandardError => e
  puts "Error: #{e.message}"
end
```
Using `rescue StandardError => e` helps track errors with more clarity.

---
## 8. Use Lambdas for Short, Reusable Blocks
Lambdas are great for defining short functions.

**Bad:**
```ruby
def double(n)
  n * 2
end

puts double(5)
```

**Good:**
```ruby
double = ->(n) { n * 2 }
puts double.call(5)
```
Lambdas are useful when passing behavior as an argument to methods.

---
## 9. Use `RuboCop` to Enforce Coding Standards
[RuboCop](https://github.com/rubocop/rubocop) is a Ruby linter that enforces best practices automatically.

To install:
```sh
gem install rubocop
```
To check your code:
```sh
rubocop my_file.rb
```
It suggests improvements and helps keep your code consistent.

---
## 10. Prefer `freeze` for Immutable Constants
Prevent accidental modification of constants using `freeze`.

**Bad:**
```ruby
CURRENCIES = ["USD", "EUR", "GBP"]
CURRENCIES << "JPY"
```

**Good:**
```ruby
CURRENCIES = ["USD", "EUR", "GBP"].freeze
```

---
## Conclusion
By following these best practices, you can write **clean, efficient, and maintainable** Ruby code. Using tools like **RuboCop**, keeping methods small, using idiomatic syntax, and leveraging **lambdas** can make a significant difference in code quality.

Reference [The Ruby Style Guide](https://rubystyle.guide/) & [Rubocop](https://github.com/rubocop/rubocop)