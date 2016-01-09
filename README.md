# db.php
simplified work with MySQL databases for your website, convenient wrapper for mysqli

Contains `DB` class as well as procedural shortcuts (uses DB singleton)

### Why I created this library? 
Because of much easier and simplier work with queries and results. Compare:

**db.php use:**
```php
$sql = "SELECT * FROM table ORDER by id";
$rows = db_array($sql); //db opened automatically based on $CONFIG, errors handled automatically
foreach ($rows as $k => $row) {
    printf ("%s (%s)\n", $row["Field1"], $row["Field1"]);
}
```

**"native" mysqli use:**
```php
$link = mysqli_connect("localhost", "my_user", "my_password", "world"); //connect
if (mysqli_connect_errno()) { //check connection
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}
$query = "SELECT * FROM table ORDER by id";
if ($result = mysqli_query($link, $query)) {
    while ($row = mysqli_fetch_assoc($result)) { //fetch associative array
        printf ("%s (%s)\n", $row["Field1"], $row["Field1"]);
    }
    mysqli_free_result($result); //free result set
}
mysqli_close($link); //close connection
```

## Procedural API

Note, there should be global $CONFIG variable with defined DB connection settings:
```php
$CONFIG = array(
    'DB'    => array(
                'DBNAME'    => 'your_database_name',
                'USER'      => 'your_database_user',
                'PWD'       => 'password',
                'HOST'      => 'localhost', //localhost or other host
                'PORT'      => '', //if empty default MySQL port used
                ),
);
```
This way if you use procedural interface, db.php will automatically open connection when first function called.

### Summary
- `db_value($sql)` get single value via arbitrary sql
- `db_value($table_name, $where[, $field_name])` get single value from table/where conditions and optional field_name(if not passed - first field value returned)
- `db_value($table_name, $where, 'count(*)')` get count(*) from table/where

- `db_row($sql)` get single row via arbitrary sql
- `db_row($table_name, $where [, $orderby])` get single row (first row) by table/where and optional order by

- `db_array($sql)` get all rows via arbitrary sql
- `db_array($table_name, $where [[, $orderby], $limit])` get all rows by table/where, optional order by, optional limit

- `db_col($sql)` get all values from first column
- `db_col($table_name, $where [, $field_name [, $orderby]])` get $field_name column values by table/where and optional order by

- `db_query($sql)` raw sql query execution
- `db_exec($sql)` raw non-select query execution

- `db_insert($table_name, $data [, $options])` insert new row into db, $options= array('replace'=>true, 'ignore'=>true, 'no_identity'=>true). $data could be array of arrays if you want to insert multiple rows (see examples)

- `db_update($table_name, $data, $id)` update record by primary key (by default, column named `id` is a primary key)
- `db_update($table_name, $data, $id, $id_column_name [[, $more_set], $more_where])` update record by primary key named by $id_column_name with optional additional SET and WHERE conditions
- `db_update($table_name, $data, $where)` update record by where conditions (AND)

- `db_delete($table_name, $id [[, $id_column_name = 'id'], $more_where=''])` delete record by id

- `db_is_record_exists($table_name, $value, $field [, $not_value [, $not_field]])` check that record exists in db, optionally check  $not_field (default `id`) should not be equal to $not_value

- `db_identity()` get last inserted id
- `dbqi($value)`quote variable as integer
- `dbq($value[, $field_type])` quote variable

### Examples

```php
#get one field value
$users_ctr = db_value('select count(*) from users');

#get one field value with condition
$where = array(
    'id' => 1,
);
$email = db_value('user', $where, 'email');

#get count of rows value with condition
$where = array(
    'status' => 0,
);
$active_users_ctr = db_value('user', $where, 'count(*)');

#get one table row
$id=1;
$row = db_row("users", array('id' => $id));
$row = db_row("select * from users where id=".dbqi($id));

#get one table row with condition and order by
$row = db_row("users", array('status' => 0), 'id desc');

#get all table records
$rows = db_array('select * from users');
$rows = db_array('users', array());

#get all table records with condition
$status=0;
$rows = db_array('select * from users where status='.dbqi($status));
$rows = db_array('users', array('status' => 0));
$rows = db_array('select * from users where email='.dbq('test@test.com'));
$rows = db_array('users', array('email' => 'test@test.com'));

#get all table records with condition, order by 'id desc' and limit 2
$rows = db_array("users", array('status' => 0), 'id desc', 2);

#get first column all values
$ids = db_col("select id from users");

#get named column all values with condition and order
$emails = db_col("users", array('status' => 0), 'email', 'id desc');

#raw select query execution
$res = db_query("select * from users");
$rows = $res->fetch_all();
$res->free();

#raw non-select query execution
db_exec("update users set status=0");

#insert a row
$vars=array(
    'nick'   => 'Jon',
    'email'  => 'john@test.com',
);
$new_id=db_insert('users', $vars);

#replace a row (by primary or unique key)
$vars=array(
    'nick'   => 'John',
    'email'  => 'john@test.com',
);
$new_id=db_insert('users', $vars, array('replace'=>true));

#insert with options - ignore and no_identity
$vars=array(
    'users_id'   => 1,
    'options_id'  => 23,
);
db_insert('users_options', $vars, array('ignore'=>true, 'no_identity'=>true));

#multi-rows insert
$vars=array(
    array(
        'nick'   => 'John',
        'email'  => 'john@test.com',
    ),
    array(
        'nick'   => 'Bill',
        'email'  => 'bill@test.com',
    ),
    array(
        'nick'   => 'Robert',
        'email'  => 'robert@test.com',
    ),
);
db_insert('users', $vars);

#update record by primary key
$id=1;
$vars=array(
    'nick'   => 'Jon',
    'email'  => 'john@test.com',
);
db_update('users', $vars, $id);

#update record by column key
$email=1;
$vars=array(
    'nick'   => 'Jon',
    'upd_time'   => '~!now()', #will set upd_time=now()
);
db_update('users', $vars, $email, 'email');

#update record by column key with additonal set/where
$email='john@test.com';
$vars=array(
    'nick'   => 'Jon',
);
db_update('users', $vars, $email, 'email', ', upd_time=now()', ' and status=0');

#update record by where
$vars=array(
    'nick'   => 'John',
    'upd_time'   => '~!now()', #will set upd_time=now()
);
$where=array(
    'email'  => 'john@test.com',
    'status' => 0,
)
db_update('users', $vars, $where);

#check if record exists for particular field/value
$is_exists=db_is_record_exists('users', 'john@test.com', 'email');

#check if record exists for particular field/value and NOT with other id
$is_exists=db_is_record_exists('users', 'john@test.com', 'email', 1);  #will check: where email='john@test.com' and id<>1

#check if record exists for particular field/value and NOT with other field/value
$is_exists=db_is_record_exists('users', 'john@test.com', 'email', 'John', 'nick');  #will check: where email='john@test.com' and nick<>'John'

```
