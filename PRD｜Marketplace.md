# PRD｜Marketplace（MVP）

> 状态：当前产品基线
>
> 本文是 Marketplace 的唯一当前生效 PRD。它将企业内容分发框架与原 Plugin Marketplace 的市场、详情、供给和 InfCode 串联能力合并为一个产品。
>
> 历史基线：`【WIP】PRD｜Marketplace.md`、`PRD｜企业内容分发平台 Marketplace.md`。

## 1. 产品定位与目标

Marketplace 是企业内容发现、供给、审核、发布、分发、安装、使用、更新、统计和审计的一体化平台。它用一套统一对象和责任边界，连接作者供给、企业治理与成员在 InfCode 中的实际使用。

Plugin 是 Marketplace 支持的内容类型之一，不等同于 Marketplace。MVP 统一承载 Skill、Plugin、Rule、Prompt 和 Template 五种内容，并让它们共享可追溯的版本、发布、分发、安装与数据回流链路。

### 1.1 MVP 目标

1. 成员能找到、理解、收藏可信内容，并在 IDE 中完成真实安装与使用。
2. 作者能创建或上传内容，经过校验、审核和发布后触达指定受众。
3. 审核人能围绕不可变版本做出可追溯决定，系统按冻结配置完成发布与分发。
4. 内容的市场触达、安装、启用、使用、更新和治理数据回流至同一内容记录。

### 1.2 非目标与工程基线

MVP 不实现复杂 RBAC、普通发布链路的多级审批、计费、跨组织交易或独立管理后台；也不在 Marketplace 中重建外部数据源的权限体系。第 12.6 节可选的 break-glass 能力即使启用双人复核，也不扩展普通链路的审批层级。

正式前端必须基于现有 `apps/marketplace-web` 与 `webview` 增量开发，复用既有市场、详情、Release、Installation 和 InfCode 同步链路，不另起一套正式前端或平行数据模型。

## 2. 用户、角色与 MVP 权限

MVP 不预设“作者、发布运维、分发运维”等 Marketplace 业务角色；`Member` 是唯一的普通企业身份，具体责任由当前对象和动作决定。MVP 沿用组织既有授权字段 `access_control_mode`，取值仅为 `Open` 或 `Custom`，不是 Marketplace 新增模式。

`Open` 下，所有 Member 默认拥有普通非审批能力：浏览、收藏、创建、上传、编辑自有内容、配置发布与分发计划、提交审核、发起 IDE 安装、重试发布失败、管理 Distribution / 再次下发，以及查看 Marketplace 数据与审计。`marketplace.publish.retry`、`marketplace.distribution.manage` 和 `marketplace.audit.read` 是能力标识，不代表预设角色；在 `Open` 下对全体 Member 默认有效，仍须校验对象归属、数据可见性和业务 scope，并记录审计。`Custom` 下，企业可按同一组能力点与 scope 自行收紧或分配，本 PRD 不提供角色模板或职责分离编排。

审核权限点固定为 `marketplace.review.decide`，仅显式授予的成员或用户组获得。审核人的有效审核范围是“身份系统授予的组织 / 团队 / 项目 / 内容类型范围”与 `ReviewTask.assignment` 的交集。提交人不得审核自己提交的 `ContentVersion`，此约束不被权限点或用户组成员身份覆盖。

完整审计的读取能力点固定为 `marketplace.audit.read`，但其默认与自定义授权仍按上述 `Open / Custom` 规则，且始终受数据可见性和业务对象 scope 约束。审核通过后，系统依据提交时冻结的版本与 `DistributionPlan` 自动发布，无需作者再次操作。创建、上传、校验、提交、审核决定、发布、分发配置、安装、更新、撤回和版本回退等关键动作均记入审计。

紧急安全控制是可选的平台级 break-glass 能力，默认未启用，不属于 Marketplace 基础 MVP 的角色体系，也不随 `Open` 授予普通 Member。仅当企业显式启用平台安全策略时，才复用组织既有的安全管理与审批主体，将其映射为 `marketplace.emergency.request`、`marketplace.emergency.approve` 和 `marketplace.emergency.lift`，并校验身份 scope 与 `EmergencyControl.scope` 的交集。本 PRD 只定义该能力与 Marketplace 对象的联动，不要求本期新建角色管理、多级审批后台或默认权限。其双人复核是可选能力自身的安全约束，不推导普通发布、分发和安装链路实现复杂 RBAC 或多级审批。

### 2.1 最小词汇与权限

| 词汇 / 范围 | 定义 |
| --- | --- |
| Member | 企业组织身份与业务操作者，可作为作者、提交人或审核人 |
| User | 个人安装环境，是 `InstallationTarget` 取值，不是角色 |
| 市场与自有内容 | 成员可查看对其可见的市场内容和本人内容 |
| 作者数据 | 作者可查看自己内容的触达、安装与使用数据 |
| 审核数据 | 持有 `marketplace.review.decide` 的审核人可查看身份授权范围与任务分配范围交集内的提交、版本快照、校验和审核证据 |
| 完整审计 | `marketplace.audit.read` 按 `Open / Custom` 生效，并始终受数据可见性与业务 scope 约束；不得因 Open 而越过对象边界 |
| 发布重试 | 能力点固定为 `marketplace.publish.retry`；`Open` 下 Member 在可见与业务 scope 内可重试，Content Owner 始终可重试自有内容；`Custom` 可按需收紧，全部操作审计 |
| 分发管理 | 能力点固定为 `marketplace.distribution.manage`；`Open` 下 Member 在可见与业务 scope 内可再次下发或管理 suppression，`Custom` 可按需收紧；受影响成员恢复自己的 suppression 不需要该能力点 |
| 紧急安全控制 | 可选 break-glass 能力；企业启用后才将既有安全主体映射到 request / approve / lift，不随 Open 授予全体 Member，其 scope 与双人复核规则见第 12.6 节 |

### 2.2 责任矩阵

| 责任 | 成员 / 作者 | 审核人 | 系统 |
| --- | --- | --- | --- |
| 发现与安装 | 浏览、收藏、发起 IDE 安装 | 同普通成员 | 校验可见性、兼容性和安装策略 |
| 创建与提交 | 创建、上传、编辑自有内容，配置范围并提交 | 可作为作者执行同类操作 | 校验包，同时冻结版本与 `DistributionPlan`，创建审核任务 |
| 审核决定 | 无审核权 | 在身份授权与任务分配的交集内审核非本人提交的版本 | 校验 `marketplace.review.decide`，记录不可重复的决定和证据 |
| 发布与分发 | 提交前完成发布与分发计划 | 核对冻结的计划 | 通过后创建 `Release`，并从 `DistributionPlan` 生成实际 `Distribution` |
| 发布异常与 suppression | Content Owner 重试同一冻结快照；受影响成员恢复自己的默认安装；其他 Member 按 `Open / Custom` 下的有效能力与 scope 重试或再次下发 | 审核权不产生重试或分发管理权 | 校验 `marketplace.publish.retry` / `marketplace.distribution.manage`、对象所有权与范围交集，并记录审计 |
| 数据与审计 | 查看市场、自有内容及本人内容数据 | 查看授权与分配交集内的提交与证据 | 聚合指标，校验 `marketplace.audit.read` 后按范围提供完整审计 |

## 3. 内容体系

MVP 中的所有内容类型共享 `Content → ContentVersion → Release → Distribution → Installation` 对象链路，但保留各自的包结构、校验门槛与详情字段。

| 类型 | 产品定义 | 独立发布的最小承载 |
| --- | --- | --- |
| Skill | 面向 Agent 的可复用任务能力 | `SKILL.md`；可附带 references、scripts 和 assets |
| Plugin | 将多项能力统一发布、安装和管理的复合安装单元 | Manifest 与组件清单；可组合 Skill、Rule、Prompt、Template、MCP、Hook、Command、Agent 等 |
| Rule | 对 Agent 或工程持续生效的行为与风险约束 | 规则正文、生效范围与优先级 |
| Prompt | 可复用的任务提示模板 | Prompt 正文、变量定义与示例 |
| Template | 文档、项目或工作流的可复用交付骨架 | 模板文件、生成参数与目标位置 |

独立发布的 Skill、Rule、Prompt 和 Template 各自拥有独立的 `Content`、`ContentVersion`、`Release` 和 `Installation`，可被单独发现、安装、更新和撤回。Plugin 内组件默认只属于 Plugin 包，不自动生成独立市场条目；只有经过单独发布的组件才获得独立身份。Plugin 的详细组成、创建门槛和配置边界留在附录 A。

## 4. 双端信息架构与职责边界

MVP 不新增独立企业管理后台。市场发现、作者供给、审核和数据查看收敛在 Marketplace Web；与本地环境相关的安装管理收敛在 InfCode IDE 的 IDE Market。

### 4.1 Marketplace Web

```text
Marketplace Web
├── 市场首页
├── 内容详情
└── 个人中心
    ├── 本地（我的内容）
    ├── 收藏
    ├── 待我审核
    ├── 数据概览
    ├── 内容发布
    └── 审核详情
```

Web 负责安装前的发现、理解、收藏和风险判断，以及内容创建、发布配置、审核和数据查看的入口。“本地（我的内容）”只包含本人创建、导入或上传的内容，不包含从市场安装的内容。Web 可发起受控的 IDE 安装意图并展示跨端状态摘要，不做完整安装管理。

### 4.2 InfCode IDE / IDE Market

```text
InfCode IDE
└── IDE Market
    ├── 待确认安装
    ├── 已安装
    ├── 待配置
    ├── 更新可用
    ├── 组织管理
    └── 异常与版本回退
```

IDE Market 承接目标环境确认及真实的下载、安装、配置、启停、更新、版本回退和卸载，并向 Web 回传状态摘要。IDE 不承担内容发布与人工审核；受组织管理的内容在此展示策略与操作限制。本 PRD 统一使用“版本回退”表达 IDE 安装版本处置；内容发布侧使用“撤回”，分发侧使用“暂停 / 结束”。

## 5. 统一对象与状态

### 5.1 字段归属与市场聚合

| 对象 | 自有字段与责任 |
| --- | --- |
| `Content` | 保存稳定内容身份、内容类型、唯一管理权字段 `owner_member_id`、可选稳定维护团队 `owner_team_id` / `owner_team_name` 和默认标签，不存放某次发布产物或发布状态。MVP 不支持 Owner 转移；创建、上传或导入时 owner / creator / uploader 为同一当前 Member，后续上传替换等来源事实不得改写 `owner_member_id`。维护团队只表达支持团队，不等于 Owner、`publisher` 或 `DistributionScope.Team`。 |
| `ContentVersion` | 冻结包内容、权限声明、兼容性、依赖、`source_ecosystem`、`upstream_owner_id`、`upstream_owner_name`、`supply_mode`、发布说明和 `planned_publisher_id`。`planned_publisher_id` 是当前 `Release` 的计划签发与维护责任主体，随提交冻结。 |
| `Release` | 保存最终 artifact、checksum、`publisher_id`、`publisher_name`、既有时间字段 `released_at`、新增 `publish_sequence`、审核与校验证据，表示可被分发和安装的不可变发行物。新 Release 发布成功时 `publish_sequence` 对同一 `Content` 单调递增；`publisher_id` 必须来自已审核快照的 `planned_publisher_id`。 |

市场聚合信息由 `Content` 的稳定字段与当前受众的最新可见 `Release` 派生，不回写为 `Content` 上的发布事实。

**现有 Schema 就地兼容迁移表**

现有 `ReleaseSchema` 和 `InstallationSchema` 都使用 `additionalProperties: false`，持久化状态也固定 `schema_version=0.1.0`。因此下表中任何新字段、可选性变更或索引变更，都必须先发布新 Schema / API 版本和存储迁移，再开始双写；不得将本 PRD 的规划字段直接写入 0.1.0 对象。

| 现有对象 / 字段 | 现有代码中的含义 | 统一模型映射 | 迁移、回填与移除门槛 |
| --- | --- | --- | --- |
| `Release.plugin_id` | Plugin Manifest 与 Release 的必填一致标识，现有构建、索引和校验都使用它 | 保留为 `content_id` 的 Plugin 兼容别名；旧 Plugin 原值原样映射为 `content_id`。非 Plugin 新记录以 `content_id` 为权威，不伪造 `plugin_id` | 新 Schema 先增加 `content_id` 并将 `plugin_id` 调整为可选兼容字段；Plugin 双写且必须相等，冲突时 fail closed。旧读取适配器仅对 Plugin 投影 `content_id → plugin_id`，非 Plugin 不进入旧 Plugin-only API |
| `Release.channel` | `stable / preview / internal` 的现有发行物成熟度 / 构建通道；当前种子是 `preview` | 作为 Release 技术元数据保留，不映射为 Distribution 可见范围、安装策略或 `update_policy` | 原值保留，不从 Distribution 派生或回填；如未来取消通道，先迁移构建索引与所有读取方 |
| `Release.supply_mode` | `original / mirror / adapter / compose` 的必填发行快照 | 权威值迁入冻结的 `ContentVersion.supply_mode`；兼容期 Release 保留同值快照 | 旧 Release 先回填所属 ContentVersion；双写不一致时 fail closed；全部读取方切换至 ContentVersion 后才可停止 Release 冗余写入 |
| `Release.source_items[]` | 引用现有 SourceItem registry 的 ID 集合，可为空；构建种子前校验引用存在 | 回填为 ContentVersion 的包级 / 组件级 `source_evidence[]` 引用；Compose 保留全部来源 ID，不压成单一 upstream owner | 保留 registry ID 与冻结证据的可追溯映射；完成证据回填、完整性校验和新读取切换前不得删除该引用 |
| `Release.adapter_version` | 所有现有 supply mode 都必填的构建 / 适配管线版本，当前不只用于 `adapter` | 作为不可变 Release 技术证据保留，不从 `supply_mode` 推断，也不改写内容语义 | 旧值原样保留；只有在新构建证据字段完成回填并且校验器已切换后，才可考虑更名或移除 |
| `Release.released_by` 与新增 `publisher_id / publisher_name` | `released_by` 是执行 Release 生成的 actor / service ID，现有数据不含签发责任语义 | `released_by` 保留为执行主体并映射 AuditEvent actor；`publisher_id / publisher_name` 独立来自已审核的 `planned_publisher_id` 及冻结展示名 | 旧 Plugin 可从与 Release checksum 一致的 Manifest `publisher` 回填 publisher，不得从 `released_by` 推断。先回填 AuditEvent 与 publisher，冲突时 fail closed；执行者证据完整迁入审计后才可停止冗余写入 |
| Release 其他必填字段 | `id`、`version`、`artifact_path`、`package_sha256`、`manifest_sha256`、`file_manifest`、`gate_results`、`released_at` 构成现有不可变发行证据 | 原语义保留；新增 `content_version_id`、`publish_sequence` 和审核证据只做 Schema 版本化扩展 | 先按 `plugin_id + version + checksum` 确定性回填版本关联和序号，遇到多义不写入并报错；旧证据字段不因新模型删除 |
| `Installation.plugin_id` | 现有安装的 Plugin 标识，并参与唯一键 | 保留为 `content_id` 的 Plugin 兼容别名；非 Plugin 新安装以 `content_id` 为权威 | 旧 Plugin 原值回填 `content_id`；Plugin 兼容期双写且必须相等，冲突时 fail closed；非 Plugin 不伪造 `plugin_id`，旧 Plugin-only 读取层不返回该类记录 |
| `Installation.scope{type,id,display_name?}` | User / Project 写入环境对象，不是 Distribution 受众 scope | 字段名演进为 `target: InstallationTarget(type,id,display_name?)`，取值与原对象一一对应 | 新 Schema 先允许双写 `scope + target`；读取优先 target，缺失时再适配 scope，两者不一致时 fail closed；不得把 Organization / Team DistributionScope 回填为 InstallationTarget |
| Installation 唯一键 | `account_id + plugin_id + scope.type + scope.id`，对应现有 `installationKey()` 与持久化重复检查 | 演进为 `account_id + content_id + target.type + target.id` | 先回填 content / target 并建新唯一索引，再对 Plugin 双写新旧键；读取先查新键，无命中才查旧键。若两键命中不同 installation_id，或回填后多条记录折叠到同一新键，必须 fail closed、不合并实例并进入人工数据修复 |
| Installation 其他必填 / 规划字段 | `release_id`、`desired_state`、`actual_state`、`configuration_status`、`installed_at/by`、`client_revision` 等是现有安装事实；`effective_distribution_id`、策略快照、`reason_code`、`snapshot_stale` 尚未在 0.1.0 Schema 中 | 现有字段保持原语义；规划字段在新 Schema 中版本化新增，不借用 `actual_state` 或其他字段代表 | 迁移先写新 schema_version，按可验证的当前 Distribution 回填；无法唯一确定时留空 / 标记待对账，不臆测策略快照 |

所有旧 API 字段的移除使用同一门槛：最低支持的 Web、API 与 IDE 版本均已覆盖新字段，完成存量回填与对账，且连续两个正式发布周期没有旧字段读取或写入流量；在达到门槛前保留双写、读取适配、冲突告警与回滚能力。

### 5.2 对象、责任与基数

```text
Content 1 ── N ContentVersion
ContentVersion 1 ── 1 DistributionPlan
ContentVersion 1 ── 0..N ValidationResult
ContentVersion 1 ── 0..1 ReviewTask
ContentVersion（通过版本）1 ── 0..1 Release
Release 1 ── 1..N Distribution
Distribution 1 ── 0..N Installation
Installation 1 ── 0..N UsageEvent
Installation 1 ── 0..N UpdateOperation

Content / Release / DistributionScope 1 ── 0..N EmergencyControl

InstallIntent - - - ACCEPTED ack（installation_id）- - - > Installation

Member ── Favorite ── Content
Actor ── AuditEvent ── 任意业务对象
```

