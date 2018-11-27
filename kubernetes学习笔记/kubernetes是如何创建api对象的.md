# kubernetes是如何创建api对象的

一个yaml文件提交之后 , 会交到apiserver那里 , apiserver的handler会更加根据资源定义来找到要创建的类型定义 

在创建这个类型的对象的过程中 , APIServer会进行一个Convert工作, 即把用户提交的一个yaml文件转换成一个Super Version的对象 , 它是该API资源类型所有版本的字段全集 , 接下来APIServer会进行Admission()和Validation()操作 

Admission操作包括Admission Controller和Initializer

Validation操作是要验证这个API字段是否合法 , 如果在APIServer中的Resgistry结构体中没有找到这个对象 , 则说明这个没有经过验证 

最后APIServer会把验证过的API对象转换成用户最初提供的版本 , 进行序列化操作 并通过Etcd的API把他保存下来