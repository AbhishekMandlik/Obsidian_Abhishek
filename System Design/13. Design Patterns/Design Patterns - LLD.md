## Singleton
Example we have 3 components, and these need to access some resource.
Instead of all the Components having their own network instances Singleton's idea is to put a central resource in a single instance of that resource than duplicating it.
Only one instance: Single point of access for the resource
It Uses: 
1. Network manager
2. Database access
3. Logging
4. Utility classes

Disadvantages:
1. It breaks the single responsibility principle. It means that a singleton will basically manage its own state will allow others to create only one instance. (The reason is that a Singleton class often has **two responsibilities**)
	1. It performs it's actual job.
	2. It controls it's own instantiation.
	3. So class is responsible for both DB functionality + Instance/lifecycle management which conflicts Single Responsibility Principle(SRP).
2. A particular class should create a component when required but singleton creates it's own instance.
3. Testability issue: Couple tightly, scalability issue.
4. State for life: Once a singleton has been instantiated you can only use that particular instance.


```
Code:
class Database:
    def __init__(self):
        print("Creating DB connection")

db1 = Database()
db2 = Database()
db3 = Database()

### Here we have 3 instances of a Database. with singleton we would want
ONE DB onject. 
db1 = Database.get_instance()
db2 = Database.get_instance()

print(db1 is db2) 
### TRUE
```

```
                    Singleton
                       │
              ┌────────┴────────-┐
              │ DatabaseManager  │
              └────────┬─────────┘
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
   UserService   OrderService   PaymentService
```

In a multi-threaded environment when we run multiple threads for a singleton, multiple instance of the resource are created. To solve this, we lock the singleton function for one thread at a time to prevent race conditions. We lock the function which checks if the instance is present or not so that during race conditions multiple instances are not created.

## Factory 
>Separate object creation from object usage, so the client doesn't need to know which concrete class is being created.
1. Requirements Change in our code depending on what new functionality is needed, or we need to refactor the code. Factory method will solve the problem as this refactoring affects the creator of the object and user of the object. It will create separation in those two and allow us to change our code more easily.
2. Dynamic Switching: Multiple DB and we would want to switch in between those Db based on certain requirements. So we do not want tight coupling between them.
3. Separation of concerns: Separate the creation from the utilisation. 
```
### Without a Factory Method...
class OrderService:
    def __init__(self):
        self.db = PostgreSQLDatabase()
        
### Here OrderService knows about PostgreSQLDatabase but later we would ### like to change it to some other function then what would we do?

### SOMETHING LIKE THIS COMES TO OUR MIND
class OrderService:
    def __init__(self, db_type):
        if db_type == "postgres":
            self.db = PostgreSQLDatabase()
        elif db_type == "mysql":
            self.db = MySQLDatabase()
        elif db_type == "mongo":
            self.db = MongoDatabase()

### BUT Here the problem is now OrderService is responsible for two  things:
1. Using the DB
2. Deciding which DB to solve.
This will be solved by Factory method
```
### How is this done
1. Design logic is hidden from the client. It only knows how to use it.
2. Multiple types of objects can be created by the client without actually knowing which exact type it is actually using.
3. Creation is removed from a client.
4. Useful for frequent code changes.
![[Pasted image 20260818150810.png]]All the DBs have one interface. So we don't have any idea what is going on behind.

### Components Required
1. Interface or abstract class that defines the common functionality.
2. Interface Implementations.
3. Factory class that instantiates the right implementation.

```
// We will separate creation from usage
We create one common interface://
class Database:
    def connect(self):
        pass

    def save(self, data):
        pass
        
//Then://
class PostgreSQLDatabase(Database):
    def connect(self):
        ...
    def save(self, data):
        ...

class MongoDatabase(Database):
    def connect(self):
        ...
    def save(self, data):
        ...

Database is the abstraction. It says that any class that behaves lik DB must provide connect() and save(). It does not say how those operations should be working.
We have this abstraction because now our application can work witth any DB implementation all of them provide:
1. connect()
2. save()
This is polymorphism.
Polymorphism allows you to treat different concrete objects through the same interface, while each object provides its own implementation of the behavior.
//The client knows that he has a DB but does not need to know what DB he has in the back, if this is Mongo, PostgreSQL//
```

### The Factory
```
class DatabaseFactory:

    @staticmethod
    def create_database(db_type):
        if db_type == "postgres":
            return PostgreSQLDatabase()

        if db_type == "mysql":
            return MySQLDatabase()

        if db_type == "mongo":
            return MongoDatabase()

        raise ValueError("Unsupported database")
# Client in main function:
db = DatabaseFactory.create_database("postgres")
db.connect()
db.save(data)

# Client knows what he want like(postgres) but does not create it's instance, Factory handles it.
```

