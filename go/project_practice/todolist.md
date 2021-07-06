## TodoList
[demo](http://www.todolist.cn)，只需完成后端api接口

### 需求
- 创建、更新、删除
- 完成/取消完成
- 搜索（搜索条件：1、名称-模糊匹配 2、日期范围 3、是否完成）
- 统计最近六个月的每个月完成数和新增数，要求使用redis存储统计数据
- 进阶需求：拖动排序（在网上搜索一下后端应该如何实现）

### 步骤
- 设计出表结构
- 设计所有需要的api接口，遵循[restful风格](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
- 代码实现

### 依赖库
- https://github.com/go-chi/chi
- https://github.com/timonwong/dbr
- https://github.com/go-redis/redis
