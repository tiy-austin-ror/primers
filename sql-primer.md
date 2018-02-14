# SQL: A Primer
SQL is the leading language used to interact with Relational Databases like the ones used by most web sites. We will be using SQL as a way to QUERY existing data in our database and to INSERT new data to be saved for later.

## Getting Started
The first we need to do is decide on the database we are using. For now, we will be using [SQLite3](https://www.sqlite.org/) as our database of choice. SQLite is an awesome and lightweight database that powers nearly every iPhone app and is the default database for Rails.

Since we are using SQLite3, we can enter a sql prompt by typing `sqlite3` anywhere in the terminal.

If we want to *open* an existing sqlite database, we simply state the path to the database we are attempting to open, as in: `sqlite3 my_store.sqlite3`

## Writing Queries
Once we have a sql prompt open, we can start to write queries against the data that we have. In our database we have a **table** called `products` and inside the `products` table we have the following **rows** of data. The **fields** of the `products` table are (`id`, `name`, `cost`, and `quantity`).

##### products
| id |    name    |    cost  | quantity |
|--- | ----       | ----     |   ---    |
| 1  | Quadcopter | 199.99   | 10 |
| 2  | Teleporter | 99999    | 1  |
| 3  | VR Headset | 250.00   | 20 |
| 4  | Motion Sensor | 50.00 | 99 |

### SYNTAX
#### `SELECT desired_fields FROM table_name;`
The basics of any SQL query are to select the fields you want from a particular table.
_(Note the `;` semicolon at the end of the line. **Every** SQL statement must end with a semicolon)_

### Getting Everything
#### `SELECT * FROM products;`
If we are simply looking for every product and every **column** then we can ask for `*` from `products`. In SQL `*` is considered a wildcard that says to return every field.

### Putting a LIMIT on How Much We Get Back
#### `SELECT * FROM products LIMIT(1);`
If we are not concerned with getting back every record, but instead want a certain _amount_ of records. We can achieve that via the `LIMIT` function

### Starting with an OFFSET
#### `SELECT * FROM products LIMIT(-1) OFFSET(5);`
If we need to start at a certain row and begin our query from there, we have this handy `OFFSET` function built in to do just that. See how `LIMIT` is being passed `-1`? That is because you cannot use `OFFSET` by itself in sqlite. Setting limit to -1 states that there is no limit.

### Only Getting Certain Fields
#### `SELECT name, cost FROM products;`
If we do not need every single field from table, it is better to ask for the fields we need explicitly. Notice that they are comma separated.

### Maintaining ORDER with Results
#### `SELECT * FROM products ORDER BY name;`
When we need our results in a particular order, we have the `ORDER BY` keyword to let us supply which field they should be sorted by. This will make the results alphabetical if the type of the field specified is a string or numerically if it is an integer.


### Limiting The Results with WHERE
#### `SELECT * FROM products WHERE quantity > 15;`
#### `SELECT name FROM products WHERE quantity > 0 AND cost < 200.00;`
#### `SELECT * FROM products WHERE name = 'Quadcopter' || name = 'Duecopter';`
Sooner or later, we are going to want only specific records from our database. Using `WHERE` we can place a **boolean statement** to the end of our query to filter which rows we receive back.
We can also chain multiple boolean statements together through the use of the `AND` and `OR` keywords. They are functionally equivalent to `&&` and `||`.

### Joining Tables
#### `SELECT * FROM students INNER JOIN checkins ON students.id = checkins.student_id WHERE checkins.id = 1;`
This query is asking for all the fields on the student that has the same `id` as the `student_id` field on the checkins table.
The above query is against these two tables:

**students**

| id  | name |
|---  | ---- |
| 23  | Jake |
| 45  | Finn |

**checkins**

| id | student_id | time |
|--- | ----       | ---- |
| 1  | 23         | 8:00 |
| 2  | 45         | 9:00 |

Sometimes you need to make queries against data in multiple tables, but not all the data in both tables, just some. Joins are here to help. Joins are also a bit tricky at first. Don't feel bad if they take a little bit to wrap your head around. I still find myself scratching my head each time I go to write one. If you are a visual learner like myself, links like [this one](http://blog.codinghorror.com/a-visual-explanation-of-sql-joins/), [this one](http://www.sql-join.com/), and [this one](http://www.sitepoint.com/understanding-sql-joins-mysql-database/) should be helpful.
### Other Useful Functions
- `COUNT`
- `SUM`
- `AVG`
- `MAX`
- `MIN`
- `LENGTH`
- `LOWER`
- `UPPER`
- `RANDOM`
- `REPLACE` _(Think of it as `gsub` in Ruby)_
- `TRIM`

## Adding, Updating, and Deleting Rows

### INSERT-ing New Rows
##### `INSERT INTO products (id, name, cost, quantity) VALUES (5, rocket, 9.99, 100);`
##### `INSERT INTO products (name, cost, quantity) VALUES (gocart, 999, 10);`
Creating a row inside a table in a database is done by 'inserting' a list of values that correspond to the fields of that table. If you look at the second example, you can see that I left off the `ID` field. Because I know that my ID field is a `PRIMARY KEY` I can leave off setting it explicitly, because SQLITE will take care of that for me. From the SQlite Site:
>On an INSERT, if the ROW_ID or INTEGER PRIMARY KEY column is not explicitly given a value, then it will be filled automatically with an unused integer, usually one more than the largest ROWID currently in use. This is true regardless of whether or not the AUTOINCREMENT keyword is used.

### UPDATE-ing and SET-ing Rows
##### `UPDATE products SET name = "Go Kart" WHERE name = 'gocart';`
Updating a row is done by setting the values of certain fields; we specify which rows we want by adding a WHERE clause to the end. It is **very** important to be sure that your `WHERE` clause is correct before running an update query. Otherwise, you risk changing the data in rows that you did not intend.

### DELETE-ing Some Data
##### `DELETE FROM products WHERE id = 2;`
Deleting a row is a simple query that starts with the `DELETE` keyword and takes a `WHERE` clause. Like with `UPDATE` it is **so very** important that your `WHERE` clause is correct. You are better off triple checking your query _before_ running the delete command than realizing you've deleted way more rows than you wanted.