### Why is this useful?
```
# Without Factory:
Service A → PostgreSQLDatabase()
Service B → PostgreSQLDatabase()
Service C → PostgreSQLDatabase()
Service D → PostgreSQLDatabase()

# With Factory:
Service A ─┐
Service B ─┤
Service C ─┼──> DatabaseFactory
Service D ─┘          │
                      ↓
                  PostgreSQL
You don't have to change at many places...
It separates the code that decides which concrete object to instantiate from the code that consumes that object.
```

### Main Factor: Separation of Concerns:
```
Without Factory:
OrderService
│
├── decide which DB
├── instantiate DB
└── use DB

With Factory:
OrderService
│
└── use DB
&
DatabaseFactory
│
└── decide + create DB

So:

                    Application
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
         Factory              Services
              │                   │
              ↓                   ↓
       Create Database       Use Database

Creation and utilisation are separated.
```


## Abstract Factory
1. It is factory that creates factory.
2. It provides a way to access functionality without caring about the implementation.
3. One level of abstraction over the factory pattern.
4. Separation of concerns.
5. Allows for testability.

### Multiple related products
Imagine we are building an application that supports different cloud providers.
```
For **AWS**, we need:
AWS
 ├── Storage
 └── Database

For **Azure**, we need:
Azure
 ├── Storage
 └── Database

For **GCP**, we need:
GCP
 ├── Storage
 └── Database
```
We don't need to create a DB we need to create a family of related object.
Like in factory we were doing only for creating a DB now we have many more things.
### Architecture
```
We can define two product interfaces:

             Storage
                ↑
       ┌────────┼────────┐
       │        │        │
     AWS      Azure      GCP

             Database
                ↑
       ┌────────┼────────┐
       │        │        │
     AWS      Azure      GCP

Then we define an Abstract Factory:

              CloudFactory
                  |
          ┌───────┴────────┐
          ↓                ↓
    create_storage()   create_database()

And concrete factories:

                CloudFactory
                     ↑
          ┌──────────┼──────────┐
          │          │          │
      AWSFactory  AzureFactory GCPFactory

Each factory creates a matching family of products.
```

```
// EXAMPLE
from abc import ABC, abstractmethod


# ---------- Product Interfaces ----------

class Database(ABC):

    @abstractmethod
    def connect(self):
        pass


class Storage(ABC):

    @abstractmethod
    def upload(self, file):
        pass


# ---------- AWS Products ----------

class AWSDatabase(Database):

    def connect(self):
        print("Connected to AWS RDS")


class AWSStorage(Storage):

    def upload(self, file):
        print(f"Uploading {file} to AWS S3")


# ---------- Azure Products ----------

class AzureDatabase(Database):

    def connect(self):
        print("Connected to Azure SQL")


class AzureStorage(Storage):

    def upload(self, file):
        print(f"Uploading {file} to Azure Blob Storage")


# ---------- Abstract Factory ----------

class CloudFactory(ABC):

    @abstractmethod
    def create_database(self):
        pass

    @abstractmethod
    def create_storage(self):
        pass


# ---------- Concrete Factories ----------

class AWSFactory(CloudFactory):

    def create_database(self):
        return AWSDatabase()

    def create_storage(self):
        return AWSStorage()


class AzureFactory(CloudFactory):

    def create_database(self):
        return AzureDatabase()

    def create_storage(self):
        return AzureStorage()


# ---------- Client ----------

def run_application(factory: CloudFactory):

    database = factory.create_database()
    storage = factory.create_storage()

    database.connect()
    storage.upload("data.csv")


def main():

    cloud = "aws"

    if cloud == "aws":
        factory = AWSFactory()

    elif cloud == "azure":
        factory = AzureFactory()

    run_application(factory)


if __name__ == "__main__":
    main()
```


### Main difference between factory and Abstract Factory
```
Factory
Usually focuses on creating one type of product.
DatabaseFactory
      ↓
Database
Abstract Factory
Creates multiple related products.

              CloudFactory
             /            \
            ↓              ↓
        Database         Storage
            ↓              ↓
       AWSDatabase      AWSStorage

So:
Factory = one product family/type
Abstract Factory = multiple related products that should work together
```


