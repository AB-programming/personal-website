---
external: false
title: .NET模板的使用
description: .NET的模板是一种命令生成项目和文件的机制，可以快速创建特定类型的项目和文件，使用 `dotnet new` 命令实现模板的创建和使用。
date: 2026-05-04
---

# .NET模板介绍

.NET的模板是一种命令生成项目和文件的机制，可以快速创建特定类型的项目和文件，使用 `dotnet new` 命令实现模板的创建和使用。

使用 `dotnet new list` 命令可以查看已经安装的模板，dotnet有一些内置的模板，例如webapi、blazor等。

.NET的模板还可以帮助我们将自己常用的项目结构、可复用的功能安装为模板，直接使用。

# 创建和使用自定义模板

以一个webapi项目为例，这里集成了授权认证、数据库（SQL Server）等。
在项目目录下新建.template.config目录，在.template.config目录下创建template.json文件，template.json内容如下：
```json
{  
  "$schema": "http://json.schemastore.org/template",  
  "author": "AB-programming",  
  "classifications": [  
    "Web/API"  
  ],  
  "name": "Dotnet8-Api-Template",  
  "identity": "MyCompany.MyApiTemplate",  
  "shortName": "dotnet8-api-template",  
  "tags": {  
    "language": "C#",  
    "type": "project"  
  },  
  "sourceName": "Dotnet8ApiTemplate",  
  "preferNameDirectory": true,  
  "symbols": {  
    "DbCatalog": {  
      "type": "parameter",  
      "description": "Database name (Initial Catalog)",  
      "datatype": "text",  
      "replaces": "$catalog$",  
      "defaultValue": "dotnet8-api-template",  
      "isRequired": true  
    },  
    "DbUserId": {  
      "type": "parameter",  
      "description": "Database user ID",  
      "datatype": "text",  
      "replaces": "$userid$",  
      "defaultValue": "sa",  
      "isRequired": true  
    },  
    "DbPassword": {  
      "type": "parameter",  
      "description": "Database password",  
      "datatype": "text",  
      "replaces": "$password$",  
      "defaultValue": "123456",  
      "isRequired": true  
    }  
  }}
```

name和shortName是模板名和缩短名称，使用模板时指定后者，sourceName指定了项目源码中的替换名称，可用于替换命名空间等，此例源码中Dotnet8ApiTemplate都会被替换为使用模板时指定的名称。
symbols中指定了要替换的配置信息，例如配置文件中需要配置数据库连接信息，指定了 `Data Source=.;Initial Catalog=$catalog$; User ID=$userid$;Password=$password$;TrustServerCertificate=True`，在template.json的symbols中配置了这些，可以在使用此模板时指定参数，如 `--DbUserId=sa` 就将配置文件中userid替换成了sa。

在项目根目录下执行 `dotnet new install .` 即可将模板安装到.NET中。如需卸载可执行 `dotnet new uninstall 项目位置`

在创建新项目的位置，执行
```shell
dotnet new dotnet8-api-template -n Example --DbCatalog=example --DbUserId=sa --DbPassword=12345678
```

就可以创建一个基于刚刚自定义模板的项目，dotnet8-api-template是模板的缩短名称，`-n` 后的参数是项目名称，创建后模板项目中所有的Dotnet8ApiTemplate都会被替换成Example。

# 相关资源
模板示例地址：[自定义模板示例](https://github.com/AB-programming/dotnet8-api-template)
