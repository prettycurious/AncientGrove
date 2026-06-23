# 《AncientGrove / 古林回响》项目开发文档

## 1. 项目基本信息

| 项目 | 内容 |
|---|---|
| 项目名称 | AncientGrove |
| 中文名称 | 古林回响 |
| 英文名称 | Echoes of the Ancient Grove |
| 项目类型 | 小型第三人称动作冒险 Demo |
| 开发引擎 | Unreal Engine 5.8 |
| 开发方式 | 蓝图优先，后期逐步引入 C++ |
| 目标平台 | macOS / Windows PC |
| 开发人员 | 个人开发 |
| 当前阶段 | v1.0 HUD 与 Boss 战血条已完成基础接线，继续整体流程测试和失败提示收尾 |
| 核心目标 | 完成一个 5~10 分钟可玩的第三人称动作冒险 Demo |

---

## 1.1 当前项目状态

更新时间：2026-06-23

当前项目已经完成敌人 AI 基础闭环和宝箱交互通关闭环。当前 Demo 已可以完成“击败敌人后打开宝箱并显示试炼完成”的基础流程。v1.0 收尾阶段正在进行 HUD 接线和输入接线校对。2026-06-18 排查发现：`BP_PlayerCharacter` 中 `IA_Attack` 左键攻击事件误接了交互失败提示逻辑，导致左键攻击时也会显示“附近没有可交互对象”。重新接线截图已校对通过：`IA_Attack` 现在只负责攻击，`IA_Interact` 负责宝箱交互和无交互对象提示。随后发现“附近没有可交互对象”显示后不会自动隐藏，已校对 `IA_Interact` 无效分支后的 `Delay -> Is Valid?(CurrentInteractChest) -> ClearHUDPrompt` 临时提示清理流程。玩家受伤后的 HUD 血量刷新问题也已修正：`BP_PlayerCharacter` 在扣减 `CurrentHP` 后调用 `RefreshHUDHealth`，`WBP_PlayerHUD` 已通过 `PlayerCharacterRef` 读取 `CurrentHP / MaxHP` 并刷新 `PB_PlayerHP` 和 `TXT_HPValue`。2026-06-23 已将敌人血量表现从“头顶血条”调整为“Boss 战 HUD 血条”：`WBP_PlayerHUD` 内新增 Boss 血条区域，`BP_PlayerCharacter` 负责转发显示、隐藏和刷新请求，`BP_AncientGuardian` 在玩家首次进入 `DetectRange` 且 Boss 未死亡时显示 Boss 血条，受伤后刷新，死亡后隐藏。同日还修正了 Boss 追踪分支：`AI MoveTo` 应接在 `AttackRange` 判断的 `False` 分支，表示玩家在检测范围内但未进入攻击范围时持续追踪；`CanAttack` 判断的 `False` 分支保持空置，用于等待攻击冷却。随后修复 HUD 底部信息重叠问题：`TXT_InteractPrompt` 上移到 `BossHealthBox` 上方，Boss 血条继续固定在底部中间。

已完成内容：

