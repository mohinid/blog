---
layout: post
title: Design Pricinples vs Design Patterns
subtitle: Understanding the Difference
tags: [design patterns, design principles, coding practices, interview question]
comments: true
thumbnail-img: /assets/img/design-patterns.png
---

Most of the software engineering discussion starts with talking about Common Design Patterns and Principles. Well, I could not tell the exact difference between these two in an interview! Here is what I found in the after research.

### Design Principles vs Design Patterns: Understanding the Difference

Design principles and design patterns are both important concepts in software development, but they serve different purposes and focus on different aspects of the design process.

---

### **Design Principles**
Design principles are **general guidelines** or **best practices** that aim to make software design more robust, maintainable, and flexible. They are high-level concepts that guide the design process and help developers make good decisions when creating a software architecture.

#### Key Characteristics of Design Principles:
1. **High-level guidelines**: They provide a foundation or philosophy to follow when designing a system.
2. **Abstract**: They do not provide specific implementations or solutions, but rather ideas that guide you in making decisions during the development process.
3. **Goal-oriented**: The main purpose of design principles is to reduce complexity, promote reusability, and enhance the scalability of the code.
4. **Focus on maintainability**: They emphasize long-term code health, aiming to make systems easier to change, extend, and maintain.

#### Common Design Principles:
- **SOLID Principles**:
  - **S**ingle Responsibility Principle (SRP): A class should have one and only one reason to change.
  - **O**pen/Closed Principle (OCP): Software entities should be open for extension, but closed for modification.
  - **L**iskov Substitution Principle (LSP): Subtypes should be substitutable for their base types.
  - **I**nterface Segregation Principle (ISP): No client should be forced to depend on methods it does not use.
  - **D**ependency Inversion Principle (DIP): High-level modules should not depend on low-level modules; both should depend on abstractions.
  
- **DRY (Don't Repeat Yourself)**: Avoid duplicating code, aim to reuse code effectively.
  
- **KISS (Keep It Simple, Stupid)**: Simplicity is key; avoid overcomplicating solutions.

- **YAGNI (You Aren’t Gonna Need It)**: Don’t add functionality until it is absolutely necessary.

#### Example:
- **Single Responsibility Principle (SRP)**:
  If you have a class `Invoice`, it should only handle things related to invoicing, not reporting or printing. Separating concerns keeps your code clean and easier to manage.
  
---

### **Design Patterns**
Design patterns are **proven solutions** to **common problems** in software design. They are templates for solving recurring issues in software development. While design principles guide how to write good code in general, design patterns provide specific, reusable solutions that have been established over time in the programming community.

#### Key Characteristics of Design Patterns:
1. **Specific solutions**: Design patterns provide detailed, structured approaches to solving common problems.
2. **Reusable**: They offer repeatable solutions that can be applied in different scenarios.
3. **Lower-level**: They focus on solving well-defined problems within the design, such as object creation, organization, and interaction.
4. **Proven practices**: Patterns are based on well-established practices from real-world software development experience.

#### Types of Design Patterns (According to the **Gang of Four**):
1. **Creational Patterns**: Focus on object creation mechanisms, trying to create objects in a manner suitable to the situation.
   - **Singleton**: Ensures a class has only one instance and provides a global point of access to it.
   - **Factory Method**: Defines an interface for creating objects but lets subclasses alter the type of objects that will be created.
   
2. **Structural Patterns**: Deal with object composition and the relationship between entities.
   - **Adapter**: Allows incompatible interfaces to work together.
   - **Decorator**: Attaches additional responsibilities to an object dynamically.

3. **Behavioral Patterns**: Concerned with object collaboration and responsibility.
   - **Observer**: Defines a one-to-many dependency between objects, where when one object changes state, the dependents are notified and updated.
   - **Strategy**: Allows for defining a family of algorithms and makes them interchangeable.

#### Example:
- **Observer Pattern**: 
  This pattern can be used when an object (say `WeatherStation`) needs to notify other objects (like `WeatherDisplay` or `MobileApp`) when it changes state. Instead of calling each object directly, the `WeatherStation` can notify all registered observers at once, keeping the system loosely coupled.

```ruby
class WeatherStation
  attr_accessor :observers

  def initialize
    @observers = []
  end

  def add_observer(observer)
    observers << observer
  end

  def notify_observers(temperature)
    observers.each { |observer| observer.update(temperature) }
  end
end

class WeatherDisplay
  def update(temperature)
    puts "The temperature is now #{temperature} degrees."
  end
end

station = WeatherStation.new
display = WeatherDisplay.new

station.add_observer(display)
station.notify_observers(28)
```

---

### **Key Differences**:

| Aspect                   | **Design Principles**                                 | **Design Patterns**                                    |
|--------------------------|-------------------------------------------------------|--------------------------------------------------------|
| **Definition**            | General guidelines for writing clean, maintainable code | Reusable solutions to common design problems             |
| **Abstraction Level**     | High-level, philosophical                             | Lower-level, specific implementations                   |
| **Purpose**               | Guide decision-making in software architecture        | Provide proven, repeatable solutions to recurring problems |
| **Examples**              | SOLID, DRY, KISS, YAGNI                               | Singleton, Factory, Observer, Strategy                  |
| **Reusability**           | Not directly reusable but shapes how you design       | Directly reusable as templates for solving issues       |
| **Focus**                 | Maintainability, flexibility, clarity                 | Solving recurring design challenges                     |

### Summary:
- **Design Principles**: High-level, abstract rules that guide good design choices (like SOLID, DRY).
- **Design Patterns**: Concrete solutions to specific problems (like Singleton, Factory).

Hope it helps you in choosing the right term for your next disussion!