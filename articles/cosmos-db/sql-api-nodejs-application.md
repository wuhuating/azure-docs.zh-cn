---
title: "生成适用于 Azure Cosmos DB 的 Node.js Web 应用 | Microsoft Docs"
description: "此 Node.js 教程探讨了如何使用 Microsoft Azure Cosmos DB 从 Azure 网站上托管的 Node.js Express Web 应用程序来存储和访问数据。"
keywords: "应用程序开发, 数据库教程, 了解 node.js, node.js 教程"
services: cosmos-db
documentationcenter: nodejs
author: mimig1
manager: jhubbard
editor: cgronlun
ms.assetid: 9da9e63b-e76a-434e-96dd-195ce2699ef3
ms.service: cosmos-db
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 01/30/2018
ms.author: mimig
ms.openlocfilehash: 441f352555f40c0467df4c466d58ac35e32f9e61
ms.sourcegitcommit: 95500c068100d9c9415e8368bdffb1f1fd53714e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/14/2018
---
# <a name="_Toc395783175"></a>使用 Azure Cosmos DB 生成 Node.js Web 应用程序
> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Java](sql-api-java-application.md)
> * [Python](sql-api-python-application.md)
> 
> 

[!INCLUDE [cosmos-db-sql-api](../../includes/cosmos-db-sql-api.md)]

本 Node.js 教程介绍了如何使用 Azure Cosmos DB 和 SQL API，通过在 Azure 网站上托管的 Node.js Express 应用程序存储和访问数据。 用户构建一个简单的基于 Web 的任务管理应用程序（即待办事项应用），用于创建、检索和完成任务。 任务存储为 Azure Cosmos DB 中的 JSON 文档。 本教程演示如何创建和部署应用，并说明每个代码片段的功能。

![在本 Node.js 教程中创建的 My Todo List 应用程序的屏幕截图](./media/sql-api-nodejs-application/cosmos-db-node-js-mytodo.png)

