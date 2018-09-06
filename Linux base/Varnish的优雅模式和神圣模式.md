今天在和同事谈论现网后端tomcat挂起导致前端varnish线程堆积，最终造成varnish挂掉或前端web处理繁忙的问题。由于之前详细的看过varnish的文档，当时记得有一个上帝模式还是圣杯模式来着（由于英向汉翻译的问题，可能有N种翻译方法）。由于时隔相当一段时间了，所以记得不大清楚，后来就去网上查了下，现在汉译的统称叫———— 神圣模式和优雅模式可以应用到上面提到的场景。这里再做一个tag总结成博文记录下。

#### 一、Grace mode（优雅模式）

为什么使用Grace mode ，优雅模式的优点是什么？

如果你每秒需要相应成千上万的点击，等待的请求队列就会很巨大。这里有两个潜在问题：一个是thundering herd problem（这个无法翻译。。。wiki有专门的对应解释），突然增加一千个线程去提供内容，会让负载变得很高；第二个，没有人喜欢等。为了解决这个问题，我们指示varnish去保持缓存的对象超过他们的TTL（就是该过期的，不让它过期），并且去提供旧的内容给正在等待的请求。 既然要提供旧的内容，首先我们必须有内容去提供。所以，我们使用以下VCL，以使varnish保持所有对象超出了他们的TTL30分钟。

```
sub vcl_fetch {
  set beresp.grace = 30m;
}
```

这样，varnish还不会提供旧对象。为了启用varnish去提供旧对象，我们必须在请求上开启它。下面表示，我们接收15s的旧对象：

```
sub vcl_recv {
  set req.grace = 15s;
}
```

varnish的优势在于内存级的cache，所以内存的多少决定了cache的数据量的多少。如果开启了优雅模式，在TTL到期后，我们仍不将其从mem里清除掉，而是要再等待一段时间才清除，这就无端的浪费了资源。但如果你开启了健康检查，你可以检查后端是否出问题。如果出问题了，我们可以提供长点时间的旧内容。如果后端没有问题，我们可以将该时间设置的短一些。这就在保证优雅的本身，减少了资源的浪费，其配置如下：

```
if (! req.backend.healthy) {
   set req.grace = 5m;
} else {
   set req.grace = 15s;
}
```

所以，综上所述。优雅模式的主要功能有以下两点：

1、合并请求，当N个客户端请求同一个页面的时候，varnish只发送一个请求到后端服务器，然后让其他几个请求挂起等待返回结果，返回结果后，复制请求的结果发送给客户端。。

2、通过提供旧的内容，避免请求扎堆。如果后端提供旧的内容，减少后端和前端请求的压力，而且为后端的重启和切换提供了时间。

#### 二、神圣模式（Saint mode）

有时候,服务器很古怪,他们发出随机错误,您需要通知 varnish 使用更加优雅的方式处理 它,这种方式叫神圣模式(saint mode)。Saint mode 允许您抛弃一个后端服务器或者另一个尝试的后端服务器或者 cache 中服务陈旧的内容。如：

```
sub vcl_fetch {
    if (beresp.status == 500) {
        set beresp.saintmode = 10s;
        return (restart);
　　}
　　
    set beresp.grace = 5m;
}
```

如上面的配置，当我们设置beresp.saintmode为10秒时，varnish会不请求该服务器10秒。或多或少可以算是一个黑名单。restart被执行时，如果我们有其他后端可以提供该内容，varnish会请求它们。当没有其他后端可用，varnish就会提供缓存中的旧内容。

#### 三、grace和saint模式的局限性

以上两种模式也并非是万能的，如当请求正在被获取时，如果你的请求失败，会被扔到vcl_error中。由于vcl_error对数据集的访问会在前端显示，所以你不能启用优雅模式和神圣模式。在以后发布的版本中会解决这个问题，但是这里我们还是可以做些尽量避免该问题，官方给出的原文是：

```
Declare a backend that is always sick.
Set a magic marker in vcl_error
Restart the transaction
Note the magic marker in vcl_recv and set the backend to the one mentioned
Varnish will now serve stale data is any is available
```

这段话理解上比较费力，也有人做了一个中文翻译版是：

```
1、声明总是出状况的后端
2、在vcl_error中设置magic marker
3、重启事务
4、注意vcl_recv中的magic marker，并设置后端为之前提到的。
5、varnish现在将提供旧任何可获得的数据
```

