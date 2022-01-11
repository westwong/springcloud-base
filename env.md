## 运行环境

- 该项目组件均在[OpenJDK11](http://jdk.java.net/archive/)下编译运行，所以建议您使用 JDK 11或上以版本。

- 包管理工具使用使用[Maven](https://maven.apache.org/download.cgi),版本3.0以上应该都可以，我使用的idea自带Maven版本为Maven 3.6.1。

- 使用的Spring-Cloud版本为：Greenwich.SR2

  ```xml
   <properties>
          <java.version>11</java.version>
          <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
   </properties>
  ```

- 数据库使用[Mysql](https://www.mysql.com/) 8.0。（推荐docker安装）

- 缓存库使用[Redis](https://redis.io/download) 5.0。（推荐docker安装）

- 如果你习惯常规Spring Boot 打成jar包发布并部署那么看到这里就差不多。

## docker

- 这是一个简单的docker安装教程。

- docker运行环境为CentOS 7。

- 我的docker使用的是linux版本，运行我的虚拟机中。

- 安装docker

  ```shell
  # step 1: 安装必要的一些系统工具
  sudo yum install -y yum-utils device-mapper-persistent-data lvm2
  # Step 2: 添加软件源信息
  sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  # Step 3: 更新并安装 Docker-CE
  sudo yum makecache fast
  sudo yum -y install docker-ce
  # Step 4: 开启Docker服务
  sudo service docker start
  # Stap 5: 开机自动启动docker
  sudo systemctl enable docker
  
  注意：其他注意事项在下面的注释中
  # 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
  # vim /etc/yum.repos.d/docker-ce.repo
  #   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
  #
  # 安装指定版本的Docker-CE:
  # Step 1: 查找Docker-CE的版本:
  # yum list docker-ce.x86_64 --showduplicates | sort -r
  #   Loading mirror speeds from cached hostfile
  #   Loaded plugins: branch, fastestmirror, langpacks
  #   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
  #   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
  #   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
  #   Available Packages
  # Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
  # sudo yum -y install docker-ce-[VERSION]
  # 注意：在某些版本之后，docker-ce安装出现了其他依赖包，如果安装失败的话请关注错误信息。例如 docker-ce 17.03 之后，需要先安装 docker-ce-selinux。
  # yum list docker-ce-selinux- --showduplicates | sort -r
  # sudo yum -y install docker-ce-selinux-[VERSION]
  
  # 通过经典网络、VPC网络内网安装时，用以下命令替换Step 2中的命令
  # 经典网络：
  # sudo yum-config-manager --add-repo http://mirrors.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
  # VPC网络：
  # sudo yum-config-manager --add-repo http://mirrors.could.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
  ```

- 如果你想实现远程编译需要开启docker远程api

  首先找到docker的宿主主机文件:

  ```bash
  vi /lib/systemd/system/docker.service
  ```

  修改以ExecStart开头的行：

  ```bash
  ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
  ```

  然后重启服务:

  ```shell
  sudo systemctl daemon-reload
  sudo service docker restart
  ```

  测试：

  ```bash
  curl http://localhost:2375/version
  ```

  **还就是记住要开放防火墙的端口**,我踩过得坑不希望你们再掉进去！

- 然后就可以修改pom文件中对应的地址

  ```xml
  <dockerHost>http://192.168.0.45:2375</dockerHost> <!-- docker  -->
  ```

  相关[docker-maven-plugin](https://github.com/spotify/docker-maven-plugin)的使用配置方法自行百度，我就不开展了。
- 最后如果你想使用docker镜像云仓库可以使用[阿里云](https://help.aliyun.com/document_detail/60997.html)的，免费申请。
  

  

  