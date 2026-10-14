# DSP日志堆积巡检项（DSP_LOG_BACKLOG）配置实操指南

## 概述

在 ops-api 的 configmap 中添加 `DSP_LOG_BACKLOG` 巡检项，用于检测 DSP 日志是否严重堆积。通过 PromQL 查询 Pulsar subscription backlog 指标，阈值 10000 条。

## 环境信息

| 项目 | 值 |
|------|-----|
| 服务器 | 10.74.167.22 |
| 集群分支 | develop-DSP3.0.34 |
| 服务 | ops-api (namespace: operations) |
| ConfigMap | ops-api-config |
| 巡检配置文件键 | inspect-items-zh-CN.yaml |
| 前端入口 | 系统运维 > 系统巡检 |

## 一、导出 ConfigMap

```bash
kubectl -n operations get configmap ops-api-config -o yaml > /root/wjx/ops-api-config-latest.yaml
```

## 二、查看现有巡检配置结构

```bash
# 查看 configmap 中有哪些数据键
kubectl -n operations get configmap ops-api-config -o jsonpath='{.data}' | python3 -c "import sys,json; print('\n'.join(json.load(sys.stdin).keys()))"

# 查看巡检配置中的 Pulsar 相关巡检项
kubectl -n operations get configmap ops-api-config -o jsonpath='{.data.inspect-items-zh-CN\.yaml}' | grep -A15 "PULSAR_MSG_BACKLOG"
```

现有 Pulsar 巡检项（参考格式）：

```yaml
              - name: "PULSAR_MSG_BACKLOG"
                description: "消息积压总量超过阈值"
                scenarios: ["BUSINESS", "FULL"]
                plugin_type: "vm"
                plugin_config:
                  query: "sum(pulsar_subscription_back_log{}) > 30000000"
                  result_failure_expr: "{{gt .value 30000000}}"
                  failure_level: "warn"
                  result_format: "string"
                  success_template: "检测到您当前平台的Pulsar消息积压量正常..."
                  failure_template: "检测到您当前平台的Pulsar消息积压总量超过3000万条..."
```

## 三、在线编辑 ConfigMap 添加巡检项

```bash
kubectl -n operations edit configmap ops-api-config
```

**操作步骤：**

1. 输入 `:set paste` 开启粘贴模式（防止自动缩进）
2. 输入 `/PULSAR_MSG_BACKLOG` 回车，定位到该巡检项
3. 找到 `PULSAR_MSG_BACKLOG` 的 `failure_template` 行末尾
4. 按 `o` 新开一行，粘贴以下内容：

```yaml
              - name: "DSP_LOG_BACKLOG"
                description: "DSP日志是否严重堆积"
                scenarios: ["BUSINESS", "FULL"]
                plugin_type: "vm"
                plugin_config:
                  query: 'sum(pulsar_subscription_back_log{topic=~"persistent://10001001/default/http_log|persistent://10001001/default/dsp_http_log.*"})'
                  duration: "5m"
                  step: "60"
                  result_failure_expr: "{{gt .value 10000}}"
                  failure_level: "warn"
                  result_format: "string"
                  success_template: "日志堆积正常，当前堆积量: {{.value}} 条"
                  failure_template: "日志严重堆积，当前堆积量: {{.value}} 条，超过阈值 10000 条，请检查富化服务和入库服务是否正常"
```

5. 按 `Esc` → `:set nopaste` → `:wq` 保存

**注意事项：**
- `- name: "DSP_LOG_BACKLOG"` 的 `-` 必须和上面 `PULSAR_MSG_BACKLOG` 的 `-` 完全对齐（14个空格）
- `plugin_config` 下的字段（query、duration 等）必须和上面的 `query` 对齐（18个空格）
- 不能用 Tab，必须全部是空格

## 四、重启 ops-api 使配置生效

```bash
kubectl -n operations rollout restart deployment ops-api
kubectl -n operations rollout status deployment ops-api --timeout=120s
```

## 五、验证配置已写入

```bash
# 验证 configmap 中包含新巡检项
kubectl -n operations get configmap ops-api-config -o jsonpath='{.data.inspect-items-zh-CN\.yaml}' | grep -A15 "DSP_LOG_BACKLOG"
```

预期输出：

```
          - name: "DSP_LOG_BACKLOG"
            description: "DSP日志是否严重堆积"
            scenarios: ["BUSINESS", "FULL"]
            plugin_type: "vm"
            plugin_config:
              query: 'sum(pulsar_subscription_back_log{topic=~"persistent://10001001/default/http_log|persistent://10001001/default/dsp_http_log.*"})'
              duration: "5m"
              step: "60"
              result_failure_expr: "{{gt .value 10000}}"
              failure_level: "warn"
              result_format: "string"
              success_template: "日志堆积正常，当前堆积量: {{.value}} 条"
              failure_template: "日志严重堆积，当前堆积量: {{.value}} 条，超过阈值 10000 条，请检查富化服务和入库服务是否正常"
```

