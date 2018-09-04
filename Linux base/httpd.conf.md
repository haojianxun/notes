# httpd.conf

[TOC]

httpd.conf的位置 /etc/httpd/conf/httpd.conf

httpd2.4的新特性:

1. MPM支持DSO机制,按需以模块化加载
2. event MPM生产环境可用
3. 异步读写机制
4. 支持每模块和每目录的单独日志级别定义
5. 每请求相关的专用配置
6. 增强版的表达式分析
7. 毫秒级的持久连接
8. 基于FQDN的虚拟主机也不再需要NameVirutalHost指令
9. 新指令,AlloverrideList
10. 支持用户自定义变量的使用
11. 降低内存消耗

**ServerRoot "/etc/httpd"**

> ServiceRoot   
>
> Description:Base directory for the server installation
>
> Syntax:ServerRoot directory-path

常是Apache存放配置文件和日志文件的地方,默认的地方是在/etc/httpd

Listen 80  

> 这个是配置监听的端口,也可以加ip:port的形式

ServerAdmin root@localhost   

> 这个是设置网络管理员的电子邮件地址

ServerName www.example.com:80  

> ## ServerName
>
> Description:	Hostname and port that the server uses to identify itself
> Syntax:	ServerName [scheme://]domain-name|ip-address[:port]
>
> 设置服务器主机名称,如果服务器又域名地址,则填域名地址,没有的话,则填ip地址

DocumentRoot "/var/www/html"

> DocumentRoot  
>
> 描述:Directory that forms the main document tree visible from the web
>
> 语法:DocumentRoot directory-path
>
> 这个是配置服务器主目录,默认是/var/www/html.比如访问http://my.example.com/index.html指 /var/www/html/index.html.如果该目录写的不是绝对路径,那么就会被判定为相当于serviceroot的路径,注意这个路径后面不能跟斜杠,写DocunmentRoot "/var/www/html"的正确的  写DocunmentRoot "/var/www/html/"是不正确的  因为这个要写的目录的路径所在

设置默认文档

> DirectoryIndex  后面跟一个文件名 ,如果有多个文件名，每个文件名之间必须用空格进行分隔，Apache会根据文件名的先后顺序查找在DirectoryIndex语句中指定的文件名。如果能找到第1个则调用第1个，否则再寻找并调用第2个，依次类推
>
> 默认文档是指在网页浏览器中输入Web站点的IP地址或者域名显示出来的Web页面，也就是通常所说的主页。在缺省情况下，Apache的默认文档名为index.html

ErrorLog "logs/error_log"

CustomLog "logs/access_log"

> 错误日志  访问日志
>
> 错误日志记录了Apache在启动和运行时发生的错误，所以当Apache出错的时候，应该首先检查这个日志文件。通常错误日志的文件名为error_log,如果该文件路径不是绝对那么它被认为是相对于所述ServerRoot。
>
> 关于日志很重要 还有管道日志的用法  日志级别 还有日志格式等等  这个之后单独说

AddDefaultCharset UTF-8

> 设定默认字符集,设置了服务器返回给客户端计算机的默认字符集,如果出现中文乱码 可以设置为GB2312



<Directory>

> Description:	Enclose a group of directives that apply only to the named file-system directory, sub-directories, and their contents.
> Syntax:	<Directory directory-path> ... </Directory>
>
> <Directory>和 </Directory>用于封装一组指令，使之仅适用于指定的目录和其目录中的子目录和文件

<Location>

> | Description: | Applies the enclosed directives only to matching URLs |
> | :----------- | :--------------------------------------- |
> | Syntax:      | <Location URL-path\|URL> ... </Location> |

什么是.htaccess文件

> .htaccess 是Apache HTTP Server的文件目录系统级别的配置文件的默认的名字。它提供了在主配置文件中定义用户自定义指令的支持。 这些配置指令需要在 .htaccess 上下文 和用户需要的适当许可。可以通过AllowOverride指令来控制

什么是apr

>  Apache Portable Runtime   apache可移植运行库  是Apache HTTP服务器的支持库，提供了一组映射到下层操作系统的API。在早期 的Apache版本中，应用程序本身必须能够处理各种具体操作系统平台的细节，并针对不同的平台调用不同的处理函数。如果操作系统不支持某个特定的功能，APR将提供一个模拟的实现。这样程序员使用APR编写真正可在不同平台上移植的程序。
>  最初，APR是作为Apache HTTP服务器的一部分而存在的，但是Apache软件基金会将其延伸成一个单独的项目。其他的应用程序可以使用APR来实现平台无关性

什么是CGI

> Common Gateway Interface/CGI 通用网关接口 可以让一个客户端，从网页浏览器向执行在网络服务器上的程序请求数据。CGI描述了服务器和请求处理程序之间传输数据的一种标准。CGI不是一门编程语言。它是网页的表单和你写的程序之间通信的一种协议,CGI是一种通信协议，它把用户传递过来的数据转变成一个k－v的字典。这个字典中不光有用户的数据，还有HTTP协议的参数。它做的就是把数据，组织成一个固定结构形式的数据。方便任何符合CGI协议的程序调用

什么是MPM

> 多路处理模块，Multi-Processing Modules，MPM。负责绑定本机网络端口、接受请求，并调度子进程来处理请求。
>
> Linux中主要有3种模式,但是我们一般情况下用的是一些2个
>
> 3种主要的MPM模式
>
> 1. prefork 一个非线程型的、预派生的MPM,每一个服务进程都可以处理请求，一个父进程管理服务进程池的大小。它适合于没有线程安全库，需要避免线程兼容性问题的系统。它是要求将每个请求相互独立的情况下最好的MPM，这样若一个请求出现问题就不会影响到其他请求
>
>    工作方式:
>
>    一个单独的控制进程(父进程)负责产生子进程，这些子进程用于监听请求并作出应答。Apache总是试图保持一些备用的(spare)或者是空闲的子进程用于迎接即将到来的请求。这样客户端就不需要在得到服务前等候子进程的产生
>
>    在Unix系统中，父进程通常以root身份运行以便邦定80端口，而Apache产生的子进程通常以一个低特权的用户运行。User和Group指令用于设置子进程的低特权用户。运行子进程的用户必须要对它所服务的内容有读取的权限，但是对服务内容之外的其他资源必须拥有尽可能少的权限。
>
>    MaxRequestsPerChild指令控制服务器杀死旧进程产生新进程的频率。
>
> 2. worker 线程型的MPM，实现了一个混合的多线程多处理MPM，允许一个子进程中包含多个线程.实现的是一个混合的多线程多进程web服务器。由于使用线程来处理请求，所以可以处理海量请求，而系统资源的开销却小于基于进程的web服务器。它也使用了多进程，每个进程又有多个线程，以获得基于进程的MPM的稳定性。但它的兼容性并不是很好，一些旧的程序无法工作在worker下。它最重要的配置指令是控制每个子进程生成子线程的数量ThreadsPerChild和控制生成线程的总数MaxRequestWorkers
>
>    工作方式:
>
>    ​        每个进程可以拥有的线程数量是固定的。服务器会根据负载情况增加或减少进程数量。一个单独的控制进程(父进程)负责子进程的建立。每个子进程可以建立ThreadsPerChild数量的服务线程和一个监听线程，该监听线程监听接入请求并将其传递给服务线程处理和应答。
>
>    　　Apache总是试图维持一个备用(spare)或是空闲的服务线程池。这样，客户端无须等待新线程或新进程的建立即可得到处理。初始化时建立的进程数量由StartServers指令决定。随后父进程检测所有子进程中空闲线程的总数，并新建或结束子进程使空闲线程的总数维持在MinSpareThreads和MaxSpareThreads所指定的范围内。由于这个过程是自动调整的，几乎没有必要修改这些指令的缺省值。可以并行处理的客户端的最大数量取决于MaxClients指令。活动子进程的最大数量取决于MaxClients除以ThreadsPerChild的值。
>
>    　　有两个指令设置了活动子进程数量和每个子进程中线程数量的硬限制。要想改变这个硬限制必须完全停止服务器然后再启动服务器(直接重启是不行的)，ServerLimit是活动子进程数量的硬限制，它必须大于或等于MaxClients除以ThreadsPerChild的值。ThreadLimit是所有服务线程总数的硬限制，它必须大于或等于ThreadsPerChild指令。这两个指令必须出现在其他workerMPM指令的前面。
>
>    在设置的活动子进程数量之外，还可能有额外的子进程处于"正在中止"的状态但是其中至少有一个服务线程仍然在处理客户端请求，直到到达MaxClients以致结束进程，虽然实际数量会很小。这个行为能够通过以下禁止特别的子进程中止的方法来避免：
>
>    * 将MaxRequestsPerChild设为"0"
>
>    * 将MaxSpareThreads和MaxClients设为相同的值
>
>    　　一个典型的针对workerMPM的配置如下：
>
>    　　ServerLimit 16
>
>    　　StartServers 2
>
>    　　MaxClients 150
>
>    　　MinSpareThreads 25
>
>    　　MaxSpareThreads 75
>
>    　　ThreadsPerChild 25
>
>    　　在Unix中，为了能够绑定80端口，父进程一般都是以root身份启动，随后，Apache以较低权限的用户建立子进程和线程。User和Group指令用于设置Apache子进程的权限。虽然子进程必须对其提供的内容拥有读权限，但应该尽可能给予它较少的特权。另外，除非使用了suexec ，否则，这些指令设置的权限将被CGI脚本所继承。
>
>    MaxRequestsPerChild指令用于控制服务器建立新进程和结束旧进程的频率。
>
> 3. event 一个标准workerMPM的实验性变种  apache2.4默认的模式
>
> MPM常见指令
>
> - StartServers 指令
>
>   StartServers指令设置了服务器启动时建立的子进程数量。
>
>   因为子进程数量动态的取决于负载的轻重，所有一般没有必		 要调整这个参数。
>
> - MinSpareServers 指令
>
>   MinSpareServers指令设置空闲子进程的最小数量。
>
>   所谓空闲子进程是指没有正在处理请求的子进程。如果当前	空闲子进程数少于MinSpareServers ，那么Apache将以最大每秒一个的速度产生新的子进程。
>
>   只有在非常繁忙机器上才需要调整这个参数。将此参数设的太大通常是一个坏主意。
>
> - MaxSpareServers 指令
>
>   MaxSpareServers指令设置空闲子进程的最大数量。
>
>   所谓空闲子进程是指没有正在处理请求的子进程。如果当前有超过MaxSpareServers数量的空闲子进程，那么父进程将杀死多余的子进程。
>
>   只有在非常繁忙机器上才需要调整这个参数。将此参数设的太大通常是一个坏主意。如果将该指令的值设置为比MinSpareServers小，Apache将会自动将其修改成"MinSpareServers+1"。
>
> - MaxClients 指令
>
>   MaxClients指令设置了允许同时伺服的最大接入请求数量。
>
>   任何超过MaxClients限制的请求都将进入等候队列，直到达到ListenBacklog指令限制的最大值为止。一旦一个链接被释放，队列中的请求将得到服务。
>
>   对于非线程型的MPM(也就是prefork)，MaxClients表示可以用于伺服客户端请求的最大子进程数量，默认值是256。要增大这个值，必须同时增大ServerLimit 。
>
>   对于线程型或者混合型的MPM(也就是beos或worker)，MaxClients表示可以用于伺服客户端请求的最大线程数量。线程型的beos的默认值是50。对于混合型的MPM默认值是16(ServerLimit)乘以25(ThreadsPerChild)的结果。因此要将MaxClients增加到超过16个进程才能提供的时候，必须同时增加ServerLimit的值。
>
> - MaxRequestsPerChild 指令
>
>   MaxRequestsPerChild指令设置每个子进程在其生存期内允许伺服的最大请求数量。到达MaxRequestsPerChild的限制后，子进程将会结束。如果MaxRequestsPerChild为"0"，子进程将永远不会结束。
>
>   不同的默认值
>
>   在mpm_netware和mpm_winnt上的默认值是"0"。
>
>   将MaxRequestsPerChild设置成非零值有两个好处：
>
>   - 可以防止(偶然的)内存泄漏无限进行，从而耗尽内存。
>   - 给进程一个有限寿命，从而有助于当服务器负载减轻的时候减少活动进程的数量。
>
>   注意：
>
>   　　对于KeepAlive链接，只有第一个请求会被计数。事实上，它改变了每个子进程限制最大链接数量的行为。
>
> - ThreadsPerChild 指令
>
>   这个指令设置了每个子进程建立的线程数。
>
>   子进程在启动时建立这些线程后就不再建立新的线程了。如果使用一个类似于mpm_winnt只有一个子进程的MPM，这个数值要足够大，以便可以处理可能的请求高峰。如果使用一个类似于worker有多个子进程的MPM，每个子进程所拥有的所有线程的总数要足够大，以便可以处理可能的请求高峰。
>
>   对于mpm_winnt，ThreadsPerChild的默认值是64；对于其他MPM是25。
>
> - ThreadLimit 指令
>
>   这个指令设置了每个子进程可配置的线程数ThreadsPerChild上限。
>
>   任何在重启期间对这个指令的改变都将被忽略，但对ThreadsPerChild的修改却会生效。
>
>   使用这个指令时要特别当心。如果将ThreadLimit设置成一个高出ThreadsPerChild实际需要很多的值，将会有过多的共享内存被分配。如果将ThreadLimit和ThreadsPerChild设置成超过系统的处理能力，Apache可能无法启动，或者系统将变得不稳定。该指令的值应当和ThreadsPerChild可能达到的最大值保持一致。
>
>   对于mpm_winnt，ThreadLimit的默认值是1920；对于其他MPM这个值是64。
>
>   ​
>
>   注意：
>
>   Apache在编译时内部有一个硬性的限制"ThreadLimit 20000"(对于mpm_winnt是"ThreadLimit 15000")，不能超越这个限制。
>
> - ServerLimit 指令
>
>   对于preforkMPM，这个指令设置了MaxClients最大允许配置的数值。
>
>   对于workerMPM，这个指令和ThreadLimit结合使用设置了MaxClients最大允许配置的数值。任何在重启期间对这个指令的改变都将被忽略，但对MaxClients的修改却会生效。
>
>   使用这个指令时要特别当心。如果将ServerLimit设置成一个高出实际需要许多的值，将会有过多的共享内存被分配。如果将ServerLimit和MaxClients设置成超过系统的处理能力，Apache可能无法启动，或者系统将变得不稳定。
>
>   对于preforkMPM，只有在需要将MaxClients设置成高于默认值256的时候才需要使用这个指令。要将此指令的值保持和MaxClients一样。
>
>   对于workerMPM，只有在需要将MaxClients和ThreadsPerChild设置成需要超过默认值16个子进程的时候才需要使用这个指令。不要将该指令的值设置的比MaxClients 和ThreadsPerChild需要的子进程数量高。
>
>   ​
>
>   注意：
>
>   Apache在编译时内部有一个硬限制"ServerLimit 20000"(对于	preforkMPM为"ServerLimit 200000")。不能超越这个限制。
>
> 总结
>
> 如果站点需要更好伸缩性可以选择worker或event线程化的MPM，而需要更好的稳定性和兼容性以适应一些旧的软件的站点可以用prefork 。目前最的apache版本2.4中已经对event的性能做了很多优化。所以其已经从一个实验性变种变成了一真正可用性的模块。

启用MPM模块

> 配置的路径在/etc/httpd/conf.models.d/00-mpm.conf
>
> 要启用那个模块就启用 但是只能启用一个  其他的得注释掉 默认是prefork
>
> 要是自己编译的话在extra目录下  
>
> 先找到httpd.conf  先把mpm模块打开(去掉注释include语句 对应模块的那句 )
>
> 在找到loadmodule 模块 启用它
>
> 如果重启出现欢迎页面 ,可以先把欢迎页面改名(欢迎页面是welcome那个页面)

配置文件注意的地方

> 在配置directory 中  主配置文件  记得把index选项去叫  最好的方法是在其前面加个 "－ ''    基于主机的访问 require all  granted    拒绝的话是require all deny   基于某个地址是   require ip IPADDR  或者也可以指定 拒绝的主机require not ip IPADDR   
>
> 可以控制特定主机的访问
>
> require host HOSTNAME 
>
> require not host HOSTNAME
>
> HOSTNAME的格式可以是以下就个写法
>
> 1. FQDN
> 2. domin.tld 指定域名下的所以主机
>
> 控制特定主机的访问可以这样,最好加个<RequireAll>的标签
>
> 比如:<RequireALL>
>
> ​		Require all granted
>
> ​		Require not ip 192.168.1.1
>
> ​	</RequireAll>
>
> 在2.4中  任何目录下的页面只有被授权才能被访问  可以使用directory标签来授权

SSL模块

> 安装mod_ssl
>
> 配置文件在/etc/httpd/conf.d下的 ssl.conf
>
> 开启这个模块的配置文件在/etc/httpd/conf.modules.d/00-ssl.conf





