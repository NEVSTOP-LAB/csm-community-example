---
name: csm-module-config
description: '完善 csmapp.ini 的 [csm-module] 节，并校验路径真实存在（缺失则自动修复，无法修复时告警）。触发：「更新模块配置」「同步模块信息」「刷新 csm-module」。数据源：.lvcsm 为主（路径权威），csm-modules.yaml 为辅（索引各模块 README）。'
argument-hint: '[模块名]'
user-invocable: true
disable-model-invocation: true
---

# CSM 模块配置更新

完善 `csmapp.ini` 的 `[csm-module]` 节。

## 数据源

| 优先级 | 数据源 | 用途 |
|--------|--------|------|
| 主要 | `<项目>.lvcsm` 中 `[CSMVI.*]` 的 `Path` | VI 路径（扫描模块获得，权威） |
| 辅助 | `csm/csm-modules.yaml` 的 `path` → 各模块 README「模块列表」 | 识别入口 VI、剔除辅助/测试 VI |

> `.lvcsm` 的 `[CSMModule.*]` 是实例配置，**不是** `[csm-module]`，忽略。

## 步骤

**1. 提取 VI 路径** — 读 `.lvcsm`，取所有 `[CSMVI.<库>:<vi>.vi]` 的 `Path`，转 `<AppDir>/csm/...` 为 `csm\...`。过滤 `_example/`、`_test/`、`_support/`、`CSM-Utils` 下的 VI。

**2. 确认模块目录** — 读 `csm-modules.yaml`，用各条目 `path` 将步骤一的 VI 按模块目录归类。

**3. 识别入口 VI** — 按 yaml 的 `path` 读各模块 `README.md`「模块列表」：
- 表中列出的 VI → 入口点
- 多候选 → 以表格为准，优先无后缀的纯 VI 名
- `.lvcsm` 有但表格未列 → 标记人工复核

**4. 推导别名** — 从模块目录名去前缀（`CSM-ModSets-`/`CSM-Modsets-`）；若 `csmapp.ini` 已有则沿用。

**5. 写入** — 更新 `[csm-module]`：`<别名> = csm\<相对路径>`。保留有效条目，删除过期条目。

**6. 校验（必须）** — 逐条检查路径指向的 `.vi` 是否真实存在：
- 不存在 → 在 `csm/` 下递归搜同名 `.vi`；唯一匹配自动修正，多个取最简路径
- 搜不到 → 告警：别名、当前路径、搜索范围，建议手动检查

## 已有条目处理优先级

对 `[csm-module]` 中已存在的条目，按以下优先级处理（不得静默删除）：

1. **路径存在且属于已注册模块** → 保留或更新
2. **路径存在但不属于已注册模块** → 保留并告警为人工复核
3. **路径不存在** → 执行自动修复（搜同名 `.vi`）
4. **自动修复失败** → 保留原条目并报告无效路径，不静默删除

## 决策速查

| 场景 | 处理 |
|------|------|
| `csm-modules.yaml` 缺失或解析失败 | 不删除任何现有条目；仅基于 `.lvcsm` 生成候选；报告注册表不可用 |
| 无「模块列表」 | 目录下非辅助 VI 均视为候选 |
| 多 VI 候选 | 选无 `(SingleRef)`/`(Multi-Refs)` 后缀的 |
| `.lvcsm` 有但不在注册目录 | 标记人工复核 |
| 路径文件不存在 | 搜同名 `.vi` 自动修复；找不到则告警 |

## 相关文件

- `csmapp.ini` — 目标，`[csm-module]` 节
- `<项目>.lvcsm` — `[CSMVI.*]` 路径（权威）
- `csm/csm-modules.yaml` — 模块注册表，`path` 索引
- 各模块 `README.md` — 入口 VI 列表（由 yaml `path` 定位）
