## Cloud Toolkit 部署应用到阿里云轻量应用服务器

由于阿里云 ECS 云助手只能支持 VPC 网络机器，因此，轻量应用服务器只能通过 Host 模式手动添加机器，采用标准 SSH 协议来进行部署

![mark](http://man.hhaxmm.cn/blog/20190602/KMHm7w1yIn0s.png)

如上图所示，在菜单

`Tools - Alibaba Cloud - Alibaba Cloud View - Host`中打开机器视图界面，如下图：

![mark](http://man.hhaxmm.cn/blog/20190602/769m8WoRWbQu.png)

点击右上角`Add Host`按钮，出现添加机器界面

<img src='http://man.hhaxmm.cn/blog/20190602/Dk87id6EY37N.png' align ='left'>

# 部署

![mark](http://man.hhaxmm.cn/blog/20190602/vnlQatKHB6yX.png)

在 IntelliJ IDEA 中，鼠标右键项目工程名，在出现的菜单中点击 **Alibaba Cloud - Deploy to Host...**，会出现如下部署窗口：

<img src='http://man.hhaxmm.cn/blog/20190602/X5Hcgbcu53Gy.png' align ='left'>

在 Deploy to Host 对话框设置部署参数，然后单击 Run，即可执行初次部署。

## 部署参数说明：

- Deploy File：部署文件包含两种方式。
  - Maven Build：如果当前工程采用 Maven 构建，可以使用 Cloud Toolkit 直接构建并部署。
  - Upload File：如果当前工程并非采用 Maven 构建，或者本地已经存在打包好的部署文件，可以选择并直接上传本地的部署文件。
- Target Deploy host：在下拉列表中选择Tag，然后选择要部署的服务器。
- Deploy Location ：输入在 ECS 上部署路径，如 /root
- Commond：输入应用启动命令，如 sh /root/start.sh start platform-0.0.1-SNAPSHOT。

> 因为是SpringBoot打包的jar项目，我写了一个脚本用来启动SpringBoot项目，获取脚本内容，请关注我的微信公众号，回复“脚本”就可以获得

