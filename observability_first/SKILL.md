---
name: Observability First
description: Crash/ANR/Logs/Performance 指标与回馈闭环
---

# Observability First (可观测性优先)

## Instructions
- 先定义要观测的关键流程与指标
- 依序创建 Crash/ANR、结构化日志、性能指标
- 一次只补强一类信号，避免噪音扩散
- 完成后对照 Quick Checklist

## When to Use
- 发布前需要稳定监控与回馈闭环
- 事故频繁但缺乏可定位信息
- 需要把性能与稳定性纳入日常决策

## Example Prompts
- "请创建支付流程的可观测性指标与事件"
- "请设计 Crash/ANR 的告警门槛"
- "帮我创建结构化日志的字段规格"

## Workflow
1. 定义关键流程与 SLO
2. 创建 Crash/ANR 与结构化事件
3. 加入性能指标与告警门槛
4. 创建回馈回路（分析 -> 修复 -> 验证）

## Practical Notes (2026)
- 先有指标再谈优化，避免主观调整
- 事件字段需一致，便于查找与汇整
- 告警门槛要可运行且可回溯

## Minimal Template
```
目标: 
关键流程: 
指标/事件: 
告警门槛: 
验收: Quick Checklist
```

---

## Signals & SLOs

### 关键流程清单
- 启动、登录、主要交易流程
- 网络请求成功率、平均延迟
- ANR/Crash 率与回复时间

### 指标分级
- P0: Crash/ANR 率、关键流程失败率
- P1: 首次渲染时间、列表滚动流畅度
- P2: 特定功能的转换率或完成率

---

## Structured Events

### 事件字段规格
- event_name, flow_id, user_tier, build_version
- latency_ms, result, error_code

### 事件纪录原则
- 只记录高价值事件，避免淹没
- 同一流程使用一致字段

---

## Crash / ANR Strategy

- Crash/ANR 需标注关键上下文字段
- Non-Fatal 只记录高价值错误
- 针对高频问题创建告警

---

## Performance Signals

- Startup、列表滚动、关键页面渲染
- 量测结果进 CI Gate

---

## Quick Checklist

- [ ] 关键流程与 SLO 定义完成
- [ ] Crash/ANR 字段一致且可追踪
- [ ] 事件字段可查找且可汇整
- [ ] 性能指标有量测与门槛
- [ ] 告警与回馈回路已创建
