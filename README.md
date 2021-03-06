# jenkins-构建docker

## 配置环境

下载jenkins
<pre>
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
mv jenkins.war /usr/local/src/
</pre>

运行jenkins
<pre>
nohup java -jar /usr/local/src/jenkins.war --ajp13Port=-1 --httpPort=8089 &
</pre>

安装jenkins的docker插件

![image](https://github.com/shenxinli/jenkins-/blob/master/jenkins-docker-plugin.png)

安装docker
请参照https://github.com/shenxinli/kubernetes/blob/master/README.md 中，安装docker一节

部署私有deoker仓库
<pre>
docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry  registry
</pre>

编辑docker启动参数
<pre>
vi /usr/lib/systemd/system/docker.service
</pre>
增加参数
<pre>
--tls=false -H unix:///var/run/docker.sock -H tcp://0.0.0.0:4375 --insecure-registry 172.18.170.87:5000
</pre>
写入docker-register的ip和端口号,如下图：

![image](https://github.com/shenxinli/jenkins-/blob/master/modify-docker-service.png)

在jenkins中配置配置私有docker仓库,[系统管理]-[系统设置]下新增云

![image](https://github.com/shenxinli/jenkins-/blob/master/jenkins-docker-instance.png)

## 创建部署任务

![image](https://github.com/shenxinli/jenkins-/blob/master/new-jenkins-task.png)

### 配置任务属性
设置git远程库和访问账户

![image](https://github.com/shenxinli/jenkins-/blob/master/jenkins-task-properties.png)

配置构建属性
![image](https://github.com/shenxinli/jenkins-/blob/master/jenkins-task-build.png)

上述配置完成后，已经可以在jenkins服务器上生成本地docker-image.点击[立即构建]，等待构建完成。
<pre>
[root@izwz9evja68nbrfvyzr9ecz docker]# docker images
</pre>
![image](https://github.com/shenxinli/jenkins-/blob/master/docker-images-local.png)

### 将本地docker-image，push到docker私有仓库中
由于jenkins服务器和docker私有仓库在同一台机器上，因此Ip用本地127.0.0.1
<pre>
docker tag chinawaytek/eureka-server 127.0.0.1:5000/eureka-server:latest
docker push 127.0.0.1:5000/eureka-server:latest
</pre>

![image](https://github.com/shenxinli/jenkins-/blob/master/jenkins-post-steps.png)

<pre>
[root@izwz9evja68nbrfvyzr9ecz docker]# curl http://127.0.0.1:5000/v2/_catalog
</pre>

![image](https://github.com/shenxinli/jenkins-/blob/master/show-docker-repo.png)
