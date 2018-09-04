# varnish中的acl控制

[TOC]

## 定义acl列表

```;
acl forbidden {
	include "/etc/varnish/ip.dat";
}

ip.dat内容:
"10.0.0.1"/24;
"192.16.0.1"/24;
```

## vcl_recv实现

```
sub vcl_recv {
	if (client.ip ~forbidden) {
	error 505 "Forbidden";
	}
}
```

### obj.status实现

```
sub vcl_error {
set obj.http.Content-Type = "text/html; charset=utf-8";
if (obj.status == 750) {
set obj.http.Location = "http://www.google.com/";
set obj.status = 302;
deliver;
}
else {
synthetic {"
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html>
<head>
<title>"} obj.status " " obj.response {"</title>
</head>
<body>
<h1>Error "} obj.status " " obj.response {"</h1>
<p>"} obj.response {"</p>
</body>
</html>
"};
}
return (deliver);
}
```

生效

````
varnishd -d -f /etc/varnish/my.vcl
service varnish restart
````





