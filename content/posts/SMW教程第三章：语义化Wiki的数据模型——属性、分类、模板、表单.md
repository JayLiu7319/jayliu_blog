+++
date = '2023-11-04T14:17:02+08:00'
draft = false
title = 'SMW教程第三章：语义化Wiki的数据模型——属性、分类、模板、表单'
featured_image = 'https://opengraph.githubassets.com/04dfd6c68d3e4474ec57360ec23f051e8ff9f3d4f1c94786364e7ef9645b374c/SemanticMediaWiki/SemanticResultFormats'
description = '本文介绍了以Semantic Mediawiki作为知识图谱底座的基本数据模型，以及数据模型在MediaWiki上的实现方式'
categories = '教程'
+++

## 一、语义化 Wiki 的数据模型

![PF数据结构.drawio](https://images.jayliu.tech/blog/2024/11/91729a7dd8afc9a81366490e75ae5888.png)



上图非常重要，它说明了在我们的**语义化框架**中所使用的数据模型，一定要深刻理解；在上一讲中，我们为大家介绍了 MediaWiki 页面 (即图中绿色框) 的基本编辑与操作，这也是传统 Wiki 的关键内容；而在该讲，我们将会为大家介绍**语义化的 Wiki 平台**所涉及到的其他一系列概念，如分类、页面、属性、模板、表单等。

那么什么是语义化，为什么要对 wiki 平台进行语义化呢？

**下图为维基百科的天津大学页面**：

![1674806946729](https://images.jayliu.tech/blog/2024/11/1f78df216e59239c4900fd256223bdcb.png)



在传统的 wiki 网站中 (如维基百科)，对于**结构化数据**的定义、存储、查询并没有一套完整成熟的机制。例如维基百科中 “**天津大学**” 这个**实体**，它的结构化数据在页面上显示在右侧信息框中 (我们也称其为 infobox)，这个信息框是由编辑者**手动维护**的。对于维基百科上大量页面的编辑和维护，绝大多数都依赖志愿者的自发组织；且由于数据量越来越大，内容越来越复杂，对于页面的编辑和维护已经不再像我们上一节那样简单易行。



![image-20230127191506689](https://images.jayliu.tech/blog/2024/11/72bddbbf392d655ac01f55a0af43636d.png)



只是简单的一页百科页面，却要涉及到 **121 个模板**，以及**大量的 Lua 脚本**，这带来了大量的学习成本和高昂的维护代价。

**最最致命的问题是**：在原始的 MediaWiki 中，所有的数据都以文本形式存储在关系数据库中，即使一个页面中存在大量的结构化数据 (如天津大学的创建时间、校长、校训等)，也无法被识别与处理。举个例子：想要在维基百科中查询**全世界所有在1890年到1995年间建立的大学**，是无法做到的，因为维基百科的**关系型数据库实际上是囫囵吞枣，只存储了页面全部的文本数据，但无法分辨页面中文本数据的意义**。

因此，为了**定义、存储、检索**页面中的结构化数据，实现语义化的查询，我们就不得不引入新的方案，来进一步设置页面本身所具有的**属性**、**分类**等结构化信息，并以某种形式存储到数据库中，最后使用一种方案进行语义化的查询检索。而更近一步，我们可以将**语义化的数据**与**知识图谱的基本框架**对应，创建我们自己的知识图谱。

总的来说，要解决页面语义化和结构化数据的问题，就需要解决语义数据的**定义、存储、检索**这三大问题

- 页面结构化数据 (或语义数据) 的**定义**：Page Forms
- 页面结构化数据 (或语义数据) 的**存储**：Semantic MediaWiki 或 Cargo
- 页面结构化数据 (或语义数据) 的**检索**：Semantic MediaWiki 或 Cargo

在我们的教程中，我们使用 Page Forms 和 Semantic MediaWiki 作为我们的语义化方案和实现知识图谱的基础：

- 每个**分类**都视作一个**本体**；分类与子分类的树形结构，代表了本体树的树形结构；
- 本体中的每一个**页面**，代表了知识图谱中与本体对应的一个单独的**实体**；
- 每一个实体有其自己的非页面型**属性字段与对应的值**，作为 (实体)-[属性]-(属性值) **三元组**；
- 每一个实体有其自己的页面型**属性字段与对应的页面**，作为 (实体)-[关系]-(实体) **三元组**；
- 实体与实体之间以**页面型属性字段相互关联**；

这里出现了许多新的概念，我们在下面给出详细说明；另外我们将在下一讲为同学们详细解释知识图谱的概念，以及它们在 MediaWiki 上的实现方法。

## 二、属性

> 该特性由 **Semantic MediaWiki** 与 **Page Forms** 共同提供

”属性 “为 MediaWiki 的页面添加了更细致的数据管理；**属性 (properties) 及类型 (types)** 是 [Semantic MediaWiki](https://www.semantic-mediawiki.org/wiki/Semantic_MediaWiki) 中定义结构化数据 (或语义数据) 的基本方式。可以将属性**视为对页面 (或实体) 的一系列规范化的描述词**；

在页面编辑中，我们可以使用一些简单的方式来定义当前页面的属性，比如我们在”天津大学 “页面编辑了一段文字：

~~~wiki
天津大学（Tianjin University），创办于1895年，简称 “天大” ，坐落于天津市，是中华人民共和国教育部直属的首批全国重点大学，是国家“双一流”建设高校、 国家“211工程”和“985工程”建设高校，中国工程院和教育部10所工程教育改革试点高校之一，与南开大学为兄弟院校……
~~~

对其进行语义标注 (我们会在后面详细介绍) 后，可以得到：

~~~wiki
天津大学（Tianjin University），创办于[[创办时间::1895]]年，简称 “天大” ，坐落于[[学校位置::39°6'28"N, 117°10'22"E|天津市]]，是中华人民共和国教育部直属的首批全国重点大学，是国家“双一流”建设高校、 国家“211工程”和“985工程”建设高校，中国工程院和教育部10所工程教育改革试点高校之一，与[[兄弟院校::南开大学]]为兄弟院校……
~~~

点击保存，然后在左侧工具栏选择 “**浏览属性** “(这里以经典皮肤为例做演示，其他皮肤也有对应的工具，请自行查找其位置，或在特殊页面中选择” 浏览属性 “)：

![image-20230127191749375](https://images.jayliu.tech/blog/2024/11/ad6d2f6527db1bc5f055be4f155673c9.png)

可以看到当前页面定义了我们刚刚建好的一些三个属性：**创办时间、学校位置、兄弟院校**。

![image-20230127191844710](https://images.jayliu.tech/blog/2024/11/24e50802c30c22ad4a31e3493ac581d0.png)

通过这种方式，我们就能简单实现对天津大学这一实体的结构化数据的创建与语义化。

### 1、属性类型

我们在上文中为天津大学添加了三个属性：**创办时间、学校位置、兄弟院校**。而实际上，这三个属性对应的值 (内容)，各有其不同的要求：

- **兄弟院校**对应的值，应当是与大学分类下的**其他页面实体**，如页面 “南开大学”
- **学校位置**对应的值，应当是一个格式正确的**地理坐标**，如 39°6'28 “N，117°10'22” E
- **创办时间**对应的值，应当是一个格式正确的**日期时间**，如 1895

实际上，我们还可以很多创建其他的属性，如 “校训”、“校长”、“教职工人数” 等等，每一个属性对应的值，都会有一定的要求和规范。因此，属性本身也有不同的 “类型”，这里给出 MediaWiki 的属性类型：

|                       属性类型（英文）                       | 属性类型（中文） |                           属性例子                           |                           数值例子                           |
| :----------------------------------------------------------: | ---------------- | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    [Page](https://www.huijiwiki.com/wiki/特殊:类型/Page)     | **页面型**       | [Property:Has example](https://www.huijiwiki.com/wiki/属性:Has_example) | [Semantic MediaWiki](https://www.huijiwiki.com/index.php?title=Semantic_MediaWiki&action=edit&redlink=1) |
|    [Text](https://www.huijiwiki.com/wiki/特殊:类型/Text)     | **文本型**       | [Property:SomeProperty](https://www.huijiwiki.com/wiki/属性:SomeProperty) |        Did you create the page for Tokyo 東京 ? Yes ✓        |
| [Annotation URI](https://www.huijiwiki.com/wiki/特殊:类型/Annotation_URI) | 注释URI型        | [Property:Has annotation uri](https://www.huijiwiki.com/wiki/属性:Has_annotation_uri) | [http://www.semantic-mediawiki.org](http://www.semantic-mediawiki.org/) |
| [Boolean](https://www.huijiwiki.com/wiki/特殊:类型/Boolean)  | 布尔型           | [Property:Has boolean](https://www.huijiwiki.com/wiki/属性:Has_boolean) |                             true                             |
|    [Code](https://www.huijiwiki.com/wiki/特殊:类型/Code)     | 代码型           | [Property:Has code](https://www.huijiwiki.com/wiki/属性:Has_code) |                                                              |
|    [Date](https://www.huijiwiki.com/wiki/特殊:类型/Date)     | 日期型           | [Property:Has date](https://www.huijiwiki.com/wiki/属性:Has_date) |                       2015-05-22T17:32                       |
|   [EMail](https://www.huijiwiki.com/wiki/特殊:类型/EMail)    | 电子邮件地址型   | [Property:Has email address](https://www.huijiwiki.com/wiki/属性:Has_email_address) | [president@whitehouse.gov](mailto:president@whitehouse.gov)  |
| [Geographic coordinate](https://www.huijiwiki.com/wiki/特殊:类型/Geographic_coordinates) | 地理坐标型       | [Property:Has coordinates](https://www.huijiwiki.com/wiki/属性:Has_coordinates) |                   32°42′54″ N, 117°9′45″ W                   |
|  [Number](https://www.huijiwiki.com/wiki/特殊:类型/Number)   | 数值型           | [Property:Has number](https://www.huijiwiki.com/wiki/属性:Has_number) |                           47000.11                           |
| [Quantity](https://www.huijiwiki.com/wiki/特殊:类型/Quantity) | 数量型           |  [Property:Area](https://www.huijiwiki.com/wiki/属性:Area)   |                           1052km²                            |
|  [Record](https://www.huijiwiki.com/wiki/特殊:类型/Record)   | 记录型           | [Property:Soccer result](https://www.huijiwiki.com/wiki/属性:Soccer_result) |                1 9月 2016 (中国, 韩国, 2, 3)                 |
| [Telephone number](https://www.huijiwiki.com/wiki/特殊:类型/Telephone_number) | 电话号码型       | [Property:Telephone number](https://www.huijiwiki.com/wiki/属性:Telephone_number) |            [+1-800-225-5288](tel:+1-800-225-5288)            |
| [Temperature](https://www.huijiwiki.com/wiki/特殊:类型/Temperature) | 温度型           | [Property:Has temperatureExample](https://www.huijiwiki.com/wiki/属性:Has_temperatureExample) |                             23°C                             |
|     [URL](https://www.huijiwiki.com/wiki/特殊:类型/URL)      | URL型            | [Property:Has_URL](https://www.huijiwiki.com/index.php?title=属性:Has_URL&action=edit&redlink=1) |   [http://www.whitehouse.gov](http://www.whitehouse.gov/)    |

在所有属性中，**页面型**与**文本型**是我们最常用的两类数据类型；

- **页面型属性**，其值为平台上其他实体页面。正是由于页面型属性的存在，实体和实体之间可以通过某些具体的属性建立有向连接；**除了页面型属性，其他属性都是无法建立实体之间的三元组关系的。**也就无法表达”*天津大学的兄弟院校是南开大学* “这一经典的” *主 - 谓 - 宾* “三元组。
- **文本型属性**，其值为文本、段落或其他难以划分类型的内容。在**不确定如何为属性分配类型时，首选文本型属性**。

对于其他属性，有一些特殊的要求：

- **日期型属性**：支持多种不同的日期格式，使用方法参考 SMW 官方文档 https://www.semantic-mediawiki.org/wiki/Help:%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B_%E6%97%A5%E6%9C%9F%E5%9E%8B
- **地理坐标型属性**：该属性只有在安装了 Maps 插件之后才能正确支持，支持多种不同的地理坐标格式。使用方法参考 SMW 官方文档：https://www.semantic-mediawiki.org/wiki/Help:%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B_%E5%9C%B0%E7%90%86%E5%9D%90%E6%A0%87%E5%9E%8B
- **记录型属性**：可以聚合多个属性到同一个记录型属性中，使用示例参考 SMW 文档：https://www.semantic-mediawiki.org/wiki/Help:%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B_%E8%AE%B0%E5%BD%95%E5%9E%8B#%E7%A4%BA%E4%BE%8B，另外，该属性可以被 “子对象” 特性取代。

回到我们最初的例子：

- **兄弟院校**属性，应设置类型为 “页面型”
- **学校位置**属性，应设置类型为 “地理坐标型”，注意需要安装 Maps 插件。
- **创办时间**属性，应设置类型为 “日期型”

### 2、属性定义与多语言支持

上一小结我们介绍了属性的类型，现在我们来展示如何在 MediaWiki 中定义属性。

#### 2.1 属性定义 1：可视化方式

在特殊页面中，找到 “**创建属性**”，输入属性名称，选择属性类型。如果该属性只允许特定的值，如 “月份” 只允许 1~12，那么需要将特定的值输入，并以**英文逗号**作分隔，注意设置特定值的这一步**不是必需的**，。

![image-20230127214219231](https://images.jayliu.tech/blog/2024/11/b48a7906cffac72785c65d215e529d49.png)

#### 2.2 属性定义 2：源代码方式 (推荐)

直接访问链接，注意将<属性名>替换为你想要创建的属性名称

~~~
http://localhost/mediawiki/index.php?title=属性:<属性名称>&action=edit
~~~

如：

~~~
http://localhost/mediawiki/index.php?title=属性:月份&action=edit
~~~

然后在编辑框输入代码：

~~~
[[具有类型::文本型]]
~~~

显然，只需在 `[[具有类型::<属性类型>]]` 中设置属性类型即可。

如果该属性只允许特定的值，那么只需要将特定的值设置为如下格式，并将该代码输入到编辑框中。**注意这一步不是必需的**。

~~~wiki
允许的用于该属性的值为：
* [[允许取值::1]]
* [[允许取值::2]]
* [[允许取值::3]]
* [[允许取值::4]]
* [[允许取值::5]]
* [[允许取值::6]]
* [[允许取值::7]]
* [[允许取值::8]]
* [[允许取值::9]]
* [[允许取值::10]]
* [[允许取值::11]]
* [[允许取值::12]]
~~~

定义属性之后，**暂时无法编辑属性页面，需要等待 MediaWiki 处理其作业队列**；如果等待时间较长，可以在 MediaWiki 根目录下的 maintaince 文件夹中，打开 cmd，运行命令：

~~~cmd
php runJobs.php
~~~

#### 2.3 属性的多语言支持

我们在上述例子中，我们使用**中文**定义了属性名称 “月份”，而在不同语言中可能存在同样的词语来描述相同意义，如**英文**中的 month，**德文**中的 monat 等。为了避免重复和支持国际化，我们想为一个属性添加更多语言的支持，则需要在该属性对应的页面进行编辑，添加如下代码：

~~~wiki
* [[Has preferred property label::@@@]]
** [[Has preferred property label::<属性在对应语言的名称>@<语言代码>]]
* [[Has property description::@@@]]
** [[Has property description::<属性在对应语言的描述>@<语言代码>]]
~~~

如，在 `属性:月份` 页面，我们想要添加**英语**的支持，需要编辑页面，并添加如下代码：

~~~wiki
* [[Has preferred property label::@@@]]
** [[Has preferred property label::month@en]]
* [[Has property description::@@@]]
** [[Has property description::A month is a unit of time@en]]
~~~

语言代码对照表：http://www.lingoes.cn/zh/translator/langcode.htm

尽管添加多语言支持**并不是必须**的，**在知识图谱的具体实践中，与我们教程的做法略有不同，我们往往需要使用英文来作为属性名称 (以便于数据的导入与处理)，并为该属性添加中文支持 (以便于浏览检索)**。



综上所述，如果我们想要创建三个中文属性，**创办时间、学校位置、兄弟院校**，并添加英文支持，他们的创建流程分别为：

- 访问 `http://localhost/mediawiki/index.php?title=属性:创办时间&action=edit`，输入代码：

  ~~~wiki
  [[具有类型::日期型]]
  * [[Has preferred property label::@@@]]
  ** [[Has preferred property label::founding time@en]]
  * [[Has property description::@@@]]
  ** [[Has property description::founding time of an organization@en]]
  ~~~

- 访问 `http://localhost/mediawiki/index.php?title=属性:学校位置&action=edit`，输入代码：

  ~~~wiki
  [[具有类型::地理坐标型]]
  * [[Has preferred property label::@@@]]
  ** [[Has preferred property label::school location@en]]
  * [[Has property description::@@@]]
  ** [[Has property description::School location in latitude and longitude@en]]
  ~~~

  注意，使用地理坐标类型，需要安装 Maps 插件。

- 访问 `http://localhost/mediawiki/index.php?title=属性:兄弟院校&action=edit`，输入代码：

  ~~~wiki
  [[具有类型::页面型]]
  * [[Has preferred property label::@@@]]
  ** [[Has preferred property label::sister college@en]]
  * [[Has property description::@@@]]
  ** [[Has property description::Colleges that are paired are referred to as sister colleges@en]]
  ~~~

### 3、属性使用与标注方法

#### 3.1 显式标注

|                             输入                             |                         实际展示效果                         |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|           `利用属性"月份"对[[月份::12]]加以标注。`           | 利用属性"月份"对[12](https://www.semantic-mediawiki.org/w/index.php?title=链接&action=edit&redlink=1)加以标注。 |
|         `在链接的位置上显示[[月份::12|替代文字]]。`          | 在链接的位置上显示[替代文字](https://www.semantic-mediawiki.org/w/index.php?title=链接&action=edit&redlink=1)。 |
| `要使属性[[月份::12| ]]隐藏起来，彻底不显示，请以一个空格作为替代文字。` **注意：**通道符 \| 后面的空格必不可少。（要让正文中显示空格，请输入特殊符号`&nbsp;`作为空格符）。 | 要使属性[ ](https://www.semantic-mediawiki.org/w/index.php?title=链接&action=edit&redlink=1)隐藏起来，彻底不显示，请以一个空格作为替代文字。 |
| `要在不创建属性的情况下建立带有双冒号 :: 的普通链接，请在其前面采用一个英文冒号来实现该置标的转义。比如，[[:C++ :: 操作符]]` | 要在不创建属性的情况下建立带有双冒号 :: 的普通链接，请在其前面采用一个英文冒号来实现该置标的转义。比如，[C++ :: 操作符](https://www.semantic-mediawiki.org/w/index.php?title=C%2B%2B_::_操作符&action=edit&redlink=1)。 |
| `要将一个取值，比如"链接"，赋予多个属性，请在属性名称之间插入双冒号 ::。比如，[[属性1::属性2::链接]]` | 要将一个取值，比如"链接"，赋予多个属性，请在属性名称之间插入双冒号 :: 。 比如，[链接](https://www.semantic-mediawiki.org/w/index.php?title=链接&action=edit&redlink=1)。 |

当然，需要注意的是：

- 由于我们的属性是具有特定类型的，因此在标注文本时，应当保证标注的**内容**与我们的**属性类型**一致。
- 同一个属性**可以在页面中出现多次**。比如 “兄弟院校” 属性，在”天津大学 “的页面实体中，可以标注 “南开大学”，同时也标注 “上海交通大学”。
- 尽管同一个属性可以出现多次，但同一个属性的**多个相同取值，只会保存一个取值**。比如 “兄弟院校” 属性，在 “天津大学” 的页面实体中，可以多次标注 “南开大学”，但只会存储一次。

#### 3.2 隐式标注

显式标注是为了在原有的非结构化文本上进行标注，标注之后依然能够正常展示原有文本。而在很多时候，我们并不希望将标注的属性在页面上显示出来，此时可以使用 set 隐性标注。

该函数成对地接受属性名称与取值，并采取语义的方式将其存储起来；但是，该函数并不向页面输出任何内容。示例：

~~~wiki
{{#set:月份=12}}
~~~

设置之后，页面不会显示我们编写的内容；但访问 “浏览属性” 页面，依然可以看到我们的属性已经设置在页面实体上。

我们也可以在同一个 set 下一次性设置多个页面属性，示例：

~~~wiki
{{#set:创办时间=1895
|学校位置=39°6'28"N, 117°10'22"E
|兄弟院校=南开大学
}}
~~~

当一个属性对应多个取值的时候，我们可以将 `|+sep=;` 添加到该属性结尾，并使用**英文分号；**来对多个取值分隔，示例：

~~~wiki
{{#set:创办时间=1895
|学校位置=39°6'28"N, 117°10'22"E
|兄弟院校=南开大学;上海交通大学|+sep=;
}}
~~~

综上所述，我们对属性的使用和标注有显式和隐式两种方式。

- **显式方式标注**，便于对**非结构化**文本进行语义化。另外，由于显式标注语法非常简单，在数据处理阶段，我们可以使用 **NLP** 的方式对文本进行**实体识别和关系抽取**，并按照显式标注语法，来对非结构化文本进行批量标注。这种方式也便于将文本进行众包编辑。
- **隐式方式标注**，便于对**结构化**数据进行语义化。这种方式不会影响页面的显示逻辑和显示内容，且可以批量设置多个属性和属性的多个值，编写较为直接简单。
- 实践中，我们将**显式与隐式结合使用**，来对页面实体的结构化、非结构化数据进行语义化。

### 4、逆属性

了解即可，参看文档：

https://www.huijiwiki.com/wiki/%E5%B8%AE%E5%8A%A9:Semantic_MediaWiki/%E8%B4%9F%E5%B1%9E%E6%80%A7

### 5、子属性

了解即可，参看文档：

- https://www.semantic-mediawiki.org/wiki/Help:Special_property_Subproperty_of_(zh-hans)
- https://www.semantic-mediawiki.org/wiki/Help:%E6%8E%A8%E7%90%86#%E5%AD%90%E5%B1%9E%E6%80%A7

注意，子属性与记录型 (Record) 型属性并不相同，不要混淆。

### 6、特殊属性

了解即可，参看文档：

https://www.semantic-mediawiki.org/wiki/Help:%E7%89%B9%E6%AE%8A%E5%B1%9E%E6%80%A7

## 三、分类

**分类**是 MediaWiki 的一项功能，能够自动索引页面，为读者提供某一主题下的页面实体列表。只需给页面的维基文本中加上一个或多个 **Category** 标记即可将页面归类。这些标记将在页面底部创建**指向分类页面的链接**，从而可以很方便地查看同一分类下的其他相关文章。在语义化 Wiki 中，我们常常使用分类来筛选页面实体。

所有**分类本身的页面** (如 Category：人) 都位于**分类**[命名空间](https://www.mediawiki.org/wiki/Special:MyLanguage/Help:Namespaces)中，该页面中包含该分类下的页面以及该分类中的子类。当一个页面属于一个或更多分类，这些分类将呈现在页面底部 (或是右上角，取决于[皮肤设置](https://www.mediawiki.org/wiki/Special:MyLanguage/Help:Preferences))。

![image-20230127214335841](https://images.jayliu.tech/blog/2024/11/4a9436b4364062a9d75cc3cbf74f2634.png)

分类页面本身包含两部分：

- 开头是一个像其他页面的可编辑区域，可以放置内容；
- 底部是包含于这个分类中的所有页面的列表。该列表是固定存在、自动生成的。该列表按字典顺序排列，以链接的形式呈现 (实际上是按照 Unicode 排序)。

在特殊页面中，我们可以：

- **页面表单->创建分类**下对分类进行创建
- **页面列表->分类**下查看所有分类
- **页面列表->追踪分类**，这里的分类是由 MediaWiki 自动生成的，主要用来查看平台上存在问题的页面
- **维护报告->需要的分类**，这里的分类没有进行创建，但是在页面实体中已经被使用。

### 1、页面归类

要为页面归类，只需在页面最后添加这样一行 (**名称**是要归入的分类的名称)：

```
[[Category:<名称>]]
```

比如你可以在 “天津大学” 页面最后添加：

~~~
[[Category:大学]]
~~~

来将天津大学这个页面实体归类到 “大学” 分类下。

你可以为页面添加多个 Category 标记，页面名称将列入所有这些分类页面中。比如：

~~~
[[Category:大学]][[Category:副省部级单位]]
~~~

### 2、创建分类

需要说明的是，即使不主动创建分类，只要在页面中归类过，该分类就已经在 MediaWiki 中存在了。但是这样的分类会与其他页面相对孤立，也无法构成分类树等结构，因此依然需要我们手动创建。

在 ***Category：***命名空间中创建一个页面将会创建一个分类。分类页面的创建与普通页面的创建相同 (参见上一节中的页面创建步骤)，创建时注意在页面名称前加上 “`Category:`” 即可。创建分类之后，可以在分类页进行编辑，并对分类进行一些约束 (注意这些约束不是必要的，按需添加即可)。例如：

- 某一个分类可能是另一个分类的**子分类**。例如 “**大学**” 分类是 “**学校**” 分类的**子分类**，因此，在创建”**Category：大学** “页面后，编辑该页面，加入：

  ~~~wiki
  [[Category:学校]]
  ~~~

  即可将 “**大学**” 分类设置为 “**学校**” 分类的**子分类**。

- 某一个分类可能对应了一个**表单**，可以使用 Page Forms 的表单功能来创建这个分类下的页面。在创建分类页面后，编辑该页面，加入：

  ~~~wiki
  {{#default_form:2211 事件}}
  ~~~

  即可将分类与表单进行绑定。例如

  ~~~wiki
  {{#default_form:学校实体表单}}
  ~~~

- 在下一章我们还会使用更加细致的方式来管理分类页，并定义分类对应的 schema。

另外也可以在特殊页面中，选择 “页面表单->创建分类”，并选择分类的默认表单与子分类。

### 3、分类层次与分类树

分类可以是某分类的子分类。与其他页面类似，可将 Category 添加到分类页面的底端，从而将该分类归入某个母分类。

在设计分类时，应当把所有分类组织成围绕某个根分类的层次结构。分类结构可构成一种树状结构，包含不同的分支。重点是要存在链接到根分类的母子关系链。

![](https://images.jayliu.tech/blog/2024/11/63b34b32e320d40ab7790b1a1ebe1f67.png)

MediaWiki 自 1.31 版本起默认打包了[扩展：CategoryTree](https://www.mediawiki.org/wiki/Special:MyLanguage/Extension:CategoryTree) 扩展，可以在特殊页面中的**页面列表->分类树**中显示出分类的树形结构，或在某个分类页下看到该分类的子分类树。也可以在编辑页面时，加入代码 `‎<categorytree>` 即可显示当前平台上的分类树结构；

### 4、分类页面的链接

要创建一个指向分类页面的链接，需要在 “分类” 前加上冒号 (没有这个冒号，页面将被归入该分类)：

`[[:Category:Help]]` → **[Category:Help](https://www.mediawiki.org/wiki/Category:Help)**

改变链接所显示的文本，请使用管道符号 “`|`” 传递文本：

`[[:Category:Help|帮助分类]]` → **[帮助分类](https://www.mediawiki.org/wiki/Category:Help)**

[重定向](https://www.mediawiki.org/wiki/Special:MyLanguage/Help:Redirects)到分类页面也需使用冒号，否则该页面将被加入到分类页面中。

### 5、分类与模板

如果您在模板 (即嵌入页面) 中添加 `[[Category:Cats]]`，则该模板和嵌入此模板的页面**都**将被添加到 Cats 分类中。

- 如果您只希望对**模板本身**进行分类，则应确保分类标记写在 `<noinclude>` 标签之内。
- 如果您只希望对**使用模板的页面**进行分类，而不希望对模板本身进行分类，则应确保分类标记写在 `<includeonly>` 标签之内。

由于 MediaWiki 平台的缓存问题，透过模板添加的分类**可能**需要一段时间才能生效。您可以在页面上执行一个[空编辑](https://www.mediawiki.org/wiki/Special:MyLanguage/Manual:Null_edit)以立即更新其分类。

### 6、分类重命名

如果您移动了一个分类，由于[重定向功能不适用于分类](https://www.mediawiki.org/wiki/Help:Categories/zh#Redirecting_a_category)，所有分类标签都会出错，**因此必须尽量避免重命名一个分类**。

## 四、模板

### 1、模板介绍

模板其实也是一种标准 wiki 页面，但它主要会被**[嵌入 (transclude)](https://www.mediawiki.org/wiki/Special:MyLanguage/Transclusion)** 到其它页面中。模板的页面名称最前面都有 “`Template:`” 或者 “`模板:`“——将它分配到该[命名空间](https://www.mediawiki.org/wiki/Special:MyLanguage/Help:Namespaces)。

下面展示了模板的最简单使用方法。假设你创建了一个名叫 “Template:Welcome” 的模板，其内容如下：

```
您好！您好！欢迎来到wiki。
```

这样，你就创建了你的第一个模板！如果你接下来插入：

```
{{Welcome}}
```

到其他任一页面，那么，访问这个插入了模板的页面时，将会看到 “您好！欢迎来到 wiki。” 的字样，而不是看到 `{{Welcome}}`。模板的内容被 “嵌入 (transclude)” 到了其他页面，也就是说，被整合到了这个页面。你可以在任一页面的任何位置添加 `{{Welcome}}`，来欢迎其他人。**设想一下该模板被用在 100 个页面中**，假设你后来将模板内容改成：

```
嗨，您好！欢迎来到这个奇妙的wiki。
```

并且**重新访问之前用到了该模板的那 100 个页面**，你将会看到**新的文字**，而不是原先的文字。用这个方法可以**批量改变 100 个页面**，而并不是手动修改这 100 个页面的内容，因为这个模板是被嵌入这 100 个页面中的。

这只是模板的一个最基本的应用。配合模板参数、Page Forms 插件、解析器函数等，我们可以实现非常高级、灵活的功能，以控制页面样式、数据传递等。

### 2、Transclusion 页面嵌入机制

**Transclusion** (嵌入) 通常是指通过引用将一个文件的内容纳入另一个文件中。在 MediaWiki 中，它是指使用 MediaWiki 的模板功能，将相同的内容包含在多个页面中，而不必单独编辑这些页面。

![Transclusion(wtd-ck)](https://images.jayliu.tech/blog/2024/11/162ee3110e58ba322416d0fd4c7b7a0a.png)

以上图为例，页面 B 的内容为 “foo “，我们在编辑页面 A 时，想要将页面 B 的内容嵌入到当前页面，置于” Lorem ipsum “与” dolor sit amet “两段文字之间，可以使用 `{{B}}` 这种写法来嵌入：

~~~wiki
Lorem ipsum
{{B}}
dolor sit amet
~~~

每当目标页面 A 被访问时，源页面 B 的全部内容将在**花括号标签的位置**被渲染。在我们访问 A 页面时，即可看到：

~~~wiki
Lorem ipsum
foo
dolor sit amet
~~~

当然，正如图上所示，这种页面嵌入是可以多层嵌套的 (如页面 B 嵌入 A，A 再嵌入 X)。

#### 2.1 嵌入语法

如果被嵌入页面来源是在模板命名空间 (例如 “**模板：Welcome**”)，只需使用名称本身，单独使用：`{{Welcome}}`。

如果来源是在主文章命名空间 (例如 “**VisualEditor**”)，则必须在名称前面加上冒号：`{{:VisualEditor}}`。

如果源代码在任何其他命名空间 (例如 “**用户：Example**”)，则必须使用全名，包括命名空间。`{{User:Example}}`

如果源文件是目标页面的一个**子页面**，(例如 “**Transclusion/ja**”)，你可以简单地指定该子页面的名称，而不考虑名称空间。`{{/ja}}`

#### 2.2 局部嵌入

通过使用 “**noinclude**”、“**onlyinclude**” 和 “**includeonly**” 标记，可以将一个页面的**部分内容**而不是全部内容嵌入到其他页面。比如，例如，你可以将**苹果**的全部相关内容 (如烟台苹果、红富士、蛇果等) 包含在页面”**苹果** “中，而在制作苹果派的页面” **苹果派食谱** “中，仅仅传入页面” 苹果 “中的” **烟台苹果** “的内容。

假设**页面 B 是源页面**，我们想要将页面 B 的部分内容嵌入到页面 A，可以对页面 B 中的内容进行标记：

- noinclude - 标记 `<noinclude>...</noinclude>` 意味着 noinclude 标记之间的文本只在**源页面 (页面 B) 上显示，不会嵌入到调用页面 (页面 A) 并显示**。这对模板文档和类别很有用。
- includeonly - 标记 `<includeonly>...</includeonly>` 意味着标记之间的文本将在**源页面上 (页面 B) 不显示，会嵌入到调用页面 (页面 A) 并显示**。这可能很有用，例如，在包含模板的页面上添加类别，而不将模板本身添加到这些类别。
- onlyinclude - 标记 `<onlyinclude>...<onlyinclude>` 意味着标记之间的文本在**源页面上 (页面 B) 显示，会嵌入到调用页面 (页面 A) 并显示**。该标签**优先于其他标签**。如果一个页面上至少有一对 “onlyinclude” 标签，那么每当这个页面被嵌入时，**只有 “onlyinclude” 标签内的内容被嵌入**。

举例：

| 页面 | 编辑内容                                                     | 显示内容                 |
| ---- | ------------------------------------------------------------ | ------------------------ |
| B    | \<noinclude>我爱\</noinclude><br/><br/>\<includeonly>天大\</includeonly><br/><br/>校园 | 我爱<br/><br/>校园       |
| A    | 这里是{{:B}}                                                 | 这里是<br/>天大<br/>校园 |

若在页面 B 的编辑内容中加入一行，则：

| 页面 | 编辑内容                                                     | 显示内容                                   |
| ---- | ------------------------------------------------------------ | ------------------------------------------ |
| B    | \<noinclude>我爱\</noinclude><br/><br/>\<includeonly>天大\</includeonly><br/><br/>校园<br />\<onlyinclude>TianJin University\</onlyinclude> | 我爱<br/><br/>校园<br />TianJin University |
| A    | 这里是{{:B}}                                                 | 这里是TianJin University                   |

**注意，此时只有 onlyinclude 的内容会被嵌入到页面 A 之中！**

### 3、模板创建

要创建一个模板，主要是两种方式，一种是使用源代码方式创建，一种是使用 PageForms 的可视化引导创建。我们以上文中的**兄弟院校、创办时间、学校位置**三个属性为例来为天津大学页面创建一个**信息框模板**。

#### 3.1 PageForms 可视化创建

在特殊页面中，找到 “**页面表单-创建模板**”，点击并填写字段：

![image-20230201220412656](https://images.jayliu.tech/blog/2024/11/f7f7bf8f4e1c15183980a903eb849c4d.png)

![image-20230201221102273](https://images.jayliu.tech/blog/2024/11/fbba4a0758d2b88e7c8137bc2a28d9a8.png)

注意需要开启 “Use full wikitext instead of #template_display”。这里需要解释的几个选项

- 模板名称：指定模板名称之后，会在命名空间 “模板” 下，创建一个页面。如这里会创建一个 “模板：大学信息框 “的页面。

- 模板所定义的分类：这里指的是 “**使用该模板的页面**” 所归类到的分类。如在天津大学页面中调用大学信息框模板，会将 “天津大学” 页面归类到 “大学” 这个分类下。

- 字段名称：指的是在调用模板时，标签显示的字段名称。

  ![image-20230201221249785](https://images.jayliu.tech/blog/2024/11/4bf902bacba5a4193209b9d6361db184.png)

- 显示标签：指的是在 “**使用该模板的页面**” 上，表格中显示的字段名称。

  ![image-20230201221540429](https://images.jayliu.tech/blog/2024/11/50016f4bf18d0d4073d26da3f53effc0.png)

- 语义属性：指的是在 “**使用该模板的页面**” 上，定义的语义**属性**名称。在天津大学页面点击浏览属性：

  ![image-20230201221754641](https://images.jayliu.tech/blog/2024/11/26bfeb5ef3475bf5b7d9d771075d047e.png)

- 聚合：如”天津大学 “页面调用了该模板，这里主要用来显示哪些页面反向链接了当前页面。

- 输出格式：可选表格、信息框、纯文本、小节等。这里选择**信息框**即可。

#### 3.2 源代码方式

直接访问链接，注意将<模板名>替换为你想要创建的模板名称：

~~~
http://localhost/mediawiki/index.php?title=模板:<属性名称>&action=edit
~~~

如：

~~~
http://localhost/mediawiki/index.php?title=模板:月份&action=edit
~~~

然后在编辑框输入代码：

~~~html
<noinclude>
这是“大学信息框”模板。调用它时应该采用如下格式：
<pre>
{{大学信息框
|兄弟院校模板标签=
|创办时间模板标签=
|学校位置模板标签=
}}
</pre>
编辑页面以阅读模板文本。
</noinclude>

<includeonly>
{| style="width: 30em; font-size: 90%; border: 1px solid #aaaaaa; background-color: #f9f9f9; color: black; margin-bottom: 0.5em; margin-left: 1em; padding: 0.2em; float: right; clear: right; text-align:left;"
! style="text-align: center; background-color:#ccccff;" colspan="2" |<span style="font-size: larger;">{{PAGENAME}}</span>
|-
! 兄弟院校（Sister college）
| {{#arraymap:{{{兄弟院校模板标签|}}}|,|x|[[兄弟院校::x]]}}

|-
! 创办时间（Founding time）
| [[创办时间::{{{创办时间模板标签|}}}]]
|-
! 学校位置（School location）
| [[学校位置::{{{学校位置模板标签|}}}]]
|-
! 
|{{#ask:[[Foaf:homepage::{{SUBJECTPAGENAME}}]]|format=list}}
|}

[[分类:大学]]
</includeonly>

~~~

在理解 Transclusion 页面嵌入机制之后，该源代码并不难理解。我们不妨在 “天津大学” 页面调用该模板，编辑页面，加入以下代码：

~~~wiki
{{大学信息框
|兄弟院校模板标签=南开大学
|创办时间模板标签=1895
|学校位置模板标签=39°6'28"N, 117°10'22"E
}}
~~~

那么在源代码中：

- `noinclude` 标签中的内容，会显示在页面 “模板：大学信息框” 上，不会嵌入到天津大学页面。

- `includeonly` 标签中的内容，不会显示在页面 “模板：大学信息框” 上，但会嵌入到天津大学页面。需要注意，这个标签中的内容结合了我们所学到的**魔术字、wiki 表格语法**以及**属性的显式标注**等知识点。

- 我们在天津大学页面中调用模板，输入的 “1895” 会传递给**模板标签变量**”创办时间模板标签 “，然后在模板内部，使用语法 ` {{{创办时间模板标签|}}}`，即可获取到” 南开大学 “这一文本。因此，源代码中的：

  ~~~wiki
  | [[创办时间::{{{创办时间模板标签|}}}]]
  ~~~

  经过模板处理之后，在天津大学页面上，等于嵌入了

  ~~~wiki
  | [[创办时间::1895]]
  ~~~

- `arraymap` 是一个特殊的解析器函数，我们会在下面章节的 **Page Forms 解析器函数**中进行解释。

### 4、模板使用

在页面中调用模板，填入符合语义属性要求的数据即可：

~~~wiki
{{大学信息框
|兄弟院校模板标签=南开大学
|创办时间模板标签=1895
|学校位置模板标签=39°6'28"N, 117°10'22"E
}}
~~~

需要注意的是，我们在模板中使用的语义属性的显式标注，因此**输入的数据需要满足其格式要求**。如 “学校位置模板标签” 对应的数据，就应填入一个正确的经纬度坐标，而不能是 “中国天津市南开区天津大学卫津路校区” 这样的文字信息。

### 5、模板语法与复杂模板

实际上，模板中的内容是可以在源代码中进行自定义修改，并实现更多特殊展示效果与功能的。创建模板之后，进入到模板页面，点击编辑即可对模板的源代码进行修改。比如我们可以在模板中调整**信息框的宽度**和**标题行背景色**：

~~~html
{| style="width: 40em; font-size: 90%; border: 1px solid #aaaaaa; background-color: #f9f9f9; color: black; margin-bottom: 0.5em; margin-left: 1em; padding: 0.2em; float: right; clear: right; text-align:right;"
! style="text-align: center; background-color:#27ba9b;" colspan="2" |<span style="font-size: larger;">{{PAGENAME}}</span>
|-
! 兄弟院校（Sister college）
| {{#arraymap:{{{兄弟院校模板标签|}}}|,|x|[[兄弟院校::x]]}}

|-
! 创办时间（Founding time）
| [[创办时间::{{{创办时间模板标签|}}}]]
|-
! 学校位置（School location）
| [[学校位置::{{{学校位置模板标签|}}}]]
|-
! 
|{{#ask:[[Foaf:homepage::{{SUBJECTPAGENAME}}]]|format=list}}
|}
~~~

![image-20230201224608142](https://images.jayliu.tech/blog/2024/11/2c1820a16b0a9069bc390f406506c48a.png)

事实上，你可以在**模板中添加任意形式的内容**，甚至在模板内部再调用其他模板，来控制实体页的展示方式！

而对模板进行编辑和自定义，需要了解模板的基本语法：

- 维基百科模板语法 (了解)：https://zh.wikipedia.org/wiki/Help:%E6%A8%A1%E6%9D%BF
- huijiwiki 模板教程 1 (了解)：https://www.huijiwiki.com/wiki/%E5%B8%AE%E5%8A%A9:%E6%A8%A1%E6%9D%BF#%E8%B0%83%E7%94%A8%E6%A8%A1%E6%9D%BF
- huijiwiki 模板教程 2 (了解)：https://www.huijiwiki.com/wiki/%E5%B8%AE%E5%8A%A9:%E6%A8%A1%E6%9D%BF%E5%8F%82%E6%95%B0

注意在以上教程中，解释了如 “**模板嵌套**”，“**命名参数**”，“**位置参数**”，“**参数嵌套**” 的问题。**这里将在作业中进行考察。**

## 五、表单

在完成了创建属性、创建分类、创建模板之后，我们对于 MediaWiki 的页面实体控制已经可以实现批量化、自动化的创建和修改了。而为了便于非专业用户编辑和使用，Page Forms 还提供了 “**表单**” 功能。

### 1、表单创建

在特殊页面中，找到”**页面表单-创建表单** “：

![image-20230201232307573](https://images.jayliu.tech/blog/2024/11/1257fedf5ffc3204531aaba2280398bc.png)

在添加元素下，点击添加 “**大学信息框**” 模板：

![image-20230201232533527](https://images.jayliu.tech/blog/2024/11/dce017b589a0a8d04b6ed631af87c3f1.png)

这里要解释一些功能：

- 添加该模板后，可以选择 “**在所创建的页面中允许此模板的多个 (或零个) 实例**”。以大学信息框模板为例，如勾选该项，可以在该页面添加多个大学信息框的 infobox。

- 表单标签：用户在表单输入时的标签提示。

  ![image-20230201232933245](https://images.jayliu.tech/blog/2024/11/00b729542eb51257ea4982e5d4714610.png)

- 输入项类型：选择默认即可，也可改为其他自定义的输入项。

- 其他参数：可以选择一些如 placeholder 等其他参数，以控制表单输入页的样式和功能。

- 添加小节：除模板的字段外，也可以将单纯的文本作为表单的输入项，按照小节来添加内容。

点击创建即可完成表单的定义。

### 2、表单使用

点击创建后，进入表单界面，输入 “天津大学” 进入表单编辑页：

![image-20230201233241993](https://images.jayliu.tech/blog/2024/11/8cd98d78cc7d1f846288e870be104aa9.png)

可用该表单来可视化编辑天津大学页的内容：

![image-20230201233327360](https://images.jayliu.tech/blog/2024/11/7b13ec758b1c55554c2867b23c99269c.png)

## 六、Page Forms 的概念与使用

Page Forms (2016年之前被称为 Semantic Forms) 是 MediaWiki 的一个扩展，允许用户使用**表单**来**添加、编辑和查询数据**。它最初是作为 Semantic MediaWiki 扩展的一个分支而创建的，以便能够编辑那些通过 SMW 来存储其参数的模板，这就是为什么它最初被称为 “Semantic Forms”。不过，它现在也可以与 Cargo 扩展一起工作，或者在两个扩展都没有安装的情况下单独工作。

一般来说，Page Forms 是与 Semantic MediaWiki 或者 Cargo 共同使用的。它们为 MediaWiki 提供了**结构化语义数据**的解决方案。

### 1、Page Forms 的简单使用

在特殊页面中，找到 “**页面表单-创建类**。在这里 Page Forms 插件提供了一个可视化的界面，来**一次性创建某类数据的属性、模板、分类、表单**。如果你填写了这些字段并点击” 提交 “，该页面将自动创建所有必要的属性、模板、分类、表单页面。然后你就可以到创建的表单页面，开始输入数据。

![image-20230201231034046](https://images.jayliu.tech/blog/2024/11/a21ef8947b45a32b6cc78d0e04bd13b4.png)

### 2、Page Forms 相关解析器函数与具体文档

Page Forms 的具体文档，包括

#### 2.1 解析器函数

Page Forms 提供了多个解析器函数，如 `arraymap`、`arraytemplate`、`default form` 等，这里主要介绍两个常用函数：

##### 2.1.1 arraymap

arraymap 是一个灵活的函数，用以展开一段内容中的每一个元素，具体方法：

- `{{#arraymap:<要展开的内容>|<分隔符>|<变量>|<变量公式>|<展开后的分隔符>}}`

如果要展开的内容是一个逗号隔开的字符串，例如：`关羽,张飞,马超,赵云,黄忠`

- `{{#arraymap:关羽,张飞,马超,赵云,黄忠|,|@@@|前一段@@@后一段}}| && }}`

这段代码的意思就是将关羽、张飞、马超、赵云、黄忠视为要展开的 5 个元素 (由指定的分隔符 “，” 从要展开的字符串里获得)，每个元素都被放到公式 “前缀@@@后缀” 中去并用符号 “&&” 隔开，代码展开之后等价于：

~~~
前一段关羽后一段&&前一段张飞后一段&&前一段马超后一段&&前一段赵云后一段&&前一段黄忠后一段
~~~

注意：arraymap 函数常用于模板中，用来接收和处理同一字段输入的多个值。

##### 2.1.2 default form

`default form` 函数用来为当前页面制定默认使用的表单，指定后页面就可以使用表单编辑。

- 写法：`{{#default_form:表单名称}}`，
- 例如：`{{#default_form:大学信息框}}`，

#### 2.2 参考文档

以下文档为英文文档，在用到时参考即可。

- **表单定义语法**：如何在表单名字空间 “Form：” 之中的页面定义表单。涵盖完整的表单定义语法，包括 `{{{info}}}`、`{{{for template}}}`、`{{{end template}}}`、`{{{field}}}`、`{{{section}}}` 以及 `{{{standard input}}}`。同时还包括如何添加选项卡和工具提示。

  https://www.mediawiki.org/wiki/Special:MyLanguage/Extension:Page_Forms/Defining_forms

- **表单输入类型**：列出所有允许的输入类型，以及每种类型的参数。

  https://www.mediawiki.org/wiki/Extension:Page_Forms/Input_types

## 七、作业安排

本周主要介绍了 Semantic MediaWiki 中的基本概念、特性和使用方法。从**页面实体、属性**，到**分类、模板、表单**，这一套架构构成了 MediaWiki 的**语义化架构**。我们可以在这个架构下来对知识体系进行重构，在知识库之上创建知识图谱。本周学习的内容较多，但是由于这几部分内容关联性较强，因此放在一起进行介绍，更容易让大家理解整个架构。

作业：

- 任务 1：浏览、学习基本的 CSS 语法，并用 **HTML 语言 + CSS** 修改原有的天津大学的百科介绍页面 (**两周任务**)
- 任务 2：安装 Maps 插件，插件版本为 10.0
- 任务 3：测试 Transclusion 页面嵌入机制，要求在平台上使用 “**noinclude**”、“**onlyinclude**” 和 “**includeonly**” 标记展示嵌入机制的功能。
- 任务 3：在 MediaWiki 中创建 “**大学**” 的属性、分类、模板、表单，并应用在 “天津大学” 页面上。要求：
  - 表单定义时，添加元素至少 3 个模板，其中包括一个信息框模板；添加 2 个以上小节；
  - 表单的输入类型，必须使用到 `datepicker` 类型、`combobox` 类型 (参考官网文档)；
  - 不可使用默认信息框样式，需要进行自定义的样式修改；
  - 信息框中属性至少为 10 条，属性类型至少为 4 种；
  - 模板定义时，要用到 `arraymap函数`，且展开后的分隔符需要使用分号 `;`。