1. 已创建 UE 5.8 蓝图项目 `AncientGrove`；
2. 已创建主地图 `M_AncientGrove`；
3. 已创建 `BP_AncientGroveGameMode`，并设置默认 Pawn 为 `BP_PlayerCharacter`；
4. 已创建玩家角色 `BP_PlayerCharacter`，包含移动、跳跃、跑步、攻击输入和基础血量变量；
5. 已创建攻击输入和攻击动画 Montage；
6. 已创建敌人蓝图 `BP_AncientGuardian`；
7. 敌人已包含生命值、受伤、死亡、检测范围、攻击范围、攻击间隔、玩家引用等变量；
8. 敌人蓝图中已加入 `AI Move To` 追击节点，`Pawn` 使用 `Self`，`Target Actor` 使用 `PlayerRef`，`Acceptance Radius` 设置为 `80.0`；
9. 主地图中已放置 `BP_AncientGuardian`；
10. 主地图中已存在 `NavMeshBoundsVolume` 和 `RecastNavMesh`；
11. 已调整 `NavMeshBoundsVolume` 范围，使其覆盖敌人活动区域；
12. 玩家靠近敌人一定范围后，敌人已可以自动追踪玩家；
13. 敌人追上玩家后已可以主动攻击玩家；
14. 玩家被敌人攻击后，血量已可以正常扣减；
15. 已创建宝箱交互逻辑；
16. 玩家不靠近宝箱时按 `E` 会提示“附近没有可交互对象”；
17. 玩家靠近宝箱但敌人未死亡时按 `E` 会提示“请先击败遗迹守卫”；
18. 玩家击败敌人后靠近宝箱按 `E` 会提示“试炼完成”；
19. 宝箱打开后再次按 `E` 会提示“宝箱已经打开”；
20. v0.5 宝箱交互基础测试已通过；
21. 已创建 `Content/AncientGrove/UI/` 目录；
22. 已补充 `docs/v1.0_HUD实施步骤.md`，用于指导 `WBP_PlayerHUD` 创建和接线；
23. 已定位左键攻击误触发交互失败提示的问题；
24. 已根据截图校对 `BP_PlayerCharacter` 输入接线：`IA_Attack` 与 `IA_Interact` 职责已拆分；
25. 已校对 `IMC_Player`：`IA_Attack = Left Mouse Button`，`IA_Interact = E`；
26. 已定位“附近没有可交互对象”提示持续存在的问题，原因是显示后没有调用 `ClearHUDPrompt`；
27. 已根据截图校对 `IA_Interact` 的临时提示自动清理流程；
28. 已确认 `BP_PlayerCharacter` 中 `PlayerHUDRef` 创建、保存并用于调用 `UpdateHealth`；
29. 已确认 `WBP_PlayerHUD` 中 `PlayerCharacterRef` 有效，`UpdateHealth` 可读取玩家当前血量和最大血量；
30. 已验证玩家扣血后 HUD 血条和血量文本可以刷新；
31. 已撤销敌人头顶血条方案，`WBP_EnemyHealthBar`、`EnemyHealthBar` WidgetComponent 和 `UpdateEnemyHealthBar` 不再作为当前方案使用；
32. 已在 `WBP_PlayerHUD` 中新增 Boss 血条区域，包括 `BossHealthBox`、`TXT_BossName`、`SizeBox_BossHP` 和 `PB_BossHP`；
33. 已在 `WBP_PlayerHUD` 中新增 `ShowBossHealth`、`HideBossHealth` 和 `UpdateBossHealth`；
34. `UpdateBossHealth` 已使用 `Safe Divide(CurrentHP, MaxHP)` 再接 `Clamp(Float)`，避免 `MaxHP = 0` 时出现除零问题；
35. 已在 `BP_PlayerCharacter` 中新增 `ShowBossHealthOnHUD`、`UpdateBossHealthOnHUD` 和 `HideBossHealthOnHUD`，由玩家角色统一转发到 HUD；
36. 已在 `BP_AncientGuardian` 中通过 `PlayerRef` 调用玩家角色转发函数，避免 Boss 直接访问 `PlayerHUDRef`；
37. 已新增 `BossHealthShown` 布尔变量，用于保证 Boss 血条只在首次进入检测范围时显示一次；
38. 已校对 Boss 血条显示条件：`Distance <= DetectRange AND NOT BossHealthShown AND NOT IsDead`；
39. 已校对 Boss 受伤后调用 `UpdateBossHealthOnHUD` 刷新血条，死亡后调用 `HideBossHealthOnHUD` 隐藏血条；
40. 已更新 `docs/v1.0_敌人血条实施步骤.md`，当前内容改为 Boss 战 HUD 血条实施和校对步骤；
41. 已修正 Boss 追踪分支，`AI MoveTo` 现在归属于 `AttackRange = false` 的追踪路径；
42. 已明确 `CanAttack = false` 只表示攻击冷却中，不负责触发追踪；
43. 已修复 HUD 底部交互提示和 Boss 血条重叠问题，`TXT_InteractPrompt` 位于 `BossHealthBox` 上方。

当前状态：

```text
v0.5 宝箱交互基础目标已完成。
v1.0 收尾进行中。BP_PlayerCharacter 中 IA_Attack 与 IA_Interact 的接线已按截图校对通过，`IA_Interact` 无效分支的临时提示自动清理流程也已按截图校对通过。HUD 血量刷新链路已验证通过：玩家扣血后 `PB_PlayerHP` 和 `TXT_HPValue` 可以同步更新。敌人血量表现已改为 Boss 战 HUD 血条：开局不显示，玩家进入 `DetectRange` 后显示，Boss 受伤后刷新，Boss 死亡后隐藏。Boss 追踪和攻击分支已重新拆清：`DetectRange` 内先判断 `AttackRange`，在攻击范围外执行 `AI MoveTo`，在攻击范围内再判断 `CanAttack` 是否可以攻击。HUD 布局已避免底部交互提示和 Boss 血条重叠：交互提示显示在 Boss 血条上方。下一步需要在播放模式中完整测试显示时机、扣血刷新、死亡隐藏、持续追踪、HUD 不重叠和通关流程。
```

下一步执行顺序：