## Builder
1. Used when we have multiple parameters to initialise.
2. For Many parameters, it is impractical to build all constructors, 5 Parameter combination ->120 constructors.
3. Optional Parameters, can exclude while building object.
4. Should be easy to read.
The basic idea is that it separated the step-by-step construction of a complex object from the final object itself. It is especially useful when an object has many parameters especially optional ones.
The main problem over here is: tightly coupling of things for eg.
```
Suppose you have an Employee:
class Employee:
    def __init__(
        self,
        name,
        age,
        department,
        salary,
        email,
        phone,
        address
    ):
        ...
# Creating it becomes difficult to read:
employee = Employee(
    "Abhishek",
    25,
    "Data Science",
    100000,
    "abc@example.com",
    "9999999999",
    "Chennai"
)
# Looking at this code, it isn't immediately obvious:
# 100000 → salary?
# 9999999999 → phone?
# Chennai → address?
# You have to remember the order of parameters.
# And this gets worst with optional parameters as the code will become very difficult to understand.
```

### How builder will solve this?
```
employee = (
    EmployeeBuilder()
    .set_name("Abhishek")
    .set_age(25)
    .set_department("Data Science")
    .set_salary(100000)
    .set_email("abc@example.com")
    .set_address("Chennai")
    .build()
)
```

```
class Employee:

    def __init__(
        self,
        name,
        age=None,
        department=None,
        salary=None,
        email=None
    ):
        self.name = name
        self.age = age
        self.department = department
        self.salary = salary
        self.email = email

    def __str__(self):
        return (
            f"Employee(name={self.name}, "
            f"age={self.age}, "
            f"department={self.department}, "
            f"salary={self.salary}, "
            f"email={self.email})"
        )


class EmployeeBuilder:

    def __init__(self):
        self.name = None
        self.age = None
        self.department = None
        self.salary = None
        self.email = None

    def set_name(self, name):
        self.name = name
        return self

    def set_age(self, age):
        self.age = age
        return self

    def set_department(self, department):
        self.department = department
        return self

    def set_salary(self, salary):
        self.salary = salary
        return self

    def set_email(self, email):
        self.email = email
        return self

    def build(self):
        if self.name is None:
            raise ValueError("Name is required")

        return Employee(
            name=self.name,
            age=self.age,
            department=self.department,
            salary=self.salary,
            email=self.email
        )

employee = (
	EmployeeBuilder()
	.set_name("Abhishek")
	.set_age("22")
	.set_salary("10000")
)

We did not specify email, department this will simply leaves tham as None
```
### Method chaining and fluent interface
```
###
return self over above function
This is about method chaining....

Without return self:
builder = EmployeeBuilder()
builder.set_name("Abhishek")
builder.set_department("Data Science")
builder.set_salary(100000)

With return self:
we can do 
employee = (
	EmployeeBuilder()
	.set_name("Abhishek")
	.set_age("22")
	.set_salary("10000")
)
This is called fluent interface.
```

### What would .build() function do?
It will create the actual Employee, until then it was just collecting all the parameters.
## Decorators
1. It is also called the wrapper pattern. It allows us to update a behaviour of the class that we do not have authority over. We can add personalisation some functional changes to the third-party class.
2. Without altering existing code

### The problem Decorator solves

Imagine you are using a third-party class:
```
class EmailService:
    def send(self, message):
        print(f"Sending email: {message}")
```

You don't own this code and now want to add.
- Logging
- Encryption
- Authentication
- Metrics

### The basic idea

```
Without Decorator:
Client
  ↓
EmailService
  ↓
send()

With Decorator:
Client
  ↓
LoggingDecorator
  ↓
EmailService
  ↓
send()

The decorator adds behavior before/after calling the original object.
```

### Simple example
```
First, define a common interface:
from abc import ABC, abstractmethod
class Notification(ABC):
    @abstractmethod
    def send(self, message):
        pass

Our original implementation:

class EmailNotification(Notification):
    def send(self, message):
        print(f"Sending email: {message}")
```
Now we want logging.
Instead of changing EmailNotification, create a decorator:
```
class LoggingDecorator(Notification):
    def __init__(self, notification):
        self.notification = notification
    def send(self, message):
        print("Logging notification...")
        self.notification.send(message)
```
Now:
```
email = EmailNotification()
logged_email = LoggingDecorator(email)
logged_email.send("Hello Abhishek")
```
Output:
```
Logging notification...
Sending email: Hello Abhishek
```
### We can stack decorators
```
You can stack decorators.

email = EmailNotification()
email = LoggingDecorator(email)
email = EncryptionDecorator(email)
email = AuthenticationDecorator(email)
email.send("Hello")
```
Conceptually:
```
Client
  ↓
AuthenticationDecorator
  ↓
EncryptionDecorator
  ↓
LoggingDecorator
  ↓
EmailNotification
```
Each decorator adds one responsibility.

