# SQL Alchemy

## Object Relational Mapper
The ORM is a system of mapping python classes to SQL tables and class instances to rows and synchronizing changes of state between the two.  The ORM is an built on a lower-level system called SQLAlchemy Expression Language, a system of representing primitive relational DB constructs.  SQLAlchemy clients should generally only need to operate on the ORM layer unless very specific DB interactions are required.

Through the `Engine` class, SQLAlchemy interfaces with the underlying database through a dialect (sqlite, postgres, etc.).  The SQLite dialect leverages the built-in Python `sqlite3` module.

To use the ORM, must describe the underlying tables and then define classes to map to them.  This is simultaneously accomplished using the Declarative system, where a single base _class_, inherited by all mapped classes in the application.
```python
Base = declarative_base()
class Product(Base):        # Product table
    __tablename__ = 'products'  # required class attribute
    sku = Column(Integer, primary_key=True)
    ...
```
Declarative will replace `Column` class attributes with descriptors which allow database access.
