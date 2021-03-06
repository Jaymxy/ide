# 配置HybridDB for MySQL Reader {#concept_bd5_szb_5fb .concept}

HybridDB for MySQL Reader插件支持读取表和视图。表字段可以依序指定全部列、部分列、调整列顺序、指定常量字段和配置HybridDB for MySQL的函数如now\(\)等。

HybridDB for MySQL Reader插件通过JDBC连接器连接到远程的HybridDB for MySQL数据库，根据您配置的信息生成查询SQL语句发送到远程HybridDB for MySQL数据库，执行该SQL语句并返回结果。之后，Reader插件使用数据同步自定义的数据类型拼装结果为抽象的数据集，并传递给下游Writer处理。同时，Reader插件执行相应的SQL语句，将数据从HybridDB for MySQL库中Select出来，从而实现从HybridDB for MySQL数据库读取数据。

## 类型转换列表 {#section_s4y_fls_5fb .section}

HybridDB for MySQL Reader针对HybridDB for MySQL类型的转换列表，如下所示。

|类型分类|HybridDB for MySQL数据类型|
|:---|:---------------------|
|整数类|Int、Tinyint、Smallint、Mediumint和Bigint|
|浮点类|Float、Double和Decimal|
|字符串类|Varchar、Char、Tinytext、Text、Mediumtext和LongText|
|日期时间类|Date、Datetime、Timestamp、Time和Year|
|布尔型|Bit和Bool|
|二进制类|Tinyblob、Mediumblob、Blob、LongBlob和Varbinary|

**说明：** 

-   除上述罗列字段类型外，其他类型均不支持。
-   HybridDB for MySQL Reader插件将tinyint\(1\)视作整型。

## 参数说明 {#section_v33_lls_5fb .section}

|参数|描述|必选|默认值|
|:-|:-|:-|---|
|datasource|数据源名称，脚本模式支持添加数据源，此配置项填写的内容必须要与添加的数据源名称保持一致。|是|无|
|table|选取的需要同步的表名称，一个数据集成Job只能同步一张表。|是|无|
|column|所配置的表中需要同步的列名集合，使用JSON的数组描述字段信息 。默认使用所有列配置，例如\[ \* \]。-   支持列裁剪，即列可以挑选部分列进行导出。
-   支持列换序，即列可以不按照表schema信息顺序进行导出。
-   支持常量配置，您需要按照SQL语法格式，例如

    ```
[“id”,“table”,“1”,“‘mingya.wmy’”,“‘null’”,“to_char(a+1)”,“2.3”,“true”]
    ```

    -   id为普通列名
    -   table为包含保留字的列名
    -   1为整形数字常量
    -   ‘mingya.wmy’为字符串常量（注意需要加上一对单引号）
    -   null为空指针
    -   CHAR\_LENGTH\(s\)为计算字符串长度函数
    -   2.3为浮点数
    -   true为布尔值
-   column必须显示指定同步的列集合，不允许为空。

|是|无|
|splitPk|HybridDB for MySQL Reader进行数据抽取时，如果指定splitPk，表示您希望使用splitPk代表的字段进行数据分片，数据同步因此会启动并发任务进行数据同步，这样可以提高数据同步的效能。-   推荐splitPk用户使用表主键，因为表主键通常情况下比较均匀，因此切分出来的分片也不容易出现数据热点。
-   目前splitPk仅支持整型数据切分，不支持字符串、浮点、日期等其他类型 。如果您指定其他非支持类型，忽略plitPk功能，使用单通道进行同步。
-   如果splitPk不填写，包括不提供splitPk或者splitPk值为空，数据同步视作使用单通道同步该表数据 。

|否|无|
|where|筛选条件，在实际业务场景中，往往会选择当天的数据进行同步，将where条件指定为gmt\_create\>$bizdate。-   where条件可以有效地进行业务增量同步。如果不填写where语句，包括不提供where的key或value，数据同步均视作同步全量数据。
-   不可以将where条件指定为limit 10，这不符合SQL WHERE子句约束。

|否|无|
|querySql（高级模式，向导模式不提供）|在部分业务场景中，where配置项不足以描述所筛选的条件，您可以通过该配置型来自定义筛选SQL 。当配置此项后，数据同步系统就会忽略tables、columns和splitPk配置项，直接使用这项配置的内容对数据进行筛选，例如需要进行多表join后同步数据，使用`[“id”,“table”,“1”,“‘mingya.wmy’”,“‘null’”,“to_char(a+1)”,“2.3”,“true”]`。当您配置querySql时，HybridDB for MySQL Reader直接忽略table、column、where和splitPk条件的配置，querySql优先级大于table、column、where、splitPk选项。datasource会使用它解析出用户名和密码等信息。|否|无|
|singleOrMulti（只适合分库分表）|表示分库分表，向导模式转换成脚本模式主动生成此配置`“singleOrMulti”:“multi”`，但配置脚本任务模板不会直接生成此配置，您需要手动添加，否则只会识别第一个数据源。singleOrMulti只是前端在用，后端没有用这个做分库分表判断。|是|multi|

