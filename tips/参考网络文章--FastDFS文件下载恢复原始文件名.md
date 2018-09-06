### 经验一：FastDFS文件下载恢复原始文件名

原文地址:http://www.ttlsa.com/fastdfs/fastdfs-experience-sharing/

应用背景

文件被上传到FastDFS后Storage服务端将返回的文件索引（FID），其中文件名是根据FastDFS自定义规则重新生成的，而不是原始文件名，例如： group2/M00/00/89/eQ6h3FKJf_PRl8p4AUz4wO8tqaA688.apk

使用http下载时如不加处理，显示给用户的文件名会是这样的eQ6h3FKJf_PRl8p4AUz4wO8tqaA688.apk，这样的用户体验很不好。由于FastDFS不会存储原始文件名，也没有提供恢复原始文件名的方法，所以需要应用系统自己想办法恢复原始文件名。 解决方法

通过在项目中多次尝试，找到一种较简单的实现方法，实现过程如下：

一，应用系统在上传文件到FastDFS成功时将原始文件名和“文件索引（FID）”保存下来（例如：保存到数据库）。

二，用户点击下载的时用Nginx的域名和FID拼出url，然后在url后面增加一个参数，指定原始文件名。例如： http://121.14.161.48:9030/group2/M00/00/89/eQ6h3FKJf_PRl8p4AUz4wO8tqaA688.apk?attname=filename.apk

三，在Nginx上进行如下配置，这样Nginx就会截获url中的参数attname，在Http响应头里面加上字段 Content-Disposition “attachment;filename=$arg_attname”。

```
location /group2/M00 {
root /data/store/data;
if ($arg_attname ~ "^(.*).apk") {
    add_header Content-Disposition "attachment;filename=$arg_attname";
}
ngx_fastdfs_module;
}
```

四，浏览器发现响应头里面有Content-Disposition “attachment;filename=$arg_attname”时，就会把文件名显示成filename指定的名称。

完整的请求和响应消息如下：

请求包：

```
Request URL:http://121.14.161.48:9030/group2/M00/00/89/eQ6h3FKJf_PRl8p4AUz4wO8tqaA688.apk?attname=filename.apk
Request Method:GET
Status Code:200 OK
Request Headersview source
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding:gzip,deflate,sdch
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
Connection:keep-alive
Host:121.14.161.48:9030
Referer:http://appandroidpcfront.test.uae.uc.cn/apps
User-Agent:Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.57 Safari/537.36
Query String Parametersview sourceview URL encoded
attname:filename.apk
```

返回包： Response Headersview source Accept-Ranges:bytes Connection:keep-alive Content-Disposition:attachment;filename=filename.apk Content-Length:21821632 Date:Thu, 28 Nov 2013 11:40:46 GMT Last-Modified:Mon, 18 Nov 2013 02:48:19 GMT Server:nginx/1.4.3

### 经验二：从文件的使用技巧

应用背景

使用FastDFS存储一个图片的多个分辨率的备份时，希望只记录源图的FID，并能将其它分辨率的图片与源图关联。可以使用从文件方法。 解决方法

名词注解：主从文件是指文件ID有关联的文件，一个主文件可以对应多个从文件。

```
主文件ID = 主文件名 + 主文件扩展名
从文件ID = 主文件名 + 从文件后缀名 + 从文件扩展名
```

以本场景为例：主文件为原始图片，从文件为该图片的一张或多张缩略图。

流程说明：

```
先上传主文件（即：原文件），得到主文件FID
然后上传从文件（即：缩略图），指定主文件FID和从文件后缀名，上传后得到从文件FID。
```

java伪代码，如下：

```
public class FastDFSUtils {    
private static Logger logger = Logger.getLogger(FastDFSUtils.class);
static{
    try {
        ClientGlobal.init("D:/WorkSpace/app-filesystem/conf/fdfs_client.conf");
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}

public static String uploadFile(String filePath) throws Exception{        
    String fileId = ""; 
    String fileExtName = ""; 
    if (filePath.contains(".")) { 
        fileExtName = filePath.substring(filePath.lastIndexOf(".") + 1); 
    } else { 
        logger.warn("Fail to upload file, because the format of filename is illegal."); 
        return fileId; 
    } 

    //建立连接 
    /*.......*/

    //上传文件 
    try { 
        fileId = client.upload_file1(filePath, fileExtName, null); 
    } catch (Exception e) { 
        logger.warn("Upload file \"" + filePath + "\"fails"); 
    }finally{
        trackerServer.close();
    }        
    return fileId; 
}


public static String uploadSlaveFile(String masterFileId, String prefixName, String slaveFilePath) throws Exception{
    String slaveFileId = ""; 
    String slaveFileExtName = ""; 
    if (slaveFilePath.contains(".")) { 
        slaveFileExtName = slaveFilePath.substring(slaveFilePath.lastIndexOf(".") + 1); 
    } else { 
        logger.warn("Fail to upload file, because the format of filename is illegal."); 
        return slaveFileId; 
    } 

    //建立连接 
    /*.......*/

    //上传文件 
    try { 
        slaveFileId = client.upload_file1(masterFileId, prefixName, slaveFilePath, slaveFileExtName, null); 
    } catch (Exception e) { 
        logger.warn("Upload file \"" + slaveFilePath + "\"fails"); 
    }finally{
        trackerServer.close();
    }

    return slaveFileId;
}

public static int download(String fileId, String localFile) throws Exception{   
    int result = 0;
    //建立连接 
    TrackerClient tracker = new TrackerClient();
    TrackerServer trackerServer = tracker.getConnection();
    StorageServer storageServer = null;
    StorageClient1 client = new StorageClient1(trackerServer, storageServer); 

    //上传文件 
    try { 
        result = client.download_file1(fileId, localFile); 
    } catch (Exception e) { 
        logger.warn("Download file \"" + localFile + "\"fails"); 
    }finally{
        trackerServer.close();
    }

    return result;
}

public static void main(String[] args) {
    try {
            String masterFileId = uploadFile("D:/Tmp/apk/t01134ede0e696735e7.png");
            System.out.println(masterFileId);
            download(masterFileId, "D:/Tmp/apk/master.png");

            String slaveFileId = uploadSlaveFile(masterFileId, "_120x120", "D:/Tmp/apk/PC.png");
            System.out.println(slaveFileId);

            download(slaveFileId, "D:/Tmp/apk/slave.png");
        } catch (Exception e) {
            logger.error("upload file to FastDFS failed.", e);
        }
    }
}
```

上面代码运行后打印的文件Id为：

```
主文件：group1/M00/00/00/wKhbylJx1zkIAAAAAAApPcQL87AAAAAAQCmDxUAAClV522.png
    从文件：group1/M00/00/00/wKhbylJx1zkIAAAAAAApPcQL87AAAAAAQCmDxUAAClV522_120x120.png
```

### 注意：

FastDFS中的主从文件只是在文件ID上有联系。FastDFS server端没有记录主从文件对应关系，因此删除主文件，FastDFS不会自动删除从文件。删除主文件后，从文件的级联删除，需要由应用端来实现。