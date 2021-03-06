# Node context {#concept_ycz_ts4_kfb .concept}

This topic describes the node context functions. The node context is used to transfer the parameter between upstream and downstream nodes. The basic method uses the node context function as the first defined output parameters, and their values on the upstream node. Then the defined input parameter on the downstream node. The value references the output parameters of the upstream node. You can use this parameter in the downstream node to obtain values, which is transferred from the upstream node.

Node context parameter can be configured at **Schedule** \> **Node Context**in a specific node, as shown in the following figure.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22783/155253290913560_en-US.png)

## Output parameters {#section_ssq_fwq_kfb .section}

**The Node Output Parameters** can be defined in **Node Context**. The two types of Output Parameter values are the **Constant** and **Variable**. The **Constant**is a fixed string and the **Variable**are global variables supported by the system. The output parameter can be reused in the downstream node as an input parameter value, after the upstream node is submitted with the output parameter.

**Note:** The assigned value to the defined Output parameter on the current node through internal code writing, for example the PyODPS node is not supported.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22783/155253290913561_en-US.png)

The fields are described as follows.

|**Field**|**Description**|**Note**|
|**No.**|The value of No. is generated by the system and automatically increased.|N/A|
|**Parameter name**|The defined output parameter name.|N/A|
|**Type**|The parameter type.|There are two types of output parameter values, which are the Constant and Variable.|
|**Value**|The source value.| 1.  The string can be entered when the selected Type is Constant.
2.  When the selected type is Variable, the following parameters are supported: System variables, Schedule built-in parameters, Customized parameters $ \{...\}and $ \[…\] .

 |
|**Description**|A brief description of the parameters.|N/A|
|**Action**|**Edit** and **Delete** can be selected|**Edit** and **Delete** are not supported when a downstream node dependency exists. Before adding references to the upstream nodes, please ensure the upstream output is defined correctly.|

## Input parameters {#section_crg_bxq_kfb .section}

**The Node Input Parameters** are used for defining a reference to the output of the upstream node which it is dependent on, and it can be used inside the node similar to that as other parameters.

-   The definition of **The Node Input Parameters**
    1.  Add a dependent upstream node on the **Scheduling Dependencies**.

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22783/155253291013562_en-US.png)

    2.  Add an input parameter definition with value, which references the upstream node, in the **Node Context** \> **The Node Input Parameters.**

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22783/155253291013563_en-US.png)

        The fields are described as follows.

        |**Field**|**Description**|**Note**|
        |**No.**|The value of No. is generated by the system and is automatically increased.|N/A|
        |**Parameter name**|The defined input parameter name.|N/A|
        |**Value of the source**|The parameter value source that is a reference to the upstream node value.|The specific parameter value when the upstream node is running.|
        |**Description**|A brief description of the parameters.|Automatically parsed from the upstream node.|
        |**Parent Node ID**|Parent Node ID|Automatically parsed from the upstream node.|
        |**Action**|**Edit** and **Delete** can be selected|N/A|

-   Use of input parameters

    The reuse format defined input parameter is similar to that as to other systems. The format is `${input parameter name}`. For example, a reference in a shell node is shown in the following figure.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22783/155253291013564_en-US.png)


## Global variables supported by the system {#section_tbm_k1r_kfb .section}

-   System variable

    ```
    
    $ {projectid}: Project ID
    $ {project name}: MaxCompute project name
    $ {nodeid}: Node ID
    $ {gmtdate}: 00:00:00 at the instance date,format: 'yyyy-mm-dd 00:00:00'.
    $ {taskid}: Instance Task ID
    $ {seq}: Task instance sequence number,represents the instance's sequence number in the same node on current day.
    $ {cyctime}: instance time
    $ {status}: Status of instance-Success, Failure
    $ {bizdate}: Business Date
    $ {finishtime}: Instance End Time
    $ {taskType}: Instance Status——NORMAL,MANUAL,PAUSE,SKIP,UNCHOOSE,SKIP_CYCLE
    $ {nodeName}: Node name
    
    ```

-   For more information about additional parameter settings, see [Parameter configuration](reseller.en-US/User Guide/Data development/Scheduling configuration/Parameter configuration.md#).

## Examples {#section_a4x_n5x_5fb .section}

The node test22 is the upstream node of node test223. Please configure the **Node Context** \> **The Node Output Parameters** on node test22. In this example, the parameter name is date1 and the value is $\{yyyymmdd\}, click **Run** as shown in the following figure.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22783/155253291032180_en-US.png)

After node test22 is submitted configure the downstream node test223.

**Note:** Please ensure the**Dependencies** \> **Upstream Node Output Name** in test223 similar to **Dependencies** \> **Output Name** in test22.

Enter the parameter name of test22 date1 in the **Node Context** \> **The Node Input Parameter** \> **Parameter Name,** and there will be options available in the**Value** of the drop-down menu. Choose the specific source and click **Save**.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/22783/155253291032181_en-US.png)

