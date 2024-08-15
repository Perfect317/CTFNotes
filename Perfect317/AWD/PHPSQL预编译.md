# PHP SQL 预编译
## 预编译

```php
$sql = "SELECT * FROM users WHERE username = ? AND password = ?";
$stmt = $mysqli->prepare($sql);
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();
```

i: integer
d: double
s: string
b: blob
