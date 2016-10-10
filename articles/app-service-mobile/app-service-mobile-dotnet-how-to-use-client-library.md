<properties
	pageTitle="使用应用服务移动应用托管客户端库 (Windows | Xamarin) | Azure"
	description="了解如何在 Windows 和 Xamarin 应用中使用 Azure 应用服务移动应用的 .NET 客户端。"
	services="app-service\mobile"
	documentationCenter=""
	authors="ggailey777"
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.date="06/11/2016"
	wacn.date=""/>

# 如何使用 Azure 移动应用的托管客户端

[AZURE.INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

##概述

本指南说明如何在 Windows 应用和 Xamarin 应用中使用 Azure 应用服务移动应用的托管客户端库执行常见方案。如果是移动服务的新手，最好先完成 [Azure Mobile Apps quickstart]（Azure 移动应用快速入门）教程。在本指南中，我们侧重于客户端托管的 SDK。有关移动应用的服务器端 SDK 的详细信息，请参阅 [.NET 服务器 SDK](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/) 或 [Node.js 服务器 SDK](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/) 的文档。

## 参考文档

客户端 SDK 的参考文档位于此处：[Azure Mobile Apps .NET client reference]（Azure 移动应用 .NET 客户端参考）。还可以在 [Azure-Samples GitHub 存储库]中找到多个客户端示例。

##<a name="setup"></a>安装与先决条件

假设已创建并发布移动应用后端项目（至少包含一个表）。在本主题使用的代码中，表的名称为 `TodoItem`，其中包含以下列：`Id`、`Text` 和 `Complete`。这就是当完成 [Azure Mobile Apps quickstart]（Azure 移动应用快速入门）时所创建的同一个表。

相应的类型化客户端 C# 类型如下：

	public class TodoItem
	{
		public string Id { get; set; }

		[JsonProperty(PropertyName = "text")]
		public string Text { get; set; }

		[JsonProperty(PropertyName = "complete")]
		public bool Complete { get; set; }
	}

请注意，[JsonPropertyAttribute] 用于定义客户端类型与表之间的 *PropertyName* 映射。

若要了解如何在移动应用后端创建新表，请参阅 [.NET 服务器 SDK](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#define-table-controller)主题或 [Node.js 服务器 SDK 主题](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#howto-dynamicschema)中的信息。如果已在 Azure 门户中使用快速入门项目创建移动应用后端，则也可以在 [Azure 门户]中使用“简易表”设置。

###如何安装托管的客户端 SDK 包

使用以下方法之一从 [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/) 安装适用于移动应用的托管客户端 SDK 包：

+ **Visual Studio** 右键单击项目，单击“管理 NuGet 包”，搜索 `Microsoft.Azure.Mobile.Client` 包，然后单击“安装”。

+ **Xamarin Studio** 右键单击项目，单击“添加”>“添加 NuGet 包”，搜索 `Microsoft.Azure.Mobile.Client ` 包，然后单击“添加包”。

在主活动文件中，请记得添加以下 **using** 语句：

	using Microsoft.WindowsAzure.MobileServices;

###<a name="symbolsource"></a>如何在 Visual Studio 中使用调试符号

[SymbolSource] 上提供了 Microsoft.Azure.Mobile 命名空间的符号。若要将 SymbolSource 与 Visual Studio 集成，请参阅 [SymbolSource 说明]。

##<a name="create-client"></a>创建移动应用客户端

以下代码创建用于访问移动应用后端的 [MobileServiceClient] 对象。

	MobileServiceClient client = new MobileServiceClient("MOBILE_APP_URL");

在上述代码中，请将 `MOBILE_APP_URL` 替换为移动应用后端的 URL，可以在 [Azure 门户]中移动应用后端的边栏选项卡内找到此 URL。这是一般做法，我们建议使用客户端的单一实例。

## 使用表

以下部分详细说明如何搜索和检索记录，以及修改表中的数据。本文涵盖以下主题：

* [创建表引用](#instantiating)
* [查询数据](#querying)
* [筛选返回的数据](#filtering)
* [为返回的数据排序](#sorting)
* [在页中返回数据](#paging)
* [选择特定的列](#selecting)
* [按 ID 查找记录](#lookingup)
* [处理非类型化查询](#untypedqueries)
* [插入数据](#inserting)
* [更新数据](#updating)
* [删除数据](#deleting)
* [冲突解决和乐观并发](#optimisticconcurrency)
* [绑定到 Windows 用户界面](#binding)
* [更改页面大小](#pagesize)

###<a name="instantiating"></a>如何创建表引用

访问或修改后端表中数据的所有代码均将调用 `MobileServiceTable` 对象的函数。通过对 `MobileServiceClient` 的实例调用 [GetTable] 方法可以获取对表的引用，如下所示：

    IMobileServiceTable<TodoItem> todoTable = client.GetTable<TodoItem>();

这是类型化的序列化模型。也支持非类型化的序列化模型。下面[创建对非类型化表的引用]：

	// Get an untyped table reference
	IMobileServiceTable untypedTodoTable = client.GetTable("TodoItem");

在非类型化查询中，必须指定基础 OData 查询字符串。

###<a name="querying"></a>如何查询移动应用中的数据

本部分介绍如何向包含以下功能的移动应用后端发出查询：

- [筛选返回的数据]
- [为返回的数据排序]
- [在页中返回数据]
- [选择特定的列]
- [按 ID 查找数据]

>[AZURE.NOTE] 将强制使用服务器驱动的页大小来防止返回所有行。这可以防止对大型数据集发出的默认请求对服务造成负面影响。若要返回 50 个以上的行，请根据[在页中返回数据]所述使用 `Take` 方法。

###<a name="filtering"></a>如何筛选返回的数据

以下代码演示了如何通过在查询中包含 `Where` 子句来筛选数据。该代码将返回 `Complete` 属性等于 `false` 的 `todoTable` 中的所有项。[Where] 函数针对该表将一个行筛选谓词应用到查询。

	// This query filters out completed TodoItems and items without a timestamp.
	List<TodoItem> items = await todoTable
	   .Where(todoItem => todoItem.Complete == false)
	   .ToListAsync();

可以使用消息检查软件（例如浏览器开发人员工具或 [Fiddler]）来查看发送到后端的请求的 URI。从下面的请求 URI 中，可以看出已修改查询字符串：

	GET /tables/todoitem?$filter=(complete+eq+false) HTTP/1.1

服务器 SDK 将此 OData 请求转换成类似于以下示例的 SQL 查询：

	SELECT *
	FROM TodoItem
	WHERE ISNULL(complete, 0) = 0

传递给 `Where` 方法的函数可以包含任意数目的条件。例如，以下行：

	// This query filters out completed TodoItems where Text isn't null
	List<TodoItem> items = await todoTable
	   .Where(todoItem => todoItem.Complete == false && todoItem.Text != null)
	   .ToListAsync();

服务器 SDK 转换成类似于以下示例的 SQL 查询：

	SELECT *
	FROM TodoItem
	WHERE ISNULL(complete, 0) = 0
	      AND ISNULL(text, 0) = 0

上述 `WHERE` 语句将查找 `Complete` 状态设置为 false 且 `Text` 不为 null 的项。

此查询也可以拆分成多个子句：

	List<TodoItem> items = await todoTable
	   .Where(todoItem => todoItem.Complete == false)
	   .Where(todoItem => todoItem.Text != null)
	   .ToListAsync();

这两种方法是等效的，可以换用。前一个选项（在一个查询中连接多个谓词）更为精简，也是我们推荐的方法。

`Where` 子句支持可转换成 OData 子集的操作。其中包括关系运算符（==、!=、<、<=、>、>=）、数学运算符（+、-、/、*、%）、数字精度（Math.Floor、Math.Ceiling）、字符串函数（Length、Substring、Replace、IndexOf、StartsWith、EndsWith）、日期属性（Year、Month、Day、Hour、Minute、Second）、对象的访问属性，以及组合了上述所有操作的表达式。在考虑服务器 SDK 支持的操作时，可以参考 [OData v3 文档]。

###<a name="sorting"></a>如何为返回的数据排序

以下代码演示如何通过在查询中包含 [OrderBy] 或 [OrderByDescending] 函数来为数据排序。该代码将返回 `todoTable` 中的项，这些项已按 `Text` 字段的升序排序。

	// Sort items in ascending order by Text field
	MobileServiceTableQuery<TodoItem> query = todoTable
					.OrderBy(todoItem => todoItem.Text)
 	List<TodoItem> items = await query.ToListAsync();

	// Sort items in descending order by Text field
	MobileServiceTableQuery<TodoItem> query = todoTable
					.OrderByDescending(todoItem => todoItem.Text)
 	List<TodoItem> items = await query.ToListAsync();

###<a name="paging"></a>如何在页中返回数据

默认情况下，后端只返回前 50 行。你可以通过调用 [Take] 方法来增加返回的行数。将 `Take` 与 [Skip] 方法一起使用可以请求查询返回的总数据集的特定“页”。执行以下查询后，将返回表中的前三个项。

	// Define a filtered query that returns the top 3 items.
	MobileServiceTableQuery<TodoItem> query = todoTable
					.Take(3);
	List<TodoItem> items = await query.ToListAsync();

以下经过修改的查询将跳过前三个结果，返回其后的三个结果。实际上这是数据的第二“页”，其页大小为三个项。

	// Define a filtered query that skips the top 3 items and returns the next 3 items.
	MobileServiceTableQuery<TodoItem> query = todoTable
					.Skip(3)
					.Take(3);
	List<TodoItem> items = await query.ToListAsync();

你还可以使用 [IncludeTotalCount] 方法来确保查询获取应该返回的<i>所有</i>记录的总计数，并忽略指定的任何 take 分页/限制子句：

	query = query.IncludeTotalCount();

这是将硬编码分页值传递给 `Take` 和 `Skip` 方法的简化方案。在实际应用中，你可以对页导航控件或类似的 UI 使用类似于上面的查询，让用户导航到上一页和下一页。

>[AZURE.NOTE]若要重写移动应用后端中的 50 行限制，还必须将 [EnableQueryAttribute] 应用到公共 GET 方法，并指定分页行为。将以下语句应用到该方法后，最大返回行数将设置为 1000：
>
>    [EnableQuery(MaxTop=1000)]

### <a name="selecting"></a>如何选择特定的列

可以通过在查询中添加 [Select] 子句来指定要包含在结果中的属性集。例如，以下代码演示了如何做到只选择一个字段，以及如何选择并格式化多个字段：

	// Select one field -- just the Text
	MobileServiceTableQuery<TodoItem> query = todoTable
					.Select(todoItem => todoItem.Text);
	List<string> items = await query.ToListAsync();

	// Select multiple fields -- both Complete and Text info
	MobileServiceTableQuery<TodoItem> query = todoTable
					.Select(todoItem => string.Format("{0} -- {1}",
						todoItem.Text.PadRight(30), todoItem.Complete ?
						"Now complete!" : "Incomplete!"));
	List<string> items = await query.ToListAsync();

到目前为止所述的所有函数都是加性函数，我们可以不断地调用它们，每次调用都能进一步影响查询。再提供一个示例：

	MobileServiceTableQuery<TodoItem> query = todoTable
					.Where(todoItem => todoItem.Complete == false)
					.Select(todoItem => todoItem.Text)
					.Skip(3).
					.Take(3);
	List<string> items = await query.ToListAsync();

### <a name="lookingup"></a>如何：按 ID 查找数据

使用 [LookupAsync] 函数可以查找数据库中具有特定 ID 的对象。

	// This query filters out the item with the ID of 37BBF396-11F0-4B39-85C8-B319C729AF6D
	TodoItem item = await todoTable.LookupAsync("37BBF396-11F0-4B39-85C8-B319C729AF6D");

### <a name="untypedqueries"></a>如何执行非类型化查询

使用非类型化的表对象执行查询时，必须通过调用 [ReadAsync] 显式指定 OData 查询字符串，如以下示例中所示：

	// Lookup untyped data using OData
	JToken untypedItems = await untypedTodoTable.ReadAsync("$filter=complete eq 0&$orderby=text");

此时，你将获取一些可以像属性包一样使用的 JSON 值。有关 JToken 和 Newtonsoft Json.NET 的详细信息，请参阅 [Json.NET] 站点。

### <a name="inserting"></a>如何将数据插入移动应用后端

所有客户端类型必须包含名为 **Id** 的成员，其默认为字符串。需要有此 **Id** 才能执行脱机 CRUD 操作。以下代码演示如何使用 [InsertAsync] 方法将新行插入表中。参数包含要作为 .NET 对象插入的数据。

	await todoTable.InsertAsync(todoItem);

如果在传递给 `todoTable.InsertAsync` 调用的 `todoItem` 中未包含唯一的自定义 ID 值，则服务器将会生成一个 ID 值，并在返回到客户端的 `todoItem` 对象中设置该值。

若要插入非类型化数据，你可以按如下所示利用 Json.NET。

	JObject jo = new JObject();
	jo.Add("Text", "Hello World");
	jo.Add("Complete", false);
	var inserted = await table.InsertAsync(jo);

以下示例使用电子邮件地址作为唯一的字符串 ID。

	JObject jo = new JObject();
	jo.Add("id", "myemail@emaildomain.com");
	jo.Add("Text", "Hello World");
	jo.Add("Complete", false);
	var inserted = await table.InsertAsync(jo);

###使用 ID 值

移动应用支持为表的 **ID** 列使用唯一的自定义字符串值。这样，应用程序便可为 ID 使用自定义值（如电子邮件地址或用户名）。

字符串 ID 可提供以下优势：

* 无需往返访问数据库即可生成 ID。
* 更方便地合并不同表或数据库中的记录。
* ID 值能够更好地与应用程序的逻辑相集成。

如果插入的记录中未设置字符串 ID 值，移动应用后端将为 ID 生成唯一值。可以在客户端或后端中使用 [Guid.NewGuid] 方法生成自己的 ID 值。

    JObject jo = new JObject();
    jo.Add("id", Guid.NewGuid().ToString("N"));

###<a name="modifying"></a>如何修改移动应用后端中的数据

以下代码演示如何通过 [UpdateAsync] 方法使用新信息更新具相同 ID 的现有记录。参数包含要作为 .NET 对象更新的数据。

	await todoTable.UpdateAsync(todoItem);

若要插入非类型化数据，可按如下所示利用 [Json.NET]：

	JObject jo = new JObject();
	jo.Add("id", "37BBF396-11F0-4B39-85C8-B319C729AF6D");
	jo.Add("Text", "Hello World");
	jo.Add("Complete", false);
	var inserted = await table.UpdateAsync(jo);

更新时，必须指定 `id` 字段。这是后端识别要更新哪个实例的方式。可以从 `InsertAsync` 调用的结果中获取 `id` 字段。如果尝试更新项但未提供 `id` 值，将引发 `ArgumentException`。

###<a name="deleting"></a>如何删除移动应用后端中的数据

以下代码演示如何使用 [DeleteAsync] 方法删除现有实例。可以通过 `todoItem` 中设置的 `id` 字段来标识实例。

	await todoTable.DeleteAsync(todoItem);

若要删除非类型化数据，可按如下所示利用 Json.NET：

	JObject jo = new JObject();
	jo.Add("id", "37BBF396-11F0-4B39-85C8-B319C729AF6D");
	await table.DeleteAsync(jo);

请注意，发出删除请求时，必须指定 ID。其他属性不会传递到服务，否则服务会将它们忽略。`DeleteAsync` 调用的结果通常是 `null`。可以从 `InsertAsync` 调用的结果中获取要传入的 ID。如果尝试删除项但未指定 `id` 字段，将引发 `MobileServiceInvalidOperationException`。

###<a name="optimisticconcurrency"></a>如何使用乐观并发解决冲突

两个或两个以上客户端可能会同时将更改写入同一项目。如果没有任何冲突检测，则最后一次写入会覆盖任何以前的更新，即使这并不是所需要的结果。 *乐观并发控制* 假设每个事务均可以提交，因此不使用任何资源锁定。提交事务之前，乐观并发控制将验证是否没有其他事务修改了数据。如果数据已修改，则将回滚正在提交的事务。

移动应用通过使用 `version` 系统属性列（该列是为移动应用后端中的每个表定义的）跟踪对每个项的更改来支持乐观并发控制。每次更新某个记录时，移动应用都将该记录的 `version` 属性设置为新值。在每次执行更新请求期间，会将该请求包含的记录的 `version` 属性与服务器上的记录的同一属性进行比较。如果随请求传递的版本与后端不匹配，客户端库将引发 `MobileServicePreconditionFailedException<T>` 异常。该异常中提供的类型就是包含记录服务器版本的后端中的记录。然后，应用程序可以借助此信息来确定是否要使用后端中正确的 `version` 值再次执行更新请求以提交更改。

在 `version` 系统属性的表类中定义列，启用乐观并发，例如：

    public class TodoItem
    {
        public string Id { get; set; }

        [JsonProperty(PropertyName = "text")]
        public string Text { get; set; }

        [JsonProperty(PropertyName = "complete")]
        public bool Complete { get; set; }

        // *** Enable Optimistic Concurrency *** //
        [JsonProperty(PropertyName = "version")]
        public string Version { set; get; }
    }


使用非类型化表的应用程序通过在表的 `SystemProperties` 中设置 `Version` 标志来启用乐观并发，如下所示。

	//Enable optimistic concurrency by retrieving version
	todoTable.SystemProperties |= MobileServiceSystemProperties.Version;

除了启用乐观并发以外，还必须在调用 [UpdateAsync] 时捕获代码中的 `MobileServicePreconditionFailedException<T>` 异常。将正确的 `version` 应用到更新的记录，并使用解决的记录调用 [UpdateAsync]，即可解决冲突。以下代码演示如何解决检测到的写入冲突：

	private async void UpdateToDoItem(TodoItem item)
	{
    	MobileServicePreconditionFailedException<TodoItem> exception = null;

	    try
    	{
	        //update at the remote table
    	    await todoTable.UpdateAsync(item);
    	}
    	catch (MobileServicePreconditionFailedException<TodoItem> writeException)
	    {
        	exception = writeException;
	    }

    	if (exception != null)
    	{
			// Conflict detected, the item has changed since the last query
        	// Resolve the conflict between the local and server item
	        await ResolveConflict(item, exception.Item);
    	}
	}


	private async Task ResolveConflict(TodoItem localItem, TodoItem serverItem)
	{
    	//Ask user to choose the resoltion between versions
	    MessageDialog msgDialog = new MessageDialog(
            String.Format("Server Text: "{0}" \nLocal Text: "{1}"\n",
            serverItem.Text, localItem.Text),
            "CONFLICT DETECTED - Select a resolution:");

	    UICommand localBtn = new UICommand("Commit Local Text");
    	UICommand ServerBtn = new UICommand("Leave Server Text");
    	msgDialog.Commands.Add(localBtn);
	    msgDialog.Commands.Add(ServerBtn);

    	localBtn.Invoked = async (IUICommand command) =>
	    {
        	// To resolve the conflict, update the version of the item being committed. Otherwise, you will keep
        	// catching a MobileServicePreConditionFailedException.
	        localItem.Version = serverItem.Version;

    	    // Updating recursively here just in case another change happened while the user was making a decision
	        UpdateToDoItem(localItem);
    	};

	    ServerBtn.Invoked = async (IUICommand command) =>
    	{
	        RefreshTodoItems();
    	};

	    await msgDialog.ShowAsync();
	}

有关详细信息，请参阅 [Offline Data Sync in Azure Mobile Apps]（Azure 移动应用中的脱机数据同步）主题。

###<a name="binding"></a>如何将移动应用数据绑定到 Windows 用户界面

本部分说明如何使用 Windows 应用中的 UI 元素显示返回的数据对象。可以运行以下示例代码，使用查询绑定到列表源，从而找出 `todoTable` 中的不完整项并在极简单的列表中显示这些项。[MobileServiceCollection] 创建移动应用感知的绑定集合。

	// This query filters out completed TodoItems.
	MobileServiceCollection<TodoItem, TodoItem> items = await todoTable
		.Where(todoItem => todoItem.Complete == false)
		.ToCollectionAsync();

	// itemsControl is an IEnumerable that could be bound to a UI list control
	IEnumerable itemsControl  = items;

	// Bind this to a ListBox
	ListBox lb = new ListBox();
	lb.ItemsSource = items;

托管运行时中的某些控件支持名为 [ISupportIncrementalLoading] 的接口。当用户滚动浏览时，此接口允许控件请求更多的数据。系统通过 [MobileServiceIncrementalLoadingCollection]（可自动处理来自控件的调用）为这个适用于通用 Windows 应用程序的接口提供内置支持。若要在 Windows 应用中使用 `MobileServiceIncrementalLoadingCollection`，请执行以下代码：

    MobileServiceIncrementalLoadingCollection<TodoItem,TodoItem> items;
    items = todoTable.Where(todoItem => todoItem.Complete == false).ToIncrementalLoadingCollection();

    ListBox lb = new ListBox();
    lb.ItemsSource = items;

若要在 Windows Phone 8 和“Silverlight”应用上使用新的集合，请在 `IMobileServiceTableQuery<T>` 和 `IMobileServiceTable<T>` 上使用 `ToCollection` 扩展方法。若要实际加载数据，请调用 `LoadMoreItemsAsync()`。

	MobileServiceCollection<TodoItem, TodoItem> items = todoTable.Where(todoItem => todoItem.Complete==false).ToCollection();
	await items.LoadMoreItemsAsync();

当你使用通过调用 `ToCollectionAsync` 或 `ToCollection` 创建的集合时，可以获取可绑定到 UI 控件的集合。此集合支持分页，也就是说，控件可以要求该集合“加载更多项”，而该集合也确实会这样做。这样看来，无需执行任何用户代码，控件就能启动工作流。但是，由于集合要从网络加载数据，因此可以预料到这种加载有时会失败。若要处理这种故障，你可以重写 `MobileServiceIncrementalLoadingCollection` 中的 `OnException` 方法，以处理调用控件执行的 `LoadMoreItemsAsync` 后发生的异常。

最后，假设你的表包含许多字段，但你只想在控件中显示其中的某些字段。在这种情况下，你可以参考上面“[选择特定的列](#selecting)”部分中的指导，选择要在 UI 中显示的特定列。

###<a name="pagesize"></a>更改页面大小

默认情况下，Azure 移动应用针对每个请求最多返回 50 个项。可以通过增大服务器中的页面大小上限，以及增大客户端中请求的页面大小，来更改此设置。若要增大请求的页面大小，请使用 `PullAsync` 的重载指定 `PullOptions`：

    PullOptions pullOptions = new PullOptions
		{
			MaxPageSize = 100
		};

假设已在服务器中使 PageSize 等于或大于 100，这样，每个请求最多可返回 100 个项。

##<a name="#customapi"></a>使用自定义 API

自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

通过在客户端上调用某一个 [InvokeApiAsync] 方法重载来调用自定义 API。例如，以下代码行向后端上的 **completeAll** API 发送 POST 请求：

    var result = await client.InvokeApiAsync<MarkAllResult>("completeAll", System.Net.Http.HttpMethod.Post, null);

请注意，这种类型化方法调用要求定义 **MarkAllResult** 返回类型。支持类型化和非类型化的方法。


##<a name="authentication"></a>对用户进行身份验证

移动应用支持使用各种外部标识提供者对应用用户进行身份验证和授权，这些提供者包括：Microsoft 帐户和 Azure Active Directory。你可以在表中设置权限，以便将特定操作的访问权限限制给已经过身份验证的用户。你还可以在服务器脚本中使用已经过身份验证的用户的标识来实施授权规则。有关详细信息，请参阅[向应用程序添加身份验证]教程。

支持两种身份验证流：_客户端托管_流和_服务器托管_流。服务器流依赖于提供者的 Web 身份验证界面，因此可提供最简便的身份验证体验。客户端流依赖于提供者和设备特定的 SDK，因此允许与设备特定的功能进行更深入的集成。

>[AZURE.NOTE] 建议在生产应用中使用客户端托管流。

若要设置身份验证，必须向一个或多个标识提供者注册应用。标识提供者将为应用生成客户端 ID 和客户端机密，然后在后端中进行设置，使用该标识提供者启用 Azure 应用服务身份验证/授权。有关详细信息，请遵循 [Add authentication to your app]（将身份验证添加到应用）教程中的详细说明。

本部分介绍以下主题：

+ [客户端托管的身份验证](#client-flow)
+ [服务器托管的身份验证](#serverflow)
+ [缓存身份验证令牌](#caching)

###<a name="client-flow"></a>客户端托管的身份验证

应用可以独立联系标识提供者，然后在用后端登录期间提供返回的令牌。使用此客户端流可为用户提供单一登录体验，或者从标识提供者中检索其他用户数据。这可能比使用服务器流更有利，因为它提供更直观的 UX 风格，并允许其他自定义。

提供了以下客户端流身份验证模式的示例：

+ [Active Directory 身份验证库](#adal)
+ [Live SDK](#client-livesdk)

#### <a name="adal"></a>使用 Active Directory 身份验证库对用户进行身份验证

可以使用 Active Directory 身份验证库 (ADAL)，从使用 Azure Active Directory 身份验证的客户端启动用户身份验证。

1. 根据 [How to configure App Service for Active Directory login]（如何为 Active Directory 登录配置应用服务）教程的说明，为 AAD 登录配置移动应用。请务必完成注册本机客户端应用程序的可选步骤。

2. 在 Visual Studio 或 Xamarin Studio 中打开项目，然后添加对 `Microsoft.IdentityModel.CLients.ActiveDirectory` NuGet 包的引用。搜索时，请包含预发行版。

3. 根据使用的平台，将以下代码添加到应用程序。在每条代码中进行以下替换：

	* 将 **INSERT-AUTHORITY-HERE** 替换为在其中预配应用程序的租户的名称。格式应为 https://login.chinacloudapi.cn/contoso.partner.onmschina.cn。可以在 [Azure 经典管理门户]中 Azure Active Directory 的“域”选项卡复制此值。
	
	* 将 **INSERT-RESOURCE-ID-HERE** 替换移动应用后端的客户端 ID。可以在门户中“Azure Active Directory 设置”下面的“高级”选项卡获取此值。
	
	* 将 **INSERT-CLIENT-ID-HERE** 替换为从本机客户端应用程序复制的客户端 ID。
	
	* 将 **INSERT-REDIRECT-URI-HERE** 替换为站点的 _/.auth/login/done_ 终结点（使用 HTTPS 方案）。此值应类似于 \_https://contoso.chinacloudsites.cn/.auth/login/done_。
	
	每个平台所需的代码如下：
	
	**Windows:**
	
	    private MobileServiceUser user;
	    private async Task AuthenticateAsync()
	    {
	        string authority = "INSERT-AUTHORITY-HERE";
	        string resourceId = "INSERT-RESOURCE-ID-HERE";
	        string clientId = "INSERT-CLIENT-ID-HERE";
	        string redirectUri = "INSERT-REDIRECT-URI-HERE";
	        while (user == null)
	        {
	            string message;
	            try
	            {
	                AuthenticationContext ac = new AuthenticationContext(authority);
	                AuthenticationResult ar = await ac.AcquireTokenAsync(resourceId, clientId, 
						new Uri(redirectUri), new PlatformParameters(PromptBehavior.Auto, false) );
	                JObject payload = new JObject();
	                payload["access_token"] = ar.AccessToken;
	                user = await App.MobileService.LoginAsync(
						MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, payload);
	                message = string.Format("You are now logged in - {0}", user.UserId);
	            }
	            catch (InvalidOperationException)
	            {
	                message = "You must log in. Login Required";
	            }
	            var dialog = new MessageDialog(message);
	            dialog.Commands.Add(new UICommand("OK"));
	            await dialog.ShowAsync();
	        }
	    }
	
	**Xamarin.iOS**
	
	    private MobileServiceUser user;
	    private async Task AuthenticateAsync(UIViewController view)
	    {
	        string authority = "INSERT-AUTHORITY-HERE";
	        string resourceId = "INSERT-RESOURCE-ID-HERE";
	        string clientId = "INSERT-CLIENT-ID-HERE";
	        string redirectUri = "INSERT-REDIRECT-URI-HERE";
	        try
	        {
	            AuthenticationContext ac = new AuthenticationContext(authority);
	            AuthenticationResult ar = await ac.AcquireTokenAsync(resourceId, clientId, 
					new Uri(redirectUri), new PlatformParameters(view));
	            JObject payload = new JObject();
	            payload["access_token"] = ar.AccessToken;
	            user = await client.LoginAsync(
					MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, payload);
	        }
	        catch (Exception ex)
	        {
	            Console.Error.WriteLine(@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
	        }
	    }
	
	**Xamarin.Android**
	
	    private MobileServiceUser user;
	    private async Task AuthenticateAsync()
	    {
	        string authority = "INSERT-AUTHORITY-HERE";
	        string resourceId = "INSERT-RESOURCE-ID-HERE";
	        string clientId = "INSERT-CLIENT-ID-HERE";
	        string redirectUri = "INSERT-REDIRECT-URI-HERE";
	        try
	        {
	            AuthenticationContext ac = new AuthenticationContext(authority);
	            AuthenticationResult ar = await ac.AcquireTokenAsync(resourceId, clientId, 
					new Uri(redirectUri), new PlatformParameters(this));
	            JObject payload = new JObject();
	            payload["access_token"] = ar.AccessToken;
	            user = await client.LoginAsync(
					MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, payload);
	        }
	        catch (Exception ex)
	        {
	            AlertDialog.Builder builder = new AlertDialog.Builder(this);
	            builder.SetMessage(ex.Message);
	            builder.SetTitle("You must log in. Login Required");
	            builder.Create().Show();
	        }
	    }
	    protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
	    {
	        base.OnActivityResult(requestCode, resultCode, data);
	        AuthenticationAgentContinuationHelper.SetAuthenticationAgentContinuationEventArgs(requestCode, resultCode, data);
	    }

####<a name="client-livesdk"></a>使用 Microsoft 帐户和 Live SDK 进行单一登录

若要对用户进行身份验证，必须在 Microsoft 帐户开发人员中心注册你的应用程序。然后，必须将此注册连接到移动应用后端。完成 [Register your app to use a Microsoft account login]（注册应用以使用 Microsoft 帐户登录）中的步骤，创建 Microsoft 帐户注册并将注册连接到移动应用后端。如果你同时拥有 Windows 应用商店和 Windows Phone 8/Silverlight 版本的应用，请先注册 Windows 应用商店版本。

以下代码使用 Live SDK 进行身份验证，并使用返回的令牌登录到移动应用后端。

	private LiveConnectSession session;
 	//private static string clientId = "<microsoft-account-client-id>";
    private async System.Threading.Tasks.Task AuthenticateAsync()
    {

        // Get the URL the Mobile App backend.
        var serviceUrl = App.MobileService.ApplicationUri.AbsoluteUri;

        // Create the authentication client for Windows Store using the service URL.
        LiveAuthClient liveIdClient = new LiveAuthClient(serviceUrl);
        //// Create the authentication client for Windows Phone using the client ID of the registration.
        //LiveAuthClient liveIdClient = new LiveAuthClient(clientId);

        while (session == null)
        {
            // Request the authentication token from the Live authentication service.
			// The wl.basic scope should always be requested.  Other scopes can be added
            LiveLoginResult result = await liveIdClient.LoginAsync(new string[] { "wl.basic" });
            if (result.Status == LiveConnectSessionStatus.Connected)
            {
                session = result.Session;

                // Get information about the logged-in user.
                LiveConnectClient client = new LiveConnectClient(session);
                LiveOperationResult meResult = await client.GetAsync("me");

                // Use the Microsoft account auth token to sign in to App Service.
                MobileServiceUser loginResult = await App.MobileService
                    .LoginWithMicrosoftAccountAsync(result.Session.AuthenticationToken);

                // Display a personalized sign-in greeting.
                string title = string.Format("Welcome {0}!", meResult.Result["first_name"]);
                var message = string.Format("You are now logged in - {0}", loginResult.UserId);
                var dialog = new MessageDialog(message, title);
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }
            else
            {
                session = null;
                var dialog = new MessageDialog("You must log in.", "Login Required");
                dialog.Commands.Add(new UICommand("OK"));
                await dialog.ShowAsync();
            }
        }
    }

有关 [Windows Live SDK] 的详细信息，请参阅相关文档。

###<a name="serverflow"></a>服务器托管的身份验证
注册标识提供者后，使用提供者的 [MobileServiceAuthenticationProvider] 值对 [MobileServiceClient] 调用 [LoginAsync] 方法。例如，以下代码使用 Microsoft 启动服务器流登录。

	private MobileServiceUser user;
	private async System.Threading.Tasks.Task Authenticate()
	{
		while (user == null)
		{
			string message;
			try
			{
				user = await client
					.LoginAsync(MobileServiceAuthenticationProvider.Microsoft);
				message =
					string.Format("You are now logged in - {0}", user.UserId);
			}
			catch (InvalidOperationException)
			{
				message = "You must log in. Login Required";
			}

			var dialog = new MessageDialog(message);
			dialog.Commands.Add(new UICommand("OK"));
			await dialog.ShowAsync();
		}
	}

如果使用的标识提供者不是 Microsoft，请将上述 [MobileServiceAuthenticationProvider] 的值更改为提供者的值。

在服务器流中，Azure 应用服务通过以下方式管理 OAuth 2.0 身份验证流：显示选定提供者的登录页，并在用户成功使用标识提供者登录后生成应用服务身份验证令牌。[LoginAsync 方法]返回 [MobileServiceUser]，后者提供已经过身份验证的用户的 [UserId]，以及 JSON Web 令牌 (JWT) 形式的 [MobileServiceAuthenticationToken]。你可以缓存此令牌，并在它过期之前重复使用。有关详细信息，请参阅[缓存身份验证令牌](#caching)。

###<a name="caching"></a>缓存身份验证令牌

在某些情况下，使用客户端流程时，存储身份验证令牌甚至来自提供程序的访问令牌可避免在首次成功身份验证后调用登录方法。

Windows 应用商店和 UWP 应用可以使用 [PasswordVault] 在成功登录后缓存当前身份验证令牌，如下所示：

	await client.LoginAsync(MobileServiceAuthenticationProvider.Microsoft);		

	PasswordVault vault = new PasswordVault();
	vault.Add(new PasswordCredential("Microsoft", client.currentUser.UserId, 
		client.currentUser.MobileServiceAuthenticationToken));

UserId 值存储为凭据的 UserName，令牌存储为 Password。在后续启动时，可以检查 **PasswordVault** 中的缓存凭据。以下示例使用找到的缓存凭据，否则尝试再次向后端进行身份验证：

	// Try to retrieve stored credentials.
	var creds = vault.FindAllByResource("Microsoft").FirstOrDefault();
	if (creds != null)
	{
		// Create the current user from the stored credentials.
		client.currentUser = new MobileServiceUser(creds.UserName);
		client.currentUser.MobileServiceAuthenticationToken = 
			vault.Retrieve("Microsoft", creds.UserName).Password;
	}
	else
	{
		// Regular login flow and cache the token as shown above.
	}

注销用户时，还必须删除存储的凭据，如下所示：

	client.Logout();
	vault.Remove(vault.Retrieve("Microsoft", client.currentUser.UserId));

Xamarin 应用使用 [Xamarin.Auth API](https://components.xamarin.com/view/xamarin.auth/) 将证书安全存储在 **Account** 对象中。有关使用这些 API 的示例，请参阅 [ContosoMoments photo sharing sample](https://github.com/azure-appservice-samples/ContosoMoments/tree/dev)（ContosoMoments 照片分享示例）中的 [AuthStore.cs](https://github.com/azure-appservice-samples/ContosoMoments/blob/dev/src/Mobile/ContosoMoments/Helpers/AuthStore.cs) 代码文件。

使用客户端托管的身份验证时，也可以缓存从提供程序（例如 Microsoft）获取的访问令牌。可以提供此令牌，从后端请求新的身份验证令牌，如下所示：

	var token = new JObject();
	// Replace <your_access_token_value> with actual value of your access token
	token.Add("access_token", "<your_access_token_value>");

	// Authenticate using the access token.
	await client.LoginAsync(MobileServiceAuthenticationProvider.Microsoft, token);


##<a name="pushnotifications"></a>推送通知

以下主题介绍了推送通知：

* [注册推送通知](#register-for-push)
* [获取 Windows 应用商店包 SID](#package-sid)
* [使用跨平台模板注册](#register-xplat)

###<a name="register-for-push"></a>如何注册推送通知

使用移动应用客户端可向 Azure 通知中心注册推送通知。注册时，你将获得从平台特定的推送通知服务 (PNS) 获取的句柄。然后你就可以在创建注册时提供此值以及任何标记。以下代码将用于推送通知的 Windows 应用注册到 Windows 通知服务 (WNS)：

    private async void InitNotificationsAsync()
    {
        // Request a push notification channel.
        var channel =await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        // Register for notifications using the new channel.
        await MobileService.GetPush().RegisterNativeAsync(channel.Uri, null);
    }

如果要推送到 WNS，必须获取 Windows 应用商店包 SID（如下所示）。请注意，在此示例中，注册包含两个标记。有关 Windows 应用的详细信息，包括如何注册模板，请参阅 [Add push notifications to your app]（将推送通知添加到应用）。

请注意，不支持从客户端请求标记。注册时将静默删除标记请求。如果想要使用标记注册设备，请创建自定义 API，使用通知中心 API 自动执行注册。调用[自定义 API](#customapi) 而不是 `RegisterNativeAsync()` 方法。

###<a name="package-sid"></a>如何获取 Windows 应用商店包 SID

在 Windows 应用商店应用中启用推送通知需有包 SID。包 SID 也可用于其他操作（如 Windows 单一登录）。需要向 Windows 应用商店注册应用程序才能收到包 SID。

若要获取此值，请执行以下操作：

1. 在“Visual Studio 资源管理器”中，右键单击 Windows 应用商店应用项目，单击“应用商店”>“将应用与应用商店关联...”。
2. 在向导中，单击“下一步”，使用 Microsoft 帐户登录，在“保留新应用名称”中键入应用的名称，然后单击“保留”。
3. 成功创建应用注册后，选择新应用名称，单击“下一步”、“关联”。这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。
4. 使用 Microsoft 帐户登录到 [Windows 开发人员中心]。在“我的应用”下面，单击刚刚创建的应用注册。
5. 单击“应用管理”>“应用标识”，然后向下滚动找到“包 SID”。

包 SID 的许多用法将其视为 URI，在这种情况下，需要使用 _ms-app://_ 作为方案。记下包 SID 的版本，其中串联了此值作为前缀。

Xamarin 应用需要一些额外的代码才能将 iOS 上运行的应用注册到 Apple Push Notification 服务 (APNS)。有关详细信息，请参阅适用于平台的主题：

* [Xamarin.Android](/documentation/articles/app-service-mobile-xamarin-android-get-started-push/#add-push)
* [Xamarin.iOS](/documentation/articles/app-service-mobile-xamarin-ios-get-started-push/#add-push)

###<a name="register-xplat"></a>如何注册推送模板以发送跨平台通知

若要注册模板，请结合模板使用 `RegisterAsync()` 方法，如下所示：

        JObject templates = myTemplates();
        MobileService.GetPush().RegisterAsync(channel.Uri, templates);

模板类型是 JObject，可以包含采用以下 JSON 格式的多个模板：

        public JObject myTemplates()
        {
            // single template for Windows Notification Service toast
            var template = "<toast><visual><binding template="ToastText01"><text id="1">$(message)</text></binding></visual></toast>";

            var templates = new JObject
            {
                ["generic-message"] = new JObject
                {
                    ["body"] = template,
                    ["headers"] = new JObject
                    {
                        ["X-WNS-Type"] = "wns/toast"
                    },
                    ["tags"] = new JArray()
                },
                ["more-templates"] = new JObject {...}
            };
            return templates;
        }

方法 **RegisterAsync()** 也接受辅助磁贴：

        MobileService.GetPush().RegisterAsync(string channelUri, JObject templates, JObject secondaryTiles);

请注意，所有标记因安全考虑而删除。若要将标记添加到安装或安装中的模板，请参阅 [Work with the .NET backend server SDK for Azure Mobile Apps]（使用适用于 Azure 移动应用的 .NET 后端服务器 SDK）。

若要使用这些注册的模板发送通知，请参阅 [Notification Hubs APIs]（通知中心 API）。

##<a name="misc"></a>其他主题

###<a name="errors"></a>如何：处理错误

当后端发生错误时，客户端 SDK 将引发 `MobileServiceInvalidOperationException`。以下示例演示如何处理后端返回的异常：

	private async void InsertTodoItem(TodoItem todoItem)
	{
		// This code inserts a new TodoItem into the database. When the operation completes
		// and App Service has assigned an Id, the item is added to the CollectionView
		try
		{
			await todoTable.InsertAsync(todoItem);
			items.Add(todoItem);
		}
		catch (MobileServiceInvalidOperationException e)
		{
			// Handle error
		}
	}

在 [Mobile Apps Files Sample]（移动应用文件示例）中可以找到有关处理错误状态的另一个示例 - [LoggingHandler] 示例提供日志记录委托处理例程（请参阅下文），将发出的请求记录到后端。这样，便可以在不依赖 Fiddler 的情况下轻松调试 Xamarin 应用程序。

###<a name="headers"></a>如何自定义请求标头

若要支持特定的应用程序方案，可能需要自定义与移动应用后端之间的通信。例如，你可能需要将一个自定义标头添加到每个传出请求，甚至要更改响应状态代码。可以通过提供自定义 [DelegatingHandler] 来实现此目的，如以下示例中所示：

    public async Task CallClientWithHandler()
    {
        HttpResponseMessage[]
        MobileServiceClient client = new MobileServiceClient("AppUrl",
            new MyHandler()
            );
        IMobileServiceTable<TodoItem> todoTable = client.GetTable<TodoItem>();
        var newItem = new TodoItem { Text = "Hello world", Complete = false };
        await todoTable.InsertAsync(newItem);
    }

    public class MyHandler : DelegatingHandler
    {
        protected override async Task<HttpResponseMessage>
            SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            // Change the request-side here based on the HttpRequestMessage
            request.Headers.Add("x-my-header", "my value");

            // Do the request
            var response = await base.SendAsync(request, cancellationToken);

            // Change the response-side here based on the HttpResponseMessage

            // Return the modified response
            return response;
        }
    }


<!-- Anchors. -->
[筛选返回的数据]: #filtering
[为返回的数据排序]: #sorting
[在页中返回数据]: #paging
[选择特定的列]: #selecting
[按 ID 查找数据]: #lookingup

<!-- Images. -->

<!-- Internal URLs. -->
[Azure Mobile Apps quickstart]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started/
[Add authentication to your app]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started-users/
[向应用程序添加身份验证]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started-users/
[Work with .NET backend SDK]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Work with the .NET backend server SDK for Azure Mobile Apps]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[How to use the Node.js backend SDK]: /documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/
[How to: Define a table controller]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#how-to-define-a-table-controller
[Define Tables using a Dynamic Schema]: app-service-mobile-node-backend-how-to-use-server-sdk.md#TableOperations
[Offline Data Sync in Azure Mobile Apps]: /documentation/articles/app-service-mobile-offline-data-sync/
[Add push notifications to your app]: /documentation/articles/app-service-mobile-windows-store-dotnet-get-started-push/
[Register your app to use a Microsoft account login]: /documentation/articles/app-service-mobile-how-to-configure-microsoft-authentication/
[How to configure App Service for Active Directory login]: /documentation/articles/app-service-mobile-how-to-configure-active-directory-authentication/

<!-- Microsoft URLs. -->
[Azure Mobile Apps .NET client reference]: https://msdn.microsoft.com/zh-cn/library/azure/mt419521(v=azure.10).aspx
[MobileServiceClient]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient(v=azure.10).aspx
[MobileServiceCollection]: https://msdn.microsoft.com/zh-cn/library/azure/dn250636(v=azure.10).aspx
[MobileServiceIncrementalLoadingCollection]: https://msdn.microsoft.com/zh-cn/library/azure/dn268408(v=azure.10).aspx
[MobileServiceAuthenticationProvider]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceauthenticationprovider(v=azure.10).aspx
[MobileServiceUser]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceuser(v=azure.10).aspx
[MobileServiceAuthenticationToken]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceuser.mobileserviceauthenticationtoken(v=azure.10).aspx
[GetTable]: https://msdn.microsoft.com/zh-cn/library/azure/jj554275(v=azure.10).aspx
[创建对非类型化表的引用]: https://msdn.microsoft.com/zh-cn/library/azure/jj554278(v=azure.10).aspx
[DeleteAsync]: https://msdn.microsoft.com/zh-cn/library/azure/dn296407(v=azure.10).aspx
[IncludeTotalCount]: https://msdn.microsoft.com/zh-cn/library/azure/dn250560(v=azure.10).aspx
[InsertAsync]: https://msdn.microsoft.com/zh-cn/library/azure/dn296400(v=azure.10).aspx
[InvokeApiAsync]: https://msdn.microsoft.com/zh-cn/library/azure/dn268343(v=azure.10).aspx
[LoginAsync]: https://msdn.microsoft.com/zh-cn/library/azure/dn296411(v=azure.10).aspx
[LoginAsync 方法]: https://msdn.microsoft.com/zh-cn/library/azure/dn296411(v=azure.10).aspx
[LookupAsync]: https://msdn.microsoft.com/zh-cn/library/azure/jj871654(v=azure.10).aspx
[OrderBy]: https://msdn.microsoft.com/zh-cn/library/azure/dn250572(v=azure.10).aspx
[OrderByDescending]: https://msdn.microsoft.com/zh-cn/library/azure/dn250568(v=azure.10).aspx
[ReadAsync]: https://msdn.microsoft.com/zh-cn/library/azure/mt691741(v=azure.10).aspx
[Take]: https://msdn.microsoft.com/zh-cn/library/azure/dn250574(v=azure.10).aspx
[Select]: https://msdn.microsoft.com/zh-cn/library/azure/dn250569(v=azure.10).aspx
[Skip]: https://msdn.microsoft.com/zh-cn/library/azure/dn250573(v=azure.10).aspx
[UpdateAsync]: https://msdn.microsoft.com/zh-cn/library/azure/dn250536.(v=azure.10)aspx
[UserID]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.mobileservices.mobileserviceuser.userid(v=azure.10).aspx
[Where]: https://msdn.microsoft.com/zh-cn/library/azure/dn250579(v=azure.10).aspx
[Azure 门户]: https://portal.azure.cn/
[Azure 经典管理门户]: https://manage.windowsazure.cn/
[EnableQueryAttribute]: https://msdn.microsoft.com/zh-cn/library/system.web.http.odata.enablequeryattribute.aspx
[Guid.NewGuid]: https://msdn.microsoft.com/zh-cn/library/system.guid.newguid(v=vs.110).aspx
[ISupportIncrementalLoading]: http://msdn.microsoft.com/zh-cn/library/windows/apps/Hh701916.aspx
[Windows 开发人员中心]: https://dev.windows.com/overview
[DelegatingHandler]: https://msdn.microsoft.com/zh-cn/library/system.net.http.delegatinghandler(v=vs.110).aspx
[Windows Live SDK]: https://msdn.microsoft.com/zh-cn/library/bb404787.aspx
[PasswordVault]: http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.security.credentials.passwordvault.aspx
[ProtectedData]: http://msdn.microsoft.com/zh-cn/library/system.security.cryptography.protecteddata%28VS.95%29.aspx
[Notification Hubs APIs]: https://msdn.microsoft.com/zh-cn/library/azure/dn495101.aspx
[Mobile Apps Files Sample]: https://github.com/Azure-Samples/app-service-mobile-dotnet-todo-list-files
[LoggingHandler]: https://github.com/Azure-Samples/app-service-mobile-dotnet-todo-list-files/blob/master/src/client/MobileAppsFilesSample/Helpers/LoggingHandler.cs#L63
[Azure-Samples GitHub 存储库]: https://github.com/Azure-Samples

<!-- External URLs -->
[JsonPropertyAttribute]: http://www.newtonsoft.com/json/help/html/Properties_T_Newtonsoft_Json_JsonPropertyAttribute.htm
[OData v3 文档]: http://www.odata.org/documentation/odata-version-3-0/
[Fiddler]: http://www.telerik.com/fiddler
[Json.NET]: http://www.newtonsoft.com/json
[SymbolSource]: http://www.symbolsource.org/
[SymbolSource 说明]: http://www.symbolsource.org/Public/Wiki/Using

<!---HONumber=Mooncake_0919_2016-->