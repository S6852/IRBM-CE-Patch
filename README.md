# IRBM - Combat Extended Compatibility

为 [InterRim Ballistic Missile](https://steamcommunity.com/sharedfiles/filedetails/?id=)（`kazepsi.irbm`）提供 [Combat Extended](https://steamcommunity.com/sharedfiles/filedetails/?id=2890901044) 环境下的平衡补丁。通过 XPath 调整弹头爆炸参数与 CIWS 对地攻击数值，不重写 IRBM 的 C# 逻辑。

## 依赖

- RimWorld 1.6
- Harmony (`brrainz.harmony`)
- Combat Extended (`CETeam.CombatExtended`)
- InterRim Ballistic Missile (`kazepsi.irbm`)

## 加载顺序

```
Harmony → Combat Extended → IRBM → 本补丁
```

`About.xml` 已声明 `loadAfter`，一般放在 IRBM 之后即可。

## 当前进度（MVP / 阶段 1）

| 内容 | 文件 | 状态 |
|------|------|------|
| 战术高爆弹头 `IRBM_Warhead_Tactical_HE` | `1.6/Patches/IRBM/Warheads_Tactical.xml` | 已 patch |
| 机炮 CIWS `IRBM_CIWS_Guns` | `1.6/Patches/IRBM/Buildings_CIWS.xml` | 已 patch |
| 激光 CIWS `IRBM_CIWS_Laser` | `1.6/Patches/IRBM/Buildings_CIWS.xml` | 已 patch |

其余弹头与陷阱补丁见 `docs/CE_COMPAT_PLAN.md` 后续阶段。

## 已知限制

- IRBM 爆炸无 CE 破片，靠提高 `damageAmount` / `armorPenetration` 补偿
- CIWS 未接入 CE 弹药与标准炮塔系统
- 战略弹头、特殊弹头尚未平衡

## 开发文档

- [开发计划](docs/CE_COMPAT_PLAN.md)
- [平衡记录](docs/BALANCE_LOG.md)
- [测试清单](docs/TEST_CHECKLIST.md)

**警告：这个模组还在开发和技术探索中！可能会出现平衡问题以及意料之外的bug！除非你真的需要这个补丁，否则不要下载**
