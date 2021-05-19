# 第三章 教程

*3.1 服务器的连接与断开*

*3.2 输入查询*

*3.3 创建并使用数据库*

*3.4 获取数据库和表信息*

*3.5 批处理模式下使用mysql*

*3.6 常用查询示例*

*3.7 结合Apache使用MySQL*

本章通过展示如何使用mysql客户端来创建并使用一个简单的数据库，提供了一个教程简介。

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
当你向一个AUTO_INCREMENT列插入一个值时，则该列将被设置为该值，并且序号也将被重置，以便下一个自动生成的值依次位于最大列值之后。比如：
```sql
INSERT INTO animals (id,name) VALUES(100,'rabbit');
INSERT INTO animals (id,name) VALUES(NULL,'mouse');
SELECT * FROM animals;
+-----+-----------+
| id  | name      |
+-----+-----------+
|   1 | dog       |
|   2 | cat       |
|   3 | penguin   |
|   4 | lax       |
|   5 | whale     |
|   6 | ostrich   |
|   7 | groundhog |
|   8 | squirrel  |
| 100 | rabbit    |
| 101 | mouse     |
+-----+-----------+
```
更新一个存在的`AUTO_INCREMENT`列的值也会重置`AUTO_INCREMENT`序列。

你可以使用`LAST_INSERT_ID()`SQL函数或者`mysql_insert_id()`C API函数检索最新自动生成的`AUTO_INCREMENT`值。这两个函数是面向连接的，所以其他进行插入操作的连接不会影响他们的返回值。

为`AUTO_INCREMENT`列选择数据类型时，选择能容纳你所需要的最大序号的最小的整形数据类型。当该列到达数据类型的最大值时，下一次生成序号的尝试将会失败。如果可能，使用`UNSIGNED`属性，以便允许更大的取值范围。比如，如果使用`TINYINT`，则最大允许序号为127。对于`TINYINT UNSIGNED`，最大序号为255。所有整型类型的取值范围，见11.1.2节 整型数据类型（精确值）-INTEGER,INT,SMALLINT,TINYINT,MEDIUMINT,BIGINT。

> Note
> 
>对于多行插入，`LAST_INSERT_ID()`和`mysql_insert_id()`其实返回的是所插入行中的第一个`AUTO_INCREMENT`值。这使得多行插入在复制环境中的其他服务器上可以正确再现。（使得多行插入在复制环境中的从服务器上可以正确再现。）

想要`AUTO_INCREMENT`的值以非1开始，可以用`CREATE TABLE`或`ALTER TABLE`语句设置该值：
```sql
mysql> ALTER TABLE tbl AUTO_INCREMENT = 100;
```
**InnoDB Notes**

关于InnoDB存储引擎下`AUTO_INCREMENT`特定用法的信息，见15.6.1.6节 `AUTO_INCREMENT`在InnoDB中的处理方式。

**MyISAM Notes**

对于MyISAM表，你可以在多列索引的第二列上指定`AUTO_INCREMENT`。此时，`AUTO_INCREMENT`列生成的值为`MAX(auto_increment_column)+1 WHERE prefix=given-prefix`。当你向将数据放入有序组时，这很有用。

```sql
CREATE TABLE animals (
    grp ENUM('fish','mammal','bird') NOT NULL,
    id MEDIUMINT NOT NULL AUTO_INCREMENT,
    name CHAR(30) NOT NULL,
    PRIMARY KEY (grp,id)
) ENGINE=MyISAM;

INSERT INTO animals (grp,name) VALUES
    ('mammal','dog'),('mammal','cat'),
    ('bird','penguin'),('fish','lax'),('mammal','whale'),
    ('bird','ostrich');

SELECT * FROM animals ORDER BY grp,id;
```
输出结果：
```sql
+--------+----+---------+
| grp    | id | name    |
+--------+----+---------+
| fish   |  1 | lax     |
| mammal |  1 | dog     |
| mammal |  2 | cat     |
| mammal |  3 | whale   |
| bird   |  1 | penguin |
| bird   |  2 | ostrich |
+--------+----+---------+
```

在这种情况下（AUTO_INCREMENT 列是多列索引的一部分）， 如果删除了任意组中具有最大AUTO_INCREMENT值的行，则该AUTO_INCREMENT值将被重用。即使对于通常不会重用AUTO_INCREMENT值的MyISAM表，也会发生这种情况。

如果AUTO_INCREMENT列是多个索引的一部分，则MySQL使用从该AUTO_INCREMENT列开始的索引（如果有的话）生成序列值 。例如，如果animals表包含索引PRIMARY KEY (grp, id) 和INDEX (id)，则MySQL将忽略 PRIMARY KEY用于生成序列值的。结果，该表将包含一个序列，而不是每个grp值一个序列。

**延伸阅读**

有关`AUTO_INCREMENT`的更多信息，见：
* 如何将`AUTO_INCREMENT`属性分配给列，见13.1.20节 `CREATE TABLE`语句和13.1.9节 `ALTER TABLE`语句。
* `AUTO_INCREMENT`的行为取决于`ON_AUTO_VALUE_ON_ZERO`SQL模式，见5.1.11节 服务器的SQL模式。
* 如何使用`LAST_INSERT_ID()`函数找到包含最近`AUTO_INCREMENT`值的列，见12.16节 信息函数。
* 设置`AUTO_INCREMENT`的值为要使用的值，见5.1.8节 服务器系统变量。
* 15.6.1.6节 `AUTO_INCREMENT`在InnoDB中的处理方式。
* `AUTO_INCREMENT`和复制，见17.5.1.1节 复制和`AUTO_INCREMENT`。
* 可以用于复制的与`AUTO-INCREMENT`相关的系统变量（`auto_increment_increment`和`auto_increment_offset`），见5.1.8节 服务器系统变量。

## 3.7 将MySQL与Apache结合使用
有一些程序可以让你验证MySQL数据库中的用户，还可以让你将日志文件写入MySQL表。

你可以通过将下面的字符加入到Apache配置文件中，以改变Apache日志文件格式，使其对于MySQL来说更加易读：
```
LogFormat \
        "\"%h\",%{%Y%m%d%H%M%S}t,%>s,\"%b\",\"%{Content-Type}o\",  \
        \"%U\",\"%{Referer}i\",\"%{User-Agent}i\""
```
要将一个上述格式的日志文件加载到MySQL，你可以使用下面的语句：
```sql
LOAD DATA INFILE '/local/access_log' INTO TABLE tbl_name
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '\\'
```
要创建上述指定的表，使其具有与`LogFormat`行写入到日志文件中的列相对应的列。

