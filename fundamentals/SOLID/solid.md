SOLID is a set of 5 design principles that help you write code that is easier to change, test, and extend.


1. S — Single Responsibility Principle

A class should have one reason to change.

Bad example:

```python
class Invoice:
    def calculate_total(self):
        pass

    def save_to_database(self):
        pass

    def send_email(self):
        pass
```

This class does too much:

calculates invoice
saves invoice
sends email

Better:

```python
class Invoice:
    def calculate_total(self):
        pass
```

```python
class InvoiceRepository:
    def save(self, invoice):
        pass
```

```python
class EmailService:
    def send_invoice(self, invoice):
        pass
```

Real world example:
A chef cooks, a cashier takes payments, and a delivery person delivers. One person doing all jobs makes the system harder to manage.

2. O — Open/Closed Principle

Code should be open for extension, but closed for modification.

Bad example:

```python
class DiscountCalculator:
    def calculate(self, customer_type, price):
        if customer_type == "regular":
            return price * 0.95
        elif customer_type == "vip":
            return price * 0.80
        elif customer_type == "student":
            return price * 0.90
```

Every new discount type requires changing this class.

Better:

```python
class Discount:
    def apply(self, price):
        return price


class RegularDiscount(Discount):
    def apply(self, price):
        return price * 0.95


class VipDiscount(Discount):
    def apply(self, price):
        return price * 0.80


class StudentDiscount(Discount):
    def apply(self, price):
        return price * 0.90


class DiscountCalculator:
    def calculate(self, discount, price):
        return discount.apply(price)
```

Usage:

```python
calculator = DiscountCalculator()
final_price = calculator.calculate(VipDiscount(), 100)
print(final_price)  # 80
```

Now adding a new discount does not require modifying existing logic.

3. L — Liskov Substitution Principle

A child class should be usable wherever the parent class is expected.

Bad example:

```python
class Bird:
    def fly(self):
        pass


class Sparrow(Bird):
    def fly(self):
        print("Flying")


class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins cannot fly")
```

Problem: Penguin is a Bird, but it breaks the expected behavior of Bird.

Better:

```python
class Bird:
    pass


class FlyingBird(Bird):
    def fly(self):
        pass


class Sparrow(FlyingBird):
    def fly(self):
        print("Flying")


class Penguin(Bird):
    def swim(self):
        print("Swimming")
```

Real world example:
Not every bird can fly, so “fly” should not belong to the general Bird abstraction.

4. I — Interface Segregation Principle

A class should not be forced to implement methods it does not need.

Bad example:

```python
class Worker:
    def work(self):
        pass

    def eat(self):
        pass


class Robot(Worker):
    def work(self):
        print("Robot working")

    def eat(self):
        raise Exception("Robot does not eat")
```

Better:

```python
class Workable:
    def work(self):
        pass


class Eatable:
    def eat(self):
        pass


class HumanWorker(Workable, Eatable):
    def work(self):
        print("Human working")

    def eat(self):
        print("Human eating")


class RobotWorker(Workable):
    def work(self):
        print("Robot working")
```

Real world example:
A printer should not be forced to support scanning if it is only a printer.

5. D — Dependency Inversion Principle

High-level code should depend on abstractions, not concrete implementations.

Bad example:

```python
class MySQLDatabase:
    def save(self, data):
        print("Saving to MySQL")


class UserService:
    def __init__(self):
        self.database = MySQLDatabase()

    def create_user(self, user):
        self.database.save(user)
```

Problem: UserService is tightly coupled to MySQL.

Better:

```python
class Database:
    def save(self, data):
        pass


class MySQLDatabase(Database):
    def save(self, data):
        print("Saving to MySQL")


class PostgreSQLDatabase(Database):
    def save(self, data):
        print("Saving to PostgreSQL")


class UserService:
    def __init__(self, database: Database):
        self.database = database

    def create_user(self, user):
        self.database.save(user)
```

Usage:

```python
db = PostgreSQLDatabase()
service = UserService(db)
service.create_user({"name": "Ayşe"})
```

Now UserService can work with MySQL, PostgreSQL, MongoDB, or a mock database in tests.

Interview-style summary

SOLID helps us create maintainable object-oriented code.

Single Responsibility: one class, one job.
Open/Closed: add new behavior without changing old code.
Liskov Substitution: child classes should not break parent expectations.
Interface Segregation: avoid forcing classes to implement unnecessary methods.
Dependency Inversion: depend on abstractions, not concrete classes.

A good sentence for interviews:

SOLID principles help reduce tight coupling, improve testability, and make the codebase easier to extend safely.
