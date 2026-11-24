# OpenFrp OPENAPI 使用指南
欢迎使用 OpenFrp开放映射 API，本指南会帮助您使用我们所提供的API及相关内容。

阅读前请注意
注意： OpenFrp OPENAPI 均以POST/GET方式请求，请求格式为JSON。

本文将采用 Postman 工具做请求示例，您可自行前往 Postman 官网下载参照。

除本文所列出的 API 以外，您可自行使用网络抓包工具抓取 OpenFrp 相关的 API 请求，这不会违反用户协议、服务条款等。在传输时，我们采用明文方式传输数据，使用HTTPS保持安全加密。

本文中所显示的大多数密钥、会话ID、用户账户与密码均为虚构，不可用于登录等操作。一般情况下，我们会使用演示账户（test@test.com）来进行操作。本文将尽可能保证获取的量一致。

我们没有义务为您解决您在开发过程中所遇到的错误问题，在使用过程中所遇到的大多数问题都可能与我们无关！

本文列出的 API 某些地方可能过时或者有误了，所以欢迎您在使用的同时，帮助我们更新修订本文档。（您可通过提交pr或issue等方式来帮助我们）:-)

# OpenFrp OPENAPI 使用条款
在您使用 OpenFrp OPENAPI 时，您 必须 在项目的 明显可见位置 注明使用 OpenFrp OPENAPI ，若要出于美观考虑可尝试降低饱和色，不应恶意隐藏相关内容！若您违反本约定，我们保留撤销对您使用 OpenFrp OPENAPI 的一切权利！在使用 OpenFrp OPENAPI 进行商业活动时，必须获得 OpenFrp Project 项目组的书面协议授权！

需要在项目有关位置中添加 “此项目由社区开发，OpenFrp 官方不负责除节点问题以外的技术支持” 并尽可能附加技术支持方式。

您必须在请求 OpenFrp OPENAPI 或是有关资源站点时携带应用程序 UA （User Agent）,否则 可能被视为无效请求而被防火墙拦截。您亦可加入第三方开发者交流群（421531763）向我们提交您的应用程序UA。

# OpenFRP 控制台 API 文档

> **Base URL**: `http://api.openfrp.net`
>
> 本文档共包含 14 个业务接口，一般可以满足你的需求，另附登录流程与认证方式。

---

## 目录

