---
title: Use sequelize ORM in Nodejs
date: 2019-06-23 10:48:40
author: gongzq5
tags: 
- sequelize
- nodejs
- ORM
- 后端
category:
- 后端
---



记录一下如何使用Sequelize框架，包括如何放在项目里进行组织，如何使用框架进行CRUD的操作



<!-- more -->



## 开始之前

### 先讲讲我的开发配置

* **语言**：Nodejs
* **后端框架**：KOA2
* **数据库：** MySQL 5.6



### 选择一个ORM框架

如何选择是个大问题，百度了一些，大家推荐的基本是这四种

* **TypeORM**：读了读代码，是用**typescript**写的，风格上和**Java**非常像，用注解什么的，感觉还不错，备选

* **ORM2**：看了几篇博客，好像有点小坑，还是算了

* **Knex**：推荐使用Bookshelf.js，支持Oracle，放弃了，毕竟已经确定了是用 KOA2 （类似express的风格了）

* **Sequelize：**这个应该是应用最广泛的了，看了看文档也很齐全，中文文档就有不下五六个版本，应该资源是不缺的，而且相比起其他的更像是javascript，就是他了！

基本大家都实现了`Promise`，所以基本都可以使用`async / await`



## Sequelize 

### 又是开始之前

* [一篇翻译的不错的中文文档：https://github.com/demopark/sequelize-docs-Zh-CN](<https://github.com/demopark/sequelize-docs-Zh-CN>)
* [官方给的详细的API参考：http://docs.sequelizejs.com/identifiers](<http://docs.sequelizejs.com/identifiers>)

> 一定要善用后边这个文档，我是后期才使用这个文档，感觉前边吃了不少亏



### Get Started

#### 安装

```bash
npm install --save sequelize
npm install --save mysql2 # 可能要自己选择具体的数据库驱动，我们是MYSQL，就是这个了
```

#### 建立数据库连接

```javascript
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
    host: 'your ip address',
    dialect: 'mysql', // Or mariadb, postgres, mssql ... 
    operatorsAliases: false,
    dialectOptions: {
        // 字符集
        charset: "utf8mb4",
        collate: "utf8mb4_unicode_ci",
        supportBigNumbers: true,
        bigNumberStrings: true
    },
    pool: {
        max: 5,
        min: 0,
        acquire: 30000,
        idle: 10000
    },
    timezone: '+08:00' //东八时区
});

module.exports = {
    sequelize
};
```

可以使用 `.authenticate()` 函数来测试连接.

```javascript
sequelize
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```



#### 建表和使用

说时迟那时快，马上就到了最重要的一步，建表。话不多说直接上代码。

```javascript
// from task_table.js
module.exports = function(sequelize, DataTypes){
    return sequelize.define('task',{
        task_id:{
            type:DataTypes.INTEGER,
            primaryKey:true,
            allowNull:false,
            autoIncrement:true
        },
        ......
     );
};
```

这就完成了一个表的模型的定义，到使用时，我们要在其他文件中 `require`这个文件，如下：

```javascript
const TaskTable = sequelize.import('task_table');

// 用查询举例

async function searchTaskByID(task_id) {
    return await TaskTable.findByPk(task_id); // find by primary key
}
```

使用时调用这个函数就可以返回根据`id`查询到的`task`对象了。

这样，最简单的使用就完成了，具体的建表和表查询的API咱们等下继续讨论。



### 建表

之前我们已经给出了建表的一段简单的代码，现在具体的看看还可以提供什么

#### `freezeTableName`

```
return sequelize.define('user',{
    id: { type:DataTypes.INTEGER},
}, {
    // 如果为 true 则表的名称和 model 相同，即 user
    // 为 false MySQL创建的表名称会是复数 users
    // 如果指定的表名称本就是复数形式则不变
    freezeTableName: true
});
```

如果为 true 则表的名称和 model 相同，即 user，为 false MySQL创建的表名称会是复数 users，如果指定的表名称本就是复数形式则不变

#### `type`

**详情请参考[API文档里关于DataTypes类的说明](<http://docs.sequelizejs.com/variable/index.html#static-variable-DataTypes>)**

包含了一些主要使用的数据类型，我们常用的有`Integer, Float, Char(), Text, Date ...`，例子可以看上边的定义

#### 访问器和设置器

每个属性都可以设置 `get(), set()`函数，可以方便的获取一些格式化的值，比如对于时间的处理，由于不同的时间有不同的表示格式，我们可以在访问器这个级别来使用库来转换这个格式

```javascript
updatedAt: {
    type: DataTypes.DATE,
    get() {
        return moment(this.getDataValue('updatedAt')).format('YYYY-MM-DD HH:mm:ss');
    }
}
```

这样，我们拿出来的值就不是标准时间 `****T****Z`这种奇怪的格式，而是形如 `YYYY-MM-DD HH:mm:ss` 的时间了

#### 外键约束

使用 `reference` 可以指定外键，如下：

```javascript
team_id:{
    type:DataTypes.INTEGER,
    allowNull:false,
    references: {
        model: 'team',
        key: 'team_id',
        deferrable: Sequelize.Deferrable.INITIALLY_IMMEDIATE
    }
},
```

`deferrable`是指定依赖的关系，比如添加时外部表没有该键如何处理，删除时是否要级联删除等……请查看[文档的详细介绍](<http://docs.sequelizejs.com/variable/index.html#static-variable-Deferrable>)

#### 列属性定义的一些其他参数

| 参数名       | 可选取值 | 解释   | 备注 |
| ------------ | -------- | ------ | ---- |
| `primaryKey` | ` true | false`  | 主键 |      |
| `allowNull` |  ` true | false`        | 可以为空 |      |
| `autoIncrement` | `true | false` | 自增属性 |      |



### 查询

#### 从where开始

Sequelize 提供了不少查询的API，可以直接使用，比如

```javascript
let task = await Task.findAll({
    where: {
        task_id: 1
    }
    attributes: ['task_id']
}));
```

相当于`SELECT task_id FROM task WHERE task_id=1`，返回值是一个数组，里边包含了0个或多个 Sequelize 的 model 对象（也可以理解为查询结果转换为的Json对象）

基本上使用就是通过 `where` 语句进行限定，如果想要使用`And | Or` ，可以使用官方提供的`Op`类，如下：

```javascript
await Task.findAll({
    where: {
        task_id: {
        	[Op.or]: [1,2,3,4]
        }
    }
    attributes: ['task_id']
}));
```

我们回到这个返回值来看，这个对象还是和普通的Json不太一样，sequelize 为我们包装了不少东西，比如可以提供`get()`的访问。

#### 关联查询

感觉这是一个比较麻烦的地方，关键是资料还贼少，大概谢谢自己总结的一些地方

[先上官方文档](<http://docs.sequelizejs.com/class/lib/associations/base.js~Association.html>)

##### 建立关联

关联要使用到几个关系，分别是[BelongsTo](http://docs.sequelizejs.com/class/lib/associations/belongs-to.js~BelongsTo.html), [BelongsToMany](http://docs.sequelizejs.com/class/lib/associations/belongs-to-many.js~BelongsToMany.html), [HasMany](http://docs.sequelizejs.com/class/lib/associations/has-many.js~HasMany.html), [HasOne](http://docs.sequelizejs.com/class/lib/associations/has-one.js~HasOne.html)

其中 `BelongsTo`和`HasOne`对应于**1：1的关联**

而`HasMany`对应于**1：m的关联**

而`BelongsToMany`对应于**n:m的关联**

###### 1:1

我们可以预先定义这个关联

```
UserInfo.belongsTo(User, {foreignKey: 'username'})
// 或 User.hasOne(UserInfo, {foreignKey: 'username'})
```

这两种写法都会给 `UserInfo` 加入一列，作为外键，指向 `User` 的`username` 上，当然如果需要这个添加反映到数据库上，需要使用 `Sequelize.sync()`才可以

完成以上这一步之后，当你获得从 `findAll` 的API得到的 `UserInfo` 对象时，可以使用 `getUsers()` 这样的函数，来获取和他关联的 `user` 对象。

###### 1:m

和上边差不多，只是换成了 `hasMany` 而已

###### m:n

这个是面向多对多的关联的，比如用户加入小组，需要在用户和小组之间建立一个联系，大概像这样：

```
| id | username | team_id | ... |
| 0  | user1    | 1       | ... |
...
```

我们可以使用 `BelongsToMany` 这个关联，需要提供一个` through `的表，比如：

```javascript
Team.belongsToMany(User, {through: Members, foreignKey: 'team_id', target_key: 'username'})
```

需要预先定义一个 `Members` 的 `Model`，可以不写东西，会自动添加两边的主键进去，也可以自己先定义一些其他的需要的列，比如状态什么的。



##### 使用关联进行查询

查询时需要使用的是`include`方法，如下：

```
Task.belongsTo(User)
return await Task.findAll({
    where: { ... },
    include: [{
        model: User
    }]
})
```

相当于先在关联的外键上做一个` Join` ，然后进行查询。可以看到 `include` 提供的是一个数组，所以是可以提供多个表级联的查询的。

可能每次在前边写 `belongsTo` 感觉比较奇怪，一种可选的写法是：

```
return await Task.findAll({
    where: { ... },
    include: [{
        association: Task.belongsTo(User)
    }]
})
```

也是差不多的



### Update、Delete、Create

这三个就直接一起讲了，没太大区别

```
models.TR.create({
    username: username,
    task_id: task_id,
    state: models.status_code.tr.WAITING_TO_BE_DONE
})

models.Task.update({
    state: models.status_code.task.ACCEPETED_AND_DOING
}, {
    where: {
        task_id: task_id
    }
})
            
TR.destroy({
	where: {
		task_id: task_id
	}
})
```

具体参数写了什么不需要管，反正大概就是这个意思啦，要注意删除用的是 `destroy` 



### sql 聚合函数

聚合函数是指 `count` 啊什么的，比如：

`[[sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']]`，调用了`count`函数，对`hats` 列进行计数，并将结果存为`no_hats`，我们可以在`include`或`findAll`的参数里直接使用这个。



### 使用原始SQL语句

到现在肯定有同学发现了，还是不灵活，有不少操作很难完成，幸好我们可以直接使用 sql 语句进行查询，如下：

```javascript
await sequelize.query(
    "SELECT * FROM `task` where `task_id` NOT IN (SELECT `task_id` FROM `teamtask`) AND `publisher` = \'" + org_name + "\'", 
    { type: sequelize.QueryTypes.SELECT}
    ).then(result => {
        return result
    });
```

这里当然是可以进行参数绑定的，我就直接裸加了，如果需要参数绑定请自行百度一下吧哈哈（当然这个语句还是可以用两个`awiat`完成的，我没什么例子了随便写写啦）



### 一些小tips

一是使用redis进行缓存，这个可以单独去搞，也就不在这里赘述了。

除此以外，我们可以尝试着让一些查询尽可能的并行化

#### Promise.all

这里要提到一个工具：`Promise.all([])`，举例如下：

我们接受任务后需要对任务的状态进行修改，即这样的需求：

```javascript
await models.TR.create(...);
await models.Task.update(...);
```

这就是2个`await`语句，我们可以把这个包装一下：

```javascript
let result = await Promise.all([
    models.TR.create(...),
    models.Task.update(...)
]) 
```

相当于把两个串行改成了并行的，还是可以省不少时间的，特别是访问多了以后

要注意每个异步语句都是会立即返回一个`promise`对象的，因此我们可以用循环运行所有的这些操作，并将返回值存储起来，然后放到`Promise.all()`的参数中，等待其完成

#### Transaction

把所有的时间打包作为一个事务提交，理论上应该免去了网络的时延啊，虽然我没有用，但是应该是一个很好的选择。



### 目录结构

我们项目使用的目录结构大致如下：

```bash
root
├─......
├─controllers
├─models
└─tables
```

* controllers负责处理router模块处理的URL后分发的请求，对参数进行分析，处理主要的业务逻辑部分；
* models负责与数据库进行交互，使用我们上面提到的这一堆API进行实现；
* tables负责建表的语句；

以上三个文件夹有明显的分层，存在由上到下的调用的依赖，同级之间尽量不发生依赖关系，也不会出现依赖上级代码的情况。如果可以的话，我也推荐在每个文件夹写一个总的导出的文件，到上一层时，使用这个文件进行导入，而不是导入每个下层文件夹里所有的文件，这样也可以保证所有的建表语句会在开始时运行，而不会出现最后发现有个表没有用到的情况。