# Stage 2B 交接文档：特效弹头的 CE 适配

> 交接对象：负责实现 `1.6/Patches/IRBM/Warheads_Tactical_Special.xml` 的 agent
> 前置阅读：本文档自包含，无需读完整对话历史；`CE_COMPAT_PLAN.md` 为总体规划。
> 状态基线：Stage 1（HE/Cluster/Inferno 等常规战术弹头）已完成并经游戏内验证生效。

## 1. 核心结论（已验证，请勿推翻重查）

以下结论均已通过反汇编 IRBM.dll、游戏内实验或日志分析验证：

1. **IRBM 的伤害管线**：弹头爆炸由 C# 类 `WorldObjectComp_TacticalPayload.ExecuteWithProps` 执行，分两段——爆心核心区逐目标直接 `TakeDamage`，随后调用原版 `GenExplosion.DoExplosion`。两条路径都原样传入 XML 中的 `armorPenetration` 字段。**XML 数值是生效的**，不存在被 C# 丢弃的问题。
2. **hediffs 绕过护甲**：payloadComps 里的 `hediffs` 列表由 `ApplyHediffConfigs` 直接施加到小人身上，不经过任何护甲结算。CE 无法拦截，这是设计意图。
3. **CE 的弹药注入**：CE 启动时（日志 `Combat Extended :: Ammo injected`）将原版炮弹 defName（`Shell_HighExplosive`、`Shell_Incendiary`、`Shell_Smoke` 等）**原地改造**为 81mm 弹药——defName 不变，标签和数值变。因此 IRBM 所有 costList **无需任何修改**，配方链自动完好（已截图验证：组装界面正确显示"81mm 迫击炮弹"）。
4. **CE 的 AP 刻度按护甲类别区分**：利器 = mm RHA（步枪弹约 10~25），钝器 = MPa（Bomb 类伤害走这条），**热能 = 原版式百分比**（Flame 等走这条）。给 AP 定值前必须先确认该 damageDef 的 armorCategory。
5. **不能给 def 添加 C# 类中不存在的字段**（如仿 CE 的 `armorPenetrationBlunt`），会在启动时报 XML error 且无代码消费它。

## 2. 各特效弹头的处理决定

| 弹头 | damageDef | 处理 | 理由 |
|---|---|---|---|
| Acid（强酸） | AcidBurn | **需要补丁**：加 armorPenetration | 见 §3 任务 1 |
| Inferno（地狱火，属 Stage 1 文件） | Flame | **需要修正**：AP 14 → 1.5 | 见 §3 任务 2 |
| Smoke（烟幕） | Smoke | 零改动 | 杀伤载荷是 BlindSmoke 气体，CE 不经手 |
| Tox（毒气） | Stun | 零改动 | 载荷是 ToxGas 气体 |
| Deadlife（死灵尘） | Stun | 零改动 | 载荷是 DeadlifeDust 气体 |
| Neural（心灵冲击） | Stun | 零改动 | Stun 不过甲；PsychicShock hediff 绕过护甲（结论 2） |
| Cryo（冷冻） | Frostbite | 零改动 | 无护甲交互痛点，80 点伤害保持原版 |
| Thermobaric（温压） | Vaporize | 零改动 | Vaporize 不走利/钝护甲管线；Stage 1 已调过伤害/半径 |
| Mine（布雷） | Smoke | 零改动 | 载荷是地雷生成 |
| Luciferium（魔鬼素） | Smoke | 零改动 | 载荷是 hediffs（结论 2） |

## 3. 待办任务（共 2 项）

### 任务 1：Acid 弹头添加 armorPenetration

- 目标文件：`1.6/Patches/IRBM/Warheads_Tactical_Special.xml`（当前是只含注释的占位文件）
- 源 def：`IRBM_Warhead_Tactical_Acid`，其 payloadComps 里**没有** `armorPenetration` 节点 → 必须用 `PatchOperationAdd`，不能用 Replace（Replace 找不到节点会报错）
- 该 def 只有一个 payloadComp，xpath 不需要 `[1]` 索引
- **数值前置校验**：先确认 `AcidBurn` 的 armorCategory（在原版/Odyssey 的 DamageDefs 里查 `<armorCategory>`，或游戏内对不同护甲实测）。热能类 → AP 写 1.5；利器类 → 按 mm 刻度写 15~20（酸弹定位是破甲，描述为 "melting through thick armor"）
- 补丁模板（沿用平铺 Operation 风格，禁止 FindMod / Sequence 包装，理由见 §4）：

```xml
<Operation Class="PatchOperationAdd">
  <xpath>Defs/IRBM.MissileWarheadDef[defName="IRBM_Warhead_Tactical_Acid"]/payloadComps/li[@Class="IRBM.WorldObjectCompProperties_TacticalPayload"]</xpath>
  <value>
    <armorPenetration>1.5</armorPenetration>
  </value>
</Operation>
```

### 任务 2：修正 Inferno 的 AP 刻度

- 目标文件：`1.6/Patches/IRBM/Warheads_Tactical.xml`（Stage 1 文件，已存在）
- 现状：`IRBM_Warhead_Tactical_Inferno` 的 armorPenetration 被 Stage 1 设成了 14
- 修改：14 → 1.5（Flame 走热能护甲，百分比刻度；14 无害但单位错误）
- 同时更新该 Operation 上方的注释

## 4. 项目约定（必须遵守）

1. **平铺 Operation，不用 PatchOperationFindMod / PatchOperationSequence**。CE 和 IRBM 已是 About.xml 硬依赖；FindMod 按模组显示名匹配（不是 packageId），写错会静默失效；Sequence 会吞掉单条报错。
2. **Replace 只用于已存在的节点，Add 只用于不存在的节点**，动手前对照源 def（`IRBM/1.6/Defs/MissileDefs/MissileWarhead.xml`）逐条核实。
3. **多 payloadComp 的 def 必须带 `[1]`/`[2]` 索引**（本次任务不涉及，但如扩展范围务必检查——MiniNuke、Antigrain、Suppression、Stonehenge、Milira_Plasma 都是双 payload）。
4. 每条 Operation 上方保留一行注释：原版值 → 补丁值 → 理由。

## 5. 验证清单

1. 启动游戏，日志中不得出现任何 `Patch operation ... failed` 或 XML error（平铺写法下每条操作独立报错，日志干净即全部命中）。
2. 开发者模式发射 Acid 弹头打中甲目标（钝/利中档、热能 <50% 的小人），应出现可观烧伤；对照原版行为差异。
3. Inferno 行为应与修改前无肉眼差异（1.5 与 14 在热能刻度下都远超封顶值）。
4. 其余特效弹头各发射一枚冒烟测试：气体正常扩散、hediff 正常上身、无红字。
5. 测试时的已知干扰：爆炸烟雾特效大小固定、不反映真实半径；集束弹头（clusterCount > 1）落点有散布。测半径用固定间隔的墙数缺口，或临时把 clusterCount 设 1。

## 6. 已知背景噪音（勿误判为本补丁问题）

- `[CE] Deflection ... has null verb, overriding AP` 警告：仅在利器→钝器偏转转换时触发，与 Bomb/爆炸类伤害无关；地图生成时的 Bite 伤害会周期性打出这条。
- `[IRBM] ...` 开头的日志：原模组作者遗留的调试输出（发布版带 pdb），每次发射必刷，正常现象。
- 补丁模组 About.xml 的 modDependencies 缺 steamWorkshopUrl 警告：纯提示，与功能无关（如需消除，给依赖项补 steamWorkshopUrl 即可）。
