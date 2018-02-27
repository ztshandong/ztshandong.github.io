# Windows安装
```sh
>1.5
schtasks /create /sc onstart /tn memcached /tr "'D:\memcached-x86\memcached.exe' -m 512"

schtasks /delete /tn memcached
```