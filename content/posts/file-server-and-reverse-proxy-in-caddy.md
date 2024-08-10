+++
title = 'Caddy: file_server和reverse_proxy'
date = 2024-08-10T08:39:14+08:00
draft = false
+++

```caddy
example.com {
    root * /var/www/html
    file_server
    reverse_proxy localhost:8000
}
```

上面的配置是不正确的，`file_server`和`reverse_proxy`都没有matcher，虽然没有报错，但`filer_server`并不会生效，可修改为：

```caddy
example.com {
    handle_path /static/* {
        root * /var/www/html
        file_server
    }
    reverse_proxy localhost:8000
}
```
