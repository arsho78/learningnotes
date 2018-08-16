## Chapter 1 XML概述 ##
XML(Extensible Markup Language, 可扩展标记语言)

一份XML文件至少应满足如下几个要求：

- 整个XML文档有且仅有一个根元素
- 每个元素都由开始标签和结束标签组成，除非使用空元素语法
- 元素与元素之间应该合理嵌套
- 元素的属性必须有属性值

MVC（Model，View，Control 模式，视图，控制器）

## Chapter 2 XML 文档规则 ##

- 格式不良好(malformed)的XML文档：完全没有遵守XML文档基本规则的XML文档
- 格式良好(well-formed)但无效的XML文档：遵守了XML文档基本规则，但没有使用DTD或Schema定义语义约束的XML文档，或者定义了约束但没有遵守语义约束的XML文档
- 有效(valid)的XML文档：遵守了XML文档基本规则，也定义并遵守了DTD和Schema语义约束

### XML文档的整体结构 ###

- 有且仅有一个根元素
	- DOM(Document Object Model)
- 元素必须合理结束
	- 所有开始标签都必须有配对的结束标签
	- 空元素必须严格使用空元素语法: <tagname />
	- XML的语法严格区分大小写
- 元素之间必须合理嵌套
- 元素的属性必须有值
	- 只有在子元素包含的内容完全是字符串的时候才能将子元素改写为属性
	- 所有属性必须有值，且每个属性值必须使用引号引起来，既可以是单引号，也可以是双引号，但必须成对使用
	- 元素的属性是无序不能重复的，相当于set，而子元素是有序可以重复的，相当于list

### XML声明 ###
声明不是必须的，但如果有，必须放在第一行，并且同时指定声明的version属性，encoding属性指定的编码必须和保存文件时使用的编码相同

	<?xml version="1.0" encoding="GB2312" standalone="yes"?>

### XML元素的基本规则 ###
#### 合法的标签名 ####

- 标签名可以由字母，数字，下划线，中划线，冒号和点号组成，但不能以数字，中划线和点号开始
- 标签名不能包含< > , $等符号
- 标签名中尽量不要使用冒号，除非是在使用命名空间
- 标签名不能以任何大小写的“xml”组合开始
- 标签名不能包含空格
- 标签名区分大小写，同一标签的开始签名和结束签名必须一致

#### 空元素 ####
<tagname />

空元素不能包含子元素，也不能包含字符串内容，但可以拥有属性

### 字符数据 ###

**特殊字符的显示**

- 使用实体引用
	- `&lt;` = `<`
	- `&gt;` = `>`
	- `&amp;` = `&`
	- `&apos;` = `'`
	- `&quot;` = `"`
- 使用CDATA标记

	`<![CDATA[文本内容]]>`

### 注释 ###
<!-- 注释字符串 -->

- 注释内可以包含元素和标签
- 不要把注释放在标签内使用
- 声明前不能有注释
- 不能在注释中使用双中划线
- 不能以--->结尾

### 处理指令 ###
    <?处理指令名 处理指令信息?>

### 标签属性的使用 ###

- 元数据（关于数据的数据）才应该存储为属性，而数据本身应该存储为元素
- 属性通常提供不属于数据组成部分的信息，如果信息属于实体本身，则使用子元素来描述
- XML元素的属性必须显示指定属性值，属性值必须由引号引起来
- 同一个XML元素里不能有重名的属性
- XML元素里的属性是无序的

**属性的缺点**

- 无法包含多个值
- 无法描述树状结构
- 不能扩展
- 难以阅读和维护

### 换行处理 ###

- Windows： CR(\r) && LF(\n)
- Unix & XML: LF(\n)
- Mac: CR(\r)


## DTD详解 ##

3种引入DTD方式：

- 内部DTD
- 外部DTD
- 公用DTD

