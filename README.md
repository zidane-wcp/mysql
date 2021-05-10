[TOC]



# 第三章 教程

*3.1 服务器的连接与断开*

*3.2 输入查询*

*3.3 创建并使用数据库*

*3.4 获取数据库和表信息*

*3.5 批处理模式下使用mysql*

*3.6 常用查询示例*

*3.7 结合Apache使用MySQL*



本章通过展示如何使用mysql客户端程序来创建并使用一个简单的数据库，提供了一个教程简介。

mysql（有时被称为终端监视器或监视器）是一个交互式程序，可以让你连接到MySQL服务器、运行查询以及浏览查询结果。mysql还可以用在批处理模式下：事先将你的查询语句存放在一个文件中，告诉mysql来执行这个文件中的内容。这里介绍了mysql的这两种用法。

查看mysql系统的选项列表，请使用选项--help调用它：

```shell
shell> mysql --help
```

本章假设你的机器上已经安装了mysql，并能连接到MySQL服务器，如果没有，请联系你的MySQL管理员。（如果你是管理员，你需要查阅本手册相关部分，比如第五章 MySQL服务器管理。）

本章介绍了构建并使用一个数据库的整个过程。如果你只对如何访问一个已经存在的数据库感兴趣，那你可能想跳过介绍如何创建数据库和表的章节。

由于本章本质上是教程，很多细节需要被忽略。你可以查阅本手册相关章节来获得本章所涉及主题的更多信息。

## 3.1 服务器的连接与断开

为了连接到服务器，你一般需要一个MySQL用户名和密码。如果服务器没有运行在本地，你还需要指定一个主机名。联系你的管理员，以查找用于连接的参数（即主机名、用户名、密码）。只要有了正确的参数，你就可以连接到数据库：

```shell
shell> mysql -h host -u user -p
Enter password: ********
```

host表示MySQL服务器运行的主机名，user表示MySQL账户的用户名。用你配置的合适值替代他们。********表示你的密码。mysql显示“Enter password:”提示符时，输入密码。

如果可行，你应该会看到一些介绍性信息，后面是提示符“mysql>”：

```shell
shell> mysql -h host -u user -p
Enter password: ********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25338 to server version: 8.0.26-standard

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql>
```

提示符“mysql>”告诉你mysql已经准备接受SQL语句。如果你在运行MySQL的同一台机器上登录，你可以省略host参数：

```shell
shell> mysql -u user -p
```

如果你尝试登录时，得到错误消息：ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)，意思是MySQL服务器实例未运行。咨询管理员或参考第二章 安装和更新MySQL 中适合你操作系统的部分。

有关尝试登录时遇到的其他问题的帮助，见附录B.3.2 使用MySQL程序时的常见错误。

有些数据库安装允许用户以匿名（未命名）用户身份连接到运行在本地的服务器。如果你的机器是这种情况，那你可以通过不带任何选项的mysql连接到该服务器：

```shell
shell> mysql
```

连接成功后，你可以随时在“mysql>”提示符下键入quit（或\q、exit）断开连接。

在Unix系统上，你也可以通过Control+D快捷键断开连接。

以下各节的大部分示例都假设你已经连接到数据库。他们通过“mysql>”提示符来表明。

## 3.2 输入查询

确保你已经连接到服务器，如上节所述。这样做本身没有选择任何数据库，但是可行的。在这一点上，了解有关如何发出查询的知识比直接跳转到创建表、将数据加载到表中以及从表中检索数据更重要。本节介绍输入查询的基本准则，你可以尝试几种服务器查询来熟悉mysql的工作方式。

这里有一个简单的查询，要求服务器告诉你它的版本号和当前日期。在“mysql>”提示符后输入以下内容并按回车：

```sql
mysql> select version(),current_date;
+-----------+--------------+
| version() | current_date |
+-----------+--------------+
| 8.0.22    | 2021-04-23   |
+-----------+--------------+
1 row in set (0.00 sec)

mysql>
```

这个查询说明了mysql的几个方面：

- 一个查询一般包括一条SQL语句和一个分号。（也有一些可以省略分号的特殊情况，quit就是其中之一。）
- 当你发送一个查询，mysql将其发送到服务器上执行，并显示结果，然后打印另一个“mysql>”提示符，准备接受另一个查询。
- mysq以表格格式显示查询输出（行和列）。第一行包含各列的标签。接下来的行是查询结果。通常，列标签是你从数据库表中获取的列名，如果要检索表达式而不是一个表中列的值（上面例子所示），则mysql使用表达式本身标记该列。
- mysql会显示返回了多少列和查询执行了多长时间，这是你对服务器性能有个大概的了解。这些值是不精确的，因为他们表示的是wall clock时间（而不是cpu或机器时间），而且他们会受到诸如服务器负载和网络延迟等因素的影响。（为了方便，“rows in set”这一行有时不会在本章后面的例子中出现。）

关键字可以任意大小写输入，比如下面的查询是等价的：

```sql
mysql> SELECT VERSION(), CURRENT_DATE;
mysql> select version(), current_date;
mysql> SeLeCt vErSiOn(), current_DATE;
```

这是另外一个查询，它演示了你可以将mysql用作一个简单的计算器。

```sql
mysql> SELECT SIN(PI()/4), (4+1)*5;
+------------------+---------+
| SIN(PI()/4)      | (4+1)*5 |
+------------------+---------+
| 0.70710678118655 |      25 |
+------------------+---------+
1 row in set (0.02 sec)
```

目前为止展示的查询都是相对简短的、单行的语句。你也可以在一行中输入多条语句，只需将他们用分号结尾。

```sql
mysql> SELECT VERSION(); SELECT NOW();
+-----------+
| VERSION() |
+-----------+
| 8.0.13    |
+-----------+
1 row in set (0.00 sec)

+---------------------+
| NOW()               |
+---------------------+
| 2018-08-24 00:56:40 |
+---------------------+
1 row in set (0.00 sec)
```

一个查询不需要在一行中给出，因此需要几行的冗长查询不是问题。mysql通过查找终止符分号来确定语句的结尾，而不是输入行的末尾。（换句话说，mysql接受自由格式的输入：先收集输入行，直到遇到分号才会执行。）

下面是一个简单的多行语句：

```sql
mysql> SELECT
    -> USER()
    -> ,
    -> CURRENT_DATE;
+---------------+--------------+
| USER()        | CURRENT_DATE |
+---------------+--------------+
| jon@localhost | 2018-08-24   |
+---------------+--------------+
```

在这个例子中，在你输入多行查询语句中的一行语句后，注意提示符是如何从mysql>变到->的。这是mysql在提示语句还未完整，并在等待其余语句。提示符是你的朋友，因为它会提供有价值的反馈。如果你利用这个反馈，则你始终可以知道mysql正在等待什么。

如果你正在输入过程中，突然不想执行这条语句了，可以通过输入\c来取消执行。

```sql
mysql> SELECT
    -> USER()
    -> \c
mysql>
```

同样注意这里的提示符，在输入\c之后它变回了mysql>，提供了反馈，提示mysql准备接收新的查询。

下表显示的是你可能会遇到的每个提示符，并总结了他们对于mysql所处状态的含义。

