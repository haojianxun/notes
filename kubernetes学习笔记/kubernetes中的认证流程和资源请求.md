# kubernetes中的认证流程和资源请求

如果人人都可以通过kubectl来进入集群系统 ,进行增删查改的话 , 那就危险了

如果用户要想进入系统进程操作, 要经历3种安全认证

- **认证**  任何用户要想进行资源操作 , 通过api-server之前要进行认证
- **授权检查**   通过认证之后, 查看他有没有相应的权限对资源进行操作
- **准入控制**   通过权限检查之后 , 要说进行对其他的资源级联操作 , 看看能不能对其他的资源进行操作



kubernetes是高度模块化的 , 这3个步骤都可以由插件进行来完成, 比如认证过程 , 可以有n个插件 , 来支持多中方式认证 ,  比如:

- **支持令牌认证**       所谓令牌认证就是 双方有个域共享密钥 , 由于kubernetes提供rustful风格的接口, 他的所有服务都是通过http协议提供的 , 因此我们的认证信息只能经由http的认证首部进行传递 , 传递的就是token , 由kubectl带着token , 通过http传递给api-server来认证
- **ssl**  能让客户端确认服务器信息 

双方认证通过就ok了  下一步进入到授权检查



**授权检查**

- RBAC

**准入控制:**

后续的安全检查



客户端----->API server

api-server要识别这个用户有没有权限 , 需要哪些信息来认证

需要以下信息

user: username , uid

group:

extra:

如果要请求特定api-server , 因为api-server是进行分组的 , 要访问特定的api资源的话 , 必须要进行标识 资源的标识要进行path标识  比如https://192.168.200.140:6443/api/apps/v1/namespaces/dafault/deployment/myapp-deploy , 也可以通过`kubectl proxy --port=8080` 来进行





**资源类型**

- 对象  比如pod
- 集合  list  同一类型的对象集合

**资源的操作**

可以对资源进行增删改查 , 

请求的时候可以带一个动作 , 这个动作有以下:

http request verb: 

- get
- post
- put
- delete

之后http的请求动作映射到api-service中的api request verb中

api request verb:

- get
- list
- create
- update
- patch
- watch
- proxy
- delete
- deletecollection

动作之后跟资源 还有namespaces 还有api group等等



所以认证是要进行身份认证  授权认证是检查权限



**有哪些客户端需要和api-server打交道**

- 集群外面的  比如集群外面的kubectl    

- 内部pod   

  kubernetes的内部地址 用` kubectl get svc`查看  , 这个是pod用来和api-server

  pod和api-server打交道要进行认证 , 要进行证书认证  那证书的地址放在哪里呢 , 其实是放在 pod中  , 在和api-server打交道的时候要进行账号认证 , 人要访问api-server的话用的是useraccount   那pod要想访问api-server的话, 用是serviceaccount  , 认证的toekn通过secret来创建 , 自己默认挂载在存储卷, 即使你没有定义存储卷 ,  认证的话是通过带着这个token来的 ,  他的作用是仅仅来获取自己pod的相关信息的 , 不能看别人的 , 如果是管理的pod , 要自己创建一个serviceaccount , 之后再提升权限 , 创建的serviceaccoun自带一个由secret创建的token , token里面是内容是用于连接api-server的时候用到的信息 ,  他只能进行认证 , 不能搞权限 , 搞权限的话得另外授权 ,  

