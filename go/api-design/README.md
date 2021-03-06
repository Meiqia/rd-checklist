# API 设计指南

## 目录

- [概念](#概念)
- [资源名称](#资源名称)
    - [资源集合用复数名](#资源集合用复数名)
    - [单个资源用 ID 标识](#单个资源用-id-标识)
        - [已鉴权用户](#已鉴权用户)
        - [多种格式的资源 ID](#多种格式的资源-id)
    - [资源单例用单数名或复数名](#资源单例用单数名或复数名)
- [标准方法](#标准方法)
    - [集合资源](#集合资源)
    - [单例资源](#单例资源)
- [自定义方法](#自定义方法)
- [参考资料](#参考资料)


## 概念

- Tenant：租户
- Conversation：对话
- Message：消息
- Visitor：访客
- Agent：客服


## 资源名称

### 资源集合用复数名

| 资源 | Bad | Good |
| --- | --- | --- |
| 租户集合 | /tenant | /tenants |
| 租户下的对话集合 | /tenant/conversation | /tenants/{tenantID}/conversations |

### 单个资源用 ID 标识

| 资源 | Bad | Good |
| --- | --- | --- |
| 一个租户 | /tenant | /tenants/{tenantID} |
| 租户下的一条对话 | /tenant/conversation | /tenants/{tenantID}/conversations/{convID} |
| 对话中的一条消息 | /tenant/conversation/message | /tenants/{tenantID}/conversations/{convID}/messages/{messageID} |

#### 已鉴权用户

对于已鉴权用户，URI 可以使用 me 作为资源 ID（[参考1](https://developers.google.com/gmail/api/reference/rest/v1/users.messages.attachments/get#path-parameters)、[参考2](https://softwareengineering.stackexchange.com/questions/273348/should-i-specify-the-userid-in-the-rest-url-structure)、[参考3](https://stackoverflow.com/questions/35719797/is-using-magic-me-self-resource-identifiers-going-against-rest-principles)）。

| 资源 | Bad | Good |
| --- | --- | --- |
| 当前客服所属租户 | /tenant | /tenants/me |
| 当前客服接待的对话集合 | /agent/conversations | /agents/me/conversations |
| 当前客服在某条对话中的回复消息集合 | /agent/conversation/messages | /agents/me/conversations/{convID}/messages |

#### 多种格式的资源 ID

有时候，我们想支持多种风格的资源定位方式，比如可以根据 name 或 id 定位资源，此时资源 ID 可以是多种格式的（[参考](https://docs.konghq.com/gateway/2.7.x/admin-api/#retrieve-service)）。

| 资源 | Bad | Good |
| --- | --- | --- |
| 根据 name 或 id 定位单个租户 | /tenant?name=xx&id=yy | /tenants/{name or id} |
| 根据 name 或 id 定位单个客服 | /agent?name=xx&id=yy | /agents/{name or id} |

### 资源单例用单数名或复数名

| 资源 | Bad | Good |
| --- | --- | --- |
| 对话的接待客服（一条对话只有一个接待客服） | /tenant/conversation/agent | /tenants/{tenantID}/conversations/{convID}/agent |
| 客服的个人设置（一个客服只有一套个人设置） | /agent/settings | /agents/{agentID}/settings |


## 标准方法

### 集合资源

| 场景 | 函数方法 | HTTP 方法 | 请求体 | 响应状态码 | 响应体 | 示例 |
| --- | --- | --- | --- | --- | --- | --- |
| 获取资源列表 | List | GET | 无 | 200 | 资源列表 | GET /tenants |
| 获取单个资源 | Get | GET | 无 | 200 | 单个资源 | GET /tenants/{tenantID} |
| 创建资源 | Create | POST | 单个资源 | 200（或 204） | 单个资源（或无） | POST /tenants |
| 更新资源 | Update | PUT （或 PATCH 部分更新）| 单个资源 | 200（或 204） | 单个资源（或无） | PUT /tenants/{tenantID} |
| 删除资源 | Delete | DELETE | 无 | 204 | 无 | DELETE /tenants/{tenantID} |

备注：

- 创建/更新资源通常返回 200；如果不需要返回已创建/已更新的资源内容，也可以返回 204（响应体为空）。


### 单例资源

| 场景 | 函数方法 | HTTP 方法 | 请求体 | 响应状态码 | 响应体 | 示例 |
| --- | --- | --- | --- | --- | --- | --- |
| 获取资源列表 | List | / | / | / | / | / |
| 获取单个资源 | Get | GET | 无 | 200 | 单个资源 | GET /conversations/{convID}/agent |
| 创建资源 | Create | / | / | / | / | / |
| 更新资源 | Update | PUT （或 PATCH 部分更新）| 单个资源 | 200（或 204） | 单个资源（或无） | PUT /conversations/{convID}/agent |
| 删除资源 | Delete | / | / | / | / | / |

备注（[参考](https://cloud.google.com/apis/design/design_patterns#singleton_resources)）:

- 单例资源不能定义 Create、Delete 或者 List 方法
- 单例资源默认随着父资源的创建（或删除）而创建（或删除）；如果单例资源没有父资源，那就隐含它是存在的
- 标准方法只能是 Get 和 Update，当然也可以使用自定义方法


## 自定义方法

如果某些操作场景，用标准方法表达起来不够自然，那可以选择自定义方法。

典型场景（[参考](https://cloud.google.com/apis/design/custom_methods)）：

| 场景 | 函数方法 | 示例 |
| --- | --- | --- |
| 批量获取 | BatchGet | GET /conversations:batchGet |
| 发邮件 | SendMail | POST /operations:sendMail |
| 复杂查询 | Search | POST /conversations:search |
| 复制 | copy | POST /settings:copy |


## 参考资料

- [Google API design guide](https://cloud.google.com/apis/design)