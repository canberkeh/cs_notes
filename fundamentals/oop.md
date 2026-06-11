# Object-Oriented Programming in Python

Object-Oriented Programming, or OOP, is a way to organize code around objects.
An object is something that has data and behavior.

For example, a bank account has:

- Data: owner, balance
- Behavior: deposit money, withdraw money, show balance

In Python, we use a `class` as the blueprint, and we create objects from that
class.

## Classes and Objects

A class is a blueprint. An object is a real thing created from that blueprint.

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        if amount > self.balance:
            print("Insufficient funds")
            return

        self.balance -= amount

    def show_balance(self):
        print(f"{self.owner}'s balance is ${self.balance}")
```

Usage:

```python
account = BankAccount("Aylin", 100)

account.deposit(50)
account.withdraw(30)
account.show_balance()
```

Output:

```text
Aylin's balance is $120
```

Here:

- `BankAccount` is the class.
- `account` is an object.
- `owner` and `balance` are object data.
- `deposit`, `withdraw`, and `show_balance` are object behavior.

## The `__init__` Method

`__init__` runs when a new object is created.

```python
def __init__(self, owner, balance=0):
    self.owner = owner
    self.balance = balance
```

This stores data inside the object.

```python
account = BankAccount("Aylin", 100)
```

That means:

```text
Create a BankAccount object.
Set owner to "Aylin".
Set balance to 100.
```

## What Is `self`?

`self` means "this specific object."

```python
aylin_account = BankAccount("Aylin", 100)
mehmet_account = BankAccount("Mehmet", 500)
```

Each object has its own data:

```python
print(aylin_account.balance)   # 100
print(mehmet_account.balance)  # 500
```

`self.balance` means "the balance that belongs to this specific account."

## Real-World Example: Website User

Imagine you are building a website. Each user has a username, an email, and a
login state.

```python
class User:
    def __init__(self, username, email):
        self.username = username
        self.email = email
        self.is_logged_in = False

    def login(self):
        self.is_logged_in = True
        print(f"{self.username} logged in")

    def logout(self):
        self.is_logged_in = False
        print(f"{self.username} logged out")
```

Usage:

```python
user = User("codexfan", "user@example.com")

user.login()
print(user.is_logged_in)

user.logout()
print(user.is_logged_in)
```

OOP helps here because each user object owns its own username, email, and login
state.

## The Four Main Pillars of OOP

The four big OOP ideas are:

- Encapsulation
- Abstraction
- Inheritance
- Polymorphism

## Encapsulation

Encapsulation means keeping data and behavior together, while protecting the
internal state from invalid changes.

This is risky:

```python
account.balance = -999999
```

A better version controls how the balance changes:

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self._balance = balance

    def deposit(self, amount):
        if amount <= 0:
            print("Deposit must be positive")
            return

        self._balance += amount

    def withdraw(self, amount):
        if amount <= 0:
            print("Withdrawal must be positive")
            return

        if amount > self._balance:
            print("Insufficient funds")
            return

        self._balance -= amount

    def get_balance(self):
        return self._balance
```

The underscore in `_balance` means "this is internal; do not change it
directly."

Python does not strictly block access, but this convention tells other
programmers to use the methods instead.

Encapsulation answers this question:

```text
How do I keep an object's data valid and organized?
```

## Abstraction

Abstraction means exposing the important actions while hiding the complicated
internal details.

Real-world example: when you drive a car, you use the steering wheel, brake
pedal, gas pedal, and gear selector. You do not manually control fuel injection,
engine timing, brake pressure distribution, or cooling.

The car gives you a simple interface.

```python
class Car:
    def start(self):
        self._check_engine()
        self._inject_fuel()
        self._ignite()
        print("Car started")

    def accelerate(self):
        print("Car is moving faster")

    def brake(self):
        print("Car is slowing down")

    def _check_engine(self):
        print("Checking engine")

    def _inject_fuel(self):
        print("Injecting fuel")

    def _ignite(self):
        print("Igniting engine")
```

Usage:

```python
car = Car()
car.start()
car.accelerate()
car.brake()
```

The outside code only needs to know:

