1. 关闭远程访问权限

`REVOKE ALL PRIVILEGES ON *.* FROM 'root'@'%' ;`

2. 改密码

`set password for cms@'%' = password('4rerfg3e45rg45yh456');`

3. 替换文本脚本

```bash
// 当前目录即子目录下的'bd81904ab8b3bc91'替换为'4rerfg3e45rg45yh456'
grep -rl 'bd81904ab8b3bc91' . | xargs sed -i 's/bd81904ab8b3bc91/4rerfg3e45rg45yh456/g'
```

