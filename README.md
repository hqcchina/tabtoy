# tabtoy

游戏客户端,服务器的策划表格数据导出

![tabtoylogo](doc/logo.png)

# 版本
* 2019年10月: Fork原项目进行简化改造

# 优点

* 编写电子表格, 导出. 只需2步, 即可导出数据!

* 跨平台运行, 无第三方依赖

* 支持多种数据文件格式导出：csv, json

* 支持多种代码文件格式导出: golang, C#, lua

* 单元格字段列顺序随意调整, 自动检查错误, 精确报错位置

* 强类型, 导出时自动类型检查, 提前暴露表格错误

* 支持纵向填写字段作为配置, 将电子表格作为快速配置文件

* 支持导出Tag匹配,导出需要的部分, 避免客户端混合服务器私密数据

* 充分利用CPU多核进行导出

# 使用手册

![电子表格](doc/table_v2.png)
功能强大的的导出表格式

![电子表格](doc/log_v2.png)
详细的输出日志及报错信息,以及精准到单元格的多语言报错

## 准备电子表格文件

格式请参考:
	
	https://github.com/davyxu/tabtoy/blob/master/v2/example/Sample.xlsx
	
## 准备tabtoy二进制
	
* 已经编译好的二进制:
	
	[https://github.com/huangqingcheng/tabtoy/releases](https://github.com/huangqingcheng/tabtoy/releases)
	
* 手动编译获取最新版
	
	go get github.com/huangqingcheng/tabtoy
	
## 编写导出命令行

范例:
		
```bat

tabtoy --mode=v2 --json_out=config.json --combinename=Config Table.xlsx

```
* 注意: 不要将这个命令行指令对例子表格进行导出, 例子表格包含类型信息, 需要多个表格组合导出

## Golang读取例子

[例子](https://github.com/davyxu/tabtoy/tree/master/v2/example/golang)
	
```golang
	config := table.NewConfigTable()

	if err := config.Load("Config.json"); err != nil {
		panic(err)
	}

	for index, v := range config.SampleByID {
		fmt.Println(index, v)
	}
```
	
	

## C#读取例子
	
	[例子](https://github.com/davyxu/tabtoy/tree/master/v2/example/csharp)

```csharp
   using (var stream = new FileStream("../../Config.bin", FileMode.Open))
   {
       stream.Position = 0;

       var reader = new tabtoy.DataReader(stream);

       var config = new table.Config();

       var result = reader.ReadHeader(config.GetBuildID());
       if ( result != FileState.OK)
       {
           Console.WriteLine("combine file crack!");
           return;
       }


       table.Config.Deserialize(config, reader);

       // 直接通过下标获取或遍历
       var directFetch = config.Sample[2];

       // 添加日志输出或自定义输出
       config.TableLogger.AddTarget(new tabtoy.DebuggerTarget());

       // 取空时, 当默认值不为空时, 输出日志
       var nullFetchOutLog = config.GetSampleByID(0);

   }
```

## lua读取例子

[例子](https://github.com/davyxu/tabtoy/tree/master/v2/example/lua)

```lua
-- 添加搜索路径
package.path = package.path .. ";../?.lua"

-- 加载
local t = require "Config"

-- 直接访问原始数据
print(t.Sample[1].Name)

-- 通过索引访问
print(t.SampleByID[103].ID)

print(t.SampleByName["黑猫警长"].ID)
```


## 更多示例
	
[例子](https://github.com/davyxu/tabtoy/blob/master/v2/example)

* 注意: 例子中展现的是一般项目中多表的使用方法

	共享的类型信息会被统一放在Globals表中, 最终导出时, 需要将Globals和其他表一起配合导出


## 详细文档

[文档](https://github.com/davyxu/tabtoy/blob/master/doc/Manual_V2.md)

[错误描述](https://github.com/davyxu/tabtoy/blob/master/doc/error_v2.md)


## 功能扩展

### 多表合并导出
一些表格, 如角色, 道具的表格, 会将角色或道具充当不同的角色和功能, 例如: 道具作为装备
道具作为宠物,  boss配置, 怪物配置, npc配置等等

tabtoy支持按功能分类后的表格, 导出时保持一致的表头及类型, 方便策划拆表进行多人协作

[例子](https://github.com/davyxu/tabtoy/tree/master/v2/example/combine)

```bat

tabtoy --mode=v2 --json_out=CombineConfig.json --combinename=Config Item.xlsx+Item_Equip.xlsx+Item_Pet.xlsx

```

需要合并的同类表格间使用'+'互相连接


### 导出Tag匹配
如果客户端使用C#并读取二进制导出数据, 服务器使用golang开发读取json

* 我们不希望新手引导的表导入到服务器的配置文件中, 可以这么做:

	在新手引导表的@Types表单里添加OutputTag: [".cs", ".bin"]

* 我们不希望服务器的ip配置表导入到客户端的配置文件中, 可以这么做:

	在ip配置表的@Types表单里添加OutputTag: [".go", ".json"]

减少数据冗余和保证客户端数据安全, tabtoy已经为你考虑

### 支持纵向导出, 用于配置表
![电子表格](doc/vertical_v2.png)

[例子](https://github.com/davyxu/tabtoy/tree/master/v2/example/verticalconfig)


# FAQ

## 能够将某些列隐藏不导出?

在列名字前加#即可, 例如: ID 一列不导出, 将第一行的ID列改为#ID即可

同样的, Sheet表单上也支持#,带#开头的表单不会被导出

P.S. 不要将@Types表单加#

## 不同的表能生成指定不同语言的源码么?
比如: A表生成A类, B表生成B类分别使用.
不支持, tabtoy拥有统一的类型系统和数据分析设计,虽然在一个工程里可以手动导出2次形成2个类使用,但会遇到很多麻烦, 比如:类型冲突.
所以, tabtoy提倡在一个工程里一次将表导出, 表只有1个入口. 客户端和服务器可以分别生成不同的入口.


# 备注

感谢davyxu