| 对象 | 责任及关系 |
| --- | --- |
| `ValidationResult` | 对指定版本的自动解析、安全、权限和兼容性校验证据；不代替人工决定。 |
| `ReviewTask` | 每个 `ContentVersion` 最多生成一项并绑定版本 checksum，审核包内容、权限、来源声明、`planned_publisher_id` 和冻结的发布计划（`DistributionPlan`），记录 assignment、审核人、决定、意见和证据。任务状态只取 `PENDING`、`DECIDED`、`CANCELLED`。 |
| `DistributionPlan` | 审核前的发布计划快照，包含单调递增的 `plan_version`、非空 `scopes[]`、`policy`、`effective_at`、`update_policy` 和支持 Owner；提交审核时与 `ContentVersion` 一起冻结。 |
| `PublishTask` | 记录通过后的自动发布执行，幂等输入由 `ContentVersion`（含 `planned_publisher_id`）+ `DistributionPlan` + review decision 组成；发布重试不得更换签发方或其他快照字段。 |
| `Distribution` | 审核通过并发布成功后，从 `DistributionPlan.scopes[]` 逐项生成的实际分发记录；每条引用同一 `Release`，并复制 `plan_version`、对应 scope、策略、生效时间、更新策略和支持 Owner。可选 `replaces_distribution_id` 仅允许用于 `SCHEDULED` 替换。 |
| `Installation` | 某个 `Release` 在 User 或 Project 目标中的真实安装实例；必须保存 `effective_distribution_id` 以及当时生效的策略和版本锁定快照。 |
| `UpdateOperation` | 某条 Installation 的检查与更新过程记录，`Installation 1:N UpdateOperation`；保存候选、确认、执行、结果和幂等关系，但 Installation 仍是当前版本、启用与实际状态的唯一事实源。 |
| `EmergencyControl` | 关联 Content 或 Release，并以 DistributionScope 和解析后的 targets 确定命中范围的独立紧急策略控制；生命周期、权限和安装处置见第 12.6 节，不替代 Distribution 或 Installation。 |
| `InstallIntent` | 短期 Web→IDE 跨端交接对象，不进入 `Content → … → Installation` 的持久化业务链，也不可替代 `Installation`。仅在 IDE `ACCEPTED` 后以 `installation_id` 关联真实安装实例；过期后可清理。 |
| `Favorite` | Member 与 `Content` 之间的收藏关系；不代表已安装。 |
| `UsageEvent` | 安装后的启用、执行结果、耗时等最小使用遥测，关联安装实例、内容和版本。 |
| `AuditEvent` | 关键操作与状态变化的不可变证据，记录主体、对象、前后值、原因和时间。 |

### 5.3 内容版本、审核决定与发布任务

| 当前状态 | 合法目标状态 | 条件 |
| --- | --- | --- |
| `DRAFT` | `IN_REVIEW` | 提交审核，同时冻结 `ContentVersion` 与已配置的 `DistributionPlan` |
| `IN_REVIEW` | `CHANGES_REQUESTED` | 审核人要求修改 |
| `IN_REVIEW` | `REJECTED` | 审核人拒绝，当前版本终止 |
| `IN_REVIEW` | `REVIEW_WITHDRAWN` | `ReviewTask.status=PENDING`、尚无决定且尚未创建 `PublishTask` 时由作者撤回 |
| `IN_REVIEW` | `PUBLISHED` | `ReviewTask.status=DECIDED`、`decision=APPROVED` 且系统成功创建 `Release` 与实际 `Distribution` |
| `PUBLISHED` | `WITHDRAWN` | 发布侧撤回，系统同时结束该版本关联的全部 `Distribution` |

`ReviewTask` 的状态转换固定如下：

| 当前状态 | 合法目标状态 | 条件 |
| --- | --- | --- |
| `PENDING` | `DECIDED` | 审核人提交 `APPROVED`、`CHANGES_REQUESTED` 或 `REJECTED` 决定 |
| `PENDING` | `CANCELLED` | 作者合法撤回尚未决定且尚未创建 `PublishTask` 的审核 |

`DECIDED` 与 `CANCELLED` 均为任务终态；只有 `DECIDED` 具有 review decision。合法撤回时，系统原子设置 `ReviewTask.status=CANCELLED`、将冻结版本置为 `REVIEW_WITHDRAWN` 并记录 `AuditEvent`。该版本不可编辑、不可重提；作者如需继续，必须以它为来源创建新的 `ContentVersion=DRAFT`、生成新 checksum 和新的 `ReviewTask`，不得复用旧任务。

`CHANGES_REQUESTED` 后旧版本保持不变，作者通过新建 `DRAFT` 修订；`REVIEW_WITHDRAWN`、`CHANGES_REQUESTED` 与 `REJECTED` 都是当前版本的终态。只有 `PUBLISHED` 可转为 `WITHDRAWN`。

`APPROVED` 是 `ReviewTask` 的决定值，不是稳定的 `ContentVersion` 状态。若审核决定已为 `APPROVED` 但自动发布失败，`ContentVersion` 暂留 `IN_REVIEW`，该 `ReviewTask.status=DECIDED` 且不可重复决定，`PublishTask=PUBLISH_FAILED`。系统只能重试同一幂等输入，包括含 `planned_publisher_id` 的 `ContentVersion`、发布计划（`DistributionPlan`）和 review decision；如需改动内容、签发方或发布计划，必须创建新版本并重新审核。

### 5.4 发布计划与实际分发

**`DistributionPlan` 状态**

| 当前状态 | 合法目标状态 | 条件 |
| --- | --- | --- |
| `UNCONFIGURED` | `CONFIGURED` | 作者已完成非空 `scopes[]`、策略、生效时间、更新策略和支持 Owner 配置 |
| `CONFIGURED` | `FROZEN` | 提交审核，与 `ContentVersion` 同时冻结 |

发布计划（`DistributionPlan`）的 `update_policy` 只取 `FOLLOW_LATEST_COMPLIANT` 或 `PINNED_VERSION`。MVP 选择 `PINNED_VERSION` 时必须满足 `pinned_version=ContentVersion.version`，只能锁定本次提交并获批的新版本；`Release` 创建后，每条实际记录写入 `Distribution.release_id=pinned_release_id=本次新 Release.id`，不得借本次审批锁定历史版本。

启用 `FOLLOW_LATEST_COMPLIANT` 的新排序前，必须先按同一 `Content` 内的 `released_at ASC`、`release_id ASC` 为历史 Release 确定性回填 `publish_sequence`。回填未完成时，全量沿用既有 `released_at DESC`、`release_id ASC` 排序，不得让少数非空 sequence 无条件压过时间更晚的旧记录；回填完成后才统一按 `publish_sequence DESC`、`released_at DESC`、`release_id ASC` 稳定排序取首个合规版本。

`UNCONFIGURED` 属于审核前的发布计划，不属于发布后的实际分发。审核通过且自动发布成功后，一个发布计划按非空 `scopes[]` 生成 `1..N Distribution`：每个 scope 元素恰好生成一条记录，并复制 `policy`、`effective_at`、`update_policy` 和支持 Owner；不得把多个 scope 压成一条无法独立计算受众的记录。

**`Distribution` 状态**

| 初始 / 当前状态 | 合法目标状态 | 条件 |
| --- | --- | --- |
| 初始 | `SCHEDULED` | 计划在未来生效 |
| 初始 | `ACTIVE` | 计划立即生效 |
| `SCHEDULED` | `ACTIVE` / `ENDED` | 到达生效时间，或生效前结束 |
| `ACTIVE` | `PAUSED` / `ENDED` | 暂停当前分发，或结束分发 |
| `PAUSED` | `ACTIVE` / `ENDED` | 恢复分发，或结束分发 |

带 `replaces_distribution_id` 的新 `SCHEDULED` 可在创建时与其将替换的 `ACTIVE` 暂时共存，但 `SCHEDULED` 不参与第 5.5 节候选计算。到达 `effective_at` 时，系统先校验新旧记录属于同一 content、受众和策略层级，且替换引用有效；校验通过后在同一事务中依次执行旧 `Distribution → ENDED`、新 `SCHEDULED → ACTIVE`、重算有效策略与 `Installation` 管理锁，最后才按新 `plan_version` 清除或迁移 `DEFAULT` suppression。只有整个事务成功才提交。

激活或重算失败时，新记录保持 `SCHEDULED`，写入 `activation_failure_reason`，发送告警并记录 `AuditEvent`；旧 `ACTIVE` 与 suppression 均保持不变。没有 `replaces_distribution_id` 时按普通激活处理，但仍须先执行第 5.5 节重叠冲突校验，不得绕过 fail closed。

### 5.5 范围、有效策略与安装目标

- `DistributionScope` 决定内容对谁可见或下发，MVP 支持 Organization、Team、Project 和 Member。
- `InstallationTarget` 决定内容写入哪个环境并生效，MVP 支持 User 和 Project。

团队受众表示可见性或下发范围，不是本地写入位置。例如，对 Team 可见的内容，仍由成员在 IDE 中选择安装到 User 或某个 Project。

有效 `Distribution` 的候选集只包含同时满足以下条件的记录：状态为 `ACTIVE`；关联 `ContentVersion=PUBLISHED`，且已存在关联该版本的 `Release`；受众命中当前成员或安装目标；与当前 IDE 及目标环境兼容；且关联版本未撤回。`Release` 没有独立状态机，可安装性由 ContentVersion、Distribution、兼容性与安全策略联合计算。

在候选集中，先按安装策略 `FORCED > DEFAULT > OPTIONAL` 选择；策略相同时，再按范围精确度 `Member > Project > Team > Organization` 选择唯一有效 `Distribution`。MVP 在写入或激活前，禁止同 content、同受众、同策略、同范围精确度但指向不同 release 的重叠配置。若检测到历史异常，系统必须 fail closed，不创建 `Installation`，并记录分发冲突 `AuditEvent`，不使用任意 tie-breaker。`Installation` 必须保留最终计算结果的 ID 和快照，避免后续范围变化改写历史事实。

### 5.6 安装状态

| 当前状态 | 合法目标状态 | 条件 |
| --- | --- | --- |
| `requested` | `installing` / `uninstalling` / `error` | 开始安装、取消并进入卸载流程，或请求失败 |
| `installing` | `configuration_required` / `ready` / `error` | 需补充配置、已可用，或安装失败 |
| `configuration_required` | `ready` / `error` / `uninstalling` | 配置完成、配置失败，或取消安装 |
| `ready` | `disabled` / `installing` / `uninstalling` / `error` / `unknown` | 禁用、更新安装、卸载、运行异常，或状态暂时不可确认 |
| `disabled` | `ready` / `uninstalling` / `error` / `unknown` | 重新启用、卸载、运行异常，或状态暂时不可确认 |
| `error` | `installing` / `uninstalling` / `unknown` | 重试安装、清理失败实例，或状态暂时不可确认 |
| `uninstalling` | `removed` / `error` | 卸载及清理完成，或卸载失败 |
| `removed` | `requested` | 对已移除内容发起重新安装 |
| `unknown` | `installing` / `ready` / `disabled` / `uninstalling` / `error` | 从同步不确定恢复到 IDE 新确认的实际状态 |

上表与 `packages/marketplace-domain/src/installation-state.ts` 及 `InstallationSchema.actual_state` 一致。`ready → installing` 用于安装更新，`removed → requested` 用于重新安装。`unknown` 是可持久化的 `actual_state`，表示 IDE 与 Marketplace 暂时无法确认安装实态；界面必须显示“状态待同步”，不得渲染为“未安装”。同步恢复后只能按运行时允许的 `unknown` 转换进入新确认状态。

## 6. 内容供给与来源治理

Marketplace 用三组唯一规范字段分别表达生态类别、具体上游与当前发行责任，再以 `supply_mode` 表达平台如何接入。这些字段不得与受众范围或安装目标混用。

### 6.1 来源生态 `source_ecosystem`

`source_kind` 与 `source_ecosystem` 都是随 `ContentVersion` 冻结的来源事实。`source_kind` 描述包级来源形态，只取 `NATIVE`（企业或平台直接创作）、`MIRRORED`（固定一个主要外部上游）或 `COMPOSED`（组合多项来源或组件）；它不替代第 6.4 节描述接入责任的 `supply_mode`。`source_ecosystem` 描述单项来源所属生态，只取以下枚举：

| 取值 | 定义 |
| --- | --- |
| `EnterpriseOfficial` | 原始内容由当前企业生产 |
| InfCode | 原始内容由 InfCode 生产 |
| Claude | 原始内容来自 Claude 生态中的可追溯主体 |
| `ThirdParty` | 原始内容来自其他可验证个人、厂商、合作伙伴或开源项目 |

Team 等组织单元属于 `DistributionScope`，不进入 `source_ecosystem`。

目录、搜索和筛选统一使用派生字段 `source_facets`，它不是新的来源事实：普通内容的 facets 为非空 `source_ecosystem` 的单元素集合；`source_kind=COMPOSED` 时，facets 为非空包级 `source_ecosystem`（若存在）与全部 `components[].source_evidence[].source_ecosystem` 的去重集合。索引可以物化该集合，但必须能从冻结版本证据重建，不能反向改写 `source_kind`、`source_ecosystem` 或组件证据。`publisher_id` / `publisher_name` 始终是独立且唯一的发行责任维度，不进入 `source_facets`。

### 6.2 具体上游 `upstream_owner_id` 与 `upstream_owner_name`

`upstream_owner_id` 是具体上游作者或组织的稳定标识，`upstream_owner_name` 是当次版本冻结的展示名。Compose 如有多个上游，对 Plugin 应在 `components[].source_evidence[]` 中逐项记录这两个字段，其他复合类型采用等价的逐组件证据；不得将多个主体拼成包级名称或任意选择一个充当主要上游。

### 6.3 发行责任 `publisher_id` 与 `publisher_name`

`publisher_id` 是 Marketplace 中当前 `Release` 签发和维护责任主体的稳定标识，`publisher_name` 是发布时冻结的展示名。每个 `Release` 必须有且只有一个 `publisher_id`，并由其对版本、校验、更新与撤回负责。只有未来出现独立于发行责任主体的数字签名身份时，才新增 `issuer`，MVP 不设置该字段。

市场可同时展示“来源”和“签发方”：来源生态展示与筛选来自 `source_facets`，具体来源来自相应包级或组件级 upstream owner；签发方来自最新可见 `Release.publisher_id` 和 `Release.publisher_name`。例如，Claude 生态中的 Anthropic 内容由 InfCode 完成 Mirror 并发布时：`source_kind=MIRRORED`、`source_ecosystem=Claude`、`upstream_owner_id=anthropic`、`upstream_owner_name=Anthropic`、`publisher_id=infcode`、`publisher_name=InfCode`。

### 6.4 供给方式 `supply_mode` 与组合约束

| 方式 | 定义 | 维度约束与典型组合 |
| --- | --- | --- |
| Original | 由企业、InfCode 或受信主体创建并发行完整内容 | 上游生产者与签发方必须一致，签发方对内容、评测、版本和更新负全责 |
| Mirror | 在许可允许范围内镜像外部上游固定版本并完成适配 | 上游必须是外部可追溯主体，当前签发方是 InfCode 或企业官方，并承担固定版本、署名、扫描与兼容验证 |
| Adapter | 不复制限制内容，只发行转换规则、适配代码和来源锁定信息 | 必须记录用户自行获取原始内容的上游来源，并单独记录适配器的签发方 |
| Compose | 将多项官方连接能力、内容与 InfCode 工作流组合为一个发行物 | 可有多个上游来源，但必须有且只有一个对组合后 `Release` 负责的签发方 |

三个维度分开存储，但并非任意组合；每种 `supply_mode` 都必须满足上表的上游可追溯性、签发责任和授权条件，否则不得生成正式 `Release`。

### 6.5 选品与正式发布门槛

候选内容使用同一选品漏斗：

1. **需求强度**：是否覆盖大量用户重复发生的任务。
2. **任务闭环**：安装后是否能以明确入口完成稳定结果。
3. **来源与维护**：是否具备可信发行者、合法许可和持续维护人。
4. **组合价值**：是否能将能力、约束、连接或模板组合成更完整的任务价值。
5. **风险与验证成本**：权限是否可拆分、安装失败是否可恢复、版本是否可回退、行为是否可稳定评测。

选品决定“选不选”，供给方式决定“怎么接”。Mirror-first 是成熟开源内容的默认供给方式，不是选品方法，也不是批量镜像。所有候选先通过选品漏斗，再根据授权、结构和交付边界选择 Original、Mirror、Adapter 或 Compose。

正式 `Release` 的证据分为通用必填与条件必填：

| 证据层级 | 适用范围 | 必须固定的内容 |
| --- | --- | --- |
| 通用必填 | 所有 `Release` | content/version 标识、最终 artifact checksum、file manifest checksum、`publisher_id`、`publisher_name`、发布时间、校验结果、审核结果和兼容性 |
| Git 上游条件必填 | 有 Git 上游的 Mirror、Adapter 或 Compose | `upstream_owner_id`、`upstream_owner_name`、repository URL、tag/ref/commit、原始路径、许可和第三方声明 |
| 非 Git 外部上游条件必填 | 有非 Git 外部上游的 Mirror、Adapter 或 Compose | `upstream_owner_id`、`upstream_owner_name`、canonical URL 或资源 ID、可验证版本号、内容摘要或不可变 digest（按来源可得字段记录），以及适用的许可和第三方声明 |
| 无外部上游 | Original | 不要求上游仓库或外部资源字段，不得伪造 URL、tag、ref、commit 或外部资源 ID |

Original 无外部上游时只记录真实存在的内部来源证据和通用证据。用户安装的是经验证的不可变发行物，不是上游仓库或外部资源的实时副本；同一版本的文件、权限和组件清单不得静默变化。

## 7. 市场发现与推荐

本章定义 Marketplace Web 的市场首页、目录、推荐、搜索、筛选、列表状态与进入详情的体验。正式实现以 `apps/marketplace-web/src/pages/MarketplaceHome.tsx`、`apps/marketplace-web/src/components/PluginCard.tsx`、`apps/marketplace-web/src/pages/PluginDetail.tsx` 为增量基础；企业 Demo 仅作需求参考，不复制为另一套市场前端或数据模型。

