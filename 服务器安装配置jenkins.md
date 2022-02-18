

## 服务器安装配置jenkins(已有java环境的前提下)

[参考文章](https://blog.csdn.net/w6990548/article/details/106242009)

1. 下载jenkins到服务器

```shell
https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/
```

2. 在服务器上安装jenkins

   ```shell
   yum -y localinstall jenkins-2.319.3-1.1.noarch.rpm
   ```

   

注：[清华大学jenkins开源镜像](https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/)

3. jenkins启动、关闭、查看状态 \

```SHELL
启动: systemctl start jenkins
```

```shell
关闭： systemctl enable jenkins
```

```shell
查看端口占用状态： netstat -tunlp
```

4.提升jenkin的权限

![image-20220217155925847](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217155925847.png)

### # 配置github hooks

![image-20220217155536123](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217155536123.png)



## 进行操作服务器凭据生成，和连接github服务器

4. 系统配置=> 全局配置找到github服务器，勾选管理hook

   ![image-20220217135459847](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217135459847.png)

5. 点击添加，添加凭据，选择类型ssh username with private key类型，

   用户名随便填，将服务器的私钥复制在private key中

   ![image-20220217154815465](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217154815465.png)

   

   

6. 点击连接测试，出现蓝色框文字，则代表成功![image-20220217152443646](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217152443646.png)

   

   ## 创建多分支流水线项目，并完成配置

   1.创建多分支流水线项目

![image-20220217153052864](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217153052864.png)

2. 点击项目配置，进入配置页，点击分支源的添加凭据

   ![image-20220217153824513](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217153824513.png)

   3.因为是私人仓库，所以需要使用username with password类型，用户名随便起，密码必须要github access-token

   ![image-20220217154456194](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217154456194.png)

   4.填入项目仓库地址，点击验证，出现相关文字，则代表连接成功

   ![image-20220217155256545](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217155256545.png)

   ### **生成github-token过程如下**

   1.点击setting

   ![image-20220217135934770](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217135934770.png)

   2.点击Developer settings![image-20220217140004495](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217140004495.png)

   3.点击personnal access tokens

   4.点击generate new token

   ![image-20220217140126023](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217140126023.png)

   5.起个名字，还要勾选repo选项，然后**复制产生的github-token**![image-20220217151717627](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217151717627.png)





## 遇到的问题

1.提示仓库没权限

解决方法：在服务器上生成ssh-key将公钥，配置在github上

![image-20220217160659764](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217160659764.png)

2.提示yarn command not found

解决方法：在系统的全局配置中，环境变量选项，添加服务器上$PATH的值

![image-20220217160206817](C:\Users\30962\AppData\Roaming\Typora\typora-user-images\image-20220217160206817.png)







后续如果需要jenkins需要将内容打包到非本机服务器可以看看 Publish over FTP 这个插件



jenkinsFile文件可以参考 搜搜 `Jenkinsfile Pipeline`语法



下面是公司里一个项目的Jenkinsfile文件的配置(用的插件Publish over SSH,现在这个插件漏洞太多不再支持)

```shell
pipeline {
  agent any
  environment {
    IS_MASTER = sh (script: "if [ \"$BRANCH_NAME\" == \"master\" ];then echo -n true;else echo -n false;fi", returnStdout: true)
  }
  stages {
    stage('Prepare') {
      steps {
        sh '''
          git submodule sync --recursive
          git submodule update --init --recursive -f
          yarn
        '''
      }
    }

    stage('Build & Deploy') {
      steps {
        sh '''
          branch='2.2'
          if $IS_MASTER;
          then yarn ci $branch;
          else yarn ci $BRANCH_NAME;
          fi;
        '''
      }
    }
  }
}
```

