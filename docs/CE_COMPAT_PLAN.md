# IRBM × Combat Extended 兼容补丁 — 开发计划

## 0. 项目信息

| 项 | 值 |
|---|---|
| 目标模组 | InterRim Ballistic Missile (`kazepsi.irbm`) |
| 兼容目标 | Combat Extended Continued (`CETeam.CombatExtended`) |
| 游戏版本 | RimWorld 1.6 |
| 补丁类型 | 独立兼容模组（XML 平衡补丁为主） |
| 作者 | （填写你的名字） |
| 状态 | 规划阶段 |

### 项目目标

为 IRBM 在 CE 环境下提供**可玩且平衡**的体验，主要通过 XPath 调整爆炸伤害与 CIWS 对地攻击参数，**不**重写 IRBM 的 C# 逻辑。

### 非目标（第一版不做）

- 将 CIWS 改为 `Building_TurretGunCE`
- 为 CIWS 接入 CE 弹药系统
- 为弹头添加 CE 破片（`CompProperties_Fragments`）
- 修改导弹拦截、雷达、世界地图战略系统
- 向 CE 官方仓库提交 PR（可作为后续里程碑）

---

## 1. 技术背景摘要

### IRBM 伤害如何运作

| 内容类型 | Def / 类 | 伤害路径 | CE 补丁方式 |
|---------|-----------|---------|-------------|
| 战术/战略弹头 | `IRBM.MissileWarheadDef` → `WorldObjectCompProperties_TacticalPayload` / `StrategicPayload` | `GenExplosion.DoExplosion` + `DamageInfo` | XPath 改 `damageAmount`、`armorPenetration`、`explosionRadius` 等 |
| 近防炮对地攻击 | `ThingDef` + `IRBM.CIWSExtension` | `ApplyGroundDamage` → `DamageInfo` | XPath 改 `groundDamage*` 字段 |
| 近防炮反导 | `interceptionRange` / `interceptionEfficiency` | IRBM 自有系统 | **第一版不改** |
| 反粒子地雷 | `IRBM_TrapIED_Antigrain_NonRemovable` | 原版 `CompProperties_Explosive` | 第二版可选：换 `CompProperties_ExplosiveCE` |
| 雷达 / 发射井 / 卫星 | 非直接 CE 战斗 | — | **不需要补丁** |

### 关键限制

1. IRBM 弹头**不是** `ThingDef` 炮弹，不能套 `CompProperties_ExplosiveCE` / `ProjectilePropertiesCE`。
2. IRBM 近防炮**不是**标准炮塔，不能套 `PatchOperationMakeGunCECompatible` 或 `Building_TurretGunCE`。
3. CE 炮弹的杀伤很大程度来自**破片**；IRBM 爆炸没有破片，需用更高的 `damageAmount` + `armorPenetration` 弥补。
4. CE 护甲用 mm RHA（Sharp）体系；IRBM 原版 `armorPenetration` 1.5～2.0 对 CE 重甲几乎无效。

### CE 参照弹药（平衡锚点）

| CE 参照物 | damageAmountBase | explosionRadius | 备注 |
|-----------|------------------|-----------------|------|
| 81mm 迫击炮 HE | 156 | 2.5 | + 破片 16大+25小 |
| 120mm 迫击炮 HE | 237 | 3.5 | + 破片 40大+50小 |
| 155mm 榴弹炮 HE | 546 | 5.5 | + 破片 40大+80小 |
| 战斧巡航导弹 | 5653 | 19.5 | + 破片 |
| 20x138mmB AP（机炮弹） | 43/发 | — | Sharp AP 34 |

**战术 HE 弹头对标基准：** 约等于「2 枚 120mm HE」（`clusterCount=2`），但无破片，单发 `damageAmount` 建议 220–280，`explosionRadius` 缩小到 5–7。

---

## 2. 推荐文件夹结构