### 7.1 首页与目录

首页保留现有的 Hero、推荐区和市场列表：Hero 说明 Marketplace 可发现的企业内容与可信来源；推荐区提供少量可直接进入详情的内容；市场列表承载完整目录、搜索、筛选和内容卡片。推荐区不因当前目录搜索或筛选而被清空；目录的结果数量、筛选条件和状态只描述当前列表。

`Header.tsx` 中现有的“添加”入口继续保留为快捷入口，可进入创建或上传；两种入口最终都进入同一个“我的内容 / 内容发布”流程，复用草稿、校验、分发配置、提交审核和发布链路，不形成第二套供给流程。用户画像由 InfCode 用户系统维护，Marketplace Web 只读取经授权的行业、角色和任务等主动画像，不在 Web 建立独立画像事实源。

目录统一展示 All、Skill、Plugin、Rule、Prompt、Template 六种内容类型。All 表示不限制类型；其余类型是可独立发布的市场内容。目录和卡片还必须支持展示或筛选以下受众可见的聚合信息：

| 维度 | 市场语义 |
| --- | --- |
| 内容类型 | All、Skill、Plugin、Rule、Prompt、Template |
| 来源生态 | 目录派生字段 `source_facets`：由冻结版本的包级与组件级来源计算，可包含 EnterpriseOfficial、InfCode、Claude、ThirdParty 中一项或多项 |
| 上游来源 | 可追溯的具体上游来源主体 |
| 签发方 | 筛选规范字段为最新可见 `Release.publisher_id` / `Release.publisher_name`；与来源 `source_facets` 分开筛选和展示 |
| 发布团队（可选展示） | `Content.owner_team_id` / `Content.owner_team_name`，只表示维护与支持团队；不能替代签发方，也不等同于 `DistributionScope.Team` |
| 分发范围 | `DistributionScope`：Organization、Team、Project、Member；仅作为可见性和下发范围，不等同于 IDE 安装位置 |
| 兼容状态 | 目录静态兼容、Web 已授权环境预检或待 IDE 最终确认的摘要 |
| 安装策略 | `FORCED`、`DEFAULT`、`OPTIONAL`，用于识别组织强制、默认下发或成员主动安装 |
| 任务属性 | 行业、任务及其他稳定目录标签 |

### 7.2 推荐与排序

推荐先按当前成员可见性与安全门槛确定候选，再按以下优先级排序：

1. 企业强制或默认分发策略；
2. 可见性、兼容性与安全状态；
3. 管理员精选和团队适用性；
4. 成员主动维护并由 InfCode 用户系统提供的画像；
5. 组织内近期采用趋势。

有主动画像时可称“为你推荐”，并说明使用的是哪些可见的主动维度；无画像时称“通用推荐”或“热门推荐”，不得暗示存在个人画像。推荐不使用消费型兴趣流，不基于敏感项目内容推断画像，也不得以下载量替代内容质量、审核或安全结论。

兼容性按三层表达：

1. **目录静态兼容**：基于客户端最低版本、平台、静态依赖等目录事实；
2. **Web 已授权环境快照预检**：只有 Web 已取得目标环境上下文时才使用该快照提示风险；
3. **IDE 目标 User / Project 最终校验**：IDE 在具体安装目标上执行的权威结论。

就兼容性而言，推荐只能因全局撤回或明确的静态不兼容排除内容。没有权威目标环境时，目录显示“待 IDE 确认”，不得隐藏仅对某个 Project 不兼容的内容；Web 预检也不得替代 IDE 最终校验。

### 7.3 搜索、筛选与可恢复导航

搜索在当前可见目录结果中匹配名称、简介、类型、`source_facets` 中的来源生态、上游来源、签发方、发布团队、行业、任务和稳定标签。筛选采用“同层多选并集、跨层交集”：例如，同一来源层选中多个生态时，只要内容的 `source_facets` 与所选集合有交集即命中；内容类型、行业、任务、来源生态、签发方、分发范围、兼容状态和安装策略等不同层之间必须同时满足。All 不增加类型限制。发布团队只作展示和支持信息，不是签发方筛选的替代字段。

当前搜索词和全部筛选条件必须写入 URL，首屏与硬刷新都从 URL 恢复状态。沿用现有市场实现，连续输入搜索词或调整筛选时 replace 当前市场历史项；进入详情时 push 新历史项；浏览器前进、后退和详情返回按历史项恢复同一结果、筛选、搜索词与滚动位置。用户从卡片主体进入详情后返回，仍回到原列表上下文及此前滚动位置。卡片主体与主按钮是独立交互目标：点击主体进入详情，点击主按钮只执行该内容的当前主动作，不应意外导航到详情。

加载时展示明确的加载状态；无结果时保留搜索与筛选条件，说明是“无匹配结果”而非目录为空，并允许清空搜索或重置筛选。目录或搜索请求失败时，必须保留当前 URL 条件和最后成功的旧结果，同时提供重试；不得因失败静默清空条件或把旧结果伪装成最新结果。

### 7.4 内容卡片与跨端状态摘要

每张内容卡片至少展示：名称、内容类型、简介、由 `source_facets` 得出的一个或多个来源生态、签发方、可选发布团队、当前受众可见的最新版本、审核 / 扫描摘要、兼容性、安装数、收藏数和近 30 天活跃度。来源、签发方、发布团队和供给方式必须分开表达，避免将 `source_facets`、具体上游、`publisher`、`Content.owner_team` 或 `supply_mode` 混为同一字段。

卡片与详情必须按以下四个正交维度渲染状态，不能把任一维度压成“已安装”或“未安装”单一结论：

| 维度 | 事实来源与展示规则 |
| --- | --- |
| `InstallIntent` | 瞬态 Web→IDE 交接状态；`GENERATED` / `OPENED` 时显示“等待 IDE 确认”独立徽标。既有 `Installation` 始终可由统一 API 正常读取；ack 前不得把 intent 推断或伪装成新增 `Installation` |
| `Installation.actual_state` | 统一 Installation API 返回的真实安装实例状态；同一内容有多个 User / Project 实例时展示各状态计数 |
| 更新状态 | 是否可更新是独立徽标和次操作，不覆盖安装实态或主 CTA |
| 组织策略 | 强制、默认、受组织管理等是独立徽标和次操作，不覆盖安装实态或主 CTA |

| `Installation.actual_state` | 卡片 / 详情摘要与主 CTA |
| --- | --- |
| `requested`、`installing` | 等待安装；前往 IDE 查看 |
| `configuration_required` | 待配置；完成配置 |
| `ready` | 已可用；去使用，存在多个 ready 范围时先选择范围 |
| `disabled` | 已禁用；前往 IDE 启用 |
| `error`、`unknown` | 异常或状态待同步；前往 IDE 排障。`unknown` 不得显示为未安装 |
| `uninstalling` | 正在卸载；前往 IDE 查看 |
| `removed` | 无活跃安装，但保留安装历史；仅所有实例均为 `removed` 或不存在时才可视为无活跃安装 |

主 CTA 先基于统一 Installation API 返回的既有 User / Project 实例计算，`InstallIntent` 不覆盖该结果。任一 `ready` 优先“去使用”；否则 `configuration_required` 为“完成配置”；否则存在 `requested`、`installing` 或 `uninstalling` 时“前往 IDE 查看”；否则 `error` 或 `unknown` 时“前往 IDE 排障”；否则 `disabled` 时“前往 IDE 启用”。`removed` 等同无活跃安装但保留历史；只有不存在其他 Installation 实例时，才可进入无活跃安装分支。此时若存在有效的 `GENERATED` / `OPENED` intent，主 CTA 为“等待 IDE 确认”，并提供“重新唤起”次操作且复用同一 `idempotency_key`，不得再次创建意图；只有无活跃 Installation 且无有效 pending intent 时才显示“在 InfCode 中安装”。可更新与组织管理只作为额外徽标和次操作。多个实例不能压缩为单一状态；部分失败必须显示“X个可用、Y个异常”。

Marketplace Web 无完整已安装列表，不提供完整“已安装 N”列表，也不在 Web 实现启停、更新、卸载、安装范围管理或版本回退；这些实际环境操作统一进入 IDE Market。

## 8. 内容详情

本章定义所有内容类型共用的详情壳、安装前风险判断和 Web 到 IDE 的受控交接。详情页以 `PluginDetail.tsx` 为实现基础扩展为通用内容详情；返回市场时必须保留第 7 章定义的搜索、筛选和滚动上下文。

### 8.1 通用详情结构

详情页按以下顺序展示；只隐藏实际不适用的可选内容类型区块，不以空壳或“无”区块填充页面：

1. 基本信息与主操作：名称、类型、简介、来源生态、上游来源、签发方、可选发布团队、最新可见版本、收藏与当前安装状态摘要；
2. 能力说明、适用场景与输入 / 输出：说明解决的问题、前置条件、可编辑示例输入和预期结果；
3. 内容组成：面向该类型的实际文件、组件或生成物；
4. 安全与权限：权限声明、同意时机、扫描 / 校验摘要、风险与阻断原因；
5. 兼容性与依赖：InfCode 版本、适用的目标环境、外部连接、必需依赖及待配置条件；
6. 版本与审核证据：最新可见 `Release`、不可变 artifact 与 manifest checksum、审核结果、校验结果、来源锁定和许可；
7. 分发原因与组织策略：为何对当前成员可见、命中的 `DistributionScope`、有效安装策略、组织限制及可操作范围；
8. 企业使用数据：安装、收藏、近 30 天活跃、采用趋势和质量信号；数据须按当前成员的授权范围聚合；
9. Owner、支持与审计入口：维护责任主体、支持渠道，以及受 `marketplace.audit.read` 约束的审计入口。

权限、扫描、审核和其他信任区块不得因数据为空而静默隐藏，必须分别明确显示“已验证无额外权限”、数据缺失或尚未完成；扫描和审核的通过、失败、缺失与未完成也必须与实际证据一致。

Plugin 详情保留现有 Plugin 专属区块，并增量补充不可变 Release 证据。完整的 Plugin 专属组成、组件展示与校验规则统一引用附录 A，本章不重复展开。

### 8.2 主操作与 Web → IDE 安装

未安装且可安装内容的主按钮固定为“在 InfCode 中安装”。Marketplace Web 只做安装前的风险预检、权限与兼容性摘要和组织策略说明；它不创建 Installation。点击后，Web 生成或唤起受控的瞬态 `InstallIntent`，不将它持久化为 `Installation`。

`InstallIntent` 是产品闭环中的短期交接记录，字段必须完整包含 `intent_id` / `request_id`、`content_id`、`release_id`、`source_page`、`permission_snapshot_hash`、可选 `target_hint`、`expires_at` 和 `idempotency_key`。其状态机为 `GENERATED → OPENED → ACCEPTED`，或进入 `EXPIRED` / `FAILED`。IDE 返回 `ACCEPTED` ack 时必须附带 `installation_id`；在此之前 Web 显示“等待 IDE 确认”，但仍可从统一 Installation API 正常读取同一内容的既有实例。ack 前禁止的是把该 intent 推断、创建或伪装成新增 Installation；ack 后才以返回的 `installation_id` 关联真实安装实例。重试未过期意图必须复用同一 `idempotency_key`；意图过期后才新建意图。附录 B 可重复该协议的传输规范，本章只定义产品闭环和必填字段。

IDE 收到意图后必须重新校验登录身份、目标 User / Project 环境、兼容性、权限与组织策略；即 IDE 重新校验完成前不得创建安装。校验通过后才创建唯一的 `Installation`，并执行下载、安装、默认启用或待配置处理。安装范围管理、启停、配置、更新、版本回退和卸载均在 IDE Market 中完成，Web 仅展示同步状态摘要和“前往 IDE”入口。

Web 唤起 IDE 失败、意图过期或 IDE 未确认承接时，不得显示“安装成功”。页面必须保留详情上下文，明确说明尚未创建或确认安装，并提供重试唤起与复制操作信息的方式。

### 8.3 已安装内容的使用与状态

已安装并可用的内容主操作是“去使用”：它只唤起 InfCode 会话并预填一条可编辑 query，不自动发送、不自动执行。等待 IDE、待配置、可更新、组织管理或状态待同步时，详情页显示对应摘要和进入 IDE Market 的操作，不在 Web 侧修改真实安装状态。

### 8.4 详情加载、恢复与不可用状态

| 详情状态 | 页面与操作规则 |
| --- | --- |
| 加载中 | 展示加载状态；在尚未取得可操作安装事实前，不展示可执行的安装 CTA |
| 加载失败 | 保留面包屑、名称等已知上下文与市场 URL 条件，提供重试或返回市场 |
| 404、撤回或下架 | 保留已知名称和来源；无缓存时展示 content ID 与不可用原因；禁用安装，提供替代内容或返回市场 |
| 缓存陈旧 | 标记“状态待同步”；发布状态未知时禁用安装，直到取得权威状态 |
| IDE 唤起失败 | 不显示安装成功；保留详情与返回上下文，提供重试唤起和复制操作信息 |

无论从详情返回、重试加载还是从不可用状态返回，均按第 7.3 节的 URL 历史项恢复搜索、筛选和滚动位置。

## 9. 个人中心

个人中心通过现有 Marketplace Web Header 的用户入口进入，使用市场内部二级导航，不成为市场一级模块。正式实现必须增量复用现有 Header、内容卡片、详情、表格和状态样式，不新增平行前端、独立个人站点或第二套内容事实源。

### 9.1 入口与导航

个人中心二级导航固定为以下四项，并按此顺序展示：

| 二级入口 | 可见范围 | 主要用途 |
| --- | --- | --- |
| 本地（我的内容） | 当前 Member | 管理本人供给的内容、草稿、待处置版本与后续版本 |
| 收藏 | 当前 Member | 管理账号收藏的市场内容 |
| 待我审核 | 同时持有 `marketplace.review.decide` 且任务范围匹配的 Member | 处理分配范围内仍为 `PENDING` 的审核任务 |
| 数据概览 | 当前 Member | 查看本人内容的数据摘要；具体指标、口径和审计权限见第 13 章 |

用户从任一入口进入内容发布、审核详情或市场详情后，返回时恢复原入口、筛选和列表位置。无审核权限时隐藏“待我审核”；通过直达链接访问时返回明确的“无审核权限”或“任务不在授权范围”，不展示空白页面。其他入口加载失败、无数据或内容不可用时同样给出可恢复状态与下一步操作。

### 9.2 本地（我的内容）

本地只包含 `owner_member_id` 为当前 Member 的 `Content`；creator、uploader 和导入人仅作为来源展示，不参与管理权判断。MVP 创建、上传或导入内容时，owner / creator / uploader 都记录为同一当前 Member，且不支持 Owner 转移；后续上传替换、重新导入等只更新来源事实，不改变 `owner_member_id`。MVP 不新增 `ContentCollaborator`，也不因团队、审核或市场可见关系把内容加入本地。这里绝不包含从市场安装的他人内容；后者的 `Installation` 只在 IDE Market 管理。

每行必须分开呈现三个版本事实，不能合成一个“当前版本”：

| 字段 | 展示规则 |
| --- | --- |
| 名称、类型 | 使用 `Content` 的稳定身份与内容类型 |
| 活动草稿 | 当前可编辑的 `ContentVersion=DRAFT`；没有则显示“无活动草稿” |
| 当前待处置版本 | 当前 `IN_REVIEW`、`CHANGES_REQUESTED`、`REVIEW_WITHDRAWN` 或对应 `PUBLISH_FAILED` 的冻结版本与处置原因 |
| 最新可见 Release | 当前受众可见的最新 `Release`；没有则明确未发布 |
| 位置或来源 | 展示本地位置、上传来源、仓库来源或在线创建来源中的真实值 |
| 市场状态、审核状态、修改时间 | 分别展示市场生命周期、ReviewTask / PublishTask 摘要和最近修改时间 |

行级主动作按以下优先级唯一确定：`PUBLISH_FAILED` 时，当前 Member 命中 `owner_member_id` 或第 2 章定义的 `marketplace.publish.retry` 有效授权才显示“重试发布”，否则只读“查看失败详情” → `CHANGES_REQUESTED` 时“创建修订草稿” → `IN_REVIEW` 时“查看审核” → 存在 `DRAFT` 时“继续编辑” → 最新版本已 `PUBLISHED` 时“创建新版本”。`REVIEW_WITHDRAWN` 提供“基于撤回版本新建草稿”；所有新草稿都生成新版本和新 checksum，不覆盖冻结版本。次操作可进入市场详情或版本历史；没有适用动作时显示原因，不渲染无效按钮。

### 9.3 收藏

收藏是 `Member → Favorite → Content` 关系，收藏不等于安装，也不创建 `InstallIntent`、`Installation` 或本地内容。收藏页可查看内容详情、在 InfCode 中安装和取消收藏；安装、更新、卸载及安装失败都不自动新增或取消收藏，取消收藏也不改变任何既有安装实例。

收藏内容被撤回、对成员不可见或暂无合规 `Release` 时，保留可识别的收藏记录与不可用原因，禁用安装并允许取消收藏，不把记录静默删除。

### 9.4 待办拆分与优先级

“待我审核”只列出当前 Member 同时满足 `marketplace.review.decide`、身份授权范围、`ReviewTask.assignment` 范围且 `ReviewTask.status=PENDING` 的任务；其中高风险待审优先，再按等待时长和提交时间排序。被领取处理的任务仍是 `PENDING`，只增加处理人和领取时间，不引入新任务状态。

`CHANGES_REQUESTED` 进入作者在本地（我的内容）中的“待修订”，不再占用审核人的待办。`PublishTask=PUBLISH_FAILED` 进入“发布异常 / 已处理”，审核人只能只读查看原决定与失败证据，不得再次决策，也不得把任务放回待我审核；系统自动重试与人工重试规则见第 10.7 节。`DECIDED` 和 `CANCELLED` 的 ReviewTask 进入已处理历史。

