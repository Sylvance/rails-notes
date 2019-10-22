# Active record Basics

Notes on Active Record. More details [here.](https://guides.rubyonrails.org/active_record_basics.html)

## Overview

- Activerecord is an ORM(Object Relational Mapping) framework. This simply means it is a way of connecting/mapping objects(in your code) to tables in a database.
- You don't have to write SQL since it is abstracted
- Active Record helps us do the following:

    ```
    - Represent models and their data.
    - Represent associations between these models.
    - Represent inheritance hierarchies through related models.
    - Validate models before they get persisted to the database.
    - Perform database operations in an object-oriented fashion.
    ```

## Conventions

#### Naming conventions

- Pluralize your class names to find the respective database table. e.g for a class `Book`, you should have a database table called `books`. For `BookClub` the database is seperated with underscores as `book_clubs`.

#### Schema conventions

- `Foreign keys` - should be like `item_id` or `order_id`. singular and seperated with underscore. Used in associations. 
- `Primary keys` - By default, Active Record will use an integer column named id as the table's primary key (bigint for PostgreSQL and MySQL, integer for SQLite).

## Creating Active Record Models

To create an Activerecord model subclass `ApplicationRecord` as below:
    ```
    class Product < ApplicationRecord; end
    ```

This is all. This will create a `Product` model, mapped to a `products` table at the database.

Suppose this SQL creates the products table
```
CREATE TABLE products (
   id int(11) NOT NULL auto_increment,
   name varchar(255),
   PRIMARY KEY  (id)
);
```
The products table would look something like this if it had data;

| id | name      |
| ---| --------- |
|  1 |  Lotion   |
|  2 |  Handwash |
|  3 |  Bodywash |

Thus, you would be able to write code like the following:
```
p = Product.new
p.name = "Nail polish"
puts p.name # "Nail polish"
```

## Overriding the Naming conventions

Let's say you have a database table that has a different name which is not conventional, then you can override the Model to use this database table. You will do this like so;

```
class Product < ApplicationRecord
  self.table_name = "my_products"
end
```

To override the column that should be used as the Primary key  do as below;

```
class Product < ApplicationRecord
  self.primary_key = "product_id"
end
```

## CRUD: Reading and Writing Data
- Create
    - Instantiate object like so
        ```
        product = Product.new
        product.name = "Some Book"
        product.price = 10

        or

        product = Product.new do |p|
            p.name = "Some Book"
            p.price = 10
        end
        ```
     - Save/commit object to database
        ```
        product.save
        ```

- Read
    - return a collection with all users
        `users = User.all`
    - return the first user
        `user = User.first`
    - return the first user named David
        `david = User.find_by(name: 'David')`
    - find all users named David who are Code Artists and sort by created_at in reverse chronological order
        `users = User.where(name: 'David', occupation: 'Code Artist').order(created_at: :desc)`

-  Update
    - Long way to update
        ```
        user = User.find_by(name: 'David')
        user.name = 'Dave'
        user.save
        ```
    - Shorthand
        ```
        user = User.find_by(name: 'David')
        user.update(name: 'Dave')
        ```
    - Update several records in bulk
        ```
        User.update_all name: 'Dave'
        ```
- Delete
    - Find and destroy
        ```
        user = User.find_by(name: 'David')
        user.destroy
        ```
    - Delete several records in bulk, you may use destroy_by or destroy_all method
        ```
        # find and delete all users named David
        User.destroy_by(name: 'David')
        
        # delete all users
        User.destroy_all
        ```
## Validations

Active Record allows you to validate the state of a model before it gets written into the database. The `validates` hook is such an example;

```
class User < ApplicationRecord
  validates :name, presence: true
end
 
user = User.new
user.save  # => false
user.save! # => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```
 For more info look at the `validations notes`.

## Callbacks

Active Record callbacks allow you to attach code to certain events in the life-cycle of your models.
For more info look at the `callbacks notes`.

## Migrations
For more info look at the `migration notes`.
