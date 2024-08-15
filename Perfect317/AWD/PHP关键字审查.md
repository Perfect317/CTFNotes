### 检查上传文件类型是否合法

```php
$upload_whitelist = "/jpg|png|gif|txt/i";  // upload白名单

$filename = 'example.jpg';  // 示例文件名，可以替换成实际需要检查的文件名

// 提取文件扩展名
$file_extension = pathinfo($filename, PATHINFO_EXTENSION);

if (preg_match($upload_whitelist, $file_extension)) {
    echo "文件类型合法";
} else {
    echo "文件类型不合法";
}
```

### 检查SQL语句是否合法

```php
$sql_blacklist = "/drop |dumpfile\b|INTO FILE|union select|outfile\b|load_file\b|multipoint\(/i";

$sql = 'SELECT * FROM users';  // 示例SQL语句，可以替换成实际需要检查的SQL语句

if (preg_match($sql_blacklist, $sql)) {
    echo "SQL语句不合法";
} else {
    echo "SQL语句合法";
}
```

### 检查是否包含远程代码执行的黑名单函数

```php
$rce_blacklist = "/`|var_dump|str_rot13|serialize|base64_encode|base64_decode|strrev|eval\(|assert|file_put_contents|fwrite|curl_exec\(|dl\(|readlink|popepassthru|preg_replace|preg_filter|mb_ereg_replace|register_shutdown_function|register_tick_function|create_function|array_map|array_reduce|uasort|uksort|array_udiff|array_walk|call_user_func|array_filter|usort|stream_socket_server|pcntl_exec|passthru|exec\(|system\(|chroot\(|scandir\(|chgrp\(|chown|shell_exec|proc_open|proc_get_status|popen\(|ini_alter|ini_restore|ini_set|LD_PRELOAD|ini_alter|ini_restore|ini_set|base64 -d/i";

$code = 'system("ls")';  // 示例代码，可以替换成实际需要检查的代码

if (preg_match($rce_blacklist, $code)) {
    echo "包含远程代码执行的黑名单函数";
} else {
   

    echo "代码不包含远程代码执行的黑名单函数";
}
```
