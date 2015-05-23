# EasyDB - Simple Database Abstraction Layer

PDO lacks brevity and simplicity; EasyDB makes separating data from instructions
easy (and aesthetically pleasing).

EasyDB was created by [Paragon Initiative Enterprises](https://paragonie.com)
as part of our effort to encourage better [application security](https://paragonie.com/service/appsec) practices.

## Why Use EasyDB? Because it's cleaner!

Let's refactor the following legacy insecure code snippet to prevent SQL injection.

```php
$query = mysql_query("SELECT * FROM comments WHERE blogpostid = {$_GET['blogpostid']} ORDER BY created ASC");
while($row = mysql_fetch_assoc($query)) {
    $template_engine->render('comment', $row);
}
```

### The PDO Way

```php
$db = new \PDO('mysql;host=localhost;dbname=something', 'username', 'putastrongpasswordhere');

$statement = $db->prepare(
    'SELECT * FROM comments WHERE blogpostid = ? ORDER BY created ASC'
);
$exec = $statement->execute([$_GET['blogpostid']]);
if ($exec !== false) {
    $rows = $exec->fetchAll(\PDO::FETCH_ASSOC);
    foreach ($rows as $row) {
        $template_engine->render('comment', $row);
    }
}
```

That's a little wordy for such a simple task. If we do this in multiple places,
we end up repeating ourselves a lot.

### The EasyDB Solution

```php
$db = new EasyDB('mysql;host=localhost;dbname=something', 'username', 'putastrongpasswordhere');

$rows = $db->run('SELECT * FROM comments WHERE blogpostid = ? ORDER BY created ASC', $_GET['blogpostid']);
foreach ($rows as $row) {
    $template_engine->render('comment', $row);
}
```

We made it a one-liner.

## What else can EasyDB do quickly?

### Insert a row into a database table

```php
$db->insert('comments', [
    'blogpostid' => $_POST['blogpost'],
    'userid' => $_SESSION['user'],
    'comment' => $_POST['body'],
    'parent' => isset($_POST['replyTo']) ? $_POST['replyTo'] : null
]);
```

### Update a row from a database table

```php
$db->update('comments', [
    'approved' => true
], [
    'commentid' => $_POST['comment']
]);
```

### Delete a row from a database table

```php
// Delete all of this user's comments
$db->delete('comments', [
    'userid' => 3
]);
```

### Fetch a single row from a table

```php
$exists = $db->row(
    "SELECT * FROM users WHERE userid = ?",
    $_GET['userid']
);
```

### Fetch a single column from a single row from a table

```php
$exists = $db->cell(
    "SELECT count(id) FROM users WHERE email = ?",
    $_POST['email']
);
/* OR YOU CAN CALL IT THIS WAY: */
$exists = $db->single(
    "SELECT count(id) FROM users WHERE email = ?", 
    array(
        $_POST['email'] 
    )
);
```

### What if I need PDO for something specific?

```php
$pdo = $db->getPdo();
```