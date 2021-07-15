# PM2 

## 1. 安装

````shell
npm install pm2@latest -g
# or
yarn global add pm2
````

## 2. 更新

````shell
pm2 update
````

## 3. 进程管理

### 3.1 启动

````shell
pm2 start app.js --name 'my-api'
pm2 start web.js --name 'web-interface'

# 停止
pm2 stop app_name	#pm2 stop web-interface
# 重启
pm2 restart app_name	#pm2 restart web-interface
# 重新加载
pm2 reload app_name  #pm2 reload web-interface
# 删除
pm2 delete app_name		#pm2 delete web-interface

# 可以使用以下内容替换app_name
# all  所有进程
# id   指定进程id

# 启动APP，并输出日志流
pm2 start api.js --attach

# 启动APP，并传参
pm2 start api.js -- arg1 arg2

# Crom time restart
# ----> CLI
pm2 start app.js --cron-restart="0 0 * * *"
# Or
pm2 restart app.js --cron-restart="0 0 * * *"
# ----> Configuration file
module.exports = {
	apps: [{
		name: 'Business News Watcher',
		script: 'app.js',
		instances: 1,
		cron_restart: '0 0 * * *',
		env: {
			NODE_ENV: 'development'
		},
		env_production: {
			NODE_ENV: 'production'
		}
	}]
}

# 延迟重启
# 设置自动重启的延迟重启策略
# ----> CLI
pm2 start app.js --restart-delay=3000
# ----> Configuration file
module.exports = {
	script: 'app.js',
	restart_delay: 3000
}

# 指数补偿的延迟重启
# 每次crash重启，会基于上次延迟时间增加延迟时长，延迟频繁的重启造成的服务压力
# ----> CLI
pm2 start app.js --exp-backoff-restart-delay=100
# ----> Configuration file
module.exports = {
	script: 'app.js',
	exp_backoff_restart_delay: 100
}

# 不自动重启
# ----> CLI
pm2 start app.js --no-autorestart
# ----> Configuration file
module.exports = {
	script: 'app.js',
	autorestart: false
}

# 指定错误码不自动重启
# ----> CLI
pm2 start app.js --stop-exit-codes 0
# ----> Configuration file
module.expors = [{
	script: 'app.js',
	stop_exit_codes: [0]
}]
````

### 3.2 进程列表

````shell
pm2 list
# or
pm2 [list|ls|l|status]

# 指定排序规则
pm2 list --sort name:desc
# or
pm2 list --sort [name|id|pid|memory|cpu|status|uptime][:asc|desc]
````

查看某一进程详情

````shell
pm2 describe 0
````

### 3.3 设置重启临界内存

* CLI 方法

````shell
pm2 start big-array.js --max-memory-restart 20M
````

* JSON方法

````shell
{
	"name": "max_mem",
	"script": "big-array.js",
	"max_memory_restart": "20M"
}
````

* 编程

````shell
pm2.start({
	name: "max_mem",
	script: "big-array.js",
	max_memory_restart: "20M"
}, function(err, proc) {
	// Processing
	
})
````

***单位： 可以是K, M, G***

### 3.4 需要同时管理多个应用，或者需要指定多个选项时，可以使用配置文件。如ecosystem.config.js:

````javascript
module.exports = {
  apps: [{
    name: 'limit worker',
    script: './worker.js',
    args: 'limit'
  }, {
    name: 'rotate worker',
    script: './worker.js',
    args: "rotate"
  }]
}
````

````shell
pm2 start ecosystem.config.js
````

## 4. 日志

### 4.1 实时日志

````shell
pm2 logs

#Options:
#  --json                json log output
#  --format              formated log output
#  --raw                 raw output
#  --err                 only shows error output
#  --out                 only shows standard output
#  --lines <n>           output the last N lines, instead of the last 15 by default
#  --timestamp [format]  add timestamps (default format YYYY-MM-DD-HH:mm:ss)
#  --nostream            print logs without lauching the log stream
#  --highlight [value]   highlights the given value
#  -h, --help            output usage information

# 清空日志信息
pm2 flush
# Or
pm2 flush <api> # 清空<api>对应的应用name或id日志

# 启动时的log选项
pm2 start app.js [OPTIONS]
#		-l --log [path]              specify filepath to output both out and error logs
#		-o --output <path>           specify out log file
#		-e --error <path>            specify error log file
#		--time                       prefix logs with standard formated timestamp
#		--log-date-format <format>   prefix logs with custom formated timestamp
#		--merge-logs                 when running mutiple process with same app name, do not split file by id
````

## 5. 启动脚本