```
IRBM-CE-Compatibility/          # 你的补丁模组根目录
├── About/
│   └── About.xml
├── LoadFolders.xml             # 可选，若只做 1.6
├── 1.6/
│   └── Patches/
│       └── IRBM/
│           ├── Warheads_Tactical.xml      # 阶段 1
│           ├── Warheads_Tactical_Special.xml  # 阶段 2
│           ├── Warheads_Strategic.xml     # 阶段 3
│           ├── Buildings_CIWS.xml         # 阶段 1
│           └── Buildings_Traps.xml        # 阶段 4（可选）
├── docs/
│   ├── CE_COMPAT_PLAN.md         # 本计划
│   ├── BALANCE_LOG.md            # 测试记录与数值迭代
│   └── TEST_CHECKLIST.md         # 测试清单
└── README.md                     # 简短说明与加载顺序
```

### About.xml 要点

```xml
<ModMetaData>
  <name>IRBM - Combat Extended Compatibility</name>
  <packageId>你的名字.irbm.ce</packageId>
  <supportedVersions><li>1.6</li></supportedVersions>
  <description>CE balance patch for InterRim Ballistic Missile.</description>
  <modDependencies>
    <li>
      <packageId>kazepsi.irbm</packageId>
      <displayName>InterRim Ballistic Missile</displayName>
    </li>
    <li>
      <packageId>CETeam.CombatExtended</packageId>
      <displayName>Combat Extended</displayName>
    </li>
  </modDependencies>
  <loadAfter>
    <li>brrainz.harmony</li>
    <li>CETeam.CombatExtended</li>
    <li>kazepsi.irbm</li>
    <!-- 若有 IRBM 扩展包，列在 irbm 之后 -->
  </loadAfter>
</ModMetaData>
```

---

## 3. 开发阶段

### 阶段 0：环境准备（0.5 天）

- [ ] 本地安装 IRBM、CE 及依赖（Harmony 等）
- [ ] 确认加载顺序：Harmony → CE → IRBM → **本补丁**
- [ ] 开启 Dev Mode，准备测试地图
- [ ] 准备测试用小人：裸奔 / 防弹衣 / 动力甲或海军甲
- [ ] 创建 `BALANCE_LOG.md` 记录每次改动与测试结果

### 阶段 1：最小可玩补丁 MVP（1–2 天）

**目标：** 验证补丁框架可用，解决最明显的 CE 失衡。

**包含内容：**

1. **战术高爆弹头** `IRBM_Warhead_Tactical_HE`
2. **机炮 CIWS** `IRBM_CIWS_Guns`
3. **激光 CIWS** `IRBM_CIWS_Laser`

**建议初始数值（待实测调整）：**

| Def | 字段 | 原版 | CE 建议起点 |
|-----|------|------|-------------|
| `IRBM_Warhead_Tactical_HE` | damageAmount | 150 | 260 |
| | armorPenetration | 1.5 | 18 |
| | explosionRadius | 14.9 | 6.5 |
| `IRBM_CIWS_Guns` | groundDamageAmount | 18 | 32 |
| | groundDamageArmorPenetration | 2.0 | 30 |
| | groundDamageInterval | 3 | 4 |
| `IRBM_CIWS_Laser` | groundDamageAmount | 64 | 52 |
| | groundDamageArmorPenetration | 2.0 | 20 |
| | groundDamageType | Beam | Beam（或试 CE_Laser） |

**验收标准：**

- [ ] 游戏启动无 XML 报错
- [ ] 战术 HE 对动力甲小人能造成可见伤害（非完全刮痧）
- [ ] 战术 HE 对裸奔小人不会一发清图（若过强则降 damage 或 radius）
- [ ] CIWS 强制对地攻击能杀伤 CE 重装
- [ ] CIWS 反导功能仍正常

### 阶段 2：战术弹头全覆盖（2–3 天）

按子类分批 patch，每批完成后测试。