# 第四章 MySQL程序
*4.1 MySQL程序概述*

*4.2 使用MySQL程序*

*4.3 服务器和服务器启动程序*

*4.4 与安装相关的程序*

*4.5 客户端程序*

*4.6 管理和实用工具程序*

*4.7 程序开发实用工具*

*4.8 其他程序*

*4.9 环境变量*

*4.10 MySQL中Unix信号的处理*

本章对Oracle公司提供的MySQL命令行程序进行了简短的概述。也对这些程序运行时指定选项的基本句法进行了讨论。大部分程序都有特定于其自身操作的选项，但选项句法是相似的。最后，本章提供了关于各个程序更详细的描述，包括他们可识别的选项。
## 4.1 MySQL程序概述

MySQL安装中有很多不同的程序。本节提供这些程序的简短概述。后面的章节会提供每个程序更详细的描述，除了NDB集群程序。每个程序的描述都说明了其调用句法和它所支持的选项。23.4节 NDB集群程序 描述了NDB集群特定的程序。

除了那些特定于平台的程序外，大部分MySQL发行版本都包括了所有的这些程序。（比如，服务器启动脚本不适用于Windows。）一个例外是RPM发行版，它更加专用。服务器有一个RPM程序，客户端程序也有一个，以此类推。如果你发现缺失了一个或多个程序，见第二章 安装和更新MySQL，获得关于发行版的类型，以及他们包含哪些程序的信息。可能的原因是该发行版并不包含所有的程序，需要安装其他软件包。

每个MySQL程序都有很多不同的选项。大部分程序提供一个`--help`选项，你可以通过它来获取程序不同选项的描述。比如，尝试`mysql --help`。

你可以通过在命令行或选项文件中为MySQL程序指定选项，来覆盖其默认选项。有关调用程序和指定程序选项的信息，见4.2节 使用MySQL程序。

MySQL服务器，mysqld程序，是主要的程序，它做了MySQL安装的大部分工作。服务器附有几个相关的脚本，可以帮助你启动或停止服务器：
* msqld  
SQL守护程序（即MySQL服务器）。想要使用客户端程序，mysqld必须运行，因为客户端通过连接到服务器来访问数据库。见4.3.1节 mysqld—MySQL服务器。
* mysqld_safe  
服务器启动脚本。mysqld_safe会尝试启动mysqld。见4.3.2节 mysqld_safe—MySQL服务器启动脚本。
* mysql.server  
服务器启动脚本。这个脚本用来在System V风格的系统上运行包含脚本的目录，为特定的运行级别开启系统服务。它通过调用mysqld_safe启动MySQL服务器。见4.3.3节 mysql.server—MySQL服务器启动脚本。
* mysqld_multi  
服务器启动脚本，可以启动或停止安装在系统上的多个服务器。见4.3.4节 mysqld_multi—管理多个MySQL服务器。

有几个程序在MySQL安装和更新期间，执行设置操作：
* comp_err  
该程序在MySQL构建/安装期间使用。它从错误源码文件中编译错误消息文件。见4.4.1节 comp_err—编译MySQL错误消息文件。
* mysql_secure_installation  
该程序可以让你的MySQL安装提高安全性。见4.4.2节 mysql_secure_installation—提高MySQL安装安全性。
* mysql_ssl_rsa_setup  
该程序创建支持安全连接所需要的SSL安全证书、密钥文件、RSA密钥对文件，如果这些文件缺失了。mysql_ssl_rsa_setup创建的文件可以用于使用SSL或RSA的安全连接。见4.4.3节 mysql_ssl_rsa_setup—创建SSL/RSA文件。
* mysql_tzinfo_to_sql  
该程序使用主机系统zoneinfo数据库（描述时区文件的集合）中的内容加载`mysql`数据库中的时区表。见4.4.4节 mysql_tzinfo_to_sql—加载时区表。
* mysql_upgrate  
在MySQL8.0.16之前，该程序用在MySQL升级操作之后。根据新版本的MySQL的变化，来升级权限表，检查表的不兼容性，并在必要时修复它。见4.4.5节 mysql_upgrade—检查和升级MySQL表。  
从MySQL8.0.16开始，MySQL服务器由mysql_upgrade执行的升级任务提前了。（详细信息见2.11.3节 MySQL升级过程都升级了什么。）

