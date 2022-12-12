# OpenFrp OPENAPI 使用指南

**欢迎使用 OpenFrp开放映射 API，本指南会帮助您使用我们所提供的API及相关内容。**

>## 阅读前请注意
>
>1. **注意：** OpenFrp API 均使用 POST/GET 方式请求，并
>JSON 格式请求。
>
>2. 本文将采用 Postman 工具做请求示例，您可自行前往 Postman 官网下载参照。
>
>3. 除本文所列出的 API 以外，您可自行使用网络抓包工具抓取 OpenFrp 相关的 API 请求，这不会违反用户协议、服务条款等。在传输时，我们采用明文方式传输数据，使用HTTPS保持安全加密。
>
>4. 本文中所显示的大多数密钥、会话ID、用户账户与密码均为虚构，不可用于登录等操作。一般情况下，我们会使用演示账户（test@test.com）来进行操作。本文将尽可能保证获取的量一致。
>
>5. 我们没有义务为您解决您在开发过程中所遇到的错误问题，在使用过程中所遇到的**大多数**问题都可能与我们**无关**！
>

***

## OpenFrp OPENAPI 使用条款

在您使用 OpenFrp OPENAPI 时，您 **必须** 在项目的 **显著位置** 注明使用 OpenFrp OPENAPI ，并必须确保用户在显著位置明确可见！若您违反本约定，**我们保留撤销对您使用 OpenFrp OPENAPI 的一切权利**！在使用 OpenFrp OPENAPI 进行商业活动时，必须获得 OpenFrp Project 项目组的**书面协议**授权！

***

**全局键值说明:**

```json
{
    ...
    "flag": true,
    "msg": "Message"
}
```

 \* OpenFrp API 所有API均包含 ``flag``、``msg`` 两个键值，``flag`` 键值用于确定请求是否成功，返回值为 ``true``，``msg`` 键值用于返回API的消息，例如“登录成功”、“OK”等。需要您注意。

***

**目前, OpenFrp API地址为：**
>**<https://of-dev-api.bfsea.xyz>**

您可通过在地址后添加路径来访问API，下文仅向您提供相关API路径以供您参考。

***

## 1. Login 登录API

>API路径：
>/user/login

* 此 API 可帮助您通过账号密码登录到 OpenFrp，除获取节点信息和获取公告外，均需要登录获取会话ID与API认证信息才可用。

本 API 需要这些内容：用户账户（用户名或邮箱）、用户密码

### 请求示例

请求类型：``POST``

请求地址：``https://of-dev-api.bfsea.xyz/user/login``

请求内容：

```json
{
    "user": "test@test.com",
    "password": "test@test.com"
} 
```

>*``User``项支持用户名或邮箱(不区分大小写)，``Password``项请使用明文传输(区分大小写)*

在请求正常的情况下，您会得到以下返回值：

```json
{
    "data": "0ea7223e83d0482cb76eae26b9f7a602",
    "flag": true,
    "msg": "登录成功！"
}
```

> *``data`` 项为本次登录的会话ID(长期可用，可保存到文件中备用)。  ``flag`` 项为API请求状态，针对所有OpenFrp API均可用，返回值 ``true`` 则请求正常。``msg`` 项为登录后服务器的返回信息，可用于登录确认。*

在收到返回的同时，您需要同时读取返回值 **``Header``** 中的 ``Authorization``内容，并将其保存为变量备用。在文档标注 *``Header``* 的条目需要在请求中的 **``Header``** 附带``Authorization``及其内容。

如您获取到的Header中的``Authorization``为

```text
OPENFRPeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIwZWE3MjIzZTgzZDA0ODJjYjc2ZWFlMjZiOWY3YTYwMiIsImV4cCI6MTY2NzgwMDUzNH0.jx3VZqUYNhtOYCauHHHb0L9gmQTLBelIsz7hN8zPdJPdpB5paPPYTG1tT7PMVWyGcoOezcyNSKWMpOPZChVuTw
```