#### 2A — 爆炸类战术弹头（高优先级）

| defName | 原版 damage | 原版 AP | 原版 radius | cluster | 补丁策略 |
|---------|------------|---------|-------------|---------|----------|
| `IRBM_Warhead_Tactical_HE` | 150 | 1.5 | 14.9 | 2 | 对标 120mm×2 |
| `IRBM_Warhead_Tactical_Cluster` | 50 | 0.8 | 7.9 | 25 | 提高 AP，控 radius，注意 25 簇总伤害 |
| `IRBM_Warhead_Tactical_Inferno` | 80 | 1.0 | 9.9 | 10 | 保留 Flame，适度提 damage |
| `IRBM_Warhead_Tactical_Thermobaric` | 80 | — | 20.9 | 10 | Vaporize，提 damage，缩 radius |
| `IRBM_Warhead_Tactical_Fission` | 200 | 75 | 12.9 | — | AP 可能已够，测后再调 |
| `IRBM_Warhead_Tactical_MiniNuke` | 800 | 60 | 26.9 | 3 | 对标小型战术核，重点测 |
| `IRBM_Warhead_Tactical_BunkerBuster` | 1500 | 800 | 5.9 | — | 穿甲弹，先实测再动 |
| `IRBM_Warhead_Tactical_Antigrain` | 600+180 EMP | 75 | 24.9+35.9 | 3 | 两个 payload，用 `[1]` `[2]` 分别 patch |
| `IRBM_Warhead_Tactical_Suppression` | 240+240 | — | 20.9 | 5 | Stun+EMP 双 payload |
| `IRBM_Warhead_Tactical_Stonehenge` | 1200+240 | 800 | 15.9+35.9 | — | 巨石阵跨射弹头 |

#### 2B — 特殊效果类（中优先级）

| defName | 主要机制 | 补丁重点 |
|---------|----------|----------|
| `IRBM_Warhead_Tactical_Acid` | AcidBurn | 适度提 damage |
| `IRBM_Warhead_Tactical_Cryo` | Frostbite | 适度提 damage |
| `IRBM_Warhead_Tactical_Neural` | Stun + PsychicShock | 主要测 hediff 是否生效 |
| `IRBM_Warhead_Tactical_Smoke` | 烟雾 | 伤害低，可不动 |
| `IRBM_Warhead_Tactical_Tox` | ToxGas | 测毒气 + Biotech |
| `IRBM_Warhead_Tactical_Deadlife` | DeadlifeDust | 测 Anomaly DLC |
| `IRBM_Warhead_Tactical_Mine` | 布雷 | 地雷爆炸另见阶段 4 |
| `IRBM_Warhead_Tactical_Luciferium` | 气体+hediff | 伤害低，可不动 |

#### 2C — 不纳入第一版 CE 平衡

| defName | 原因 |
|---------|------|
| `IRBM_Warhead_AntiAir_*`（3个） | 反导弹头，不打地面 pawn |
| `IRBM_Warhead_Recon` | 侦察卫星，无伤害 |
| `IRBM_Warhead_*Satellite*`（4个） | 卫星部署，伤害在别的系统 |
| `IRBM_Warhead_Tactical_Milira_Plasma` | 联动弹头，需 Milira + 单独 patch |

### 阶段 3：战略弹头（1–2 天，低优先级）

战略弹头伤害极高，CE 下可能仍然过强。策略：**先实测，再小幅调整**，避免过度削弱核威慑体验。