没有时间完成本教程且只想获取完整的解决方案？ 没有问题，可以从 [GitHub][GitHub] 获得完整的示例解决方案。 只需读取[自述](https://github.com/Azure-Samples/documentdb-node-todo-app/blob/master/README.md)文件，了解如何运行该应用。

## <a name="_Toc395783176"></a>先决条件
> [!TIP]
> 本 Node.js 教程假定你之前有使用 Node.js 和 Azure 网站的经验。
> 
> 

在按照本文中的说明操作之前，应确保已拥有下列项：

* 如果你还没有 Azure 订阅，可以在开始前创建一个 [免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js][Node.js] v0.10.29 或更高版本。 我们建议使用 Node.js 6.10 或更高版本。
* [Express 生成器](http://www.expressjs.com/starter/generator.html)（可以通过 `npm install express-generator -g` 安装）
* [Git][Git]。

## <a name="_Toc395637761"></a>步骤 1：创建 Azure Cosmos DB 数据库帐户
让我们首先创建一个 Azure Cosmos DB 帐户。 如果已有一个帐户，或者要在本教程中使用 Azure Cosmos DB 模拟器，可以跳到[步骤 2：创建新的 Node.js 应用程序](#_Toc395783178)。

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [cosmos-db-keys](../../includes/cosmos-db-keys.md)]

## <a name="_Toc395783178"></a>步骤 2：创建新的 Node.js 应用程序
现在让我们来了解如何使用 [Express](http://expressjs.com/) 框架创建基本的 Hello World Node.js 项目。

1. 打开最喜欢的终端，如 Node.js 命令提示符。
2. 导航到要在其中存储新应用程序的目录。
3. 使用 Express 生成器生成名叫 **todo**的新应用程序。
   
        express todo
4. 打开新的 **todo** 目录并安装依赖项。
   
        cd todo
        npm install
5. 运行新应用程序。
   
        npm start
6. 在浏览器中导航到 [http://localhost:3000](http://localhost:3000) 即可查看新应用程序。
   
    ![了解 Node.js - 浏览器窗口中 Hello World 应用程序的屏幕截图](./media/sql-api-nodejs-application/cosmos-db-node-js-express.png)

    然后，要停止应用程序，可在终端窗口中按 CTRL+C 并仅在 Windows 计算机上单击“y”终止批处理作业。

## <a name="_Toc395783179"></a>步骤 3：安装其他模块
**package.json** 文件是在项目的根目录中创建的文件之一。 此文件包含一系列其他模块，它们是 Node.js 应用程序必需的。 稍后，在将此应用程序部署到 Azure 网站时，使用此文件以确定需要在 Azure 上安装哪些模块来支持应用程序。 我们仍需要为本教程多安装两个包。

1. 返回终端，通过 npm 安装 **async** 模块。
   
        npm install async --save
2. 通过 npm 安装 **documentdb** 模块。 这是所有 Azure Cosmos DB magic 发生的模块。
   
        npm install documentdb --save

## <a name="_Toc395783180"></a>步骤 4：在 node 应用程序中使用 Azure Cosmos DB 服务
这负责处理所有的初始安装和配置，现在让我们看看为什么要用这步，可以编写一些使用 Azure Cosmos DB 的代码。

### <a name="create-the-model"></a>创建模型
1. 在项目目录中，在 package.json 文件所在的目录创建名为“models”的新目录。
2. 在 **models** 目录中，创建一个名为 **task-model.js** 的新文件。 此文件将包含我们应用程序所创建任务的模型。
3. 在同一个 **models** 目录中，创建名为 **cosmosdb-manager.js** 的另一个新文件。 此文件将包含一些可重用的有用代码，我们会在整个应用程序期间用到这些代码。 
4. 将以下代码复制到 **cosmosdb-manager.js**
    ```nodejs
    let DocumentDBClient = require('documentdb').DocumentClient;

    module.exports = {
    getOrCreateDatabase: (client, databaseId, callback) => {
        let querySpec = {
        query: 'SELECT * FROM root r WHERE r.id = @id',
        parameters: [{ name: '@id', value: databaseId }]
        };

        client.queryDatabases(querySpec).toArray((err, results) => {
        if (err) {
            callback(err);
        } else {
            if (results.length === 0) {
            let databaseSpec = { id: databaseId };
            client.createDatabase(databaseSpec, (err, created) => {
                callback(null, created);
            });
            } else {
            callback(null, results[0]);
            }
        }
        });
    },

    getOrCreateCollection: (client, databaseLink, collectionId, callback) => {
        let querySpec = {
        query: 'SELECT * FROM root r WHERE r.id=@id',
        parameters: [{ name: '@id', value: collectionId }]
        };

        client.queryCollections(databaseLink, querySpec).toArray((err, results) => {
        if (err) {
            callback(err);
        } else {
            if (results.length === 0) {
            let collectionSpec = { id: collectionId };
            client.createCollection(databaseLink, collectionSpec, (err, created) => {
                callback(null, created);
            });
            } else {
            callback(null, results[0]);
            }
        }
        });
    }
    };
    ```
5. 保存并关闭 **cosmosdb-manager.js** 文件。
6. 在 **task-model.js** 文件的开头添加以下代码，引用前面创建的 **DocumentDBClient** 和 **cosmosdb-manager.js**： 

    ```nodejs
    let DocumentDBClient = require('documentdb').DocumentClient;
    let docdbUtils = require('./docdbUtils');
    ```
7. 接下来，将添加代码以定义和导出 Task 对象。 这负责初始化我们的 Task 对象，并设置我们将使用的数据库和文档集合。  

    ```nodejs
    function TaskModel(documentDBClient, databaseId, collectionId) {
      this.client = documentDBClient;
      this.databaseId = databaseId;
      this.collectionId = collectionId;
   
      this.database = null;
      this.collection = null;
    }
   
    module.exports = TaskModel;
    ```
8. 然后添加以下代码以定义 Task 对象上的其他方法，该对象可与存储在 Azure Cosmos DB 中的数据进行交互。

    ```nodejs
    let DocumentDBClient = require('documentdb').DocumentClient;
    let docdbUtils = require('./cosmosdb-manager');

    function TaskModel(documentDBClient, databaseId, collectionId) {
    this.client = documentDBClient;
    this.databaseId = databaseId;
    this.collectionId = collectionId;

    this.database = null;
    this.collection = null;
    }

    TaskModel.prototype = {
    init: function(callback) {
        let self = this;

        docdbUtils.getOrCreateDatabase(self.client, self.databaseId, function(err, db) {
        if (err) {
            callback(err);
        } else {
            self.database = db;
            docdbUtils.getOrCreateCollection(self.client, self.database._self, self.collectionId, function(err, coll) {
            if (err) {
                callback(err);
            } else {
                self.collection = coll;
            }
            });
        }
        });
    },

    find: function(querySpec, callback) {
        let self = this;

        self.client.queryDocuments(self.collection._self, querySpec).toArray(function(err, results) {
        if (err) {
            callback(err);
        } else {
            callback(null, results);
        }
        });
    },

    addItem: function(item, callback) {
        let self = this;

        item.date = Date.now();
        item.completed = false;

        self.client.createDocument(self.collection._self, item, function(err, doc) {
        if (err) {
            callback(err);
        } else {
            callback(null, doc);
        }
        });
    },

    updateItem: function(itemId, callback) {
        let self = this;

        self.getItem(itemId, function(err, doc) {
        if (err) {
            callback(err);
        } else {
            doc.completed = true;

            self.client.replaceDocument(doc._self, doc, function(err, replaced) {
            if (err) {
                callback(err);
            } else {
                callback(null, replaced);
            }
            });
        }
        });
    },

    getItem: function(itemId, callback) {
        let self = this;
        let querySpec = {
        query: 'SELECT * FROM root r WHERE r.id = @id',
        parameters: [{ name: '@id', value: itemId }]
        };

        self.client.queryDocuments(self.collection._self, querySpec).toArray(function(err, results) {
        if (err) {
            callback(err);
        } else {
            callback(null, results[0]);
        }
        });
    }
    };

    module.exports = TaskModel;
    ```
9. 保存并关闭 **task-model.js** 文件。 

### <a name="create-the-controller"></a>创建控制器
1. 在项目的 **routes** 目录中，创建一个名为 **tasklist.js** 的新文件。 
2. 将以下代码添加到 **tasklist.js**。 这会加载 **tasklist.js**使用的 DocumentDBClient 和 async 模块。 这还定义了 **TaskList** 函数，将向该函数传递我们之前定义的 **Task** 对象的一个实例：
   
    ```nodejs
    let DocumentDBClient = require('documentdb').DocumentClient;
    let async = require('async');

    function TaskList(taskModel) {
    this.taskModel = taskModel;
    }

    module.exports = TaskList;
    ```
3. 继续向 **tasklist.js** 文件添加用于 **showTasks、addTask** 和 **completeTasks** 的方法：
   
   ```nodejs
    TaskList.prototype = {
    showTasks: function(req, res) {
        let self = this;

        let querySpec = {
        query: 'SELECT * FROM root r WHERE r.completed=@completed',
        parameters: [
            {
            name: '@completed',
            value: false
            }
        ]
        };

        self.taskModel.find(querySpec, function(err, items) {
        if (err) {
            throw err;
        }

        res.render('index', {
            title: 'My ToDo List ',
            tasks: items
        });
        });
    },

    addTask: function(req, res) {
        let self = this;
        let item = req.body;

        self.taskModel.addItem(item, function(err) {
        if (err) {
            throw err;
        }

        res.redirect('/');
        });
    },

    completeTask: function(req, res) {
        let self = this;
        let completedTasks = Object.keys(req.body);

        async.forEach(
        completedTasks,
        function taskIterator(completedTask, callback) {
            self.taskModel.updateItem(completedTask, function(err) {
            if (err) {
                callback(err);
            } else {
                callback(null);
            }
            });
        },
        function goHome(err) {
            if (err) {
            throw err;
            } else {
            res.redirect('/');
            }
        }
        );
    }
    };
    ```        
4. 保存并关闭 **tasklist.js** 文件。

### <a name="add-configjs"></a>添加 config.js
1. 在项目目录中创建一个名为 **config.js**的新文件。
2. 将以下内容添加到 **config.js**。 这会定义我们的应用程序所需的配置设置和值。
   
    ```nodejs
    let config = {}
   
    config.host = process.env.HOST || "[the URI value from the Azure Cosmos DB Keys page on http://portal.azure.com]";
    config.authKey = process.env.AUTH_KEY || "[the PRIMARY KEY value from the Azure Cosmos DB Keys page on http://portal.azure.com]";
    config.databaseId = "ToDoList";
    config.collectionId = "Items";
   
    module.exports = config;
    ```
3. 在 **config.js** 文件中，使用 [Microsoft Azure 门户](https://portal.azure.com)中 Azure Cosmos DB 帐户的“密钥”页中的值更新 HOST 和 AUTH_KEY 的值。
4. 保存并关闭 **config.js** 文件。

### <a name="modify-appjs"></a>修改 app.js
1. 在项目目录中，打开 **app.js** 文件。 此文件早于 Express Web 应用程序创建。
2. 将以下代码添加到 **app.js** 的顶部：
   
    ```nodejs
    var DocumentDBClient = require('documentdb').DocumentClient;
    var config = require('./config');
    var TaskList = require('./routes/tasklist');
    var TaskModel = require('./models/taskModel');
    ```
3. 此代码定义要使用的配置文件，并继续将该文件中的值读取到我们不久要使用的某些变量中。
4. 替换 **app.js** 文件中的以下两行：
   
    ```nodejs
    app.use('/', index);
    app.use('/users', users); 
    ```
   
    使用下面的代码段：
   
    ```nodejs
    let docDbClient = new DocumentDBClient(config.host, {
        masterKey: config.authKey
    });
    let taskModel = new TaskModel(docDbClient, config.databaseId, config.collectionId);
    let taskList = new TaskList(taskModel);
    taskModel.init();
   
    app.get('/', taskList.showTasks.bind(taskList));
    app.post('/addtask', taskList.addTask.bind(taskList));
    app.post('/completetask', taskList.completeTask.bind(taskList));
    app.set('view engine', 'jade');
    ```
5. 这些行（使用从 **config.js** 中读取的值）定义 **TaskModel** 对象的新实例以及与 Azure Cosmos DB 的新连接，初始化该任务对象，并将窗体操作绑定到 **TaskList** 控制器上的方法。 
6. 最后，保存并关闭 **app.js** 文件，我们就快完成了。

## <a name="_Toc395783181"></a>步骤 5：生成用户界面
现在让我们把注意力转向构建用户界面，因此用户可以与我们的应用程序进行切实的交互。 我们创建的 Express 应用程序使用 **Jade** 作为视图引擎。 有关 Jade 的详细信息，请参阅 [http://jade-lang.com/](http://jade-lang.com/)。

1. **views** 目录中的 **layout.jade** 文件用作其他 **.jade** 文件的全局模板。 在此步骤中，将对其进行修改以使用 [Twitter Bootstrap](https://github.com/twbs/bootstrap)（一个可以轻松设计美观网站的工具包）。 
2. 打开 **views** 文件夹中的 **layout.jade** 文件，将内容替换为以下代码：

    ```
    doctype html
    html
      head
        title= title
        link(rel='stylesheet', href='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/css/bootstrap.min.css')
        link(rel='stylesheet', href='/stylesheets/style.css')
      body
        nav.navbar.navbar-inverse.navbar-fixed-top
          div.navbar-header
            a.navbar-brand(href='#') My Tasks
        block content
        script(src='//ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.2.min.js')
        script(src='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/bootstrap.min.js')
    ```

    这会有效告知 **Jade** 引擎来为应用程序呈现某些 HTML，并创建名为 **content** 的**块**，我们可在其中提供内容页面的布局。

    保存并关闭此 **layout.jade** 文件。

3. 现在打开 **index.jade** 文件（应用程序将要使用的视图），并将文件内容替换为以下代码：
   
        extends layout
        block content
           h1 #{title}
           br
        
           form(action="/completetask", method="post")
             table.table.table-striped.table-bordered
               tr
                 td Name
                 td Category
                 td Date
                 td Complete
               if (typeof tasks === "undefined")
                 tr
                   td
               else
                 each task in tasks
                   tr
                     td #{task.name}
                     td #{task.category}
                     - var date  = new Date(task.date);
                     - var day   = date.getDate();
                     - var month = date.getMonth() + 1;
                     - var year  = date.getFullYear();
                     td #{month + "/" + day + "/" + year}
                     td
                       input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
             button.btn.btn-primary(type="submit") Update tasks
           hr
           form.well(action="/addtask", method="post")
             .form-group
               label(for="name") Item Name:
               input.form-control(name="name", type="textbox")
             .form-group
               label(for="category") Item Category:
               input.form-control(name="category", type="textbox")
             br
             button.btn(type="submit") Add item
   

这会扩展布局，并为我们先前在 **layout.jade** 文件中看到的 **content** 占位符提供内容。
   
在此布局中，我们创建了两个 HTML 窗体。

第一个窗体中的表包含我们的数据和按钮，该按钮允许我们通过发布控制器的 **/completetask** 方法更新项。
    
第二个窗体包含两个输入字段和一个按钮，该按钮允许我们通过发布控制器的 **/addtask** 方法来新建项。

这应该是应用程序工作所需的所有内容了。

## <a name="_Toc395783181"></a>步骤 6：在本地运行应用程序
1. 要在本地计算机上测试应用程序，请在终端中运行 `npm start` 以启动应用程序，并刷新 [http://localhost:3000](http://localhost:3000) 浏览器页。 该页此时看起来应如下图所示：
   
    ![浏览器窗口中 MyTodo List 应用程序的屏幕截图](./media/sql-api-nodejs-application/cosmos-db-node-js-localhost.png)

    > [!TIP]
    > 如果收到有关 layout.jade 文件或 index.jade 文件的缩进错误，请确保这两个文件中的头两行都已经左对齐，没有空格。 如果头两行之前留有空格，请删除这些空格，将这两个文件保存，然后刷新浏览器窗口。 

2. 使用“项”、“项名”和“类别”字段输入新任务，并单击“添加项”。 这会在 Azure Cosmos DB 中创建具有这些属性的文档。 
3. 页面应更新为在 ToDo 列表中显示新建项。
   
    ![ToDo 列表中具有新的项的应用程序屏幕截图](./media/sql-api-nodejs-application/cosmos-db-node-js-added-task.png)
4. 要完成任务，只需选中“完成”列中的复选框，并单击“更新任务” 。 这将更新已创建的文档并将其从视图中删除。

5. 若要停止应用程序，可在终端窗口中按 CTRL+C 并单击“Y”终止批处理作业。

## <a name="_Toc395783182"></a>步骤 7：将应用程序开发项目部署到 Azure 网站
1. 如果尚未部署，则启用 Azure 网站的 git 存储库。 可以在 [Local Git Deployment to Azure 应用服务](../app-service/app-service-deploy-local-git.md) （从本地 GIT 部署到 Azure 应用服务）主题中找到如何执行此操作的说明。
2. 将 Azure 网站添加为 git 远程。
   
        git remote add azure https://username@your-azure-website.scm.azurewebsites.net:443/your-azure-website.git
3. 通过推送到远程进行部署。
   
        git push azure master
4. 在几秒钟内，git 将完成 Web 应用程序发布并启动浏览器，从中可查看在 Azure 中运行的简单作品！

    祝贺你！ 刚才构建了第一个使用 Azure Cosmos DB 的 Node.js Express Web 应用程序并将其发布到了 Azure 网站。

    若需下载或参考本教程的完整参考应用程序，可从 [GitHub][GitHub] 下载。

## <a name="_Toc395637775"></a>后续步骤

* 希望使用 Azure Cosmos DB 执行规模和性能测试？ 请参阅[使用 Azure Cosmos DB 执行规模和性能测试](performance-testing.md)
* 了解如何[监视 Azure Cosmos DB 帐户](monitor-accounts.md)。
* 在 [Query Playground](https://www.documentdb.com/sql/demo)中对示例数据集运行查询。
* 浏览 [Azure Cosmos DB 文档](https://docs.microsoft.com/azure/cosmos-db/)。

[Node.js]: http://nodejs.org/
[Git]: http://git-scm.com/
[GitHub]: https://github.com/Azure-Samples/documentdb-node-todo-app

