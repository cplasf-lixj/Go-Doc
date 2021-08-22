# MongoDB

## 1. 三个概念

* 数据库（database)

  * 数据库是一个仓库，在仓库中可以存放集合。

  ````shell
  # 显示当前所有的数据库
  show dbs
  show databases
  
  # 当前当前所处的数据库
  db
  
  # 切换数据库, 如果数据库不存在，则创建数据库，否则切换到指定数据库
  use 数据库名
  
  # 删除数据库
  db.dropDatabase()
  ````

* 集合（collection）

  * 集合类似于数组，在集合中可以存放文档。

  ```shell
  # 显示当前数据所有集合
  show collections
  
  # 创建集合
  db.createCollection(name, options)
  # name: 要创建的集合名称
  # options: 可选参数，指定内存大学及索引的选项
  #		capped:	boolean	是否创建固定集合。固定集合指固定大小的集合。达到最大值会自动覆盖最早文档
  #		autoIndexId: boolean 是否自动在_id字段创建索引。默认值false。
  #		size: number	capped为true时，必须指定该字段。
  #		max: number		指定固定集合中保护文档的最大数量
  
  # 删除集合
  db.collection.drop()
  ```

* 文档（document）

  * 文档数据库中的最小单温，我们存储和操作的内容都是文档。

  ````shell
  # 插入文档
  # 插入的数据主键已经存在，则会抛 org.springframework.dao.DuplicateKeyException 异常，提示主键重复，不保存当前数据。
  db.collection.insert(document)
  
  # inserOne 和 insertMany
  db.collection.insertOne(
  	<document>,
  	{
  		writeConcern: <document>
  	}
  )
  
  db.collection.insertMany(
  	[<document 1>, <document 2>, ...],
  	{
  		writeConcern: <document>,
  		ordered: <boolean>
  	}
  )
  # document：要写入的文档。
  # writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求。
  # ordered：指定是否按顺序写入，默认 true，按顺序写入。
  
  # 更新文档
  db.collection.update(
  	<query>,
  	<update>,
  	{
  		upsert: <boolean>,
  		multi: <boolean>,
  		writeConcern: <document>
  	}
  )
  # query : update的查询条件，类似sql update查询内where后面的。
  # update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
  # upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew。 true为插入，默认是false，不插入。
  # multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
  # writeConcern :可选，抛出异常的级别。
  
  # 删除文档
  # 2.6版本以前
  db.collecton.remove(
  	<query>,
  	<justOne>
  )
  # 2.6版本以后
  db.collection.remove(
  	<query>,
  	{
  		justOne: <boolean>,
  		writeConcern: <document>
  	}
  )
  # query :（可选）删除的文档的条件。
  # justOne : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
  # writeConcern :（可选）抛出异常的级别。
  
  # 3.2版本之后
  db.collection.deleteOne({ status: 'D' })		# 删除等于D第一个文档
  db.collection.deleteMany({ status: 'A' })		# 删除等于A的全部文档
  db.collection.deleteMany({})								# 删除全部文档
  
  # 查询文档
  db.collection.find(query, projection)
  # query: 可选，使用查询操作符指定查询条件
  # projection: 可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。
  
  # pretty() 方法以格式化的方式来显示所有文档。
  db.collection.find().pretty()
  ````

  

## 2. Mac安装

````shell
# 安装community版
# 1. 先tap MongoDB的formula
brew tap mongodb/brew
# 2. 安装MongoDB 社区版
brew install mongodb-community
````

## 3. 运行MongoDB

````shell
# 启动MongoDB
brew services start mongodb-community
# 停止MongoDB
brew services stop mongodb-community
# 重启MongoDB
brew services restart mongodb-community
````

## 4. 连接MongoDB

````shell
# 终端连接
mongo				
# 标准URI连接语法
mongodb://[username:password@]host1[:port1][,host2[:port2]][,...][,hostN[:portN]][/[database][?options]]

# mongodb://  					这是固定格式，必须要指定
# username:password@		可选项，若设置，连接数控服务器后会尝试登陆这个数据库
# host1 								必须至少指定一个host，为要连接的服务器的地址。
# postX 								可选项，为服务器端口号，默认为27017
# /database							如果指定username:password@,连接并验证登陆到指定数据库。默认为打开test
# ?options							连接项，如果不只用/database,则前面加上/。选项为键值对name=value,键值对通过&或;隔开
````

## 5. MongoDB的CRUD

### 5.1 Create

`db.collection.insertOne()`			 向集合中插入单个文档

`db.collection.insertMany()`			向集合中插入多个文档

`db.collection.insert()`					向文档中插入单个或多个文档

当选项中`upsert:true`时，以下方法也可以向集合中插入新文档

`db.collection.update()`					

`db.collection.updateOne()`

`db.collection.updateMany()`

`db.collection.findAndModify()`

`db.collection.findOneAndUpdate()`

`db.collection.findOneAndReplace()`

`db.collection.bulkWrite()`

### 5.2 Read



### 5.3 Update



### 5.4 Delete