## Observer
### Main problem?
In this scenario: If there is a time intensive process and many Users keep on asking it if the process is done and what output has been generated then this would be resource intensive. Instead what we can do is implement a object called as registry in between these...  So that it can notify the user when an event has occurred.. similar to event driven architecture.
1. It Defines a subscription mechanism, Similar to how we subscribe to a news paper.
2. Notify multiple objects simultaneously.
3. One to many relationship.

> Observer defines a one-to-many relationship where one object (the Subject) maintains a list of dependent objects (Observers) and automatically notifies them when its state changes.

```
                  Process
                 (Subject)
                     │
              maintains list
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        User A      User B     User C
       Observer    Observer   Observer
```
When the process completes:
```
Process completes
       ↓
Subject notifies
       ↓
┌──────┼──────┐
↓      ↓      ↓
A      B      C
```
### Two Main Components:
1. Subject: 
	1. The object whose state changes. It maintains a list of observers.
	2. Example: Long Running Process, Order.
2. Observer:
	1. The object that wants to know when something changes.
	2. Example: User, UI.
Subject has observers list and it notifies as and when event is done.

### Basic code example
Let's build a simple process.
```
from abc import ABC, abstractmethod
# Observer interface
class Observer(ABC):


    @abstractmethod
    def update(self, result):
        pass
```
Now create observers:
```
class User(Observer):
    def __init__(self, name):
        self.name = name

    def update(self, result):
        print(f"{self.name} received result: {result}")
```
Now the Subject:
```
class Process:
    def __init__(self):
        self.observers = []
        self.result = None


    def subscribe(self, observer):
        self.observers.append(observer)


    def unsubscribe(self, observer):
        self.observers.remove(observer)


    def notify(self):
        for observer in self.observers:
            observer.update(self.result)


    def complete_process(self, result):
        self.result = result
        print("Process completed!")
        self.notify()

```
Using it:
```
def main():
    process = Process()

    user1 = User("Abhishek")
    user2 = User("Rahul")
    user3 = User("Priya")
    
    process.subscribe(user1)
    process.subscribe(user2)
    process.subscribe(user3)
    
    # Long-running process finishes
    process.complete_process("Model generated successfully")
    
if __name__ == "__main__":
    main()
```
Output:
```
Process completed!
Abhishek received result: Model generated successfully
Rahul received result: Model generated successfully
Priya received result: Model generated successfully
```
Nobody asked if the process is done, The process notified everyone based on if it is subscribed or not.

### Functionalities:
SUBSCRIBE():
```
Process.subscribe(user1)
This means that User 1 wants to recieve notifications from this process.
```
NOTIFY():
```
self.notify()
This will notify all the users that have been subscribed to the process.
```

> Registry is not essential part of the observer pattern, The subject itself maintains the list. It was just to give an Idea of what is really happening in the backend.

### One important disadvantage
Too many observers can make notification behaviour difficult to reason about.
```
For example:

Subject
  ↓
Observer A
  ↓
changes something
  ↓
Observer B
  ↓
triggers another event
  ↓
Observer C

You can end up with complex event chains.
```

## State
Thinking of your program as a state machine. It simply means that your algorithm can only be in a certain number of pre-defined states and can't deviate from those.
1. Object changes it's behaviour based on an internal state.
2. At any moment, there's a finite number of stated a program can be in.
3. State can be encapsulated in an object.
> An object behaviour changes when its internal state changes, as if the object changed its class.

It is like a state machine:
1. States.
2. Transitions.
3. Actions/ Behaviour.
```
             confirm()
PLACED ─────────────────→ CONFIRMED
   ↑                          │
   │                          │ ship()
   │                          ↓
   │                       SHIPPED
   │                          │
   │                       deliver()
   │                          ↓
   └───────────────────── DELIVERED
   
Object can be only in certain valid states.
The same method behaves differently based on which state you are currently in. Like you can't cancel order unless you are in confirmed state.
```

### The naive implementation
You could implement this with a giant `if/elif`:
```
class Order:
    def __init__(self):
        self.state = "PLACED"

    def cancel(self):
        if self.state == "PLACED":
            print("Order cancelled")
        elif self.state == "CONFIRMED":
            print("Order cancelled")
        elif self.state == "SHIPPED":
            print("Cannot cancel shipped order")
        elif self.state == "DELIVERED":
            print("Cannot cancel delivered order")
```
But imagine your application has:
```
PLACED
CONFIRMED
PAYMENT_PENDING
PAID
PACKED
SHIPPED
DELIVERED
CANCELLED
RETURNED
REFUNDED
```
Now every method could become:
```
if state == ...
elif state == ...
elif state == ...
elif state == ...
```
You end up with a huge amount of conditional logic.
That's where the State pattern helps.
### State Pattern:
Instead of putting all behavior inside Order, we create separate state objects.