1. 保留当前已验证可用的 `NavMeshBoundsVolume` 和 `Acceptance Radius = 80.0` 设置；
2. 保存当前已验证通过的宝箱、玩家、敌人和地图资产；
3. 在 UE 中点击 编译（Compile）并确认 `BP_PlayerCharacter` 没有报错；
4. 点击 保存（Save）或 文件（File）-> 保存全部（Save All）；
5. 测试左键攻击不再显示“附近没有可交互对象”；
6. 测试不靠近宝箱按 `E` 时显示“附近没有可交互对象”，约 1.5 秒后自动隐藏；
7. 测试靠近宝箱按 `E` 时仍能打开或显示对应宝箱提示；
8. 测试按 `E` 后立刻走进宝箱范围时，不会把新的“按 E 打开宝箱”提示误清掉；
9. 按 `docs/v1.0_HUD实施步骤.md` 继续校对 `WBP_PlayerHUD` 的交互提示和结果提示；
10. 添加或验证玩家死亡后的失败提示；
11. 验证玩家血条在多次受击后持续刷新；
12. 在 UE 中点击 编译（Compile）并确认 `WBP_PlayerHUD` 没有报错；
13. 在 UE 中点击 编译（Compile）并确认 `BP_PlayerCharacter` 和 `BP_AncientGuardian` 没有报错；
14. 在播放（Play）模式中测试开局远离 Boss 时 Boss 血条不显示；
15. 靠近 Boss 进入 `DetectRange` 后确认 Boss 血条显示，并显示当前满血状态；
16. 攻击 Boss 后确认 `PB_BossHP` 会随 `CurrentHP` 下降；
17. 击败 Boss 后确认 Boss 血条隐藏；
18. 确认玩家离开追击范围后 Boss 血条是否继续显示；当前设计是进入 Boss 战后继续显示；
19. 测试玩家在 `DetectRange` 内但不在 `AttackRange` 内时，Boss 会持续执行 `AI MoveTo` 追踪；
20. 测试玩家进入 `AttackRange` 后，Boss 会按 `CanAttack` 和 `AttackInterval` 攻击；
21. 同时显示 Boss 血条和宝箱交互提示时，确认 `TXT_InteractPrompt` 不遮挡 Boss 名称和 `PB_BossHP`；
22. 进行一次从出生到通关的完整流程测试。

---

## 1.2 开发日志索引

每日开发记录按天拆分存放在 `docs/devlog/` 目录中。

| 日期 | 阶段 | 文档 |
|---|---|---|
| 2026-06-04 | 工程搭建 | [2026-06-04.md](devlog/2026-06-04.md) |
| 2026-06-05 | 输入、攻击动画、敌人原型 | [2026-06-05.md](devlog/2026-06-05.md) |
| 2026-06-08 | 玩家攻击和敌人受伤死亡 | [2026-06-08.md](devlog/2026-06-08.md) |
| 2026-06-09 | 敌人 AI 和 NavMesh 初步排查 | [2026-06-09.md](devlog/2026-06-09.md) |
| 2026-06-10 | 敌人 AI 闭环、宝箱交互原型和文档更新 | [2026-06-10.md](devlog/2026-06-10.md) |
| 2026-06-11 | 宝箱交互测试通过，HUD 收尾启动 | [2026-06-11.md](devlog/2026-06-11.md) |
| 2026-06-18 | 输入提示排查和 HUD 血量刷新验证 | [2026-06-18.md](devlog/2026-06-18.md) |
| 2026-06-19 | 项目文档引擎版本校对 | [2026-06-19.md](devlog/2026-06-19.md) |
| 2026-06-20 | HUD 提示清理、UI 目录和 UE 术语校对 | [2026-06-20.md](devlog/2026-06-20.md) |
| 2026-06-22 | Unreal MCP 复查和敌人头顶血条 | [2026-06-22.md](devlog/2026-06-22.md) |
| 2026-06-23 | Boss 战 HUD 血条、检测范围触发和蓝图校对 | [2026-06-23.md](devlog/2026-06-23.md) |

说明：

```text
2026-06-06 和 2026-06-07 暂未发现核心项目资产更新，因此暂不单独建当天记录。
后续如果补充了当天实际开发内容，可以继续追加对应日期文件。
```

---

## 2. 项目定位

《古林回响》是一款小型第三人称动作冒险 Demo。玩家将扮演一名进入古老森林遗迹的冒险者，在神秘林地中探索、战斗，并完成一场简单的遗迹试炼。

本项目不是一开始制作完整开放世界游戏，而是作为后续大型 3D 动作冒险游戏的基础原型。第一阶段重点验证第三人称角色控制、镜头、攻击、敌人 AI、血量系统、交互系统和小关卡闭环。

