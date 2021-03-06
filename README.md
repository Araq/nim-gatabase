
# Nim-Gatabase

![screenshot](https://raw.githubusercontent.com/juancarlospaco/nim-gatabase/master/temp.jpg "Postgres and SQLite high-level ORM for Nim")


# Features

- [Postgres >= 10](https://www.postgresql.org) and [SQLite](https://sqlite.org), Async and Sync.
- UTF-8 encoding.
- [All SQL are `const`.](https://nim-lang.org/docs/manual.html#statements-and-expressions-const-section)
- Database user must have a password.
- Database connection is to hostname.
- [Self-Documentation Comments supported  (Postgres).](https://www.postgresql.org/docs/11/sql-comment.html)
- [Configurable AutoVacuum supported  (Postgres).](https://www.postgresql.org/docs/11/runtime-config-autovacuum.html)
- Backups for Databases supported.
- The `timeout` argument is on Seconds.
- `DISTINCT` supported on SQL, `bool` type, `false` by default.
- `LIMIT` supported on SQL, `int` type, `int.high` by default.
- `OFFSET` supported on SQL, `int` type, `0` by default.
- No OS-specific code, so it should work on Linux, Windows and Mac.
- Compatible with [`db_postgres`](https://nim-lang.org/docs/db_postgres.html) and [Ormin](https://github.com/Araq/ormin).
- You can still use std lib `db_postgres` and `db_sqlite` as normal (same connection).
- You can write with Gatabase and read with Ormin.
- [Functional, all functions are `func` (Effects free).](https://nim-lang.org/docs/manual.html#procedures-func)
- Debug raw SQL when not build for Release.
- Table Helpers (ready-made Table templates for common uses).
- Single file. 0 Dependency. Self-Documented.
- Compile with `-d:sqlite` to enable SQLite and disable Postgres.
- Compile with `-d:noFields` to disable Fields feature, for smaller binary (Kb), etc.
- No code for Postgres when compiled for SQLite, no code for SQLite when compiled for Postgres.
- Run `nim doc gatabase.nim` for more Documentation. `runnableExamples` provided.
- Run `nim c -r gatabase.nim` for an Example.
- **Pull Requests welcome!.**


# Fields

- `StringField`
- `Int8Field`
- `Int16Field`
- `Int32Field`
- `IntField`
- `Float32Field`
- `FloatField`
- `BoolField`
- `PDocumentField`
- `ColorField`
- `HashField`
- `HttpCodeField`
- `PegField`


# Use

```nim
import gatabase

# Database init (change to your user and password). For SQLite compile with -d:sqlite
var database = Gatabase(user: "MyUserHere", password: "Passw0rd!", host: "localhost",
                        dbname: "database", port: 5432, timeout: 42)
database.connect()

# Engine
echo gatabaseVersion
echo gatabaseIsPostgres
echo gatabaseIsFields
echo database.uri
echo database.enableHstore()  # Postgres HSTORE Extension.
echo database.getVersion()
echo database.getEnv()
echo database.getPid()
echo database.listAllUsers()
echo database.listAllDatabases()
echo database.listAllSchemas()
echo database.listAllTables()
echo database.getCurrentUser()
echo database.getCurrentDatabase()
echo database.getCurrentSchema()
echo database.getLoggedInUsers()
echo database.forceCommit()
echo database.forceRollback()
echo database.forceReloadConfig()
echo database.isUserConnected(username = "juan")
echo database.getDatabaseSize(databasename = "database")  
echo database.getTableSize(tablename = "mytable")

# Database
echo database.createDatabase("testing", "This is a Documentation Comment")
echo database.grantSelect("testing")
echo database.grantAll("testing")
echo database.getDatabaseSize()
echo database.renameDatabase("testing", "testing2")
echo database.getTop(3)
echo database.dropDatabase("testing2")

# User
echo database.createUser("pepe", "PaSsW0rD!", "This is a Documentation Comment")
echo database.changePasswordUser("pepe", "p@ssw0rd")
echo database.renameUser("pepe", "BongoCat")
echo database.dropUser("BongoCat")

# Schema
echo database.createSchema("memes", "This is a Documentation Comment", autocommit=false)
echo database.renameSchema("memes", "foo")
echo database.dropSchema("foo")

when not defined(noFields): # Compile with `-d:noFields` to disable Fields feature.
  let   # Fields
    a = newInt8Field(int8.high, "name0", "Help here", "Error here")
    b = newInt16Field(int16.high, "name1", "Help here", "Error here")
    c = newInt32Field(int32.high, "name2", "Help here", "Error here")
    d = newIntField(int.high, "name3", "Help here", "Error here")
    e = newFloat32Field(42.0.float32, "name4", "Help here", "Error here")
    f = newFloatField(666.0.float64, "name5", "Help here", "Error here")
    g = newBoolField(true, "name6", "Help here", "Error here")
    # fails = newInt8Field(int64.high, "name9", "Input an int8", "Integer overflow error")
  assert a is Field
  assert b is Field

  # Tables
  echo database.createTable("table_name", fields = @[a, b, c, d, e, f, g],
                            "This is a Documentation Comment")
  echo database.getAllRows("table_name", limit=255, offset=2, `distinct`=true)
  echo database.searchColumns("table_name", "name0", $int8.high, 255)
  echo database.changeAutoVacuumTable("table_name", true)
  echo database.getTableSize(tablename = "table_name")
  echo database.renameTable("table_name", "cats")
  echo database.dropTable("cats")

# Table Helpers (ready-made "Users" table from 3 templates to choose)
echo database.createTableUsers(tablename="usuarios", kind="medium")
echo database.dropTable("usuarios")

# Backups
echo database.backupDatabase("database", "backup0.sql").output
echo database.backupDatabase("database", "backup1.sql", dataOnly=true, inserts=true).output

# db_postgres compatible (Raw Queries)
echo database.db.getRow(sql"SELECT current_database(); /* Still compatible with Std Lib */")

database.close()


## Async Postgres Gatabase Example.
proc asyncExample() {.async.} =
  var database = AsyncGatabase(user: "juan", password: "juan",
                               host: "localhost", dbname: "database",
                               port: Port(5432), timeout: 10, connectionCount: 2)
  database.connect()
  var futures = @[Future[seq[Row]]]
  for i in 0..9:
    futures.add database.getAllRows(sql"SELECT NOW(), pg_sleep(1);", @[])
  for gatabaseFuture in futures:
    echo(await gatabaseFuture)
  database.close()

waitFor asyncExample()


# Check the Docs for more...
```

**Creating a Table with Fields:**

```nim
echo database.createTable(tablename="table_name", fields = @[a, b, c, d, e, f, g], comment="This is a Documentation Comment", autocommit=true)
```

**Produces the SQL:**

```sql
CREATE TABLE IF NOT EXISTS table_name(
  id SERIAL PRIMARY KEY,
  name0 smallint DEFAULT 127,
  name1 smallint DEFAULT 32767,
  name2 integer DEFAULT 2147483647,
  name3 bigint DEFAULT 9223372036854775807,
  name4 decimal DEFAULT 42.0,
  name5 decimal DEFAULT 666.0,
  name6 boolean DEFAULT true
); /* This is a Documentation Comment */

COMMENT ON TABLE table_name IS 'This is a Documentation Comment';
```


# Install

- `nimble install gatabase`


# Use

Lets say you have an example file that uses Gatabase ORM, how to Compile it?.

- For Postgres `nim c yourfile.nim`.
- For SQLite `nim c -d:sqlite yourfile.nim`.
- For Postgres without Fields feature `nim c -d:noFields yourfile.nim`.


# FAQ

<details>

- Supports SQLite ?.

Yes.

- Supports MySQL ?.

No.

- Will support MySQL someday ?.

No.

- This works with Synchronous code ?.

Yes.

- This works with Asynchronous code ?.

Yes.

- Whats `-d:noFields` for ?.

Smaller binaries, less imports, more manual hand-crafted queries, simpler, etc.

- SQLite mode dont support some stuff ?.

We try to keep as similar as possible, but SQLite is very limited.

- I dont want to pass the Table name every time I use a function ?.

https://nim-lang.org/docs/manual.html#statements-and-expressions-using-statement

</details>


# Requisites

- None.

_(You need a working Postgres server up & running to use it, but not to install it)_


#### Extras

- [Recommended tool for SQL, Open source Qt5/C++ WYSIWYG & Drag'n'Drop graphical query builder.](https://pgmodeler.io/screenshots)
- [Learn SQL once, so you dont have to learn several ORMs, is actually very easy to learn.](https://pgexercises.com/questions/basic/selectall.html)
- [For a lower-level ORM see ORMin.](https://github.com/Araq/blog/blob/master/ormin.rst#ormin)
(DSL ORM, JSON WebSockets, Undocumented).