```
                    Order
                      │
                    state
                      │
         ┌────────────┼────────────┐
         ↓            ↓            ↓
     PlacedState  ShippedState  DeliveredState
```
Each state knows how to behave.
### Simple Example
First, define the State interface:
```
from abc import ABC, abstractmethod
class OrderState(ABC):
    @abstractmethod
    def cancel(self, order):
        pass
```
Now create concrete states.
```
# Placed State
class PlacedState(OrderState):
    def cancel(self, order):
        print("Order cancelled")


# Shipped State
class ShippedState(OrderState):
    def cancel(self, order):
        print("Cannot cancel shipped order")


# Delivered State
class DeliveredState(OrderState):
    def cancel(self, order):
        print("Cannot cancel delivered order")

```
Now our Order:
```
class Order:
    def __init__(self):
        self.state = PlacedState()
    def cancel(self):
        self.state.cancel(self)
    def set_state(self, state):
        self.state = state
```
Using it:
```
def main():
    order = Order()
    order.cancel()
    # Order cancelled
    order.set_state(ShippedState())
    order.cancel()
    # Cannot cancel shipped order
    order.set_state(DeliveredState())
    order.cancel()
    # Cannot cancel delivered order

if __name__ == "__main__":
    main()
```
Notice what happened.
The caller always uses: order.cancel()
But the behaviour changes depending on: order.state()
## Strategy
1. Class behaviour or algorithm can be changed at run-time.
2. Objects contain algorithm logic.
3. We can add or remove strategies on fly, without changing the program structure.

> Strategy lets you define multiple interchangeable algorithms, encapsulate each algorithm in its own class, and choose which algorithm to use at runtime.

The Key word is algorithm

### Create a strategy Interface:
```
from abc import ABC, abstractmethod
class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, weight):
        pass
```

### Create Different Strategies
Each strategy contains it's own method.
```
# Standard Shipping
class StandardShipping(ShippingStrategy):
    def calculate(self, weight):
        return weight * 10
  
# Express Shipping
class ExpressShipping(ShippingStrategy):
    def calculate(self, weight):
        return weight * 20

# International Shipping
class InternationalShipping(ShippingStrategy):
    def calculate(self, weight):
        return weight * 50
```
Now:
```
              ShippingStrategy
                    │
          ┌─────────┼──────────┐
          ↓         ↓          ↓
      Standard    Express   International
      Strategy    Strategy    Strategy
```
Each object contains a different algorithm. Objects contain algorithm logic.

### The main class using strategy
Create a class to order:
```
class Order:

    def __init__(self, shipping_strategy):
        self.shipping_strategy = shipping_strategy

    def calculate_shipping(self, weight):
        return self.shipping_strategy.calculate(weight)
```
Order does not contain:
1. Standard Calculation.
2. Express Calculation.
3. International Calculation.
Instead it says that give me shipping strategy and I will use it.
### We can change  the strategy during runtime.
```
def main():
    order = Order(StandardShipping())
    print(order.calculate_shipping(10))
```
If customer wants express shipping We can change the strategy.
```
order.shipping_strategy = ExpressShipping()
```

### Small code:
```
from abc import ABC, abstractmethod
# Strategy interface
class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, weight):
        pass


# Concrete strategies
class StandardShipping(ShippingStrategy):
    def calculate(self, weight):
        return weight * 10


class ExpressShipping(ShippingStrategy):
    def calculate(self, weight):
        return weight * 20


class InternationalShipping(ShippingStrategy):
    def calculate(self, weight):
        return weight * 50


# Context
class Order:
    def __init__(self, strategy):
        self.strategy = strategy
    def set_strategy(self, strategy):
        self.strategy = strategy
    def calculate_shipping(self, weight):
        return self.strategy.calculate(weight)


def main():

    order = Order(StandardShipping())
    print("Standard:", order.calculate_shipping(10))
    order.set_strategy(ExpressShipping())
    print("Express:", order.calculate_shipping(10))
    order.set_strategy(InternationalShipping())
    print("International:", order.calculate_shipping(10))


if __name__ == "__main__":
    main()
```

We can add more methods/objects on the fly without changing the order function.

> **Strategy encapsulates interchangeable algorithms behind a common interface and allows the client to select or change the algorithm at runtime.**




# THE END