---

## 3. 项目目标

### 3.1 短期目标

完成一个小型可玩 Demo，包含以下内容：

1. 第三人称角色移动；
2. 镜头跟随与鼠标视角控制；
3. 跑步、跳跃基础动作；
4. 普通攻击；
5. 一个基础敌人；
6. 敌人追击和攻击玩家；
7. 玩家和敌人的血量系统；
8. 宝箱交互；
9. 击败敌人后打开宝箱；
10. 显示通关提示。

### 3.2 中期目标

在第一版 Demo 基础上，继续扩展：

1. 攻击动画优化；
2. 受击反馈；
3. 死亡动画；
4. 简单 UI；
5. 音效和打击反馈；
6. 小型关卡美化；
7. 简单任务提示；
8. 本地存档。

### 3.3 长期目标

作为未来制作大型 3D 动作冒险游戏的基础，逐步扩展为：

1. 多种敌人；
2. 多种武器；
3. 闪避、格挡、锁定系统；
4. 简单 Boss 战；
5. 道具收集；
6. 小型区域探索；
7. 简单剧情；
8. 多关卡结构。

---

## 4. 游戏背景设定

在一片被遗忘的古老森林中，隐藏着一座失落遗迹。传说遗迹深处存放着远古文明留下的神秘徽章，只有通过试炼的人才能打开遗迹宝箱。

玩家作为一名年轻冒险者，进入森林遗迹，面对守卫遗迹的古老生物。击败守卫后，玩家可以打开宝箱，获得遗迹徽章，并完成试炼。

第一版 Demo 不展开复杂剧情，只保留简单背景，用于支撑玩法闭环。

---

## 5. 核心玩法

### 5.1 核心玩法循环

游戏第一版核心循环如下：

```text
移动探索 → 发现敌人 → 战斗 → 击败敌人 → 打开宝箱 → 完成试炼
```

### 5.2 玩家目标

玩家需要在小型森林遗迹场景中：

1. 控制角色移动和探索；
2. 遭遇遗迹守卫；
3. 使用普通攻击击败敌人；
4. 靠近宝箱并按交互键打开；
5. 看到“试炼完成”提示。

### 5.3 第一版胜利条件

```text
敌人死亡数量达到要求
并且玩家成功打开宝箱
```

### 5.4 第一版失败条件

```text
玩家生命值降为 0
```

---

## 6. 第一版 Demo 功能范围

### 6.1 必须实现功能

| 模块 | 功能 | 说明 |
|---|---|---|
| 玩家角色 | 移动 | WASD 控制移动 |
| 玩家角色 | 跳跃 | Space 跳跃 |
| 玩家角色 | 跑步 | Left Shift 跑步 |
| 玩家角色 | 攻击 | 鼠标左键普通攻击 |
| 镜头 | 第三人称跟随 | 鼠标控制视角 |
| 敌人 | 追击玩家 | 玩家进入范围后敌人靠近 |
| 敌人 | 攻击玩家 | 进入攻击范围后造成伤害 |
| 血量 | 玩家血量 | 玩家受击扣血 |
| 血量 | 敌人血量 | 敌人受击扣血 |
| 死亡 | 敌人死亡 | 血量为 0 后死亡 |
| 交互 | 宝箱交互 | 按 E 打开宝箱 |
| 通关 | 胜利提示 | 打开宝箱后显示完成提示 |

### 6.2 暂不实现功能

第一版暂不实现以下内容：

1. 开放世界；
2. 多武器系统；
3. 背包系统；
4. 装备系统；
5. 复杂任务系统；
6. NPC 对话；
7. 联机功能；
8. 数据库；
9. 账号登录；
10. 商城系统；
11. 复杂剧情；
12. 高级画质；
13. 大型地图；
14. 复杂解谜；
15. 爬墙、滑翔、骑乘等高级动作。

---

## 7. 操作设计

| 操作 | 按键 |
|---|---|
| 前后左右移动 | W / A / S / D |
| 视角控制 | 鼠标移动 |
| 跳跃 | Space |
| 跑步 | Left Shift |
| 普通攻击 | 鼠标左键 |
| 交互 | E |
| 暂停菜单 | Esc，后续实现 |

---

## 8. 角色设计

### 8.1 玩家角色

玩家角色为第三人称动作角色，第一版只需要基础动作能力。

#### 基础属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---:|---|
| MaxHP | Float | 100 | 最大生命值 |
| CurrentHP | Float | 100 | 当前生命值 |
| WalkSpeed | Float | 500 | 普通移动速度 |
| SprintSpeed | Float | 800 | 跑步速度 |
| AttackDamage | Float | 25 | 普通攻击伤害 |
| IsAttacking | Boolean | false | 是否正在攻击 |
| IsDead | Boolean | false | 是否死亡 |

