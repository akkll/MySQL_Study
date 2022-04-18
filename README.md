> # Mysql是怎样运行的——笔记

看了一下这本书的前言，作者告诫我们千万要按照章节顺序读，因此我决定每章记录下一些笔记，留待后续的复习。

# 一、初识Mysql

## 1.1CS架构

MySQL与微信的结构类似，都使用了C(客户端)S(服务端) 结构。日常使用MySQL的顺序如下：

1. 启动 MySQL 服务器程序. 
2. 启动 MySQL 客户端程序，并连接到服务器程序.
3. 在客户端程序中输入命令语句并将其作为请求发送给服务器程序。服务器程序在收到这些请求后，根据请求的内容来操作具体的数据，并将结果返回给客户端。

运行 过程中的 MySQL 服务器程序和客户端程序在本质上来说都算是计算机中的**进程**，其中代表 MySQL 服务器程序的进程称为 MySQL 数据库**实例**( instance ). 

## 1.3启动mysql服务器程序

### 1.3.1主要bin文件（可执行文件）

1. **mysqld**——表示 MySQL 服务器程序运行这个可行文就可以直接启动一个MySQL 服务器进程。
2. **mysqld_ safe**——它是一个启动脚本它会间接调用 mysqld 并持续监控服务器的运行状态。当服务器进程出现错误，它还可以帮助重启服务器程序。另外，使用 mysqld_safe启动 MySQL 服务器程序时，它会将服务器程序出错信息诊断信息输到错误日志，以方便后期查找发生错误的原因。
3. **mysql.server**——mysql.erver也是一个启动脚本，它会间接调用mysqld_ safe，在执行mysqld_ safe 时， 在后面添加start参数就可以启动服务器程序了，如**mysql.server start**。在后面添加stop，如**mysql.server stop**，关闭了服务器程序。（需要注意 mysql. server 文件其实是一个**链接文件**，它对应 实际文件是 ../support-files/mysql.server。）
4. **mysqld_multi**——启动或停止多个服务器进程，并报告它们的运行状态。

### 1.3.2启动MySQL服务器程序的方式

两种方式——手动方式和服务方式

1. 手动启动。在MySQL安装目录下bin目录中启动mysqld。

2. 服务方式。~~摆了哈哈😋，估计用不到，用到再补。~~反转了😅，服务器的关闭需要用到这个。(注意，这里只有使用**管理员权限**运行cmd才能使用下述指令)

   * 首先要把mysql注册为Windows的服务，具体如下

     > "完整的可执行文件路径 --install [-manual]  [服务名]
     >
     > 其中-manual的作用就是 Windows 系统启动的时候不自动启动该服务，否则会自动启动。服务名也可以省略，默认为MySQL，但是我的可能是版本不一样，**默认的服务名是MySQL80**.

   * 启动服务器可以使用

     > net start MySQL80

   * 关闭服务器可以使用

     > net stop MySQL80

## 1.4启动MySQL客户端程序

在cmd中进入bin文件夹，并执行下述语句即可启动服务端程序

> mysql -h主机名 -u用户名 -p密码
>
> 例如：mysql -hlocalhost -uroot -psql*\*\*\*\*\*
>
> 或者使用：mysql -hlocalhost -uroot -p

关闭服务端程序：在mysql>提示符后面输入下述命令即可

> quit
>
> exit
>
> \q

也可以同时启动多个MySQL的客户端，每个客户端程序是互不影响的。

## 1.5客户端与服务器连接的过程

### 1.5.1 TCP/IP

MySQL 采用TCP/IP作为服务器和客户端之间的网络通信协议。

MySQL 服务器会默认监听 3306 端口.可以在服务器启动的命令行键入**mysqld -P3307**将端口号改为3307。在连接的指令中还可以加入-P来指定端口，例如：

> mysql -hlocalhost -uroot -psql*\*\*\*\*\* -P3307

## 1.6服务器处理客户端请求

![885458831519bfd563b5cf6660a4855](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\885458831519bfd563b5cf6660a4855.png)

### 1.6.1第一部分：连接管理

客户端通过各种方式与服务器端建立连接。

* 在客户端连接到服务器端时，服务器内部会创建一个线程专门处理与这个客户端的交互。
* 在客户端与服务器端连接断开时，服务器不会马上销毁这个线程，而是会将其缓存起来，等待与另一个请求连接的客户端连接。如此不必频繁地创建与销毁线程，节省开销。

然而如果线程分配得太多会严重影响性能，因此需要对可以同时连接到服务器的客户端数量进行限制。



客户端发起连接时，需要携带主机信息、用户名与密码等信息留待服务器程序认证，认证失败则拒绝连接。

另外，如果客 户端程序和服务器程序不运行在一台计算机上，我们还可以通过采用**传输层安全性** (Transport Layer Security, TLS) 协议对连接进行加密，从而保证数据传输的安全性。

当连接建立后，与该客户端关联的服务器线程会 一直等待客户端发送过来的请求。MySQL 服务器接收到的请求只是一个**文本消息**，该文本消息还要经过各种处理。

### 1.6.2第二部分：解析与优化

**查询缓存**：MySql服务器会把把刚刚处理过的查询请求和结果缓存起来，留待下一次相同的请求返回。并且查询缓存可以在不同的客户端之间共享。

但是如果两个查询请求有任何字符上的不同(例如，空 格、注释、大小写)。都会导致缓存不会命中。

如果查询请求中包含某些系统函数、用户 自定义变量和函数 系统衰，如mysql，information——schema数据库中的表， 则这个请求就不会被缓存。

MySQL 的缓存系统会监测涉及的每张表，只要该表的结构或者数据被修改，则与该表有关的所有查 询缓存都将变为无效并从查询缓存中删除！

**语法解析**：因为客户端程序发送过来的请求只是一段文本，所以 MySQL 服务器程序首先要对这段文本进行分析，判断请求的语法是否正确，然后从文本中将要查询的表、各种查询条件都提取出来放到MySQL服务器内部使用的一些数据结构。

**查询优化**：服务器端已经获取了执行的一些必要信息，如要查询的表和列等，单仅此并不够，直接执行的话性能不高，因此MySQL服务器端还要进行优化，结果是产生一个执行计划，这个执行计划表明了应该使用哪些索引执行查询，以及表之间的连接顺序是啥样。

### 1.6.3第三部分：存储引擎

截止上述步骤，服务器还没有真正访问真实的表中数据。

MySQL 服务器把 数据的存储和提取操作都封装到了一个名为存储引擎的模块中。

**存储引擎的作用**：表是由一行一行的记录组成的，但这只是一个逻辑上的概念.在物理上如何表示记录，怎么从表中读取数据， 以及怎么把数据写入具体的物理存储器上，都是存储引擎负责的事情。

**MySQL服务器处理请求的功能划分**：连接管理、查询缓存、语法解析、查询优化这些并不涉及真实数据存取的功能划分 serve 层的功能，存取真实数据的功能划分为存储引擎层的功能。各种不同的存储引擎为 serve 层提供统一的调用接口，其中包含了几十个不同用途的底层函数，比如"读取索引第一 条记录 11 11 读取索引下一条记录" "插入记录"等。

在 server 层完成了查询优化后，只需按照生成的执行计划调用底层存储引擎提供的接口获取到数据后返回给客户端就好了。

**注意**：server 层和存储引擎层交互时，一般是以记录为单位的。select时server层是一条记录一条记录判断where条件决定取舍的。

## 1.8关于存储引擎的一些操作