| Prompt | Meaning                                                      |
| ------ | ------------------------------------------------------------ |
| mysql> | Ready for new query                                          |
| ->     | Waiting for next line of multiple-line query                 |
| '>     | Waiting for next line, waiting for completion of a string that began with a single quote (') |
| ">     | Waiting for next line, waiting for completion of a string that began with a double quote (") |
| `>     | Waiting for next line, waiting for completion of an identifier that began with a backtick (`) |
| /*>    | Waiting for next line, waiting for completion of a comment that began with /* |

当你打算在单行上发出查询时，但是忘记了终止符分号，多行语句通常就是在这种情况下偶然出现的。在此情况下，mysql等待更多的输入：

```sql
mysql> SELECT USER()
    ->
```

如果上述情况发生了（你觉得你已经输入完了一条语句，但只得到了->提示符的回应），则很可能mysql正在等待分号。如果你不注意提示符告诉你的信息，你可能会等待很久，直到意识到你需要做点什么。输入一个分号来完成语句的输入，然后mysql会执行它：

```sql
mysql> SELECT USER()
    -> ;
+---------------+
| USER()        |
+---------------+
| jon@localhost |
+---------------+
```

'>和">提示符一般出现在字符串收集过程中（另一种说法是MySQL正在等待完整的字符串）。在MySQL中，你可以用单引号或双引号将字符串围绕（比如'hello'或"goodbye"），而且mysql允许输入的字符串分散在多行中。当你看到'>或">提示符时，意味着你输入了一个以单引号或双引号开始的字符串，但还未输入用来结束该字符串的匹配的引号。这通常表明你无意间忽略了一个引号。比如：

```sql
mysql> SELECT * FROM my_table WHERE name = 'Smith AND age < 30;
    '>
```

如果你输入了这条select语句，然后按下回车并等待结果，什么都没发生。不要好奇为什么这个查询如此耗时，而要注意'>提示符给出的提示。它告诉你mysql期待看到一个未终止字符串的剩余部分。（你注意到上面语句中的错误了吗？'Smith字符串丢失了第二个单引号。）

此时你会怎么做？最简单的就是取消这个查询。然而，你不能仅仅输入\c，因为mysql会将其解释为其收集字符串的一部分。而是先要输入第二个单引号（这样mysql就知道你已经完成了字符串的输入），然后输入\c。

```sql
mysql> SELECT * FROM my_table WHERE name = 'Smith AND age < 30;
    '> '\c
mysql>
```

提示符变成了mysql>，表明mysql已经准备好接收新的查询。

提示符`>与'>和">类似，表示你已经开始但未完成反引号的标识符。

知道'>">`>提示符表示的含义很重要，因为如果你错误的输入了一个未终止的字符串，那你后面输入的所有的行都可能会被mysql忽略——包括quit。这可能会造成混乱，尤其是当你不知道你需要提供结束的引号才能取消当前查询的时候。

> Note
>
> 从现在开始，多行语句将在没有辅助提示符的情况下编写，从而使复制和粘贴语句更容易，便于你尝试。

## 3.3 创建并使用数据库

*3.3.1 创建并选择数据库*

*3.3.2 创建表*

*3.3.3 向表中加载数据*

*3.3.4 从表中检索数据*



一旦你知道如何输入SQL语句，就可以访问数据库了。

假如你家中有一些宠物（你的野生动物），你想跟踪有关他们的不同类型的信息。想做到这一点，你可以创建表来保存你的数据，并加载所需要的信息。然后你可以通过从表中检索数据，来回答有关你动物的不同问题。本节向你展示如何实现以下操作：

- 创建数据库
- 创建表
- 向表中加载数据
- 以不同方法从表中检索数据
- 使用多个表

这个动物数据库很简单（刻意的），但不难想象现实世界中使用类似数据库的情况。比如，一个农民用类似数据库追踪家畜信息，或者兽医追钟病人记录。从MySQL网站可以获得后面章节中用到的包含一些查询和简单数据的动物分布（ A menagerie distribution）。位于[https://dev.mysql.com/doc/](https://dev.mysql.com/doc/.)，tar和Zip压缩格式都可用。

使用show语句获取服务器上现存的数据库：

```sql
mysql> SHOW DATABASES;
+----------+
| Database |
+----------+
| mysql    |
| test     |
| tmp      |
+----------+
```

这个mysql数据库介绍用户访问权限。test数据库被用作用户进行尝试的工作区。

在你的机器上show语句显示的数据库列表可能不一样；show databases语句不会显示你没有SHOW DATABASES权限的数据库，见13.7.7.14节 SHOW DATABASES语句。

如果test数据库存在，尝试访问它：

```sql
mysql> USE test
Database changed
```

use与quit一样，不需要分号。（如果你愿意，你也可以加上分号来终止此类语句，无妨）。另一方面来说，use语句很特殊，它必须在单行中给出。

你可以在接下来的例子中使用test数据库（如果你已经访问它），但你在这个数据库下可以被任何访问它的用户移除。为此，你应该向MySQL管理员寻求使用你自己数据库的权限。假如你想让自己的数据库名为menagerie，管理员需要执行这样的语句：

```sql
mysql> GRANT ALL ON menagerie.* TO 'your_mysql_name'@'your_client_host';
```

your_mysql_name是分配给你的MySQL用户名，your_client_host是你用来连接服务器的机器的主机。

### 3.3.1 创建并选择数据库

如果管理员设置你的权限是，为你创建了数据库，你可以开始使用它。否则，你需要自己创建：

```sql
mysql> CREATE DATABASE menagerie;
```

在Unix系统下，数据库名是区分大小写的（不像SLQL关键字），所以你必须总是以menagerie来引用你的数据库，不能使用Menagerie、MENAGERIE或其他变种。表名也是如此。（在Windows下，此限制并不适用，尽管你必须在整个给定查询中使用相同的字母大写来引用数据库和表。但是，由于多种原因，建议的最佳实践始终是使用与数据库创建时相同情况的字母大写。）

> Note
>
> 当你尝试创建数据库时，如果得到一个错误信息比如ERROR 1044 (42000): Access denied for user 'micah'@'localhost' to database 'menagerie' ，意味着你的账户没有执行此操作所需要的权限。与管理员讨论或见6.2节 访问控制和账户管理。

创建数据库并不意味着已经选择并使用它，你必须明确的做到这一点。使用menagerie作为当前数据库，使用以下语句：

```sql
mysql> USE menagerie
Database changed
```

你的数据库只需创建一次，但你每次开始一个mysql会话时都必须选择该数据库以供使用。可以像上面的例子中那样使用use语句来做到这一点。或者你也可以在用命令行调用mysql命令时选择数据库。只需要在需要提供的连接参数后面指定数据库的名字，比如：

```shell
shell> mysql -h host -u user -p menagerie
Enter password: ********
```

> Important
>
> 命令中的menagerie不是你的密码。如果你想在命令行中-p选项后面提供密码，则-p和密码之间不能有空格（as -ppassword,  not as -p password）。但是不推荐在命令行中输入密码，因为这会将你的密码暴露给登录到你机器上的其他用户。

> Note
>
> 你可以使用select database()语句查看当前数据库。

### 3.3.2 创建表

创建数据库很简单，但此时show tables语句告诉你数据库是空的：

```sql
mysql> SHOW TABLES;
Empty set (0.00 sec)
```

难点在于决定你的数据库的结构应该是怎样的：需要那些表，每个表中应该有哪些列。

你想要一个包含你的每个宠物记录的表。此表可称为pet表，并且至少要包含每个动物的名字。由于名字本身不是很有趣，所以表还应该包含其他信息。比如，如果你家有多人都养宠物，你可能希望列出每个宠物的主人。你可能还希望记录诸如品种和性别等基础描述性信息。

年龄呢？年龄可能挺有趣，但它不适合存储到数据库里。年龄会随时间变化，意味着你必须经常更新你的记录。相反，最好存储一个诸如生日的固定值。然后无论你何时需要年龄，你可以计算当前日期和生日的差值。MySQL提供了日期计算函数，所以这不难。存储生日相比年龄还有其他优势，比如：

- 你可以将数据库用于某些任务，比如为即将到来的宠物生日生成提醒。（如果你觉得此类查询有些愚蠢，请注意，在业务数据库中，你可能会问相同的问题，比如明确在当前周或当月你需要向哪些客户发送生日祝福，这类计算机辅助的personal touch。）
- 你可可以用当前日期以外的日期来计算年龄。比如，如果你在数据库中存储了死亡日期，你可以轻松的计算宠物死亡时的年龄。

你也可以考虑在pet表中其他类型的有用的信息，但目前已经确认可行的有：姓名、主人、品种、性别、出生日期和死亡日期。

使用create table语句来指定表的结构：

```sql
mysql> CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),
       species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
```

varchar对姓名、主人和品种列来说是个好的选择，因为列值的长度会不一样。这些列值长度的定义不需要相同，不需要是20。你通常可以选择从1到65535的任何合理的长度。如果选择不当，后来又发现需要更长的取值，MySQL提供alter table语句可供使用。

可以选择几类值来代表动物记录的性别，比如m和f，male和female。最简洁的是使用单个字符m和f。

对于出和死亡列选用data数据类型是相当明显的选择。（fairly obvious choice）

一旦你创建了一个条，show tables语句就会产生一些输出：

```sql
mysql> SHOW TABLES;
+---------------------+
| Tables in menagerie |
+---------------------+
| pet                 |
+---------------------+
```

想要核实表是按照你所期待的方式建立的，可以使用describe语句：

```sql
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
```

你可以随时使用describe语句，比如你忘记饿了表中列的名字或类型。

更多关于MySQL数据类型的信息，见第11章 数据类型。

### 3.3.3 向表中加载数据

创建表后，你需要填充它，load data语句和insert语句可用于此。

假设宠物记录如下描述。（注意，MySQL的日期格式为'YYYY-MM-DD'，可能跟你以前用的不一样。）

| **name** | **owner** | **species** | **sex** | **birth**  | **death**  |
| -------- | --------- | ----------- | ------- | ---------- | ---------- |
| Fluffy   | Harold    | cat         | f       | 1993-02-04 |            |
| Claws    | Gwen      | cat         | m       | 1994-03-17 |            |
| Buffy    | Harold    | dog         | f       | 1989-05-13 |            |
| Fang     | Benny     | dog         | m       | 1990-08-27 |            |
| Bowser   | Diane     | dog         | m       | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen      | bird        | f       | 1998-09-11 |            |
| Whistler | Gwen      | bird        |         | 1997-12-09 |            |
| Slim     | Benny     | snake       | m       | 1996-04-29 |            |

因为你是从一个空表开始的，一个简单的填充方法是先创建一个text文件，此文件中的每行数据对应每个动物，然后用一条语句将文件中的内容加载到表中。

你可以创建pet.txt文件，每行包含一条记录，值之间用tab符分割，然后按照create table语句中列出的列的顺序，给出每列数据。对于丢失的值（比如为止的性别和还活着的动物的死亡日期），你可以用null值。在text文件中，用\N代表null值。比如，Whistler这条记录应该像下面这样（值之间的空白是一个tab符）：

```
Whistler        Gwen    bird    \N      1997-12-09      \N
```

使用如下语句，将pet.txt文件的内容加载到pet表中：

```sql
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;
```

如果您在Windows上使用\r\n用作行终止符的编辑器创建了文件 ，则应改用以下语句：

```sql
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet
       LINES TERMINATED BY '\r\n';
```

（在一台运行macOS的苹果机器上，你可能会想将\r用作行终止符。）

你可以根据需要在load data语句中显式指定分隔符和行尾标记，但默认是tab符和换行符。这足以使语句正确的读取pet.txt文件。

如果语句失败了，可能是因为你的MySQL安装默认未启用本地文件功能，见6.1.6节 load data local语句的安全注意事项，查阅如何改变该默认值。

如果你想每次添加一条新纪录，可以使用inser语句。以最简单的形式，你可以按create table语句中列出列的顺序为每一列提供值。假设Diane得到了一直名为Puffball的新仓鼠，你可以向下面这样添加一条新纪录：

```sql
mysql> INSERT INTO pet
       VALUES ('Puffball','Diane','hamster','f','1999-03-30',NULL);
```

字符串和日期值此处被指定为带引号的字符串。而且用insert语句，可以将null值作为丢失值直接插入。你不需要向load data语句中那样使用\N。

在此例子中你应该能看到，最初使用多个INSERT语句而不是单个LOAD DATA 语句来加载记录时会涉及更多类型的输入。

### 3.3.4 从表中检索数据

*3.3.4.1 选择所有数据*

*3.3.4.2 选择指定行*

*3.3.4.3 选择指定列*

*3.3.4.4 排序行*

*3.3.4.5 日期计算*

*3.3.4.6 使用NULL值*

*3.3.4.7 模式匹配*

*3.3.4.8 计算行数*

*3.3.4.9 使用多个表*



select语句用来从表中提取数据，该语句的一般用法为：

```sql
SELECT what_to_select
FROM which_table
WHERE conditions_to_satisfy;
```

what_to_select指示您要查看的内容。可以是列的列表，也可以是*，表示所有列。 which_table表示要从中检索数据的表。该WHERE 子句是可选的。如果存在，conditions_to_satisfy指定行必须满足的一个或多个检索条件。

#### 3.3.4.1 选择所有数据

select语句最简单的格式是检索表里的一切：

```sql
mysql> SELECT * FROM pet;
+----------+--------+---------+------+------------+------------+
| name     | owner  | species | sex  | birth      | death      |
+----------+--------+---------+------+------------+------------+
| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |
| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |
| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |
| Fang     | Benny  | dog     | m    | 1990-08-27 | NULL       |
| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |
| Whistler | Gwen   | bird    | NULL | 1997-12-09 | NULL       |
| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |
| Puffball | Diane  | hamster | f    | 1999-03-30 | NULL       |
+----------+--------+---------+------+------------+------------+
```

select * 格式是select所有列的简写。比如在将原始数据集导入之后，如果你想检索整个表，那这很有用。比如你可能突然觉得Bowser的生日看起来不太对。查阅原始的动物血统记录后，你发现正确的出生年份是1989，而不是1979。

至少有两种办法解决此问题：

- 编辑pet.txt文件以修改错误，然后使用delete和load data语句清空并重新加载pet表：

```sql
mysql> DELETE FROM pet;
mysql> LOAD DATA LOCAL INFILE 'pet.txt' INTO TABLE pet;
```

但是，如果这样做，你还必须重新输入Puffball记录。

- 用update语句只修改错误的记录

```sql
mysql> UPDATE pet SET birth = '1989-08-31' WHERE name = 'Bowser';
```

update语句只修改有问题的语句，不需要重新加载整个表。

select 有一个例外情况。如果表包含隐藏列，不会包含他们。更多信息，见13.1.20.10 隐藏列。

#### 3.3.4.2 选择特定行

如前面章节所示，检索整个表很简单，只需要忽略select语句中的where子句。但通常你不想看到整个表，特别是表很大时。相反，你通常对回答一些特定的问题更感兴趣，在此情况下，你可以对你所需要的信息指定一些约束。让我们从与宠物有关的问题的角度来看一些选择查询。

你可以从表中只选择特定的行，比如，如果你想核实Bowser的出生日期的变更，你可以选择Bowser的记录，如下：

```sql
mysql> SELECT * FROM pet WHERE name = 'Bowser';
+--------+-------+---------+------+------------+------------+
| name   | owner | species | sex  | birth      | death      |
+--------+-------+---------+------+------------+------------+
| Bowser | Diane | dog     | m    | 1989-08-31 | 1995-07-29 |
+--------+-------+---------+------+------------+------------+
```

输出确认该年的正确记录时1989，不是1979。

字符串比较通常不区分大小写，因此你可以将name指定为bowser、BOWSER等等，查询结果相同。

你可以在任意列上指定条件，不只是name列。比如，你想知道哪些动物是在1998年及之后出生的，那就测试birth列：

```sql
mysql> SELECT * FROM pet WHERE birth >= '1998-1-1';
+----------+-------+---------+------+------------+-------+
| name     | owner | species | sex  | birth      | death |
+----------+-------+---------+------+------------+-------+
| Chirpy   | Gwen  | bird    | f    | 1998-09-11 | NULL  |
| Puffball | Diane | hamster | f    | 1999-03-30 | NULL  |
+----------+-------+---------+------+------------+-------+
```

你可以组合这些条件，比如定位雌性狗：

```sql
mysql> SELECT * FROM pet WHERE species = 'dog' AND sex = 'f';
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
```

上面的查询使用了and逻辑运算符，也有or运算符：

```sql
mysql> SELECT * FROM pet WHERE species = 'snake' OR species = 'bird';
+----------+-------+---------+------+------------+-------+
| name     | owner | species | sex  | birth      | death |
+----------+-------+---------+------+------------+-------+
| Chirpy   | Gwen  | bird    | f    | 1998-09-11 | NULL  |
| Whistler | Gwen  | bird    | NULL | 1997-12-09 | NULL  |
| Slim     | Benny | snake   | m    | 1996-04-29 | NULL  |
+----------+-------+---------+------+------------+-------+
```

and和or可以混合使用，尽管and的优先级比or高。如果你同时用这两种运算符，最好使用括号明确的指定条件如何分组：

```sql
mysql> SELECT * FROM pet WHERE (species = 'cat' AND sex = 'm')
       OR (species = 'dog' AND sex = 'f');
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Claws | Gwen   | cat     | m    | 1994-03-17 | NULL  |
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
```

#### 3.3.4.3 选择特定列

如果你不想看到表中的整个行，你可以只给出你感兴趣的列的名字，用逗号分隔。比如，你想知道你的动物是何时出生的，选择name和birth列：

```sql
mysql> SELECT name, birth FROM pet;
+----------+------------+
| name     | birth      |
+----------+------------+
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Buffy    | 1989-05-13 |
| Fang     | 1990-08-27 |
| Bowser   | 1989-08-31 |
| Chirpy   | 1998-09-11 |
| Whistler | 1997-12-09 |
| Slim     | 1996-04-29 |
| Puffball | 1999-03-30 |
+----------+------------+
```

为了找到谁拥有宠物，使用下面的查询：

```sql
mysql> SELECT owner FROM pet;
+--------+
| owner  |
+--------+
| Harold |
| Gwen   |
| Harold |
| Benny  |
| Diane  |
| Gwen   |
| Gwen   |
| Benny  |
| Diane  |
+--------+
```

注意，这个查询只是从每条记录中简单的检索了owner列，有些记录出现了两次或更多。为了最大减少输出，可以通过添加distinct关键字，检索每个唯一的输出记录只出现一次。

```sql
mysql> SELECT DISTINCT owner FROM pet;
+--------+
| owner  |
+--------+
| Benny  |
| Diane  |
| Gwen   |
| Harold |
+--------+
```

你可以用where子句来组合行选择和列选择。比如，只得到狗和猫的生日，用一下查询：

```sql
mysql> SELECT name, species, birth FROM pet
       WHERE species = 'dog' OR species = 'cat';
+--------+---------+------------+
| name   | species | birth      |
+--------+---------+------------+
| Fluffy | cat     | 1993-02-04 |
| Claws  | cat     | 1994-03-17 |
| Buffy  | dog     | 1989-05-13 |
| Fang   | dog     | 1990-08-27 |
| Bowser | dog     | 1989-08-31 |
+--------+---------+------------+
```

#### 3.3.4.4 排序行

你可能注意到之前的例子中，结果行都是无序显示的。当以某种有意义的方式对行进行排序时，通常更容易检查查询输出。要对结果进行排序，可使用ORDER BY子句。

这里是按日期排序的动物生日：

```sql
mysql> SELECT name, birth FROM pet ORDER BY birth;
+----------+------------+
| name     | birth      |
+----------+------------+
| Buffy    | 1989-05-13 |
| Bowser   | 1989-08-31 |
| Fang     | 1990-08-27 |
| Fluffy   | 1993-02-04 |
| Claws    | 1994-03-17 |
| Slim     | 1996-04-29 |
| Whistler | 1997-12-09 |
| Chirpy   | 1998-09-11 |
| Puffball | 1999-03-30 |
+----------+------------+
```

在字符串类型的列上的排序，与其他类型的比较操作一样，通常不区分大小写。这意味着除了大小写相同外，其他列均为定义顺序（This means that the order is undefined for columns that are identical except for their case. ）。你可以使用binary关键字，强制对列进行区分大小写的排序，像这样：ORDER BY BINARY col_name。

默认排序是升序，最小的值在最前面。为了升序排列，需要在排序的列名后面加上DESC关键字：

```sql
mysql> SELECT name, birth FROM pet ORDER BY birth DESC;
+----------+------------+
| name     | birth      |
+----------+------------+
| Puffball | 1999-03-30 |
| Chirpy   | 1998-09-11 |
| Whistler | 1997-12-09 |
| Slim     | 1996-04-29 |
| Claws    | 1994-03-17 |
| Fluffy   | 1993-02-04 |
| Fang     | 1990-08-27 |
| Bowser   | 1989-08-31 |
| Buffy    | 1989-05-13 |
+----------+------------+
```

你可以对多个列排序，每个列可以有不同的排序方向。比如，动物类型按升序排列，生日按降序排列（最年轻的动物在最前面），使用下面的查询：

```sql
mysql> SELECT name, species, birth FROM pet
       ORDER BY species, birth DESC;
+----------+---------+------------+
| name     | species | birth      |
+----------+---------+------------+
| Chirpy   | bird    | 1998-09-11 |
| Whistler | bird    | 1997-12-09 |
| Claws    | cat     | 1994-03-17 |
| Fluffy   | cat     | 1993-02-04 |
| Fang     | dog     | 1990-08-27 |
| Bowser   | dog     | 1989-08-31 |
| Buffy    | dog     | 1989-05-13 |
| Puffball | hamster | 1999-03-30 |
| Slim     | snake   | 1996-04-29 |
+----------+---------+------------+
```

DESC关键字仅适用于紧挨着它前面的列（birth），不会影响species列的排序顺序。

#### 3.3.4.5 日期计算

MySQL提供了几个可以用于日期计算的函数，比如，计算年龄或提取日期的一部分。

为计算你每个宠物的年龄，可使用TIMESTAMPDIFF()函数。他的参数是要结果要表示的单位，以及两个日期的差值。以下查询显示了每个宠物的出生日期、当前日期和年龄（以年为单位）。别名（age）用来使最后的输出列标签更有意义。

```sql
mysql> SELECT name, birth, CURDATE(),
       TIMESTAMPDIFF(YEAR,birth,CURDATE()) AS age
       FROM pet;
+----------+------------+------------+------+
| name     | birth      | CURDATE()  | age  |
+----------+------------+------------+------+
| Fluffy   | 1993-02-04 | 2003-08-19 |   10 |
| Claws    | 1994-03-17 | 2003-08-19 |    9 |
| Buffy    | 1989-05-13 | 2003-08-19 |   14 |
| Fang     | 1990-08-27 | 2003-08-19 |   12 |
| Bowser   | 1989-08-31 | 2003-08-19 |   13 |
| Chirpy   | 1998-09-11 | 2003-08-19 |    4 |
| Whistler | 1997-12-09 | 2003-08-19 |    5 |
| Slim     | 1996-04-29 | 2003-08-19 |    7 |
| Puffball | 1999-03-30 | 2003-08-19 |    4 |
+----------+------------+------------+------+
```

此查询有效，但如果以某种顺序显示行，则可以更轻松地查看结果。这时可以添加ORDER BY name子句，将输出结果按name排序：

```sql
mysql> SELECT name, birth, CURDATE(),
       TIMESTAMPDIFF(YEAR,birth,CURDATE()) AS age
       FROM pet ORDER BY name;
+----------+------------+------------+------+
| name     | birth      | CURDATE()  | age  |
+----------+------------+------------+------+
| Bowser   | 1989-08-31 | 2003-08-19 |   13 |
| Buffy    | 1989-05-13 | 2003-08-19 |   14 |
| Chirpy   | 1998-09-11 | 2003-08-19 |    4 |
| Claws    | 1994-03-17 | 2003-08-19 |    9 |
| Fang     | 1990-08-27 | 2003-08-19 |   12 |
| Fluffy   | 1993-02-04 | 2003-08-19 |   10 |
| Puffball | 1999-03-30 | 2003-08-19 |    4 |
| Slim     | 1996-04-29 | 2003-08-19 |    7 |
| Whistler | 1997-12-09 | 2003-08-19 |    5 |
+----------+------------+------------+------+
```

如果想用age来排序输出结果，而不是name，那就用不同的ORDER BY子句：

```sql
mysql> SELECT name, birth, CURDATE(),
       TIMESTAMPDIFF(YEAR,birth,CURDATE()) AS age
       FROM pet ORDER BY age;
+----------+------------+------------+------+
| name     | birth      | CURDATE()  | age  |
+----------+------------+------------+------+
| Chirpy   | 1998-09-11 | 2003-08-19 |    4 |
| Puffball | 1999-03-30 | 2003-08-19 |    4 |
| Whistler | 1997-12-09 | 2003-08-19 |    5 |
| Slim     | 1996-04-29 | 2003-08-19 |    7 |
| Claws    | 1994-03-17 | 2003-08-19 |    9 |
| Fluffy   | 1993-02-04 | 2003-08-19 |   10 |
| Fang     | 1990-08-27 | 2003-08-19 |   12 |
| Bowser   | 1989-08-31 | 2003-08-19 |   13 |
| Buffy    | 1989-05-13 | 2003-08-19 |   14 |
+----------+------------+------------+------+
```

一个类似的查询可以计算一个死了的动物在死的时候的年龄，这要检查death列是否是null。然后，对这些非null值，计算death和birth值的差值：

```sql
mysql> SELECT name, birth, death,
       TIMESTAMPDIFF(YEAR,birth,death) AS age
       FROM pet WHERE death IS NOT NULL ORDER BY age;
+--------+------------+------------+------+
| name   | birth      | death      | age  |
+--------+------------+------------+------+
| Bowser | 1989-08-31 | 1995-07-29 |    5 |
+--------+------------+------------+------+
```

该查询中使用了death is not null而不是，death<>null，因为null是一个特殊的值，不能使用一般的比较运算符进行比较，后面将会讨论。见3.3.4.6节 使用null值。

如果你想知道哪些动物在下个月生日呢？对于这种类型的计算，年和日是不相关的，你只是想提取birth列的月份部分。MySQL提供了几个用于提取部分日期的函数，比如YEAR()、MONTH()以及DATOFMONTH()。这里适合的函数是MONTH()。为了知道它是如何工作的，运行一个简单的查询，来显示birth和MONTH(birth)的值：

```sql
mysql> SELECT name, birth, MONTH(birth) FROM pet;
+----------+------------+--------------+
| name     | birth      | MONTH(birth) |
+----------+------------+--------------+
| Fluffy   | 1993-02-04 |            2 |
| Claws    | 1994-03-17 |            3 |
| Buffy    | 1989-05-13 |            5 |
| Fang     | 1990-08-27 |            8 |
| Bowser   | 1989-08-31 |            8 |
| Chirpy   | 1998-09-11 |            9 |
| Whistler | 1997-12-09 |           12 |
| Slim     | 1996-04-29 |            4 |
| Puffball | 1999-03-30 |            3 |
+----------+------------+--------------+
```

找到下个月出生的动物也很简单。假设当前月是4月，你可以像下面这样找到5月出生的动物：

```sql
mysql> SELECT name, birth FROM pet WHERE MONTH(birth) = 5;
+-------+------------+
| name  | birth      |
+-------+------------+
| Buffy | 1989-05-13 |
+-------+------------+
```

如果当前月份是12月，那就会有点小问题。你不能只是对12月进行加1操作，然后找到13月出生的动物，因为没有这样的月份。相反，你应该找在1月份出生的动物。

你可以写一个不管当前月份是多少，都可以正常工作的查询，这样你就不必使用一个特定的月份数。DATE_ADD()函数可以在一个给定的日期上添加一个时间间隔。如果CURDATE()的值加一个月，然后用MONTH()提取月份部分，结果就是我们要找的月份：

```sql
mysql> SELECT name, birth FROM pet
       WHERE MONTH(birth) = MONTH(DATE_ADD(CURDATE(),INTERVAL 1 MONTH));
```

实现此任务的另一个方法是先提取当前日期的月份部分，如果当前月份是12，然后用取模函数MOD()对其包装成0，然后对其加1。

```sql
mysql> SELECT name, birth FROM pet
       WHERE MONTH(birth) = MOD(MONTH(CURDATE()), 12) + 1;
```

MONTH()返回一个1到12之间的数。MOD(something, 12)返回一个0到11的数。所以加1操作必须在MOD()之后，否则我们将从11月到1月（11月经过上述公式后得到1月）。

如果用了无效的日期，运算会失败并产生警告。

```sql
mysql> SELECT '2018-10-31' + INTERVAL 1 DAY;
+-------------------------------+
| '2018-10-31' + INTERVAL 1 DAY |
+-------------------------------+
| 2018-11-01                    |
+-------------------------------+
mysql> SELECT '2018-10-32' + INTERVAL 1 DAY;
+-------------------------------+
| '2018-10-32' + INTERVAL 1 DAY |
+-------------------------------+
| NULL                          |
+-------------------------------+
mysql> SHOW WARNINGS;
+---------+------+----------------------------------------+
| Level   | Code | Message                                |
+---------+------+----------------------------------------+
| Warning | 1292 | Incorrect datetime value: '2018-10-32' |
+---------+------+----------------------------------------+
```

#### 3.3.4.6 使用NULL值

在你习惯使用它之前，NULL值可能是令人惊讶的。从概念上讲，NULL值意味着缺少的未知值，与其他类型的值处理方法不同。

为测试NULL值，使用IS NULL和IS NOT NULL，如下所示：

```sql
mysql> SELECT 1 IS NULL, 1 IS NOT NULL;
+-----------+---------------+
| 1 IS NULL | 1 IS NOT NULL |
+-----------+---------------+
|         0 |             1 |
+-----------+---------------+
```

你不能使用诸如=、<和<>的算数比较操作符来测试NULL。为证实，尝试以下查询：

```sql
mysql> SELECT 1 = NULL, 1 <> NULL, 1 < NULL, 1 > NULL;
+----------+-----------+----------+----------+
| 1 = NULL | 1 <> NULL | 1 < NULL | 1 > NULL |
+----------+-----------+----------+----------+
|     NULL |      NULL |     NULL |     NULL |
+----------+-----------+----------+----------+
```

因为使用任何值与NULL进行算术比较结果都为NULL，此类查询没法得到有意义的结果。

MySQL中，0或NULL表示假，其他值都表示真。布尔操作默认的真值是1。

上述解释了为什么要对NULL进行特殊处理，用death IS NULL而不是death<>NULL来计算动物是否还活着是很有必要的。

两个NULL在GROUP BY子句中被认为是相等的。

当执行ORDER BY操作时，如果是升序操作，那么NULL会放到最前面，如果是降序操作，NULL会放到最后面。

使用NULL值时，一个常见的错误是假定没法将0或空字符串插入到一个非空列中，这是不对的，这些都是实际的值，然而NULL的意思是没有值。可以使用IS NULL和IS NOT NULL来测试下：

```sql
mysql> SELECT 0 IS NULL, 0 IS NOT NULL, '' IS NULL, '' IS NOT NULL;
+-----------+---------------+------------+----------------+
| 0 IS NULL | 0 IS NOT NULL | '' IS NULL | '' IS NOT NULL |
+-----------+---------------+------------+----------------+
|         0 |             1 |          0 |              1 |
+-----------+---------------+------------+----------------+
```

所以向一个非空列中插入0或空字符串是完全可以的，因为0和空字符串并不是空值，见附录B3.4.3节 NULL值的问题。

#### 3.3.4.7 模式匹配

MySQL提供标准的SQL模式匹配，以及基于扩展的正则表达式的模式匹配形式，类似于vi、grep和sed之类的unix实用程序使用的扩展正则表达式。

SQL模式匹配可以使你用“_”匹配任意单个字符，使用“%”匹配任意个字符长度的字符串，包括零个字符。MySQL中，SQL模式默认不区分大小写。下面的例子说明了这一点。SQL模式中不要使用=和<>，应使用LIKE或NOT LIKE比较运算符。

以下查询以b开头的name列的行：

```sql
mysql> SELECT * FROM pet WHERE name LIKE 'b%';
+--------+--------+---------+------+------------+------------+
| name   | owner  | species | sex  | birth      | death      |
+--------+--------+---------+------+------------+------------+
| Buffy  | Harold | dog     | f    | 1989-05-13 | NULL       |
| Bowser | Diane  | dog     | m    | 1989-08-31 | 1995-07-29 |
+--------+--------+---------+------+------------+------------+
```

以下查询以fy结尾的name列的行：

```sql
mysql> SELECT * FROM pet WHERE name LIKE '%fy';
+--------+--------+---------+------+------------+-------+
| name   | owner  | species | sex  | birth      | death |
+--------+--------+---------+------+------------+-------+
| Fluffy | Harold | cat     | f    | 1993-02-04 | NULL  |
| Buffy  | Harold | dog     | f    | 1989-05-13 | NULL  |
+--------+--------+---------+------+------------+-------+
```

以下查询包含w的name列的行：

```sql
mysql> SELECT * FROM pet WHERE name LIKE '%w%';
+----------+-------+---------+------+------------+------------+
| name     | owner | species | sex  | birth      | death      |
+----------+-------+---------+------+------------+------------+
| Claws    | Gwen  | cat     | m    | 1994-03-17 | NULL       |
| Bowser   | Diane | dog     | m    | 1989-08-31 | 1995-07-29 |
| Whistler | Gwen  | bird    | NULL | 1997-12-09 | NULL       |
+----------+-------+---------+------+------------+------------+
```

为查询name中包含五个字符的行，使用五个“_”：

```sql
mysql> SELECT * FROM pet WHERE name LIKE '_____';
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Claws | Gwen   | cat     | m    | 1994-03-17 | NULL  |
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
```

MySQL提供的另一种类型的模式匹配是使用扩展的正则表达式。当你测试这种模式的匹配时，要使用REGEXP_LIKE()函数（或者REGEXP、RLIKE运算符，与REGEXP_LIKE()函数是同义词）。

下面的列表描述了扩展正则表达式的特点：

- “.”匹配任意单个字符。
- 字符集[...]与括号内的所有字符匹配，比如[abc]可以匹配a、b或c。为确定一个字符范围，用破折号，[a-z]匹配所有字母，[0-9]匹配所有数字。
- *匹配在其之前的食物的零个或多个实例。比如x*匹配任意数量的x字符，[0-9]*匹配任意数量的数字，*匹配任意数量的任何东西。
- 如果一个模式匹配到了测试用例中的所有值，则该正则表达式就是模式匹配成功了。（这与LIKE模式匹配不同，LIKE模式匹配只有在匹配到整个值是才算匹配成功。）
- 为了使一个模式能匹配测试值的开头和结尾，用^匹配模式的开头，用$匹配模式的结尾。

为了找到b开头的名字，使用^匹配名字的开头：

```sql
mysql> SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b');
+--------+--------+---------+------+------------+------------+
| name   | owner  | species | sex  | birth      | death      |
+--------+--------+---------+------+------------+------------+
| Buffy  | Harold | dog     | f    | 1989-05-13 | NULL       |
| Bowser | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
+--------+--------+---------+------+------------+------------+
```

若要强制使正则表达式进行区分大小写的比较，可以使用区分大小写的排序规则，或者使用BINARY关键字将字符串中的一个设置为二进制字符串，或者指定模式控制字符“c”。下面的查询只匹配以小写字母b开头的名字：

```sql
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b' COLLATE utf8mb4_0900_as_cs);
SELECT * FROM pet WHERE REGEXP_LIKE(name, BINARY '^b');
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^b', 'c');
```

为找到以fy结尾的名字，使用$匹配名字结尾：

```sql
mysql> SELECT * FROM pet WHERE REGEXP_LIKE(name, 'fy$');
+--------+--------+---------+------+------------+-------+
| name   | owner  | species | sex  | birth      | death |
+--------+--------+---------+------+------------+-------+
| Fluffy | Harold | cat     | f    | 1993-02-04 | NULL  |
| Buffy  | Harold | dog     | f    | 1989-05-13 | NULL  |
+--------+--------+---------+------+------------+-------+
```

为找到包含一个w的名字，使用下列查询：

```sql
mysql> SELECT * FROM pet WHERE REGEXP_LIKE(name, 'w');
+----------+-------+---------+------+------------+------------+
| name     | owner | species | sex  | birth      | death      |
+----------+-------+---------+------+------------+------------+
| Claws    | Gwen  | cat     | m    | 1994-03-17 | NULL       |
| Bowser   | Diane | dog     | m    | 1989-08-31 | 1995-07-29 |
| Whistler | Gwen  | bird    | NULL | 1997-12-09 | NULL       |
+----------+-------+---------+------+------------+------------+
```

因为一个正则表达式会对值中任意位置出现的模式进行匹配，所以在前面的例子中没必要在模式两侧放置通配符以使其匹配整个值，这在SQL模式中也是正确的。

为找到包含五个字符的名字，使用^和$匹配名字的开头和结尾，中间用“.”：

```sql
mysql> SELECT * FROM pet WHERE REGEXP_LIKE(name, '^.....$');
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Claws | Gwen   | cat     | m    | 1994-03-17 | NULL  |
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
```

也可以使用{n}操作符（重复n次）重写前面的查询：

```sql
mysql> SELECT * FROM pet WHERE REGEXP_LIKE(name, '^.{5}$');
+-------+--------+---------+------+------------+-------+
| name  | owner  | species | sex  | birth      | death |
+-------+--------+---------+------+------------+-------+
| Claws | Gwen   | cat     | m    | 1994-03-17 | NULL  |
| Buffy | Harold | dog     | f    | 1989-05-13 | NULL  |
+-------+--------+---------+------+------------+-------+
```

更多关于正则表达式句法的信息，见12.8.2节 正则表达式。

#### 3.3.4.8 计算行数

数据据经常被用来回答这类问题：一种特定类型的数据在表中出现了多少次？比如，你可能想知道你有多少宠物，或者每个主人有多少宠物，或者你可能想对宠物进行各种普查操作。

计算你所拥有的宠物的数量，与宠物表中有多少行是同样的问题，因为每个宠物占一行记录。count(*)计算行数，所以统计你的宠物的查询如下：

```sql
mysql> SELECT COUNT(*) FROM pet;
+----------+
| COUNT(*) |
+----------+
|        9 |
+----------+
```

之前你检索了拥有宠物的人的名字。如果你想知道每个主人有多少宠物，可以使用count()：

```sql
mysql> SELECT owner, COUNT(*) FROM pet GROUP BY owner;
+--------+----------+
| owner  | COUNT(*) |
+--------+----------+
| Benny  |        2 |
| Diane  |        2 |
| Gwen   |        3 |
| Harold |        2 |
+--------+----------+
```

上面的查询使用GROUP BY将所有记录按照owner列进行分组。将count()和GROUP BY结合起来使用对于描述各个分组的数据很有用。

下面的例子展示了使用不同方式进行动物普查操作，每个品种动物的数量：

```sql
mysql> SELECT species, COUNT(*) FROM pet GROUP BY species;
+---------+----------+
| species | COUNT(*) |
+---------+----------+
| bird    |        2 |
| cat     |        2 |
| dog     |        3 |
| hamster |        1 |
| snake   |        1 |
+---------+----------+
```

每种性别动物数量：

```sql
mysql> SELECT sex, COUNT(*) FROM pet GROUP BY sex;
+------+----------+
| sex  | COUNT(*) |
+------+----------+
| NULL |        1 |
| f    |        4 |
| m    |        4 |
+------+----------+
```

（此处的输出中，NULL表示性别未知。）

每个品种和性别的组合中，动物的数量：

```sql
mysql> SELECT species, sex, COUNT(*) FROM pet GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| bird    | NULL |        1 |
| bird    | f    |        1 |
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
| hamster | f    |        1 |
| snake   | m    |        1 |
+---------+------+----------+
```

当你使用count()时，没必要检索整个表，比如对于上面的例子，你也可以只检索dogs和cats，如下：

```sql
mysql> SELECT species, sex, COUNT(*) FROM pet
       WHERE species = 'dog' OR species = 'cat'
       GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
+---------+------+----------+
```

或者你想知道性别已知的动物中，每种性别的动物有多少：

```sql
mysql> SELECT species, sex, COUNT(*) FROM pet
       WHERE sex IS NOT NULL
       GROUP BY species, sex;
+---------+------+----------+
| species | sex  | COUNT(*) |
+---------+------+----------+
| bird    | f    |        1 |
| cat     | f    |        1 |
| cat     | m    |        1 |
| dog     | f    |        1 |
| dog     | m    |        2 |
| hamster | f    |        1 |
| snake   | m    |        1 |
+---------+------+----------+
```

**除了选择count()值外，如果你还选择指定的列，则GROUP BY子句应该展现这个指定的列，否则将发生以下情况：**

- 如果启用了ONLY_FULL_GROUP_BY sql模式，将会发生错误：

```sql
mysql> SET sql_mode = 'ONLY_FULL_GROUP_BY';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT owner, COUNT(*) FROM pet;
ERROR 1140 (42000): In aggregated query without GROUP BY, expression
#1 of SELECT list contains nonaggregated column 'menagerie.pet.owner';
this is incompatible with sql_mode=only_full_group_by
```

- 如果禁用了ONLY_FULL_GROUP_BY，查询将会把所有的行视为一个分组，但是为每个指定的列检索出来的值是不确定的。服务器可以从任意行中自由选择值：

```sql
mysql> SET sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT owner, COUNT(*) FROM pet;
+--------+----------+
| owner  | COUNT(*) |
+--------+----------+
| Harold |        8 |
+--------+----------+
1 row in set (0.00 sec)
```

关于count(expr)的行为和相关优化，见12.20.3节 MySQL对count()的处理，和12.20.1节 统计函数介绍。

#### 3.3.4.9 使用多个表

pet表记录了你有哪些宠物。如果你还想记录关于他们的其他信息，比如看兽医和何时生的小宠物这类他们生命中的事件，你需要另一个表。这个表应该是什么样的？他需要包含以下信息：

- 宠物名，以便知道每个事件适用于哪个动物。
- 日期，以便知道事件是何时发生的。
- 一个描述时间的字段。
- 一个事件类型字段，如果你想将时间分类。

考虑到这些因素，event表的CREATE TABLE语句应该是这样的：

```sql
mysql> CREATE TABLE event (name VARCHAR(20), date DATE,
       type VARCHAR(15), remark VARCHAR(255));
```

与pet表一样，通过创建以tab为分隔符的包含以下信息的文本文件，将原始记录导入表中是很简单的。

| name     | date       | type     | remark                      |
| -------- | ---------- | -------- | --------------------------- |
| Fluffy   | 1995-05-15 | litter   | 4 kittens, 3 female, 1 male |
| Buffy    | 1993-06-23 | litter   | 5 puppies, 2 female, 3 male |
| Buffy    | 1994-06-19 | litter   | 3 puppies, 3 female         |
| Chirpy   | 1999-03-21 | vet      | needed beak straightened    |
| Slim     | 1997-08-03 | vet      | broken rib                  |
| Bowser   | 1991-10-12 | kennel   |                             |
| Fang     | 1991-10-12 | kennel   |                             |
| Fang     | 1998-08-28 | birthday | Gave him a new chew toy     |
| Claws    | 1998-03-17 | birthday | Gave him a new flea collar  |
| Whistler | 1998-12-09 | birthday | First birthday              |

像这样加载数据：

```sql
mysql> LOAD DATA LOCAL INFILE 'event.txt' INTO TABLE event;
```

根据在pet表上执行的查询所学到的知识，你应该能够在event表的记录上执行检索，原理是相同的。event表何时不足以回答你可能提出的问题？

假如你想知道每个宠物生产时他们多少岁。之前我们已经知道如何用两个日期值计算年龄。母亲的生产日期在event表中，但计算年龄还需要出生日期，其存储在pet表中。这意味着这个查询需要两个表：

```sql
mysql> SELECT pet.name,
       TIMESTAMPDIFF(YEAR,birth,date) AS age,
       remark
       FROM pet INNER JOIN event
         ON pet.name = event.name
       WHERE event.type = 'litter';
+--------+------+-----------------------------+
| name   | age  | remark                      |
+--------+------+-----------------------------+
| Fluffy |    2 | 4 kittens, 3 female, 1 male |
| Buffy  |    4 | 5 puppies, 2 female, 3 male |
| Buffy  |    5 | 3 puppies, 3 female         |
+--------+------+-----------------------------+
```

关于这个查询有几个注意事项：

- from子句连接了两个表，因为这个查询需要从两个表中提取数据。
- 当从多个表中组合（连接）信息时，你需要指定一个表中的记录如何匹配另一个表中的记录。这不难，因为两个表都有name列。这个查询基于name列的值来匹配两个表中的记录。 该查询使用INNER JOIN子句来组合表。当且仅当两个表都符合ON子句指定的条件时，INNER JOIN子句才会允许其中一个表中的行出现的结果中。这个例子中，on子句指定pet表中的name列必须与event表中的name列匹配。如果一个name在一个表中出现了，但另一个表中没出现，则该行不会出现在结果中，因为不符合on子句指定的条件。
- 因为name列在两个表中都出现了，因此当你检索该列时，你必须指定该列来自哪个表。这是通过在列名前面添加表名来实现的。

**不是必须用两个不同的表执行连接。如果你想比较同一个表中的不同记录，****用一个表连接自身有时也是很有用的。比如，如果你想在宠物中查找生育对，你可以将pet与其自身连接，生成具有相似物种的活着的雌性和雄性动物的候选对：**

```sql
mysql> SELECT p1.name, p1.sex, p2.name, p2.sex, p1.species
       FROM pet AS p1 INNER JOIN pet AS p2
         ON p1.species = p2.species
         AND p1.sex = 'f' AND p1.death IS NULL
         AND p2.sex = 'm' AND p2.death IS NULL;
+--------+------+-------+------+---------+
| name   | sex  | name  | sex  | species |
+--------+------+-------+------+---------+
| Fluffy | f    | Claws | m    | cat     |
| Buffy  | f    | Fang  | m    | dog     |
+--------+------+-------+------+---------+
```

此查询中，我们为表名指定别名来引用列，并保持与每个列引用关联的表实例。

## 3.4 获取数据库和表的信息

如果你忘记了数据库或表的名字，或者给定表的结构（比如，他的列名是什么？）时怎么办？MySQL通过支持几个提供有关数据库和表信息的语句来解决此问题。

前面见过SHOW DATABASES语句，可以列出服务器管理的数据库。使用DATABASE()函数，可以查看当前被选择的数据库：

```sql
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| menagerie  |
+------------+
```

如果你还没选择数据库，则结果为null。

使用下列语句，查看当前数据库包含的表（比如，当你不确定表的名字）：

```sql
mysql> SHOW TABLES;
+---------------------+
| Tables_in_menagerie |
+---------------------+
| event               |
| pet                 |
+---------------------+
```

这条语句产生的输出中，列的名字总是为“Tables_in_db_name”，db_name就是当前数据库的名字。更多信息见13.7.7.39节 SHOW DATABASES语句。

如果你想查看表的结构，DESCRIBE语句可以帮忙。它显示表中行的信息：

```sql
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
```

“Field”表示列名，“Type”表示数据类型，“NULL”表示该列是否可以为NULL值，“Key”表示该列是否有索引，“Default”指定该列的默认值，“Extra”显示该列的特殊信息：如果列创建时使用了AUTO_INCREMENT选项，那么Extra的值就是auto_increment，而不是空。

DESC是DESCRIBE的缩写形式，详细信息见13.8.1节 DESCRIBE语句。

你可以使用SHOW CREATE TABLE语句来获得创建一个存在的表时所需要的CREATE TABLE语句。见13.7.7.10节 SHOW CREATE TABLE语句。

如果一个表上有索引，SHOW INDEX FROM tbl_name语句会显示索引信息。关于该语句的更多信息见13.7.7.22节 SHOW INDEX语句。

## 3.5 批处理模式下使用MySQL

前面章节中，使用mysql是直接输入语句并浏览结果。你也可以在批处理模式下运行mysql。为此，将你想要运行的语句放到一个文件中，告诉mysql从这个文件中读取输入：

```shell
shell> mysql < batch-file
```

如果你在Windows系统上运行mysql，并且文件中有些特殊字符会产生问题，你可以这样做：

```shell
C:\> mysql -e "source batch-file"
```

如果你需要在命令行中指定连接参数，应使用如下命令：

```shell
shell> mysql -h host -u user -p < batch-file
Enter password: ********
```

当使用这种方法使用mysql时，你要创建一个脚本文件，然后执行此脚本。

如果脚本文件中的有些语句产生错误时你想继续执行此脚本，你应该使用--force命令行选项。

为什么使用脚本？这里有几个原因：

- 如果你重复执行某个查询（比如每天或每周），使用脚本可以让你避免每次执行时重复输入。
- 你可以在现有查询脚本文件中，通过复制和编辑脚本文件，生成类似的新查询脚本。
- 在开发查询时，批处理模式也很有用，特别是对于多行语句或多语句序列。如果你犯了个错误，你不需要重新输入所有语句，只需要编辑脚本改正其错误，然后告诉mysql重新执行它。
- 如果有一个会产生很多输出的查询，你可以通过pager运行输出，而不用看着他滚动到屏幕顶部之外：（就是利用shell的more命令，逐页产生输出。）

```shell
shell> mysql < batch-file | more
```

- 也可以将输出截取到文件中，以便后续处理：

```shell
shell> mysql < batch-file > mysql.out
```

- 你可以将你的脚本分发给其他人，以便他们也可以运行这些语句。
- 有些情况不允许交互式查询，比如，当你在一个cron（计划任务）任务中执行查询时，必须使用批处理模式。

批处理模式或交互模式运行mysql时，默认的输出格式是不同的，批处理模式更简洁。比如交互模式运行select distinct species from pet输出如下：

``` sql
+---------+
| species |
+---------+
| bird    |
| cat     |
| dog     |
| hamster |
| snake   |
+---------+
```

批处理模式输出如下：

``` 
species
bird
cat
dog
hamster
snake
```

如果你想在批处理模式下得到交互模式的输出格式，使用mysql -t。要将执行的语句回显到输出，使用mysql -v。

你也可以在mysql>提示符下使用source命令或\.命令来使用脚本：

```shell
mysql> source filename;
mysql> \. filename
```

更多信息见4.5.1.5节 从一个文本文件中执行SQL语句。

## 3.6 常用查询示例

*3.6.1 列的最大值*

*3.6.2 拥有某列最大值的行*

*3.6.3 每组中列的最大值*

*3.6.4 拥有某一分组某列最大值的行*

*3.6.5 使用用户自定义变量*

*3.6.6 使用外键*

*3.6.7 两个条件的搜索*

*3.6.8 计算每天的访问量*

*3.6.9 使用AUTO_IMCREMENT*



这里有一些使用MySQL解决常用问题的示例。

一些示例使用shop表保存每个交易商的每件商品的价格。假设每个交易商对于每件商品都有一个固定的价格，因此（article, dealer）是表的主键。

启动mysql命令行工具并选择一个数据库：

```shell
shell> mysql your-database-name
```

使用一下语句创建并填充示例表：

```sql
CREATE TABLE shop (
    article INT UNSIGNED  DEFAULT '0000' NOT NULL,
    dealer  CHAR(20)      DEFAULT ''     NOT NULL,
    price   DECIMAL(16,2) DEFAULT '0.00' NOT NULL,
    PRIMARY KEY(article, dealer));
INSERT INTO shop VALUES
    (1,'A',3.45),(1,'B',3.99),(2,'A',10.99),(3,'B',1.45),
    (3,'C',1.69),(3,'D',1.25),(4,'D',19.95);
```

执行完上述语句后，该表应该有以下内容：

```sql
SELECT * FROM shop ORDER BY article;

+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|       1 | A      |  3.45 |
|       1 | B      |  3.99 |
|       2 | A      | 10.99 |
|       3 | B      |  1.45 |
|       3 | C      |  1.69 |
|       3 | D      |  1.25 |
|       4 | D      | 19.95 |
+---------+--------+-------+
```

### 3.6.1 列的最大值

“最高的商品编号是多少？”

```sql
SELECT MAX(article) AS article FROM shop;
+---------+
| article |
+---------+
|       4 |
+---------+
```

### 3.6.2 拥有某列最大值的行  IMPORTANT

任务：查找最贵的商品的商品编号、交易商和价格。

使用子查询很容易可以完成上述任务：

```sql
SELECT article, dealer, price
FROM   shop
WHERE  price=(SELECT MAX(price) FROM shop);

+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0004 | D      | 19.95 |
+---------+--------+-------+
```

其他方法：**使用left join**，或将所有的行按照价格降序排序，然后使用limit子句获得第一行：

```sql
SELECT s1.article, s1.dealer, s1.price
FROM shop s1
LEFT JOIN shop s2 ON s1.price < s2.price
WHERE s2.article IS NULL;

SELECT article, dealer, price
FROM shop
ORDER BY price DESC
LIMIT 1;
```

> Note
>
> 如果有几个商品都是最贵的价格，价格都是19.95，使用limit这种方法只能获得他们中的一个。

### 3.6.3 每个分组中列的最大值

任务：查找每种商品的最贵价格。

```sql
SELECT article, MAX(price) AS price
FROM   shop
GROUP BY article
ORDER BY article;

+---------+-------+
| article | price |
+---------+-------+
|    0001 |  3.99 |
|    0002 | 10.99 |
|    0003 |  1.69 |
|    0004 | 19.95 |
+---------+-------+
```

### 3.6.4 拥有某一分组某列最大值的行 IMPORTANT

**任务：对于每种商品，找到拥有最贵价格的交易商或交易商们。**

此问题可以用如下的子查询解决：

```sql
SELECT article, dealer, price
FROM   shop s1
WHERE  price=(SELECT MAX(s2.price)
              FROM shop s2
              WHERE s1.article = s2.article)
ORDER BY article;

+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0001 | B      |  3.99 |
|    0002 | A      | 10.99 |
|    0003 | C      |  1.69 |
|    0004 | D      | 19.95 |
+---------+--------+-------+
```

上述例子中，使用了效率比较差的相关子查询（见13.2.11.7节 相关子查询）。解决此问题的其他可选择的方法有在from子句中使用非相关子查询、使用LEFT JOIN、使用窗口函数的共用表表达式。

非相关子查询：

```sql
SELECT s1.article, dealer, s1.price
FROM shop s1
JOIN (
  SELECT article, MAX(price) AS price
  FROM shop
  GROUP BY article) AS s2
  ON s1.article = s2.article AND s1.price = s2.price
ORDER BY article;
```

**LEFT JOIN（自连接）：**

```sql
SELECT s1.article, s1.dealer, s1.price
FROM shop s1
LEFT JOIN shop s2 ON s1.article = s2.article AND s1.price < s2.price
WHERE s2.article IS NULL
ORDER BY s1.article;
```

LEFT JOIN基于以下原理进行工作：当s1.price达到其最大值时，s2.price就没有一个更大的值了，相应的s2.price的值就是NULL。见13.2.10.2 JOIN子句。

**使用窗口函数的共用表表达式：**

```sql
WITH s1 AS (
   SELECT article, dealer, price,
          RANK() OVER (PARTITION BY article
                           ORDER BY price DESC
                      ) AS `Rank`
     FROM shop
)
SELECT article, dealer, price
  FROM s1
  WHERE `Rank` = 1
ORDER BY article;
```

### 3.6.5节 使用用户自定义变量

你可以使用用户变量来记录结果，而不用将其存储在客户端的临时变量中。（见9.4节 用户自定义变量）。

比如，要查找具有最贵和最便宜价格的商品，可以这样做：

```sql
mysql> SELECT @min_price:=MIN(price),@max_price:=MAX(price) FROM shop;
mysql> SELECT * FROM shop WHERE price=@min_price OR price=@max_price;
+---------+--------+-------+
| article | dealer | price |
+---------+--------+-------+
|    0003 | D      |  1.25 |
|    0004 | D      | 19.95 |
+---------+--------+-------+
```

> Note
>
> 在用户变量中存储一个数据库对象的名字比如表名或列名也是可能的，然后可以将此变量用在SQL语句中。然而，这需要使用一条预处理语句。更多信息见13.5节 预处理语句。

### 3.6.6 使用外键

在MySQL中，InnoDB表支持外键约束检查。见15章 InnoDB存储引擎和1.7.2.3节 外键约束差异。

仅连接两个表时，不需要外键约束。对于InnoDB存储引擎以外的存储引擎来说，当使用`REFERENCES tbl_name(col_name)`子句定义一个列时，该子句没有实际的效果，服务器只会把它当作当前定义的列的备忘录或注释，而不是引用其他表的列。当使用该句法时，意识到以下几点是极其重要的：

* MySQL不会执行任何检查来确认`col_name`是否存在于`tbl_name`表中（甚至`tbl_name`表本身是否存在）。

* 在你定义的表中执行的诸如删除行的操作时，MySQL不会去响应他们而在tbl_name表上执行相同的操作。换句话说，这个句法不会引起`on delete`或`on update`行为的任何责任。（尽管你可以在`REFERENCES`子句中写上`ON DELETE`或`ON UPDATE`子句，但将会被忽略。）

* 此句话创建一个列，但不会创建任何索引和键。

你可以将这样创建的列用作连接列，如下所示：
``` sql
CREATE TABLE person (
    id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
    name CHAR(60) NOT NULL,
    PRIMARY KEY (id)
);

CREATE TABLE shirt (
    id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
    style ENUM('t-shirt', 'polo', 'dress') NOT NULL,
    color ENUM('red', 'blue', 'orange', 'white', 'black') NOT NULL,
    owner SMALLINT UNSIGNED NOT NULL REFERENCES person(id),
    PRIMARY KEY (id)
);

INSERT INTO person VALUES (NULL, 'Antonio Paz');

SELECT @last := LAST_INSERT_ID();

INSERT INTO shirt VALUES
(NULL, 'polo', 'blue', @last),
(NULL, 'dress', 'white', @last),
(NULL, 't-shirt', 'blue', @last);

INSERT INTO person VALUES (NULL, 'Lilliana Angelovska');

SELECT @last := LAST_INSERT_ID();

INSERT INTO shirt VALUES
(NULL, 'dress', 'orange', @last),
(NULL, 'polo', 'red', @last),
(NULL, 'dress', 'blue', @last),
(NULL, 't-shirt', 'white', @last);

SELECT * FROM person;
+----+---------------------+
| id | name                |
+----+---------------------+
|  1 | Antonio Paz         |
|  2 | Lilliana Angelovska |
+----+---------------------+

SELECT * FROM shirt;
+----+---------+--------+-------+
| id | style   | color  | owner |
+----+---------+--------+-------+
|  1 | polo    | blue   |     1 |
|  2 | dress   | white  |     1 |
|  3 | t-shirt | blue   |     1 |
|  4 | dress   | orange |     2 |
|  5 | polo    | red    |     2 |
|  6 | dress   | blue   |     2 |
|  7 | t-shirt | white  |     2 |
+----+---------+--------+-------+

SELECT s.* FROM person p INNER JOIN shirt s
   ON s.owner = p.id
 WHERE p.name LIKE 'Lilliana%'
   AND s.color <> 'white';
+----+-------+--------+-------+
| id | style | color  | owner |
+----+-------+--------+-------+
|  4 | dress | orange |     2 |
|  5 | polo  | red    |     2 |
|  6 | dress | blue   |     2 |
+----+-------+--------+-------+
```
当以这种方式使用`REFERENCES`子句时，它不会在`SHOW CREATE TABLE`或`DESCRIBE`语句的输出中出现。
```sql
SHOW CREATE TABLE shirt\G
*************************** 1. row ***************************
Table: shirt
Create Table: CREATE TABLE `shirt` (
`id` smallint(5) unsigned NOT NULL auto_increment,
`style` enum('t-shirt','polo','dress') NOT NULL,
`color` enum('red','blue','orange','white','black') NOT NULL,
`owner` smallint(5) unsigned NOT NULL,
PRIMARY KEY  (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4
```
在MySAM表的列定义中以这种方式使用`REFERENCES`时，它将被作为注释或提醒。

### 3.6.7 使用两个键搜索
当`OR`使用一个键时，很容易被优化，`AND`也一样。 

当用`OR`组合两个不同的键来搜索时，就难办了。

```sql
SELECT field1_index, field2_index FROM test_table
WHERE field1_index = '1' OR  field2_index = '1'
```
这种情况已优化，见8.2.1.3节 索引合并优化。

使用`UNION`将两个分开的`SELECT`语句的输出结果组合起来，可以有效的解决这个问题。见13.2.10.3节 `UNION`子句。

每一个`SELECT`只能对一个键进行搜索与优化。
### 3.6.8 计算每日访问量
下面的例子说明如何使用`BIT_COUNT()`函数计算用户每月访问网站的天数。
```sql
CREATE TABLE t1 (year YEAR, month INT UNSIGNED,
             day INT UNSIGNED);
INSERT INTO t1 VALUES(2000,1,1),(2000,1,20),(2000,1,30),(2000,2,2),
            (2000,2,23),(2000,2,23);
```
示例表包含代表用户访问页面的年-月-日值。要确定这些访问在每个月中出现了多少天，使用下面的查询：
```sql
SELECT year,month,BIT_COUNT(BIT_OR(1<<day)) AS days FROM t1
       GROUP BY year,month;
```
输出：
```sql
+------+-------+------+
| year | month | days |
+------+-------+------+
| 2000 |     1 |    3 |
| 2000 |     2 |    2 |
+------+-------+------+
```
上述查询计算了对于每个年/月组合，表中有多少个不同的天，并自动删除重复的条目。
### 3.6.9 使用AUTO_INCREMENT
`AUTO_INCREMENT`属性可以用来为一条新记录生成一个唯一的标识符：
```sql
CREATE TABLE animals (
     id MEDIUMINT NOT NULL AUTO_INCREMENT,
     name CHAR(30) NOT NULL,
     PRIMARY KEY (id)
);

INSERT INTO animals (name) VALUES
    ('dog'),('cat'),('penguin'),
    ('lax'),('whale'),('ostrich');

SELECT * FROM animals;
```
输出结果：
```sql
+----+---------+
| id | name    |
+----+---------+
|  1 | dog     |
|  2 | cat     |
|  3 | penguin |
|  4 | lax     |
|  5 | whale   |
|  6 | ostrich |
+----+---------+
```
`AUTO_INCREMENT`列没有指定任何值，所以MySQL自动为其分配一个序列号。你也可以明确的为这一列分配为0，来生成一个序列号，除非`NO_AUTO_VALUE_ON_ZERO`SQL模式启用了。比如：

```sql
INSERT INTO animals (id,name) VALUES(0,'groundhog');
```
如果列被声明为`NOT NULL`也可以通过为该列分配NULL值来生成序列号，比如：
```sql
INSERT INTO animals (id,name) VALUES(NULL,'squirrel');
```