#### 玩家行为

1. 移动；
2. 跑步；
3. 跳跃；
4. 普通攻击；
5. 受击；
6. 死亡；
7. 与宝箱交互。

---

## 9. 敌人设计

### 9.1 敌人名称

```text
遗迹守卫
Ancient Guardian
```

### 9.2 敌人定位

第一版当前只有一个敌人，因此将其作为小型 Boss / 精英守卫处理，用于验证战斗系统、AI 追击逻辑和 Boss 战 HUD 血条。

### 9.3 敌人属性

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---:|---|
| MaxHP | Float | 100 | 最大生命值 |
| CurrentHP | Float | 100 | 当前生命值 |
| MoveSpeed | Float | 300 | 移动速度 |
| AttackDamage | Float | 10 | 攻击伤害 |
| DetectRange | Float | 800 | 发现玩家范围 |
| AttackRange | Float | 150 | 攻击范围 |
| AttackInterval | Float | 1.5 | 攻击间隔 |
| IsDead | Boolean | false | 是否死亡 |
| BossHealthShown | Boolean | false | Boss 血条是否已经显示 |

### 9.4 敌人行为

第一版敌人行为如下：

```text
待机 → 发现玩家 → 追击玩家 → 进入攻击范围 → 攻击玩家 → 死亡
```

Boss 战 HUD 显示逻辑：

```text
玩家首次进入 DetectRange
→ Boss 未死亡
→ BossHealthShown = false
→ 显示 Boss 血条
→ 刷新当前 Boss 血量
```

### 9.5 AI 简化方案

第一版可以先用蓝图实现简单 AI，不立即使用复杂 Behavior Tree。

基础逻辑：

1. 周期性检测玩家距离；
2. 如果玩家进入 DetectRange，则敌人追击；
3. 如果玩家进入 AttackRange，则敌人攻击；
4. 如果玩家首次进入 DetectRange，则通过 HUD 显示 Boss 血条；
5. 敌人血量为 0 时死亡，并隐藏 Boss 血条。

当前追踪和攻击分支约定：

```text
Distance <= DetectRange
→ Distance <= AttackRange
  → True：进入攻击分支，再判断 CanAttack
  → False：执行 AI MoveTo 追踪玩家
```

注意：

```text
CanAttack = false 只表示攻击冷却中，不应该接 AI MoveTo。
AI MoveTo 应接在 AttackRange 判断的 False 分支。
```

---

## 10. 战斗系统设计

### 10.1 玩家攻击

第一版玩家只有一种普通攻击。

攻击流程：

```text
鼠标左键输入
→ 判断当前是否可以攻击
→ 设置 IsAttacking = true
→ 播放攻击动画
→ 执行攻击判定
→ 命中敌人后造成伤害
→ 攻击结束
→ 设置 IsAttacking = false
```

### 10.2 攻击判定

第一版建议使用简单的 Sphere Trace 或 Box Trace。

判定逻辑：

1. 玩家攻击时，在角色前方生成一次检测；
2. 如果检测到敌人，则调用敌人的受伤逻辑；
3. 敌人扣除玩家攻击伤害；
4. 如果敌人血量小于等于 0，则进入死亡状态。

### 10.3 敌人攻击

敌人进入攻击范围后，按攻击间隔对玩家造成伤害。

攻击流程：

```text
敌人接近玩家
→ 判断距离是否小于 AttackRange
→ 播放攻击动作
→ 对玩家造成伤害
→ 等待攻击间隔
→ 再次攻击
```

---

## 11. 交互系统设计

### 11.1 宝箱交互

宝箱是第一版 Demo 的核心通关对象。

宝箱状态：

| 状态 | 说明 |
|---|---|
| 未开启 | 默认状态 |
| 可交互 | 玩家靠近时显示提示 |
| 已开启 | 玩家按 E 后打开 |
| 禁止开启 | 敌人未击败时不能打开 |

### 11.2 宝箱开启条件

```text
所有遗迹守卫已被击败
```

### 11.3 交互提示

玩家靠近宝箱时显示：

```text
按 E 打开宝箱
```

如果敌人未击败，显示：

```text
请先击败遗迹守卫
```

宝箱打开后显示：

```text
试炼完成
```

---

## 12. UI 设计

第一版 UI 保持简单，只实现必要信息。

### 12.1 玩家 HUD

包含：

1. 玩家血条；
2. 交互提示；
3. 通关提示；
4. 死亡提示；
5. Boss 战血条。

### 12.2 UI 文案