数据概览入口在 MVP 中保留，但本章只定义入口和本人内容范围，不预设指标、统计窗口或全量审计可见性；这些规则统一由第 13 章定义。

## 10. 发布与审核

发布与审核在同一条 `Content → ContentVersion → ReviewTask → Release → Distribution` 链路上完成。页面动作沿用第 5 章的对象、状态与幂等发布约束，不另设“提交后手工上架”、第二次发布确认或平行审核状态机。

### 10.1 创建入口、上传兼容协议与路由

现有 Header“添加”中的创建与上传快捷入口都进入同一个个人中心发布流程。现有 `UploadDialog` 的文件读取、`.json` 格式与 5 MB 大小限制、上传请求，以及服务端解析和校验必须保留，不实现第二套上传组件或草稿模型。

Plugin submission 协议按兼容方式演进：

| 协议部分 | 要求 |
| --- | --- |
| 请求 | 在现有 `file_name`、`content_base64` 上增加可选 `request_id`；客户端提供时以已认证 `account_id + request_id` 为幂等键，旧客户端缺失时由服务端生成并在响应返回 |
| 标识 | 兼容期保留旧 `id`，并新增同值别名 `submission_id` |
| 兼容响应 | 兼容期保留 `plugin_id`、`version`，增量返回 `content_id`、`content_version_id`、`validation_summary` |
| 重放 | 相同账号以相同 `request_id` 重放时返回同一 `id` / `submission_id`、同一草稿与同一组 Content / Version ID，不重复解析或创建 |

服务端生成 `request_id` 只保证该次调用内部唯一；旧客户端若在没有复用响应 `request_id` 的情况下重新发起请求，服务端不能识别为重放。客户端重试若需幂等，必须保存并回传服务端返回值，或升级为首次请求即发送 `request_id`。

上传成功后，Web 使用 `content_id` 与 `content_version_id` 打开在现有 React 应用内增量实现的统一发布路由，不以 Toast 为流程终点。若路由跳转失败，服务端已创建的 `ContentVersion=DRAFT` 仍持久存在，本地（我的内容）显示“继续编辑”；重试导航不得重传包或新建草稿。

Web 在线创建 Skill 直接创建 `Content + ContentVersion=DRAFT` 并进入同一发布路由。现有 IDE `/create-plugin` 继续负责 Plugin 创建；完成后通过同一 submission API 回传上述 submission、Content 和 Version ID，并打开同一 Web 发布页。两种创建方式不建立第二套草稿、校验或发布流程。仓库导入未接入时明确标记不可用，不以演示数据或成功 Toast 伪装已导入。

### 10.2 内容发布页

内容发布页主流程只保留提交审核所需内容：

| 区块 | MVP 内容 |
| --- | --- |
| 提交必需信息 | 类型、名称、唯一标识、简介、Owner、来源声明、标签和 `planned_publisher_id` |
| 包 / 能力 | 文件树、入口文件、能力说明、输入输出和运行示例 |
| 权限依赖 | 工具、MCP / 数据源、网络、文件、写操作、脚本与依赖内容 |
| 兼容性 | InfCode 最低版本、平台、运行环境和目标环境要求 |
| 版本 / 说明 | 版本号、变更摘要和升级注意事项 |
| 自动校验 | 结果、证据、阻断级别、原始日志入口及与上一版本的差异 |
| 发布计划（`DistributionPlan`） | `scopes[]`、`policy`、`effective_at`、`update_policy`、条件必填的 `pinned_version` 和支持 Owner |
| 动作与状态 | 保存草稿、提交审核、合法撤回、按意见新建修订版、查看只读审核结果或重试发布 |

版本历史、市场发行摘要、安装覆盖、数据和审计不在发布页重复实现；页面只提供折叠侧栏链接，分别复用现有详情、版本视图、数据概览和受权限控制的审计入口。不同内容类型只扩展自身包结构与校验项，不复制整页；Plugin 专属要求引用附录 A。

### 10.3 草稿、提交与撤回

- **保存草稿**：允许必填信息、非阻断校验或异步校验尚未完成；必须保存当前结果和未完成原因。已发现的阻断问题可保存在草稿中，但不可提交审核。
- **提交审核**：要求必填项完整、发布计划（`DistributionPlan`）达到 `CONFIGURED` 且所有阻断校验通过；系统冻结 `ContentVersion`（包含 `planned_publisher_id`）和发布计划，生成 artifact 与 file manifest checksum，并按内容类型规则为该版本生成至多一个（`0..1`）`ReviewTask`。生成任务时状态为 `PENDING`；暂未匹配审核人时保留 PENDING 任务和冻结快照并明确提示待分配，不能静默跳过审核。
- **撤回审核**：只有 `ReviewTask.status=PENDING`、尚无 review decision 且尚未创建 `PublishTask` 时允许。系统原子设置 `ReviewTask.status=CANCELLED`、将版本置为 `REVIEW_WITHDRAWN` 并记录 `AuditEvent`；冻结版本不可编辑或重提。
- **撤回后继续**：作者必须基于 `REVIEW_WITHDRAWN` 版本创建新的 `ContentVersion=DRAFT`，生成新 checksum，并在再次提交时创建新的 `ReviewTask`；不得复用已取消任务。
- **按意见修订**：收到“需修改”后必须创建新的 `ContentVersion=DRAFT`，继承可复用信息并保留原冻结版本、意见与验收标准；禁止直接编辑旧版本。
- **发布衔接**：作者提交后不再执行二次“发布”按钮。只有审核“通过并发布”且自动发布成功，版本才按第 5 章进入 `PUBLISHED`。

### 10.4 自动校验

自动校验是绑定指定 `ContentVersion` checksum 的机器证据，不是人工审核，也不替代审核人的责任判断。每项结果必须标明通过、警告、阻断或未完成，并保留可追溯证据。MVP 至少覆盖：

1. 包 / 目录结构、必需文件和文件清单完整性；
2. manifest 或 frontmatter 的语法、字段与内容类型约束；
3. Content ID、版本号、依赖 ID 与重复 / 冲突检查；
4. 恶意内容、秘密泄露、可执行脚本与危险行为扫描；
5. 权限声明与实际文件 / 能力的一致性，包含未声明权限；
6. InfCode、平台、运行环境与依赖兼容性；
7. 相对上一版本的文件、metadata、依赖、兼容性和权限差异，新增权限单独突出。

校验规则升级后不得静默改写历史决定；重新提交必须对新的不可变版本生成新证据。

### 10.5 审核页

审核页在同一视图完成队列切换、内容理解、风险判断、版本对比和决定：

1. 左侧“待审 / 处理中”只包含 `ReviewTask.status=PENDING` 的任务；“已处理”包含 `DECIDED`、`CANCELLED` 和只读的“发布异常”记录，展示类型、提交人、等待时长、风险级别与处理结果；
2. 中间主体展示基本信息、README 或 `SKILL.md` 渲染、文件树与运行示例；
3. 差异区展示新增、删除、修改的文件、metadata、依赖、兼容性、发布说明和新增权限；
4. 校验证据区展示每项结果、原始证据、阻断级别与日志；
5. 权限风险区突出外部连接、敏感数据、写操作、脚本执行和权限升级；
6. 责任与发布区完整展示提交人、Owner、`planned_publisher_id` 对应的计划签发方，以及发布计划（`DistributionPlan`）中的 `scopes[]`、策略、生效时间、更新策略、固定版本和支持 Owner；
7. 只有 `PENDING` 任务的决策区提供“通过并发布 / 需修改 / 拒绝”；其他任务只读展示决定、取消原因或发布失败证据。

审核人必须在决定前查看冻结的发布计划，不能只审核包而看不到目标受众、安装策略、更新规则或支持责任方。

### 10.6 决策规则

| 决定 | 意见要求 | 对同一冻结快照的结果 |
| --- | --- | --- |
| 通过并发布 | 默认可选；接受风险例外时必填例外原因、范围和责任人 | `ReviewTask.status=DECIDED`、`decision=APPROVED`；系统按同一 checksum 与冻结配置原子创建 `Release` 和 `1..N Distribution`，成功后将版本置为 `PUBLISHED` |
| 需修改 | 必填，必须写明问题与可验证的验收标准 | `ReviewTask.status=DECIDED`、`decision=CHANGES_REQUESTED`；当前版本进入 `CHANGES_REQUESTED`，作者创建新草稿，原任务不得复用 |
| 拒绝 | 必填，必须说明不可接受原因 | `ReviewTask.status=DECIDED`、`decision=REJECTED`；当前版本进入 `REJECTED` 并终止，历史证据保留 |

提交人不得审核自己提交的版本；即使持有权限或属于任务用户组，决策区也必须禁用并说明原因。存在任何阻断项时不能“通过并发布”，审核人只能要求修改或拒绝。每个决定绑定提交时的 checksum；包、权限、`planned_publisher_id` 或发布计划发生变化时，原决定不得迁移到新快照。

### 10.7 审核通过后的自动发布失败

自动发布失败完全沿用第 5 章：`ContentVersion` 暂留 `IN_REVIEW`，`ReviewTask.status=DECIDED` 且 `decision=APPROVED`，任务不可重复决定，`PublishTask=PUBLISH_FAILED`。该记录进入“发布异常 / 已处理”，审核人只读查看，不返回待我审核。

系统自动按退避策略重试同一冻结快照。人工重试不需要也不允许改写审核决定；有效主体与范围完全按第 2 章 `access_control_mode=Open / Custom`、Content Owner 和 `marketplace.publish.retry` 规则计算，本章不定义新模式或新默认权限。每次重试都保持相同 `ContentVersion`（含 `planned_publisher_id`）、发布计划、review decision、checksum 和幂等键，并记录 `AuditEvent`。

若要修改内容、计划签发方或发布计划，必须创建新 `ContentVersion` 并重新校验、提交和审核，不能借发布重试绕过审核。

## 11. 企业分发

企业分发把审核前的发布计划（`DistributionPlan`）转换为审核通过后的实际 `Distribution`。本章只定义策略配置、受众、生命周期效果与安装执行边界；有效候选、冲突和状态转换沿用第 5 章，不定义第二套算法或状态机。

### 11.1 安装策略

| 策略 | 成员体验 | 组织边界 |
| --- | --- | --- |
| `OPTIONAL` 可选安装 | 市场可见，成员主动在 IDE 选择目标并安装 | 允许卸载，并可自行更新到有效 `Distribution` 允许的合规版本 |
| `DEFAULT` 默认安装 | 命中范围后由组织下发；成员可禁用或卸载 | 用户卸载后生成 suppression，组织重置 suppression 必须满足第 11.4 节并留审计 |
| `FORCED` 强制安装 | 命中范围后由组织下发并保持启用 | 仅 `ACTIVE` 时禁止成员禁用或卸载；组织可按更新策略锁定版本 |

策略决定主动安装还是组织下发以及成员能否处置，不改变 `Installation.actual_state` 的统一状态定义，也不创造按策略拆分的安装记录类型。

### 11.2 发布计划配置与冻结

作者必须在提交审核前配置发布计划（`DistributionPlan`），并与 `ContentVersion` 同时冻结。字段至少包括：

| 字段 | 要求 |
| --- | --- |
| `plan_version` | 同一 Content 的发布计划版本，创建新计划时单调递增 |
| `scopes[]` | 非空分发范围（`DistributionScope`）数组；每项包含 Organization、Team、Project 或 Member 类型及稳定标识 |
| `policy` | `OPTIONAL`、`DEFAULT` 或 `FORCED` |
| `effective_at` | 立即生效或明确的计划生效时间 |
| `update_policy` | `FOLLOW_LATEST_COMPLIANT` 或 `PINNED_VERSION` |
| `pinned_version` | `update_policy=PINNED_VERSION` 时提交必填，且必须等于本次 `ContentVersion.version`；发布后 `Distribution.release_id=pinned_release_id=本次新 Release.id` |
| 支持 Owner | 对成员支持、失败处置和策略说明负责的 Member 或团队 |

审核人必须查看上述完整配置及其相对上一版本的变化。计划签发方、`scopes[]`、策略、生效时间、更新策略、固定版本或支持 Owner 的变化都属于审核输入；提交后如需修改，必须新建版本重新审核。

### 11.3 从计划到实际分发

审核“通过并发布”且自动发布成功后，一个发布计划生成 `1..N Distribution`：`scopes[]` 中每个元素生成一条实际记录，引用同一本次新 `Release`，并复制 `plan_version`、`policy`、`effective_at`、`update_policy` 和支持 Owner；固定版本记录的 `pinned_release_id` 与该 `release_id` 相同。实际 `Distribution` 驱动市场可见性、`DEFAULT` / `FORCED` 的组织下发和更新限制，发布计划本身不代表已下发或已安装。

未来生效的计划替换使用第 5.4 节 `replaces_distribution_id` 与原子激活事务。替换记录在 `SCHEDULED` 期间不参与候选、不改变旧 `ACTIVE`、管理锁或 suppression；只有激活、有效策略重算与 Installation 锁更新全部成功后，才按新 `plan_version` 处理 suppression。激活失败保持旧效果并展示失败原因，不形成半切换状态。

实际 `Distribution` 的 `SCHEDULED / ACTIVE / PAUSED / ENDED` 状态转换、有效候选筛选、`FORCED > DEFAULT > OPTIONAL` 优先级、范围精确度、重叠冲突 fail closed 和发布撤回联动，全部沿用第 5 章。实现不得另设目录优先级、任意 tie-breaker 或第二种撤回规则。

### 11.4 策略 × 生命周期

| 策略 / 生命周期 | 市场可见 | 新安装与组织下发 | 已有安装 | 管理锁 | 更新 | 审计 |
| --- | --- | --- | --- | --- | --- | --- |
| `SCHEDULED + 任意策略` | 生效前不可见 | 不安装、不下发 | 不影响 | 不加锁 | 不更新 | 记录计划、替换目标、到期检查、激活结果或 `activation_failure_reason` |
| `ACTIVE + OPTIONAL` | 范围内可见 | 成员主动安装 | 保留，由成员管理 | 无 | 按 `update_policy` | 安装、更新、卸载及失败 |
| `ACTIVE + DEFAULT` | 范围内可见 | 组织下发；成员仍可主动安装 | 可禁用或卸载；卸载生成 suppression | 无强制锁 | 按 `update_policy` | 下发、成员处置、suppression 与重置 |
| `ACTIVE + FORCED` | 范围内可见 | 组织下发 | 保持启用 | 禁止成员禁用 / 卸载 | 按 `update_policy`，可固定版本 | 下发、策略拒绝、更新与失败 |
| `PAUSED + 任意策略` | 范围内可见但标记暂停，无安装 CTA | 停止本记录的新安装和组织下发 | 保留已有包；按全部命中分发重算后的有效策略处置 | 本记录停止贡献 `FORCED` 锁；无其他有效 `FORCED` 时释放 | 暂停本记录驱动的更新 | 暂停、重算、锁变化、成员处置与恢复 |
| `ENDED + 任意策略` | 停止本记录带来的可见性 | 永久停止本记录的新安装和组织下发 | 移除本记录影响；按重算后的有效策略管理 | 本记录永久停止贡献；无其他有效 `FORCED` 时释放 | 停止本记录驱动的更新 | 结束、重算、锁变化和后续成员处置 |
| `ContentVersion=WITHDRAWN` | 停止关联版本带来的可见性 | 结束关联 `Distribution`，停止其新安装和下发 | 显示“版本已撤回”；按其他命中分发重算后的策略管理 | 关联记录停止贡献；无其他有效 `FORCED` 时释放 | 停止撤回版本驱动的更新 | 撤回、关联分发结束、重算与安装提示 |

任何 `PAUSED`、`ENDED`、`ContentVersion=WITHDRAWN` 或策略变化，必须先针对每个受影响 `InstallationTarget` 原子重算全部命中的 `Distribution`，再按新的唯一有效策略更新 `Installation` 快照和管理锁。矩阵中的“释放”只表示当前记录不再贡献 `FORCED` 控制；若仍命中其他 `ACTIVE + FORCED`，最终管理锁必须保留，不能无条件释放。

`FORCED → DEFAULT` 或 `FORCED → OPTIONAL` 生效时，系统在上述原子重算后更新既有 `Installation` 的有效策略快照，不新建安装记录；只有新有效策略分别为 `DEFAULT` 或 `OPTIONAL` 时才释放相应控制。`PAUSED → ACTIVE` 也必须先重算，只有重新成为唯一有效 `FORCED` 时才能重新施加锁。

用户卸载 `ACTIVE + DEFAULT` 安装后，系统为该 content、安装目标和有效 Distribution 生成 suppression。suppression 只阻止组织自动重装，不阻止成员主动重新安装；`ACTIVE` 期间不得因定时对账自动清除。新 `plan_version` 只有在对应 `Distribution` 成功激活后才清除旧 suppression，或在仍需保留成员选择时迁移到新有效 Distribution；创建 `SCHEDULED` 或激活失败都不得提前处理。除此之外，人工重置只允许受影响成员主动恢复自己的默认安装，或持有 `marketplace.distribution.manage` 且身份授权范围与业务对象范围交集命中的操作者明确“再次下发”；每次都记录操作者、原因、前后版本和结果 `AuditEvent`，避免默认安装演变为锁死。

`ContentVersion=WITHDRAWN` 若涉及必须强制禁用的安全风险，不能借普通撤回继续维持 `FORCED` 锁；必须另走有明确授权、原因、影响范围和恢复条件的紧急处置，并记录完整审计。

### 11.5 受众与写入目标

分发范围（`DistributionScope`）是内容对谁可见或向谁下发的受众；安装目标（`InstallationTarget`）是内容实际写入并生效的 User 或 Project 环境。二者必须独立保存和展示。

“研发团队可见”只表示 Team 受众命中，不等于安装到 Team 文件路径。成员主动安装 `OPTIONAL` 内容时仍由 IDE 选择 User 或具体 Project；组织下发 `DEFAULT` / `FORCED` 内容时，也为每个实际 User 或 Project 目标创建统一 `Installation`，记录 `effective_distribution_id`、策略与版本锁定快照，并把下载、安装、配置和启用结果回写到同一实例。

