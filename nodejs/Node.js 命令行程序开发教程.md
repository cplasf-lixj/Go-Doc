

# 一、可执行脚本

​	首先，使用JavaScript语言，写一个可执行脚本hello。

````javascript
#!/usr/bin/env node
console.log("hello world");
````

然后，修改helloquanx

````shell
$ chmod 755 hello
````

现在，hello就可以执行了。

````shell
$ ./hello
hello world
````

如果想把hello前面的路径去除，可以将hello的路径加入环境变量PATH。但是，另一种更好的做法，是在当前目录下新建package.json，写入下面的内容。

````shell
{
	"name": "hello",
	"bin": {
		"hello": "hello"
	}
}
````

然后执行npm link命令。

````shell
$ npm link
````

现在再执行hello，就不用输入路径了。

# 二、命令行参数的原始写法

命令行参数可以用系统变量process.argv获取。

下面是一个脚本hello。

````javascript
#!/usr/bin/env node
console.log('hello ', process.argv[2]);
````

执行时，直接在脚本文件后面，加上参数即可。

````shell
$ ./hello tom
helle tom
````

上面代码中，实际执行的是node ./hello tom，对应的process.argv是['node', '/path/to/hello', 'tom']。

# 三、新建进程

脚本可以通过child_process模块新建子进程，从而执行Unix系统命令。

````javascript
#!/usr/bin/env node
var name = process.argv[2];
var exec = require('child_process').exec;

var child = exec('echo hello ' + name, function(err, stdout, stderr) {
  if (err) throw err;
  console.log(stdout);
})
````

用法如下。

````shell
$ ./hello tom
hello tom
````

# 四、shelljs模块

