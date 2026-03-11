# 实施计划: {feature-name}

## TASKS

- TASK:data-model DEPENDS:none COMPLEXITY:medium FILES:src/models/role.ts,src/migrations/001.ts
  创建角色数据模型和数据库迁移

- TASK:core-logic DEPENDS:none COMPLEXITY:high FILES:src/services/rbac.ts
  实现RBAC核心权限判定逻辑

- TASK:api-layer DEPENDS:data-model,core-logic COMPLEXITY:medium FILES:src/routes/role.ts
  实现角色管理API接口

- TASK:tests DEPENDS:api-layer COMPLEXITY:medium FILES:tests/rbac.test.ts
  编写单元测试和集成测试

## WAVES

- WAVE-1: data-model, core-logic
- WAVE-2: api-layer
- WAVE-3: tests

## ACCEPTANCE

- [ ] 所有测试通过
- [ ] 接口符合设计文档定义
- [ ] 代码无lint错误