### 1.8.1查看当前服务器程序支持的存储引擎

>  show engines

### 1.8.2设置表的存储引擎

1.创建表时可以设置表的存储引擎，例如

>CREATE TABLE 表名(
>
>建表语句;
>
>)ENGINE=存储引擎名称;

2.修改表存储引擎

> ALTER table 表名 ENGlNE =存储引擎名称;

**3.查询表结构**

> **show create table [数据库名].[表名]\G;**

# 二、MySQL的调控按钮一启动选项和系统变量

## 2.1 启动选顶和配置文件

### 2.1.1 在命令行上使用选项

若要禁止各个客户端使用TCP/IP网络进行通信，可以在启动服务器程序的命令行中执行下述指令：

> mysqld --skip-networking

修改默认的存储引擎：

> mysqld --default-storage-engine=引擎名称

## 2.2系统变量

### 2.2.2查看系统变量

输入以下指令：

> SHOW VARIABLES [LIKE 匹配的模式]

例如要查询默认的存储引擎：

> SHOW VARIABLES like 'default_storage_engine';

重要的系统变量还有**max_connections**(允许同时连接的客户端数)

### 2.2.3设置系统变量

# 三、字符集和比较规则

## 3.1字符集和比较规则简介

### 3.1.1字符集简介

计算机中实际存储的是二进制数据，要把哪些字符映射成二进制数据？也就是界定字符范围。将字符映射成二进制数据的过程叫作**编码**，将二进制数据映射到字符的过程叫作**解码**。

### 3.1.2比较规则简介

即如何比较两个字符的大小，而一种字符集可能对应的比较规则不同。

### 3.1.3一些重要的字符集

**开摆**

## 3.2MySQL 中支持的字符集和比较规则

### 3.2.2 字符集的查看

> show charset;

一些重要的字符集

![c0a87864a4a237658f9cfce49a5a62e](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\c0a87864a4a237658f9cfce49a5a62e.png)

### 3.2.3 比较规则的查看

> SHOW COLLATION [LIKE 匹配的模式];
>
> 例如：
>
> SHOW COLLATION like 'utf8\_%'

## 3.3 字符集和比较规则的应用

### 3.3.1 四种级别的字符集和比较规则

MySQL四个级别的字符集和比较规则，分别是服务器级别、数据库级别、表级别、列级别。

**服务器级别**：两个系统变量表示服务器级别的字符集和比较规则。

> show variables like '**character_set_server**';//查看字符集
>
> show variables like '**collation_server**';//查看比较规则

**数据库级别**：在**创建**和**修改**数据库时可以指定该数据库的字符集和比较规则。

