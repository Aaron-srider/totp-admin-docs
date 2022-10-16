# 文件管理

## Artifact与文件

两个最基本的实体是：Artifact和文件。Artifact可以理解为一个存储文件的容器，可以在Aritfact中上传文件或文件夹。Artifact、File与用户的逻辑视图如图所示：

![image-20220916105846966](totp-admin-后端架构-文件与Artifact.png)

每个用户都可以创建多个Aritfact，并可将一个Artifact共享到多个设备模板。

## 文件索引

文件存储采用数据库索引+Linux文件系统相结合的形式。索引和文件系统的大致关系如下图：

![avatar](./后端架构-文件存储-索引.png)

如上图所示，数据库中不存储文件本身，而是存储文件的元数据。另外，为了方便管理文件版本，使用file_version表记录文件的各个版本的元数据。

文件版本的path属性指向了文件的最终存储路径。

## 文件存储

不存储所谓的文件本身，只存储文件的各个版本。如不指定文件版本，一个文件指的就是该文件的当前版本。

Artifact存储：每个Artifact对应磁盘上一个具体的文件夹。所有的Artifact存储在一个指定的文件夹中（下称RootPath），id为 xxx 的Artifact 相对RootPath的路径为：/.xxx.versions。

文件夹存储：每个文件夹对应磁盘上的一个文件夹，文件夹名就是文件夹的file_id。

文件存储：每个文件都有自己的版本文件夹，文件的所有版本存储在该文件夹中。文件夹的名称由两部分组成：file_id + .versions。各个版本的文件名就是对应版本的版本号。例如，文件：/app/src/main.cpp （app，src，main.cpp的id分别为xxx，yyy，zzz）有3个版本，版本号分别是：0.0.1、0.0.2和0.0.3，文件所在的Artifact的Id为：aaa，那么该文件的版本目录如下图所示：

![image-20220916110818383](totp-admin-后端架构-文件版本管理.png)

## Artifact与设备模板

用户创建了Artifact后，可选择关联到指定的设备模板（设备模板的属主可以不是自己），所以Artifact与设备模板是多对多的关系。

## 文件操作权限控制

大致将文件的操作分为4类：查看（browse），下载（download），修改（modify），上传（upload）。

* 查看：查看Artifact中的所有文件夹及文件，查看所有文件的历史版本。
* 下载：下载Artifact中的所有文件夹及文件，及其历史版本。
* 修改：删除Artifact，修改Artifact名，在Artifact中上传文件或文件夹，删除Artifact中的文件夹或文件，修改Artifact中文件夹或文件的名称。
* 上传：上传文件夹或文件到Artifact中。

文件权限：上述四种文件操作的集合，大致分为两种：可读写，只读。

* 可读写：可以执行所有4种操作。
* 只读：可以查看（browse），下载（download）。

文件权限控制逻辑：

* 用户可以对自己的Artifact执行所有上述4类操作。
* 对于共享到设备模板的Artifact，在共享时指定设备模板的属主可以对文件进行的操作。设备模板属主对模板的Artifact能进行的操作范围将不超过共享时Artifact属主指定的操作。

为了达到权限控制的目的，后端将文件操作细分为两步，首先请求操作token，再由token对文件进行操作。下图展示下载文件的大致步骤：

![image-20220916113502819](totp-admin-后端架构-文件权限控制-示例-下载文件.png)

## 开放能力

对第三方app开放所有4中文件操作，与前端唯一不同的是，第三方应用调用 GET /app-access/file/op/token 请求文件操作token。下面展示第三方App下载文件的步骤：

![image-20220916114140880](totp-admin-后端架构-文件权限控制-示例-下载文件-appaccess.png)