````shell
# 使用pm2 startup命令产生启动脚本
$ pm2 startup
[PM2] Init System found: launchd
[PM2] You have to run this command as root. Execute the following command:
      sudo su -c "env PATH=$PATH:/home/unitech/.nvm/versions/node/v14.3/bin pm2 startup <distribution> -u <user> --hp <home-path>
      
# 将上面的命令复制/粘贴到终端, 重启应用
$ sudo su -c "env PATH=$PATH:/home/unitech/.nvm/versions/node/v14.3/bin pm2 startup <distribution> -u <user> --hp <home-path>

# 通过pm2 save命令，在reboot之后可以复活所有的应用
$ pm2 save

# 通过pm2 unstartup命令失效并删除重启配置
$ pm2 unstartup
````

## 6. 配置文件(Configuration File)

生成配置文件

````shell
# 使用 pm2 init 命令生成一个配置文件
$ pm2 init

# 这会产生一个ecosystem.config.js的文件
module.exports = {
  apps : [{
  	name: 'app1',
    script: 'index.js',
    watch: '.'
  }, {
  	name: 'app2',
    script: './service-worker/',
    watch: ['./service-worker']
  }]
};
````

配置文件的操作

````shell
# Start all applications
$ pm2 start ecosystem.config.js

# Stop all
$ pm2 stop ecosystem.config.js

# Restart all
$ pm2 restart ecosystem.config.js

# Reload all
$ pm2 reload ecosystem.config.js

# Delete all
$ pm2 delete ecosystem.config.js

# 指定进程，选项为 --only <app_name>
$ pm2 start ecosystem.config.js --only app1
````

通过`env_*`选项设置不同的环境变量

````json
module.exports = {
  apps: [{
    name: 'app1',
    script: './index.js',
    env_development: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production'
    }
  }]
}
````

通过`--env [env name]`切换环境

````shell
$ pm2 start ecosystem.config.js --env production
$ pm2 restart ecosystem.config.js --env development
````

## 7. 控制台

```bash
pm2 monit
```

在终端显示的实时控制台，监控CPU/内存使用率

## 8. 集群模式

PM2包含在每个产生的进程之间共享所有HTTP[s]/Websocekt/TCP/UDP连接的负载均衡。

```shell
$ pm2 start app.js -i max

# 依赖可用的CPUs
$ pm2 start app.js -i 0
$ pm2 start app.js -i -1

# 启动3个进程
$ pm2 start app.js -i 3
```

或者通过js/yaml/json文件

````javascript
module.exports = {
  apps: [{
    script: 'api.js',
    instances: 'max',
    exec_mode: 'cluster'
  }]
}
````

`-i`或`instances`选择的可选值：

* `0` 或者 `max`  所有的CPU
* `-1` CPU数 - 1
* `number`  number个CPU

## 9. 文件改变时重启应用

使用`--watch`选项可用很容易实现。`--ignore-watch`指定忽略改变的文件目录

````shell
pm2 start app.js --watch --ignore-watch="node_modules"
````

## 10. 部署

通过以下命令部署应用到一个或多个服务上：

````shell
$ pm2 deploy <configuration_file> <environment> <command>

# 如:
$ pm2 deploy production setup
````

支持的`commond`：

````shell
Commands:
  setup                run remote setup commands
  update               update deploy to the latest release
  revert [n]           revert to [n]th last deployment or 1
  curr[ent]            output current release commit
  prev[ious]           output previous release commit
  exec|run <cmd>       execute the given <cmd>
  list                 list previous deploy commits
  [ref]                deploy to [ref], the "ref" setting, or latest tag
````

通过`revert`选项回退直接的部署：

````shell
$ pm2 deploy production revert 1		# 回滚到-1的部署
$ pm2 deploy production revert 2 		# 回滚到-2的部署
````

通过`exec`选项向每个服务执行命令：

````shell
$ pm2 deploy production exec "pm2 reload all"
````

**当应用需要建立完连接后(如数据库连接等)再启动应用，需要在CLI命令提供`--wait-ready`或者在配置文件中提供`wait_ready: true`。 然后应用中，准备完毕的地方添加`process.send('ready')`**

```javascript
var http = require('http')

var app = http.createServer(function(req, res) {
  res.writeHead(200)
  res.end('hey')
})

var listener = app.listen(0, function() {
  console.log('Listening on port ' + listener.address().port)
  // Here we send the ready signal to PM2
  process.send('ready')
})
```

```shell
$ pm2 start app.js --wait-ready

# 配置超时时间
$ pm2 start app.js --wait-ready --listen-timeout 10000
```



# 应用：

官网: https://pm2.keymetrics.io/