请将其添加到您请求操作的Header中，如：
``Authorization:OPENFRPeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIwZWE3MjIzZTgzZDA0ODJjYjc2ZWFlMjZiOWY3YTYwMiIsImV4cCI6MTY2NzgwMDUzNH0.jx3VZqUYNhtOYCauHHHb0L9gmQTLBelIsz7hN8zPdJPdpB5paPPYTG1tT7PMVWyGcoOezcyNSKWMpOPZChVuTw``

*此内容可能不时更新，需要您注意。请不要使用文档中的内容。*

***

## 2. 获取用户信息API *``Header``*

>API路径：
>/frp/api/getUserInfo

* 此 API 可帮助您获取用户账户的所有信息，非常有用。

本 API 需要用户已登录，程序已获取用户的会话ID和Authorization验证，并将Authorization写入到header中。

会话ID仍使用 ``0ea7223e83d0482cb76eae26b9f7a602`` 为示例

### 请求示例

请求类型：``POST``

请求地址：``https://of-dev-api.bfsea.xyz/frp/api/getUserInfo``

请求内容：

```json
{
    "session": "0ea7223e83d0482cb76eae26b9f7a602"
}
```

> *``session`` 值请填写登录时获取的会话ID，会话ID有效期为4小时，可能需要每4小时更新一次(重新登录)。*

在请求正常的情况下，您会得到以下返回值：

```json
{
    "data": {
        "outLimit": 1536,
        "used": 0,
        "token": "e900d8f2498202114ec2e9b0597dfb66",
        "realname": false,
        "regTime": 1626512283.000000000,
        "inLimit": 1536,
        "friendlyGroup": "普通用户",
        "proxies": 4,
        "id": 4,
        "email": "test@test.com",
        "username": "test",
        "group": "normal",
        "traffic": 2171
    },
    "flag": true,
    "msg": "ok"
}
```

> *返回值解释：*
> ``data`` 下的键值
> 键名        | 值内容意
> ----------- | ----------------------------  
> outLimit    | 上行带宽（Kbps）
> used        | 已用隧道（条）
> token       | 用户密钥（32位字符）
> realname    | 是否已进行实名认证（已认证为 ``true`` ，未认证为 ``false`` 。）
> regTime     | 注册时间（Unix时间戳）
> inLimit     | 下行带宽（Kbps）
> friendlyGroup | 用户组名称（文字格式友好名称，可直接输出显示。）
> proxies     | 总共隧道条数（条）
> id          | 用户注册ID
> email       | 用户注册邮箱
> username    | 用户名（用户账户）
> group       | 用户组（系统识别标识）（``normal`` 为普通用户）
>traffic      | ~~红绿灯~~ 剩余流量（Mib）

***

## 3. 获取用户隧道列表 *``Header``*

>API路径：/frp/api/getUserProxies

* 此 API 可帮助您获取用户当前账户下的所有隧道信息（隧道列表）。

本 API 需要用户已登录，程序已获取用户的会话ID和Authorization验证，并将Authorization写入到header中。

会话ID仍使用 ``0ea7223e83d0482cb76eae26b9f7a602`` 为示例

### 请求示例

请求类型：``POST``

请求地址：``https://of-dev-api.bfsea.xyz/frp/api/getUserProxies``

请求内容：

```json
{
    "session": "0ea7223e83d0482cb76eae26b9f7a602"
}
```

> *``session`` 值请填写登录时获取的会话ID，会话ID有效期为4小时，可能需要每4小时更新一次(重新登录)。*

在请求正常的情况下，您会得到以下返回值：

