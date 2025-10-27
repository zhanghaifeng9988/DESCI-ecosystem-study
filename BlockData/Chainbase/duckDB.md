# DuckDB 简介
DuckDB 是一个开源的嵌入式关系型数据库管理系统，专为数据分析和处理任务优化，具有以下显著优势：

## 核心特点
🚀 高性能
利用现代 CPU 架构的矢量化执行引擎、列式存储及高效的查询优化技术，可高效处理 GB 至 TB 级别数据。
内存计算：数据可完全存储在内存中处理，采用向量化查询执行引擎
🧩 轻量嵌入
单个二进制文件，依赖极少，适合嵌入各种本地应用程序或脚本中，无需服务端部署。

🛠️ 易用性强
完全支持标准ANSI  SQL，具备 Python、R、Node.js、C++ 等多语言接口，适合数据科学与工程任务。
Python集成：通过duckdb模块直接操作

💾 混合存储架构
支持内存优先的查询执行，亦可将数据持久化至本地磁盘（.duckdb 文件），兼顾性能与持久性。


## 推荐使用场景
📊 本地数据分析：替代 Pandas、SQLite 等工具，适合分析师和科学家处理中等规模数据集。
📦 嵌入式数据应用：构建需要本地数据处理能力的桌面或 Web 应用。
🏢 轻量数据仓库：中小企业可使用其作为单机型数据仓库或 ETL 工具的中间缓存。

## DuckDB示例
以下是如何使用 DuckDB 的一些具体示例：

分析财务数据：DuckDB 可用于计算财务比率、跟踪趋势并识别异常。
分析日志数据：DuckDB 可用于识别日志数据中的模式、解决问题并跟踪用户行为。
分析科学数据：DuckDB 可用于对科学数据执行复杂的计算、生成可视化并构建预测模型。

# 本地安装相关（linux）
环境：Rocky Linux 9.3	  4c 2G
## linux当前用户下执行：
curl https://install.duckdb.org | sh

## 添加环境变量 
[zhf@localhost ~]$ cd .duckdb/cli/1.3.2 
程序这个目录中，cli目录下有软连接
将这个软件链接，作为环境变量，添加到/etc/profile.d/duckdb.sh 脚本文件中，
脚本的添加内容：export PATH="$PATH:/home/zhf/.duckdb/cli/latest"