**内部DTD**紧跟在XML声明和处理命令之后，以<!DOCTYPE开始，以]>结束

	<? xml version="1.0" encoding="GBK" standalone="yes" ?>
	<!DOCTYPE 根元素名[
		元素描述
	]>
	xml文档主体部分
	

**外部DTD**引用<!DOCTYPE开始，以>结束：

	<!DOCTYPE 根元素名 SYSTEM "外部DTD的uri">

**公用DTD**需要增加DTD标识名，并把`SYSTEM`替换为`PUBLIC`

	<!DOCTYPE 根元素名 PUBLIC "DTD的标识名" "公用DTD的URI">


## chapter 3 DTD详解 ##

- 内部DTD。紧跟在XML声明和处理指令之后，以`<!DOCTYPE`开始，以`]`结束。
- 外部DTD。`<!DODTYPE 根元素名 SYSTEM "外部DTD的URI">`。
- 公用DTD。`<!DODTYPE 根元素名 PUBLIC "DTD的标识名" "公用DTD的URI">`

DTD文档并不是xml文档。他的结构如下：

- 第一行是DTD声明部分，该声明与xml声明的语法相同。
- 0到多个注释部分，DTD注释与xml注释的语法完全相同。
- 0到多个<!ELEMENT...>定义，每个<!ELEMENT...>定义一个xml元素。
- 0到多个<!ATTLIST...>定义，每个<!ATTLIST...>为一个xml元素定义一个属性。
- 0到多个<!ENTITY...>定义，每个<!ENTITY...>定义一个xml实体。
- 0到多个<!NOTATION...>定义，每个<!NOTATION...>定义一个xml符号。

### 定义元素 ###
元素的类型可以是：

- 任意类型。`<!ELEMENT 元素名 ANY>`。 DTD必须定义xml文档中允许出现的所有元素。xml文档中只能出现在DTD中定义过的元素。应尽量避免使用ANY规则来定义xml元素。
- 空元素。`<!ELEMENT 元素名 EMPTY>`。
- 字符串内容。`<!ELEMENT 元素名 (#PCDATA)>`
- 混合内容。`<!ELEMENT 父元素名 (#PCDATA | 子元素1 | 子元素2 ...)*>`。
	- `#PCDATA`必须放在最前面。
	- `#PCDATA`和个子元素之间只能用竖线分隔，不要用逗号，也不要分组。
	- 不要在各子元素后添加?, \*, +等表示频率的修饰符。

### 定义子元素 ###

有序的子元素。子元素之间用英文逗号隔开。
互斥的子元素。子元素之间用竖线隔开。
子元素出现的频率：
`+`：子元素可以出现1次或多次。
`*`：子元素可以出现0次或多次。
`?`：子元素可以查询0次或1次。
无标志：必须出现且仅出现一次。

在DTD中定义无序子元素的方法：先将多个无序子元素定义为一个互斥的元素组，然后再使用`+`或者`*`修饰元素组。

### 定义元素属性 ###
`<!ATTLIST 属性所属的元素 属性名 属性类型 [元素对属性的约束] [默认值]>`

- 当没有指定`元素对属性的约束`时，等同于`#IMPLIED`，必须指定`默认值`
- 当`元素对属性的约束`是`#REQUIRED`时，不能指定`默认值`。必须为所属元素提供该属性
- 当`元素对属性的约束`是`#IMPLIED`时，不能指定`默认值`。该属性可有可无
- 当`元素对属性的约束`是`#FIXED`时，必须为该属性指定`默认值`，该属性值是固定的，由系统按`默认值`自动添加值，即使用户手动添加，也必须和`默认值`相同

属性的类型：

- CDATA             ： 字符串数据
- (en1 | en2 | en3) ： 枚举数据
- ID                ： 唯一标识符
- IDREF             ： 唯一标识符引用，属性值必须是一个已有的ID属性值
- IDREFS            ： 多个唯一标识符引用，以空格隔开
- NMTOKEN           ： 字符串数据，只能使用字母，数字，英文下划线，中划线，点号和冒号。
- NMTOKENS          ： 多个NMTOKEN值，以空格隔开
- ENTITY            ： 一个外部实体，例如图片文件
- ENTITIES          ： 多个外部实体，以空格隔开
- NOTATION          ： 在DTD中声明过的符号。尽量避免使用。
- xml:              ： 预定义的xml值
	
