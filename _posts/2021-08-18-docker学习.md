# docker命令
https://www.cnblogs.com/wyt007/p/11154156.html

### 镜像操作命令
```
systemctl start docker //启动docker
systemctl stop docker //停止docker
docker search <需要的镜像名称>  //查找镜像
docker images //查看安装的镜像
docker pull <镜像名称>  //拉取镜像
docker rmi <镜像ID>  //删除镜像
docker `docker images -q` //删除全部镜像
```
### 容器命令
1. 查看容器
查看运行容器
```
docker ps
```
查看全部容器
```
docker ps -a
```
2. 启动容器
```
docker run <选项>
 eg.运行名为mysql1的容器使用mysql镜像
docker run -it --name=mysql1 mysql /bin/bash
```
守护方式运行
```
docker -di  --name=<容器名> <镜像>
```
3. 推出容器
```
exit
```
4. 进入容器
```
docker exec -it <容器名> /bin/bash
```
5. 复制文件
```
docker cp <文件名> <容器名>:<路径>  /bin/bash
将hello.class复制到容器mysql1的根目录下
docker cp hello.class mysql1:/ /bin/bash
```
6. 目录挂载
```
docker run -it --name=<容器名> -v <挂载路径>:<容器路径> <镜像名> /bin/bash
eg.挂载/usr/local/myhtml到容器目录
docker run -it --name=mysql01 -v /usr/local/myhtml:/usr/local/myhtml mysql /bin/bash
```
7. 查看容器信息
```
docker inspect <容器名> <选项>
使用 --format='{{.筛选选项}}'  进行筛选
```
8. 删除容器
```
docker rm <容器名>
```
### 部署Mysql容器
创建mysql容器，名为mysql01,映射宿主机的33306端口号为容器的3306端口号，配置mysql的root密码为991226Ab#
```
 docker run -it --name=mysql01 -p 33306:3306 -e MYSQL_ROOT_PASSWORD=991226Ab# mysql
```
此时使用工具即可连接

## 部署Redis
拉取redis镜像
构建容器
```
docker run -it --name=myredis -p 6379:6379 redis
```
使用客户端工具访问
## 部署nginx
拉取nginx镜像
构建容器
```
docker run -it --name=mynginx -p 80:80 nginx 
```
在浏览器访问80端口就可以访问到
要修改配置：
配置文件位置在/etc/nginx

## 备份与迁移

1. 保存容器为镜像
```
docker commit <容器名> <要保存的镜像名>
eg.保存mysql01容器为镜像
docker commit mysql01 mysql_01
```
2. 保存镜像为一个文件
```
eg.保存mysql_01镜像为文件mysql.tar
docker save -o mysql.tar mysql_01
```
3. 读取文件为镜像
```
eg.将mysql.tar读取
docker load -i mysql.tar
```
## Dockerfile
1. 常用命令
- FROM 镜像名:版本名  定义使用哪个镜像构建流程
- MAINTAINER user_name  声明镜像的创建者
- ENV key value 设置环境变量(可写多条)
- RUN command  
- ADD source_dir/file dest_dir/file 将宿主机文件复制到容器，若是压缩文件，将会自动解压
- COPY  source_dir/file dest_dir/file 与add相同，但是不会解压
- WORKKDIR path_dir 设置工作目录
2. 构建jdk1.8的docker镜像
创建目录,存放镜像文件和压缩包
```
mkdir -p /usr/local/dockerjdk8
```
上传jdk8的压缩包到宿主机
移动压缩包到dockerjdk8目录下
进入dockerjdk8,创建Dockerfile
编写Dockerfile
```shell
FROM centos
MAINTAINER lizhengliang
WORKDIR /usr
RUN mkdir /usr/local/java
ADD jdk-8u301-linux-x64.tar.gz /usr/local/java/

ENV JAVA_HOME /usr/local/java/jdk1.8.0_301
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
```
构建镜像
```
docker build -t='jdk1.8' .   //最后这个点代表当前目录
```
## 构建私服
拉取registry镜像
```
docker pull registry
```
修改配置文件
```
vim /etc/docker/daemon.json
{
  	"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"],
	"insecure-registries":["192.168.182.129:5000"]
}
```
创建registry容器
```
docker run -d -v /repositories:/var/lib/registry -e REGISTRY_STORAGE_DELETE_ENABLED=true -p 5000:5000 --restart=always --privileged=true --name myregistry registry'
```
此时访问ip地址：5000/v2/_catalog即可看到一个json对象，键为registries,值为空，因为没有添加镜像

添加镜像到私服
首先要给需要上传的镜像打标签
```
docker tag <镜像名称> <标签名>
标签名格式：
私服地址/镜像名称

docker tag jdk1.8 192.168.182.129:5000/jdk1.8
```
上传到私服
```
docker push 192.168.182.129:5000/jdk1.8
```
再次访问ip地址：5000/v2/_catalog即可看到一个json对象，键为registries,值为一个数组，其中有一个镜像叫做jdk1.8



删除repo

```
docker exec registry rm -rf /var/lib/registry/docker/registry/v2/repositories/<私服中的镜像名>
```

5.清楚掉blob

```
docker exec registry bin/registry garbage-collect /etc/docker/registry/config.yml
```

## 远程操作docker部署微服务

在项目中添加Dockermaven插件依赖

```xml
 <build>
        <finalName>app</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>1.5.4.RELEASE</version>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.1.0</version>
                <executions>
                    <execution>
                        <id>build-image</id>
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <imageName>192.168.182.129:5000/${project.artifactId}:${project.version}</imageName>
                    <baseImage>jdk1.8</baseImage>
                    <entryPoint>["java", "-jar","/${project.build.finalName}.jar"]</entryPoint>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                    <dockerHost>http://192.168.182.129:2375<//dockerHost>
                </configuration>
            </plugin>

        </plugins>
    </build>
```

打开docker的远程访问
修改配置文件,

```
vim /lib/systemd/system/docker.service
修改ExecStart
ExecStart=/usr/bin/dockerd --containerd=/run/containerd/containerd.sock

vim /etc/docker/daemon.json
{
  	"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"],
   	"hosts":["192.168.182.129:2375","unix://var/run/docker.sock"],
	"insecure-registries":["192.168.182.129:5000"]
}
```
 重新加载配置文件并重启docker
 ```
 systemctl daemon-reload
 systemctl restart docker
 ```
 外部访问需要开放指定端口号
 查看开放的端口号

 ```
 firewall-cmd --list=ports
 ```
 开放端口号
 ```
 firewall-cmd --add-port=80/tcp --permanent --zone=public 
 ```
 重启防火墙
 ```
 firewall-cmd --reload
 ```
 查看全部端口号
 ```
 netstat -ntlp
 ```
查看指定端口号

 ```
  netstat -ntlp|grep 端口号
 ```
 使用mvn命令构建项目
 ```
 mvn install 
 ```
 使用docker插件的构建命令构建镜像,并且上传到服务器
 ```
 mvn docker:build -DpushImage
 ```
 此时宿主机上就会有这个项目的镜像，可以使用docker images查看就能看到

构建项目容器，启动就完成了
```
docker run -it --name=myspringboot -p 8080:8080 192.168.182.129:5000/springboot-mybatis
```

 访问192.168.182.129:8080/department/findAll即可

