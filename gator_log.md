## CH2L1 -> Postgres

In this chapter we installed `postgresql`
and `psql` the client

```bash

sudo apt install postgresql

```


Following are the useful commands for the Postgres service.
```bash

sudo service postgres start
sudo service postgres stop
sudo service postgres status

```
There is also pgadmin, but psql is fine.


You can connect to the Postgres using psql like the following:
```bash

sudo -u postgres psql

```

Update postgres password
```bash

sudo PASSWD postgres

````


Creating a database:
```bash

CREATE DATABASE gator;

```


Connect to the said database:
```bash

\c gator

```



Set the user password (this is what you will use to connect to DB)
```bash

ALTER USER postgres PASSWORD 'postgres';

```




## CH2L2 -> Goose Migrations:
We installed goose tool will is a database migration tool.

We created `.sql` file in `sql/schema` directory. I have to use 
`<some-number>_<file-name>.sql` for the convention e.g `001_users.sql`

We wrote our SQL query in the file.

```sql
-- +goose Up
CREATE TABLE users (
	id		PRIMARY KEY,
	created_at	TIMESTAMP NOT NULL,
	updated_at	TIMESTAMP NOT NULL,
	name		TEXT
);

```
There is also goose specific comments:

`-- +goose Up` `-- +goose Down` 
Note that these are case sensitive.





Connection string:

Username is the username we altered the creating the db
using psql.
`protocol://username:password@host:port/database`




Example:
```bash

protocol://postgres:postgres@localhost:5432/gator

```

Then we test our connection to the table:
```bash

psql protocol://postgres:postgres@localhost:5432/gator

```


Then we go inside `sql/schema` directory and run the migration
command:

```bash

goose postgres <connection-string> up

```

You can check if the migration was successful using by going into
the database.

```

sudo -u postgres sql

\c gator

\dt

```