### 定义实体 ###
实体相当于字符串类型的具名变量。

		<!ENTITY 实体名 "实体值"> // 定义
		&实体名;                  // 使用
		
#### 参数实体 ####
参数实体只能在DTD文档中使用。必须前向定义，即先定义后使用。

		<!ENTITY % 实体名 "实体值"> // 定义
		%实体名;                    // 使用

#### 外部实体 ####
使用外部文件的内容替代实体名所在的位置。

		<!ENTITY 实体名 SYSTEM "实体值所在文件的URI"> // 定义
		&实体名;                                      // 使用

#### 外部参数实体 ####
使用外部文件的内容替代参数实体名所在的位置。 

		<!ENTITY % 实体名 PUBLIC|SYSTEM [公用实体标识名] "实体值所在文件的URI"> // 定义
		%实体名;                                                                // 使用

### 定义符号 ###

		<!NOTATION notation SYSTEM "value">
		<!NOTATION notation PUBLIC "name" "value">

符号值的类型：

- MIME类型
- 外部程序所在路径

符号的作用：

- 定义未解析实体
- 作为NOTATION类型的属性的值

当外部实体的文件满足以下条件时，该实体是可解析的外部实体：

- 外部文件是一个文本文件
- 该文件满足xml要求的结构化文档

引用其他的文件的实体都是未解析实体。不能直接使用，必须在实体的定义中使用NOTATION来指定处理的方法。此时NOTATION类似ENTITY或ENTITIES的属性。

		<!ENTITY % 实体名 PUBLIC|SYSTEM [公用实体标识名] "实体值所在文件的URI" NDATA notation>

使用ENTITY和ENTITIES作为属性值的类型时，这些实体必须是未解析实体，也就是说定义这些实体时，必须要指定他们的NOTATION属性值。

而NOTATION类型的属性值只能是符号名，并且在定义这些属性时必须用枚举的方法列出该属性所支持的值。

		<!ATTLIST 属性所属的元素 属性名 NOTATION (值1 | 值2) 约束 默认值>
		
## chapter 4 XML Schema 基本语法 ##
### XML Schema 根元素 ###
XML Schema 本身也是XML文档，根元素是 <schema.../>

XML Schema 根元素的属性：

- `xmlns[:xxx]="a namespace"` 为XML文档引入语义约束，属性值是引入的语义约束的命名空间，xxx是任意的标识名。
- `targetNamespace="a namespace"` 指定此Schema属于哪个命名空间。
- `xmlns="a namespace"` 指定不需要使用前缀的命名空间，可以直接使用该命名空间内的组件而不需要加前缀。只能有一个xmlns属性。注意：如果`targetNamespace`属性和`xmlns`的属性值不一样，则必须定义一个带前缀的`xmlns`属性指向`targetNamespace`，并在每个新定义的组件前加上该前缀。
- `elementFormDefault="qualified | unqualified"` 指定XML文档使用该Schema中定义的局部元素时是否必须使用命名空间限定。
- `attributeFormDefault="qualified | unqualified"` 指定XML文档使用该Schema中定义的局部属性时是否必须使用命名空间限定。

### 在XML中引用无命名空间的Schema ###
需要在根元素中加入以下两个属性值：

- `xmlns:xsi:` 该属性值总是"http://www.w3.org/2001/XMLSchema-instance"
- `xsi:noNamespaceSchemaLocation` 指定XML Schema文件的URI，既可以是URL地址，也可以是本地磁盘的相对路径。

### XML中引用有命名空间的Schema ###

