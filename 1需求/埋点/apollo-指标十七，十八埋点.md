```
对外提供接口
/apollo/internal/开头的，在apolloserver/apollo_api/apps/internal/views里面

例如：

@router.post("/internal/risk_visualization/data_label_by_ids", summary="获取静态数据相关数据",
             response_model=List[schemas.DataLabelInfo])
async def get_data_source_by_ids(info: schemas.DataLabelIds, session: Session = Depends()):
    return service.DataLabel.get_label_by_ids(session, info)



五个对外接口
  1. 接口路径规律

  统一前缀: /apollo/internal/risk_visualization/

  具体路径:
  - /apollo/internal/risk_visualization/data_category - 数据分类查询
  - /apollo/internal/risk_visualization/data_source_by_ids - 数据源ID查询
  - /apollo/internal/risk_visualization/data_label_by_ids - 标签ID查询
  - /apollo/internal/risk_visualization/data_label_by_pages - 标签分页查询
  - /apollo/internal/risk_visualization/data_level - 数据级别查询









指标定义：

  1. 指标定义和注册 (apollo_api/monitor/metrics.py)

  指标十七：API时延Histogram

  # 指标十七：apollo对外提供API时延Histogram
  API_LATENCY_HISTOGRAM = Histogram(
      'apollo_api_latency_seconds',           # 指标名称
      'Apollo API request latency in seconds', # 指标描述
      ['endpoint', 'status', 'method'],        # 标签维度
      buckets=[0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]  # 12个桶
  )

  指标十八：API请求总数Counter

  # 指标十八：API请求总数Counter
  API_REQUEST_COUNTER = Counter(
      'apollo_api_requests_total',             # 指标名称
      'Total number of API requests',          # 指标描述
      ['endpoint', 'status', 'method']         # 标签维度
  )

  指标注册机制

  def register_metrics():
      """注册所有Prometheus指标"""
      try:
          REGISTRY.register(API_LATENCY_HISTOGRAM)  # 注册指标十七
      except ValueError:
          pass  # 如果已注册则忽略

      try:
          REGISTRY.register(API_REQUEST_COUNTER)    # 注册指标十八
      except ValueError:
          pass


  # 初始化时自动注册指标
  register_metrics()
  
  
  
  
  2. 埋点中间件实现 (apollo_api/middlewares/api_metrics.py)

  中间件类定义

  class APIMetricsMiddleware(BaseHTTPMiddleware):
      """对外API性能监控中间件"""

      def __init__(self, app):
          super().__init__(app)
          self.route_mappings = RouteActionMappings()

  设计特点：
  - 继承关系: 继承BaseHTTPMiddleware，获得标准的中间件能力
  - 初始化优化: 在__init__中预加载路由映射，避免每次请求重复加载
  - 配置复用: 直接复用RouteActionMappings配置，保持一致性


  核心埋点逻辑

  async def dispatch(self, request: Request, call_next):
      """记录API时延和请求计数"""
      # 1. 记录开始时间
      start_time = time.time()

      # 2. 执行请求 (继续处理后续中间件和业务逻辑)
      response = await call_next(request)

      # 3. 计算时延
      duration = time.time() - start_time

      # 4. 获取endpoint信息
      endpoint = self._get_endpoint_name(request)

      # 5. 过滤：只统计/apollo/internal/开头的对外业务API
      if endpoint is None:
          return response

      # 6. 指标埋点调用
      record_api_latency(endpoint, str(response.status_code), request.method, duration)  # 
  指标十七
      record_api_request(endpoint, str(response.status_code), request.method)           # 
  指标十八
      return response

  Endpoint获取和过滤逻辑





把复杂的URL路径转换成简单易懂的业务名称，方便做监控统计
  def _get_endpoint_name(self, request: Request) -> str:
      """从路由映射获取endpoint名称"""
      method = request.method
      path = request.url.path

      # 过滤：只统计/apollo/internal/开头的对外业务API
      if not path.startswith("/apollo/internal/"):
          return None

      # 使用现有的路由映射逻辑
      route_key = f"{method}:{path}"

      # O(1)查找普通路由
      if route_key in self.route_mappings.NORMAL_ROUTE_ACTION_MAP:
          return self.route_mappings.NORMAL_ROUTE_ACTION_MAP[route_key].get("resource", path)

      # 正则路由匹配
      for regex, rule_method, info in self.route_mappings.REGEX_ROUTE_ACTION_MAP:
          if method == rule_method and regex.match(path):
              return info.get("resource", path)

      # 默认返回路径（去除路径参数）
      return self._extract_endpoint_from_path(path)


  想象一下，你有很多API请求：
  - GET /apollo/dashboard/datasources/123
  - GET /apollo/dashboard/datasources/456
  - POST /apollo/internal/risk_visualization/data_category

  如果不做转换，监控数据会显示：
  - /apollo/dashboard/datasources/123 - 10次请求
  - /apollo/dashboard/datasources/456 - 8次请求
  - /apollo/internal/risk_visualization/data_category - 5次请求

  这样统计很分散，看不出来真正的业务情况。

  转换后变成：
  - Datasource Configuration - 18次请求  ✅ 业务清晰
  - /apollo/internal/risk_visualization/data_category - 5次请求



  三种方法的具体解释

  确实提供了三种方法，按从快到慢的顺序执行：

  方法1：精确匹配（O(1) 最快）

  route_key = f"{method}:{path}"
  if route_key in self.route_mappings.NORMAL_ROUTE_ACTION_MAP:
      return self.route_mappings.NORMAL_ROUTE_ACTION_MAP[route_key].get("resource", path)

  什么情况用？
  - URL完全固定的接口
  - 例如：POST:/apollo/dashboard/datasources

  怎么找？
  - 就像查字典一样，直接翻到那一页
  - 速度最快，O(1)时间复杂度

  返回什么？
  - 返回配置文件中预设的业务名称
  - 如："Datasource Configuration"

  方法2：正则匹配（中等速度）

  for regex, rule_method, info in self.route_mappings.REGEX_ROUTE_ACTION_MAP:
      if method == rule_method and regex.match(path):
          return info.get("resource", path)

  什么情况用？
  - URL中有参数的接口
  - 例如：PUT:/apollo/dashboard/datasources/123
  - 例如：DELETE:/apollo/dashboard/datasources/456

  怎么找？
  - 就像用模式匹配找东西
  - 例如：找所有符合"datasources/数字"的URL
  - 速度中等，需要逐一匹配

  返回什么？
  - 同样返回业务名称
  - 如："Datasource Configuration"

  方法3：智能路径提取（兜底方法）

  return self._extract_endpoint_from_path(path)

  什么情况用？
  - 前两种方法都找不到匹配时
  - 例如：/apollo/internal/risk_visualization/data_category/123

  怎么找？
  - 自动去掉路径中的ID参数
  - /apollo/internal/risk_visualization/data_category/123
  - → /apollo/internal/risk_visualization/data_category

  返回什么？
  - 返回去掉参数后的路径
  - 如：/apollo/internal/risk_visualization/data_category








  路径参数提取逻辑

  def _extract_endpoint_from_path(self, path: str) -> str:
      """从路径提取endpoint名称，去除路径参数"""
      # 示例：/apollo/internal/risk_visualization/data_category/123 -> 
  /apollo/internal/risk_visualization/data_category
      parts = path.split('/')
      result_parts = []

      for part in parts:
          # 跳过空字符串和纯数字（通常是ID参数）
          if part and not part.isdigit():
              result_parts.append(part)
          elif part and part.isdigit():
              # 遇到数字参数后停止，因为后面的可能是其他参数
              break

      return '/' + '/'.join(result_parts)



 算法详解：

  输入示例:
  path = "/apollo/internal/risk_visualization/data_category/123"

  处理过程:
  1. 分割路径: ["", "apollo", "internal", "risk_visualization", "data_category", "123"]
  2. 逐个检查:
    - "": 跳过（空字符串）
    - "apollo": 保留（不是数字）
    - "internal": 保留（不是数字）
    - "risk_visualization": 保留（不是数字）
    - "data_category": 保留（不是数字）
    - "123": 停止（遇到数字参数）

  输出结果:
  "/apollo/internal/risk_visualization/data_category"



```



![[截屏2025-12-02 19.33.27.png]]



![[截屏2025-12-02 19.33.39.png]]