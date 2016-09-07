---
title: nodejs + neo4j + d3js 集群图的实现
date: 2016-09-07 14:23:36
tags: 
- neo4j
- nodejs
categories: 
- 前端
---

由于公司后面工作的需要,写了个Demo,配上文档。
<!-- more -->
### 1.准备工作

neo4j、nodejs、express的安装

#### 安装

这里主要讲nodejs的安装，neo4j可以直接去官网下载安装即可。

①OS X

安装home-brew

> $/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

安装wget

> $brew install wget

下载nvm安装程序，赋予775权限后安装

> $wget https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh
> $chmod 775 install.sh
> $./install.sh

配置环境变量，并保存退出，应用环境变量

>$vim ~/.profile
>
>export NVM_DIR="$HOME/.nvm"
>[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
>
>$source .profile

安装所需的node版本

> $nvm install v4.2.1

安装淘宝镜像

> $npm install -g cnpm --registry=https://registry.npm.taobao.org

②Ubuntu
与OS X下类似，不需要安装homebrew，直接使用apt-get

③windows

下载nodejs官方Windows版程序、安装，然后配置系统环境变量。

下载npm源码解压到D:\npmjs，在命令提示符窗口中执行下面操作，完成npm的安装。

> D:\\>cd npmjs
>
> D:\npmjs>node cli.js install -gf

npm安装好以后，将"D:\nodejs\node_modules"加入系统环境变量NODE_PATH中。

在安装好nodejs之后可以通过npm来安装最新版的express

>$npm install express -g

安装完express模块之后，需要再安装：express-generator //express命令工具

### 2.创建项目

#### 用express创建一个项目的框架

首先我们在工作空间创建一个名字叫neo4j_d3js的网站，用ejs模板引擎。

> $express -e neo4j_d3js	

-e表示ejs模板引擎，不写的话默认创建jade模板引擎。

然后进入创建好的项目目录并安装项目所依赖的包

> $cd neo4j_d3js
> $npm install

在这里需要注意的是，一般我会用cnpm来安装依赖，我用的是淘宝的镜像，安装依赖包会相比于直接npm来说会稍快。

然后在neo4j_d3js的目录下使用命令

> $node app

就可以启动项目了。

tips1: 如果项目不能启动，看看app.js文件中有没有端口监听。如果没有的话，在`moudle.exort = app:`语句之前添加`app.listen(3000);`；

tips2: Ctrl + c终止运行。

在项目启动后，我们可以在浏览器的地址栏里面输入http://127.0.0.1/或者http://locahost:3000，到这里我们就完成了项目的重要的第一步。

下面是对目录的一个大致说明

-node_modules 项目中依赖的包
-public 公共资源存放的目录
-routes 学名叫做路由，里面放着一些路由文件
-views 视图
-app.js 项目的入口文件。

### 3.创建视图文件

我们上面用express创建项目视图文件是ejs后缀名，但是我们一般习惯使用html后缀名，那么我们怎么让它识别html视图文件呢？

在app.js文件中，找到

> app.set('view engine', 'ejs');

把它替换成：

> app.set('view engine', 'html');

再用`app.engine()`方法注册模板引擎的后缀名。

> app.engine('html', require('ejs').__express);//两个下划线

然后我们创建一个index.html(本身有，改后缀名即可)

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"/>
    <title>集群图</title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <h1>集群图</h1>
  </body>
</html>
```

### 4.路由控制

修改routes文件夹中的index.js

```javascript
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res) {
	res.render('index', { title: 'index' });
});
  