- 每引入一个有命名空间的XML Schema就在XML文档的根元素里增加一个`xmlns[:xxx]`属性，注意只能有一个不带前缀的`xmlns`属性。
- 如果XML根元素中已有`xsi:schemaLocation`属性，则在该属性值后为该XML Schema追加一项，追加项保持 `schemaNamespace schemaURI` 的格式。如果XML根元素中还没有`xsi:schemaLocation`属性，则为其增加该属性，并设置属性值为`schemaNamespace schemaURI`。

一份XML文档中可以引入无数有命名空间的XML Schema，但最多只能引入一份无命名空间的XML Schema。

### Schema中的注释 ###
除了传统的<!-- -->注释方式，也可以使用<annotation.../>添加注释。

<annotation.../>还可以使用以下2个子元素

- <documentation.../>: 放置适合人阅读的信息。
- <appinfo.../>: 放置针对其他应用程序的信息。

<annotation.../>元素里可以出现任意多个<documentation.../>子元素，也可以出现任意多个<appinfo.../>子元素，而且没有任何顺序要求。

### Schema的数据类型 ###

- 简单类型：可以作为XML元素的类型，也可以作为XML属性的类型。
- 复杂类型：只能作为XML元素的类型。

Schema派生的方式：

- 限制：使用<restriction.../>元素为原有类型增加一个或多个额外的约束。
- 列表：使用<list.../>定义。
- 联合：使用<union.../>

内建基本类型，內建派生类型和用户通过限制派生出来的类型都只能包含单个值组件，因此这些数据类型统称为原子类型。

### Schema内置类型 ###

#### 字符串及相关类型 ####

- string：保留所有字符串内容，包括空白。
- normalizedString：将字符串内容中包含的换行，制表符和回车符都替换成空白。
- token：将字符串内容中包含的换行，制表符和回车符都替换成空白，删除字符串前后的空白，中间的空白压缩成单个空白。
- Name：字符串内容是一个合法的XML标签名，由字母，数字，下划线，中划线，冒号和点号组成，且不能以数字，中划线或点号开头。
- NCName：字符串内容是一个不带命名空间前缀的XML标签名。与Name类型的区别在于，NCName不能包含冒号。
- QName：字符串内容是一个带命名空间前缀的XML标签名，但他允许省略命名空间前缀，如果省略了之后，QName类型的值不能以冒号开头。如果使用了前缀，该前缀必须有相对应的命名空间。

#### 数值类型 ####
XML Schema里的float类型的值无须添加f或F后缀。当在XML中输入5.6时，既可以代表float类型，也可以代表double类型的值。

float和double类型可以有以下几个特殊值：

- -INF
- INF
- NaN
- +0
- -0

正零大于负零，NaN大于所有数值，INF大于其他所有浮点数

decimal类型类似Java中的BigDecimal类，表示精确小数。

- decimal更精确，可保证18位有效小数。
- decimal不支持使用科学计数法。
- decimal不支持-INF，INF和NaN等特殊值。

Schema中integer代表任意大的整数，int代表32位的有符号整数。

#### 日期，时间类型 ####

| 数据类型   | 格式                     | 说明                                                                                                                              |
| ---        | ---                      | ---                                                                                                                               |
| date       | YYYY-MM-DD               | 日期                                                                                                                              |
| time       | hh:mm:ss.sss             | 时间，最后的sss表示毫秒数                                                                                                         |
| dateTime   | YYYY-MM-DDThh:mm:ss.ssss | 日期时间，格式字符串中的T是必需的，是日期和时间的分隔符                                                                           |
| gYear      | YYYY                     | 年                                                                                                                                |
| gYearMonth | YYYY-MM                  | 年月，MM不能指定13                                                                                                                |
| gMonth     | --MM                     | 月，格式字符串中前面两个中划线是必需的                                                                                            |
| gMonthDay  | --MM-DD                  | 月日，格式字符串中前面两个中划线是必需的                                                                                          |
| gDay       | ---DD                    | 日，格式字符串中前面3个中划线是必需的                                                                                             |
| duration   | PnYnMnDTnHnMnS           | 格式字符串中的P是固定的，Y，M，D，H，M，S分别表示年，月，日，时，分，秒，其中Y，M，D，H，M前的n必须是整数，S前的n可以有小数部分。