[shelljs](https://www.npmjs.com/package/shelljs)模块重新包装了child_process，调用系统命令更加方便。它需要安装后使用。

````shell
npm install --save shelljs
````

然后，改写脚本。

````javascript
#!/usr/bin/env node
var name = process.argv[2]
var shell = require('shelljs')

shell.exec("echo hello " + name)
````

上面代码是shelljs的本地模式，即通过exec方法执行shell命令。此外还有全局模式，允许直接在脚本中写shell命令。

````javascript
require('shelljs/global')

if (!which('git')) {
  echo('Sorry, this script requires git')
  exit(1)
}

mkdir('-p', 'out/Release')
cp('-R', 'stuff/*', 'out/Release')

cd('lib')
ls('*.js').forEach(function(file) {
  sed('-i', 'BUILD_VERSION', 'v0.1.2', file)
  sed('-i', /.*REMOVE_THIS_LINE.*\n/, '', file)
  sed('-i', /.*REPLACE_LINE_WITH_MACRO.*\n/, cat('macro.js'), file)
});

cd('..');

if (exec('git commit -am "Auto-commit"').code != 0) {
  echo("Error: Git commit failed");
  exit(1)
}
````

# 五、yargs模块

​	shelljs只解决如何调用shell命令，而yargs模块能够解决如何处理命令行参数。它也需要安装。

````shell
$ npm install --save yargs
````

yargs 模块提供argv对象，用来读取命令行参数。请看改写后的hello。

````javascript
#!/usr/bin/env node
var argv = require('yargs').argv;

console.log('hello ', argv.name);
````

使用时，下面两种用法都可以。

````shell
$ hello --name=tom
hello tom

$ hello --name tom
hello tom
````

也就是说，process.argv的原始返回值如下。

````shell
$ node hello --name=tom
[ 'node',
	'/path/to/myscript.js',
	'--name=tom' ]
````

yargs 可以将上面的结果改为一个对象，每个参数项就是一个键值对。

````javascript
var argv = require('yargs').argv;

// $ node hello --name=tom
// argv = {
// 	name: tom
//};
````

如果将argv.name改成argv.n，就可以使用一个字母的短参数形式。

````shell
$ hello -n tom
hello tom
````

可以使用alias方法，指定name是n的别名。

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.alias('n', 'name')
	.argv;

console.log('hello', argv.n);
````

这样一来，短参数和长参数就都可以使用了。

````shell
$ hello -n tom
hello tom

$ hello --name tom
hello tom
````

argv对象有一个下划线（_）属性，可以获取非连词线开头的参数。

````javascript
#!/usr/bin/env node
var argv = require('yargs').argv;

console.log('hello ', argv.n)
console.log(argv._);
````

用法如下。

````shell
$ hello A -n tom B C
hello tom
[ 'A', 'B', 'C' ]
````

# 六、命令行参数的配置

yargs 模块还提供3个方法，用来配置命令行参数。

​	• demand：是否必选

​	• default：默认值

​	• describe：提示

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.demand(['n'])
	.default({n: 'tom'})
	.describe({n: 'your name'})
	.argv;

console.log('hello', argv.n);
````

上面代码指定n参数不可省略，默认值tom，并给出一行提示。

options方法允许将所有这些配置写进一个对象。

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.options('n', {
    alias: 'name',
    demand: true,
    default: 'tom',
    describe: 'your name',
    type: 'string'
  })
	.argv;

console.log('hello', argv.n);
````

有时，某些参数不需要值，只起到一个开关作用，这时可以用boolean方法指定这些参数返回布尔值。

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.boolean(['n'])
	.argv;

console.log('hello', argv.n);
````

上面代码中，参数n总是返回一个布尔值，用法如下。

````shell
$ hello
hello false
$ hello -n
hello true
$ hello -n tom
hello true
````

boolean方法也可以作为属性，写入options对象

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.options('n', {
    boolean: true
  })
	.argv;

console.log('hello', argv.n);
````

# 七、帮助信息

yargs模块提供以下方法，生成帮助信息。

​	• usage：用法格式

​	• example：提供例子

​	• help：显示帮助信息

​	• epilog：出现在帮助信息的结尾

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.option('n', {
    alias: 'name',
    demand: true,
    default: 'tom',
    describe: 'your name',
    type: 'string'
  })
	.usage('Usage: hello [options]')
	.example('hello -n -tom', 'say hello to Tome')
	.help('h')
	.alias('h', 'help')
	.epilog('copyright 2021')
	.argv;

console.log('hello', argv.n)
````

执行结果如下。

````shell
$ hello -h

Usage: hello [options]

Options:
	-n, --name your name [string] [required] [default: "tom"]
	-h, --help Show help [boolean]
	
Examples:
	hello -n tom  say hello Tom

copyright 2021
````

# 子命令

​	yargs模块还允许通过command方法，设置Git风格的子命令。

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.command("morning", "good morning", function (yargs) {
    console.log("Good Morning")
  })
	.command("evening", "good evening", function (yargs) {
    console.log("Good Evening")
  })

console.log('hello ', argv.n)
````

用法如下。

````shell
$ hello morning -n tom
Good Morning
hello tom
````

可以将这个功能与shelljs模块结合起来。

````javascript
#!/usr/bin/env node
var argv = require('yargs')
	.command("morning", "good morning", function (yargs) {
    echo("Good Morning")
  })
	.command("evening", "good evening", function (yargs) {
    echo("Good Evening")
  })

console.log('hello ', argv.n)
````

每个子命令往往有自己的参数，这是就需要在回调函数中单独指定。回调函数中，要先用reset方法重置yargs对象。

````javascript
#!/usr/bin/env node
require('shelljs/global')
var argv = require('yargs')
	.command('morning', "good morning", function (yargs) {
    echo("Good Morning")
    var argv = yargs.reset()
    	.option('m', {
        alias: 'message',
        description: "provide any sentence"
      })
    	.help('h')
    	.alias('h', 'help')
    	.argv;
    
    echo(argv.m);
  })
	.argv;
````

用法如下。

````shell
$ hello morning -m "Are you hungry?"
Good Morning
Are you hungry?
````

# 九、其他事项

### （1）返回值

​	根据Unix传统，程序执行成功返回0，否则返回1。

````javascript
if (err) {
  process.exit(1)
} else {
  process.exit(0)
}
````

### (2) 重定向

​	Unix允许程序之间使用管道重定向数据。

````shell
$ ps aux | grep 'node'
````

​	脚本可以通过监听标准输入的data事件，获取重定向的数据。

````javascript
process.stdin.resume()
process.stdin.setEncoding('utf8')
process.stdin.on('data', function(data) {
  process.stdout.write(data)
})
````

​	下面是用法

````shell
$ echo 'foo' | ./hello
hello foo
````

### (3) 系统信号

​	操作系统可以向执行中的进程发送信号，process对象能够监听信号事件。

````javascript
process.on('SIGINT', function() {
  console.log('Got a SIGNIT')
  process.exit(0)
})
````

发送信号的方法如下。

````shell
$ kill -s SIGINT [process_id]
````