| defName | 备注 |
|---------|------|
| `IRBM_Warhead_Strategic_Standard` | damage 10000，重点测 |
| `IRBM_Warhead_Strategic_Clean` | damage 500×10 次 |
| `IRBM_Warhead_Strategic_Surgical` | damage 3000×10 |
| `IRBM_Warhead_Strategic_Collapse` | damage 2000×30 |
| `IRBM_Warhead_Strategic_GammaRay` | damage 100×300，仅生物 |
| `IRBM_Warhead_Strategic_SolarFlare` | EMP×20 |
| `IRBM_Warhead_Strategic_CloudSeeding` | 无伤害 |
| `IRBM_Warhead_Strategic_Thunderstorm` | Stun×20 |
| `IRBM_Warhead_Strategic_DeepFreeze` | Frostbite×120 |
| `IRBM_Warhead_Strategic_BloodRain` | 天气，Anomaly |
| `IRBM_Warhead_Strategic_Lava` | Flame×120，Odyssey |
| `IRBM_Warhead_Strategic_Vacuum` | 天气，Odyssey |

### 阶段 4：附属内容与抛光（可选）

- [ ] `IRBM_TrapIED_Antigrain_NonRemovable` → 评估是否换 `CompProperties_ExplosiveCE`
- [ ] CIWS `fillPercent` / `MaxHitPoints` 微调
- [ ] `IRBM_CIWS_Stonehenge` 对地伤害（通常已够强，低优先）
- [ ] 中/英文 Keyed 说明："本补丁为 CE 平衡向"
- [ ] 与 IRBM 扩展包（史实弹体等）的 `loadAfter` 兼容测试

### 阶段 5：发布（可选）

- [ ] 整理 README：依赖、加载顺序、已知限制
- [ ] 上传 Steam Workshop 或 GitHub
- [ ] 向 IRBM 作者告知 / 征求合并
- [ ] 考虑向 CE 官方提 Patch Request

---

## 4. XPath 模板库

### 4.1 战术弹头（单 payload）

```xml
<Operation Class="PatchOperationFindMod">
  <mods><li>CETeam.CombatExtended</li></mods>
  <match Class="PatchOperationSequence">
    <operations>
      <li Class="PatchOperationReplace">
        <xpath>Defs/IRBM.MissileWarheadDef[defName="IRBM_Warhead_Tactical_HE"]/payloadComps/li[@Class="IRBM.WorldObjectCompProperties_TacticalPayload"]/damageAmount</xpath>
        <value><damageAmount>260</damageAmount></value>
      </li>
      <li Class="PatchOperationReplace">
        <xpath>Defs/IRBM.MissileWarheadDef[defName="IRBM_Warhead_Tactical_HE"]/payloadComps/li[@Class="IRBM.WorldObjectCompProperties_TacticalPayload"]/armorPenetration</xpath>
        <value><armorPenetration>18</armorPenetration></value>
      </li>
      <li Class="PatchOperationReplace">
        <xpath>Defs/IRBM.MissileWarheadDef[defName="IRBM_Warhead_Tactical_HE"]/payloadComps/li[@Class="IRBM.WorldObjectCompProperties_TacticalPayload"]/explosionRadius</xpath>
        <value><explosionRadius>6.5</explosionRadius></value>
      </li>
    </operations>
  </match>
</Operation>
```

### 4.2 多 payload 弹头（用序号）

```xml
<!-- 第一个 payload -->
/xpath/.../payloadComps/li[@Class="IRBM.WorldObjectCompProperties_TacticalPayload"][1]/damageAmount
<!-- 第二个 payload -->
/xpath/.../payloadComps/li[@Class="IRBM.WorldObjectCompProperties_TacticalPayload"][2]/damageAmount
```

### 4.3 战略弹头

将 `TacticalPayload` 换为 `StrategicPayload`：

```xml
/xpath/Defs/IRBM.MissileWarheadDef[defName="IRBM_Warhead_Strategic_Standard"]/payloadComps/li[@Class="IRBM.WorldObjectCompProperties_StrategicPayload"]/damageAmount
```

### 4.4 CIWS 对地伤害

```xml
<xpath>Defs/ThingDef[defName="IRBM_CIWS_Guns"]/modExtensions/li[@Class="IRBM.CIWSExtension"]/groundDamageAmount</xpath>
```