#### boolean 类型 ####
boolean类型只能接受true，false，0或者1

#### anyURI 类型 ####
任何合法的URI值

#### 二进制数据 ####

- hexBinary： 0~9，a~f，A~F，长度必须是偶数。
- base64Binary：0~9，a~z，A~Z，长度必须是4的倍数。

### 使用限制派生新类型 ###
使用<simpleType.../>定义新的简单类型。使用<complexType.../>定义复杂类型。其属性如下：

- `id`：指定<simpleType.../>和<complexType.../>元素的唯一标识。
- `name`：新数据类型的名称。


使用<restriction.../>在基类型上添加限制从而产生新的类型。

- `id`：该restriction.../元素的唯一标识。
- `base`：指定该限制的基类型。

限制约束的种类：

- 枚举约束
	- enumeration：元素或者属性的值必须是枚举值中的一个。
- 精度约束
	- fractionDigits：对于任意精度的十进制书起作用，用于定义小数点后的最大位数。约束decimal派生出来的非decimal类型，如integer时，约束值只能是0.
	- totalDigits：decimal及其派生类型的数值最多能有几位数（包括整数和小数部分）。如果与fractionDigits同时使用，则totalDigits的值必须大于等于fractionDigits的值。
- 长度约束。约束基于string的数据类型，QName，anyURI和二进制数据的字符串的长度，还可以用于约束列表类型列表项的数量。
	- length：元素或者属性值的字符长度。
	- minLength：字符串的最小长度。
	- maxLength：字符串的最大长度。
- 范围约束
	- minExclusive：元素或属性的下限值（不包括）。
	- maxExclusive：元素或属性的上限值（不包括）。必须大于minExclusive的值。
	- minInclusive：元素或属性的下限值（包括）。
	- maxInclusive：元素或属性的上限值（包括）。必须大于minInclusive的值。
- 正则表达式约束
	- pattern：数据类型的值必须匹配的正则表达式。
- 空白处理。
	- whiteSpace：字符串中的空白的处理方式。可以是：
		- preserve：保留字符串中的空白。
		- replace：将所有的制表符，换行符和回车符用空格代替。
		- collapse：先执行replace，再删除开头和结尾的空格，并将中间的空格压缩成一个。对除string和normalizedString两种数据类型之外的数据类型添加该约束时，约束值只能是collapse。
	
#### 指定类型的两种方式 ####

- 使用元素或者属性的Type属性指定他们的类型。
- 使用<simpleType.../>或者<complexType.../>子元素指定元素或属性的类型。

在XML Schema的根元素下定义的任何元素都可以作为XML文档的根元素使用。

- 全局的有名字的数据类型：<simpleType.../>和<complexType.../>元素直接作为<schema.../>元素的子元素使用，此时需要为这两个元素指定他们的`name`属性，其值就是新定义的数据类型的名称。
- 局部的匿名数据类型：<simpleType.../>和<complexType.../>作为<element.../>，<attribute.../>，<restriction.../>元素的子元素使用，其作用域仅限于所处的上级元素内。

### 使用<list.../>派生列表类型 ###
列表类型的值以空白作为分隔符。对多个空格的处理于token类型相同。

指定列表内部数据类型的方式也有两种：

- 为<list.../>元素的itemType属性指定列表元素使用已定义的全局数据类型。
- 为<list.../>元素增加一个<simpleType.../>子元素来使用这个新定义的局部数据类型。

列表类型可以使用如下限制约束：

- 长度约束：length，minLength，maxLength
- 枚举约束：enumeration
- 正则表达式约束：pattern
- 空白处理：whiteSpace。其值只能是collapse。