注：其中magic marker是其参数中的一部分，具体可以参看[官方wiki](https://www.varnish-cache.org/trac/wiki/VCLExampleLongerCaching)上的示例。

以上内容主要参看官方的如下页面：

[varnish 3.0.5官网文档](https://www.varnish-cache.org/docs/3.0/tutorial/handling_misbehaving_servers.html)

[varnish static book部分](https://www.varnish-software.com/static/book/Saving_a_request.html)

#### 四、完整示例

由于版本和参数变更的问题，示例中的配置并不保证能完全适用您所用的版本，不过具体可以在该示例的基础上做下适当的增减。其中一些涉及到的参数也可以对比官方文档做下适当更改。

```
backend web1 {
    .host = "172.16.2.31";
    .port = "80";
    .probe = {
        .url = "/";
        .interval = 10s;
        .timeout = 2s;
        .window = 3;
        .threshold = 3;
    }
}
backend web2 {
    .host = "172.16.2.32";
    .port = "80";
    .probe = {
        .url = "/";
        .interval = 10s;
        .timeout = 2s;
        .window = 3;
        .threshold = 3;
    }
}
# 定义负载均衡组
director webgroup random {
    {
       .backend = web1;
       .weight = 1;
    }
    {
       .backend = web2;
       .weight = 1;
    }
}
# 允许刷新缓存的ip
acl purgeAllow {
     "localhost";
     "172.16.2.5";
}
sub vcl_recv {
    # 刷新缓存设置
    if (req.request == "PURGE") {
        #判断是否允许ip
        if (!client.ip ~ purgeAllow) {
             error 405 "Not allowed.";
        }
        #去缓存中查找
        return (lookup);
    }
    std.log("LOG_DEBUG: URL=" + req.url);
    set req.backend = webgroup;
    if (req.request != "GET" && req.request != "HEAD" && req.request != "PUT" && req.request != "POST" && req.request != "TRACE" && req.request != "OPTIONS" && req.request != "DELETE") {
         /* Non-RFC2616 or CONNECT which is weird. */
         return (pipe);
    }
    # 只缓存 GET 和 HEAD 请求
    if (req.request != "GET" && req.request != "HEAD") {
        std.log("LOG_DEBUG: req.request not get!  " + req.request );
        return(pass);
    }
    # http 认证的页面也 pass
　　if (req.http.Authorization) {
        std.log("LOG_DEBUG: req is authorization !");
        return (pass);
    }
    if (req.http.Cache-Control ~ "no-cache") {
        std.log("LOG_DEBUG: req is no-cache");
        return (pass);
    }
    # 忽略admin、verify、servlet目录，以.jsp和.do结尾以及带有?的URL,直接从后端服务器读取内容
    if (req.url ~ "^/admin" || req.url ~ "^/verify/" || req.url ~ "^/servlet/" || req.url ~ ".(jsp|php|do)($|?)") {
        std.log("url is admin or servlet or jsp|php|do, pass.");
        return (pass);
    }
    # 只缓存指定扩展名的请求, 并去除 cookie
    if (req.url ~ "^/[^?]+.(jpeg|jpg|png|gif|bmp|tif|tiff|ico|wmf|js|css|ejs|swf|txt|zip|exe|html|htm)(?.*|)$") {
        std.log("*** url is jpeg|jpg|png|gif|ico|js|css|txt|zip|exe|html|htm set cached! ***");
        unset req.http.cookie;
        # 规范请求头,Accept-Encoding 只保留必要的内容
        if (req.http.Accept-Encoding) {
            if (req.url ~ ".(jpg|png|gif|jpeg)(?.*|)$") {
                remove req.http.Accept-Encoding;
            } elsif (req.http.Accept-Encoding ~ "gzip") {
                set req.http.Accept-Encoding = "gzip";
            } elsif (req.http.Accept-Encoding ~ "deflate") {
                set req.http.Accept-Encoding = "deflate";
            } else {
                remove req.http.Accept-Encoding;
            }
        }
        return(lookup);
    } else {
        std.log("url is not cached!");
        return (pass);
    }
}
sub vcl_hit {
    if (req.request == "PURGE") {
        set obj.ttl = 0s;
        error 200 "Purged.";
    }
    return (deliver);
}
sub vcl_miss {
    std.log("################# cache miss ################### url=" + req.url);
    if (req.request == "PURGE") {
        purge;
        error 200 "Purged.";
    }
}
sub vcl_fetch {
    # 如果后端服务器返回错误，则进入 saintmode
    if (beresp.status == 500 || beresp.status == 501 || beresp.status == 502 || beresp.status == 503 || beresp.status == 504) {
        std.log("beresp.status error!!! beresp.status=" + beresp.status);
        set req.http.host = "status";
        set beresp.saintmode = 20s;
        return (restart);
    }
    # 如果后端静止缓存, 则跳过
    if (beresp.http.Pragma ~ "no-cache" || beresp.http.Cache-Control ~ "no-cache" || beresp.http.Cache-Control ~ "private") {
        std.log("not allow cached!   beresp.http.Cache-Control=" + beresp.http.Cache-Control);
    return (hit_for_pass);
    }
    if (beresp.ttl <= 0s || beresp.http.Set-Cookie || beresp.http.Vary == "*") {
        /* Mark as "Hit-For-Pass" for the next 2 minutes */
        set beresp.ttl = 120 s;
        return (hit_for_pass);
    }
    if (req.request == "GET" && req.url ~ ".(css|js|ejs|html|htm)$") {
        std.log("gzip is enable.");
        set beresp.do_gzip = true;
        set beresp.ttl = 20s;
    }
    if (req.request == "GET" && req.url ~ "^/[^?]+.(jpeg|jpg|png|gif|bmp|tif|tiff|ico|wmf|js|css|ejs|swf|txt|zip|exe)(?.*|)$") {
        std.log("url css|js|gif|jpg|jpeg|bmp|png|tiff|tif|ico|swf|exe|zip|bmp|wmf is cache 5m!");
        set beresp.ttl = 5m;
    } elseif (req.request == "GET" && req.url ~ ".(html|htm)$") {
        set beresp.ttl = 30s;
    } else {
        return (hit_for_pass);
    }
    # 如果后端不健康，则先返回缓存数据1分钟
    if (!req.backend.healthy) {
        std.log("eq.backend not healthy! req.grace = 1m");
        set req.grace = 1m;
    } else {
        set req.grace = 30s;
    }
     return (deliver);
}
# 发送给客户端
sub vcl_deliver {
    if ( obj.hits > 0 ) {
    set resp.http.X-Cache = "has cache";
    } else {
    #set resp.http.X-Cache = "no cache";
    }
    return (deliver);
}
```