```python
car.start()
```

It does not need to know every internal step required to start the engine.

Abstraction answers this question:

```text
How do I make this object easy to use?
```

Encapsulation and abstraction are related, but they are not the same:

- Encapsulation protects and organizes internals.
- Abstraction hides complexity behind a simple interface.

## Inheritance

Inheritance lets one class reuse and extend another class.

Real-world example: different types of employees.

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def describe(self):
        print(f"{self.name} earns ${self.salary}")
```

Now create a manager:

```python
class Manager(Employee):
    def __init__(self, name, salary, department):
        super().__init__(name, salary)
        self.department = department

    def describe(self):
        print(f"{self.name} manages {self.department} and earns ${self.salary}")
```

Usage:

```python
employee = Employee("Ali", 40000)
manager = Manager("Zeynep", 70000, "Engineering")

employee.describe()
manager.describe()
```

`Manager` inherits from `Employee`, but it adds a department and customizes
`describe`.

Inheritance answers this question:

```text
How do I reuse common behavior in related classes?
```

## Polymorphism

Polymorphism means different objects can respond to the same method name in
their own way.

```python
class Dog:
    def speak(self):
        print("Woof")


class Cat:
    def speak(self):
        print("Meow")


class Bird:
    def speak(self):
        print("Tweet")
```

Usage:

```python
animals = [Dog(), Cat(), Bird()]

for animal in animals:
    animal.speak()
```

Each object has a `speak` method, but each object behaves differently.

Polymorphism answers this question:

```text
How can different objects share the same interface but behave differently?
```

## URL Shortener Example

This project is a URL shortener, so here is a domain-specific example.

```python
class ShortUrl:
    def __init__(self, original_url, short_code):
        self.original_url = original_url
        self.short_code = short_code
        self.clicks = 0

    def visit(self):
        self.clicks += 1
        return self.original_url

    def stats(self):
        return {
            "short_code": self.short_code,
            "original_url": self.original_url,
            "clicks": self.clicks,
        }
```

Usage:

```python
link = ShortUrl("https://example.com/very/long/link", "abc123")

print(link.visit())
print(link.visit())
print(link.stats())
```

Output:

```python
https://example.com/very/long/link
https://example.com/very/long/link
{'short_code': 'abc123', 'original_url': 'https://example.com/very/long/link', 'clicks': 2}
```

The shortened URL owns its data and knows how to update itself.

## URL Shortener With Abstraction

Here is a higher-level URL shortener class:

```python
class UrlShortener:
    def shorten(self, long_url):
        code = self._generate_code()
        self._save_mapping(code, long_url)
        return f"https://short.ly/{code}"

    def _generate_code(self):
        return "abc123"

    def _save_mapping(self, code, long_url):
        print(f"Saved {code} -> {long_url}")
```

Usage:

```python
shortener = UrlShortener()
short_url = shortener.shorten("https://example.com/some/very/long/url")

print(short_url)
```

The outside world just calls:

```python
shortener.shorten("https://example.com/some/very/long/url")
```

It does not need to know how the code is generated or where the mapping is
stored.

## Summary

| Concept | Meaning | Real-World Example |
| --- | --- | --- |
| Class | A blueprint | `BankAccount` blueprint |
| Object | A real instance of a class | Aylin's actual bank account |
| Encapsulation | Keep data valid and protected | Balance changes only through `deposit` and `withdraw` |
| Abstraction | Hide complexity behind simple actions | `car.start()` hides engine details |
| Inheritance | Reuse and extend a parent class | `Manager` extends `Employee` |
| Polymorphism | Same method name, different behavior | `Dog.speak`, `Cat.speak`, `Bird.speak` |

## Practice Exercise

Build a `Product` class for an online shop.

Requirements:

- Store `name`, `price`, and `stock`.
- Add a `buy(quantity)` method.
- Add a `restock(quantity)` method.
- Prevent buying more than the available stock.
- Prevent negative quantities.

Starter code:

```python
class Product:
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    def buy(self, quantity):
        pass

    def restock(self, quantity):
        pass
```

Try creating a few products and simulating purchases.