### 11.6 组织下发、失败与重试

组织下发不得新增 `Installation.actual_state`；对已按目标唯一键创建的同一 `Installation`，执行事实映射如下：

| 执行事实 | Installation 记录 |
| --- | --- |
| 客户端未承接或离线 | 保持 `requested`，写入 `reason_code=client_offline` |
| 目标不可达 | 保持 `requested`，写入 `reason_code=target_unreachable` |
| 下载、安装、配置或启用执行失败 | 进入 `error` 并写入具体 `reason_code` |
| IDE 与 Marketplace 暂时无法确认实态 | 进入 `unknown`，恢复后按第 5.6 节既有 transition reconcile |
| 从失败、离线或不可达恢复到新状态 | 仅保留对新 `actual_state` 仍成立的当前原因；否则清空旧 `reason_code` |

`reason_code` 是统一当前态字段：任何状态确认或迁移后，只能保留对新的 `actual_state` 仍成立的当前原因。进入 `installing`、`ready`、`configuration_required`、`disabled`、`uninstalling`、`removed` 等状态时，清除旧失败原因；如当前确有非失败原因则替换为该原因。历史失败只保存在 `AuditEvent`，不得继续挂在已恢复的 Installation 上。

恢复连接或人工重试时复用同一 installation key 与 request ID，不新建记录。自动重试额度为滑动窗口内最多 3 次 / 24 小时；窗口随时间滚动，最早尝试移出窗口后恢复相应额度。达到当前窗口上限时保留 `requested` 或 `error` 及最后 `reason_code`，等待额度恢复或人工重试。每次尝试、跳过、失败、恢复和最终结果都写入 `AuditEvent`。系统可以定义独立调度任务记录退避与下次执行时间，但调度任务不是 Installation 状态。

### 11.7 MVP 管理边界与受管反馈

MVP 不新增独立管理后台；作者在内容发布页配置单版本发布计划，审核人在审核页核对，成员在 IDE Market 查看和处置真实安装。复杂目录管理、跨内容策略编排、批量例外与专用策略控制台留待后续。早期未接线的 webview 治理页面、README 设想或演示状态不视为现成功能，也不能作为企业分发已经落地的证据。

当成员尝试禁用或卸载 `ACTIVE + FORCED` 强制安装内容时，IDE 必须拒绝操作并明确显示“由组织管理”、支持 Owner / 责任方和受权限控制的审计入口，不得只禁用按钮而不解释。所有 `DEFAULT` suppression、再次下发、`FORCED` 策略拒绝、在线 / 离线失败、生命周期锁释放与恢复都写入 `AuditEvent`，并关联对应 `Installation` 和实际 `Distribution`。

## 12. IDE 安装与更新

本章只负责 InfCode IDE 中与真实 User / Project 环境有关的待确认安装、下载、安装、默认启用、待配置、更新、版本回退和状态同步。Marketplace Web 仍只生成 `InstallIntent` 并展示摘要；IDE Market 是创建和处置唯一 `Installation` 的操作面。

### 12.1 待确认安装与安装目标

IDE Market 的“待确认安装”是 `InstallIntent=OPENED` 的界面分组，不是新的 `Installation.actual_state`。IDE 收到意图后按顺序完成：

1. 校验意图未过期，且 `content_id`、`release_id`、权限快照和幂等键完整；
2. 重新校验登录身份、当前可见性、有效 `Distribution`、组织策略、客户端版本和目标环境兼容性；
3. 要求成员确认 User 或具体 Project。Project 不存在、已关闭或无权限时必须重新选择，不得静默降级为 User；
4. 校验通过后，以账号、内容和 InstallationTarget 的唯一键创建或复用一条 `actual_state=requested` 的 `Installation`，再向 Web 返回含 `installation_id` 的 `ACCEPTED` ack；
5. 开始执行后按第 5.6 节的合法状态转换回写同一实例。

组织 `DEFAULT` / `FORCED` 下发不经过成员确认，但仍对每个真实 User / Project 目标创建或复用同一唯一键的 `Installation`，执行相同的策略、兼容性和状态校验。任何意图或下发任务均不得提前生成第二条安装记录。

### 12.2 安装、配置与默认启用

| 阶段 | 合法 `actual_state` | IDE 行为与反馈 |
| --- | --- | --- |
| 已承接等待执行 | `requested` | 显示目标、版本、来源和策略；离线或不可达时保留该状态与当前 `reason_code` |
| 下载与落盘 | `installing` | 展示真实进度；校验 artifact checksum、文件清单和依赖 |
| 存在必填连接、凭据或权限配置 | `configuration_required` | 保留已安装包但不标记可用，展示缺失项和“完成配置”；凭据不回传 Marketplace |
| 无阻断配置且安装成功 | `ready` | `desired_state=enabled`，默认启用并刷新运行时注册表 |
| 配置完成 | `configuration_required → ready` | 再校验连接与权限，成功后自动启用；失败进入 `error` |
| 安装失败 | `installing → error` | 保留可诊断的 `reason_code`、日志和“重试安装”；重试复用同一 Installation，经 `error → installing` 执行 |

安装成功不能只以下载完成判定；只有 `actual_state=ready` 且运行时刷新成功才显示“已可用”。`disabled` 表示包存在但未启用；`unknown` 表示 IDE 与 Marketplace 暂时无法确认实态，两者都不得显示为“未安装”。全部状态只取第 5.6 节与 `packages/marketplace-domain/src/installation-state.ts` 已定义的枚举。

### 12.3 检查更新与合规版本

IDE Market 同时提供“检查全部更新”和每条安装的“检查更新”。两者都只刷新候选与检查结果，不直接更新；检查范围分别是当前视图中所有可管理的 Installation，或指定的唯一 Installation。

候选 `Release` 必须同时满足：关联 `ContentVersion=PUBLISHED` 且该版本未撤回、当前目标仍命中有效 `Distribution`、与当前 IDE 和目标环境兼容、通过安全与允许列表、符合该 Distribution 的 `FOLLOW_LATEST_COMPLIANT` 或 `PINNED_VERSION`，并且按第 5.4 节确定性排序后确实比 Installation 当前 `release_id` 更新。`Release` 本身不增加“已发布 / 已撤回”状态。没有合规候选时显示“已是最新合规版本”或具体策略阻断原因，不把被锁定的更高版本显示为可安装更新。

每次检查记录检查时间、当前 Release、候选 Release、结果和阻断原因。批量检查中的单项失败不阻断其他 Installation，但必须在汇总中分别显示“可更新 / 无更新 / 检查失败 / 由组织锁定”，并允许只重试失败项。

更新过程由独立的 `UpdateOperation` 记录，不把检查中、可更新或更新失败混入 `Installation.actual_state`。最小字段为：`operation_id`、`installation_id`、`from_release_id`、可选 `to_release_id`、`prior_desired_state`、`prior_actual_state`、`state=CHECKED|CONFIRMED|APPLYING|SUCCEEDED|FAILED|CANCELLED`、`check_result=UPDATE_AVAILABLE|UP_TO_DATE|POLICY_LOCKED|CHECK_FAILED`、`request_id`、`idempotency_key`、可选 `previous_operation_id`、可选 `failure_reason`、`created_at`、`updated_at` 和 `revision`。

只有 `check_result=UPDATE_AVAILABLE` 时 `to_release_id` 必填，且只允许该结果执行 `CHECKED → CONFIRMED|CANCELLED`；`UP_TO_DATE` 与 `POLICY_LOCKED` 的 CHECKED 是终态，其中 POLICY_LOCKED 必须记录 `blocking_policy_id` 与 `responsible_owner`。`CHECK_FAILED` 的 CHECKED 也是本次终态且必须有 `failure_reason`，重新检查必须创建新 operation 并以 `previous_operation_id` 指回失败项。其余合法转换仅为 `CONFIRMED → APPLYING|CANCELLED`、`APPLYING → SUCCEEDED|FAILED`，`CANCELLED` / `SUCCEEDED` / `FAILED` 为终态；失败后执行重试同样创建新 operation 并串联前一项。同一 Installation 同时最多一个 `APPLYING`；状态写入使用 revision / CAS，重复请求按 `idempotency_key` 返回原操作。

`UpdateOperation` 保存过程，Installation 仍是当前安装事实源。更新完成时，操作状态、Installation 的 `release_id`、`actual_state`、`desired_state` 和当前原因必须原子回写；失败时按下节保留旧版本与原状态。第 5 章 Installation schema 的规划扩展（如 `last_update_checked_at`、`update_policy_snapshot`、`reason_code`、`snapshot_stale`）实施前必须同步修改 `packages/marketplace-domain/src/schemas.ts`、迁移持久化数据和兼容 API；现有 schema 使用 `additionalProperties: false`，不得在未迁移时直接写入这些字段。检查时间或陈旧标记也可先保存在 `UpdateOperation` / 同步快照中。

### 12.4 更新确认、执行与版本回退

更新确认页至少展示：Release notes、当前版与目标版、文件或组件变化、权限差异、兼容性差异、依赖变化、扫描 / 审核证据以及组织更新策略。新增高风险权限必须单独突出：`OPTIONAL` / `DEFAULT` 由成员发起更新时必须由成员明确确认；`FORCED` 更新的唯一批准证据来自随 `ReviewTask` 一起审核的冻结 `DistributionPlan`，不得另造平行审批对象。

审核人必须持有 `marketplace.review.decide` 且命中组织 scope 交集；`ReviewDecision.risk_approval_evidence` 绑定 `content_version_id + checksum + distribution_plan_hash + permission_diff_hash + scopes`，并记录批准者与例外理由。IDE 必须展示风险、批准者和审计入口，成员确认不是 FORCED 执行门槛。任一绑定项变化都会使证据失效，必须创建新 `ContentVersion`、冻结新 DistributionPlan 并生成新 ReviewTask；没有完整有效证据时禁止 FORCED 更新。

更新执行前保存 `previous_release_id`、旧包校验和原启用状态。`ready` 实例的可见安装转换只使用第 5.6 节的合法路径：

```text
ready → installing → ready
```

`disabled` 实例不得为了更新而短暂进入 `ready`。IDE 在隔离区完成下载、校验和健康检查，成功后原子替换包与 Installation 的 `release_id`，`actual_state` 和 `desired_state` 始终保持 `disabled`；失败则不切换 Release。更新任务有自己的检查、执行和失败状态，不为此发明新的 `Installation.actual_state` 或非法转换。

若 `ready` 实例在新版本执行或切换后失败，先由 `installing → error` 记录 `reason_code=update_failed`，随后自动用原 artifact 执行 `error → installing → ready`。若 `disabled` 实例的隔离更新失败，Installation 保持原 `release_id`、`actual_state=disabled` 与 `desired_state=disabled`，只将更新任务标记失败；若已进入原子切换点后发现异常，则在同一事务中恢复旧包与旧 `release_id`。旧版本和原启用状态恢复成功后，更新任务仍标记失败并展示失败原因。只有原 artifact 恢复也失败时 Installation 才按合法路径进入或保持 `error`，旧 artifact 仍不得删除，并提供“重试恢复旧版本”和日志入口。这里称为“IDE 更新失败后保留旧版本 / 版本回退”，不改变已发布内容或分发生命周期。

### 12.5 组织策略与操作限制

- `ACTIVE + OPTIONAL`：成员可检查并更新到策略允许的合规版本，也可卸载。
- `ACTIVE + DEFAULT`：成员可禁用或卸载；卸载后生成第 11.4 节的 suppression，常规对账不得自动重装。
- `ACTIVE + FORCED`：IDE 保持启用，禁用和卸载操作必须被服务端与客户端同时拒绝；界面显示“由组织管理”、有效 Distribution、支持 Owner 和拒绝原因。
- `PINNED_VERSION`：不允许成员越过 `pinned_release_id`，检查更新需显示锁定版本与责任方；`FOLLOW_LATEST_COMPLIANT` 才按合规排序提供更新。
- Distribution 暂停、结束、替换或内容撤回后，IDE 先按第 11.4 节重算有效策略，再更新同一 Installation 的策略快照与管理锁，不新增安装实例。

### 12.6 紧急安全控制

本节是企业可选启用的平台级 break-glass 安全能力，不是 Marketplace 基础 MVP 默认开启的功能。未启用时不创建 `EmergencyControl`、不展示操作入口，也不要求 Marketplace 新建安全角色或审批后台；企业显式启用后，复用组织既有安全管理与审批主体，并按第 2 章映射三个 emergency 能力点。

高风险强制禁用由独立策略控制对象 `EmergencyControl` 承载，不复用 Distribution 状态，也不新增 Installation actual_state；控制优先级高于任何 Distribution policy，包括 `ACTIVE + FORCED`。最小字段为：`id`、`content_id` 或 `release_id`、`scope` 与解析后的 `targets`、`action=FORCE_DISABLE`、`status=PENDING_APPROVAL|ACTIVE|REJECTED|CANCELLED|LIFTED`、`reason`、`requested_by`、可选 `approved_by`、`rejected_by`、`cancelled_by`、`lift_requested_by`、`lift_approved_by`，以及 `effective_at`（ACTIVE 必填）、`lifted_at`（LIFTED 必填）和 `revision`。

合法转换固定为 `PENDING_APPROVAL → ACTIVE|REJECTED|CANCELLED`、`ACTIVE → LIFTED`，其余均为终态。组织既有安全主体在映射后持有 `marketplace.emergency.request` 且命中 scope，发起后进入 PENDING_APPROVAL，可在尚无决定时取消本人请求；持有 `marketplace.emergency.approve` 且命中 scope 的另一名组织既有安全主体批准或拒绝，`approved_by` / `rejected_by` 不得等于 `requested_by`。解除由持有 `marketplace.emergency.lift` 的主体发起，写入 `lift_requested_by` 但 status 继续保持 ACTIVE；再由持有 `marketplace.emergency.approve` 的另一名组织既有安全主体复核，`lift_approved_by` 不得等于 `lift_requested_by`。解除未获批准时保持 ACTIVE 并审计拒绝原因。所有创建、批准、拒绝、取消、处置、解除和越权尝试均记录完整 AuditEvent，并使用 revision / CAS 防止重复决定。

`ACTIVE` 时服务端与 IDE 拒绝命中范围的新 install、update 和 enable，并按当前 actual_state 确定性处置同一 Installation：

| 当前 actual_state | desired_state 与合法处置 |
| --- | --- |
| `requested` | `desired_state=removed`，执行 `requested → uninstalling` |
| `installing` | `desired_state=removed`，执行 `installing → error`（`reason_code=emergency_force_disable`）`→ uninstalling` |
| `configuration_required` | `desired_state=removed`，执行 `configuration_required → uninstalling` |
| `ready` | `desired_state=disabled`，执行 `ready → disabled` |
| `disabled` | 保持 `desired_state=disabled` 与 `disabled` |
| `error` | `desired_state=removed`，执行 `error → uninstalling` |
| `uninstalling` | 保持 `desired_state=removed` 与 `uninstalling`，继续清理 |
| `removed` | 保持 `desired_state=removed` 与 `removed` |
| `unknown` | `desired_state=disabled`，执行 `unknown → disabled` |

每个目标的前后状态、原因和结果都写入审计；中间或最终失败保留最后合法状态与原因，后台继续处置。Distribution 的暂停、结束、替换、策略重算或 FORCED 锁释放均不解除 ACTIVE 控制。

控制生效事务还必须原子处置目标的 UpdateOperation：现有 `APPLYING → FAILED`，写入 `failure_reason=emergency_control` 和 `emergency_control_id`，并与 Installation 处置及 AuditEvent 一起提交。`CHECKED` / `CONFIRMED` 可保留，但每次进入 APPLYING 前必须重新检查全部命中的 ACTIVE 控制；若命中则统一转为 `CANCELLED`，以 `failure_reason=emergency_control` 记录阻断且不得执行更新。

每次控制进入 `ACTIVE` 或 `LIFTED` 都按目标重算全部命中的 ACTIVE EmergencyControl；只要仍有至少一条，服务端与 IDE继续拒绝 install / update / enable 并保持安全 `desired_state`。只有计数归零才解除安全限制，再重算有效 Distribution 与 Installation 策略快照，并保持当时的 `desired_state=disabled|removed`；解除不得自动恢复使用或重装。解除审计必须记录 `remaining_active_control_ids`。后续启用 / 安装需成员在允许自主管理时主动执行，或由另一次明确获批的组织恢复动作执行，并继续遵守当前 Distribution。

### 12.7 状态同步与当前原因

IDE 是实际环境状态的权威执行方，Marketplace API 是跨端共享记录。每次回写携带 `installation_id`、客户端 revision、`actual_state`、`release_id`、`desired_state`、策略快照、检查时间和当前 `reason_code`；旧 revision 不得覆盖新状态。Web 在重新获得焦点或收到事件后刷新同一记录，刷新失败显示“状态待同步”并保留上次确认值。

只有 `ready`、`disabled` 或 `error` 可按第 5.6 节进入 `unknown`。`requested`、`installing`、`configuration_required`、`uninstalling` 或 `removed` 发生同步失败时保留最后确认的 actual_state，在同步快照或进行中的 operation 上标记 `snapshot_stale=true` 并审计，禁止写入非法的 `→ unknown`。恢复时先按 revision 对账，再使用第 5.6 节允许的转换。

`reason_code` 只解释当前 `actual_state`。离线恢复、重试或同步确认后，系统按第 11.6 节清除不再成立的旧原因；历史原因保留在 `AuditEvent`，不能继续挂在已经 `ready`、`disabled` 或其他不匹配状态的 Installation 上。

## 13. 数据统计与审计

本章使用同一 Content、Version、Release、Distribution 与 Installation 身份计算市场运营、作者数据和治理证据。统计用于判断触达、转化和实际采用；`AuditEvent` 用于回答谁在何时因何原因改变了哪个对象，两者不得互相替代。

### 13.1 MVP 指标与口径

