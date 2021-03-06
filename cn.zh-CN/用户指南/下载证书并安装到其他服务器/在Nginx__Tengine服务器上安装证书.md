# 在Nginx/Tengine服务器上安装证书 {#concept_n45_21x_yfb .concept}

您可以从阿里云SSL证书服务控制台下载证书安装到您的Nginx/Tengine服务器上。

## 背景信息 {#section_ida_d3u_w8o .section}

本文档以CentOS 7、Nginx 1.15.6为例。

本文档证书名称以**domain name**为示例，如证书文件名称为**domain name.pem**，证书密钥文件名称为**domain name.key**。

下载的Nginx证书压缩文件解压后包含：

-   .pem：证书文件。PEM文件的扩展名为CRT格式。
-   .key：证书的密钥文件。申请证书时如果未选择**自动创建CRS**，则下载的证书文件压缩包中不会包含.key文件，需要您将自己手动常见的私钥文件拷贝到**cert**目录下。

**说明：** **.pem**扩展名的证书文件采用Base64-encoded的PEM格式文本文件，您可根据需要修改成其他扩展名。

证书的格式详见[主流数字证书都有哪些格式](https://www.alibabacloud.com/help/faq-detail/42214.htm)。

## 操作指南 {#section_ydh_4qb_1gb .section}

1.  登录阿里云[SSL证书控制台](https://yundunnext.console.aliyun.com/?spm=5176.2020520001.aliyun_sidebar.108.356a4bd3MLXFkb&p=cas#/overview/cn-hangzhou)。
2.  在SSL证书页面，点击**已签发**标签，定位到需要下载的证书并单击证书卡片右下角的**下载**打开**证书下载**对话框。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/66002/156274615139176_zh-CN.jpg)

3.  在**证书下载**对话框中定位到Nginx/Tengine服务器，并单击右侧**操作**栏的**下载**将Nginx版证书压缩包下载到本地。
4.  解压Nginx证书。

    您将看到文件夹中有2个文件：

    -   证书文件（以.pem为后缀或文件类型）
    -   密钥文件（以.key为后缀或文件类型）
    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/66002/156274615133690_zh-CN.png)

5.  在Nginx安装目录下创建**cert**目录，并将下载的证书文件和密钥文件拷贝到**cert**目录中。

    **说明：** 如果您在申请证书时选择**手动创建CSR**文件，请将对应的密钥文件放到**cert**目录中，并命名为**domain name.key**。

6.  打开**Nginx安装目录** \> **conf文件夹** \> **nginx.conf文件**，在**nginx.conf文件**中找到以下属性：

    ``` {#codeblock_c5v_l9b_w7a}
    
    # HTTPS server
      server {
      listen 443;
      server_name localhost;
      ssl on;
      ssl_certificate cert.pem;
      ssl_certificate_key cert.key;
      ssl_session_timeout 5m;
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
      ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
      ssl_prefer_server_ciphers on;
      location / {
    						
    ```

    修改nginx.conf文件如下：

    ``` {#codeblock_7ms_9a2_enf}
    
    # 以下属性中以ssl开头的属性代表与证书配置有关，其他属性请根据自己的需要进行配置。
    server {
    listen 443;
    server_name localhost;  # localhost修改为您证书绑定的域名。
    ssl on;   #设置为on启用SSL功能。
    root html;
    index index.html index.htm;
    ssl_certificate cert/domain name.pem;   #将domain name.pem替换成您证书的文件名。
    ssl_certificate_key cert/domain name.key;   #将domain name.key替换成您证书的密钥文件名。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  #使用此加密套件。
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #使用该协议进行配置。
    ssl_prefer_server_ciphers on;   
    location / {
    root html;   #站点目录。
    index index.html index.htm;   
    }
    }
    							
    ```

7.  保存**nginx.conf文件**后退出。
8.  重启Nginx服务器。

    **说明：** 如果您有配置多个虚拟主机Include conf/vhost，参考[虚拟主机配置SSL证书](#section_wy0_022_q1s)进行配置。

9.  （可选步骤）设置http请求自动跳转https。

    在需要跳转的http站点下添加以下rewrite语句，实现http访问自动跳转到https页面

    ``` {#codeblock_807_sev_ayj}
    server {
     listen 80;
     server_name localhost; 
    rewrite ^(.*)$ https://$host$1 permanent;
     location / {
    index index.html index.htm;
    }
    ```


## 虚拟主机配置SSL证书 {#section_wy0_022_q1s .section}

1.  打开虚拟主机配置文件vhost.conf或\*.conf，复制以下内容粘贴到下方位置、将端口改为443（https默认端口）并增加证书相关配置。

    ``` {#codeblock_2md_65b_n58}
    server {
     listen 80;
     server_name localhost ;
     location / {
    index index.html index.htm;
    }
    server {
    listen 443;
    server_name localhost;
    ssl on;
    root html;
    index index.html index.htm;
    ssl_certificate cert/domain name.pem;   #将domain name.pem替换成您证书的文件名。
    ssl_certificate_key cert/domain name.key;   #将domain name.key替换成您证书的密钥文件名。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
    index index.html index.htm;
    }
    ```

2.  保存**nginx.conf文件**后退出。
3.  重启Nginx服务器。

安装证书相关文档：

-   [在Tomcat服务器上安装SSL证书](intl.zh-CN/用户指南/下载证书并安装到其他服务器/Tomcat服务器安装SSL证书/安装PFX格式证书.md#)
-   [在Apache服务器上安装SSL证书](intl.zh-CN/用户指南/下载证书并安装到其他服务器/在Apache服务器上安装SSL证书.md#)
-   [Ubuntu系统Apache 2部署SSL证书](../../../../intl.zh-CN/最佳实践/Ubuntu系统Apache 2部署SSL证书.md#)
-   [我获取到的数字证书如何配置在自己的Apache中？](../../../../intl.zh-CN/常见问题/常见问题/我获取到的数字证书如何配置在自己的Apache中？.md#)
-   [在IIS服务器上安装证书](intl.zh-CN/用户指南/下载证书并安装到其他服务器/在IIS服务器上安装证书.md#)
-   [CentOS系统Tomcat 8.5/9部署SSL证书](../../../../intl.zh-CN/最佳实践/CentOS系统Tomcat 8.5__9部署SSL证书.md#)
-   [Jetty服务器配置SSL证书](../../../../intl.zh-CN/常见问题/常见问题/Jetty服务器配置SSL证书.md#)