| 场景 | 文案 |
|---|---|
| 靠近宝箱 | 按 E 打开宝箱 |
| 敌人未击败 | 请先击败遗迹守卫 |
| 通关 | 试炼完成 |
| 玩家死亡 | 你已倒下 |
| Boss 名称 | 遗迹守卫 |

### 12.3 HUD 实施步骤

HUD 的具体创建、控件命名、蓝图函数和测试清单记录在：

```text
docs/v1.0_HUD实施步骤.md
```

Boss 血条的具体控件、蓝图函数、检测范围触发和测试清单记录在：

```text
docs/v1.0_敌人血条实施步骤.md
```

---

## 13. 关卡设计

### 13.1 第一版关卡名称

```text
M_AncientGrove
```

### 13.2 关卡结构

第一版关卡使用小型封闭区域，避免开放世界规模过大。

关卡区域包括：

1. 玩家出生点；
2. 林地空地；
3. 遗迹入口；
4. 敌人巡逻点；
5. 宝箱位置；
6. 简单边界阻挡。

### 13.3 关卡流程

```text
玩家出生
→ 向前探索
→ 遇到遗迹守卫
→ 进入战斗
→ 击败敌人
→ 靠近宝箱
→ 按 E 打开宝箱
→ 显示试炼完成
```

---

## 14. 技术路线

### 14.1 开发引擎

```text
Unreal Engine 5.8
```

### 14.2 开发方式

第一阶段采用：

```text
蓝图优先
```

后期根据需要逐步引入 C++。

### 14.3 开发环境

| 工具 | 用途 |
|---|---|
| Unreal Engine 5.8 | 游戏开发 |
| VS Code | 文档、配置、Git 辅助 |
| Xcode | macOS 编译工具链 |
| Git | 版本管理 |

### 14.4 技术模块

| 模块 | 实现方式 |
|---|---|
| 玩家控制 | UE Character + Enhanced Input |
| 镜头 | Spring Arm + Camera |
| 跑步 | 修改 CharacterMovement 的 Max Walk Speed |
| 攻击 | 输入事件 + 动画 Montage + Trace 判定 |
| 敌人 AI | 蓝图 + AI Move To + NavMesh |
| 血量系统 | 蓝图变量 + 伤害函数 |
| 宝箱交互 | 碰撞检测 + E 键交互 |
| UI | UMG Widget |
| 关卡 | Level + Static Mesh + Collision |

---

## 15. 项目目录规划

建议项目内容统一放在 `Content/AncientGrove/` 目录下。

```text
Content/
  AncientGrove/
    Blueprints/
      Character/
        BP_PlayerCharacter
      Enemy/
        BP_AncientGuardian
      Interact/
        BP_TreasureChest
      Game/
        BP_AncientGroveGameMode
    Input/
      IA_Attack
      IA_Sprint
      IA_Interact
      IMC_Player
    UI/
      WBP_PlayerHUD
    Maps/
      M_AncientGrove
    Animations/
    Materials/
    Audio/
    Props/
```

当前 v1.0 阶段只创建 `WBP_PlayerHUD`。交互提示和结果提示放在 `WBP_PlayerHUD` 内部控件中，分别使用 `TXT_InteractPrompt` 和 `TXT_ResultMessage`。`TXT_InteractPrompt` 放在底部中间但需要高于 Boss 血条，避免和 `BossHealthBox` 重叠。如果后续 UI 复杂度提高，再考虑拆分为独立的 `WBP_InteractPrompt` 或 `WBP_Result`。

当前 Boss 血条也放在 `WBP_PlayerHUD` 内部，不再使用独立的敌人头顶血条 Widget。当前相关控件为 `BossHealthBox`、`TXT_BossName`、`SizeBox_BossHP` 和 `PB_BossHP`。

---

## 16. 蓝图命名规范

| 类型 | 前缀 | 英文全称 | 示例 |
|---|---|---|---|
| 蓝图类 | BP_ | Blueprint | BP_PlayerCharacter |
| 关卡 | M_ | Map | M_AncientGrove |
| 输入动作 | IA_ | Input Action | IA_Attack |
| 输入映射 | IMC_ | Input Mapping Context | IMC_Player |
| UI 控件 | WBP_ | 控件蓝图（Widget Blueprint） | WBP_PlayerHUD |
| 材质 | MAT_ | Material | MAT_Stone |
| 贴图 | T_ | Texture | T_Stone_BaseColor |
| 音效 | SFX_ | Sound Effect | SFX_SwordHit |
| 动画 | A_ | Animation | A_PlayerAttack |
| 动画蒙太奇 | AM_ | Animation Montage | AM_PlayerAttack |

---

## 17. 版本计划