```json
{
 "data": {
  "total": 2,
  "list": [{
   "id": 11451,
   "proxyName": "test",
   "proxyType": "tcp",
   "localIp": "127.0.0.1",
   "localPort": 1234,
   "useEncryption": true,
   "useCompression": false,
   "domain": "",
   "locations": "",
   "hostHeaderRewrite": "",
   "remotePort": 11451,
   "sk": "",
   "headerXFromWhere": "",
   "status": true,
   "node": 5,
   "lastUpdate": 1641049420.000000000,
   "lastLogin": 1660474229.000000000,
   "friendlyNode": "杭州多线1",
   "online": false,
   "connectAddress": "cn-hz-bgp-1.openfrp.top:11451"
  }, {
   "id": 19198,
   "proxyName": "test2",
   "proxyType": "tcp",
   "localIp": "127.0.0.1",
   "localPort": 1234,
   "useEncryption": true,
   "useCompression": true,
   "domain": "",
   "locations": "",
   "hostHeaderRewrite": "",
   "remotePort": 11451,
   "sk": "",
   "headerXFromWhere": "",
   "status": true,
   "node": 21,
   "lastUpdate": 1642085623.000000000,
   "lastLogin": 1660474299.000000000,
   "friendlyNode": "北京联通",
   "online": false,
   "connectAddress": "cn-bj-plc-300.openfrp.cc:11451"
  },{
    ...
  }]
 },
 "flag": true,
 "msg": "ok"
}
```

> *返回值解释：*
> ``data`` 数组的每一个组都代表着一条隧道，键值均可使用本表参考。 ``total`` 值为该用户的隧道总数。``list`` 既该用户账户下的隧道列表
> 键名        | 值内容意
> ----------- | ----------------------------  
> id    | 此隧道ID（数字）
> proxyName        | 此隧道名称（文本）
> proxyType       | 此隧道类型（包括: ``tcp udp http https stcp xtcp``）
> localIp    | 此隧道的本地IP地址
> localPort     | 此隧道的本地端口
> useEncryption     | 此隧道是否启用数据加密（``true``/``false``）
> useCompression | 此隧道是否启用数据压缩（``true``/``false``）
> domain     | 此绑定的域名（仅HTTP/S）
> locations          | 此的URL路由
> hostHeaderRewrite       | 此隧道的HOST重写
> remotePort    | 此隧道的远程端口
> sk       | 此隧道的访问密码
>headerXFromWhere      | 此隧道的请求来源
>status  |   此隧道状态(面板状态，禁用状态无法启动隧道)（``true``/``false``）
>node | 此隧道所属的节点号（数字）
>lastUpdate | 此隧道最后一次更新隧道的时间(修改隧道时间)（Unix时间戳）
>lastLogin | 此隧道最后一次在线时间(当前隧道最后一次启动时间)（Unix时间戳）
>friendlyNode | 此隧道所属的节点文本名称（文本）
>online | 此隧道当前在线状态（请求时刷新状态，状态由服务端返回数据）
>connectAddress | 由服务端自动生成的链接此隧道的IP地址（域名方式，若需要数字IP地址可能需要程序手动解析）

***

## 4. 新建隧道 *``Header``*

>API路径：/frp/api/newProxy

* 此 API 可帮助您为用户当前账户新建一条新的隧道。**您需要搭配获取节点列表API使用，因为这样您才可以获取到节点ID**

本 API 需要用户已登录，程序已获取用户的会话ID和Authorization验证，并将Authorization写入到header中。

会话ID仍使用 ``0ea7223e83d0482cb76eae26b9f7a602`` 为示例

### 请求示例

请求类型：``POST``

请求地址：``https://of-dev-api.bfsea.xyz/frp/api/newProxy``

请求内容：

```json
{
    "session": "0ea7223e83d0482cb76eae26b9f7a602",
    "node_id": 44,
    "name": "test",
    "type": "tcp",
    "local_addr": "127.0.0.1",
    "local_port": "25565",
    "remote_port": 27388,
    "domain_bind": "",
    "dataGzip": "False",
    "dataEncrypt": "False",
    "url_route": "",
    "host_rewrite": "",
    "request_from": "",
    "request_pass": ""
}
```

> *``session`` 值请填写登录时获取的会话ID，会话ID有效期为4小时，可能需要每4小时更新一次(重新登录)。*