除非指标另有说明，趋势默认同时提供近 7 日、近 30 日和可选日期范围；人数按企业内去重 Member 计算，安装按去重 `installation_id` 计算，执行按幂等后的 `content_executed` 计算。比率必须同时展示分子、分母、时间窗和去重键，跨组织数据不得因汇总泄露未授权的 Team、Project 或 Member 明细。

| 指标组 | MVP 指标 | 默认口径 |
| --- | --- | --- |
| 市场触达 | 搜索曝光、推荐曝光、详情访问、收藏人数、近 30 日增长 | 以可见内容曝光 / 访问事件为基础，Member + Content + 展示会话去重；收藏看期末有效 Favorite 与新增 / 取消变化 |
| 供给 | 新增内容、提交版本、审核通过率、需修改率、拒绝率、平均 / P95 审核时长、自动发布失败率 | `Content`、`ContentVersion`、`ReviewTask` 和 `PublishTask` 分域统计；通过率分母为期间已决定审核，不含 PENDING / CANCELLED |
| 分发转化 | 详情到 IDE 打开率、安装请求人数、包安装完成人数、可用安装成功人数、默认下发成功率、收藏到安装转化、各策略覆盖 | 包安装完成包括 `install_succeeded` 的 ready 与 configuration_required outcome；可用安装成功只计最终 `ready`，待配置必须单列且不计入有效采用 |
| 实际使用 | 已启用安装、近 7/30 日活跃 Member / Installation、成功执行次数与成功率、P50/P95 时长、失败原因、持续使用率 | 已启用只计 `actual_state=ready`；活跃要求窗口内至少一条有效执行；执行成功率以有最终结果的执行为分母；持续使用率明确 cohort 起点与后续窗口 |
| 质量治理 | 扫描阻断 / 警告、高风险权限、版本分布、更新覆盖率、更新失败率、撤回、紧急强制禁用、异常 / 过期实例 | 关联不可变 Version / Release 和当前 Installation；撤回与紧急处置分开计数，不以下载量代替质量结论 |

关键比率使用以下最小可计算口径；`分子 / 分母` 均先按去重键去重，再限制到同一归因窗口或时点快照：

| 比率 | 分子 / 分母 | 去重键 | 归因窗口或快照 | 事件 / 对象来源 |
| --- | --- | --- | --- | --- |
| 审核通过 / 需修改 / 拒绝率 | 窗口内相应 decision 数 / 窗口内全部首次有效决定数 | `review_task_id` | decision 的 `occurred_at` 落入统计窗口 | `review_decided`、ReviewTask |
| 自动发布失败率 | 首次自动发布结果为失败的获批版本数 / 窗口内触发首次自动发布的获批版本数 | `content_version_id` | 审核决定后 24 小时归因，按首次结果 | PublishTask、`version_published`、发布失败审计 |
| 详情到 IDE 打开率 | 成功 `OPENED` 的有效 intent 数 / 从详情生成的有效 intent 数 | `intent_id` | 生成后 30 分钟 | InstallIntent |
| 可用安装成功率 | 首次进入 `ready` 的安装请求数 / 已承接安装请求数 | `request_id`，回退用 `installation_id` | 请求后 24 小时 | Installation 首次 ready transition，或关联同一 install_request 的首次 `content_enabled` |
| 默认下发成功率 | 首次进入 `ready` 的 DEFAULT 目标数 / 应下发的有效 DEFAULT 目标数 | `distribution_id + target` | Distribution 激活后 24 小时 | Distribution 目标快照、Installation 首次 ready transition / `content_enabled` |
| 收藏到安装转化率 | 收藏后首次进入 `ready` 的 Member + Content 数 / 窗口内新增收藏的 Member + Content 数 | `member_id + content_id` | 收藏后 30 天；ready 必须发生在收藏之后 | Favorite 变更、Installation 首次 ready transition / `content_enabled` |
| 执行成功率 | 成功且有最终结果的执行数 / 全部有最终结果的执行数 | `execution_id` | `occurred_at` 落入统计窗口 | `content_executed` |
| 持续使用率 | cohort 中后续窗口仍有成功执行的有效采用 Installation 数 / cohort 起始窗口首次成为有效采用的 Installation 数 | `installation_id` | 默认起始 30 天 cohort，观察随后 30 天 | Installation 快照、`content_executed` |
| 更新覆盖率 | 快照时已安装目标合规 Release 的 Installation 数 / 该目标 Release 发布时符合策略与兼容性的 Installation cohort 数 | `installation_id` | 目标 Release 发布时冻结 cohort，按每日固定时点观察 | Distribution / Installation cohort 快照、UpdateOperation、Release |
| 更新失败率 | 首次 APPLYING 最终为 FAILED 的操作数 / 窗口内首次进入 APPLYING 的操作数 | `operation_id` | 操作开始后 24 小时取最终结果 | UpdateOperation、`update_succeeded`、`update_failed` |

安装量不等于有效采用。MVP 的“有效采用 Installation”必须同时满足：统计时点仍为 `actual_state=ready`（已启用）、近 30 天至少活跃一次，并且近 30 天至少存在一次结果为成功的 `content_executed`；三个条件的数量与交集必须分别展示。对没有运行时执行语义的类型，不得用安装成功冒充执行成功，应在产品确认该类型的等价成功事件前显示“有效采用口径不适用”。

### 13.2 最小行为事件

| 事件 | 触发时机与最小关联 |
| --- | --- |
| `content_created` | Content 首次成功持久化；关联 content、owner 和创建入口 |
| `version_submitted` | 不可变 ContentVersion 与 DistributionPlan 冻结并进入审核；关联 checksum、submission 和 ReviewTask |
| `review_decided` | 审核决定首次持久化；关联 reviewer、decision、意见和 checksum |
| `version_published` | Release 与实际 Distribution 成功创建、版本进入 PUBLISHED；关联 release 与 plan_version |
| `distribution_changed` | Distribution 创建、激活、暂停、恢复、替换、结束或有效策略变化；关联前后值与原因 |
| `install_requested` | IDE 接受主动安装或组织下发并创建 / 复用 Installation；关联 intent / delivery request、target 和有效 Distribution |
| `install_succeeded` | 每个 install_request 在包下载、校验和落盘完成时只发一次；`outcome=ready` 表示当次已可用，`outcome=configuration_required` 只表示包安装完成但仍不可用，不计入可用安装成功或有效采用。后续配置完成进入 ready 时不重发该 event_id，改由 Installation 首次 ready transition 或关联同一 request 的 `content_enabled` 计算可用成功 |
| `install_failed` | 安装进入 `error`；关联 stage、reason_code 和可重试性 |
| `content_enabled` | Installation 首次进入或重新进入 `ready`；区分用户、配置完成和组织保持启用 |
| `content_disabled` | Installation 进入 `disabled`；关联成员操作或紧急处置原因 |
| `content_executed` | 已安装内容发生一次有最终结果的执行；关联 installation、release、outcome、duration 和失败原因 |
| `update_checked` | 单项或批量中的每个 Installation 完成检查；关联当前 / 候选 Release 与检查结果 |
| `update_succeeded` | 新 Release 切换与健康检查全部成功；关联 previous / installed Release 和启用状态 |
| `update_failed` | 新版本执行或健康检查失败；关联失败阶段、旧版本恢复结果和最终 Installation 状态 |
| `version_withdrawn` | PUBLISHED 版本撤回并触发关联 Distribution 结束；关联原因、影响范围和处置人 |

### 13.3 事件幂等、维度与迟到数据

每个行为事件必须包含全局唯一 `event_id`、`occurred_at`、`received_at`、组织、actor（系统动作可为空但要有 service identity）、`content_id`，并按场景关联 `content_version_id`、`release_id`、`distribution_id`、`installation_id`、`request_id` / `intent_id`、来源端和 outcome。生产方重试必须复用 `event_id`；聚合层按 `event_id` 幂等，不能把重试、客户端重发或离线补传计成新行为。

默认可切分维度包括时间、内容类型、Content、Release、来源生态、签发方、供给方式、Distribution policy / scope type、InstallationTarget type、客户端版本、结果与 `reason_code`。Team、Project 和 Member 维度只能在调用者授权范围内展示；低基数结果应聚合或隐藏，运行输入、凭据、源码和敏感项目内容不得作为分析维度。

离线事件以 `occurred_at` 归入业务时间窗，以 `received_at` 标记数据新鲜度；超出报表冻结窗口的迟到数据进入更正批次并显示更新时间。删除或撤回内容不删除已经合法采集的聚合历史，但明细展示继续受当前权限与保留策略约束。

### 13.4 AuditEvent 边界

创建、提交、审核决定与撤回、发布与重试、Distribution 生命周期和替换、suppression 与再次下发、安装策略拒绝、安装 / 更新 / 版本回退、内容撤回、紧急强制禁用和权限异常均生成不可变 `AuditEvent`。每条至少记录 audit ID、actor / service identity、对象类型与 ID、前后值、原因、来源端、时间、request / idempotency key 和结果。

普通曝光和正常执行只进入行为事件，不逐条成为治理审计；当执行导致策略拒绝、权限升级、数据外发或其他治理决定时，才额外写 AuditEvent。分析事件的去重或更正不能改写审计原件；审计更正只能追加新事件引用原 audit ID。完整审计读取继续受 `marketplace.audit.read` 与对象范围交集约束。

## 14. 异常、撤回与版本回退

本章统一异常反馈与恢复责任。术语严格分域：IDE 安装更新失败后的旧版本恢复称“版本回退”；已发布 ContentVersion 停止使用称“版本撤回”；Distribution 停止新增影响称“暂停”或“结束”。三者不以一个笼统状态相互替代。

### 14.1 MVP 异常处置矩阵

| 异常 | 保留的对象与状态 | 用户反馈 | 重试 / 恢复入口 | 是否重新审核 | 必须写入的 AuditEvent |
| --- | --- | --- | --- | --- | --- |
| 上传解析或自动校验失败 | 已成功创建的 `ContentVersion=DRAFT` 和校验证据保留；未创建成功时保留本地文件与 `request_id`，不伪造草稿 | 发布页显示具体文件、规则、阻断级别和原始日志；导航失败另按第 10.1 节恢复 | 修复可编辑草稿后重校验；请求结果不确定时以同一 `request_id` 查询 / 重放 | 未提交则否；冻结版本变化后按新版本提交 | `upload_validation_failed`，记录 submission、规则、checksum / 文件摘要和结果 |
| 提交或审核阻断 | 提交前阻断保持 DRAFT；审核人要求修改时旧版本进入 `CHANGES_REQUESTED` 且不可变 | 分别显示阻断规则，或审核意见与可验证验收标准 | DRAFT 可修复后重提；`CHANGES_REQUESTED` 必须创建新 DRAFT | 创建新版本后是 | `submission_blocked` 或既有 `review_decided`，关联规则 / 决定与 checksum |
| 审核通过后自动发布失败 | ContentVersion 暂留 `IN_REVIEW`；ReviewTask 保持 `DECIDED + APPROVED`；`PublishTask=PUBLISH_FAILED` | 作者和按 `Open / Custom` 具备有效重试能力的 Member 看到失败阶段、影响、下次自动重试与“重试发布”；审核人只读 | 系统退避或有权主体以同一幂等输入重试；不可修改快照 | 同一快照否；内容、签发方或计划变化时是 | `publish_failed` / `publish_retried`，记录同一 checksum、plan_version、幂等键与结果 |
| Web 唤起 IDE 失败或意图过期 | 不创建 Installation；未过期 intent 保持可重试，失败 / 过期状态保留至清理期 | 详情保留上下文，显示“尚未创建安装”、重试和复制操作信息 | 未过期复用同一 `idempotency_key`；过期后创建新 intent | 否 | `install_handoff_failed`，关联 intent、source_page、失败原因和是否重试 |
| 下载、安装或启用失败 | 同一 Installation 进入 `error`，保存阶段、当前 `reason_code` 和诊断日志；不创建替代实例 | IDE 显示失败阶段、支持 Owner、日志和“重试安装” | 复用 installation key / request ID，执行 `error → installing`；自动额度按第 11.6 节 | 否 | 既有 `install_failed` 对应审计，记录 attempt、阶段、reason_code 和最终状态 |
| 必填连接或凭据待配置 | Installation 保持 `configuration_required`，包存在但不可用，凭据只存 IDE / 连接系统 | IDE 明确缺失项、数据访问范围和“完成配置”；Web 只显示待配置 | 补充后重新校验并执行 `configuration_required → ready`；取消则进入 `uninstalling` | 否；若包或权限声明变化则走新版本审核 | `configuration_required` / `configuration_completed` / `configuration_failed`，不记录凭据值 |
| IDE 与 Marketplace 同步失败 | 仅 `ready` / `disabled` / `error` 可合法进入 `unknown`；其他 actual_state 保留最后确认值，并在同步快照或 operation 标记 `snapshot_stale=true` | 双端显示“状态待同步”和最后确认时间，不显示未安装或伪造成功 | IDE 重连后以 revision 对账，同一 Installation 按第 5.6 节合法 transition 恢复 | 否 | `installation_sync_failed` / `installation_reconciled`，记录前后 revision、状态、snapshot_stale 和冲突处理 |
| 更新执行失败 | 保留 `previous_release_id` 和旧 artifact；按第 12.4 节恢复旧版本与原启用状态，恢复失败则保持 `error` | IDE 展示目标版本、失败阶段、旧版本恢复结果、日志和再次尝试入口 | 重试目标更新，或重试恢复旧版本；均复用同一 Installation | 否；若要发布修复包则新版本需审核 | 既有 `update_failed` 对应审计，记录目标、旧版本、恢复步骤与最终状态 |
| 已发布版本撤回 | ContentVersion `PUBLISHED → WITHDRAWN`；关联 Distribution 原子进入 `ENDED`；Installation 历史和 artifact 证据保留 | 市场禁用新安装并说明撤回原因；IDE 标记已撤回、替代版本和处置建议 | 不能恢复同一已撤回版本；修复内容创建新版本并完整审核发布 | 新版本是 | 既有 `version_withdrawn` 及 `distribution_changed` 对应审计，记录影响范围和锁重算 |
| 高风险紧急强制禁用 | `PENDING_APPROVAL` 不影响安装；双人审批进入 `ACTIVE` 后按第 12.6 节逐状态处置，持续拒绝 enable / update / install；拒绝 / 取消不生效 | IDE 显示当前审批 / 隔离状态、发起人、决定人、影响范围、恢复条件和支持 Owner | Distribution 变化不解除；ACTIVE 只可经 lift 发起 + approve 复核进入 LIFTED，解除后仍不自动启用 / 重装 | 禁用 / 解除否；新发行物是 | 请求、批准、拒绝、取消、目标处置、操作拒绝、解除及越权尝试均关联 EmergencyControl revision |

### 14.2 计划替换与恢复一致性

带 `replaces_distribution_id` 的 `SCHEDULED` 替换只有在旧记录结束、新记录激活、有效策略与 Installation 锁重算、suppression 处理全部成功后才提交。成功时 UI 展示新的 `plan_version` 和生效时间，并记录一组有同一 transaction ID 的 `distribution_changed` 审计。

任一步失败时事务不提交：新记录保持 `SCHEDULED` 并写 `activation_failure_reason`，旧 `ACTIVE`、管理锁和 suppression 均保持原状；系统告警并允许有权操作者修复原因后重试同一替换。替换失败不触发重新审核，除非要修改已经冻结并审核的 Release、scope、policy 或其他 DistributionPlan 输入。

任何离线、安装、同步、更新或替换异常恢复后，都必须按新 `actual_state` 清理已经不成立的 `reason_code`；清理动作与原失败原因通过 AuditEvent 串联，不能删除历史证据，也不能让旧错误继续污染当前健康状态。

## 15. MVP 端到端示例

本章用一个独立 Skill 和一个复合 Plugin 验证同一平台框架既能支持简单内容，也能保留 Plugin 的组成、来源、权限和配置差异。

### 15.1 独立 Skill：需求转测试用例

1. 作者从“个人中心 → 本地”在线创建 Skill，系统生成 `Content(type=Skill)` 与 `ContentVersion v0.9.0=DRAFT`，Owner 固定为当前 Member，并记录 `content_created`。
2. 作者填写能力、输入输出，保存 `SKILL.md`，声明只读当前 Project 文件且不允许网络或写操作。自动校验通过结构、秘密、脚本、权限和兼容检查后，作者配置“研发 Team + OPTIONAL + FOLLOW_LATEST_COMPLIANT”发布计划并提交；版本与计划冻结，进入 `IN_REVIEW`。
3. 审核人发现缺少失败场景，决定“需修改”。v0.9.0 进入 `CHANGES_REQUESTED` 且保持不可变；作者基于它新建 `ContentVersion v1.0.0=DRAFT`，补充示例、生成新 checksum，再次校验和提交新的 ReviewTask。
4. 审核人对 v1.0.0“通过并发布”。系统按冻结快照自动创建不可变 Release 与 Team 范围的 ACTIVE Distribution，将 v1.0.0 置为 `PUBLISHED`，不等待作者二次操作。
5. 成员从市场搜索进入详情并点击“在 InfCode 中安装”。IDE 重新校验后为指定 Project 创建唯一 Installation，执行 `requested → installing → ready`，`desired_state=enabled`，默认启用并刷新 Skill 运行时。
6. 成员在 IDE 中运行 Skill；`content_executed` 回流结果、耗时和 Release。数据概览分别显示触达、安装、已启用、近 30 日活跃、成功执行及有效采用交集。
7. v1.1.0 经相同流程发布后，成员单项检查更新，查看 Release notes、兼容性与权限差异并确认。新版本健康检查失败时，IDE 记录 `update_failed`，保留 v1.0.0 artifact，自动恢复 v1.0.0 和更新前启用状态；界面显示“更新失败，已保留旧版本 v1.0.0”。

### 15.2 复合 Plugin：研发协作助手