连接MySQL服务器的MySQL客户端程序：
* mysql  
用于交互式的输入SQL语句或以批处理模式从文件中执行SQL语句的命令行工具，见4.5.1节 mysql—MySQL命令行客户端。
* mysqladmin  
执行管理操作的客户端，比如创建或删除数据库，重载权限表，将表刷到磁盘，以及重新打开日志文件。`mysqladmin`还可以用来检索服务器的版本、进程和状态信息。见4.5.2节 mysqladmin—MySQL服务器管理程序。
* mysqlcheck  
表维护客户端，检查、修复、分析以及优化表。见4.5.3节 mysqlcheck—表维护程序。
* mysqldump  
将MySQL数据库以SQL、text或XML格式转储到文件中。见4.5.4节 mysqldump—数据库备份程序。
* mysqlimport  
一个用`LOAD DATA`语句将text文件导入到对应的表中的客户端程序。见4.5.5节 mysqlimport—数据导入程序。
* mysqlpump  
一个将MySQL数据库以SQL格式转储到文件中的客户端程序。见4.5.6节 mysqlpump—数据库备份程序。
* mysqlsh  
MySQL shell是MySQL服务器的高级客户端程序和代码编辑器。见[MySQL shell 8.0](https://dev.mysql.com/doc/mysql-shell/8.0/en/)。除了提供类似于mysql程序的SQL功能外，MySQL shell还提供JavaScript和python脚本功能，包括各种API。X DevAPI使你可以使用关系数据和文档数据，见第二十章 将MySQL用作文档存储。AdminAPI使你可以使用InnoDB集群，见[使用MySQL AdminAPI](https://dev.mysql.com/doc/mysql-shell/8.0/en/admin-api-userguide.html)。
* mysqlshow  
一个用来显示数据库、表、列以及索引信息的客户端程序。见4.5.7节 mysqlshow—显示数据库、表、以及列信息。
* mysqlslap  
一个为MySQL服务器设计的用来模拟客户端负载以及报告每个阶段的时间的客户端程序。就像多个客户端同时访问服务器一样。见4.5.8节 mysqlslap—负载模拟客户端程序。

MySQL管理和实用工具程序：
* innochecksum  
脱机的InnoDB脱机文件校验和实用工具。见4.6.2节 innochecksum—脱机InnoDB文件校验和实用工具。
* myisam_ftdump  
用来显示MyISAM表全文索引信息的实用工具。见4.6.3节 myisam_ftdump—显示全文索引信息。
* myisamchk  
描述、检查、优化和修复MyISAM表的实用工具。见4.6.4节 myisamchk—MyISAM表维护工具。
* myisamlog  
用来审阅MyISAM日志文件内容的实用工具。见4.6.5节 myisamlog—显示MyISAM日志文件内容。
* myisampack  
用来压缩MyISAM表以生成更小的只读表的实用工具。见4.6.6节 myisampack—生成压缩的、只读的MyISAM表。
* mysql_config_editor  
一个实用工具，使你可以将身份验证凭据存储在名为.mylogin.cnf的安全、加密的登录路径文件中。见4.6.7节 mysql_config_editor—MySQL配置实用工具。
* mysql_migrate_keyring  
一个实用工具，用于在一个密钥环组件和另一个密钥环组件之间迁移密钥。见4.6.8节 mysql_migrate_keyring—密钥环密钥迁移实用工具。
* mysqlbinlog  
一个从二进制日志中读取SQL语句的实用工具。二进制日志文件中包含的执行过的SQL语句的记录，可以用于崩溃恢复。见4.6.9节 mysqlbinlog—处理二进制日志文件的实用工具。
* mysqldumpshow  
用于读取和汇总慢查询日志内容的实用工具。见4.6.10节 mysqldumpshow—汇总慢查询日志。

MySQL程序开发实用工具：
* mysql_config  
一个shell脚本，该脚本可以生成编译MySQL程序时需要的选项值。见4.7.1节 mysql_config—显示编译客户端程序的选项。
* my_print_defaults  
一个实用工具，用于显示选项文件的选项组中存在哪些选项。见4.7.2节 my_print_defaults—从选项文件中显示选项。

其他实用工具：
* lz4_decompress  
一个实用工具，用来解压缩mysqlpump程序通过LZ4压缩产生的输出。见4.8.1节 lz4_decompress—解压缩mysqlpump LZ4-Compressed输出。
* perror  
一个实用工具，用来显示系统或MySQL错误代码的含义。见4.8.2节 perror—显示MySQL错误消息信息。
* zlib_decompress  
一个实用工具，用来解压缩mysqlpump程序通过ZLIB压缩产生的输出。见4.8.3节 zlib_decompress—解压缩mysqlpump ZLIB-Compressed输出。

Oracle公司还提供了MySQL Wordbench图形界面工具，用来管理MySQL服务器或数据库，可以创建、执行、评估查询，也可以将其他关系数据库管理系统的模式和数据迁移到MySQL服务器上。

MySQL客户端程序与服务器通过使用MySQL客户端/服务器库来进行通信，会用到以下环境变量。

| Environment Variable | Meaning                                  |
| -------------------- | ---------------------------------------- |
| MYSQL_UNIX_PORT      | 默认Unix套接字文件，用于连接到本地主机。 |
| MYSQL_TCP_PORT       | 默认端口号，用于TCP/IP连接。             |
| MYSQL_DEBUG          | 调试时的调试跟踪选项。                   |
| TMPDIR               | 临时表和临时文件创建的目录。             |

MySQL程序所使用的环境变量的完整列表，见4.9节 环境变量。

## 4.2 使用MySQL程序

*4.2.1 调用MySQL程序*

*4.2.2 指定程序选项*

*4.2.3 用于连接服务器的命令选项*

*4.2.4 使用命令选项连接到MySQL服务器*

*4.2.5 使用URI-Like字符串或键值对连接到服务器*

*4.2.6 使用DNS SRV记录连接到服务器*

*4.2.7 连接传输协议*

*4.2.8 连接压缩控制*

*4.2.9 设置环境变量*

### 4.2.1 调用MySQL程序

为从命令行（就是shell或者命令提示符）调用MySQL程序，请输入程序名，然后输入任意选项或需要的其他参数，来指示该程序你想让它做什么。下面的命令展示了几个程序调用的例子。
```shell
shell> mysql --user=root test
shell> mysqladmin extended-status variables
shell> mysqlshow --help
shell> mysqldump -u root personnel
```
以破折号（-或--）开头的参数指定程序的选项。选项一般指示程序应与服务器建立连接的类型或影响服务器的运行模式。4.2.2节 指定程序选项 描述了选项句法。

无现选项参数（没有前导破折号的参数）提供了额外的程序信息。比如，mysql程序会把第一个没有选项的参数理解为一个数据库名，所以命令`--user=root test`表示你想使用test数据库。

后面的章节将会描述每个程序支持哪些选项，并说明无选项参数的意义。

有些选项对于很多程序来说是相同的。最常用的是`--host(or -h),--user(or -u),--password(or -p)`这些指定连接参数的选项。他们表示MySQL服务器运行的主机，MySQL账户的用户名和密码。所有的MySQL客户端程序都能识别这些选项；这些选项使你可以指定连接哪个服务器以及该服务器上使用的用户账号。其他连接选项有`--port(or -p)`指定一个TCP/IP端口号，以及`--socket(or -s)`在Unix上指定一个Unix套接字文件（在Windows上为命名管道）。更多关于指定连接选项的信息，见4.2.4节 使用命令选项连接到MySQL服务器。

你可能发现当你调用MySQL程序时，需要使用他们安装时的路径名：`bin`目录。当你每次从`bin`目录以外的目录下运行MySQL程序时，出现了“program not found”错误，则可能是这种情况。为了更方便的使用MySQL程序，你可以将`bin`目录的路径名添加到`PATH`环境变量中。这样你就可以在运行程序时只输入他的名字就可以了，不需要输入整个路径名。比如，如果`mysql`程序安装到了`/usr/local/mysql/bin`目录下，你可以直接输入`mysql`来调用，不需要像这样调用它：`/usr/local/mysql/bin/mysql`。

有关PATH环境变量的设置说明，请查阅你命令解释器的文档。设置环境变量的句法是特定于解释器的。（有些信息在4.2.9节 设置环境变量 中给出。）修改PATH设置后，在Windows上打开一个新的控制台窗口或者在Unix上再次登录，以使修改生效。

### 4.2.2 指定程序选项

*4.2.2.1 在命令行中使用选项*

*4.2.2.2 使用选项文件*

*4.2.2.3 影响选项文件处理的命令行选项*

*4.2.2.4 程序选项修饰符*

*4.2.2.5 使用选项设置程序变量*

*4.2.2.6 选项默认值，选项期望值和等号“=”*

有几个方法可以为MySQL程序指定选项：
* 在命令行中程序名的后面列出选项。这对于程序具体调用的选项来说很常见。
* 讲选项列在一个选项文件中，当程序运行时读取这个选项文件。对于希望程序运行每次都使用的选项，这是很常见的。
* 讲选项列到环境变量里面（见4.2.9节 设置环境变量）。对于每次程序运行都使用的选项，这很有用。实际上，选项文件通常用于此目的，但在5.8.3节 在Unix上运行多个MySQL实例 中讨论了一种情况，环境变量在这种情况下就很有用。它描述了一种很好用的技术，该技术使用这类变量为服务器和客户端程序指定TCP/IP端口号和Unix套接字文件。

选项按顺序处理，所以如果一个选项被指定了多次，只有最后出现的会生效。下面的命令使用`mysql`连接到运行在`localhost`的服务器：
```sql
mysql -h example.com -h localhost
```
有一个例外：对于`mysqld`，`--user`选项的第一个实例被用作安全预防措施，用来防止在选项文件中指定的用户被命令行中指定的用户覆盖。

如果给出了冲突或相关联的选项，则后面的选项会生效，而不是前面的选项。下面的命令使`mysql`在`no column names`模式下运行：
```sql
mysql --column-names --skip-column-names
```
MySQL程序通过检查环境变量，然后处理选项文件，然后检查命令行，来确定给出哪些选项。因为后给的选项优先于先给的选项，该处理顺序意味着环境变量拥有最低的优先级，命令行选项拥有最高的优先级。

你可以通过在选项文件中为选项指定默认选项值，来利用MySQL处理选项的方式。这可让你避免每次运行程序是都要键入他们，在必要时还可以让你通过使用命令行选项来覆盖选项默认值。

#### 4.2.2.1 在命令行中使用选项

在命令行中指定程序选项需要遵循以下规则：
* 选项在命令名之后给出。
* 一个选项参数以一个破折号或两个破折号为开头，取决于选项名是短格式还是长格式。很多选项同时拥有短格式和长格式。比如，`-?`和`--help`是指示MySQL程序显示帮助信息选项的短格式和长格式。
* 选项名是区分大小写的。`-v`和`-V`都是合法的，但有不同的意思。（他们是`--verbose`和`--version`选项的短格式。）
* 有些选项在其选项名之后会有一个值。比如`-h localhost`或`--host=localhost`对客户端程序指示MySQL服务器主机名。该选项值告诉程序MySQL服务器所运行的主机名。
* 对于有选项值的长格式选项，选项名和选项值之间用等号隔开。对于有选项值的短格式选项，选项值可以紧跟在选项名之后，也可以用一个空格隔开：`-hlocalhost`和`-h localhost`是等价的。有一个例外是用于指定MySQL密码的选项。该选项的长格式可以是`--password=pass_val`或`--password`。对于后者（没有给出密码值pass_val），程序以交互方式提示你输入密码。密码选项也可以用短格式给出，`-ppass_val`或`-p`。然而，对于短格式的密码选项，如果要给出密码值，则该值必须紧跟在选项名之后，不能有空格，如果选项名后面有空格，程序无法知道后面的参数是密码值还是其他类型的参数。因此，下面的两个命令具有完全不同的含义：
```sql
mysql -ptest
mysql -p test
```
第一个命令指示`mysql`使用text作为密码值，但没指定默认数据库。第二个参数指示`mysql`提示输入密码值，并指定text作为默认数据库。

* 在选项名里面，破折号和下划线是可以互换使用的。比如，`--skip-grant-tables`和`--skip_grant_table是等价的。（然而，开头的破折号不能用下划线。）
* MySQL具有某些命令选项只能在启动时指定，也有一组系统变量，有些可以在启动时设置，可以在运行时设置，或同时设置。系统变量名必须使用下划线，不能使用破折号，而且在运行时引用时（比如，使用`set`或`select`语句）必须使用下划线。
```sql
SET GLOBAL general_log = ON;
SELECT @@GLOBAL.general_log;
```
在服务器启动时，系统变量的句法与命令选项时一样的，所以在系统变量名里面，破折号和下划线可以互换使用。比如，`--general_log=ON`和`--general-log=ON`是等价的。（对于选项文件中的系统变量也是一样的。）
* 对于拥有数字值选项值的选项，该选项值可以用`K,M,G`后缀给出，指示乘数1024，1024^2，1024^3。从MySQL8.0.14开始，后缀还可以是`T,P,E`，指示乘数1024^4，1024^5或1024^6。后缀字符可以是大写或小写。比如，下面的命令告诉`mysqladmin`ping服务器1024次，每次间隔10秒：
```shell
mysqladmin --count=1K --sleep=10 ping
```
* 当将文件名指定为选项值时，避免使用~shell元字符。可能不会按照你的预期解释它。

在命令行中给出带有空格的选项值时，必须用引号括起来。比如，`mysql`的`--execute`或`-e`选项可以将一条或多条用分号隔开的SQL语句传递给服务器。当使用这个选项时，`mysql`执行在选项值中的语句并退出。这些语句必须用引号括起来。比如：
```sql
shell> mysql -u root -p -e "SELECT VERSION();SELECT NOW()"
Enter password: ******
+------------+
| VERSION()  |
+------------+
| 8.0.19     |
+------------+
+---------------------+
| NOW()               |
+---------------------+
| 2019-09-03 10:36:48 |
+---------------------+
shell>
```
>Note
>
>长格式`--execute`后面要有等号。

要在语句中使用带引号的值，必须在语句中转义内部的引号，或者在语句内使用与引号本身不同的引号。命令处理器的功能决定了你可以选择使用单引号还是双引号以及转义引号字符的语法。例如，如果命令处理器支持使用单引号或双引号引起来的引用，则可以在语句前后使用双引号，并对语句内的所有带引号的值使用单引号。

#### 4.2.2.2 使用选项文件（to be continued）

大部分MySQL程序都可以从选项文件中（有时被称为配置文件）读取启动选项。选项文件提供了一种指定常用选项的简单方式，每次运行一个程序是不需要在命令行中输入选项。

为查明一个程序是否会读取选项文件，用`--help`选项调用该程序。（对于`mysqld`，使用`--verbose`和`--help`。）如果该程序读取选项文件，则帮助信息会告诉你其查找的文件以及识别的选项组。
>Note
>
>拥有`--no-defaults`选项的MySQL程序不会读取除`.mylogin.cnf`以外的任何选项文件。
>
>禁用了`persisted_global_load`系统变量的服务器，不会读取`mysqld-auto.cnf`。

很多选项文件是纯文本文件，可以用任意文本编辑器创建。例外是：
* `.myligin.cnf`文件包含登录路径选项。该文件是一个由`mysql_config_editor`实用程序创建的加密文件。见4.6.7节 mysql_config_editor—mysql配置实用程序。登录路径就是只允许包含`host,user,password,port,socket`选项的选项组。客户端程序使用`--login-path`选项指定从`.mylogin.cnf`文件中读取登录路径。
设置`MYSQL_TEST_LOGIN_FILE`系统变量，以指定一个备用登录路径文件名。该变量会被`mysql-test-run.pl`测试实用程序使用，也可以被`mysql_config_editor`实用程序识别，以及MySQL客户端程序比如`mysql,mysqladmin`等。

#### 4.2.2.3 影响选项文件处理的命令行选项 （to be continued）

#### 4.2.2.4 程序选项修饰符 （to be continued）

#### 4.2.2.5 用选项设置程序变量（to be continued）

#### 4.2.2.6 选项默认值，选项额外值和等号（to be continued）

### 4.2.3 用于连接服务器的命令选项（to be continued）

### 4.2.4 使用命令选项连接到MySQL服务器（to be continued）

### 4.2.5 使用URI-Like字符串或键值对连接到服务器（to be continued）

### 4.2.6 使用DNS SRV记录连接到服务器（to be continued）

### 4.2.7 连接传输协议（to be continued）

### 4.2.8 连接压缩控制（to be continued）

### 4.2.9 设置环境变量（to be continued）

## 4.3 服务器和服务器启动程序

*4.3.1 mysqld——mysql服务器*

*4.3.2 mysql_safe——MySQL服务器启动脚本*

*4.3.3 mysql.server——mysql服务器启动脚本*

*4.3.4 mysqld_multi——管理多个MySQL服务器*

本节描述mysqld，即MySQL服务器，以及用于启动服务器的几个程序。

### 4.3.1 mysqld——MySQL服务器

mysqld，也被称为MySQL服务器，是一个单线程程序，可以完成一个MySQL安装中大部分工作。他不会产生其他进程。MySQL服务器负责管理包含数据库和表的MySQL数据目录的访问。数据目录也是诸如日志文件和状态文件等其他信息的默认位置。

>Note
>
>有些安装包包含名为mysqld-debug的服务器的debug版本。调用次版本而不是mysqld版本，可获得debug支持，内存分配检查以及追踪文件支持（见5.9.1.2节 创建追踪文件）。

当MySQL服务器启动后，他会监听来自客户端程序的网络连接，并代表这些客户端管理对数据库的访问。

mysqld程序在启动时有很多选项可以被指定。以下命令可提供这些选项的完整列表：
```shell
mysqld --verbose --help
```
MySQL服务器也有一组系统变量，这些变量会影响服务器运行时的行为。系统变量何以在服务器启动时进行设置，他们中的很多也可以在运行时更改，以实现动态服务器重新配置。MySQL服务器还有一组状态变量，可以提供其运行时的信息。你可以检测这些状态变量，以获得运行时性能特征。

MySQL服务器命令选项，系统变量以及状态变量的完整描述，见5.1节 MySQL服务器。关于安装MySQL服务器和设置初始配置的信息，见第二章 安装和更新MySQL。

### 4.3.2 mysqld_safe——MySQL服务器启动脚本（to be continued）

### 4.3.3 mysql.server——MySQL服务器启动脚本（to be continued）

### 4.3.4 mysqld_multi——管理多个MySQL服务器（to be continued）

## 4.4 与安装有关的程序（to be continued）

## 4.5 客户端程序

*4.5.1 mysql——MySQL命令行客户端*

*4.5.2 mysqladmin——MySQL服务器管理程序*

*4.5.3 mysqlcheck——表维护程序*

*4.5.4 mysqldump——数据库备份程序*

*4.5.5 mysqlimport——数据导入程序*

*4.5.6 mysqlpump——数据库备份程序*

*4.5.7 mysqlshow——显示数据库、表和列信息*

*4.5.8 mysqlslap——负载模拟程序*

本节描述连接到MySQL服务器上的客户端程序。

### 4.5.1 mysql——MySQL命令行客户端

*4.5.1.1 mysql客户端选项*

*4.5.1.2 mysql客户端命令*

*4.5.1.3 mysql客户端记录*

*4.5.1.4 mysql客户端服务器端帮助*

*4.5.1.5 从Text文件执行SQL语句*

*4.5.1.6 mysql客户端Tips*

mysql是一个具有输入行编辑功能的简单SQL shell。它支持交互式和非交互式用法。交互式下，查询结果以ASCII-table格式显示，非交互式下（比如，用作过滤器），查询结果以制表符分割的格式显示。输出格式可以通过命令选项进行改变。

如果由于内存不足而无法存储很大的结果集而出现问题，使用`--quick`选项。这强制使mysql每次从服务器检索一行数据，而不是检索所有数据并在显示之前存入内存缓冲区。这是通过使用客户端/服务器中的C API函数`mysql-use-result()`实现的，而不是`mysql_store_result()`。

>Note
>
>另外，MySQL Shell提供对X DevAPI的访问。详细信息见MySQL shell8.0。

使用mysql很简单，在命令解释器提示符中调用即可，如下：
```shell
mysql db_name
```
或者：
```shell
mysql --user=user_name --password db_name
```
此时，你需要在提示符中输入密码来回应mysql的显示：
```shell
Enter password: your_password
```
然后出入一条SQL语句，以`;,\g,\G`结尾，然后按回车。

按下`Control+C`以中断当前语句（如果存在），或取消任意为完成的输入行。

你可以从一个脚本文件（批处理文件）中执行SQL语句，如下：
```shell
mysql db_name < script.sql > output.tab
```
Unix系统上，mysql会将交互执行的SQL语句记录到一个历史文件中。见4.5.1.3节 mysql客户端日志。

#### 4.5.1.1 mysql客户端选项（to be continued）

#### 4.5.1.2 mysql客户端命令（to be continued）

#### 4.5.1.3 mysql客户端记录（to be continued）

#### 4.5.1.4 mysql客户端服务器端帮助（to be continued）

#### 4.5.1.5 从Text文件中执行SQL语句（to be continued）

有时，你想让脚本展示过程信息，可用以下语句：？？？
```sql
SELECT '<info_to_display>' AS ' ';
```

#### 4.5.1.6 mysql客户端Tips（to be continued）

### 4.5.2 mysqladmin——MySQL服务器管理程序（to be continued）

### 4.5.3 mysqlcheck——表维护程序（to be continued）

### 4.5.4 mysqldump——数据库备份程序（to be continued）

### 4.5.5 mysqlinport——数据导入程序（to be continued）

### 4.5.6 mysqlpump——数据库备份程序（to be continued）

### 4.5.7 mysqlshow——显示数据库、表和列信息（to be continued）

### 4.5.8 mysqlslap——负载模拟客户端（to be continued）

mysqlslap是一个诊断程序，旨在模拟MySQL服务器的客户端负载，并报告每个阶段的时间。它模拟多个客户端访问服务器。

像这样调用mysqlslap：
```sql
mysqlslap [options]
```
有些选项比如`--create,--query`，是你能够指定包含一条语句的字符串或包含多条语句的文件。如果你制定了一个文件，默认每行必须包含一条语句。（也就是说，隐式语句定界符是换行符。）使用`--delimiter`选项制定一个不同的定界符，这使你可以指定跨越多行的语句，或将多条语句放在一行内。你不能在文件中包含评论，mysqlslap不能识别他们。

mysqlslap运行分为三个阶段：
1. 创建模式，表以及任何可选的存储程序或数据以供测试使用。这个阶段使用一个客户端连接。
2. 运行负载特使。这个阶段可以使用多个客户端连接。
3. 清除阶段（断开连接，删除表指定的表）。这个阶段使用一个客户端连接。

比如：
提供你自己的创建和查询SQL语句，用50个客户端进行查询，每个客户端200次select操作（将命令在一行中输入）：

```shell
mysqlslap --delimiter=";"
  --create="CREATE TABLE a (b int);INSERT INTO a VALUES (23)"
  --query="SELECT * FROM a" --concurrency=50 --iterations=200
```

让mysqlslap用包含两个INT列和三个VARCHAR列的表构建查询SQL语句。使用5个客户端每个20次查询。不要创建表或者插入数据（也就是说，使用之前测试例子中的模式和表）：
```shell
mysqlslap --concurrency=5 --iterations=20
  --number-int-cols=2 --number-char-cols=3
  --auto-generate-sql
```
告诉程序从指定的文件中加载创建、插入和查询SQL语句，create.sql文件有多条以分号为定界符的表创建语句，和多条以分号为定界符的插入语句。`--query`选项指定的文件应该包含多条以分号为定界符的查询语句。运行加载语句，然后运行查询语句文件中的所有查询使用5个客户端（每个查询5次）：
```shell
mysqlslap --concurrency=5
  --iterations=5 --query=query.sql --create=create.sql
  --delimiter=";"
```
mysqlslap支持以下选项，可以在命令行中指定，也可以在选项文件中的[mysqlslap]或[client]选项组中指定。

| Option Name                             | Description                                                  | Introduced | Deprecated |
| :-------------------------------------- | ------------------------------------------------------------ | ---------- | ---------- |
| --auto-generate-sql                     | Generate SQL statements automatically when they are not supplied in files or using command options |            |            |
| --auto-generate-sql-add-autoincrement   | Add AUTO_INCREMENT column to automatically generated tables  |            |            |
| --auto-generate-sql-execute-number      | Specify how many queries to generate automatically           |            |            |
| --auto-generate-sql-guid-primary        | Add a GUID-based primary key to automatically generated tables |            |            |
| --auto-generate-sql-load-type           | Specify the test load type                                   |            |            |
| --auto-generate-sql-secondary-indexes   | Specify how many secondary indexes to add to automatically generated tables |            |            |
| --auto-generate-sql-unique-query-number | How many different queries to generate for automatic tests   |            |            |
| --auto-generate-sql-unique-write-number | How many different queries to generate for --auto-generate-sql-write-number |            |            |
| --auto-generate-sql-write-number        | How many row inserts to perform on each thread               |            |            |
| --commit                                | How many statements to execute before committing             |            |            |
| --compress                              | Compress all information sent between client and server      |            | 8.0.18     |
| --compression-algorithms                | Permitted compression algorithms for connections to server   | 8.0.18     |            |
| --concurrency                           | Number of clients to simulate when issuing the SELECT statement |            |            |
| --create                                | File or string containing the statement to use for creating the table |            |            |
| --create-schema                         | Schema in which to run the tests                             |            |            |
| --csv                                   | Generate output in comma-separated values format             |            |            |
| --debug                                 | Write debugging log                                          |            |            |
| --debug-check                           | Print debugging information when program exits               |            |            |
| --debug-info                            | Print debugging information, memory, and CPU statistics when program exits |            |            |
| --default-auth                          | Authentication plugin to use                                 |            |            |
| --defaults-extra-file                   | Read named option file in addition to usual option files     |            |            |
| --defaults-file                         | Read only named option file                                  |            |            |
| --defaults-group-suffix                 | Option group suffix value                                    |            |            |
| --delimiter                             | Delimiter to use in SQL statements                           |            |            |
| --detach                                | Detach (close and reopen) each connection after each N statements |            |            |
| --enable-cleartext-plugin               | Enable cleartext authentication plugin                       |            |            |
| --engine                                | Storage engine to use for creating the table                 |            |            |
| --get-server-public-key                 | Request RSA public key from server                           |            |            |
| --help                                  | Display help message and exit                                |            |            |
| --host                                  | Host on which MySQL server is located                        |            |            |
| --iterations                            | Number of times to run the tests                             |            |            |
| --login-path                            | Read login path options from .mylogin.cnf                    |            |            |
| --no-defaults                           | Read no option files                                         |            |            |
| --no-drop                               | Do not drop any schema created during the test run           |            |            |
| --number-char-cols                      | Number of VARCHAR columns to use if --auto-generate-sql is specified |            |            |
| --number-int-cols                       | Number of INT columns to use if --auto-generate-sql is specified |            |            |
| --number-of-queries                     | Limit each client to approximately this number of queries    |            |            |
| --only-print                            | Do not connect to databases. mysqlslap only prints what it would have done |            |            |
| --password                              | Password to use when connecting to server                    |            |            |
| --pipe                                  | Connect to server using named pipe (Windows only)            |            |            |
| --plugin-dir                            | Directory where plugins are installed                        |            |            |
| --port                                  | TCP/IP port number for connection                            |            |            |
| --post-query                            | File or string containing the statement to execute after the tests have completed |            |            |
| --post-system                           | String to execute using system() after the tests have completed |            |            |
| --pre-query                             | File or string containing the statement to execute before running the tests |            |            |
| --pre-system                            | String to execute using system() before running the tests    |            |            |
| --print-defaults                        | Print default options                                        |            |            |
| --protocol                              | Transport protocol to use                                    |            |            |
| --query                                 | File or string containing the SELECT statement to use for retrieving data |            |            |
| --server-public-key-path                | Path name to file containing RSA public key                  |            |            |
| --shared-memory-base-name               | Shared-memory name for shared-memory connections (Windows only) |            |            |
| --silent                                | Silent mode                                                  |            |            |
| --socket                                | Unix socket file or Windows named pipe to use                |            |            |
| --sql-mode                              | Set SQL mode for client session                              |            |            |
| --ssl-ca                                | File that contains list of trusted SSL Certificate Authorities |            |            |
| --ssl-capath                            | Directory that contains trusted SSL Certificate Authority certificate files |            |            |
| --ssl-cert                              | File that contains X.509 certificate                         |            |            |
| --ssl-cipher                            | Permissible ciphers for connection encryption                |            |            |
| --ssl-crl                               | File that contains certificate revocation lists              |            |            |
| --ssl-crlpath                           | Directory that contains certificate revocation-list files    |            |            |
| --ssl-fips-mode                         | Whether to enable FIPS mode on client side                   |            |            |
| --ssl-key                               | File that contains X.509 key                                 |            |            |
| --ssl-mode                              | Desired security state of connection to server               |            |            |
| --tls-ciphersuites                      | Permissible TLSv1.3 ciphersuites for encrypted connections   | 8.0.16     |            |
| --tls-version                           | Permissible TLS protocols for encrypted connections          |            |            |
| --user                                  | MySQL user name to use when connecting to server             |            |            |
| --verbose                               | Verbose mode                                                 |            |            |
| --version                               | Display version information and exit                         |            |            |
| --zstd-compression-level                | Compression level for connections to server that use zstd compression | 8.0.18     |            |

## 4.6 管理和实用程序（to be continued）

## 4.7 程序开发实用程序（to be continued）

## 4.8 其他程序（to be continued）

## 4.9 环境变量（to be continued）

## 4.10 MySQL中Unix信号的处理（to be continued）

# 第五章 MySQL服务器管理

*5.1 MySQL服务器*

*5.2 MySQL数据目录*

*5.3 系统模式—mysql*

*5.4 MySQL服务器日志*

*5.5 MySQL组件*

*5.6 MySQL服务器插件*

*5.7 MySQL服务器可加载函数*

*5.8 在一台机器上运行多个MySQL实例*

*5.9 调试MySQL*

MySQL服务器（mysqld）是主要的程序，它完成了一个MySQL安装中的大部分工作。本章概述了MySQL服务器，并涵盖了常规的服务器管理：
* 服务器配置
* 数据目录，特别是mysql系统模式
* 服务器日志文件
* 管理一台机器上的多个服务器

有关管理注意的其他信息，见：
* 第六章 安全性
* 第七章 备份和恢复
* 第十七章 复制

## 5.1 MySQL服务器

*5.1.1 配置服务器*

*5.1.2 服务器默认配置*

*5.1.3 服务器配置验证*

*5.1.4 服务器选项，系统变量和状态变量参考*

*5.1.5 服务器系统变量参考*

*5.1.6 服务器状态变量参考*

*5.1.7 服务器命令选项*

*5.1.8 服务器系统变量*

*5.1.9 使用系统变量*

*5.1.10 服务器状态变量*

*5.1.11 服务器SQL模式*

*5.1.12 连接管理*

*5.1.13 IPv6支持*

*5.1.14 网络命名空间支持*

*5.1.15 MySQL服务器时区支持*

*5.1.16 资源组*

*5.1.17 服务器端帮助支持*

*5.1.18 服务器追踪客户端回话状态变更*

*5.1.19 服务器关闭过程*

mysqld就是MySQL服务器。接下来的讨论涵盖了这些MySQL服务器配置主题：
* 服务器所支持的启动选项。你可以在命令行中指定这些选项，也可以在配置文件中指定。
* 服务器系统变量。这些变量反映了启动选项的当前状态和值，有些系统变量可以在服务器运行时更改。
* 服务器状态变量。这些变量包含运行时操作的计数器和统计信息。
* 如何设置服务器SQL模式。这个设置改变了SQL句法或语义的某些方面，比如与其他数据库系统代码的兼容性，或者控制特定情况下的错误处理。
* 服务器如何管理客户端连接。
* 配置并使用IPv6和网络命名空间支持。
* 配置并使用时区支持。
* 使用资源组。
* 服务器端帮助功能。
* 提供了用于启用客户端回话状态更改的功能。
* 服务器关闭过程。根据表的类型（事务性还是非事务性）以及是否使用复制，需要考虑性能和可靠性。

MySQL8.0中增加的、弃用的或删除的MySQL服务器变量和选项的列表，见1.4节 MySQL8.0中增加的、弃用的或删除的MySQL服务器变量和选项。

>Note
>
>并非所有的MySQL服务器二进制文件和配置都支持所有存储引擎。为确定你的MySQL服务器安装支持哪些存储引擎，见13.7.7.16节 SHOW ENGINES语句。

### 5.1.1 配置服务器

MySQL服务器，mysqld，有很多命令选项和系统变量，可以在启动时进行设置，以配置其行为。为查看服务器使用的默认命令选项和系统变量，执行以下语句：
```shell
shell> mysqld --verbose --help
```
该命令生成所有mysqld选项和可配置系统变量的列表。其输出包括默认选项和变量值，看起来像下面这样：
```shell
abort-slave-event-count           0
allow-suspicious-udfs             FALSE
archive                           ON
auto-increment-increment          1
auto-increment-offset             1
autocommit                        TRUE
automatic-sp-privileges           TRUE
avoid-temporal-upgrade            FALSE
back-log                          80
basedir                           /home/jon/bin/mysql-8.0/
...
tmpdir                            /tmp
transaction-alloc-block-size      8192
transaction-isolation             REPEATABLE-READ
transaction-prealloc-size         4096
transaction-read-only             FALSE
transaction-write-set-extraction  XXHASH64
updatable-views-with-limit        YES
validate-user-plugins             TRUE
verbose                           TRUE
wait-timeout                      28800
```

要查看服务器运行时实际使用的当前系统变量值，请连接到MySQL服务器并执行以下语句：
```sql
mysql> SHOW VARIABLES;
```
要查看运行中服务器的一些统计信息和状态提示符，请运行以下语句：
```sql
mysql> SHOW STATUS;
```
系统变量和状态信息也可以用`mysqladmin`命令获得：
```sql
shell> mysqladmin variables
shell> mysqladmin extended-status
```
所有命令选项、系统变量和状态变量的完整描述，请查看以下章节：
* 5.1.7节 服务器命令选项
* 5.1.8节 服务器系统变量
* 5.1.10节 服务器状态变量

从Performance_schema可以获得更详细的监视器信息，见第二十七章 MySQL Performance_schema。另外，MySQL`sys`模式是一组对象，可以方便的访问Performance_schema收集的数据，见第二十八章 MySQL sys模式。

如果你在命令行中为mysqld或mysqld_safe指定了一个选项，仅在本次服务器调用时生效。对于每次服务器启动都使用的选项，将其放到一个选项文件里。见4.2.2.2节 使用选项文件。

### 5.1.2 服务器默认配置

MySQL服务器有很多运行参数，你可以在服务器启动时使用命令行选项或配置文件（选项文件）修改他们。服务器运行时也可以修改很多参数。有关在启动或运行时设置参数的一般说明，见5.1.7节 服务器命令参数 和 5.1.8节 服务器系统变量。

Windows上，MySQL安装器与用户交互，并在安装根目录下创建一个名为my.ini的文件作为默认选项文件。

>Note
>
>在Windows上，.ini或.cnf文件扩展名可能不会显示。

在安装过程完成后，你可以随时编辑该默认选项文件，以修改服务器使用的参数。比如，要使用文件中一个行开头处用#号注释掉的参数设置，删掉#号，并在需要时修改参数值。要禁用一个设置，在行开头加上#号，或删除它。

对于非Windows平台，在服务器安装时或数据目录初始化过程中不会创建默认选项文件。你可以按照4.2.2.2节 使用选项文件 中的指引，创建你自己的选项文件。没有选项文件时，服务器就用其默认设置来启动，见5.1.2节 服务器默认配置 如何查看这些设置。

有关选项文件格式和句法的更多信息，见4.2.2.2节 使用选项文件。

### 5.1.3 服务器配置验证

从MySQL8.0.16开始，MySQL服务器开始支持`--validate-config`选项，可以在正常运行模式下，无需运行服务器，即可检查启动配置是否有问题：
```shell
mysqld --validate-config
```
如果没发现错误，则服务器以退出代码0终止。如果发现了错误，服务器会显示一条诊断消息，并以退出代码1终止。比如：
```shell
shell> mysqld --validate-config --no-such-option
2018-11-05T17:50:12.738919Z 0 [ERROR] [MY-000068] [Server] unknown
option '--no-such-option'.
2018-11-05T17:50:12.738962Z 0 [ERROR] [MY-010119] [Server] Aborting
```
当发现任意错误时，服务器就会终止。要进行其他检查，改正最初的问题后，再次用`--validate-config`选项运行服务器。

在前面使用`--validate-config`的例子中，导致了一条错误消息的显示，而且服务器退出代码为1。根据`log_error_verbosity`的值，警告和信息消息也可以被显示，但不会立刻终止验证或以退出代码1。比如，下面的命令产生了多条警告，两条都被显示了，但没有错误发生，所以退出代码为0：
```shell
shell> mysqld --validate-config --log_error_verbosity=2
         --read-only=s --transaction_read_only=s
2018-11-05T15:43:18.445863Z 0 [Warning] [MY-000076] [Server] option
'read_only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:18.445882Z 0 [Warning] [MY-000076] [Server] option
'transaction-read-only': boolean value 's' was not recognized. Set to OFF.
```
下面的命令产生了同样的警告，而且还有一条错误，所以错误消息与警告一起被显示了，退出代码为1：
```shell
shell> mysqld --validate-config --log_error_verbosity=2
         --no-such-option --read-only=s --transaction_read_only=s
2018-11-05T15:43:53.152886Z 0 [Warning] [MY-000076] [Server] option
'read_only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:53.152913Z 0 [Warning] [MY-000076] [Server] option
'transaction-read-only': boolean value 's' was not recognized. Set to OFF.
2018-11-05T15:43:53.164889Z 0 [ERROR] [MY-000068] [Server] unknown
option '--no-such-option'.
2018-11-05T15:43:53.165053Z 0 [ERROR] [MY-010119] [Server] Aborting
```
`--avlidate-config`选项的范围仅限于配置检查，服务器不需要经历其正常的启动过程就可以执行配置检查。这样，配置检查不会初始化存储引擎和其他插件、组件等，也不会验证与未初始化的子系统相关联的选项。

`--validate-config`选项随时可以使用，但在服务器升级之后特别有用，它可以检查升级后的服务器是否会认为老版本服务器之前使用过的选项已被弃用或淘汰。比如，`tx_read_only`系统变量在MySQL5.7中被弃用，在MySQL8.0被移除。假如一个MySQL5.7服务器使用了my.cnf文件中该系统变量运行，然后升级到了8.0。使用`--validate-config`选项运行升级后的服务器来检查配置，会生成以下结果：
```shell
shell> mysqld --validate-config
2018-11-05T10:40:02.712141Z 0 [ERROR] [MY-000067] [Server] unknown variable
'tx_read_only=ON'.
2018-11-05T10:40:02.712178Z 0 [ERROR] [MY-010119] [Server] Aborting
```
`--validate-config`选项可以跟`--defaults-file`选项结合使用，仅验证一个指定文件中的选项：
```shell
shell> mysqld --defaults-file=./my.cnf-test --validate-config
2018-11-05T10:40:02.712141Z 0 [ERROR] [MY-000067] [Server] unknown variable
'tx_read_only=ON'.
2018-11-05T10:40:02.712178Z 0 [ERROR] [MY-010119] [Server] Aborting
```
记住，如果`--defaults-file`选项被指定了，则在命令行中它必须是第一个选项。（以相反的选项顺序运行前面的例子，将会产生一条消息，`--defaults-file`本身是未知的。
