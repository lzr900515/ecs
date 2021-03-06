# 通过API允许不同账号下的ECS实例内网通信 {#concept_40597_zh .task}

若您需要实现同一地域下不同账号的ECS实例内网通信，可以参考本文描述授权安全组间互访。

本文调用API的工具为阿里云CLI，请确保您已安装并配置了阿里云CLI。具体操作，请参见[安装CLI](https://www.alibabacloud.com/help/doc-detail/90765.htm)和[配置CLI](https://www.alibabacloud.com/help/doc-detail/90766.htm)。

目前授权安全组内网通信有以下两种，请根据您的实际需求选择方式。

-   [ECS实例间通信](#section_bpp_qvf_ip5)：授权同一账号两台ECS实例间的内网通信。
-   [账号间内网通信](#section_003_ysx_s54)：授权同一账号同一地域下两个安全组内所有的ECS实例的内网通信，包括授权以后购买的同一安全组内的ECS实例。

    **说明：** 账号间内网通信实际上是安全组间授权，即授权处于这两个安全组内的ECS实例后就可以实现内网通信。修改安全组配置会影响到安全组内所有的ECS实例，请根据实际需要进行操作，避免影响到ECS实例网络下运行的业务。


安全组是ECS实例的虚拟防火墙，安全组本身不提供通信能力和组网能力。授权不同安全组内的实例内网通信后，请同时确保实例可以建立内网互通的能力。

-   若实例均是经典网络类型，必须位于同一地域下。
-   若实例均是VPC类型，不同VPC间默认内网不通。建议通过公网访问的方式通信，或者通过高速通道、VPN网关和云企业网等方式提供访问能力。详情请参见[高速通道](../../../../../intl.zh-CN/产品简介/什么是高速通道？.md#)、[VPN网关](../../../../../intl.zh-CN/产品简介/什么是VPN网关.md#)和[云企业网](../../../../../intl.zh-CN/产品简介/什么是云企业网.md#)。
-   若实例网络类型不同，请设置ClassicLink允许实例通信。具体操作，请参见[经典网络和专有网络互通](../intl.zh-CN/网络/经典网络和专有网络互通.md#)。
-   若实例位于不同地域，建议通过公网访问的方式通信，或者通过高速通道、VPN网关和云企业网等方式提供访问能力。详情请参见[高速通道](../../../../../intl.zh-CN/产品简介/什么是高速通道？.md#)、[VPN网关](../../../../../intl.zh-CN/产品简介/什么是VPN网关.md#)和[云企业网](../../../../../intl.zh-CN/产品简介/什么是云企业网.md#)。

## ECS实例间通信 {#section_bpp_qvf_ip5 .section}

1.  查询两台ECS实例的内网IP地址和两台ECS实例所处的安全组ID。 您可以通过控制台或调用DescribeInstances接口获得ECS实例所属的安全组ID。假设两台ECS实例的信息如下表所示。

    |实例|IP地址|所属安全组|安全组ID|
    |--|----|-----|-----|
    |实例A|10.0.0.1|sg1|sg-bp1azkttqpldxgtedXXX|
    |实例B|10.0.0.2|sg2|sg-bp15ed6xe1yxeycg7XXX|

2.  在sg1安全组中添加放行10.0.0.2的入方向的规则。 

    ``` {#codeblock_0hm_nmt_tq8}
    aliyun ecs AuthorizeSecurityGroup --SecurityGroupId sg-bp1azkttqpldxgtedXXX --RegionId cn-qingdao --IpProtocol all  --PortRange=-1/-1. --SourceCidrIp 10.0.0.2 --NicType intranet
    ```

3.  在sg2安全组中添加放行10.0.0.1的入方向的规则。 

    ``` {#codeblock_204_bx1_0p7}
    aliyun ecs AuthorizeSecurityGroup --SecurityGroupId sg-bp15ed6xe1yxeycg7XXX --RegionId cn-qingdao --IpProtocol all  --PortRange=-1/-1. --SourceCidrIp 10.0.0.1 --NicType intranet
    ```

    **说明：** 

    -   以上命令中，地域取值为华北 1（青岛）cn-qingdao，请您根据实际情况修改。
    -   以上命令中，调用AuthorizeSecurityGroup接口添加安全组入方向的放行规则，主要关注的参数为SecurityGroupId和SourceCidrIp。
4.  等待一分钟后，使用ping命令测试两台ECS实例之间是否内网互通。

## 账号间内网通信 {#section_003_ysx_s54 .section}

1.  查询两个账号的账号名和两个账号下对应的安全组ID。 您可以通过控制台或调用DescribeInstances接口获得ECS实例所属的安全组ID。假设两个账号的信息如下表所示。

    |账号|账号ID|安全组|安全组ID|
    |--|----|---|-----|
    |账号A|a@aliyun.com|sg1|sg-bp1azkttqpldxgtedXXX|
    |账号B|b@aliyun.com|sg2|sg-bp15ed6xe1yxeycg7XXX|

2.  在sg1安全组中添加放行sg2安全组入方向的规则。 

    ``` {#codeblock_3w2_6tw_zbu}
    aliyun ecs AuthorizeSecurityGroup --SecurityGroupId sg-bp1azkttqpldxgtedXXX --RegionId cn-qingdao --IpProtocol all  --PortRange=-1/-1. --SourceGroupId sg-bp15ed6xe1yxeycg7XXX --SourceGroupOwnerAccount b@aliyun.com --NicType intranet
    ```

3.  在sg2安全组中添加放行sg1安全组入方向的规则。 

    ``` {#codeblock_hf5_rca_2hy}
    aliyun ecs AuthorizeSecurityGroup --SecurityGroupId sg-bp15ed6xe1yxeycg7XXX --RegionId cn-qingdao --IpProtocol all  --PortRange=-1/-1. --SourceGroupId sg-bp1azkttqpldxgtedXXX --SourceGroupOwnerAccount a@aliyun.com --NicType intranet
    ```

    **说明：** 

    -   以上命令中，地域取值为华北 1（青岛）cn-qingdao，请您根据实际情况修改。
    -   以上命令中，调用AuthorizeSecurityGroup接口添加安全组入方向的放行规则时，主要关注的参数为SecurityGroupId、SourceGroupId和SourceGroupOwnerAccount。
4.  等待一分钟后，使用ping命令测试查看两台ECS实例之间是否内网互通。

