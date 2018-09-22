# Docker+Jenkins+Gogs 自动构建.Net Core

-------
> 引言
> jenkins+gags 全部采用Docker安装，通过jenkins插件ssh调用外部Docker构建
> 主要实现功能：
> git代码提交至Gogs，Jenkins自动构建至Docker 

必要条件
1.Centos 7
2.Docker(题主18.06.1-ce)
3.Dot Net Core(2.1.4 ) .Net Core世界第一☝️不接受反驳🤐 😜

-------


1. 安装Jinkins
    [镜像传送门 https://hub.docker.com/r/jenkins/jenkins/](https://hub.docker.com/r/jenkins/jenkins/)
    
    1.拉取jenkins
    ```
    docker pull jenkins/jenkins
    ```
    2.启动Docker容器
    
    ```
    #新建文件夹 
    mkdir /var/jenkins
    
    #赋予权限
    chmod 777 /var/jenkins
    ```
    
    ```
    docker run -p 8080:8080 -p 50000:50000 -d  -v /var/jenkins:/var/jenkins_home --name myjenkins  jenkins/jenkins
    ```
    
    3.打开浏览器配置jenkins
    http://xxxx:8080 （8080对应上面映射的本地端口）
    ![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004801354-1407793088.jpg)



    4.查看管理员密码
```
docker logs myjenkins
或
进入myjenkins容器
docker exec -it myjenkins bash
输入上图红色路径
cat xxxxx 复制密码输入
```
5.安装推荐插件 （等待安装完成） 
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004626873-987770888.png)


    6.配置管理员密码
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004654988-1891887061.jpg)



    7.安装完成
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004721812-780468053.png)


    8.安装Publish Over SSH
进入系统管理-》插件管理-》可选插件 搜索Publish Over SSH  直接安装

    9.配置Publish Over SSH
进入系统管理-》系统设置 找到Publish Over SSH
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004825217-2138317048.jpg)




2. 安装Gogs(此步骤可忽略，可以gitlab或者github、svn)
    [镜像传送门 https://hub.docker.com/r/jenkins/jenkins/](https://hub.docker.com/r/jenkins/jenkins/)
    
    1.拉取Gogs
    ```
    docker pull gogs/gogs
    ```
    2.启动Docker容器
    
    ```
    #新建文件夹 
    mkdir -p /var/gogs
    
    #赋予权限
    chmod 777 /var/gogs

    ```
    
    ```
    docker run --name=gogs -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs
    ```
    
    3.打开浏览器配置gogs
    http://xxxx:10080 （10080对应上面映射的本地端口）
    ![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004853323-1842892360.jpg)



3. 创建.Net Core 项目并上传至gogs git
    ```
    dotnet new mvc --name helloCore
    ```
    进入项目目录上传至git
```shell
    # git init
    
    # git add .
    
    # git commint -m ”提交“
    
    # git remote add origin {远程仓库地址}
    # git push -u origin master
```
在项目根目录配置DockerFile

    ```
    FROM microsoft/aspnetcore-build as build-env
    WORKDIR /code
    COPY *.csproj ./
    RUN dotnet restore
    COPY . ./
    RUN dotnet publish -c Release -o out
    
    FROM microsoft/aspnetcore
    WORKDIR /app
    COPY --from=build-env /code/out ./
    
    EXPOSE 80 
    ENTRYPOINT [ "dotnet", "helloCore.dll" ]  
    ```
在项目根目录配置docker-compose.yml
    ```
    version: '3'
    
    services: 
      web:
        build: . 
        container_name: 'helloCore'
        ports:
          - '8888:80'
    
    ```


4. 配置jenkins任务

    1.新建自由任务

 ![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004918000-1611592964.jpg)

   
 2.配置git或者svn

![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923004946348-1584815761.jpg)



3.添加构建步骤
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923005003176-1953232496.jpg)



```
#代码如下
docker stop helloCore; #停止已存在的容器
docker rm helloCore; #删除容器
cd /var/jenkins/workspace/helloCore; #进入jenkins拉取目录
pwd;
docker-compose up --build -d;#执行docker-compose构建
docker images;
```
4.一键构建
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923005036415-244413199.jpg)



5.查看构建控制台
如果构建有错误，需按控制台错误修复
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923005057848-1629857455.jpg)



6.预览
![](https://img2018.cnblogs.com/blog/286805/201809/286805-20180923005116260-17973763.jpg)




#### 致此

