### 当前apollo主要针对接口

##### apollo
（apollo_api/routers/datasource.py）

apollo 的datasource 模块中的两个关键接口：
1. 测试数据源连接：POST /apollo/dashboard/datasources/link
   - 需要迁移到 Catalog
1. 获取数据库列表：POST /apollo/dashboard/datasources/database_info
   - 需要迁移到 Catalog

##### catlog
对应datacatlog的新接口

POST /catalog/v1/datasources/testconnection

`POST /catalog/v1/datasources/databases`

额外新增了接口：GET /catalog/v1/health


### 当前数据库驱动对应

##### apollo
当前apollo是针对文件（apollo_api/routers/datasource.py）里面的接口：

get_engine_by_type(input, res):

去获得对应的引擎

##### catlog

原本自带引擎：（catalog/engine），自带了很多引擎



### 任务一：接口迁移

接口迁移对应关系
| Apollo 接口 | Catalog 新接口 | 功能 |
|------------|---------------|------|
| POST /apollo/dashboard/datasources/link | POST /catalog/v1/datasources/testconnection | 测试数据源连接 |
| POST /apollo/dashboard/datasources/database_info | POST /catalog/v1/datasources/databases | 获取数据库列表 |
| - | GET /catalog/v1/health | 健康检查（新增） |


迁移后的调用链
前端 → Apollo → Catalog API Server → Catalog Engine → 数据源



### pod配置

![[截屏2026-02-04 15.39.59.png]]
![[截屏2026-02-04 15.40.10.png]]







### 走读问题

- [x] testconnection的接口返回和前端的是否对应修改
- [ ] url = f"{self.base_url}/catalog/v1/health"这个自定义接口是否需要
- [ ] 健康检查代码保持一致
- [ ] option三种保底方法
```
因为 Apollo 和 Catalog API 都使用 Pydantic 模型，只需要：
# ✅ 简化后：只需要 Pydantic 模型方法
option_dict = input.option.dict()
```

- [ ] """数据源类型枚举 - 与 engine.EngineType 对应"""，但是原本apollo的用不用删除旧的引擎接口函数

- [ ] 下面的例子删除

![[截屏2026-02-05 23.53.55.png|300]]




code-wal优化

代码块讲解需要有一句大白话概括这个函数