1. 作者通过现有 IDE `/create-plugin` 创建“研发协作助手”，submission API 返回同一 `Content(type=Plugin)` 与 DRAFT Version，Web 打开统一发布页。包包含一个 `@Plugin` 入口、需求拆解 Skill、提交规范 Rule、`/summarize-change` Command、必需的工单 MCP 连接和 Config Schema。
2. 发布页披露每个组件的用途、权限和来源：自研 Skill / Rule 记录内部来源；Mirror 的开源组件固定 repository、tag / commit、许可、文件清单与 checksum；Compose 后的整个 Release 只有一个计划签发方，并由其承担维护和撤回责任。MCP 只声明服务、权限和配置 Schema，不随包分发凭据。
3. 作者配置目标 Team、`DEFAULT`、`PINNED_VERSION=2.0.0` 和支持 Owner，提交后冻结组件清单、来源证据、权限、兼容性和 DistributionPlan。审核人查看新增网络权限、MCP 数据范围、版本差异和不可变证据后“通过并发布”，系统自动创建 Release 与 Distribution。
4. 命中范围的 IDE 为某 Project 创建唯一 Installation，执行 `requested → installing → configuration_required`。IDE 明确提示必填工单连接；Marketplace Web 只显示“待配置”，两端都不得显示已可用。
5. 成员在 IDE 选择企业连接实例并完成授权。连接校验成功后，同一 Installation 执行 `configuration_required → ready`，`desired_state=enabled`，默认启用并刷新 `@Plugin`、Command、Skill 和 Rule 注册；凭据始终留在连接系统。
6. 成员通过 `@Plugin` 或 `/summarize-change` 使用，事件统一关联 Plugin 的 Content、Release 和 Installation；内部组件不因此生成独立市场、安装或数据身份。检查更新时，IDE 受 `PINNED_VERSION` 限制，只展示组织允许的版本与责任方。

## 16. 验收标准

本章是可直接转换为测试用例的 MVP 验收矩阵。除“静态文档 / 工程边界”行外，测试环境必须能查询对应行为事件与 AuditEvent，并验证没有额外 Content、Release、Distribution 或 Installation 被重复创建。标为“可选 break-glass”的场景仅在企业显式启用第 12.6 节能力时验收，不作为 Marketplace 基础 MVP 的发布阻断项。

| 验收场景 | 前置对象 / 条件 | 动作 | 预期状态变化 | 界面反馈 | AuditEvent / 行为事件 | 重试或最终结果 |
| --- | --- | --- | --- | --- | --- | --- |
| 单一主 PRD 与增量工程边界 | 仓库存在本文件、两份历史 PRD、现有 `apps/marketplace-web` 与 `webview` | 检查文档状态和实现入口 | 本文件为唯一当前生效 PRD；不创建第二个正式 Web / IDE 数据模型 | 历史 PRD 指向本文件；正式页面仍在现有 React 与 IDE Market 导航中 | 不适用（静态文档 / 工程验收） | 重复检查结果一致，不依赖 Demo 运行 |
| Skill 与 Plugin 共存 | 各有一个 `ContentVersion=PUBLISHED`、存在关联 Release 且当前成员可见 | 搜索并分别进入详情 | 两者保持独立 Content / Version / Release 身份 | Skill 展示自身能力；Plugin 展示组件、来源、许可、权限与不可变证据 | 搜索 / 详情行为事件分别关联正确 content_id | 刷新或返回后内容类型与详情区块不丢失 |
| 本地、收藏与安装分域 | 当前成员拥有 A、收藏 B、IDE 安装 C | 打开个人中心和 IDE Market | A 只在本地，B 只在收藏，C 只形成 Installation；关系互不创建 | 三处列表分别展示正确对象，收藏明确不等于安装 | Favorite 与 Installation 事件分别关联 B / C；无伪造审计 | 取消收藏、卸载 C 均不改变 A 或其他关系 |
| 上传成功但发布页导航失败 | submission 已以 request_id 创建 Content 与 DRAFT Version | 模拟上传成功后的路由失败并返回本地 | 原 DRAFT 持久保留，不新增版本或 submission | 显示“继续编辑”；不得只显示成功 Toast 或要求重传包 | `content_created` 和导航失败诊断关联同一 request_id | 再次打开同一路由进入原草稿；重放返回同一 ID |
| 审核撤回 | `ContentVersion=IN_REVIEW`、ReviewTask=PENDING、无决定和 PublishTask | 作者点击撤回 | ReviewTask→CANCELLED，Version→REVIEW_WITHDRAWN，冻结快照不变 | 作者看到“已撤回，可基于该版本新建草稿”；审核队列移除 | `review_withdrawn` 记录 actor、checksum 和前后值 | 继续发布必须创建新 DRAFT、新 checksum 和新任务 |
| 审核通过并自动发布 | 非本人审核人、无阻断、Version 与 DistributionPlan 已冻结 | 点击“通过并发布” | ReviewTask→DECIDED/APPROVED；原子创建一个 Release 和 1..N Distribution；Version→PUBLISHED | 市场或计划状态按 effective_at 显示，无额外发布确认按钮 | `review_decided`、`version_published`、`distribution_changed` 关联同一 checksum / plan_version | 重放决定不重复创建对象，返回原结果 |
| 自动发布失败 | ReviewTask 已 DECIDED/APPROVED，模拟 Release 或 Distribution 创建失败 | 系统执行自动发布 | Version 暂留 IN_REVIEW，PublishTask=PUBLISH_FAILED；无半个 Release / Distribution | 作者和按 `Open / Custom` 具备有效重试能力与 scope 的 Member 看到失败阶段和重试；审核人只读且任务不回待审 | `publish_failed` 记录冻结输入、幂等键和事务结果 | 自动或有权人工以同一输入重试；成功只生成一组对象 |
| Web → IDE 唯一安装记录 | 可安装 Release、无活跃 Installation、有效 intent | Web 唤起 IDE，IDE 选择 Project 并确认 | ack 前无 Installation；ACCEPTED 后仅一条 `requested`，后续在同一 ID 转换 | Web 先显示等待确认；IDE 显示目标与风险；成功后双端读取同一状态 | `install_requested` 关联 intent_id、installation_id 和 target | 未过期重试复用 idempotency_key，不产生第二条 Installation |
| 默认启用 | 无阻断配置的 OPTIONAL / DEFAULT 内容，Installation=requested | IDE 执行安装 | `requested → installing → ready`，`desired_state=enabled` | 显示“已可用”，运行时已注册；Web 刷新同一摘要 | `install_succeeded`、`content_enabled` 关联同一 Installation | 重放成功回执不重复计数或重复启用 |
| 待配置例外 | Plugin 有必填 MCP，Installation=installing | 完成包安装，再提交有效连接配置 | `installing → configuration_required → ready`，最后默认启用 | 配置前显示缺失项且不可用；配置后显示已可用 | 前者发出 `install_succeeded(outcome=configuration_required)` 但不计可用成功 / 有效采用；ready 后再发 `content_enabled`，均不含凭据 | 配置失败进入 error；按合法 transition 重试同一实例 |
| 全部 / 单项检查更新与差异 | 同一视图有可更新、最新、锁定和检查失败实例 | 分别执行批量与单项检查 | 各创建 `UpdateOperation=CHECKED`，check_result 分别为 UPDATE_AVAILABLE / UP_TO_DATE / POLICY_LOCKED / CHECK_FAILED；仅 UPDATE_AVAILABLE 有必填 to_release_id 且可确认 | 分项显示候选、Release notes、权限 / 兼容差异、锁定或失败原因 | 每个 Installation 一条幂等 `update_checked`，关联 operation_id 与 check_result | 非可更新结果不得进入 CONFIRMED；CHECK_FAILED 重查新建 operation 并写 previous_operation_id |
| 更新并发与幂等 | 同一 Installation 已有 `UpdateOperation=APPLYING` | 以新 request 执行更新，再重放原 request | 新 request 由 CAS 拒绝；原 idempotency_key 返回原 operation；最多一个 APPLYING | 显示“更新正在进行”及原操作进度，不创建第二任务 | 并发拒绝和重放均记录 operation_id、revision 与 request_id | 原操作原子进入 SUCCEEDED / FAILED 后才允许新操作 |
| 更新失败保留旧版本 | ready 或 disabled 的旧版本可更新，模拟目标版健康检查失败 | 确认并执行更新 | UpdateOperation→FAILED；ready 按合法转换记录 error 并恢复旧版；disabled 在隔离区失败则状态始终不变；Installation 均恢复 from_release_id 与原启用状态，原 artifact 也恢复失败才最终为 error | 显示目标失败、旧版本恢复结果和日志，不显示更新成功 | `update_failed` 关联 operation_id、目标、旧版本、原启用状态与恢复结果 | 可重试目标更新或旧版恢复，不创建新 Installation |
| 高风险权限更新决策 | OPTIONAL / DEFAULT 与 FORCED 各有一项新增高风险权限的候选更新 | 分别发起更新 | OPTIONAL / DEFAULT 未获成员确认不进入 APPLYING；FORCED 仅在 risk_approval_evidence 完整且审核人持权并命中 scope 时进入 APPLYING | 前者显示成员确认；后者展示风险、ReviewDecision 批准者和审计，不把成员确认设为门槛 | 证据绑定 version、checksum、plan hash、permission diff hash 与 scopes，并关联 UpdateOperation | 任一绑定项变化立即失效，FORCED 更新被拒绝，必须新建版本 / ReviewTask；不得补造平行审批 |
| DEFAULT suppression 与再次下发 | ACTIVE+DEFAULT 安装已被成员卸载并有 suppression | 定时对账，再由成员恢复或具备有效分发管理能力与 scope 的 Member“再次下发” | 对账不重装；合法重置后清除 / 更新 suppression，并在同一 key 上 `removed → requested` | 显示“已选择不自动安装”、重置主体与原因 | `suppression_created`、`suppression_reset` 和再次下发结果 | 无权限重置被拒绝；合法重试幂等且不重复安装记录 |
| FORCED 拒绝禁用 / 卸载 | 唯一有效策略为 ACTIVE+FORCED，Installation=ready | 成员尝试禁用和卸载 | actual_state、desired_state、锁和包均不变 | 明确显示“由组织管理”、支持 Owner 与拒绝原因 | `managed_action_rejected` 分别记录动作、Distribution 和 actor | 重复尝试仍拒绝；Distribution 生命周期改变后重新计算权限 |
| 离线恢复与 reason_code 清理 | 组织下发 Installation=requested、reason_code=client_offline | 客户端上线并安装成功 | 同一实例 `requested → installing → ready`，旧 reason_code 清空 | 先显示离线 / 等待，成功后显示已可用且无旧错误 | 下发尝试、恢复、`install_succeeded` 与 reason 清理可串联 | 复用 request ID；自动重试额度符合 3 次 / 24 小时 |
| SCHEDULED 替换成功 | 旧 Distribution=ACTIVE，新记录=SCHEDULED 且引用旧记录 | 到达 effective_at 并通过全部校验 | 同一事务旧→ENDED、新→ACTIVE、锁重算并按新 plan_version 处理 suppression | 展示新策略 / 版本与生效时间，无半切换提示 | 同一 transaction ID 的 `distribution_changed` 记录全部前后值 | 事务重放返回已完成结果，不重复清除 suppression |
| SCHEDULED 替换失败 | 同上，但模拟冲突或锁重算失败 | 执行激活 | 新记录仍 SCHEDULED 并写 activation_failure_reason；旧 ACTIVE、锁、suppression 不变 | 显示激活失败、旧策略仍生效和重试入口 | `distribution_activation_failed` 记录失败阶段和未提交事务 | 修复后重试同一替换；修改冻结输入才重新审核 |
| Distribution 结束释放锁 | Installation 当前由某 ACTIVE+FORCED 管理且无其他 FORCED 候选 | 有权主体结束该 Distribution | Distribution→ENDED；重算后同一 Installation 更新有效策略快照并释放 FORCED 锁 | IDE 移除强制限制，说明策略已结束；包不被无条件删除 | `distribution_changed` 与 `installation_policy_recomputed` 关联同一目标 | 若仍命中其他 FORCED 则锁必须保留；重复结束幂等 |
| 可选 break-glass：紧急权限与 scope | 企业已启用平台安全策略；普通 Member 与分别映射 request / approve / lift 的组织既有安全主体 | 在命中与不命中 control scope 时分别请求、审批和解除 | 普通 Member、缺权限或 scope 不交集均拒绝；只有映射能力 + scope 交集允许进入下一状态 | 明确显示权限或范围不足，不泄露范围外目标 | 所有允许与拒绝记录 actor、权限点、身份 scope 和 control scope | 未启用时无入口、无 EmergencyControl；启用后也不向 Open 全体 Member 开放 |
| 可选 break-glass：紧急控制批准生效 | 企业已启用平台安全策略，映射 request 能力的组织既有安全主体命中 scope | 发起后由另一名映射 approve 能力的组织既有安全主体批准 | `PENDING_APPROVAL → ACTIVE`，批准人与发起人不同；开始确定性处置目标并拒绝 install / update / enable | IDE 展示隔离原因、双人授权和恢复条件 | 请求、审批、生效和目标处置关联同一 control id / revision | Distribution 暂停、结束或锁释放后控制仍保持 ACTIVE |
| 可选 break-glass：紧急控制拒绝 / 取消 | 分别准备两个 PENDING_APPROVAL 控制 | 独立审批人拒绝第一项；发起人在未决定前取消第二项 | 分别 `→ REJECTED`、`→ CANCELLED`，均不处置 Installation | 显示决定人、原因和终态，不显示生效中 | rejected_by / cancelled_by、权限与 scope 校验完整审计 | 终态不能重开；需重新发起新 control |
| 可选 break-glass：ACTIVE 对各安装状态的处置 | 九条命中 control 的 Installation，actual_state 覆盖全部枚举 | 控制生效并执行处置 | requested→uninstalling；installing→error(emergency)→uninstalling；configuration_required→uninstalling；ready→disabled；disabled 保持；error→uninstalling；uninstalling 保持；removed 保持；unknown→disabled | 每条显示隔离 / 清理状态及当前原因，不出现非法跳转 | 每条前后状态、desired_state、reason 和结果关联 control id | 中断后从最后合法状态继续；新 install / update / enable 始终拒绝 |
| 可选 break-glass：隔离期间成员尝试启用 | EmergencyControl=ACTIVE、Installation=disabled | 成员点击启用或更新 | Installation 与 UpdateOperation 均不变化，不进入 ready / APPLYING | 明确提示安全隔离、责任方和审计入口 | `emergency_disable_action_rejected` 记录 actor、action、control id | 重复操作仍拒绝，不消耗安装 / 更新重试额度 |
| 可选 break-glass：更新中触发紧急控制 | Installation 有 UpdateOperation=APPLYING，另有 CHECKED / CONFIRMED 操作 | EmergencyControl 进入 ACTIVE | APPLYING→FAILED；CHECKED 保留，CONFIRMED 保留；Installation 按状态矩阵处置，全部在同一事务提交 | 显示更新因安全控制终止；再次应用前重新检查控制 | operation 写 failure_reason=emergency_control、emergency_control_id，并关联 Installation 与控制审计 | CHECKED / CONFIRMED 再进入 APPLYING 前命中 ACTIVE 时转 CANCELLED 并拒绝执行 |
| 可选 break-glass：解除紧急隔离 | EmergencyControl=ACTIVE 且满足恢复条件 | 映射 lift 能力的主体发起，另一名映射 approve 能力的组织既有安全主体复核 | `ACTIVE → LIFTED`；lift_approved_by 与 lift_requested_by 不同；原子重算策略，desired_state 仍为 disabled / removed | 显示“隔离已解除，尚未恢复使用”及后续明确操作 | 解除请求、复核、重算和结果关联 control id / revision | 不自动启用或重装；成员自助或另一次获批组织恢复后才可使用 |
| 可选 break-glass：重叠紧急控制解除 | 同一目标命中两条 ACTIVE 控制 | 依次解除第一条和第二条 | 第一条 LIFTED 后仍保持安全限制；第二条 LIFTED、ACTIVE 归零后才解除限制并重算 Distribution，desired_state 不自动恢复 | 分别显示“仍受其他控制限制”和“限制已解除，尚未恢复使用” | 两次解除均记录 `remaining_active_control_ids` | 任一重算失败保持原安全限制；重试幂等 |
| 版本撤回与影响控制 | PUBLISHED Version 有 ACTIVE Distribution 和既有安装 | 有权主体撤回版本 | Version→WITHDRAWN，关联 Distribution→ENDED，重算安装锁；历史实例与证据保留 | 市场禁止新安装；IDE 标记撤回原因与替代 / 处置建议 | `version_withdrawn`、`distribution_changed` 与影响范围 | 同一版本不可恢复；替代版必须新建、审核和发布 |
| 同一安装状态跨端同步 | 分别准备 ready 与 installing 的 Installation，Web 持有较旧 revision | 模拟两者同步失败，再恢复 | ready 可合法进入 unknown；installing 保持原 actual_state，仅同步快照 `snapshot_stale=true`；旧 revision 不得覆盖 | 两者均标“状态待同步”，不把 installing 伪造成 unknown / 未安装 | `installation_sync_failed` / `installation_reconciled` 记录 revision、snapshot_stale 与恢复状态 | 重试读取同一 installation_id，按合法 transition 恢复并清理当前原因 |
| Installation schema 规划字段迁移 | 现有 schema 为 `additionalProperties:false`，尚无 reason / update snapshot 扩展字段 | 在未迁移与已迁移版本分别写扩展字段 | 未迁移请求被明确拒绝；完成 schema、存储与 API 兼容迁移后才可原子写入 | 客户端按兼容版本显示能力不可用或正常字段，不静默丢数据 | 迁移版本、拒绝原因和写入结果可审计 | 回滚客户端仍可读取兼容响应；不得绕过 schema 直接持久化 |
| Plugin 不可变证据不丢失 | 复合 Plugin 已发布并可见 | 查看详情、安装、配置和检查更新 | Manifest、组件、来源、许可、权限、checksum 始终关联同一 Release | 详情与 IDE 更新确认页展示一致证据；凭据从不随包展示 | 发布、安装和更新事件均关联相同 content / release 身份 | 刷新、跨端跳转或失败恢复后证据仍一致 |

