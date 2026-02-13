---
name: Supply Chain Security
description: 依赖治理、SCA、签章与密钥管理
---

# Supply Chain Security (供应链安全)

## Instructions
- 先盘点依赖来源与版本策略
- 创建 SCA 扫描与审核流程
- 一次只强化一个供应链节点
- 完成后对照 Quick Checklist

## When to Use
- 项目依赖多、更新频繁
- 发布前需要风险检查
- 需要创建依赖治理标准

## Example Prompts
- "请设计依赖版本锁定与更新策略"
- "帮我加上 SCA 扫描与风险门槛"
- "请创建密钥与签章管理规范"

## Workflow
1. 创建依赖来源与版本锁定策略
2. 加入 SCA 扫描与审核流程
3. 设置签章与密钥管理规范
4. 将风险门槛纳入 CI Gate

## Practical Notes (2026)
- 版本锁定与审核是最小安全基线
- 依赖更新与安全修补分开处理
- SCA 结果必须有处置规则

## Minimal Template
```
目标: 
依赖来源: 
版本策略: 
SCA 门槛: 
验收: Quick Checklist
```

---

## Dependency Governance

- 使用 Version Catalog 作为单一来源
- 重大更新需审核与回归测试
- 禁止未授权的第三方来源

---

## SCA / Vulnerability Policy

- 将依赖扫描纳入 CI
- 高风险漏洞阻挡合并
- 低风险需标注与期限修补

---

## Signing & Secrets

- 密钥仅由环境变量或安全保存注入
- Release 签章流程需可追踪
- 禁止在 Repo 内存放敏感信息

---

## Quick Checklist

- [ ] 依赖来源与版本锁定完成
- [ ] SCA 扫描纳入 CI Gate
- [ ] 风险处置规则明确
- [ ] 签章与密钥管理可追踪
- [ ] Release 前完成依赖风险检查