---

## 5. 平衡调参原则

1. **先测再改**：每次只改 1–2 个弹头或 1 座 CIWS，记录结果。
2. **保持相对梯度**：HE < 温压 < 小型核弹 < 战略核，不要压扁层级。
3. **无破片补偿**：爆炸类弹头 AP 通常需 15–30（常规 HE）或更高（穿甲）。
4. **半径与伤害成反比**：原版 radius 很大时，CE 下应缩小 radius 或依赖 `damageFalloff`。
5. **clusterCount 是乘数**：`Cluster`（25 簇×50 伤害）总输出极高，单项数值要保守。
6. **带 `ignoreDamageMultipliers` 的弹头**：穿甲弹/巨石阵/反粒子，先实测再决定是否动。
7. **反导与对地分开评**：`interceptionEfficiency` 不属于 CE 平衡范围。

### 粗算公式

```
有效爆炸火力 ≈ damageAmount × clusterCount × 半径系数
CIWS DPS ≈ (60 / groundDamageInterval) × groundDamageAmount（若持续开火）
```

---

## 6. 测试清单（`TEST_CHECKLIST.md`）

### 启动测试

- [ ] 无红色 XML 报错
- [ ] IRBM + CE + 本补丁同时启用
- [ ] 导弹组装、发射流程正常

### 弹头测试（每个弹头至少测一次）

- [ ] 对裸奔小人：有伤害，不过度秒杀全图
- [ ] 对动力甲/海军甲小人：有实质伤害（非完全免疫）
- [ ] 对建筑：伤害合理
- [ ] 特殊效果（火/毒/EMP/眩晕）仍触发

### CIWS 测试

- [ ] 机炮/激光 强制对地：能杀 CE 重装
- [ ] 反导：仍能拦截来袭导弹
- [ ] 激光 `Beam` vs `CE_Laser`：选效果正常者

### 集成测试

- [ ] AI 派系战术导弹袭击（`IRBM_AIFactionTacticalMissileStrike`）
- [ ] 与常见 CE 派系/护甲模组共存
- [ ] 有 IRBM 扩展包时加载顺序正确

---

## 7. 已知限制与后续方向

| 限制 | 说明 | 可能解法 |
|------|------|----------|
| 无 CE 破片 | 爆炸对分散重甲步兵偏弱 | 提高 AP；或 C# 生成破片 |
| CIWS 无 CE 弹道 | 无曳光弹/压制/掩体交互 | 接受；或作者合作重写 |
| 燃料非 CE 弹药 | 钢板/铀装填 | 保持原版；或第二版 C# |
| 战略弹头难精确平衡 | 伤害量级过大 | 以体验为准，少做数学对齐 |
| 联动弹头 | Milira 等 | 单独 patch 文件 + MayRequire |

---

## 8. 参考资源

- [CE Compatibility Patch Guide](https://github.com/CombatExtended-Continued/CombatExtended/wiki/Compatibility-Patch-Guide)
- IRBM 源码包：`kazepsi.irbm`，`packageId` 见 `About/About.xml`
- CE 弹药参照：`Combat Extended/Defs/Ammo/Shell/81mmMortar.xml`、`120mmMortar.xml`、`155mmHowitzer.xml`
- IRBM 关键 Def：
  - `1.6/Defs/MissileDefs/MissileWarhead.xml`
  - `1.6/Defs/ThingDefs_Buildings/Buildings_CIWS.xml`
  - `1.6/Defs/ThingDefs_Buildings/Buildings_Traps.xml`

---

## 9. 建议开发顺序（一览）

```
阶段0 环境
  ↓
阶段1 MVP（战术HE + 两座CIWS）
  ↓
阶段2A 爆炸类战术弹头
  ↓
阶段2B 特殊效果战术弹头
  ↓
阶段3 战略弹头（按需）
  ↓
阶段4 地雷/抛光/发布
```