## 附录 A：Plugin 内容类型规格

Plugin 是将多项能力作为一个 `Content(type=Plugin)` 统一发布、安装、配置、启停和更新的复合安装单元。内部组件默认不生成独立 Content、Release、Distribution 或 Installation；只有组件另行完成独立发布时，才进入主干定义的独立内容链路。

### A.1 包结构与创建最小集

| 层级 | 组成 | 自定义创建首批要求 | 作用 |
| --- | --- | --- | --- |
| 包元数据 | Manifest、名称、描述、版本、签发计划、标签、兼容性 | 必需 | 支撑市场展示、校验、安装、升级与追溯；Manifest 必须与实际文件和组件清单一致 |
| 核心能力 | 至少一个 Skill | 必需 | 定义 Plugin 要解决的问题、输入、步骤和输出 |
| 会话入口 | `@Plugin` 名称 | 必需 | 确保安装并启用后可以被明确调用 |
| 快捷入口 | Commands、Prompt 模板 | 可选 | 承载高频、固定任务或可编辑的起始输入 |
| 专业约束 | Rules | 可选；受监管或高风险场景建议提供 | 声明行为规范、格式、风险边界、人工复核和免责声明 |
| 外部连接 | MCP | 可选 | 对接知识库、业务系统、Git、Issue、设计等外部能力 |
| 生命周期自动化 | Hooks | 可选 | 在任务前后执行校验、审计、格式化或回调；必须披露触发时机 |
| 执行编排 | Agents | 可选 | 承载固定专业角色或多步骤协作 |
| 知识与交付 | Knowledge、References、Templates | 可选 | 提供资料、样例、报告模板和交付格式 |
| 安装配置 | Config Schema、依赖、权限声明 | 条件必需 | 使用外部连接、脚本、Hook、敏感数据或其他依赖时必须声明 |

现有 IDE `/create-plugin` 的首轮输入只收集要解决的问题、目标角色或行业、典型输入、期望输出，以及是否需要外部系统或资料。系统据此生成 Manifest、唯一 `@Plugin` 入口和至少一个 Skill；只有用户选择连接、自动化或受监管能力时，才继续收集 MCP、Hooks、Rules、权限和 Config Schema。高级组件不得被错误设成每个 Plugin 的创建门槛。

### A.2 组件清单与逐组件来源证据

每个冻结的 Plugin `ContentVersion` 都必须保存 `components[]`。每项至少包含 `component_id`、`type`、`name`、一句用途、入口或文件路径、权限与依赖摘要、可选 `config_schema_ref`，以及指向 `source_evidence[]` 的证据引用。详情页按 Skills、Commands、Rules、MCP、Hooks、Agents、Knowledge / References、Templates 分组展示实际存在的组件；空分组隐藏，长列表可以展开。MCP 额外展示连接对象与读写性质，Hook 展示触发时机，Rule 展示生效范围。

每项 `source_evidence[]` 至少记录：

- `component_id` 与该来源在组件中的 `purpose`；
- `source_ecosystem`、`upstream_owner_id`、`upstream_owner_name`；
- Git 来源的 repository URL、固定 tag / ref、commit、文件路径与 checksum；
- 非 Git 来源的 canonical URL 或资源 ID、可验证版本和不可变 digest；
- 适用许可证、第三方声明和允许的再分发或适配方式。

一个组件可以引用多项来源证据，同一上游承担 Skill、MCP 或模板等不同用途时也要逐项标明，不能合并成无法追溯的描述。Compose Plugin 组合多个生态来源时，必须以 `components[] + source_evidence[]` 逐组件记录上述字段；不得把多个生态、上游名称拼入主干单值字段，也不得任意选择一个组件来源充当包级来源。

包级 `ContentVersion.source_kind`、`source_ecosystem` 与 `upstream_owner_*` 只描述 Plugin 包本身可证明的直接创作来源，并随版本冻结：当前企业确实创作组合工作流时使用 `source_kind=NATIVE`、`source_ecosystem=EnterpriseOfficial` 和对应企业 upstream owner；InfCode 创作时才使用 `source_ecosystem=InfCode` 和 InfCode upstream owner。若包只是组合多个外部组件、没有可证明的单一包级上游，则使用 `source_kind=COMPOSED`，包级 `source_ecosystem` 与 `upstream_owner_*` 为空。市场按第 6.1 节从包级与组件级冻结证据派生 `source_facets`；最终 `Release` 仍有且只有一个独立的 `publisher_id` / `publisher_name`，来源 facets 不参与签发责任计算。

### A.3 依赖、权限、配置与不可变证据

Plugin 必须声明运行依赖、最低 InfCode 版本、目标环境兼容性，以及网络、文件、命令执行、外部数据读写和敏感操作权限。Manifest、`components[]`、`source_evidence[]`、Config Schema、依赖和权限声明都属于 `ContentVersion` 的冻结快照；审核通过后的 Release 还必须关联 artifact checksum、file manifest checksum、扫描、安装测试和行为评测证据，同一版本不得静默更换组件、来源、许可或权限。

MCP 服务、连接类型和配置需求可以随 Plugin 声明，但 token、API Key、Cookie、企业连接实例和其他凭据不得写入包、Manifest、跨端协议、日志或 Marketplace 事件。存在必填连接或配置时，安装落盘后进入 `configuration_required`；用户只在 IDE 或受控连接系统中选择或填写配置，校验成功后同一 Installation 才进入 `ready` 并默认启用。

Plugin 详情与 IDE 更新确认页必须展示当前页面所指 Release 的组件、用途、来源、许可、依赖、权限、兼容性和 checksum 证据。Mirror、Adapter 和 Compose 条目额外展示接入用途及其固定上游证据；已安装状态不得隐藏这些信息。每次进入详情、安装、更新或版本回退界面时，证据都绑定当时的“当前 Release”或“目标 Release”对应 `content_id + content_version_id + release_id`；Release 切换后必须随目标重新取证，不能沿用上一 Release 的证据。

## 附录 B：跨端协议

Marketplace Web 与 InfCode IDE 统一使用以下协议入口：

```text
vscode://infcode.marketplace/open
```

### B.1 参数与兼容规则

| 参数 | 规则 |
| --- | --- |
| `action` | 必填白名单，只允许 `install`、`create`、`use`、`manage`、`configure`、`profile`；其他值直接拒绝 |
| `content_id` | 新协议主键；`use` 必填，`manage` / `configure` 在未提供 `installation_id` 时必填；`install` 兼容期可携带但仅为与权威 intent 交叉校验的不可信提示 |
| `plugin_id` | 仅 Plugin 兼容期可用；服务端或 IDE 先解析为唯一 `content_id`。非 Plugin 携带该参数直接拒绝 |
| `content_id + plugin_id` | 可以同时传入，但两者必须映射到同一 Plugin Content；缺失映射、类型不符或值冲突时拒绝，不能任意选一个 |
| `source_page` | 必填的白名单页面键，不是 URL；只允许已注册枚举，用于恢复上下文、回流和问题追踪 |
| `prompt` | 仅供 `create` 或 `use` 打开一条最长 4096 字符的纯文本可编辑草稿；不得自动发送、执行或解释为 action、命令或嵌套 URI |
| `intent_id` | `install` 必填；IDE 只用它从服务端读取权威 InstallIntent，不信任 URL 中的其他安装事实 |
| `installation_id` | `manage` / `configure` 的可选精确定位提示；必须在 IDE 重新鉴权，并确认属于当前账号与对应 Content |
| `release_id` / `request_id` | 兼容期可接收的不可信提示；`install` 时如存在必须与服务端权威 InstallIntent 完全一致，否则拒绝 |
| `scope_type + scope_id` | 兼容期可接收但必须成对出现，只作待重校验的目标提示；Project 无效、关闭或无权限时拒绝或要求重选，不得降级为 User |

`install` 必须提供 `intent_id`；`create` 与 `profile` 不要求内容 ID；`use` 按表中规则提供 `content_id` 或 Plugin 兼容期可解析的 `plugin_id`；`manage` / `configure` 优先提供 `installation_id`，缺失时才以内容 ID 进入范围选择。协议不接受 `auto_send`，宿主只能以 `autoSend: false` 打开草稿；用户主动发送后才进入实际执行或创建流程。`idempotency_key`、权限快照和权威 target hint 只保存在服务端 InstallIntent 中，不作为 URL 参数或授权依据；兼容 URL 中的 request、release 和 scope 值也不能覆盖服务端事实。除上表明确允许的参数外，未知参数一律拒绝。

新客户端除 `action`、`source_page` 等通用路由参数外，Web→IDE `install` 的安装事实逐步只发送 `intent_id`，`manage` / `configure` 优先只发送 `installation_id`。兼容的 `release_id`、`request_id`、`scope_type`、`scope_id` 只有在当前 Web 与 IDE 的最低支持版本均已覆盖新协议，且遥测连续两个发布周期没有旧参数流量后才可移除；移除前仍按不可信提示接收和比对。

### B.2 IDE 重校验与安全边界

协议参数均为不可信提示。宿主只进行一次 URL 百分号解码，再按字段校验最大长度和字符集：action 与 source_page 必须命中白名单，ID 只接受规定的标识字符，prompt 仅作为纯文本。解码后的 prompt 不解析 action、命令、脚本或嵌套 URI。服务端与客户端不得记录完整深链或 prompt，只记录经最小化后的 action、白名单 source_page、对象 ID、结果和错误码。

新协议不把 Project、工作区路径、仓库信息或 target hint 写入深链；IDE 从当前本地上下文或服务端权威 InstallIntent 补全。兼容期收到成对的 `scope_type + scope_id` 时只用于比对或预选，不作为授权结论。InfCode 打开后必须重新校验登录身份、内容与 Release 可见性、当前 User / Project 环境、客户端版本、兼容性、依赖、权限、有效 Distribution 和组织策略。Project 不存在、已关闭或无权限时必须明确要求重新选择，不能静默降级为 User。校验失败时不得创建 Installation、执行操作或伪造成功状态，并应保留原 action、内容标识和可恢复上下文。

URL 中不得传 token、API Key、Cookie、企业连接实例、授权结果或任何其他凭据。需要外部连接时，协议只打开 `configure` 入口，凭据由 IDE 或受控连接系统收集和保存；Marketplace Web、协议日志和行为事件只记录不含秘密的配置状态或引用。

### B.3 InstallIntent 与唯一 Installation

`install` 必须复用第 8.2 节的 `InstallIntent` 字段和状态，不建立协议专属安装状态机：

```text
GENERATED → OPENED → ACCEPTED
          ↘ EXPIRED / FAILED
```

Web 生成或重新唤起 intent 时不创建 Installation。IDE 以当前登录身份和 URL 中的 `intent_id` 从服务端读取权威 InstallIntent，并逐项校验 `account_id`、`content_id`、`release_id`、`request_id`、target hint、有效期、权限快照和当前策略。URL 中兼容携带的 `content_id` / `plugin_id`、`release_id`、`request_id` 或成对 scope 与 intent 冲突，或 intent 过期、主体不符、映射不唯一、scope 参数只出现一个、任一校验失败时一律拒绝；URL 不能覆盖服务端值。

IDE 完成登录、目标、版本、权限、策略和兼容性重校验后，才以账号、权威 `content_id` 和 InstallationTarget 唯一键创建或复用 `actual_state=requested` 的 Installation，并返回包含 `installation_id` 的 `ACCEPTED` ack。只有该 ack 建立 Web→IDE intent 与真实 Installation 的关联；此前 Web 只能显示“等待 IDE 确认”，不能推断新增安装。未过期重试在服务端复用同一 `idempotency_key`，过期后才创建新 intent；既有 Installation 始终通过统一 Installation API 读取。

IDE 内部市场发起的主动安装不属于 Web 交接，可直接进入 IDE 安装确认而不创建 InstallIntent；但仍必须执行相同的登录、目标、版本、兼容性、权限和组织策略校验，生成并复用 `request_id` / `idempotency_key`，并遵守账号、`content_id`、InstallationTarget 唯一键及第 5.6 节状态转换，不能因此创建第二条 Installation。

### B.4 宿主动作与失败反馈

- `install`：打开待确认安装，成功承接后按第 12 章执行；
- `create`：打开会话并预填可编辑创建草稿，不自动发送；
- `use`：打开会话并预填内容名称、任务与当前工作区信息，不自动执行；
- `manage`：有 `installation_id` 时定位经鉴权的具体实例；没有时必须使用 IDE 当前完整 scope，若当前 scope 不完整或存在多个实例则打开范围选择器，不得任意选择；
- `configure`：有 `installation_id` 时定位经鉴权的待配置实例；没有时必须使用 IDE 当前完整 scope，若缺失或存在多个候选则打开范围选择器，不得默认第一项；
- `profile`：打开由 InfCode 维护的用户画像设置。

浏览器无法唤起、宿主能力缺失、用户未登录、目标无效或校验失败时，Web / IDE 必须明确显示失败原因，提供重试和复制不含凭据的操作信息，不得显示安装、创建或使用成功。Web 恢复焦点后重新读取统一状态；同步失败时保留最后确认值并显示“状态待同步”。

## 附录 C：P0 内容清单

P0 是已经确认的 12 个 Plugin 纵向样本，用于验证复合内容从来源、审核、发布到安装与使用的完整链路，不代表 Marketplace 只支持 Plugin，也不改写为虚构的多内容类型种子。独立 Skill、Rule、Prompt 和 Template 的首批清单不在本轮 P0 基线内，须由后续内容选品单独确认后进入各自的正式审核链路。

| Plugin | 接入方式 | 核心组件 | 主要来源或组成 | 当前 Preview 状态 |
|---|---|---|---|---|
| Plugin Creator | Original | Skill、Manifest 模板、校验器 | Agent Skills 规范 + InfCode 创建流程 | 已生成不可变 Preview 并通过 User / Project 安装矩阵；待会话宿主行为评测。 |
| Superpowers | Mirror | 14 个 Skills | Superpowers v6.1.1 / 固定 commit | 已保留 MIT 与来源声明并通过安装矩阵；未打包 Hook，可选遥测已单独披露。 |
| Context7 | Compose | Skill、MCP | Context7 2.2.5 | 已生成 Preview 并通过安装矩阵；待真实 MCP 限流和降级评测。 |
| 前端工程最佳实践 | Mirror | Skill、项目规则集 | Vercel React Best Practices / 固定 commit | 已保留 MIT 与固定来源并通过安装矩阵；待真实前端项目行为评测。 |
| UI 与无障碍审查 | Compose | Skill、Rule、报告模板 | InfCode Original + 可选 Browser/Figma | 已生成不再分发上游内容的 Preview 并通过安装矩阵；待连接与降级评测。 |
| 代码审查 | Original | Skill、Rule、Command、可选 GitHub | InfCode 工作流 + 可选连接 | 严重度、证据格式和读写权限已拆分；已通过安装矩阵。 |
| 测试生成 | Original | Skill、Command、框架识别规则 | InfCode 工作流 | 文件读取、写入和命令执行权限已拆分；已通过安装矩阵。 |
| 需求转测试 | Original | Skill、模板、可选 Issue 连接 | InfCode 工作流 | 通用输入与测试矩阵已定义，外部连接为可选；已通过安装矩阵。 |
| GitHub | Compose | 必需连接、Skill、Command | GitHub 平台连接 | 本地内容已通过安装矩阵；连接前保持 `configuration_required`，待真实 OAuth 验证。 |
| Figma | Compose | 必需连接、Skill、Rule | Figma 平台连接 | 读取与写入权限已拆分；连接前保持 `configuration_required`，待真实连接验证。 |
| 数据分析 | Original | Skill、报告模板、可选运行时 | InfCode 工作流 | 数据安全、大文件与不确定性规则已定义；已通过安装矩阵。 |
| 产品调研与 PRD | Original | Skill、Rule、模板、可选浏览器连接 | InfCode 工作流 | 事实、推断、决策和证据账本已定义；已通过安装矩阵。 |

以上状态仅说明 Preview 验证进度，不替代正式 Release 的来源许可、固定版本、checksum、扫描、安装隔离、行为评测、维护与撤回门槛。详细来源与组件证据按附录 A 保存，正式上架仍按第 6 章生成不可变 Release 并经过第 10 章审核。

## 附录 D：后续治理能力

以下十项属于后续治理规划，不纳入本 PRD 的 MVP 验收范围，也不能据此声称现有界面或服务已经具备相应能力：

1. **自定义 RBAC**：在现有 `Open / Custom` 与最小权限点上扩展组织角色模板、条件权限和职责分离策略。
2. **多级审批**：按内容风险、范围或金额引入串行 / 并行复核、会签和升级路径。
3. **私有镜像与隔离网络**：支持企业私有 artifact / source mirror、离线同步、网络分区和受控出网。
4. **灰度分发**：按受众比例、批次、时间窗与健康门槛逐步扩大 Distribution，不改变 Release 不可变性。
5. **分发编排与批量暂停**：在 MVP 单条 Distribution 的 `PAUSED` 之上提供跨内容、跨范围的批量暂停、恢复和策略编排。
6. **策略化自动版本回退**：以明确健康阈值、授权范围、目标 Installation、旧 Release 和审计证据自动执行 IDE 版本回退；它不等于 ContentVersion 撤回，也不等于 Distribution 暂停或结束。
7. **内容依赖图**：展示 Content、组件、Release、依赖、外部连接与受影响安装之间的可查询关系。
8. **企业质量评分**：基于审核、扫描、更新、运行质量和支持责任形成可解释评分，不以安装量代替质量。
9. **跨组织交换**：在独立授权、许可、结算和数据隔离规则下交换内容，不复用组织内可见性作为交易权限。
10. **供应商签名**：引入独立于 Marketplace `publisher_id` 的供应商数字签名、验证、轮换、吊销和信任链治理。
