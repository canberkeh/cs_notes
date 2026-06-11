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

The underscore in _balance means "this is internal; do not change it directly."
Using __balance (double underscore) makes it even harder to access from outside.

Python does not strictly block access, but this convention tells other
programmers to use the methods instead.

Encapsulation answers this question:

```text
How do I keep an object's data valid and organized?
```

## Abstraction

Abstraction means defining what an object does without exposing how it does it.


Real-world example: when you press a button on a coffee maker, you do not manually
grind the beans, heat the water, or control the brewing time.
The machine gives you a simple interface: one button, one result.

In Python, we use abstract classes to enforce this idea in code.

```python
from abc import ABC, abstractmethod


class Machine(ABC):
    @abstractmethod
    def run(self):
        pass


class CoffeeMaker(Machine):
    def run(self):
        print("Coffee is being prepared...")


class WashingMachine(Machine):
    def run(self):
        print("Laundry is being washed...")
```

Usage:

```python
coffee = CoffeeMaker()
coffee.run()

laundry = WashingMachine()
laundry.run()
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
        super().__init__(name, salary)  # calls Employee's __init__, avoids repeating code
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