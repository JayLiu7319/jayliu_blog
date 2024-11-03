+++
date = '2023-11-02T14:17:02+08:00'
draft = false
title = 'SMW 教程第一章：MediaWiki平台的Windows安装搭建'
featured_image = 'https://opengraph.githubassets.com/04dfd6c68d3e4474ec57360ec23f051e8ff9f3d4f1c94786364e7ef9645b374c/SemanticMediaWiki/SemanticResultFormats'
description = '本文提供了MediaWiki平台的安装与配置指南，包括数据库设置、插件安装和皮肤更换等，适合知识图谱平台构建初学者使用。'
+++

# SMW 教程第一章：MediaWiki平台的Windows安装搭建

## 一、培训说明

各位同学大家好，在开始我们的教程培训之前，需要向大家介绍关于本次培训的一些信息，以供大家参考

1. **培训目标**：引导各位同学掌握知识图谱的基本概念和构建思路；掌握 Mediawiki 平台的搭建、使用和开发；掌握 Semantic Mediawiki 的安装和应用；掌握从平台搭建、Schema 设计、数据获取、数据处理与语义化、数据导入导出、知识图谱可视化等全流程的技术；在教程基础上能够根据自身项目主题搭建知识图谱平台，并进行可视化分析、算法应用等；

2. **培训内容**：培训内容主要分为两个部分，一是**教程主线**，二是**相关技术**。其中，教程主线按照**章节**划分，为我们培训的**主要内容**与**规定动作**，同学们每周需要完成一个章节的教程学习并进行实操，完成对应章节的作业；相关技术按照**主题**划分，主要是在构建 MediaWiki 平台时涉及到的一系列相关知识，由同学们自由学习和互相分享，**不作强制要求**，大二项目制小组可以选择其中一个主题 (或自拟主题并经由大二经理认可)，在每周培训会上进行分享，会有相应的加分；大四毕设同学也可以根据自身能力选择并分享。

   - 教程主线：
     - 第一章：Mediawiki 平台的安装搭建、皮肤与插件使用
     - 第二章：Mediawiki 与 Semantic Mediawiki 的使用和基本概念
     - 第三章：Mediawiki 下知识图谱的概念、实现思路与 Schema 设计
     - 第四章：Mediawiki 模板设计与数据导入【基础篇】(院士基础数据)
     - 第五章：Mediawiki 模板设计与数据导入【高级篇】(院士详细数据)
     - 第六章：Mediawiki 数据查询与组件功能开发
     - 第七章：Mediawiki、Vue 与 Dgraph 知识图谱可视化 (进阶内容)
   - 相关技术
     - 数据获取：爬虫技术 (Python、Selenium、八爪鱼/火车头)
     - 数据整理：NLP 算法 (命名实体识别、实体消歧、实体统一、指代消解、关系抽取)、UML 设计、Mindmaster 导图
     - 数据呈现：前端基础 (HTML/CSS/JS)、MediaWiki 模板自动生成 (Python)、MediaWiki 模块开发 (Lua)、MediaWiki 高级插件配置与使用 (Semantic Result Formats、Maps、Cargo……)、MediaWiki Extension 开发、MediaWiki Skin 开发……
     - 算法洞察：图算法 (社团发现、异常检测、链接预测……)

3. **培训形式**：培训形式上，主要以**教程引导** + **实践作业** + **学生自组织培训分享**为主。我们会提前一周发布本周的教程与作业，并在每周的培训会上对同学们的作业进行**逐一展示、逐一检查**，保证同学落实到位。在每周的培训会上，首先由选择某一相关技术的小组或同学进行**技术分享和实操演示**，然后由助教和经理对同学们的作业进行检查与打分。

4. **培训周期**：培训时长为 6~8 周，每周固定时间在腾讯会议开培训会。



##  二、MediaWiki 平台安装 (Windows 系统)

### 1、Wampserver 安装

Wampserver 是一个 Web 集成环境，该集成环境中打包安装了包括 php、apache、mysql 以及 mariadb 等软件 (ps：如果电脑**原来已有 mysql 或者 php 服务运行**的话最好停止)。

**第一步**：Wampserver 版本使用教程包中提供的安装包安装，**一路按照默认选项点下一步即可**，注意安装过程中选择 PHP 版本为 7.3 以上。