> *提交值解释：*
> ``data`` 下的键值
> 键名        | 值内容意
> ----------- | ----------------------------  
> session    | 会话ID
> node_id        | 节点ID(纯数字，整数型)
> name       | 隧道名称(不支持中文)
> type    | 隧道类型（包括: ``tcp udp http https stcp xtcp``）
> local_addr     | 本地地址(默认可使用 ``127.0.0.1``)
> local_port     | 本地端口
> remote_port | 远程端口
> domain_bind     | 绑定的域名（非HTTP/S隧道可留空不填，仅HTTP/S隧道有效）
> dataGzip          | 是否启用数据压缩（``true``/``false``
> dataEncrypt       | 是否启用数据加密（``true``/``false``
> url_route         | URL路由
> host_rewrite      | HOST重写
> request_from      | 请求来源
> request_pass      | 访问密码

在请求正常的情况下，您会得到以下返回值：

```json
{
 "data": null,
 "flag": true,
 "msg": "创建成功",
}
```

> *注意：此API返回值的 ``msg`` 项包含多种信息，其包括隧道创建是否成功的信息内容，需要注意。此API的 ``flag`` 项返回为是否创建成功，成功为 ``true``。*

***

## 5. 删除隧道 *``Header``*

>API路径：/frp/api/removeProxy

* 此 API 可帮助您将用户的选定隧道从账户中删除。**建议您搭配获取用户隧道列表API使用，因为这样您才可以获取到隧道ID**

本 API 需要用户已登录，程序已获取用户的会话ID和Authorization验证，并将Authorization写入到header中。

会话ID仍使用 ``0ea7223e83d0482cb76eae26b9f7a602`` 为示例

### 请求示例

请求类型：``POST``

请求地址：``https://of-dev-api.bfsea.xyz/frp/api/removeProxy``

请求内容：

```json
{
    "proxy_id": 11451,
    "session": "0ea7223e83d0482cb76eae26b9f7a602"
}
```

> *``session`` 值请填写登录时获取的会话ID，会话ID有效期为4小时，可能需要每4小时更新一次(重新登录)。*

> *提交值解释：*
> 键名        | 值内容意
> ----------- |----------------------  
> session    | 会话ID
> proxy_id        | 隧道ID(纯数字，整数型)

在请求正常的情况下，您会得到以下返回值：

```json
{
 "data": null,
 "flag": true,
 "msg": "操作成功"
}
```

> *注意：此API返回值的 ``msg`` 项包含多种信息，其包括隧道删除操作是否成功的信息内容，需要注意。此API的 ``flag`` 项返回为是否操作成功，成功为 ``true``。*

## 6. **``GET``** 获取节点列表 *``Header``*

>API路径：/frp/api/getNodeList

* 此 API 可帮助您将用户的选定隧道从账户中删除。**建议您搭配获取用户隧道列表API使用，因为这样您才可以获取到隧道ID**

本 API 需要用户已登录，程序已获取用户的会话ID和Authorization验证，并将Authorization写入到header中。

会话ID仍使用 ``0ea7223e83d0482cb76eae26b9f7a602`` 为示例

### 请求示例

请求类型：``GET``

请求地址：``https://of-dev-api.bfsea.xyz/frp/api/getNodeList``

请求内容：

该请求仅需要您GET时附带Authorization即可。

> *``Authorization`` 值请填写登录时获取的会话Authorization，Authorization有效期不明确，可能需要每4小时获取一次，用于进行API鉴权。*

在请求正常的情况下，您会得到以下返回值：

