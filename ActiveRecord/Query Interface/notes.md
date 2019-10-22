# Active Query Interface

Notes on Active Record. More details [here.](https://guides.rubyonrails.org/active_record_querying.html)

## Overview

- Activerecord provides an interface to perform queries to the database without using Raw sql. Activerecord queries are database agnostic. We will use the following models for the examples below:

    ```
    class Client < ApplicationRecord
        has_one :address
        has_many :orders
        has_and_belongs_to_many :roles
    end

    class Address < ApplicationRecord
        belongs_to :client
    end

    class Order < ApplicationRecord
        belongs_to :client, counter_cache: true
    end

    class Role < ApplicationRecord
        has_and_belongs_to_many :clients
    end
    ```

## Retrieving Objects from the Database
- The following methods are used to retrieve objects from the database. The methods are:
    ```
    annotate
    find
    create_with
    distinct
    eager_load
    extending
    extract_associated
    from
    group
    having
    includes
    joins
    left_outer_joins
    limit
    lock
    none
    offset
    optimizer_hints
    order
    preload
    readonly
    references
    reorder
    reselect
    reverse_order
    select
    where
    ```
- The primary operation of Model.find(options) can be summarized as:

    - Convert the supplied options to an equivalent SQL query.
    - Fire the SQL query and retrieve the corresponding results from the database.
    - Instantiate the equivalent Ruby object of the appropriate model for every resulting row.
    - Run after_find and then after_initialize callbacks, if any.

## Examples
- Retrieving a Single Object
    - `find`
        ```
        # Find the client with primary key (id) 10.
        client = Client.find(10)
        # => #<Client id: 10, first_name: "Ryan">
        ```
        The SQL equivalent of the above is:
        ```
        SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1
        ```
        You can pass an array of primary keys
        ```
        # Find the clients with primary keys 1 and 10.
        clients = Client.find([1, 10])
        # Or even Client.find(1, 10)
        # => [#<Client id: 1, first_name: "Lifo">, 
        #      <Client id: 10, first_name: "Ryan">]
        ```
        The SQL equivalent of the above is:
        ```
        SELECT * FROM clients WHERE (clients.id IN (1,10))
        ```
    - `take`
        ```
        client = Client.take
        # => #<Client id: 1, first_name: "Lifo">
        ```

        The SQL equivalent of the above is:
        ```
        SELECT * FROM clients LIMIT 1
        ```

        You can pass in a numerical argument; for example
        ```
        clients = Client.take(2)
        # => [
        #   #<Client id: 1, first_name: "Lifo">,
        #   #<Client id: 220, first_name: "Sara">
        # ]
        ```
        The SQL equivalent of the above is:
        ```
        SELECT * FROM clients LIMIT 2
        ```
    - `first`
        ```
        client = Client.first
        # => #<Client id: 1, first_name: "Lifo">
        ```
        The SQL equivalent of the above is:
        ```
        SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1
        ```
        You can pass in a numerical argument; for example
        ```
        clients = Client.first(3)
        # => [
        #   #<Client id: 1, first_name: "Lifo">,
        #   #<Client id: 2, first_name: "Fifo">,
        #   #<Client id: 3, first_name: "Filo">
        # ]
        ```
        The SQL equivalent of the above is:
        ```
        SELECT * FROM clients ORDER BY clients.id ASC LIMIT 3
        ```
    - `last` behaves like the opposite of `first`
    - `find_by` 

        ```
        Client.find_by first_name: 'Lifo'
        # => #<Client id: 1, first_name: "Lifo">

        Client.find_by first_name: 'Jon'
        # => nil
        ```

        It is equivalent to writing:

        ```
        Client.where(first_name: 'Lifo').take
        ```

- Retrieving Multiple Objects in Batches
    Rails provides two methods that address the problem collection being larger than memory for larger tables by dividing records into memory-friendly batches for processing.
    Instead of;
    ```
    User.all.each do |user|
        NewsMailer.weekly(user).deliver_now
    end
    ```
    Use these below.
    - `find_each`
        ```
        User.find_each do |user|
            NewsMailer.weekly(user).deliver_now
        end
        ```
        This retrieves records in batches and then yields each one to the block.
        ` Options for find_each` are `:batch_size, :start, :finish, :error_on_ignore`
        
        ##### Options
        - `:batch_size` - The :batch_size option allows you to specify the number of records to be retrieved in each batch, before being passed individually to the block. For example, to retrieve records in batches of 5000:
            ```
            User.find_each(batch_size: 5000) do |user|
                NewsMailer.weekly(user).deliver_now
            end
            ```
        - `:start` - By default, records are fetched in ascending order of the primary key. The :start option allows you to configure the first ID of the sequence whenever the lowest ID is not the one you need.
        - `:finish` - Similar to the :start option, :finish allows you to configure the last ID of the sequence whenever the highest ID is not the one you need. Example:
            ```
            User.find_each(start: 2000, finish: 10000) do |user|
                NewsMailer.weekly(user).deliver_now
            end
            ```
            Another example would be if you wanted multiple workers handling the same processing queue. You could have each worker handle 10000 records by setting the appropriate :start and :finish options on each worker.
        - `:error_on_ignore` - Overrides the application config to specify if an error should be raised when an order is present in the relation.

    - `find_in_batches`
        The `find_in_batches` method is similar to find_each, since both retrieve batches of records. The difference is that find_in_batches yields batches to the block as an array of models, instead of individually.

        ```
        # Give add_invoices an array of 1000 invoices at a time.
        Invoice.find_in_batches do |invoices|
            export.add_invoices(invoices)
        end
        ```
        find_in_batches works on model classes, as seen above, and also on relations:
        ```
        Invoice.pending.find_in_batches do |invoices|
            pending_invoices_export.add_invoices(invoices)
        end
        ```
        as long as they have no ordering, since the method needs to force an order internally to iterate.

        The `find_in_batches` method accepts the same options as `find_each`.

- Conditions