module.exports = router;
```

routes文件夹中的users.js在本项目中没有作用，可自行删除。

### 5.连通neo4j和nodejs

[neo4j官方网站](http://neo4j.com/)

neo4j和nodejs之间的连接必然需要一个桥梁，前人已经帮我们铺好了路，我们只要去走就行了。这个桥梁也是这样。

#### 安装

官方提供了很多种接口，这里我选用的是`node-neo4j`，它是用来连通nodejs和neo4j的一个接口，如果要使用它，我们就需要安装这个依赖包。

在项目的根目下，使用命令

> $cnpm install node-neo4j --sava

这里有必要说一下`--save`，它在这里的作用就是把你的依赖包以及版本号写进`package.json`文件中，这样的话，如果你把你的代码分享给别人，就不用把依赖的包也一并分享，因为你的`package.json`文件中已经记录了你使用了哪些依赖包，他们只需要使用`cnpm install`命令即可在本地安装重新下载依赖包。

在`node-neo4j`包安装好之后，就可以通过引用包来实现nodejs和neo4j的连通。

#### 接口使用方法

实例化一个包装实例

```javascript
var neo4j = require('node-neo4j');
db = new neo4j('http://username:password@domain:port');
```

执行一条neo4j操作命令

```javascript
db.cypherQuery("START user = node(123) MATCH user-[:RELATED_TO]->friends RETURN friends", function(err, result){
    if(err) throw err;

    console.log(result.data); // delivers an array of query results
    console.log(result.columns); // delivers an array of names of objects getting returned
});
```

节点操作：

插入一个节点

```javascript
db.insertNode({
    name: 'Darth Vader',
    sex: 'male'
},function(err, node){
    if(err) throw err;

    // Output node properties.
    console.log(node.data);

    // Output node id.
    console.log(node._id);
});
```

读取一个节点

```javascript
db.readNode(12, function(err, node){
    if(err) throw err;

    // Output node properties.
    console.log(node.data);

    // Output node id.
    console.log(node._id);
});
```

更新一个节点

```javascript
db.updateNode(12, {name:'foobar2'}, function(err, node){
    if(err) throw err;

    if(node === true){
        // node updated
    } else {
        // node not found, hence not updated
    }
});
```

删除一个节点

```javascript
db.deleteNode(12, function(err, node){
    if(err) throw err;

    if(node === true){
        // node deleted
    } else {
        // node not deleted because not found or because of existing relationships
    }
});
```

关系操作：

插入一个关系

```javascript
db.insertRelationship(root_node_id, other_node_id, 'RELATIONSHIP_TYPE', {
    age: '5 years',
    sideeffects: {
        positive: 'happier',
        negative: 'less time'
    }}, function(err, relationship){
        if(err) throw err;

        // Output relationship properties.
        console.log(relationship.data);

        // Output relationship id.
        console.log(relationship._id);

        // Output relationship start_node_id.
        console.log(relationship._start);

        // Output relationship end_node_id.
        console.log(relationship._end);
});
```

读取一个关系

```javascript
db.readRelationship(relationship_id, function(err, relationship){
    if(err) throw err;

    // Same properties for relationship object as with InsertRelationship
});
```

更新一个关系

```javascript
db.updateRelationship(relationship_id, {
        age: '6 years'
    }, function(err, relationship){
        if(err) throw err;

        if(relationship === true){
            // relationship updated
        } else {
            relationship not found, hence not updated.
        }
});
```

删除一个关系

```javascript
db.deleteRelationship(relationship_id, function(err, relationship){
    if(err) throw err;

    if(relationship === true){
        // relationship deleted
    } else {
        // relationship not deleted because not found.
    }
});
```

更多：[node-neo4j官网](https://github.com/philippkueng/node-neo4j)，[neo4j开发文档](http://neo4j.com/docs/developer-manual/current/)，[cypher手册](http://neo4j.com/docs/cypher-refcard/3.0/)

### 6.制作一个简单的集群图

[d3js官网](https://d3js.org/)

d3js的用途很广泛，这里我仅以一个简单的集群图的制作做一个示例。

#### 在Neo4j中插入数据

首先向neo4j数据库中插入节点和关系，以下面的json为例。

```json
{
    "name": "flare",
    "children": [{
        "name": "analytics"
    }, {
        "name": "animate"
    }, {
        "name": "data"
    }, {
        "name": "display"
    }, {
        "name": "flex"
    }]
}
```

在上面的JSON中，共有6个节点，其中一个为父节点，其余为子节点，他们之间的关系为`children`。

创建节点：

> CREATE (ee:Admin { name: "flare"})//插入标签为Admin,name为flare的节点，其中ee为变量，可用任何字母或组合代替
>
> CREATE (ee:Children { name: "data"})/插入标签为Children,name为data的节点

这里仅以其中的两个节点的创建作为示例。

创建关系：

> CREATE (n:Admin)-[r:Children]->(m:Children)//把标签为Admin和Children的节点建立关系，由Admin指向Children

#### 使用nodejs获取数据

nodejs后台处理：

index.js

```javascript
/**
 * Created by uncle_ye.
 */