## 六、验证巡检执行

### 6.1 查看 ops-api 日志

```bash
kubectl -n operations logs deployment/ops-api --tail=500 | grep -i "DSP_LOG_BACKLOG"
```

预期输出（前端触发巡检后）：

```
item history created: taskID=..., itemName=DSP_LOG_BACKLOG, status=pending
item history updated: taskID=..., itemName=DSP_LOG_BACKLOG, status=running
item history updated: taskID=..., itemName=DSP_LOG_BACKLOG, status=finished
```

### 6.2 后台直接查询 PromQL 指标

```bash
# port-forward 到 VictoriaMetrics
kubectl -n monitor-platform port-forward svc/vmselect-vmcluster 18481:8481 &
sleep 2

# 查询堆积量
curl -s -G 'http://localhost:18481/select/0/prometheus/api/v1/query' \
  --data-urlencode 'query=sum(pulsar_subscription_back_log{topic=~"persistent://10001001/default/http_log|persistent://10001001/default/dsp_http_log.*"})' \
  | python3 -m json.tool

# 关闭 port-forward
kill %1
```

预期返回：

```json
{
    "status": "success",
    "data": {
        "resultType": "vector",
        "result": [
            {
                "metric": {},
                "value": [
                    1773991671,
                    "0"
                ]
            }
        ]
    },
    "stats": {
        "seriesFetched": "2"
    }
}
```

- `value[1]` = `"0"` 表示当前堆积量为 0 条
- `seriesFetched: "2"` 表示匹配到 2 个 topic

### 6.3 Grafana 面板查询

访问 Grafana → Explore 页面，数据源选择 VictoriaMetrics/Prometheus，输入：

```promql
sum(pulsar_subscription_back_log{topic=~"persistent://10001001/default/http_log|persistent://10001001/default/dsp_http_log.*"})
```

查看各 topic 分别的堆积量：

```promql
pulsar_subscription_back_log{topic=~"persistent://10001001/default/http_log|persistent://10001001/default/dsp_http_log.*"}
```

### 6.4 前端验证

访问平台 → 系统运维 → 系统巡检，执行"效果巡检"或"全量巡检"，检查 DSP_LOG_BACKLOG 项是否出现并显示结果。

## 七、巡检项配置字段说明

| 字段 | 值 | 说明 |
|------|-----|------|
| name | DSP_LOG_BACKLOG | 巡检项唯一标识 |
| scenarios | ["BUSINESS", "FULL"] | 在业务巡检和全量巡检中执行 |
| plugin_type | vm | 使用 VictoriaMetrics 查询插件 |
| query | sum(pulsar_subscription_back_log{topic=~"..."}) | 查询 DSP 相关 topic 的堆积总量 |
| duration | 5m | 查询时间范围 5 分钟 |
| step | 60 | 查询步长 60 秒 |
| result_failure_expr | {{gt .value 10000}} | 堆积量 > 10000 判定为异常 |
| failure_level | warn | 异常级别为警告 |

## 八、踩坑记录

### 8.1 override 覆盖机制不生效

最初尝试在 configmap 中添加 `inspect-items-override-zh-CN.yaml` 键，期望 ops-api 自动合并覆盖配置。

**结果**：不生效。原因是 deployment 的 volumeMounts 中没有挂载 `inspect-items-override-zh-CN.yaml`，ops-api 只读取以下文件：

```
/etc/ops/inspect/inspect-items-zh-CN.yaml
/etc/ops/inspect/inspect-items-en-US.yaml
```

**解决**：直接在 `inspect-items-zh-CN.yaml` 中添加巡检项。

### 8.2 ConfigMap apply 版本冲突

```
the object has been modified; please apply your changes to the latest version and try again
```

**原因**：导出 configmap 后，中间有其他操作（如 rollout restart）修改了 resourceVersion，导致 apply 时版本不匹配。

**解决**：使用 `kubectl edit` 在线编辑，避免版本冲突问题。

### 8.3 vi 粘贴缩进错乱

**原因**：vi 默认的自动缩进会在粘贴时添加额外空格。

**解决**：粘贴前执行 `:set paste`，粘贴后执行 `:set nopaste`。

### 8.4 API 端口确认

ops-api 的服务端口是 **8080**（不是 8088）：

```bash
kubectl -n operations get svc ops-api -o wide
# PORT(S): 8080/TCP,7770/TCP
```

---

> 日期：2026-03-20
> 集群：10.74.167.22
> 分支：develop-DSP3.0.34
