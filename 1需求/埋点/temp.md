#### 流程图

![[截屏2025-12-26 14.52.28.png]]
![[截屏2025-12-26 14.52.42.png]]
![[截屏2025-12-26 14.53.18.png]]

### 耗时日志


  | 阶段                       | 耗时    | 说明                    |
  |--------------------------|-------|-----------------------|
  | 阶段1: data-seat 获取 appIds | 2.27s | 4批请求，共获取 8351 个 appId |
  | 阶段2: warehouse 获取 API 统计 | 6.07s | 查询 499,854 条 API 数据   |
  | 总耗时                      | 8.34s |                       |


   主要瓶颈在阶段2 (warehouse)，占了总耗时的 72% (6.07s / 8.34s)

  具体来说：
  - warehouse 服务的 mission_overview_api_stats 接口耗时 6.044s
  - 需要处理 499,854 条 API 记录（约50万条）


 
   1. 找 warehouse 组协助优化 - 这个接口
  http://dsp-data-asset:8002/dataasset/public/api/mission_overview_api_stats 查询 50 万条数据花了 6
  秒，可能需要：
    - 添加数据库索引
    - 优化查询语句
    - 考虑缓存机制
  2. 阶段1 也有优化空间 - data-seat 的 4 批请求共 2.27s，可以考虑：
    - 增大 limit 值（目前是 2500）
    - 或者并行请求多批

  你现在可以拿着这个数据去和 warehouse 组沟通，让他们排查 mission_overview_api_stats 接口为什么慢。


 
 
 
 
 接口流程梳理

  整体流程图

  前端 POST 请求
      ↓
  mission_overview() 入口 (mission_center.py:583)
      ↓
  ┌─────────────────────────────────────────────────────────┐
  │ 阶段1: 获取 appIds (如果全选应用)                         │
  │   └─ get_selected_appid() → query_app_asset_with_filter()│
  │      调用 data-seat 服务获取 appIds                       │
  └─────────────────────────────────────────────────────────┘
      ↓
  ┌─────────────────────────────────────────────────────────┐
  │ 阶段2: 获取 database_ids (如果选择数据源)                 │
  │   └─ models.DataBase.list_database_ids()                 │
  └─────────────────────────────────────────────────────────┘
      ↓
  ┌─────────────────────────────────────────────────────────┐
  │ 阶段3: 静态任务计数 (数据库字段分类)                      │
  │   └─ models.DataField.get_field_by_database_ids()        │
  │      (本地数据库查询)                                     │
  └─────────────────────────────────────────────────────────┘
      ↓
  ┌─────────────────────────────────────────────────────────┐
  │ 阶段4: 动态任务计数 (API分类) ← 你关注的部分              │
  │   └─ external_models.API.get_api_and_manual_count()      │
  │      调用 warehouse 服务获取 API 统计                     │
  └─────────────────────────────────────────────────────────┘
      ↓
  返回 MissionPreview 响应

  ---
  关键文件和函数位置

  | 阶段  | 文件路径                                                     | 函数
           | 行号      |
  |-----|----------------------------------------------------------|-------------------------------
  -|---------|
  | 入口  | apollo_api/routers/dspm/mission_center/mission_center.py | mission_overview()
     | 583-606 |
  | 阶段1 | 同上                                                       |
  query_app_asset_with_filter()  | 55-87   |
  | 阶段4 | apollo_api/models/external/api.py                        |
  API.get_api_and_manual_count() | 52-78   |

  ---
  外部服务调用

  1. 调用 data-seat 服务 (获取 appIds):
    - URL:
  http://dsp-data-asset.dsp.svc.cluster.local:8002/dataasset/openapi/dashboard/apps/selector
    - 分批获取，每批 2500 条
  2. 调用 warehouse 服务 (获取 API 统计):
    - URL: http://dsp-data-asset:8002/dataasset/public/api/mission_overview_api_stats

  ---
  建议打日志的位置

  根据你的需求（只关注查询到获得 API 的时延），建议在以下位置添加耗时日志：

  位置1: mission_overview() 入口

  文件: apollo_api/routers/dspm/mission_center/mission_center.py:583

  import time

  async def mission_overview(info: schemas.MissionOverviewReq, session: Session = Depends()):
      total_start = time.time()
      dlog.info(f"[性能] mission_overview 开始, isSelectAll={info.isSelectAll},
  appIds数量={len(info.appIds) if info.appIds else 0}")

  位置2: 获取 appIds 阶段

  文件: apollo_api/routers/dspm/mission_center/mission_center.py:584-585

      # 阶段1: 获取appIds
      if info.isSelectAll:
          stage1_start = time.time()
          info.appIds = get_selected_appid(info.matchFilters, info.oppositeIds)
          dlog.info(f"[性能] 阶段1-获取appIds 耗时: {time.time() - stage1_start:.3f}s, 获取到
  {len(info.appIds)} 个appId")

  位置3: query_app_asset_with_filter() 内部每批次

  文件: apollo_api/routers/dspm/mission_center/mission_center.py:55-87

  def query_app_asset_with_filter(filter_list: List[schemas.AssetFilter]) -> List[str]:
      func_start = time.time()
      batch_count = 0
      # ... 在 while 循环内 ...
      while True:
          batch_start = time.time()
          response = requests.post(DATA_ASSET_SVC_APP_SELECTOR_URL, json=json_data)
          batch_count += 1
          dlog.info(f"[性能] data-seat 第{batch_count}批请求耗时: {time.time() - batch_start:.3f}s,
   skip={skip}")
          # ...
      dlog.info(f"[性能] query_app_asset_with_filter 总耗时: {time.time() - func_start:.3f}s")

  位置4: 获取 API 统计阶段

  文件: apollo_api/models/external/api.py:52-78

  @staticmethod
  def get_api_and_manual_count(ids, mission_overview: schemas.MissionOverviewReq):
      stage_start = time.time()
      dlog.info(f"[性能] 阶段4-获取API统计 开始, appIds数量={len(ids) if ids else 0}")
      # ... 原有代码 ...
      response = requests.post(ASSET_URL_MAP["mission_overview_api_stats"], json=req_params






## 完整流程原理图

![[截屏2025-12-26 17.10.01.png]]
![[截屏2025-12-26 17.10.17.png]]

![[截屏2025-12-26 17.10.33.png]]

![[截屏2025-12-26 17.10.51.png]]

![[截屏2025-12-26 17.11.17.png]]

![[截屏2025-12-26 17.11.30.png]]


## MongoDB查询：

![[截屏2025-12-29 10.05.22.png]]

![[截屏2025-12-29 10.05.44.png]]