其他相关内容展示：
[zhf@localhost 1.3.2]$ ll
total 50256
-rwxr-xr-x. 1 zhf zhf 51459336 Aug 21 09:37 duckdb
[zhf@localhost 1.3.2]$ du -sh ./*
50M     ./duckdb
[zhf@localhost 1.3.2]$ pwd
/home/zhf/.duckdb/cli/1.3.2

##  启动数据库
[zhf@localhost 1.3.2]$ duckdb
DuckDB v1.3.2 (Ossivalis) 0b83e5d2f6
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D

其中D就是命令行提示符，很特别吧！哈哈
英文提示的翻译是：输入“.help”获取使用提示。
已连接到临时内存数据库。
使用“.open 文件名”重新打开持久数据库。
**默认情况下，DuckDB 是运行在内存数据库中**，这意味着创建的任何表都存储在内存中，
而不是持久化到磁盘上。
**可以通过启动命令行参数的方式，将 DuckDB 连接到磁盘上的持久化数据库文件。**
任何写入该数据库连接的数据都将保存到磁盘文件中，并在重新连接到同一文件时重新加载。

.quit  退出
show databases;
┌───────────────┐
│ database_name │
│    varchar    │
├───────────────┤
│ memory        │
└───────────────┘

.open my.db  创建一个名字叫做my的数据库，如果数据库存在，则打开该数据库。
**注意，这里的数据库是内存数据，重新登录就没有了。**

## 主要命令
show tables; 显示当前数据库中的所有表
.tables  显示当前数据库中的所有表
.schema 显示当前数据库中的所有表的结构

# 连接数据库，测试数据导入
## 1. 给数据库中导入csv文件类型的数据
**注意**，zhf用户就是数据库安装的用户，测试文件也是在这个用户的home目录下进行创建。

### 创建csv文件
[zhf@localhost ~]$ echo "first_name,last_name,age" > test.csv; \
echo "Zhang,San,57" >> test.csv; \
echo "Li,Si,48" >> test.csv; \
echo "Wang,Wu,23" >> test.csv

[zhf@localhost ~]$ cat test.csv
first_name,last_name,age
Zhang,San,57
Li,Si,48
Wang,Wu,23

### 查询csv文件内容，并将内容导入数据库的表中
[zhf@localhost ~]$ duckdb my.db    --创建持久化数据库文件，打开持久化数据文件
DuckDB v1.3.2 (Ossivalis) 0b83e5d2f6
Enter ".help" for usage hints.
D SELECT * FROM read_csv_auto('test.csv');
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int64 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │
└────────────┴───────────┴───────┘
D SELECT * FROM 'test.csv';
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int64 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │
└────────────┴───────────┴───────┘
D SELECT * FROM read_csv('test.csv', delim=',', header=true, columns={'first_name': 'VARCHAR', 'last_name': 'VARCHAR','age':'INT'});
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int32 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │
└────────────┴───────────┴───────┘
D CREATE TABLE test_csv (first_name VARCHAR, last_name VARCHAR, age INT);
D COPY test_csv FROM 'test.csv' (AUTO_DETECT true);
D SELECT * from test_csv;
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int32 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │
└────────────┴───────────┴───────┘




## 2. 给数据库中导入Parquet文件类型的数据
Parquet用于数据压缩和查询性能的高效列格式。

### 安装依赖包
[root@localhost ~]# pip3 install pandas
[root@localhost ~]# pip3 install pyarrow
[root@localhost ~]# pip3 install fastparquet

### 创建Parquet文件
[zhf@localhost ~]$ python
Python 3.9.21 (main, Feb 10 2025, 00:00:00)
[GCC 11.5.0 20240719 (Red Hat 11.5.0-5)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import pandas as pd
>>> df = pd.read_csv('test.csv')
>>> df.to_parquet('test.parquet')
>>> quit()

[zhf@localhost ~]$ ls -l test*
-rw-r--r--. 1 zhf zhf   58 Aug 21 11:06 test.csv
-rw-r--r--. 1 zhf zhf 2646 Aug 21 13:59 test.parquet

### 查询Parquet文件内容，及其表解构
D select * from 'test.parquet';
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int64 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │
└────────────┴───────────┴───────┘

D SELECT * FROM read_parquet('test.parquet');
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int64 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │
└────────────┴───────────┴───────┘

D DESCRIBE SELECT * FROM 'test.parquet'; 查询表结构
┌─────────────┬─────────────┬─────────┬─────────┬─────────┬─────────┐
│ column_name │ column_type │  null   │   key   │ default │  extra  │
│   varchar   │   varchar   │ varchar │ varchar │ varchar │ varchar │
├─────────────┼─────────────┼─────────┼─────────┼─────────┼─────────┤
│ first_name  │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ last_name   │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ age         │ BIGINT      │ YES     │ NULL    │ NULL    │ NULL    │
└─────────────┴─────────────┴─────────┴─────────┴─────────┴─────────┘

DESCRIBE test.parquet;
┌─────────────┬─────────────┬─────────┬─────────┬─────────┬─────────┐
│ column_name │ column_type │  null   │   key   │ default │  extra  │
│   varchar   │   varchar   │ varchar │ varchar │ varchar │ varchar │
├─────────────┼─────────────┼─────────┼─────────┼─────────┼─────────┤
│ first_name  │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ last_name   │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ age         │ BIGINT      │ YES     │ NULL    │ NULL    │ NULL    │
└─────────────┴─────────────┴─────────┴─────────┴─────────┴─────────┘

SELECT * FROM parquet_metadata('test.parquet');
 SELECT * FROM parquet_metadata('test.parquet');
┌──────────────┬──────────────┬────────────────────┬──────────────────────┬─────────────────┬───┬─────────────────────┬─────────────────────┬──────────────┬──────────────┐
│  file_name   │ row_group_id │ row_group_num_rows │ row_group_num_colu…  │ row_group_bytes │ … │ bloom_filter_offset │ bloom_filter_length │ min_is_exact │ max_is_exact │
│   varchar    │    int64     │       int64        │        int64         │      int64      │   │        int64        │        int64        │   boolean    │   boolean    │
├──────────────┼──────────────┼────────────────────┼──────────────────────┼─────────────────┼───┼─────────────────────┼─────────────────────┼──────────────┼──────────────┤
│ test.parquet │            0 │                  3 │                    3 │             261 │ … │                NULL │                NULL │ NULL         │ NULL         │
│ test.parquet │            0 │                  3 │                    3 │             261 │ … │                NULL │                NULL │ NULL         │ NULL         │
│ test.parquet │            0 │                  3 │                    3 │             261 │ … │                NULL │                NULL │ NULL         │ NULL         │
├──────────────┴──────────────┴────────────────────┴──────────────────────┴─────────────────┴───┴─────────────────────┴─────────────────────┴──────────────┴──────────────┤
│ 3 rows                                                                                                                                             28 columns (9 shown) │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

### json
是一种开放标准文件格式和数据交换格式，DuckDB JSON 阅读器可以通过分析 JSON 文件自动推断要使用哪些配置标志。


[zhf@localhost ~]$ cat > test.json << "EOF"
[
{"first_name":"Zhang","last_name":"San","age":"57"},
{"first_name":"Li","last_name":"Si","age":"48"},
{"first_name":"Wang","last_name":"Wu","age":"23"}
]
EOF
[zhf@localhost ~]$ ll
total 548
-rw-r--r--. 1 zhf zhf  12288 Aug 21 11:43 my1.db
-rw-r--r--. 1 zhf zhf 536576 Aug 21 14:14 my.db
-rw-r--r--. 1 zhf zhf     58 Aug 21 11:06 test.csv
-rw-r--r--. 1 zhf zhf    156 Aug 21 15:01 test.json
-rw-r--r--. 1 zhf zhf   2646 Aug 21 13:59 test.parquet
[zhf@localhost ~]$ cat test.json
[
{"first_name":"Zhang","last_name":"San","age":"57"},
{"first_name":"Li","last_name":"Si","age":"48"},
{"first_name":"Wang","last_name":"Wu","age":"23"}
]


SELECT * FROM 'test.json';
┌────────────┬───────────┬─────────┐
│ first_name │ last_name │   age   │
│  varchar   │  varchar  │ varchar │
├────────────┼───────────┼─────────┤
│ Zhang      │ San       │ 57      │
│ Li         │ Si        │ 48      │
│ Wang       │ Wu        │ 23      │
└────────────┴───────────┴─────────┘
D SELECT * FROM read_json('test.json');
┌────────────┬───────────┬─────────┐
│ first_name │ last_name │   age   │
│  varchar   │  varchar  │ varchar │
├────────────┼───────────┼─────────┤
│ Zhang      │ San       │ 57      │
│ Li         │ Si        │ 48      │
│ Wang       │ Wu        │ 23      │
└────────────┴───────────┴─────────┘
D SELECT * FROM read_json('test.json', auto_detect=true);
┌────────────┬───────────┬─────────┐
│ first_name │ last_name │   age   │
│  varchar   │  varchar  │ varchar │
├────────────┼───────────┼─────────┤
│ Zhang      │ San       │ 57      │
│ Li         │ Si        │ 48      │
│ Wang       │ Wu        │ 23      │
└────────────┴───────────┴─────────┘
D SELECT * FROM read_json_auto('test.json');
┌────────────┬───────────┬─────────┐
│ first_name │ last_name │   age   │
│  varchar   │  varchar  │ varchar │
├────────────┼───────────┼─────────┤
│ Zhang      │ San       │ 57      │
│ Li         │ Si        │ 48      │
│ Wang       │ Wu        │ 23      │
└────────────┴───────────┴─────────┘
D  SELECT * FROM read_json_auto('test.json', format='array');
┌────────────┬───────────┬─────────┐
│ first_name │ last_name │   age   │
│  varchar   │  varchar  │ varchar │
├────────────┼───────────┼─────────┤
│ Zhang      │ San       │ 57      │
│ Li         │ Si        │ 48      │
│ Wang       │ Wu        │ 23      │
└────────────┴───────────┴─────────┘
D SELECT * FROM read_json_auto('test.json', format='unstructured');
┌───────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                     json                                                      │
│                         struct(first_name varchar, last_name varchar, age varchar)[]                          │
├───────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [{'first_name': Zhang, 'last_name': San, 'age': 57}, {'first_name': Li, 'last_name': Si, 'age': 48}, {'firs…  │
└───────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
D SELECT * FROM read_json_auto('test.json', records=true);
┌────────────┬───────────┬─────────┐
│ first_name │ last_name │   age   │
│  varchar   │  varchar  │ varchar │
├────────────┼───────────┼─────────┤
│ Zhang      │ San       │ 57      │
│ Li         │ Si        │ 48      │
│ Wang       │ Wu        │ 23      │
└────────────┴───────────┴─────────┘
D SELECT * FROM read_json_auto('test.json', records=false);
┌────────────────────────────────────────────────────────────┐
│                            json                            │
│ struct(first_name varchar, last_name varchar, age varchar) │
├────────────────────────────────────────────────────────────┤
│ {'first_name': Zhang, 'last_name': San, 'age': 57}         │
│ {'first_name': Li, 'last_name': Si, 'age': 48}             │
│ {'first_name': Wang, 'last_name': Wu, 'age': 23}           │
└────────────────────────────────────────────────────────────┘


# 数据库相关操作命令:
## DML DDL
D CREATE TABLE test_sql (first_name VARCHAR, last_name VARCHAR, age INT);
D INSERT INTO test_sql values('Zhang','San',57);
D INSERT INTO test_sql values('Li','Si',48);
D INSERT INTO test_sql values('Wang','Wu',23);
D  SELECT * FROM test_sql;
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int32 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │


 ##查询当前数据库中的所有**索引**信息；
SELECT * FROM duckdb_indexes(); 
## 查询指定表的所有**索引**信息； 
SELECT * FROM duckdb_indexes()
WHERE table_name = 'fundraising_spawned';  

## 查询指定表的列级注释
select database_name,schema_name ,table_name,column_name,data_type, comment 
      from duckdb_columns()
      WHERE table_name = 'molecule_labs_projects_funding_plan'
      ORDER BY column_index;

## 查询表级注释
SELECT
    schema_name,
    table_name,
    comment
FROM duckdb_tables
WHERE comment IS NOT NULL;



## 显示所有列的信息
 .mode line

## -- 查看所有数据库和schema
SELECT * FROM information_schema.schemata;

## -- 查看当前使用的schema（通常是main）
SELECT current_schema();

## -- 切换schema
SET search_path = 'metadata';
** 这样操作，会让show tables命令下，看到main这个schema和metadata这个schema下的所有表
这么操作后，就可以不带shcema名称的方式，去直接操作metadata这个schema下的表 **

## -- 或者查看当前数据库的所有表
SELECT * FROM information_schema.tables;

## -- 简单查看表列表
SHOW TABLES;





## 附加器
在 C++ 和 Java 中，附加器可以用作批量数据加载的替代方法。该类可用于高效地向数据库系统添加行，而无需使用SQL。


## 客户端API
DuckDB 有多种客户端 API

### python的API测试
- 安装依赖包
[root@localhost ~]# pip3 install duckdb

- 创建连接配置文件
[zhf@localhost ~]$  cat > test_pyapi.py << "EOF"
import duckdb
con = duckdb.connect()
con.sql("SELECT * FROM 'test.csv'").show()
con.close()
EOF
[zhf@localhost ~]$ ll
total 556
-rw-r--r--. 1 zhf zhf  12288 Aug 21 11:43 my1.db
-rw-r--r--. 1 zhf zhf 536576 Aug 21 14:14 my.db
-rw-r--r--. 1 zhf zhf    602 Aug 21 15:15 my.db.wal
-rw-r--r--. 1 zhf zhf     58 Aug 21 11:06 test.csv
-rw-r--r--. 1 zhf zhf    156 Aug 21 15:01 test.json
-rw-r--r--. 1 zhf zhf   2646 Aug 21 13:59 test.parquet
-rw-r--r--. 1 zhf zhf     92 Aug 21 15:22 test_pyapi.py

- 测试连接
[zhf@localhost ~]$  python test_pyapi.py
┌────────────┬───────────┬───────┐
│ first_name │ last_name │  age  │
│  varchar   │  varchar  │ int64 │
├────────────┼───────────┼───────┤
│ Zhang      │ San       │    57 │
│ Li         │ Si        │    48 │
│ Wang       │ Wu        │    23 │
└────────────┴───────────┴───────┘


# 本次开发环境安装配置解释
## 下载
curl -LO https://github.com/duckdb/duckdb/releases/download/v0.9.2/duckdb_cli-linux-amd64.tar.gz
## 创建数据库文件和mytest数据库，如果数据库已经存在，则进入数据库
 ./duckdb data/mytest.db

只读模式：./duckdb -readonly data/mytest.db
**注意：** dcukdb的数据库，不能同时读写，可以同时多个只读模式打开数据库文件
，但无法在有写打开的情况，继续打开读取数据库。

### nodejs的连接测试
- 安装依赖包
[root@localhost ~]# npm install duckdb

- 创建连接配置文件
[zhf@localhost ~]$  cat > test_nodeapi.js << "EOF"
const duckdb = require('duckdb');

async function testConnection() {
    const db = new duckdb.Database();
    try {
        await db.open('test.duckdb'); // 打开或创建数据库文件
        const stmt = db.prepare('SELECT 1');
        const result = await stmt.all();
        console.log(result);
    } catch (error) {
        console.error('连接失败:', error);
    } finally {
        db.close(); // 关闭数据库连接
    }
}
testConnection();
EOF

- 测试连接
注意，这个版本DuckDB v1.3.2的duckdb需要GLIBCXX_3.4.30库文件，才能正常连接数据库
我的解决方式是，再nodejs的nestjs后端开发环境中，安装duckdb v1.0.0的依赖库，
用依赖库中的工具去连接开发环境中的duckdb



# 数仓中各种数据的如何管理的思路
- 问：其实我没有想好前端要展示什么，我对区块链数的特性，还没有太多认知，
我准备先用API把数据拿回来，我仔细研读一下，有了理解了，咱们再继续推进。
目前开发工作先到这里。
我还有1个问题，前端除了展示数仓中的数据，其实我还想拿一些其他区块链媒体网站的信息：
诸如代币价格走势图，x上的舆论，新闻报道，desci团队信息，desci DAO组织的一些动向，
desci平台的一些动向，等等其他的一些数据（我还没想明白），
那么这些数据可能是非结构化的，我怎么处理，数仓还能用吗？
还是要怎么处理，我脑子一片空白。


- 答：
您的思路非常清晰，先获取数据并理解区块链数据特性是非常明智的做法。
关于您提到的非结构化数据处理问题，我来分析一下：

数据类型分析
您需要处理的数据可以分为几类：

结构化数据：

区块链交易数据（已经在DuckDB中）
代币价格数据（可以通过API获取）
市值、交易量等量化指标
半结构化数据：

X上的舆论数据（JSON格式但内容多样）
DeSci团队和DAO组织信息（可能是JSON或XML格式）
非结构化数据：

新闻报道文本
博客文章
社交媒体讨论内容
视频/音频内容（如有）
解决方案
1. 混合存储架构
您可以采用混合存储架构，结合使用：

DuckDB：处理结构化数据和简单的半结构化数据
向量数据库：存储文本的向量表示，用于语义搜索
文件存储：存储原始非结构化数据
2. 数据处理流程
原始数据 → 预处理 → 特征提取 → 存储 → 分析/展示
预处理：清洗、标准化
特征提取：从非结构化数据中提取关键信息
存储：根据数据类型选择合适的存储方式
分析/展示：整合不同来源的数据
3. 具体技术方案
对于代币价格走势图：
使用CoinGecko、CoinMarketCap等API获取结构化数据
存入DuckDB时序表
前端使用ECharts等库展示
对于X上的舆论和新闻：
数据获取：

使用Twitter API或网络爬虫
RSS订阅或新闻API
文本处理：

使用NLP提取关键信息（情感、主题、实体）
将文本转换为向量表示
存储方案：

原始文本：可以存储在JSON字段中（DuckDB支持）
提取的结构化信息：存入DuckDB表
文本向量：使用向量数据库（如Chroma、Milvus）或DuckDB的向量扩展
对于DeSci团队和DAO信息：
创建专门的表结构存储基本信息
使用JSON字段存储复杂或变化的属性
建立实体关系模型
4. 技术栈建议
后端：

DuckDB作为主数据仓库
简单的向量数据库（如ChromaDB）
Node.js处理数据获取和转换
中间件：

简单的ETL流程（可以用Node.js脚本实现）
定时任务调度（如node-cron）
前端：

数据可视化库（ECharts、D3.js）
交互式仪表板框架（如React+MUI）
5. 实施路径
先完成区块链交易数据的仓库建设
逐步添加代币价格等结构化数据
实验性地添加一些非结构化数据处理
根据实际需求和效果调整架构
这种渐进式的方法可以让您在理解数据的同时，逐步构建系统，避免过早的复杂化。

您认为这个方向是否符合您的需求？我们可以先专注于完成区块链数据的获取和存储，然后再逐步扩展到其他数据源。




#############################
powerBI 连接duckdb的token：
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6IjU0MTA5OTA0MEBxcS5jb20iLCJzZXNzaW9uIjoiNTQxMDk5MDQwLnFxLmNvbSIsInBhdCI6Ii1JVDZ5dXdNVGRpUm5kZ1FQcUVGMDV0TFBKZHI4LVVrS3FScTJnY3FXUFUiLCJ1c2VySWQiOiJmYmZiMDBjMy0wM2NmLTQ4YzctYTExMS1kMDVlOThkOTUxMTAiLCJpc3MiOiJtZF9wYXQiLCJyZWFkT25seSI6ZmFsc2UsInRva2VuVHlwZSI6InJlYWRfd3JpdGUiLCJpYXQiOjE3NjEwMjQzOTV9.nMF9xFUx3XsRFv5OQhnR0VN5XLXVGD-4wI4SPtVcAKw
