# Dockerfile基础：
  1. 每条指令名大小且必须有值
  2. 从上至下顺序执行
  3. #表示注释
  4. 每条指令都会创建一个新的镜像层，并对镜像进行提交。# 分层架构
  5. 一定要有一个基础镜像(父镜像)

# Dockerfile指令:
- FROM:父镜像，FROM极有可能产生中间层镜像，不会占用太多磁盘
	
- COPY: 宿主机src 容器dest，复制到容器中 # COPY index.html /usr/local/tomcat/webapps/ROOT/。 COPY只能使用相对路径，且最好不要使用..
	
	当前文件夹.会被打包传输至docker server中进行构造，Dockerfile中如果写..，不是宿主机当前目录的..，而是docker server当前目录的..
	因此**要拷贝到宿主机的文件应该与Dockerfile放在同一个目录**，**且COPY应该不使用..**
	且当前目录最好是新的空的**，不然打包当前目录会花很多时间（我在桌面测试，结果传输了3G的内容给docker server）
	
- MAINTAINER:作者 + 邮箱

- LABEL:标签

- RUN:镜像->容器时，即容器构建时需要执行的额外命令。可多个。多个用\ && xxxx来写。如tar -zxvf myproject.tar.gz \ && rm -f myproject.tar.gz

- EXPOSE:当前容器占用容器内部的端口声明，只是一个声明。当使用-P（随机映射）时，会自动映射EXPOSE声明的端口。实例：EXPOSE 8080 3306 暴露多个端口用空格隔开

- WORKDIR:进入容器时的初始目录(如redis的WORKDIR /data)，**也是构建镜像时执行命令的当前目录**。

- ENV:构建环境变量，可以有多个ENV。格式：key value，如 MY_PATH /~。可以在下文使用：如WORKDIR $MY_PATH。也可以在docker命令中使用如：docker cp f5fa58c8c1c4:$LOG_PATH_SMARTLIGHT /home/logs/log
	
- ADD:将宿主机目录下的文件拷贝进镜像且自动处理URL和解压tar压缩包。如：ADD my_app.tar /usr/local/my_app 不推荐使用

- VOLUME:容器数据卷，用于保存数据和持久化。挂载点指定了但不能指定对应主机的目录https://www.cnblogs.com/51kata/p/5266626.html

- CMD:指定容器运行时需要执行的命令，能配置多个CMD，但只有最后一个配置的生效。如果容器启动时指定了COMMAND那个最后一个CMD也会被覆盖。相当于默认启动命令。注意点：
	1. CMD nginx -g daemon off 不推荐使用。因为这里主进程是sh，执行完nginx -g daemon off后sh就退出了，容器为主进程服务，sh一结束容器就会关闭https://www.jianshu.com/p/78f4591b7ff0
	2. 推荐使用exec格式，如 CMD ["nginx", "-g", "daemon off;"]

- ENTRYPOINT:执行启动命令后缀，下面举例。
  1. 如果CMD ["crul","-s","http://ip.cn"]。那么启动时如果你想执行crul -s http://ip.cn -i，那么你必须写全:docker run app crul -s http://ip.cn -i。相当于运行容器时命令覆盖
  2. 如果ENTRYPOINT ["crul","-s","http://ip.cn"]。那么启动时如果你想执行crul -s http://ip.cn -i，那么可以直接加选项:docker run app -i即可。相当于运行容器时命令拼接。

- ONBUILD:如果有子镜像继承自己，执行onbuild

# 简单示范：构建Springboot项目镜像
  ```
  FROM java:openjdk-8-jre-alpine // 带jre8环境的父镜像，参考docker hub
  COPY target/xxx.jar /user/local/xxx/
  VOLUME: ["/user/local/xxx/logs"]
  EXPOSE: 8080
  ENTRYPOINT: ["java", "-jar", "/user/local/xxx/xxx.jar"]
  ```

# 构建镜像
- Docker build -f dockerFile -t docker.io/yukijudai/xxx:1.0.0
- -f指定Dockerfile文件 -t 指定标签一般为 仓库地址/仓库命名空间/项目名:版本 这种格式

# 上传至镜像仓库
- echo "$DOCKER_PASSWORD" | docker login docker.io -u "yukijudai" --password-stdin # 登录docker hub
- docker push docker.io/yukijudai/xxx:1.0.0 # 上传至镜像仓库