**对列表类型使用enumeration和pattern约束时，是对列表内容的整体（多个列表项和空格组成的整体）起作用，而不是对单个的列表元素起作用。**

### 使用<union.../>派生联合类型 ###
使用<union.../>元素创建联合类型时，需要指定一个到多个简单类型，为<union.../>指定类型有两种方式：

为<union.../>元素的memberTypes属性指定一个到多个简单类型，多个简单类型之间以空格隔开。
为<union.../>元素增加一个到多个<simpleType.../>子元素，每个<simpleType.../>子元素指定一个简单类型。

联合类型可以使用枚举约束enumeration和正则表达式约束pattern。也是对整个联合类型的值起作用。

联合类型的成员类型可以是原子类型，列表类型，也可以是联合类型。
列表类型的元素的类型可以是原子类型，联合类型，但不可以是列表类型，也不可以是任何含有列表类型的混合类型。

### 阻止派生新的简单类型 ###

使用<simpleType.../>元素指定final属性来限制派生新的类型:

- `#all`：限制该类型以任何形式派生新的类型。
- restriction，list，union的组合
- ""：默认方式

可以为Schema文档的根元素<schema.../>指定finalDefault属性，将对Schema中所有的类型起作用。

Schema还允许为任何约束指定fixed属性，属性的值只能是true或false。在使用一个已有类型派生新的类型时，如果新类型添加了和原类型相同的约束，且约束名相同，则新的约束值会覆盖原有的约束值。但如果原类型对应约束指定了fixed="true"，则不能被覆盖。

### 合并多个Schema ###

#### 使用include元素 ####

- <include.../>元素必须作为Schema文档的根元素<schema.../>的子元素。
- <include.../>元素必须放在<schema.../>元素的开头，只有<import.../>，<redefine.../>和<annotation.../>元素可以放在<include.../>元素之前。
- 使用<include.../>元素包含的Schema要么没有目标命名空间，要么其目标命名空间与当前Schema的目标命名空间相同。也就是说，使用<include.../>元素包含进来的Schema将不再保留他本身的命名空间，被包含Schema所定义的全部组件将放在当前Schema的命名空间下。

2个属性值：

- id：include元素的唯一标识，通常无须指定。
- schemaLocation：必填。<include.../>元素包含的Schema的位置，可以是相对或绝对位置。

#### 使用redefine元素 ####
<redefine.../>元素是<include.../>元素的增强版。允许当前Schema重定义被包含Schema中的Schema组件，包括重定义类型，元素组和属性组等。

重定义组件是有以下3个要求：

- 重定义的组件必须是被redefine包含进来的Schema里已有的组件。
- 重定义的组件只能基于被包含Schema里已有的组件增加限制或增加扩展。
- 如果使用增加限制的方式，则<restriction.../>元素里所包含的约束不能违反原类型里已有的约束。

#### 使用import元素 ####

- <import.../>元素必须作为Schema文档的根元素<schema.../>的子元素。
- <import.../>元素必须放在<schema.../>元素的开头，只有<include.../>，<redefine.../>和<annotation.../>元素可以放在<import.../>元素之前。
- 使用<import.../>元素包含的schema要么没有目标命名空间，要么其目标命名空间与当前Schema的目标命名空间不同，并且不能同时没有目标命名空间。也就是说，使用<import.../>元素包含进来的Schema与当前Schema的目标命名空间绝对不能相同。使用被导入的Schema的组件时必须带有前缀作为限定，除非他没有命名空间。

import元素的3个属性：

- id：<import.../>元素的唯一标识，通常无须指定。
- schemaLocation：指定该<import.../>元素包含的Schema的位置，既可以是相对位置，也可以是绝对位置。
- namespace：指定被导入Schema的目标命名空间，与当前Schema根元素中指定使用的命名空间相对应。如果被导入Schema没有目标命名空间，则该属性可以省略。