### v0.1 基础角色

目标：完成第三人称角色基础操作。

功能：

1. 创建 Third Person Blueprint 项目；
2. 创建项目目录；
3. 复制玩家角色蓝图；
4. 创建自己的地图；
5. 创建自己的 GameMode；
6. 调整移动参数；
7. 添加 Shift 跑步；
8. 添加鼠标左键攻击输入占位。

完成标准：

```text
玩家可以移动、跳跃、跑步，鼠标左键可以触发攻击 Print String。
```

---

### v0.2 攻击动画

目标：完成玩家普通攻击动作。

功能：

1. 导入攻击动画；
2. 创建攻击 Animation Montage；
3. 鼠标左键播放攻击动画；
4. 攻击期间禁止重复攻击；
5. 动画结束后恢复攻击状态。

完成标准：

```text
按鼠标左键后，角色可以播放一次完整攻击动画。
```

---

### v0.3 敌人原型

目标：创建可被攻击的敌人。

功能：

1. 创建 BP_AncientGuardian；
2. 添加敌人血量；
3. 添加受伤函数；
4. 玩家攻击命中敌人；
5. 敌人血量归零后死亡。

完成标准：

```text
玩家可以攻击敌人，敌人会扣血并死亡。
```

---

### v0.4 敌人 AI

目标：敌人可以追击和攻击玩家。

功能：

1. 添加 NavMesh；
2. 敌人发现玩家；
3. 敌人追击玩家；
4. 敌人进入攻击范围后攻击；
5. 玩家受击扣血。

当前进度：

| 项目 | 状态 | 说明 |
|---|---|---|
| NavMeshBoundsVolume | 已添加，已调整范围 | 当前范围已支持敌人追击玩家 |
| RecastNavMesh | 已存在 | 地图中已有导航数据对象 |
| 敌人实例 | 已放置 | 主地图中已有 `BP_AncientGuardian` |
| 敌人 AI 变量 | 已添加 | 包含 `DetectRange`、`AttackRange`、`AttackInterval`、`CanAttack`、`PlayerRef` |
| AI Move To | 已添加，已验证 | `Pawn=Self`，`Target Actor=PlayerRef`，`Acceptance Radius=80.0` |
| 追击行为 | 已验证 | 玩家在 `DetectRange` 内且不在 `AttackRange` 内时，敌人会通过 `AI MoveTo` 持续追踪玩家 |
| 攻击玩家 | 已验证 | 敌人追上玩家后可以主动攻击 |
| 玩家受击扣血 | 已验证 | 玩家被敌人攻击后血量可以扣减 |

v0.4 关键修正：

```text
将 AI Move To 的 Acceptance Radius 从跟随 AttackRange 的方式改为固定 80.0。
该调整让敌人追击时停得更近，从而可以稳定进入攻击逻辑。
```

后续可优化点：

1. 给敌人攻击增加动画表现；
2. 给玩家受击增加反馈；
3. 调整 `AttackRange`、`Acceptance Radius` 和胶囊体大小之间的手感；
4. 玩家血量归零后触发失败逻辑；
5. 后续可把攻击距离和移动停止距离整理成更清晰的变量。

完成标准：

```text
敌人可以主动追击玩家，并对玩家造成伤害。
```

---

### v0.5 宝箱交互

目标：实现通关闭环。

功能：

1. 创建 BP_TreasureChest；
2. 玩家靠近宝箱显示交互提示；
3. 按 E 尝试打开宝箱；
4. 判断敌人是否已击败；
5. 打开宝箱后显示试炼完成。

当前进度：

| 项目 | 状态 | 说明 |
|---|---|---|
| 宝箱交互对象 | 已实现 | 当前已能被玩家靠近并记录为当前交互对象 |
| `E` 键交互 | 已实现，已验证 | 不靠近宝箱时会提示“附近没有可交互对象” |
| 敌人未击败限制 | 已实现，已验证 | 靠近宝箱但敌人未死时会提示“请先击败遗迹守卫” |
| 敌人击败后开启 | 已实现，已验证 | 击败敌人后靠近宝箱按 `E` 会提示“试炼完成” |
| 重复开启保护 | 已实现，已验证 | 宝箱打开后再次按 `E` 会提示“宝箱已经打开” |

完成标准：

```text
玩家击败敌人后，可以打开宝箱并完成 Demo。
```

当前结论：

```text
v0.5 宝箱交互基础测试已通过。
后续可将 Print String 提示替换为正式 UMG HUD。
```

---

### v1.0 可玩 Demo

目标：完成一个 5~10 分钟可玩的完整小关卡。

功能：

