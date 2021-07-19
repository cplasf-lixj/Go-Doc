# PM2部署流程(Mac)

## 1. SSH免密登录

### 1.1 生成秘钥

在客户端执行`ssh-keygen -t 秘钥名`命令生成公私秘钥

````shell
$ ssh-keygen -t my_key
````

该命令会在用户`.ssh`目录下创建秘钥

````shell
$ cd ~/.ssh
$ ls

my_key			# 私钥
my_key.pub  # 公钥
````

### 1.2 上传公钥到服务器

执行`ssh-copy-id`命令到服务器，假设服务器地址为：`10.10.1.9`，用户为`root`

````shell
$ ssh-copy-id -i ~/.ssh/my_key.pub root@10.10.1.9

# 查看写入到服务器的公钥内容
$ cat authorized_keys
````

### 1.3 测试免密登录

````shell
$ ssh root@10.10.1.9		#执行该命令测试免密登录是否成功
````

### 1.4 ssh-add(mac的坑)

如果免密登录不成功，需要调用`ssh-add`命令：

````shell
$ ssh-add -K my_key				# 通过ssh-add将私钥添加到ssh-agent的高速缓存中
````

## 2. 创建配置文件

### 2.1 生成配置文件

````shell
$ pm2 init			# 在项目中生成ecosystem.config.js文件
````

### 2.2 根据官方文档与项目需求进行配置

````javascript
module.exports = {
  apps: [
    {
      name: 'my app',
      script: 'npm run start',
      max_memory_restart: '200M',
      env_development: {
        NODE_ENV: 'development'
      },
      env_production: {
        NODE_ENV: 'production'
      },
      log_date_format: 'YYYY-MM-DD HH:mm Z',
      error_file: './pm2_logs/err.log',
      out_file: './pm2_logs/out.log'
    }
  ],
  deploy: {
    production: {
      user: 'root',											// 服务器访问用户名
      host: '10.10.1.9',								// 服务器IP
      ref: 'origin/develop',						// git分支
      repo: 'git@.....server.git',			// git地址
      path: '/home/apps/my_app',// app部署在服务器的路径
      'pre-deploy-local': '',
      'pre-setup': '处理setup的操作',
      'post-setup': 'setup后执行的任务',
      'pre-deploy': '部署前的任务',
      'post-deploy': '部署后的任务'
    }
  }
}
````

## 3. 执行部署CLI脚本

首先要在服务端安装pm2

````shell
$ pm2 deploy ecosystem.config.js production setup
````



# 遇到的问题

### 1. 部署成功，但是应用的启动脚本未执行。

错误原因： pre-deploy和post-deploy未执行。

解决方法：待解决。

### 2. 部署时，报npm命令找不到

错误原因：本地执行pm2 deploy，使用ssh执行远程命令，非交互式环境。

解决方法：需要在服务器上进行软连接

````shell
sudo ln -s 'path of npm' /usr/bin/npm
# 对pm2、node执行同样的操作
sudo ln -s 'path of pm2' /usr/bin/pm2
sudo ln -s 'path of node' /usr/bin/node
````

### 3. 执行部署脚本重新部署时，会报错误，提示应用已存在的错误：

错误原因：pm2部署应该是有版本管理的，好像没有成功，目前原因还没有定位到；

解决方法：在服务器上执行pm2命令，将应该停止并删除

````shell
$ pm2 stop 'my app'
$ pm2 delete 'my app'
````

### 4. 重新部署时，报source目录已存在的错误

错误原因：部署时会通过git clone将项目clone到服务器上的app目录，重新部署时，该文件已存在，所以重新colne会报错。

解决方法：在`pre-setup`中执行删除目录操作

````javascript
module.exports = {
  .....
  deploy: {
  	.....
  	'pre-setup': 'rm -rf /home/apps/my_app/source',
  	.....
	}
}
````