- [登录](#登录)
  - [方式一：OAuth 授权码登录](#方式一oauth-授权码登录)
  - [方式二：Access 密钥协商登录](#方式二access-密钥协商登录)
- [认证方式](#认证方式)
  - [未登录与凭证失效](#未登录与凭证失效)
  - [Authorization 的自动续期](#authorization-的自动续期)
- [通用响应结构](#通用响应结构)
- [接口列表](#接口列表)
  - [1. 获取签到信息](#1-获取签到信息)
  - [2. 通用查询](#2-通用查询)
  - [3. 获取用户隧道列表](#3-获取用户隧道列表)
  - [4. 获取节点列表](#4-获取节点列表)
  - [5. 获取节点状态](#5-获取节点状态)
  - [6. 获取所有用户组](#6-获取所有用户组)
  - [7. 获取流量历史](#7-获取流量历史)
  - [8. 获取用户信息](#8-获取用户信息)
  - [9. 启用/禁用隧道](#9-启用禁用隧道)
  - [10. 编辑隧道](#10-编辑隧道)
  - [11. 创建隧道](#11-创建隧道)
  - [12. 删除隧道](#12-删除隧道)
  - [13. 强制下线隧道](#13-强制下线隧道)
  - [14. 刷新隧道状态](#14-刷新隧道状态)

---

## 登录

在调用需要身份校验的接口前，必须先完成登录，取得会话 ID 与 `Authorization`。以下两种方式任选其一，登录后获得的 `Authorization` 与 会话 ID 均约 8 小时有效。

### 方式一：OAuth 授权码登录

整体流程：引导用户授权 → 从回调地址取得 `code` → 用 `code` 换取会话 ID 与 `Authorization`。

**1：使用固定模板，引导用户打开浏览器进行授权**

| 项目 | 值 |
|------|-----|
| 授权地址 | `https://account.naids.com/oauth2/authorize?response_type=code&redirect_uri=<url>&client_id=openfrp` |
| 回调地址 <url> | `https://api.openfrp.net/oauth_callback` |

实际生产中，`<url>` 为你的本地服务器地址(在用户电脑上创建临时 HTTP 服务)，用于接收 `oauth_callback` 返回的 `code`。回调也可能返回 `?error=access_defined`，一般是第一次授权时用户点击了取消。

**2：通过传输 Code 登录**

**`POST /oauth2/callback?code=<code>`**

**Query 参数：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `code` | string | 是 | 回调 OAuth 接口得到的 code |

**请求体**：无。

**响应示例**（`200`）：

```json
{
  "data": "0ea7223e83d0482cb76eae26b9f7a602",
  "flag": true,
  "msg": "登录成功！"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data` | string | 本次登录的会话 ID（8 小时有效期） |
| `flag` | boolean | API 请求状态，`true` 则请求正常 |
| `msg` | string | 登录后服务器的返回信息，可用于登录确认 |

在收到返回的同时，需读取响应 Header 中的 `Authorization` 内容并保存为变量备用。文档中标注 Header 的条目需要在请求 Header 中附带 `Authorization` 及其内容：

```http
Authorization: OPENFRPeyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9...
```

> 每次请求获得的 `Authorization` 与 会话 ID 均有 8 小时有效期，且可能不时更新，请勿直接使用文档中的示例值。

### 方式二：Access 密钥协商登录

适用于已经登录面板的情况，可程序化完成登录的场景，通过 Curve25519 密钥协商认证身份。

整体流程：请求登录取得 `request_uuid` 与授权地址 → 轮询拿到加密的 `authorization_data` → 用本地私钥与服务器公钥解密，得到 `Authorization`。

**1. 请求登录**

**`POST https://access.openfrp.net/argoAccess/requestLogin`**

**请求体**（`application/json`）：

```json
{ "public_key": "base64_urlsafe_public_key" }
```

先生成 Curve25519 密钥对，将 `publicKey` 做 base64 urlsafe 后提交（示例使用 NaCl 库，库不唯一）：

```csharp
var (privateKey, publicKey) = NaCl.Curve25519XSalsa20Poly1305.KeyPair();
```

**响应示例**（`200`）：

```json
{
  "code": 200,
  "msg": "query ok",
  "data": {
    "authorization_url": "https://console.openfrp.net/usercenter?argoaccess=9a23ed00-07e6-4f11-bcfd-b0216e3bcacf",
    "request_uuid": "9a23ed00-07e6-4f11-bcfd-b0216e3bcacf"
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.authorization_url` | string | 引导用户访问的授权地址 |
| `data.request_uuid` | string | 本次登录请求的 UUID，全局有效期 5 分钟，过期后任何操作均无效 |

**2. 轮询登录结果**

**`GET https://access.openfrp.net/argoAccess/pollLogin?request_uuid=<request_uuid>`**

**Query 参数：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `request_uuid` | string | 是 | 上一步返回的 `request_uuid` |

本接口需以轮询方式请求：推荐速率 1 次/5 秒，请求最大时长 5 分钟，超出 5 分钟后的请求将无效。

**响应示例**（`200`）：

```json
{
  "code": 200,
  "msg": "query ok",
  "data": {
    "authorization_data": "CqWR/qmS9j2VHXpKVjU4Tee/Rguubdi+cY81KKALgf/WPT+LIM6txOXsg20QAYgKdSmxveDF1ZeRwCR7HS4idKxI4kj2bDyWAl0hxWRUDph30AMPLA+T3z/aQOdDrZ5+",
    "request_uuid": "9a23ed00-07e6-4f11-bcfd-b0216e3bcacf"
  }
}
```

响应 Header 中会返回服务器公钥 `x-request-public-key`，用于解密 `authorization_data`：

```http
x-request-public-key: I9E_WWIGsvz1c_FDo_awTFIMAFR7r_tvKAe90iVKJGk=
```

使用请求登录时生成的 `privateKey` 与服务器返回的公钥，通过 Curve25519XSalsa20Poly1305 解密 `authorization_data`。解密完成后即可获得 `Authorization` 的明文值，请将其保存备用。

---

## 认证方式

所有接口为以下 API Key 认证方式：

| 认证方案 | 类型 | 位置 | 参数名 |
|---------|------|------|--------|
| `Session` | apiKey | Header | `Authorization` |

**示例：**

```http
POST /frp/api/getUserInfo HTTP/1.1
Host: api.openfrp.net
Authorization: <your-token>
Content-Type: application/json
```

### 未登录与凭证失效

除授权登录相关接口外，所有接口在未登录或 `Authorization` 无效（如已过期）时，统一返回：

```json
{
  "data": null,
  "flag": false,
  "msg": "登入凭证无效，请先登入",
  "detail": null
}
```

此时需重新执行[登录](#登录)流程，取得新的会话 ID 与 `Authorization`。

### Authorization 的自动续期

需要授权的接口有概率在响应的 Header 中返回新的 `Authorization`，此时旧凭证可能已接近过期。建议每次请求后检查响应 Header，若存在 `Authorization` 则覆盖本地保存的凭证，以避免后续请求失败。

---

## 通用响应结构

除个别接口外，绝大多数接口返回统一的外层结构：

```json
{
  "data": { ... },
  "flag": true,
  "msg": "",
  "detail": null
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data` | object / array / null | 业务数据，具体结构因接口而异 |
| `flag` | boolean | 请求是否成功 |
| `msg` | string | 提示信息 |
| `detail` | null | 详情（通常为 null） |

---

## 接口列表

### 1. 获取签到信息

**`POST /frp/api/getSignInfo`**

获取当前用户的签到状态与流量统计。

**请求体**（`application/json`）：

```json
{}
```

空对象即可。

**响应示例**（`200`）：

```json
{
  "data": {
    "sign_max": "string",
    "sign_min": "string",
    "signdate": 0,
    "todaysign": 0,
    "totalsign": "string",
    "totaltraffic": "string"
  },
  "flag": true,
  "msg": "string",
  "detail": null
}
```

**响应字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.sign_max` | string | 签到上限 |
| `data.sign_min` | string | 签到下限 |
| `data.signdate` | integer | 签到日期 |
| `data.todaysign` | integer | 今日签到状态 |
| `data.totalsign` | string | 累计签到 |
| `data.totaltraffic` | string | 累计流量 |

---

### 2. 通用查询

**`GET /web/commonQuery/get`**

用于获取公告、弹窗、版本信息、软件下载列表等通用配置。

**Query 参数：**

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `key` | string | 否 | 查询键 |

**响应示例**（`200`）：

`data` 字段为 `string` 或对象，对象结构如下：

```json
{
  "data": {
    "alerts": [
      { "content": "string", "title": "string", "type": "string" }
    ],
    "cpl_announce": "string",
    "dialogs": { "content": "string" },
    "headalert": {
      "content": "string",
      "status": true,
      "title": "string",
      "type": "string"
    },
    "mdinfo": "string",
    "common_details": "string",
    "latest": "string",
    "latest_full": "string",
    "latest_msg": "string",
    "latest_ver": "string",
    "launcher_inf": {
      "latest": "string",
      "msg": "string",
      "title": "string"
    },
    "soft": [
      {
        "arch": [
          { "details": "string", "file": "string", "label": "string", "key": "string" }
        ],
        "label": "string",
        "os": "string"
      }
    ],
    "source": [
      { "label": "string", "value": "string" }
    ]
  },
  "flag": true,
  "msg": "string",
  "detail": null
}
```

**对象字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.alerts` | array | 告警列表 |
| `data.alerts[].content` | string | 告警内容 |
| `data.alerts[].title` | string | 告警标题 |
| `data.alerts[].type` | string | 告警类型 |
| `data.cpl_announce` | string | 公告内容 |
| `data.dialogs.content` | string | 弹窗内容 |
| `data.headalert` | object | 顶部告警信息 |
| `data.headalert.status` | boolean | 告警状态 |
| `data.mdinfo` | string | Markdown 信息 |
| `data.common_details` | string | 通用详情 |
| `data.latest` | string | 最新版本 |
| `data.latest_full` | string | 最新完整版本 |
| `data.latest_msg` | string | 最新消息 |
| `data.latest_ver` | string | 最新版本号 |
| `data.launcher_inf` | object | 启动器信息 |
| `data.soft` | array | 软件下载列表 |
| `data.soft[].arch` | array | 架构列表 |
| `data.soft[].arch[].details` | string | 详情 |
| `data.soft[].arch[].file` | string | 文件名 |
| `data.soft[].arch[].label` | string | 标签 |
| `data.soft[].arch[].key` | string | 键 |
| `data.soft[].label` | string | 软件标签 |
| `data.soft[].os` | string | 操作系统 |
| `data.source` | array | 来源列表 |

---

### 3. 获取用户隧道列表

**`POST /frp/api/getUserProxies`**

获取当前用户创建的所有 FRP 隧道。

**请求体**（`application/json`）：

```json
{}
```

**响应示例**（`200`）：

```json
{
  "data": {
    "list": [
      {
        "autoTls": "string",
        "connectAddress": "string",
        "custom": "string",
        "domain": "string",
        "extAddress": ["string"],
        "forceHttps": true,
        "friendlyNode": "string",
        "id": 0,
        "lastLogin": null,
        "lastUpdate": 0,
        "localIp": "string",
        "localPort": 0,
        "nid": 0,
        "nodeHostname": "string",
        "online": true,
        "proxyName": "string",
        "proxyProtocolVersion": true,
        "proxyType": "string",
        "remotePort": 0,
        "status": true,
        "uid": 0,
        "useCompression": true,
        "useEncryption": true
      }
    ],
    "total": 0
  },
  "flag": true,
  "msg": "string",
  "detail": null
}
```

**隧道对象字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `autoTls` | string | 自动 TLS 配置 |
| `connectAddress` | string | 连接地址 |
| `custom` | string | 自定义配置 |
| `domain` | string | 域名 |
| `extAddress` | array\<string\> | 扩展地址列表 |
| `forceHttps` | boolean | 是否强制 HTTPS |
| `friendlyNode` | string | 节点友好名称 |
| `id` | integer | 隧道 ID |
| `lastLogin` | null \| integer | 最后登录时间 |
| `lastUpdate` | integer | 最后更新时间 |
| `localIp` | string | 本地 IP |
| `localPort` | integer | 本地端口 |
| `nid` | integer | 节点 ID |
| `nodeHostname` | string | 节点主机名 |
| `online` | boolean | 是否在线 |
| `proxyName` | string | 隧道名称 |
| `proxyProtocolVersion` | boolean | 代理协议版本 |
| `proxyType` | string | 隧道类型 |
| `remotePort` | integer | 远程端口 |
| `status` | boolean | 状态 |
| `uid` | integer | 用户 ID |
| `useCompression` | boolean | 是否启用压缩 |
| `useEncryption` | boolean | 是否启用加密 |

---

### 4. 获取节点列表

**`POST /frp/api/getNodeList`**

获取所有可用 FRP 节点信息。

**请求体**（`application/json`）：

```json
{}
```

**响应示例**（`200`）：

```json
{
  "data": {
    "list": [
      {
        "allowEc": true,
        "allowPort": "string",
        "bandwidth": 0,
        "bandwidthMagnification": 0.0,
        "classify": 0,
        "comments": "string",
        "description": "string",
        "enableDefaultTLS": true,
        "fullyLoaded": true,
        "group": "string",
        "hostname": "string",
        "hostnameExt": "string",
        "id": 0,
        "maxOnlineMagnification": 0.0,
        "name": "string",
        "needRealname": true,
        "port": 0,
        "protocolSupport": {
          "http": true,
          "https": true,
          "tcp": true,
          "udp": true
        },
        "status": 0,
        "tlsSNIOverwrite": null,
        "unitcostEc": 0
      }
    ],
    "total": 0
  },
  "flag": true,
  "msg": "string",
  "detail": null
}
```

**节点对象字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `allowEc` | boolean | 是否允许 EC |
| `allowPort` | string | 允许的端口 |
| `bandwidth` | integer | 带宽 |
| `bandwidthMagnification` | number | 带宽倍率 |
| `classify` | integer | 分类 |
| `comments` | string | 备注 |
| `description` | string | 描述 |
| `enableDefaultTLS` | boolean | 是否启用默认 TLS |
| `fullyLoaded` | boolean | 是否已满负载 |
| `group` | string | 所属组 |
| `hostname` | string | 主机名 |
| `hostnameExt` | string | 扩展主机名 |
| `id` | integer | 节点 ID |
| `maxOnlineMagnification` | number | 最大在线倍率 |
| `name` | string | 节点名称 |
| `needRealname` | boolean | 是否需要实名 |
| `port` | integer | 端口 |
| `protocolSupport` | object | 协议支持情况 |
| `protocolSupport.http` | boolean | 支持 HTTP |
| `protocolSupport.https` | boolean | 支持 HTTPS |
| `protocolSupport.tcp` | boolean | 支持 TCP |
| `protocolSupport.udp` | boolean | 支持 UDP |
| `status` | integer | 状态 |
| `tlsSNIOverwrite` | null \| string | TLS SNI 覆盖 |
| `unitcostEc` | integer | 单位 EC 成本 |

---

### 5. 获取节点状态

**`GET /frp/api/getNodeStatus`**

获取各节点的实时运行状态。

**响应示例**（`200`）：

```json
{
  "data": [
    {
      "client_counts": 0,
      "cur_conns": 0,
      "last_update": 0,
      "load_1": 0.0,
      "load_15": 0.0,
      "load_5": 0.0,
      "m_used_p": 0,
      "maxonline": 0,
      "name": "string",
      "nid": 0,
      "proxy_type_count": "string",
      "status": 0,
      "total_traffic_in": 0,
      "total_traffic_out": 0,
      "uptime": 0,
      "version": "string"
    }
  ],
  "flag": true,
  "msg": "string",
  "detail": null
}
```

**节点状态字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `client_counts` | integer | 客户端数量 |
| `cur_conns` | integer | 当前连接数 |
| `last_update` | integer \| string | 最后更新时间 |
| `load_1` | number | 1 分钟负载 |
| `load_15` | number | 15 分钟负载 |
| `load_5` | number | 5 分钟负载 |
| `m_used_p` | integer | 内存使用百分比 |
| `maxonline` | integer | 最大在线数 |
| `name` | string | 节点名称 |
| `nid` | integer | 节点 ID |
| `proxy_type_count` | string | 代理类型计数 |
| `status` | integer | 状态 |
| `total_traffic_in` | integer | 总入站流量 |
| `total_traffic_out` | integer | 总出站流量 |
| `uptime` | integer | 运行时长 |
| `version` | string | 版本 |

---

### 6. 获取所有用户组

**`POST /frp/api/getAllGroups`**

获取系统中所有用户组信息。

**请求体**（`application/json`）：

```json
{}
```

**响应示例**（`200`）：

```json
{
  "data": [
    {
      "GExpire": "string",
      "GFriendlyName": "string",
      "GID": 0,
      "GInLimit": 0,
      "GName": "string",
      "GOutLimit": 0,
      "GProxies": 0
    }
  ],
  "flag": true,
  "msg": "string",
  "detail": null
}
```

**用户组字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `GExpire` | string | 过期时间 |
| `GFriendlyName` | string | 友好名称 |
| `GID` | integer | 组 ID |
| `GInLimit` | integer | 入站限制 |
| `GName` | string | 组名称 |
| `GOutLimit` | integer | 出站限制 |
| `GProxies` | integer | 隧道数量 |

---

### 7. 获取流量历史

**`POST /frp/api/getTrafficHistory`**

获取指定天数内的流量历史数据，按日期返回。

**请求体**（`application/json`）：

```json
{
  "history_size": 0
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `history_size` | integer | 是 | 历史数据天数 |

**响应示例**（`200`）：

```json
{
  "data": {
    "2026-11-17": 0,
    "2026-11-18": 0,
    "2026-11-19": 0,
    "2026-11-20": 0,
    "2026-11-21": 0,
    "2026-11-22": 0,
    "2026-11-23": 0
  },
  "flag": true,
  "msg": "string",
  "detail": null
}
```

> `data` 对象的键为日期字符串（`YYYY-MM-DD`），值为该日的流量整数。

---

### 8. 获取用户信息

**`POST /frp/api/getUserInfo`**

获取当前登录用户的详细信息。

**请求体**（`application/json`）：

```json
{}
```

**响应示例**（`200`）：

```json
{
  "data": {
    "email": "string",
    "friendlyGroup": "string",
    "group": "string",
    "id": 0,
    "inLimit": 0,
    "outLimit": 0,
    "proxies": 0,
    "realname": true,
    "regTime": "string",
    "token": "string",
    "traffic": 0,
    "used": 0,
    "username": "string"
  },
  "flag": true,
  "msg": "string",
  "detail": null
}
```

**用户字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `email` | string | 邮箱 |
| `friendlyGroup` | string | 友好组名 |
| `group` | string | 组名 |
| `id` | integer | 用户 ID |
| `inLimit` | integer | 入站限制 |
| `outLimit` | integer | 出站限制 |
| `proxies` | integer | 隧道数量 |
| `realname` | boolean | 是否已实名 |
| `regTime` | string | 注册时间 |
| `token` | string | 用户 Token |
| `traffic` | integer | 总流量 |
| `used` | integer | 已用流量 |
| `username` | string | 用户名 |

---

### 9. 启用/禁用隧道

**`POST /frp/api/changeProxy`**

启用或禁用指定的隧道。

**请求体**（`application/json`）：

```json
{
  "proxy_id": 0,
  "proxy_do": true
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `proxy_id` | integer | 是 | 隧道 ID |
| `proxy_do` | boolean | 是 | `true` 启用 / `false` 禁用 |

**响应示例**（`200`）：

```json
{
  "data": null,
  "flag": true,
  "msg": "string",
  "detail": null
}
```

---

### 10. 编辑隧道

**`POST /frp/api/editProxy`**

修改指定隧道的配置信息。

**请求体**（`application/json`）：

```json
{
  "custom": "string",
  "proxy_id": 0,
  "dataGzip": true,
  "dataEncrypt": true,
  "domain_bind": "string",
  "local_addr": "string",
  "local_port": 0,
  "remote_port": 0,
  "node_id": 0,
  "name": "string",
  "type": "string",
  "autoTls": "string",
  "proxyProtocolVersion": true,
  "forceHttps": true
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `custom` | string | 是 | 自定义配置 |
| `proxy_id` | integer | 是 | 隧道 ID |
| `dataGzip` | boolean | 是 | 是否启用 Gzip 压缩 |
| `dataEncrypt` | boolean | 是 | 是否启用加密 |
| `domain_bind` | string | 是 | 绑定域名 |
| `local_addr` | string | 是 | 本地地址 |
| `local_port` | integer | 是 | 本地端口 |
| `remote_port` | integer | 是 | 远程端口 |
| `node_id` | integer | 是 | 节点 ID |
| `name` | string | 是 | 隧道名称 |
| `type` | string | 是 | 隧道类型 |
| `autoTls` | string | 是 | 自动 TLS 配置 |
| `proxyProtocolVersion` | boolean | 是 | 代理协议版本 |
| `forceHttps` | boolean | 是 | 是否强制 HTTPS |

**响应示例**（`200`）：

```json
{
  "data": null,
  "flag": true,
  "msg": "string",
  "detail": null
}
```

---

### 11. 创建隧道

**`POST /frp/api/newProxy`**

创建一个新的 FRP 隧道。

**请求体**（`application/json`）：

```json
{
  "dataEncrypt": true,
  "dataGzip": true,
  "domain_bind": "string",
  "local_addr": "string",
  "local_port": 0,
  "custom": "string",
  "name": "string",
  "node_id": 0,
  "remote_port": 0,
  "type": "string",
  "autoTls": "string",
  "forceHttps": true,
  "proxyProtocolVersion": true
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `dataEncrypt` | boolean | 是 | 是否启用加密 |
| `dataGzip` | boolean | 是 | 是否启用 Gzip 压缩 |
| `domain_bind` | string | 是 | 绑定域名 |
| `local_addr` | string | 是 | 本地地址 |
| `local_port` | integer | 是 | 本地端口 |
| `custom` | string | 是 | 自定义配置 |
| `name` | string | 是 | 隧道名称 |
| `node_id` | integer | 是 | 节点 ID |
| `remote_port` | integer | 是 | 远程端口 |
| `type` | string | 是 | 隧道类型 |
| `autoTls` | string | 是 | 自动 TLS 配置 |
| `forceHttps` | boolean | 是 | 是否强制 HTTPS |
| `proxyProtocolVersion` | boolean | 是 | 代理协议版本 |

**响应示例**（`200`）：

```json
{
  "data": null,
  "flag": true,
  "msg": "string",
  "detail": null
}
```

---

### 12. 删除隧道

**`POST /frp/api/removeProxy`**

删除指定隧道。

**请求体**（`application/json`）：

```json
{
  "proxy_id": 0
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `proxy_id` | integer | 是 | 隧道 ID |

**响应示例**（`200`）：

```json
{
  "data": null,
  "flag": true,
  "msg": "string",
  "detail": null
}
```

---

### 13. 强制下线隧道

**`POST /frp/api/forceOff`**

强制下线指定隧道。

**请求体**（`application/json`）：

```json
{
  "proxy_id": 0
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `proxy_id` | integer | 是 | 隧道 ID |

**响应示例**（`200`）：

```json
{
  "data": null,
  "flag": true,
  "msg": "string",
  "detail": null
}
```

---

### 14. 刷新隧道状态

**`POST /frp/api/refreshProxyStatus`**

刷新单个隧道的在线状态。

**请求体**（`application/json`）：

```json
{
  "proxy_id": 0
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `proxy_id` | integer | 是 | 隧道 ID |

**响应示例**（`200`）：

```json
{
  "data": {
    "p_id": 0,
    "p_online": true
  },
  "flag": true,
  "msg": "string",
  "detail": null
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.p_id` | integer | 隧道 ID |
| `data.p_online` | boolean | 隧道是否在线 |

---

## 附：接口速查表

| # | 方法 | 路径 | 说明 |
|---|------|------|------|
| 1 | POST | `/frp/api/getSignInfo` | 获取签到信息 |
| 2 | GET | `/web/commonQuery/get` | 通用查询 |
| 3 | POST | `/frp/api/getUserProxies` | 获取用户隧道列表 |
| 4 | POST | `/frp/api/getNodeList` | 获取节点列表 |
| 5 | GET | `/frp/api/getNodeStatus` | 获取节点状态 |
| 6 | POST | `/frp/api/getAllGroups` | 获取所有用户组 |
| 7 | POST | `/frp/api/getTrafficHistory` | 获取流量历史 |
| 8 | POST | `/frp/api/getUserInfo` | 获取用户信息 |
| 9 | POST | `/frp/api/changeProxy` | 启用/禁用隧道 |
| 10 | POST | `/frp/api/editProxy` | 编辑隧道 |
| 11 | POST | `/frp/api/newProxy` | 创建隧道 |
| 12 | POST | `/frp/api/removeProxy` | 删除隧道 |
| 13 | POST | `/frp/api/forceOff` | 强制下线隧道 |
| 14 | POST | `/frp/api/refreshProxyStatus` | 刷新隧道状态 |