![image-20241103111702824](https://images.jayliu.tech/blog/2024/11/f374f5e6e9b3c36e9a6a223d93fe784e.png)

首次运行可能会被 360 杀毒软件报警，忽略警告即可。该软件开始正常运行以后**图标会变绿**，表明 Wamp 安装成功。

**第二步**：在浏览器输入 localhost 进入配置页面：

![image-20241103111739546](https://images.jayliu.tech/blog/2024/11/eaa108434b82cdd4aa27c7c844e70aa0.png)

**第三步**：在该页面的 **Tools 栏下**，点击 **phpmyadmin** 跳转到数据库管理页面，首次登录不用输密码 (登录用户名为 root 或 admin)，**注意选择服务器一定为 MySQL**。

![image-20241103111803261](https://images.jayliu.tech/blog/2024/11/55ebc0f1d068b9f40e25e9ad2ba6013a.png)

**第四步**：进入管理页后点击 “**SQL**” 栏，复制输入以下代码，点击执行。

~~~SQL
CREATE DATABASE my_wiki;
CREATE USER 'wikiuser'@'localhost' IDENTIFIED BY 'Smwuser1234';
GRANT ALL PRIVILEGES ON my_wiki.* TO 'wikiuser'@'localhost' WITH GRANT OPTION;
~~~

![image-20241103111849561](https://images.jayliu.tech/blog/2024/11/9530cb18cc7302b56bde87f08fb525d0.png)

安装配置完成。

### 2、Mediawiki 安装

**第一步**：选择 MediaWiki 安装包，可以在官方版本库：https://releases.wikimedia.org/mediawiki/中进行下载，也可以直接选用安装包中给定的 MediaWiki-1.35.4 版本；将`mediawiki-1.35.4.zip`压缩包解压后，放在 wampserver 的 www 文件夹下。注意需要**修改解压后的文件夹名为 mediawiki**。解压完成后，`wamp\www\mediawiki`文件夹中的文件应当如下图所示：

![image-20241103112324808](https://images.jayliu.tech/blog/2024/11/ee2a28be4355b67ee0d6c009cbcd6677.png)



**第二步**：在浏览器地址栏输入：`localhost/mediawiki`，回车后开始进行安装 mediawiki 安装。

![](https://images.jayliu.tech/blog/2024/11/0a05117660a936b083a11177174043f1.png)

**第三步**：选择语言为 **zh-中文**，**注意不要选择 zh-cn - 中文 (中国大陆)**。

![image-20241103112429196](https://images.jayliu.tech/blog/2024/11/7c511cbf4230198037221e5783f039a8.png)

**第四步**：点击继续直到出现 “**连接到数据库**”，输入数据库名称为 my_wiki，数据库用户名为 wikiuser，密码为 Smwuser1234

![image-20241103112502120](https://images.jayliu.tech/blog/2024/11/5b1898c94231bd308754719e004b0e90.png)

**第五步**：点击继续，填写并设置你的 wiki 名称、用户名、密码、邮件地址等。项目名字空间选择 “与 wiki 名称相同” 即可。

![image-20241103112530266](https://images.jayliu.tech/blog/2024/11/a7ce0847644e0426dab4dc0da2edc50b.png)

**第六步**：选择 “多问我一些问题吧”，继续进行配置。其他设置无需修改，选择 “扩展程序” 下的**全部扩展**并打勾。

![image-20241103112558076](https://images.jayliu.tech/blog/2024/11/69803732aaccdc3ab1253e16a470cbbd.png)

同时选择启用文件上传。**“已删除文件的目录” 会自动生成，无需手动设置**

![image-20241103112616045](https://images.jayliu.tech/blog/2024/11/792e0604b6f95a9dfbe8f188ef2a5a55.png)

**第七步**：点击继续并完成安装。

![img](https://images.jayliu.tech/blog/2024/11/63404a3ebd68f2303a87dcdbffce9585.webp)

提示你下载 **LocalSettings.php 文件**，保存后将它复制至你的 Wiki 站点根目录下 (即 `wamp\www\mediawiki` 文件夹下)。

重新输入你的网站地址 (localhost/mediawiki)，如果能看到跟下面差不多的画面，那么就安装成功了！

![image-20241103112710422](https://images.jayliu.tech/blog/2024/11/37639d6b9b4ebed2442d95c050e7a9a1.png)

**注意：LocalSetting.php 文件是 MediaWiki 网站的主要配置文件，我们之后的很多网站定制化都需要在该文件中进行配置和修改！**



## 三、Mediawiki 平台安装 (Ubuntu 系统)

参见官网教程：https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Debian_or_Ubuntu

## 四、插件安装

### Composer 安装

需要安装 composer 软件 (sementic 插件安装需要)

![image-20241103112734101](https://images.jayliu.tech/blog/2024/11/5f685fa3dbf85c7dac3416f8efe5bab4.png)

安装过程中，php 路径按 wampserver 使用的 php 版本路径设置 (该路径一般在 Wamp 文件夹下的 bin 文件夹的 php 文件夹中，默认是按环境变量)，之后一路 next 即可。

![image-20241103112753870](https://images.jayliu.tech/blog/2024/11/e00693d4e07b3148d82ced93c4898e84.png)

安装以后在 cmd 命令行输入 composer，回车，可以看到可以使用 composer 命令

![image-20241103112811826](https://images.jayliu.tech/blog/2024/11/100411db931aed78f6f8b1d1e8ae8f65.png)

如果 composer 命令失败，需要检查一下环境变量配置中是否有 Composer 路径：

![image-20241103112834073](https://images.jayliu.tech/blog/2024/11/ac23ff6ef69823edece51ed174f4e42d.png)

**注意该步非常重要**：在 cmd 命令行中，输入命令 `composer self-update 2.1.14`

![image-20241103112849199](https://images.jayliu.tech/blog/2024/11/66c99c587ed6fd853ea836c929c45c95.png)

### PageForms 安装

将 PageForms 安装包进行解压，解压后的文件夹放在 `wamp\www\mediawiki\extensions` 文件夹中

![image-20241103112905404](https://images.jayliu.tech/blog/2024/11/22ebb66c645aecadcbd6d74a9c1e0ea4.png)

注意解压后的路径和文件如图所示：

![image-20241103112926249](https://images.jayliu.tech/blog/2024/11/6abedc74201a63a8691fe5d30ae46e46.png)

在 LocalSetting.php 配置文件中加入以下语句：

~~~php
wfLoadExtension( 'PageForms' );
~~~

在 `\wamp\www\mediawiki\maintenance` 路径下，在 cmd 命令行中输入 `php update.php` 命令

![image-20241103112951182](https://images.jayliu.tech/blog/2024/11/b23f94bb3a7df06123675da6c128ba29.png)

### Sementic MediaWiki 安装

使用 `composer update` 命令更新 composer，**在 wiki 的根目录路径** (`wamp\www\mediawiki`) 下输入命令：

~~~
composer require mediawiki/semantic-media-wiki "~3.2"
~~~

![image-20241103113020891](https://images.jayliu.tech/blog/2024/11/a1c8e172f55cb468a73a30a8251f33b8.png)

在 LocalSetting.php 配置文件中加入以下语句：

~~~php
enableSemantics('localhost/mediawiki/');
~~~

在 `\wamp\www\mediawiki\maintenance` 路径下，在 cmd 命令行中输入 `php update.php` 命令

![image-20241103113050592](https://images.jayliu.tech/blog/2024/11/e0abbdcf38dad65a37c8ffe48581f2c3.png)



可以在 wiki 中**特殊页面**的**版本**页面查看插件安装情况。

![image-20241103113129671](https://images.jayliu.tech/blog/2024/11/10a1414db648a02c3cf4f2cbdaece1be.png)

## 四、作业安排

本周主要任务是在自己的电脑上安装好 mediawiki 平台。安装软件平台是锻炼技术能力的第一步，在安装平台、配置环境的过程中，你们会初步了解到现代软件平台的构成方式、使用方式、调试方法，了解到一个网站要以哪些东西构成和运行。此外，在查阅技术文档的过程中，也能够让你们掌握基本的查阅资料、理解文献/文档并最终解决问题的能力。

**作业安排**：

1. 在 windows 平台安装好 mediawiki 平台，具体步骤参看教程；

   (如果是 Mac 电脑，需要在 Mac 上安装 Ubuntu 虚拟机，在 Ubuntu 系统下按照官网教程进行安装 mediawiki 平台)

2. 在 mediawiki 平台中，安装好 pageforms、semantic mediawiki 插件。
   下载、安装方法，参照插件官方页面与本教程：
   https://www.mediawiki.org/wiki/Extension:Page_Forms
   https://www.semantic-mediawiki.org/wiki/Semantic_MediaWiki

3. 在 MediaWiki 和 Semantic MediaWiki 官方插件列表中，选择并安装 3 个及以上其他插件 (**注意不要与平台已有的插件重复**)，并进行基本使用

   https://www.mediawiki.org/wiki/Category:Extensions/zh

   https://www.semantic-mediawiki.org/wiki/Help:Semantic_MediaWiki_extensions

4. 为 mediawiki 平台更换一个**皮肤**

   https://www.mediawiki.org/wiki/Manual:Skins/zh

   https://www.mediawiki.org/wiki/Category:All_skins/zh

5. 浏览、学习基本的 HTML 语法，并用**纯 HTML 语言**创建一个天津大学的百科介绍页面 (**两周任务**)
   https://www.runoob.com/html/html-tutorial.html

进阶 (可供学有余力的同学或组队同学尝试)

1. 在腾讯云使用【学生优惠】购买服务器 (可报销)，并选择 ubuntu 20.04 平台搭建云服务器 (或使用本机的 ubuntu 虚拟机)
2. 按照 mediawiki 官网说明，安装并运行 MediaWiki 平台
3. 在云服务器中安装 pageforms、semantic mediawiki、cargo 这三个基本插件
4. 在云服务中安装 10 个以上其他插件
5. 给 Mediawiki 平台更换皮肤