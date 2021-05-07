### 迁移整改方案步骤 ###


----------

# 一、类库项目

>类库项目说明

此项目常用于被引用的类库，公共组件，公共类，新建此类项目以.net standard 2.0版本为标准依赖库，可同时兼容最低[.net framework 4.6.1](https://dotnet.microsoft.com/download/dotnet-framework/net461) 以及 [.netcore 3.1 ](https://dotnet.microsoft.com/download/dotnet/3.1) 兼容性列表如下图。

<img width="500" src="img/net_standard_version.png">

具体依赖可参考 [此处](https://docs.microsoft.com/zh-cn/dotnet/standard/net-standard)在我们的项目中，一般常见以MDB开头的项目，例如 **MDB** 、**MDB.CMC** 、**MDB.Customer**、 **MDB.FSS**、 **MDB.Goose**、 **MDB.Log**、  **MDB.UCC** 等。

> 快速启动

- ①、新建类库项目，这里我们使用了 *C#* + *Linux* + *库* 作为筛选项，在以下列出的列表中，需要带有Linux与Windows标签，描述为： **一个用于创建面向，.NET Standard 或 .NET Core 的类库项目**。

<img width="500" src="img/create_lib.png">


- ②、选择.NET Standard版本为2.0即可，兼容性最好。

<img width="500" src="img/config_new_project_for_lib.png">

- ③、按需从Nuget中引入MDB类库的包，选择mdbcore

此处就创建完成了，升级包体以及兼容性列表会在第三章节中具体描述

# 二、Web项目


- ①、这里我们使用了 *C#* + *Linux* + *Web* 作为筛选项，在以下列出的列表中，需要带有Linux与Windows标签，描述为： **用于创建包含RESTful HTTP服务器实力控制器的ASP.NET Core引用程序的模板。**

<img width="500" src="img/create_web_api.png">

*使用此模板的好处在于无需自己添加Controller文件夹与startup启动类，更加方便直接上手修改项目，当然，对于更强定制的项目可以使用空模板来创建，但并不在此文的讨论范畴内*

- ②、引入Web专属鉴权类
[ApiController](cs/ApiController.cs)、
[ApiAuthorizeAttribute](cs/ApiAuthorizeAttribute.cs)、
[AuthorizationManager](cs/AuthorizationManager.cs)、
[ErrorCodes](cs/ErrorCodes.cs)、
[ResponseResult](cs/ResponseResult.cs)、到web项目的根目录或web项目中任意能访问到的地方。

<img width="300" src="img/setting_property_1.png">
 

- ③、在控制器中 继承 ApiController 类

		using Microsoft.AspNetCore.Mvc;
		
		namespace MDB.NewWeb.Controllers
		{
		    public class HomeController : ApiController
		    {
		        public JsonResult Index()
		        {
		            object obj = new object();
		            return Json(obj);
		        }
		    }
		}



- ④、再引入 [App.config](cs/App.config)到**根目录**下、其中app.config并不是所谓的配置文件，而是为了兼容老版本的携程配置文件app.config而存在，引入在项目根目录 **解决方案下**，并将属性设置为**始终复制**。

<img width="500" src="img/setting_property.png">

- ⑤、配置 [App.config](cs/App.config) 相较于之前版本有些许字段差别，但具体差别不大，多多注意**Apollo.AppId**与**Apollo.DEV.Meta**。


    	<?xml version="1.0" encoding="utf-8"?>
    	<configuration>
    		<appSettings>
    			<add key="Env" value="DEV" />
    			<add key="Apollo.AppId" value="B100008" />
    			<add key="Apollo.DEV.Meta" value="http://192.168.5.219:8080/" />
    			<!--单位为分钟/多少分钟拉去一次，默认毫秒-->
    			<add key="Apollo.RefreshInterval" value="120000" />
    			<!--超时时间，单位未知-->
    			<add key="Apollo.Timeout" value="2000" />
    			<add key="Cachefolder" value="C:/opt/data" />
    		</appSettings>
    	</configuration>
    




- 快速创建结束：到此处基本的项目创建部分就已经完成了，且可以使用mdb的类库进行操作，下章节主要是类库版本的变化与需要注意的几个点


# 三、类库版本变化与参考的修改方式经验

### ①、cookie获取与设置的变化

*请注意，以下更新若无涉及到则无需更改，前二章节已涵盖了创建步骤的基本操作，以下只作为dotnet升级为dotnetcore版本的提示信息，简易修改方式与作者在mdb中总结的一些经验*

> 更新前

	string cookieValue = "";
	HttpCookie cookie = HttpContext.Current.Request.Cookies[_name];
	if (cookie != null)
	{
		cookie.Domain = _domain;
		if (cookie.Value != null)
		{
			cookieValue = cookie.Value;
		}
	}



> 更新后

	string cookies = _httpContext.Request.Cookies[_name];

### ②、常用的web函数的变化

> 更新前使用该函数来获取当前请求上下文

	var Request = HttpContext.Current.Request;

> 更新后则使用依赖注入或在ApiController中传入 HttpContext 来供第三方库操作当前请求上下文

 	public SSOCookieProvider(HttpContext context)
    {
       this._httpContext = context;
	}


### ③、常用的类库之间的变化
请注意，所有类库的版本已做过调试，且向下兼容，创建项目后使用以下组建的需要 **绝对对应** 切勿更改版本或使用错误的版本产生不必要的问题，如果出现一对多的关系，则是之前的项目被第三方发布的新版拆分为了两个项目。

|操作|原类库| 原版本| 现类库 | 现版本|说明
|----|----|----|----|----|----|
|更新| Apollo_Config_Clien   | 1.0.0| Com.Ctrip.Framework.Apollo | 1.5.3|携程库
|更新| Apollo_Config_Clien   | 1.0.0| Com.Ctrip.Framework.Apollo.Configuration.Manager | 1.5.5|携程扩展库
|更新| Qiniu|7.2.15.0|Qiniu.Net|8.0.0|七牛库
|更新|StackExchange.Redis|1.2.0|StackExchange.Redis|2.2.4|redis库
|更新|Newtonsoft.Json|11.0.0.0|Newtonsoft.Json|13.0.1|json库
|删除|Microsoft.VisualBasic|10.0.0.0|CHTCHSConv|1.0.0|繁简体转换库
|更新|HtmlAgilityPack|1.0.14|HtmlAgilityPack|1.11.30.0|解析html
|更新|NPOI|2.5.2.0|NPOI|2.5.2|EXCEL操作类
|更新|ICSharpCode.SharpZipLib|1.3.1.9|SharpZipLib|1.3.1|Zip文件使用
|更新|Gma.QrCodeNet.Encoding|0.4.0.0|Gma.QrCodeNet.Encoding|2.0.0|二维码
|更名|BouncyCastle.Crypto|1.8.6.0|Portable.BouncyCastle|1.8.10|加解密

### 四、其他补充

####一、中繁转换的变更代码

`使用的类库为 CHTCHSConv.dll`

> 中繁转换之前

`_wordList.Add(Microsoft.VisualBasic.Strings.StrConv(key,Microsoft.VisualBasic.VbStrConv.TraditionalChinese, 0));`

> 中繁转换之后

`ChineseConverter.Convert(key, ChineseConversionDirection.SimplifiedToTraditional);`

####二、需要安装的类库和建议的工具与学习地址
[dotnet core3.1下载地址](https://dotnet.microsoft.com/download/dotnet/3.1)
<br/>
[visualstudio2019下载地址](https://visualstudio.microsoft.com/zh-hans/downloads/)
<br/>
[dotnetcore教程](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/razor-pages-start?view=aspnetcore-5.0&tabs=visual-studio)

###三、docker 部分

>使用docker可直接在项目中右键创建Docker支持

<img width="500" src="img/create_docker.png">

*创建后的Docker文件如下*

> DockerFile参考

	
	FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
	WORKDIR /app
	EXPOSE 80
	
	FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
	WORKDIR /src
	COPY ["AssetCenter.Api/AssetCenter.Api.csproj", "AssetCenter.Api/"]
	COPY ["AssetCenter.BLL/AssetCenter.BLL.csproj", "AssetCenter.BLL/"]
	COPY ["mdb/MDB.Account/MDB.Customer.csproj", "mdb/MDB.Account/"]
	COPY ["mdb/MDB.CMC/MDB.CMC.csproj", "mdb/MDB.CMC/"]
	COPY ["mdb/MDB/MDB.csproj", "mdb/MDB/"]
	COPY ["mdb/MDB.UCC/MDB.UCC.csproj", "mdb/MDB.UCC/"]
	COPY ["mdb/MDB.Log/MDB.Log.csproj", "mdb/MDB.Log/"]
	COPY ["mdb/MDB.Goose/MDB.Goose.csproj", "mdb/MDB.Goose/"]
	COPY ["mdb/MDB.FSS/MDB.FSS.csproj", "mdb/MDB.FSS/"]
	COPY ["AssetCenter.Common/AssetCenter.Common.csproj", "AssetCenter.Common/"]
	COPY ["mdb/MDB.OperLog/MDB.OperLog.csproj", "mdb/MDB.OperLog/"]
	RUN dotnet restore "AssetCenter.Api/AssetCenter.Api.csproj"
	COPY . .
	WORKDIR "/src/AssetCenter.Api"
	RUN dotnet build "AssetCenter.Api.csproj" -c Release -o /app/build
	
	FROM build AS publish
	RUN dotnet publish "AssetCenter.Api.csproj" -c Release -o /app/publish
	
	FROM base AS final
	WORKDIR /app
	COPY --from=publish /app/publish .
	ENTRYPOINT ["dotnet", "AssetCenter.Api.dll"]

------

