# Shop Manager Model and Repository Classes Design Recipe

_Copy this recipe template to design and implement Model and Repository classes for a database table._

## 1. Design and create the Table

Table: orders

Columns:
id | customer_name | order_date | stock_id

## 2. Create Test SQL seeds

Your tests will depend on data stored in PostgreSQL to run.

If seed data is provided (or you already created it), you can skip this step.

```sql
-- EXAMPLE
-- (file: spec/seeds_{table_name}.sql)

-- Write your SQL seed here.

-- First, you'd need to truncate the table - this is so our table is emptied between each test run,
-- so we can start with a fresh state.
-- (RESTART IDENTITY resets the primary key)

TRUNCATE TABLE stocks, orders RESTART IDENTITY; -- replace with your own table name.

-- Below this line there should only be `INSERT` statements.
-- Replace these statements with your own seed data.

INSERT INTO stocks (item, unit_price, quantity) VALUES ('item1', 1.01, 1);
INSERT INTO stocks (item, unit_price, quantity) VALUES ('item2', 2.00, 2);
INSERT INTO orders (customer_name, order_date, stock_id) VALUES ('customer1', '2022-01-01', 1);
INSERT INTO orders (customer_name, order_date, stock_id) VALUES ('customer2', '2022-02-02', 1);
```

Run this SQL file on the database to truncate (empty) the table, and insert the seed data. Be mindful of the fact any existing records in the table will be deleted.

```bash
psql -h 127.0.0.1 shop_manager_test < spec/seeds_orders.sql
```

## 3. Define the class names

Usually, the Model class name will be the capitalised table name (single instead of plural). The same name is then suffixed by `Repository` for the Repository class name.

```ruby
# Table name: orders

# Model class
# (in lib/order.rb)
class Order
end

# Repository class
# (in lib/stock_repository.rb)
class OrderRepository
end

```

## 4. Implement the Model class

Define the attributes of your Model class. You can usually map the table columns to the attributes of the class, including primary and foreign keys.

```ruby
# EXAMPLE
# Table name: orders

# Model class
# (in lib/order.rb)

class Order

  # Replace the attributes by your own columns.
  attr_accessor :id, :customer_name, :order_date, :stock_id
end

# The keyword attr_accessor is a special Ruby feature
# which allows us to set and get attributes on an object,
# here's an example:
#
# student = Student.new
# student.name = 'Jo'
# student.name

```


*You may choose to test-drive this class, but unless it contains any more logic than the example above, it is probably not needed.*

## 5. Define the Repository Class interface

Your Repository class will need to implement methods for each "read" or "write" operation you'd like to run against the database.

Using comments, define the method signatures (arguments and return value) and what they do - write up the SQL queries that will be used by each method.

```ruby
# EXAMPLE
# Table name: orders

# Repository class
# (in lib/order_repository.rb)

class OrderRepository

  # Selecting all stocks
  # No arguments
  def all
    # Executes the SQL query:
    # SELECT id, customer_name, order_date, stock_id FROM orders;

    # Returns an array of Order objects.
  end

  # Gets a single record by its ID
  # One argument: the id (number)
  def find(id)
    # Executes the SQL query:
    # SELECT id, customer_name, order_date, stock_id FROM orders WHERE id = $1;

    # Returns a single stock object.
  end

  def create(stock)
    # INSERT INTO orders
    # (customer_name, order_date, stock_id)
    # VALUES (order.customer_name, order.order_date, order.stock_id);
    # returns nil
  end

  def delete(id)
    # DELETE FROM stocks WHERE id = $1';
    # return nil
  end

  def update(id)
    # UPDATE stocks SET customer_name = $1, order_date = $2, stock_id = $3 WHERE id = $4;
    # return nil
  end
end

```

## 6. Write Test Examples

Write Ruby code that defines the expected behaviour of the Repository class, following your design from the table written in step 5.

These examples will later be encoded as RSpec tests.

```ruby

# 1
# Get all stock

repo = OrderRepository.new

order = repo.all

order.length # =>  2

order[0].id # =>  '1'
order[0].customer_name # =>  'customer1'
order[0].order_date # =>  '2022-01-01'
order[0].stock_id # =>  '1'

order[1].id # =>  '2'
order[1].customer_name # =>  'customer2'
order[1].order_date # =>  '2022-02-02'
order[1].stock_id # =>  '1'


# 2
# Get a single user_account

repo = OrderRepository.new

order = repo.find(1)

order.id # =>  '1'
order.customer_name # =>  'customer1'
order.order_date # =>  '2022-01-01'
order.stock_id # =>  '1'

order = repo.find(2)

order.id # =>  '2'
order.customer_name # =>  'customer2'
order.order_date # =>  '2022-02-02'
order.stock_id # =>  '1'

# 3
# create a new object

repo = OrderRepository.new
order = Order.new

order.id # =>  '3'
order.customer_name # =>  'customer3'
order.order_date # =>  '2022-03-03'
order.stock_id # =>  '2'

repo.create(order)

orders = repo.all

(orders).to include(
      have_attributes(
        id: order.id,
        customer_name: order.item,
        order_date: order.order_date,
        stock_id: order.stock_id
        )
      ) # => returns an array including the new object

# 4
# deletes all objects

repo = OrderRepository.new
repo.delete(1)
repo.delete(2)
order = repo.all
order.length # => 0

# 5
# deletes one object

repo = OrderRepository.new
repo.delete(1)
order = repo.all
order.length # => 1

order[0].id # =>  '2'
order[0].customer_name # =>  'customer2'
order[0].order_date # =>  '2022-02-02'
order[0].stock_id # =>  '1'

# 6
# update existing entry

repo = OrderRepository.new

order = repo.find(2)

order.id # =>  '2'
order.customer_name # =>  'customer2'
order.order_date # =>  '2022-02-02'
order.stock_id # =>  '1'

repo.update(2)

order = repo.find(2)

order.id # =>  '2'
order.customer_name # =>  'new_customer2'
order.order_date # =>  'new_2022-02-02'
order.stock_id # =>  '1'

```

Encode this example as a test.

## 7. Reload the SQL seeds before each test run

Running the SQL code present in the seed file will empty the table and re-insert the seed data.

This is so you get a fresh table contents every time you run the test suite.

```ruby
# EXAMPLE

# file: spec/student_repository_spec.rb

def reset_order_table
  seed_sql = File.read('spec/seeds_orders.sql')
  connection = PG.connect({ host: '127.0.0.1', dbname: 'shop_manager_test' })
  connection.exec(seed_sql)
end

describe OrderRepository do
  before(:each) do
    reset_order_table
  end
end

  # (your tests will go here).

```

## 8. Test-drive and implement the Repository class behaviour

_After each test you write, follow the test-driving process of red, green, refactor to implement the behaviour._