> CREATE DATABASE 数据库名 
>
> ​	[[(DEFAULT] CHARACTER SET 字符集名称]
>
> ​	[[OEFAULT1 COLLATE 比较规则名称]; 
>
> ALTER DATABASE 数据库名 
>
> ​	[[(DEFAULT] CHARACTER SET 字符集名称]
>
> ​	[[OEFAULT1 COLLATE 比较规则名称]; 

两个系统变量表示服务器级别的字符集和比较规则。

> show variables like '**character_set_database**';//查看字符集
>
> show variables like '**collation_database**';//查看比较规则

这里的系统变量**无法**更改。**前提是先use [数据库名]**

**表级别**：与数据库级别类似，在创建和修改表的时候可以指定表的字符集和比较规则。

**列级别**：在创建和修改列的时候可以指定字符集和比较规则。

> create table 表名(
>
> ​	列名 数据类型 [character set 字符集名称] [collate 比较规则名称]
>
> );
>
> ALTER TABLE 表名 MODIFY 列名 数据类型 [CHARACTER SET 字符集名称] [COLLATE 比较规则名称]

若仅仅修改字符串或者比较规则，另一方也会随之改变。

还有一种级别的编码方式：**客户端的编码方式**，相关的系统变量为**character_set_client**。服务器与客户端建立连接后，服务器会为客户端维护一个单独的**character_set_client**变量。

### 3.3.2 客户端和服务器通信过程中使用的字符集

1.编码和解码的字符集不一样会导致乱码。

2.字符集转换：将字节序列先按照一种字符集解码，再按照另一种字符集编码，这就完成了**字符集的转换**。

3.MySQL中的字符集转换过程：

MySQL客户端与服务端遵从一定的数据格式，即**MySQL通信协议**，这个格式指明了请求和响应的每 个字节分别代表什么意思。

* **客户端发送请求**：Windows操作系统中，字符集被称为**代码页**，使用指令**chcp**可以查询代码页的唯一标识，其中936代表GBK编码，65001代表UTF-8编码。在启动客户端时若携带了**default-character-set=字符集名称**可以使用指定的字符集来编码。

* **服务器接受请求**：服务器端接受的是一串字节序列。服务器默认将该序列视为以**character_set_client**进行编码的字符串。但实际上，客户端发送的字节序列的采用的编码字符集与服务器认为的字节序列的采用的编码字符集(即**character_set_client**)是**两个不同的字符集**，且**character_set_client**这个变量可以被更改。

* **服务器处理请求**：服务器会将请求的字节序列当作采用character_set_ client对应的字符集进行编码的字节序列，不过在真正处理请求时又会将其转换为使用 SESSION 级别的系统变量 **character_set_connection** 对应的字符集进行编码的字节序列。

  > 为什么要使用了character_set_ client后还要使用character_set_connection呢？
  >
  > 因为在查询表的时候，表和列拥有固定的字符集，因此查询时按照它们固有的字符集查询即可；但是如果查询与表无关的话，例如:select 'a'='A'时，如何判别这两个字符的字符集？这时就需要有默认的字符集将其转化为字节序列，这个默认的字符集就是**character_set_connection**，对应的比较规则就是**collation_connection**。**因此列的字符集和排序规则的优先级要高于character_set_connection，也就是说字符集转换过程为：character_set_client->character_set_connection->列字符集(如果查询涉及到列)**。

* **服务器生成响应**：服务器会将读出的结果以系统变量**character_set_results**来进行编码传回客户端。每个MySQL客户端会维护一个默认的字符集，如果在登录的时候没有指定的字符集，那么就默认为操作系统的字符集。可以修改character_set_client，character_set_connection，character_set_results三个变量。

  > set character_set_client = charset_name;
  >
  > set character_set_connection = charset_name;
  >
  > set character_set_results = charset_name;
  >
  > 也可以将上述三条语句简化为
  >
  > set names charset_name;

* **客户端接收响应**：会使用客户端的默认字符串来解释接收到的字符。

### 3.3.3比较规则的应用

比较规则也叫排序规则。

## 3.4总结

字符从查询到返回到客户端的字符集转换如下

>  **客户端默认字符集——>character_set_client——>character_set_connection——>内部的字符集(列字符集)——>character_set_results——>客户端默认字符集**

# 四、InnoDB记录存储结构

## 4.2InnoDB页简介

InnoDB 个将表中的数据存储到磁盘上的存储引擎 即使我们关闭并重启服务器，数 据还是存在。由于读写磁盘的速度比读取内存的速度慢很多，因此innoDB采用了一种相较于根据记录一条一条读写更为高效方式：将数据划分为若干页，以页作为磁盘与内存交互的基本单位。InnoDB页的大小一般是16KB。一般情况下，一次最少从磁盘中读取 16KB 的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中。

系统变量innoDB_page_size表明了InnoDB 存储引 擎中的页大小，默认16384，即16KB。在服务器运行期间不可以更改这个变量。

## 4.3InnoDB行格式

表中的记录在磁盘上的存放形式被称为行格式或者记录格式。目前InnoDB已有4种不同类型的行格式，分别是COMPACT,REDUNDANT,DYNAMIC,COMPRESSED。

### 4.3.1指定行格式的语法

在建表或修改表时可以指定行格式

> CREATE TABLE 表名 (
>
> ​	列信息
>
> )ROW_FORMAT=行格式名称;
>
> ALTER TABLE ROW_FORMAT=行格式名称;

### 4.3.2COMPACT行格式

![8b1751272cbc5ecf8ed09d82ce980bf](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\8b1751272cbc5ecf8ed09d82ce980bf.png)

#### 1.记录的额外信息

##### (**1).变长字段长度列表**

MySQL支持一些变长的数据类型如：varchar，text等，这些都是长度不固定的数据类型。在存储真实数据的时候需要把这些数据的长度保存起来，这样才不至于让MySQL服务器懵逼。在COMPACT行格式中，各个变长字段的真实数据占用字节数会按照顺序**逆序存放**他们长度的十六进制数！！！**逆序存放**！！！

同时变长字段长度列表只存储非NULL的列的内容长度，不存储值为 NULL 列的内容长度。

例如：(c2是char类型，不需要填入它的长度，而c1，c3，c4都是varchar类型)![7f283aed83695b438d14abbf5e98121](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\7f283aed83695b438d14abbf5e98121.png)

依据行格式填充为：

![f2b530e12fb13fb2ddad053d378fe9f](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\f2b530e12fb13fb2ddad053d378fe9f.png)

当变长字段的长度过大，InnoDB会考虑将使用2个字节来存储长度的字节数。以下是规则：

> 令字符集所决定的每个字符的长度为W(例如utf-8编码方式决定每个字符最大长度为3，则W为3)，令变长变量限定的数据长度为M(例如一个字段的数据类型为varchar(M))，变长字段实际存储的字符串占用的字节数是L。
>
> * 如果M*W<=255，使用1字节来表示真实数据占用长度；
> * 如果M*W>255：
>   * 如果L<=127，则使用1字节表示真实数据占用长度
>   * 如果L>127，则使用2字节表示真实数据占用长度
>
> 当M\*W<=255时，数据库知道每个字节代表了一个变长字段的长度；
>
> 但是当M\*W>255时，一个字节可能不够代表一个字段的长度，因此对于长度较小的字段来说一个字节就可以表示长度，长度较大的字段则需要两个字节来表示长度。那么如何区分表示长度的字节究竟表示的是一个字符长度还是半个字符的长度？InnoDB在L上做了一些处理：将L的最高位二进制码作为标志码，当这个码为0时，说明这是一个单独的字节；当这个码为1时，说明这是半个字节长度。

##### (**2).NULL值列表**

COMPACT行格式把一条记录中值为NULL的列都统一管理放到NULL值列表里面。处理过程如下：

1. 首先统计表中**允许存储**NULL的列有哪些。（不是已经存储NULL的列；主键列和使用NOT NULL都不能存储NULL值，因此统计时不会把这些列算进去）；

2. 若表中没有允许存储NULL的列，则NULL值列表也就不存在；否则将每个存储NULL值的列对应一个二进制，将它们**逆序**排列。这个二进制只有1位，1表示该列值为NULL，0表示该列值不为NULL。

3. MySQL规定NUll值列表必须要用整个字节来表示，所以如果使用的二进制位个数不是整数个字节，则在字节的高位补0。例如

   > c1,c2,c3是3个列，c1，c2列的值都是NULL，那么它们的排列为011（要逆序），由于二进制个数不是整数个字节，则在011前面补0，得到00000011。以此类推，如果出现超过8个列允许为NULL，则使用2个或以上的字节数来表示。

##### (3).记录头信息

这部分由固定的5个字节（即40个二进制位）组成，用于描述记录的一些属性，不同的位代表不同的信息。

![137eae7436daafab659fb0156223c4a](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\137eae7436daafab659fb0156223c4a.png)

![020df8bcbfb80c8e9fab072772f24ac](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\020df8bcbfb80c8e9fab072772f24ac.png)

#### 2.记录的真实数据

记录的真实数据除了自己定义的列以外还有一些隐藏列：

![aa5f5d561fbf0ecf2af5b0b0f7a50f1](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\aa5f5d561fbf0ecf2af5b0b0f7a50f1.png)

关于ROW_ID：InnoDB表的生成主键的策略决定了ROW_ID的存在与否。主键选取的规则如下：

* 优先使用用户自己定义的主键
* 若没有定义主键，则使用一个不允许存储NULL的UNIQUE键作为主键
* 若上述都没有，则生成隐藏列ROW_ID作为主键

#### 3.char(M)列的存储格式

char(M)的存储形式与字符集有关。尽管char(M)不属于变长数据类型，但是当char()使用了变长编码字符集

> ascii编码不是变长编码字符集，因为ascii编码所能允许字符的最大字节数为1，而类似utf8，GBK等编码表示一个字符则需要可能不止一个字节，因此被称为变长编码字符集

则也被视为可变字段，这一列所占用的字节数也会被逆序存储到变长字段长度列表里。

> 例如，c1，c2，c4是varchar类型，而c3是char(m)类型，当c3的字符集是ascii时，变长字段长度列表里从左到右分别存储了c4，c2，c1的长度；当c3的字符集使用了utf8或者GBK时，变长字段长度列表里就添加了c3的长度，并且这个长度被插入在c4和c2之间。

使用变长编码字符集的char(M)类型的列要求**至少**占用M个字节，但对于varchar(M)没有这个要求。例如对于使用了UTF8编码字符集的列来说，这一列存储的信息介于10-30字节之间，即使向列中存储一个空的字符串也是10字节。这主要是希望在日后更新数据时，新数据长度大于旧数据长度且小于10字节时可以直接在原地址重新更新而不用分配一个新的记录空间。

### 4.3.3REDUNDANT行格式

REDUNDANT行格式全貌如下

![3db223fca177fa7c2a51bb7e3f3146a](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\3db223fca177fa7c2a51bb7e3f3146a.png)

#### 1.字段长度偏移列表

字段长度偏移列表相较于COMPACT的变长字段长度列表，有以下两点不同

1. 没有“变长”二字，说明REDUNDANT行格式会把记录里所有列（包括隐藏列）长度信息都逆序放到字段长度偏移列表里；
2. 多了“偏移”二字，说明REDUNDANT行格式计算列长度时不像COMPACT那么直观，而是采用两个相邻偏移量的差值计算各个列的长度

> 例如一条记录的字段长度偏移列表是这样的：*25 24 1A 17 13 0C 06*，因为时逆序排列，则从列1到列7分别是：*06 0C 13 17 1A 24 25* ，按照偏移量来计算就是：第一列是06-00，6个字节；第二列是0C-06，6个字节；第三列是13-0C，7个字节，以此类推。

#### 2.记录头信息

![ddd97f0b43093b8961716743600d608](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\ddd97f0b43093b8961716743600d608.png)

与COMPACT的记录头信息相比，REDUNDANT行格式多了n_field和1byte_offs_flag;

少了record_ type。

#### 3.1byte_offs_flag的选择

每个列对应的偏移量可以使用1-2个字节来表示，具体使用1个字节还是2个字节依据REDUNDANT行格式记录的真实占用大小。

* 真实记录的长度<=127时，每个列对应的偏移量占用1字节；
* 真实记录的长度>127且<32767(十六进制，0x7FFFF 二进制0111111111111111)时，每个列对应的偏移量占用2字节；
* 当真实数据大于32767，记录的一部分会被存放到溢出页，在本页中只保留 768 字节和20 字节的溢出页面地址

**注意这里是127！不是255！！！最高位另有作用。**

当1byte_offs_flag的值为1时，表明使用1字节存储偏移量；

当1byte_offs_flag的值为0时，表明使用2字节存储偏移量；

#### 4.REDUNDANT行格式处理NULL值

将列对应的偏移值的第一个比特位作为是否是NULL的依据，这个比特位也可以叫NULL比特位。

对于值为 NULL 的列来说，该列的类型是否为变长类型决定了该列在记录的真实数据处的存储方式。

如果存储NULL值的列是CHAR(M)类型的，那么就在真实数据部分用0填充M个字节

> 例如一列的的类型是char(10)，则在REDUNDANT行格式中使用ox00000000000000000000来表示 NULL。此外这一列对应的偏移量是0xA4，对应的二进制10100100，由于最高位是1，因此是NULL值，从0100100可以看出它的偏移量是36，与前一个偏移量相减就得到长度。

如果存储NULL值的列是变长数据类型的，不在记录的真实数据部分占用任何存储空间，偏移量与前一个一致，表示占用的真实数据的字节数为0

#### 5.CHAR(M)的存储格式

不管该列使用的字符集是什么，char(M)占用的真实数据始终是该字符集的允许最大字节数乘上M

> 例如使用UTF8的char(10)所占用的空间就是30字节，使用GBK的char(10)所占用的空间就是20字节

### 4.3.4溢出列

#### 1.溢出列

在COMPACT和REDUNDANT行列式中，对于占用存储空间非常多的列，在记录的真实数据处只会存储该列一部分数据，而其他的数据则会被分散地存储在其他页里，然后在记录的真实数据处用 20 字节存储指向这些页的地址。 具体的，在记录的真实数据处存储的字节数是768字节，另外还有20字节作为地址信息指向存放剩余数据的溢出页。

例如

![32774cc75b627b000bc6ee155fe40e2](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\32774cc75b627b000bc6ee155fe40e2.png)

是用到溢出页的列被称为溢出列，不止varchar(M)类型，TEXT，BLOB都有可能成为溢出列。

#### 2.产生溢出页的临界点

MySQL规定一个页中至少有两条记录。每个页中除了存放记录外，还要存储一些额外的信息，加起来一个138字节，其他的空间全部用来存储记录。

每个记录所需要的额外空间是27字节，分别被这样分配：

![e0cacbca4467186947c6bea9379e798](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\e0cacbca4467186947c6bea9379e798.png)

要想页面不发生溢出现象，需要满足以下不等式

> 132 + 记录数 * (27 + 每条记录的真实占用字节数) < 16384

### 4.3.5DYNAMIC行格式和COMRESSED行格式

这二者与COMPACT行格式类似，不同点在于处理溢出列的操作。

它们不会在记录的真实数据处存储该溢出列真实数据的前 768 字节，而是把该列的所有真实数据都存储到溢出页中，只在记录的真实数据处存储 20 节大小的指向溢出页的地址。另外，COMPRESSE 行格式会采用压缩算法对页面进行压缩.

![99c206b3c7f5ab9d6787aa567322cc8](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\99c206b3c7f5ab9d6787aa567322cc8.png)

# 五、InnoDB的数据页结构

## 5.2数据页结构

![352f922d1bc0d6f1d5fc35c61524f6c](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\352f922d1bc0d6f1d5fc35c61524f6c.png)

![f3f1f981df977b08c935df8df5914b1](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\f3f1f981df977b08c935df8df5914b1.png)

## 5.3记录在页中的数据

一开始生成页的时候，并没有User Records部分；当插入一条数据时，会从Free Space划出一部份区域给User Records；当Free Space被使用完，表示这一页都被使用完了，再添加数据就需要申请新的页。

### 5.3.1记录头信息详情

记录头信息结构如下

![1648566353(1)](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\1648566353(1).png)

当插入三条数据时(c1,c2为int类型，c3为varchar(10000)，字符集使用ascii，行格式使用COMPACT)：

图片中每条记录从前往后数前六个属性分别是：deleted_flag，min_rec_flag，n_owned，heap_no，record_type，next_record

![c8127015ca817b80f533d43cff3b558](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\c8127015ca817b80f533d43cff3b558.png)

- **deleted_flag**：这个属性描述该条记录是否已经被删除，已经被删除为1，否则为0。也就是说，记录被删除后只是这个属性置1，被没有从磁盘中删除。所有被删除掉的记录会组 成一个垃圾链衰，记录在这个链表中占用的空间称为可重用空间。
- **min_rec_flag**： B+树每层非叶子节点中的最小的目录项记录都会添加该标记，0说明这一记录不是 B+树每层非叶子节点中的最小的目录项。
- **n_owned**：
- **heap_no**：我们向表中插入的记录从本质上来说都是放到数据页的User Records部分。例如：

![8f5ac3b5a4c35875400fd888172f72e](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\8f5ac3b5a4c35875400fd888172f72e.png)

​	记录一条一条亲密无间排列的结构被称为堆。每条记录在堆中的相对位置就是heap_no。每新申请一条记录的存储空间时，该条记录比**物理位置**在它前边的那条记录的heap_no值大1。用户插入的列的heap_no的最小值是2，因为heap_no为0和1的记录是被自动添加到数据页里的，heap_no为0的那条记录是数据页中最小的记录，被称为Infimum记录；heap_no为1的那条记录是数据页中最大的记录，被称为作Supremum记录。**值得注意的是，记录之间也可以比大小。**对于一条完整的记录来说，比较记录的大小就是比较主键的大小。Infimum和Supremum记录的构造很简单，是由5字节大小的记录头信息和8字节大小的一个固定单词组成的，8字节的固定单词分别对应了Infimum和Supremum。堆中记录的heap_no值在分配之后就不会发生改动了，即使之后删除了堆中的某条记录，这条被删除记录的 heap_no值也仍然保持不变。

- **record_type**：这个属性表示当前记录的类型.一共有4类型的记录 其中0表示普通记录，1表示 B+ 树非叶节点的目录项记录，2表示lnfimum记录，3表示 Supremum 记录。

- **next_record**：它表示从当前记录的真实数据到下一条记录的真实数据的距离。若该属性为正值，说明当前记录的下一条记录在当前记录的后面；如果该属性值为负数，说明当前记录的下一条记录在当前记录的前面。例如一条记录的next_record值为32，说明从这条记录后找32字节就是下一条记录的真实数据。需要注意的是，这里的下一条记录不是按照插入顺序来排的，而是按照主键大小来排的。Supremum的next_record值为0. 也就是说Supremum记录之后就没有下一条记录了。

**接下来请看删除一条记录后发生了什么**：当删除了数据页中第二条记录，那么①第二条记录的deleted_flag置1；②第一条记录的next_record指向第三条记录；③第二条记录的next_record置0；④Supremum的n_owned减少了1。请注意这里第二条记录的heap_no并没有改变。当把这条记录在重新插回表中，InnoDB 并没有因为新记录的插入而为它申请新的存储空间 而是直接复用了原来被删除记录的存储空间。

## 5.4Page Directory（页目录）

InnoDB查找记录时不是简单地直接遍历User Records，而是采用了一些别的方法，这个方法类似于书籍的“根据目录查询具体页码”。具体操作如下：

1. 将所有正常的纪录（包括 Infimum，Supremum 记录，但不包括已经移除到垃圾链表的记录）划分为几个组。
2. 将每个组最大的那个记录的头信息中的n_owned属性就表示这个组内共有几条记录。
3. 把每组最后一条记录的地址偏移值取出来，按顺序**逆序**存储到靠近页尾部的地方，这个地方就是 Page Directory，也就是页目录。页目录中这些地址偏移值被称为槽(Slot)，**每个槽占用2字节**。页目录就是由多个槽组成的。（每个槽对应一组）

**划分分组的依据**：

> 1. 对于Infimum记录所在的分组只能有1条记录；
> 2. Supremum记录所在的分组拥有的记录条数只能在1~8条之间；
> 3. 其他的分组中记录的条数范围只能是在4~8条之间

**划分分组的过程**：

> 初始情况下，数据页中只有两条记录：Infimum 记录和 Supremum 记录。因此此时只有两个槽。
>
> 之后每插入一条记录，**都会从页目录中找到对应记录的主键值比待插入记录的主键值大并且差值最小的槽。**然后把那个槽对应的记录的n_record值加一，表示本组由多了一条记录，知道这一组的记录数等于8。
>
> 当一组的记录数等于8时再加一条记录，会将组中的记录拆分为两个组，一个组4条记录，一个组5条记录。这个拆分过程会在页目录中新增一个槽，记录这个新增分组中最大的那条记录的偏移量。

查询一条主键为6的记录的过程（案例表中包括Infimum 记录和 Supremum 记录在内共有18条记录，另外的16条记录主键按照1-16排列）：

> 首先知道18条记录对应的槽有5个(从0开始计数)，计最低的槽为0，记为low；最高的槽为4，记为high
>
> 计算中间槽的位置(0+4)/2=2，查看槽2对应的记录的主键为8，令high为2，low不变
>
> 计算中间槽的位置(0+2)/2=1，查看槽1对应的记录的主键为4，令low为1，high不变
>
> 此时high和low只差1，因此要查询的主键就在第2个槽里，此时需要从第2个槽所在的分组里的主键最小的开始查找4个，从槽1的最后一个开始往后查，查询到主键为6的记录

综上所述，一个数据页查找指定主键的记录时，先二分法找到所在的分组对应槽在什么地方，再遍历这一个组里的记录。

## 5.5Page Header

占用56字节，专门存储各种状态信息

![bbb217257a5373e2dd3af3519a9bfca](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\bbb217257a5373e2dd3af3519a9bfca.png)

需要注意的是：

**PAGE_DIRECTION**：假如新插入的一条记录的主键值比上一条记录的主键值大，我 们说这条记录的插入方向是右边，反之则是左边.用来表示最后一条记录插入方向的状态就是PAGE_DIRECTION。

**PAGE_N_DIRECTION**：假设连续几次插入新记录的方向都是一致的，InnoDB 会把沿着同一个方向插入记录的条数记下来，这个条数就用 PAGE_N_DIRECTION状态表示。当然，如果最后一条记录的插入方向发生了改变，这个状态的值会被消零后重新统计。

## 5.6File Header

每部分占用38字节，描述页的通用信息。

![7afb3392a23cfd4af527fe1b032b47a](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\7afb3392a23cfd4af527fe1b032b47a.png)

**FIL_PAGE_SPACE_OR_CHKSUM**：这个属性代表当前页面的校验和( checksum)。校验和就是对于一个很长的字节串来说，我们会通过某种算法计算出一个比较短的值来代表这个很长的字节串，这个比较短的值就称为校验和。这样在比较两个很长的 字节串之前，先比较这两个长字节串的校验和。如果校验和都不一样 ，则两个长字节串肯定是不同的，这样就省去了直接比较两个长字节串的时间损耗。

**FIL_PAGE_OFFSET**：一个页都有一个单独的页号，如同我们的身份证号码一样. InnoDB 通过页号来唯一定位一个页。

**FIL_PAGE_TYPE**：表示当前页的类型。

**FIL_PAGE_PREV和FIL_PAGE_NEXT**：就是本数据页的前一页和后一页。因此数据页也是双向链表结构。

## 5.7 File Trailer 文件尾部

该部分8字节组成，分为以下两个部分

**前4个字节**：页的校验和，这部分要与File Header中的校验和对应。每当一个页在内存中发生修改时，在刷新之前就要把页面的校验和算出来。因为File_Header在页 面的前边，所以 File Header 中的校验和会被首先刷新到磁盘，当完全写完后，校验和 也会被写到页的尾部.如果页面刷新成功，则页首和页尾的校验和应该是一致的。如果刷新了一部分后断电了 ，那 File Header 中的校验和就代表着己经修改过的页，而 File Trailer 中的校验和代表着原先的二者不同则意味着刷新期间发生了错误.

后4字节代表页面被最后修改时对应的 LSN 的后 字节，正常情况下应该与 File Header 部分的 FEL_PAGE_LSN 的后4字节相同.这个部分也是用于校验页的完整性。

# 六、B+树索引

## 6.1没有索引进行查找

- 当数据量比较少，只有一页数据页时
  * 以主键为搜索条件时：二分查找
  * 以非主键为搜索条件：在数据页中遍历

* 当数据量比较多，有好多页数据页时分为两个步骤

  * 遍历定位到记录所在的页；

  * 从所在的页内查找相应的记录

## 6.2索引

先创建一个表

![66e1a2ceb483e79c3ef6965a9223877](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\66e1a2ceb483e79c3ef6965a9223877.png)

存储结构如下所示：

![4d22a4ce3c3e29f80124236afbd9b78](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\4d22a4ce3c3e29f80124236afbd9b78.png)

### 6.2.1一个简单的索引方案

若想快速定位到需要查找的记录在哪些数据页中，可以想办法为快速定位记录所在的数据页而建立一个别的目录，在建这个目录的过程中必须完成两件事：

* **下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值。**

当数据页被填满时，再插入数据时会再申请一个新页，需要注意的是，申请的新页未必与上一张表的页号差1，或许上一张页号是11，下一张就是28。他们只是建立了一个链表关系。此时，要满足**下一个数据页中用户记录的主键值必须大于上一个页中用户记录的主键值。**，就需要把上一张页中主键大于下一张页的记录全部移到下一张页中，相应地将下一张页的记录移到上一页中。这个过程也叫做**页分裂**。例如

![eb9bb32d892c50d5d5f32503a006b2a](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\eb9bb32d892c50d5d5f32503a006b2a.png)

------------------------------------------------------------------------------------------**移动前**---------------------------------------------------------------------------------------

![5f94119b63d1dfdcae8f1614658158f](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\5f94119b63d1dfdcae8f1614658158f.png)

------------------------------------------------------------------------------------------**移动后**---------------------------------------------------------------------------------------

* **给所有的页建立一个目录项**

如果想从这么多页中根据主键值快速 定位某些记录所在的页，就需要给它们编制一个目录，每个页对应一个目录项，每个目录项包含两部分：页中最小的主键，记为key；页号，记为page_no

在插入许多记录之后，如图所示：

![19326e1084057a6486bb32b78b22d03](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\19326e1084057a6486bb32b78b22d03.png)

编制目录后：

![b9712950708be901aff3d858e575cb8](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\b9712950708be901aff3d858e575cb8.png)

此时即使有多个数据页的记录，根据主键查找记录也很方便，只需要二分法找出记录所在的数据页对应在哪个目录项，找出这一页，进入这一页后再按照二分查找找到对应的记录。这个目录就叫做**索引**。

### 6.2.2InnoDB的索引方案

上述的索引方案存在两个问题：

1. InnoDB使用数据页作为管理存储空间的基本单位，当记录数越来越多，目录项的数量也将越来越多，这样就无法把目录项全部放到一个连续的物理空间里（数组存储）。
2. 我们时常会对记录增删查改，如果把一张数据页里的数据全部删完，那么这一页也就没有存在的必要，这也需要把这一页之后对应的目录项全部前移一位，或者把这一目录项作为冗余放在目录项列表里，但这会浪费许多空间。

由于目录项的形式很像记录，因此InnoDB复用了数据页来存储目录项。为了与普通记录区分开，把这些用来表示目录项的记录称为**目录项记录**，区分一条记录是否为目录项记录的方式就是看记录头信息中**record_type**属性是否为1，是1就是目录项记录，0就是普通记录，2就是Infimum记录，3就是Supremum记录。（注意，目录项记录只有两列，主键值和页号，另外记录头信息中一个min_rec_flag的属性只有当这个记录为目录项记录时才有可能为1）。

![1649387041](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\1649387041.png)

现在查询主键就很方便了（例如查询主键为20）：

1. 先到存储目录项的数据页中，即页30，在页目录里二分法查找到对应的分组，进而获得对应的目录项记录，对应的12<20<200，因此主键为20的记录一定就在12对应的页号里，进而确定该目录对应页号。
2. 进入该页，根据二分法定位到主键为20的用户记录。

当存储数据页的页数不止一页时，查询主键的过程如下：

1. 确定存储目录项记录的页。
2. 通过存储目录项记录的页确定用户记录真正所在的页。
3. 在真正存储用户记录的页中定位到具体的记录。

当目录项太多了，就为这些存储目录项记录的页也设置一层更高级的目录，如下图所示：

![ce0d531f49724de7500062aa077b9a8](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\ce0d531f49724de7500062aa077b9a8.png)

这个页中的两条记录分别代表页30和页32. 如果用户记录的主键值在 [1，320) 之间，则到页 30 查找更详细的目录项记录;如果主键值不小于32 ，就到页 32 中查找更详细的目录项记录。这样的数据结构就是**B+树**。

从上图中可以看出用户的真实数据存放在B+树的最底层，这一层被称为第0层，层级依次往上加。倘若一个普通数据页存放的记录是100条，一个目录项数据页存储的目录项1000条，那么如果B+树只有1层，最多放100条用户记录，如果有两层，最多放1000\*100条记录，如果有三层最多放1000\*1000\*100条记录，四层则最多放1000\*1000\*1000\*100条记录。由于一般来说数据量不会那么大，**B+树的层数都不会超过4层**。

#### 6.2.2.1聚簇索引

在使用MySQL时使用聚簇索引不需要显式地使用index语句来创建，InnoDB引擎会**自动**为我们创建。在InnoDB存储引擎中，聚簇索引就是数据存储方式(所有的用户记录都存储在了叶子节点) ，也就是所谓的"索引即数据，数据即索引“。

满足以下两个特点的就是聚簇索引：

* 使用记录主键值的大小进行记录和页的排序，（概括起来就是**数据页里的记录要按照主键排成单向链表，B+树每一层的页都要按照逐渐大小排成双向链表**）这包括3方面的含义
  * 页内的记录按照**主键**大小排成一个单向链表，页内的记录被划分为若干组，每组主键值最大的记录的偏移量被记录在页目录的槽里。
  * 各个存放用户记录的页也是根据页中用户记录的主键大小顺序排成一个双向链表。
  * 存放目录项记录页分为不同层级，同一层级中的页也是根据页目录项记录的主键大小顺序排成一个双向链表。

* B+ 树的叶子节点存储的是**完整的用户记录**。所谓完整的用户记录，就是 这个记录中 存储了所有列的值(包括隐藏列).

#### 6.2.2.2二级索引

聚簇索引有局限性，就是它只能在搜索条件为主键的时候才可以使用。二级索引则可以解决这个问题。

要查询别的属性，可以多建几棵B+树，不同的B+树采用不同的排序方式，例如可以使用c2列的大小作为排序条件再建一棵B+树。

![1edf8c945c7e17eacbdf7686ff05191](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\1edf8c945c7e17eacbdf7686ff05191.png)

这棵B+树与聚簇索引有所不同：

* 使用记录c2列的大小进行记录和页的排序，这包括3方面的含义（与聚簇索引相比，只是在排序规则上有所不同）。
  * 页内的记录按照**c2列**大小排成一个单向链表，页内的记录被划分为若干组，每组主键值最大的记录的偏移量被记录在页目录的槽里。
  * 各个存放用户记录的页也是根据页中用户记录的**c2列**大小顺序排成一个双向链表。
  * 存放目录项记录页分为不同层级，同一层级中的页也是根据页目录项记录的**c2列**大小顺序排成一个双向链表。

* B+树的叶子节点存储的并**不是完整**的用户记录，而只是 c2 列+主键这两个列的值（与聚簇索引差距在叶子节点的存储结构）
* 目录项记录中不再是主键+页号的搭配，而变成了 c2 列+页号的搭配。

需要注意的是，由于c2这一列没有唯一性，满足c2搜索条件的记录可能有很多，我们只需要找到第一条符合条件的记录，再沿着单向链表一直遍历下去即可；如果到了页尾还没有遍历完可以通过数据页之间的双线链表遍历下一页。

例如查询c2=4.

1. 首先从根页面开始，2<4<9，因此要查找的目录项在2对应的42页

2. 进入42页，2<4<=4，因此要查找的记录在2对应的页34或4对应的35

3. 进入34页和35页，找到c2=4的第一条记录，案例中是在34页。
4. 定位到第一条符合条件的记录后，找到这条记录的主键，并使用聚簇索引找到该主键对应的完整记录。接着再回到这棵B+树的叶子节点，找到刚才那条符合条件的记录，沿着链表向后继续搜索，后续搜索到的符合条件的记录也进行上述的操作，直至后续没有符合条件的记录。

这个在二级索引中通过携带主键信息到聚簇索引中重新定位完整的用户记录的过程也称为**回表**。回表的的意义在于节省了存储空间，如果把完整的用户记录放到叶子节点是可以不用回表，但是太占地方了。

这样以非主键大小为排序规则的并需要回表操作的B+树就是**二级索引**，也叫**辅助索引**。在这个案例中把这棵树叫做c2的索引，c2被叫做**索引列**。

#### 6.2.2.3联合索引

我们也可以同时以多个列的大小作为排序规则，也就是同时为多个列建立**一个**索引。

B+树彼此间的差距就在于排序规则的不同，二级索引按照索引列的大小来排序，联合索引则按照多个列的大小来排序。例如选定c2，c3为索引列，此时每个目录项的记录不止有c2的值和页号，还有c3的值，每条记录先按照c2的列排序，若c2相同，则按照c3大小排序。

注意，联合索引只建立一棵B+树，而不是分别建立一棵B+树。

### 6.2.3InnoDB中B+树索引的注意事项

**1.根页面不动**

B+树的形成过程：

1. 每当为表创建一个B+树索引时，都会为为索引创建一个根节点页面。一开始表中没有数据，每个 B+ 树索引对应的根节点中既没有用户 ，也没有目录项记录。
2. 随后向表中插入用户记录时 先把用户记录存储到这个根节点中。
3. 在根节点可用空间分配完毕之后继续插入记录，此时会将根节点内的全部记录都移到一个新分配的页，然后对这个新页进行页分裂操作，得到另一个新页。根节点此时升级成为存储目录项的数据页，这需要把新的两个页的目录项都记录到根节点。

总的来说，**一个B+树索引一旦建立，它的根节点的页号就不会改变**。日后凡是InnoDB需要用到这个索引时，都会从那个页号对应的页取得相应数据。

**2.内节点中目录项记录的唯一性**

实际上二级索引中内节点中的目录项记录有三个属性：索引列属性，主键，页号。有主键让该目录项拥有唯一性，这是为了解决这样一个场景问题：

当前一页的最后一个记录的c2值与下一页第一个记录的c2值相同时，再新增一个具有相同c2值的记录，要把新增的记录放在哪页？在拥有主键属性后就可以比较这些记录，将c2值相同的记录进行比较，进而确定这一新增记录要被插入到哪一页。

总的来说，就是，在二级索引值相同的情况下，再按照主键值进行排序，本质上是一个联合索引。对于唯一二级索引(当某个列或列组合声明 UNIQUE 时，使会为这个列或列组合建立唯一二级索引 )也可能出现多条记录的键值相同，因此唯一二级索引的内节点的目录项记录也有主键值。

**3.一个页面至少容纳2条记录**

如果一个页面只能容纳1条记录，那么每次插入新纪录的时候都要进行页分裂，相应地也要频繁地进行根页面的分裂，这会导致进行了复杂的操作后，存储的数据却没几条，造成了性能的浪费。

### 6.2.4MyISAM中的索引方案简介

MyISAM 索引方案虽然也使用树形结构，但是却将索引和数据分开存储。

- 将表中的记录按照记录的插入顺序单独存储在一个文件中(称之为数据文件)，有多少记录就塞多少记录进去。
- 使用MyISAM存储引擎的表会把索引信息单独放在一个文件（索引文件）。MyISAM会为表的主键单独创建一个索引，只是在索引的叶子节点存放的不是完整的用户记录，而是主键和行号的组合。也就是先通过索引找到对应的行号，再通过行号去找对应的记录。这就经历了一次回表操作，**相当于MyISAM中建立的索引全是二级索引。**

### 6.2.5MySQL中创建删除索引的语句

创建表时指定建立索引的单个列或建立联合索引的多个列

> CREATE TALBE 表名(
>
> ​	各个列的情息,
>
> ​	(KEY| INDEX) 索引名 (需要被索引的单个列或多个列)
>
> )

修改表结构时添加索引

> ALTER TABLE 表名 ADD (INDEX | KEY) 索引名(需要被索引的单个列或多个列);

修改表结构时删除索引

> ALTER TABLE 表名 drop (INDEX | KEY) 索引名

# 七、B+树索引的使用

## 7.1B+树索引示意图的简化

首先新建一张表

> CREATE TABLE single_table(
> 	id INT NOT NULL AUTO_INCREMENT,
> 	key1 VARCHAR(100),
> 	key2 INT,
> 	key3 VARCHAR(100),
> 	key_part1 VARCHAR(100),
> 	key_part2 VARCHAR(100),
> 	key_part3 VARCHAR(100),
> 	common_field VARCHAR(100),
> 	PRIMARY KEY (id),
> 	KEY idx_key1 (key1),
> 	UNIQUE KEY uk_key2(key2),
> 	KEY idx_key3(key3),
> 	KEY idx_key_part(key_part1, key_part2, key_part3)
> ) ENGINE = INNODB, CHARSET=utf8;

这里一共有5份索引，1份是自动生成的聚簇索引，还有4份二级索引(其中有一个联合索引)

以下是聚簇索引的优化图：

![71f05d73b6633d23a6dc0ea59570f13](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\71f05d73b6633d23a6dc0ea59570f13.png)

以下是二级索引的优化图：

![cf40a239d544a92d4faaadabe65d75b](C:\Users\MagicBOOK\Desktop\笔记\mysql图片\cf40a239d544a92d4faaadabe65d75b.png)

## 7.2索引的代价

1. 空间上的代价：每建立一个索引就要建立一棵B+树，就要使用大量的数据页，占用大量空间。
2. 时间上的代价：如果建立了很多索引，由于对数据进行增删改的过程中也会涉及到B+树的维护，这会让存储引擎使用很多时间和空间处理页面分裂、页面回收等操作以维护节点和记录的排序。

## 7.3应用B+树索引

### 7.3.1扫描区间和边界条件

搜索条件一般可以这样被概括：where 属性 = 值或where属性 > 值或where属性 < 值，例如

> select * from single_table where id>2 and id <100

那么也就是说要查找id值在[2,100]之间的记录，在这里[2,100]这个区间被称为**扫描区间**，id>2和id<100被称为扫描区间的**边界条件**。对于下述例子：

> SELECT * from single_table WHERE key2 IN (1438. 6328) OR (key2 >= 38 AND key2 <= 79) ; 

一共三个扫描区间，分别是[1438],[6328],[38,79]。方便起见，将区间内只有一个点的叫做**单点扫描区间**，包含多个值的叫做**范围扫描区间**。要注意的是，**只有关于索引列的搜索条件才能构成扫描区间和边界条件**，其他列是不行的。例如：

> SELECT * FROM single_table WHERE key1 < ' a ' AND key3 > 'z' AND common_field = 'abc ';

对于这样的查询语句，由于key1和key3都有索引，因此分两种情况判断：

1. 若是使用idx_key1执行查询，则扫描区间就是(-∞,'a']，边界条件就是key1<'a'，key3 > 'z' 和 common_field = 'abc '就是普通的搜索条件。在获得idx_key1的二级索引记录并回表后，获取完整的记录后才能判断该记录是否符合普通的搜索条件。
2. 使用idx_key3执行查询步骤同上。

由上述可知，当使用索引执行查询时，关键就是**先找出合适的扫描区间，再回表通过聚簇索引的B+树得到完整记录后，扫描这些记录是否满足普通查询条件**。

like操作符也会产生扫描区间，不过只在匹配完整的字符串或者匹配字符串前缀时才产生合适的扫描区间。例如like 'a%'由于对于二级索寻idx_key1来说，所有字符串前缀为'a'的二级索引记录肯定是相邻的。那么key1 like 'a%'形成的扫描区间相当于['a','b)，**当like的字符串不是匹配前缀的，即'%suf%‘这种形式的，扫描区间就是(-∞,+∞)**

* AND 和 OR 对扫描区间选取的影响，例如key1 > 100 ? common_field ='abc'，都使用idx_key1作为查询索引
  * 当连接符为and时，即key1 > 100 AND common_field ='abc'时，这时需要取交集，由于common_field ='abc'根索引没关系，它的扫描区间是(-∞,+∞)，可以直接看做是TRUE，那么整个的扫描区间就是[100,+∞)。
  * 当连接符为or时，即key1 > 100 OR common_field ='abc'时，这是取并集，也就是(-∞,+∞)∪[100,+∞)=(-∞,+∞)，这时也可以将common_field ='abc' 看做TRUE。当然，由于需要扫描二级索引的全部记录，还要回表，效率还不如直接全面扫描聚簇索引，这时不会采用idx_key1索引。

根据以上的知识来简化下列例子：

> select * from single_table where (key1 > 'ayz' and key2 = 748) or (key1 < 'abc' and key1 > 'lmn') or (key1 like '%suf' and key1 > 'zzz' and (key2 < 8000 or common_field = 'abc')) 

首先看涉及哪些列：key1，key2，common_field。其中key1有二级索引idx_key1，key2有唯一二级索引idx_key2。

**当使用idx_key1执行查询时**：化简为

> select * from single_table where (key1 > 'xyz' and TRUE) or (key1 < 'abc' and key1 > 'lmn') or (key1 like '%suf' and key1 > 'zzz' and (TRUE or TRUE)) 

> key1 > 'xyz'  or (key1 < 'abc' and key1 > 'lmn') or (key1 like '%suf' and key1 > 'zzz' ) 

在这里可以看出(key1 < 'abc' and key1 > 'lmn')永远为false，key1 like '%suf'为TRUE，进一步化简

> key1 > 'xyz' or key1 > 'zzz'

这里再取交集，也就是key1 > 'xyz'，因此整体的扫描区间就是('xyz',+∞)。

**当使用idx_key2执行查询时**类似地，得到where TRUE，这就意味着需要扫描二级索引的全部记录，还要回表，得不偿失。

### 7.3.2对于联合索引的分析

在案例中key_part1，key_part2，key_part3三个列有一个联合索引，请看以下查询语句。

> SELECT * FROM single_table WHERE key_part1 = ' a' AND key_part2='b';

扫描区间是[['a','b'],['a','b']]。

> SELECT * FROM single_table WHERE key_part1 < 'a'; 

扫描区间是(-∞,'a')

> SELECT * FRCM single_table WHERE key_part2 ='a' ; 

由于二级索引记录不是直接按照key_part2的值来排的，因此不能通过这个搜索条件来减少扫描的记录。因此不使用这个索引。

### 7.3.3索引用于排序

在使用order by语句给数据排序时，一般情况下需要把数据都加载到内存里，再用一些排序算法在内存中对数据排序。有时查询的结果集可能太大以至于无法在内存中进行排序，此时就需要暂时借助磁盘的空间来存放中间结果，在排序操作完成后再把排好序的结果集返回客户端。

在MySQL中，在内存和磁盘里排序的方式被称为**文件排序**。当order by语句使用了索引的话，就有可能省去文件排序。例如

> SELECT * FROM single_table ORDER BY key_part1, key_part2, key_part3 LIMIT 10; 

 该查询语句需要先按key_part1排序，再按照key_part2、key_part3 排序。然而实际上，在这里建立的联合索引就已经在叶子节点排好序了，直接按照索引的顺序取出记录再回表，取得完整的记录后返回客户端即可。

#### 7.3.3.1使用联合索引排序的注意事项

排序的字段顺序必须要与索引列的顺序一样。要不然不能使用索引排序。

另外，当order by前已经指定了前几个索引列的值，并对后面索引列排序时，也可以使用索引排序。例如：

> SELECT * FROM sinle_table WHERE key_part1 = 'a' AND key-part2 = ' b ' OROER BY key_part3 LlMlT 10;

#### 7.3.3.2不可以使用索引排序的情况

order by也可以降序排序。

原理是：首先找到符合条件的第一条记录，然后找到这个分组里最大的一条记录，找到这个记录对应的槽，再找这个槽的上一个槽，上一个槽对应的记录的下一条记录就是这个分组里第一条记录，以此类推可以逆序输出。

**1.asc，desc混用**

对于使用联合索引排序的场景，要求每个排序列的排序规则都是asc或者desc。

**2.排序列包含非同一个索引的列**

例如SELECT * FROM single_table ORDER BY key1 , key2 LIMIT 10;

对于idx_key1二级索引，只按照key1排序，不按照key2排序，因此不能使用idx_key1排序

**3.排序列是某个联合索引的索引列 但是这些排序列在联合索引中并不连续**

例如

SELECT * FROM single_table ORDER BY key_part1 , key_part3 LIM1T 10; 

这样的排序语句，不可以使用索引。

**4.用来形成扫描区间的索引列与排序列不同**

**5.序列不是以单独列名的形式出现在ORDER BY子句中，例如order by UPPER(key1)等带了函数或进行其他操作的不可以使用索引排序**

### 7.3.4索引用于分组

SELECT key_part1, key_part2 , key_part3, COUNT(*) FROM single_table GRQUP BY key-part1,key_part2 , key_part3;

这个查询语句相当于执行了3次分组操作

- 首先把key_part1记录分组，把key_part1相同的值划分为一组；
- 在key_part1相同的分组里把key_part2值相同的记录划分为一组；
- 重复上一步划分key_part3分组

这里正好可以使用索引，如果不能使用索引则需要新建一张临时表，把扫描聚簇索引时的数据填充到这张表，接着再发送到客户端。

## 7.4回表的代价

对于InnoDB引擎来说，每个数据页都被存放在磁盘的一个或多个文件里，每个数据页的页号代表了这页在这些文件里的偏移量，例如页号为0，偏移量为0；页号为1，偏移量为16KB。看以下例子

> select * from single_table where key1 > 'a' and key1 < 'b'

对于这样一个查询语句，使用idx_key1索引来查询，在查询二级索引记录时，由于这些数据是集中在一起的，因此查询起来的速度很快，一次页面IO就可以查询出很多二级索引记录；但在回表的时候，由于在聚簇索引里查询依靠的是主键值，二级索引记录中提供的主键值是杂乱无序的，因此在聚簇索引中必须要执行很多次随机IO，造成性能下降。因此需要行回表操作的记录越多，使用二级索引进行查询的性能也就越低，某些查询宁愿使用全表扫描也不使用二级索引。

在查询时，是采用全表查询还是使用二级索引+回表，选择权在于查询优化器。优化器的大致执行过程：针对表中记录计算一些数据，后再利用这些统计数据或者访问表中的少量记录来计算需要执行回表操作的记录数，根据这些记录数来判断采用何种方式。

一般来说，使用了limit子句由于查询的数据更少，更容易让查询优化器采用二级索引+回表的方法。

## 7.5更好地创建和使用索引

1. **只为用于搜索、排序、分组的列创建索引**
2. **考虑索引列中不重复值的个数**：当索引列中重复的值太多，那么每次查询可能会查询过多记录，导致回表效率低下。
3. **索引列的类型尽量小**：当索引列类型占用的空间越小，一个数据页中就能存更多数据，一次页面IO就可以将更多记录加载到内存里，索引占用的空间也越小。
4. **可以为列前缀建立索引**：使用ALTER TABLE single_table ADD INDEX idx_key1(key1(10));来使索引的建立依据变为key1前10格字符。当列中存储的字符串包含的字符较多时，这种为列前缀建立索引的方式 可以明显减少索引大小。
5. **覆盖索引**：为了减少回表的性能损耗，建议在查询时只查询索引列和主键。
6. **让索引列以列名的形式在搜索条件中单独出现**：对于  SELECT * FROM sngle_table WHERE key2 / 2 < 4;  SELECT * FROM sngle_table WHERE key2 < 4/2; 两条查询语句来说，MySQL 并不会尝试简化 key2*2<4 表达式，而是直接认为这个搜索条件不能形成合适的扫描区间来减少需要扫描的记录数量，所以该查询语句只能以全表扫描的方式来执行。
7. **新插入记录时主键依次递增**：一个数据页如果已经满了，那么如果再插入一条记录，这条记录的主键决定了这条数据应该被插入在数据页的中间位置，这就意味着需要分裂页，造成性能损耗。
8. **避免冗余和重复索引**：如果已经为一些列设立了联合索引，又为第一个索引列设立了二级索引，那么这个二级索引就是冗余的；如果一个列就是主键，又为它设立了二级索引，那么这个索引就是重复的。