//引入依赖包
var express = require('express');
var router = express.Router();
var neo4j = require('node-neo4j');
fs = require('fs');

//连接数据库
db = new neo4j('http://username:password@domain:port');
//这里输入你自己的neo4j的账户名、密码以及端口号

function load() {

//查询语句
    var sql_root = "Match(n:Admin) return n";
    var sql_child = "Match(n:Children) return n";
    var json = {};

//根节点
    db.cypherQuery(sql_root, function (err, result) {
        if (err){
            throw err;
        }
        json.name = result.data.name;
    });

//子节点
    db.cypherQuery(sql_child, function (err, result) {
        if (err){
            throw err;
        }
        var children = [];
        for (var i = 0; i < result.data.length; i++) {
            children[i] = {name: result.data[i].name}
        }
        json.children = children;
        //把JSON写入文件
        var str = JSON.stringify(json);
        fs.writeFileSync('./public/json/output.json', str);
    });
}

router.get('/', function (req, res) {
    load();
    res.render('index', {title: 'Express'});
});

module.exports = router;
```

前端显示：

index.html

```html
<!DOCTYPE html>
<meta charset="utf-8">
<style>
    .node circle {
        fill: #fff;
        stroke: steelblue;
        stroke-width: 1.5px;
    }

    .node {
        font: 10px sans-serif;
    }

    .link {
        fill: none;
        stroke: #ccc;
        stroke-width: 1.5px;
    }
</style>
<body>
<h1>集群图</h1>
<!--//使用V3版本的d3js-->
<script src="//d3js.org/d3.v3.min.js"></script>
<script>
    //定义图的大小
    var width = 500,
        height = 550;
    //集群图树的大小
    var tree = d3.layout.tree()
            .size([height, width - 160]);
    //对角线生成器,返回值可以当做函数使用来生成三次贝塞尔曲线数据
    var diagonal = d3.svg.diagonal()
            .projection(function (d) {
                return [d.y, d.x];
            });
    //定义svg
    var svg = d3.select("body").append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", "translate(40,0)");
    //加载json
    d3.json("json/output.json", function (error, json) {
        if (error){
            throw error;
        }
        //获取所有节点和关系
        var nodes = tree.nodes(json),
            links = tree.links(nodes);
        //关系处理
        var link = svg.selectAll("path.link")
                .data(links)
                .enter().append("path")
                .attr("class", "link")
                .attr("d", diagonal);
        //节点处理
        var node = svg.selectAll("g.node")
                .data(nodes)
                .enter().append("g")
                .attr("class", "node")
                .attr("transform", function (d) {
                    return "translate(" + d.y + "," + d.x + ")";
                });
        //节点圆
        node.append("circle")
                .attr("r", 4.5);
        //节点名称
        node.append("text")
                .attr("dx", function (d) {
                    return d.children ? -8 : 8;
                })
                .attr("dy", 3)
                .attr("text-anchor", function (d) {
                    return d.children ? "end" : "start";
                })
                .text(function (d) {
                    return d.name;
                });
    });
  	//样式
    d3.select(self.frameElement).style("height", height + "px");
</script>
```

至此，一个简单的集群图就完成了。

在命令行输入npm start，在浏览器输入http://localhost:3000/即可查看。