```json
{
 "data": {
  "total": 27,
  "list": [
   {
    "allowEc": false,
    "bandwidth": 50,
    "bandwidthMagnification": 1,
    "classify": 2,
    "comments": "部分地区移动绕路 自动HTTPS 域名过白",
    "group": "vip;svip;admin;",
    "hostname": "cn-hk-cncn.openfrp.cc",
    "id": 1,
    "maxOnlineMagnification": 1,
    "name": "香港-1 「CN2」",
    "needRealname": true,
    "port": 8120,
    "status": 200,
    "unitcostEc": 1,
    "description": "香港-1 「CN2」部分地区移动绕路 自动HTTPS 域名过白",
    "protocolSupport": {
     "tcp": true,
     "udp": true,
     "xtcp": true,
     "stcp": true,
     "http": true,
     "https": false
    }
   },
   {
    "allowEc": false,
    "bandwidth": 5,
    "bandwidthMagnification": 0.5,
    "classify": 1,
    "comments": "阿里云",
    "group": "normal;vip;svip;admin;",
    "hostname": "cn-bj-bgp-3.openfrp.top",
    "id": 41,
    "maxOnlineMagnification": 3,
    "name": "北京多线-3",
    "needRealname": true,
    "port": 8120,
    "status": 200,
    "unitcostEc": 1,
    "description": "北京多线-3阿里云",
    "protocolSupport": {
     "tcp": true,
     "udp": true,
     "xtcp": true,
     "stcp": true,
     "http": false,
     "https": false
    }
   },
   ...
  ]
 },
 "flag": true,
 "msg": "ok"
}
```

**因实际节点列表JSON信息过长，本文仅截取前两个节点作演示**

> *返回值解释：*
> ``data`` 数组的每一个组都代表一个节点的信息，键值均可使用本表参考。 ``total`` 值为OpenFrp当前节点总数。``list`` 既所有节点的详细信息
> 键名        | 值内容意
> ----------- | ----------------------------  
> allowEc    | 是否允许弹性隧道* （尚未启用）
> bandwidth        | 节点的带宽数
> bandwidthMagnification       | 节点设计的带宽倍率（换算公式：用户速率*带宽倍率 = 实际获得速率）
> classify    | 节点所属区域（``1`` 中国大陆、``2`` 港澳台地区、``3``海外地区 ）
> comments     | 节点的标签（如：推荐建站 等）
> group     | 当前节点所允许使用的用户组（normal即普通用户若用户的用户组不存在于此内容内，创建隧道会被服务器拒绝）
> hostname | 节点所属主机名（节点域名）
> id     | 节点ID
> maxOnlineMagnification          | 在线倍率
> name       | 此隧道的HOST重写
> needRealname    | 节点是否需要用户实名认证
> port       | 节点运行端口
> status      | 节点状态（可参考[HTTP状态码](https://www.runoob.com/http/http-status-codes.html)）
> unitcostEc  |   隧道单价
> description | 节点介绍
> protocolSupport | 节点所支持的隧道类型(包括 ``tcp udp http https stcp xtcp`` 使用 ``true`` / ``false``作为区分是否准许，此处不再做详细解释)

***

## 7. 签到 *``Header``*

>API路径：/frp/api/userSign

* 此 API 可帮助用户调用签到API签到获取流量。**注意：您不应该设计任何有关自动签到的相关功能，因为这是违反服务条款的行为，自动签到为滥用行为行为之一。**
* **建议您在签到执行并获得回调后搭配获取用户信息API使用，以探测流量更新。**

本 API 需要用户已登录，程序已获取用户的会话ID和Authorization验证，并将Authorization写入到header中。

会话ID仍使用 ``0ea7223e83d0482cb76eae26b9f7a602`` 为示例

### 请求示例

请求类型：``POST``

请求地址：``https://of-dev-api.bfsea.xyz/frp/api/userSign``

请求内容：

```json
{
    "session": "0ea7223e83d0482cb76eae26b9f7a602"
}
```

> *``session`` 值请填写登录时获取的会话ID，会话ID有效期为4小时，可能需要每4小时更新一次(重新登录)。*

> *提交值解释：*
> 键名        | 值内容意
> ----------- |----------------------  
> session    | 会话ID

在请求正常的情况下，您会得到以下返回值：

```json
{
    "data":"签到成功，获得 4430 Mib + 额外 10 Mib 流量",
    "flag":true,
    "msg":"OK"
}
```

> *注意：此API返回值的 ``data`` 项包含多种信息，其包括签到是否成功的信息内容，需要注意。若用户已经签到，此API的 ``data`` 项返回为 ``"你今天已经签到过啦"`` 。*