## 向导模式 {#section_trc_5ls_5fb .section}

1.  选择数据源配置同步任务的**数据来源**和**数据去向**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62192/155176743332098_zh-CN.png)

    |配置|说明|
    |:-|:-|
    |**数据源**|即上述参数说明中的datasource，一般填写您配置的数据源名称。|
    |**表**|即上述参数说明中的table。|
    |**数据过滤**|您将要同步数据的筛选条件，暂时不支持limit关键字过滤。SQL语法与选择的数据源一致。|
    |**切分键**|您可以将源数据表中某一列作为切分键，建议使用主键或有索引的列作为切分键，仅支持类型为整型的字段。读取数据时，根据配置的字段进行数据分片，实现并发读取，可提升数据同步效率。**说明：** 切分键和数据同步中的选择来源有关，配置数据来源时才显示切分键配置项。

 |

2.  **字段映射**，即上述参数说明中的column。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62192/155176743332100_zh-CN.png)

    左侧的源头表字段和右侧的目标表字段为一一对应关系，单击**添加一行**可增加单个字段，鼠标放至需要删除的字段上，即可单击**删除**图标进行删除。

    |配置|说明|
    |:-|:-|
    |**同名映射**|单击**同名映射**，即可根据名称建立相应的同行映射关系，请注意匹配数据类型。|
    |**同行映射**|单击**同行映射**，即可在同行建立相应的映射关系，请注意匹配数据类型。|
    |**取消映射**|单击**取消映射**，即可取消建立的映射关系。|
    |**自动排版**|单击**自动排版**，即可根据相应的规律自动排版。|
    |**手动编辑源表字段**![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62192/155176743332103_zh-CN.png)|请手动编辑字段，一行表示一个字段，首尾空行会被采用，其他空行会被忽略。|
    |**添加一行**|添加一行的功能如下所示：    -   可以输入常量，输入的值需要使用英文单引号，如’abc’、’123’等。
    -   可以配合调度参数使用，如$\{bizdate\}等。
    -   可以输入关系数据库支持的函数，如now\(\)、count\(1\)等。
    -   如果您输入的值无法解析，则类型显示为未识别。
|

3.  **通道控制**

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62192/155176743332104_zh-CN.png)

    |配置|说明|
    |:-|:-|
    |DMU|数据集成消耗资源（包括CPU、内存、网络等资源分配）的度量单位。一个DMU描述了一个数据集成作业最小运行能力，即在限定的CPU、内存、网络等资源情况下对于数据同步的处理能力。|
    |作业并发数|可将此属性视为数据同步任务内，可从源并行读取或并行写入数据存储端的最大线程数。向导模式通过界面化配置并发数，指定任务所使用的并行度。|
    |错误记录数|错误记录数，表示脏数据的最大容忍条数。|
    |任务资源组|任务运行的机器，如果任务数比较多，使用默认资源组出现等待资源的情况，建议添加自定义资源组（目前只有华东1，华东2支持添加自定义资源组），详情请参见[新增任务资源](intl.zh-CN/使用指南/数据集成/常见配置/新增任务资源.md#)|


## 脚本开发介绍 {#section_pvs_k4s_5fb .section}

单库单表的脚本样例如下，详情请参见上述参数说明。

```
{
    "type": "job",
    "steps": [
        {
            "parameter": {
                "datasource": "px_aliyun_hymysql",//数据源名
                "column": [//源端列名
                    "id",
                    "name",
                    "sex",
                    "salary",
                    "age",
                    "pt"
                ],
                "where": "id=10001",//过滤条件
                "splitPk": "id",//切分键
                "table": "person"//源端表名
            },
            "name": "Reader",
            "category": "reader"
        },
        {
            "parameter": {}
    ],
    "version": "2.0",//版本号
    "order": {
        "hops": [
            {
                "from": "Reader",
                "to": "Writer"
            }
        ]
    },
    "setting": {
        "errorLimit": {//错误记录数
            "record": ""
        },
        "speed": {
            "concurrent": 7,//并发数
            "throttle": true,//同步速度限流
            "mbps": 1,//限流值
            "dmu": 5//dmu值
        }
    }
}
```