1. 整理小型林地遗迹地图；
2. 放置敌人和宝箱；
3. 添加玩家血条；
4. 添加 Boss 战血条；
5. 添加基础音效；
6. 添加攻击命中特效；
7. 添加胜利和失败提示；
8. 完成整体测试。

完成标准：

```text
玩家可以从出生点开始，完成探索、战斗、开宝箱、通关的完整流程。
```

---

## 18. Git 版本管理

### 18.1 忽略文件

建议 `.gitignore` 内容如下：

```gitignore
# Unreal Engine
Binaries/
DerivedDataCache/
Intermediate/
Saved/
.vscode/
.vs/
*.VC.db
*.sln
*.xcodeproj
*.xcworkspace

# macOS
.DS_Store

# Logs
*.log

# Build
Build/
```

### 18.2 提交规范

建议每完成一个小功能提交一次。

示例：

```bash
git add .
git commit -m "init ancient grove project"

git add .
git commit -m "add player sprint input"

git add .
git commit -m "add attack input placeholder"

git add .
git commit -m "add enemy health system"

git add .
git commit -m "add chest interaction"
```

---

## 19. 当前开发优先级

当前优先级如下：

```text
第一优先级：角色能动
第二优先级：攻击能触发
第三优先级：敌人能受伤
第四优先级：敌人能追击
第五优先级：宝箱能交互（已完成基础验证）
第六优先级：形成完整通关流程（已完成基础验证）
第七优先级：补充 HUD、Boss 战血条、失败提示和整体测试
```

不优先考虑：

```text
画质
大地图
剧情
复杂 UI
联网
数据库
背包
多武器
商业化
```

---

## 20. 第一阶段完成标准

当以下内容全部完成时，第一阶段结束：

1. 玩家可以正常移动、跳跃、跑步；
2. 玩家可以普通攻击；
3. 敌人可以被攻击并死亡；
4. 敌人可以追击玩家；
5. 敌人可以攻击玩家；
6. 玩家有血量；
7. Boss 战开始后显示 Boss 血条；
8. 玩家死亡后显示失败提示；
9. 宝箱可以交互；
10. 击败敌人后打开宝箱；
11. 打开宝箱后显示“试炼完成”。

最终形成一个完整闭环：

```text
出生 → 探索 → 战斗 → 击败敌人 → 打开宝箱 → 通关
```

---

## 21. 后续扩展方向

第一版 Demo 完成后，可以考虑以下扩展：

### 21.1 战斗扩展

1. 连击系统；
2. 闪避；
3. 格挡；
4. 锁定目标；
5. 受击硬直；
6. 扩展 Boss 战阶段和技能。

### 21.2 探索扩展

1. 多个宝箱；
2. 钥匙和门；
3. 简单机关；
4. 小型解谜；
5. 道具拾取。

### 21.3 角色成长

1. 生命值提升；
2. 攻击力提升；
3. 技能解锁；
4. 武器切换。

### 21.4 工程扩展

1. C++ 基础类；
2. 通用交互接口；
3. 通用伤害系统；
4. 存档系统；
5. 配置表；
6. 简单主菜单。

---

## 22. 开发原则

本项目开发过程中遵循以下原则：

1. 先完成玩法闭环，再优化画面；
2. 先做最小功能，再做复杂系统；
3. 先用蓝图实现，再考虑 C++ 重构；
4. 每次只解决一个问题；
5. 每完成一个功能及时提交 Git；
6. 不过早制作开放世界；
7. 不过早引入后端服务；
8. 不过早追求商业化；
9. 优先保证能玩；
10. 优先保证项目持续推进。

---

## 23. 当前立即执行任务

当前立即执行以下任务：

1. 保存当前所有蓝图和关卡资产；
2. 保留当前已验证可追击和攻击的 `NavMeshBoundsVolume` 范围；
3. 保留 `AI Move To` 的 `Acceptance Radius = 80.0`；
4. 保留当前已验证通过的宝箱交互逻辑；
5. 保留当前已验证通过的 HUD 血量刷新逻辑；
6. 保留当前 Boss 血条的 HUD 方案：开局隐藏、进入 `DetectRange` 后显示、受伤刷新、死亡隐藏；
7. 将交互提示、通关提示从 `Print String` 逐步迁移到 HUD；
8. 添加或验证玩家死亡后的失败提示；
9. 验证玩家血条在连续受击和死亡边界下仍能正确显示；
10. 验证 Boss 血条在进入检测范围、受伤、死亡三个阶段表现正确；
11. 进行一次完整流程测试并记录结果。

当前任务完成标准：

```text
玩家可以看到基础 HUD 和 Boss 战血条，并从出生点完成战斗、开宝箱、胜利或失败提示的完整流程。
```
