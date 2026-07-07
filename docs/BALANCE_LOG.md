# Balance Log

记录每次数值调整与游戏内测试结果。每次只改 1–2 个 Def，测完再写下一条。

| 日期 | Def | 字段 | 旧值 | 新值 | 测试结果 | 备注 |
|------|-----|------|------|------|----------|------|
| — | `IRBM_Warhead_Tactical_HE` | damageAmount / AP / radius | 150 / 1.5 / 14.9 | 260 / 18 / 6.5 | 待测 | MVP 初始值 |
| — | `IRBM_Warhead_Tactical_Cluster` | damageAmount / AP / radius | 50 / 0.8 / 7.9 | 45 / 16 / 4.8 | 待测 | 阶段2A 初始值（25簇） |
| — | `IRBM_Warhead_Tactical_Inferno` | damageAmount / AP / radius | 80 / 1.0 / 9.9 | 95 / 14 / 5.8 | 待测 | 阶段2A 初始值 |
| — | `IRBM_Warhead_Tactical_Thermobaric` | damageAmount / radius | 80 / 20.9 | 130 / 7.6 | 待测 | 阶段2A 初始值 |
| — | `IRBM_Warhead_Tactical_Fission` | damageAmount / AP / radius | 200 / 75 / 12.9 | 220 / 75 / 9.0 | 待测 | 阶段2A 初始值 |
| — | `IRBM_Warhead_Tactical_MiniNuke` | damageAmount / AP / radius | 800 / 60 / 26.9 | 900 / 70 / 14.0 | 待测 | 阶段2A 初始值（3簇） |
| — | `IRBM_Warhead_Tactical_BunkerBuster` | damageAmount / AP / radius | 1500 / 800 / 5.9 | 1600 / 900 / 5.2 | 待测 | 阶段2A 初始值 |
| — | `IRBM_Warhead_Tactical_Antigrain` | payload[1] dmg/AP/r + payload[2] dmg/r | 600/75/24.9 + 180/35.9 | 700/90/12.0 + 180/18.0 | 待测 | 阶段2A 初始值（双payload） |
| — | `IRBM_Warhead_Tactical_Suppression` | payload[1] dmg/r + payload[2] dmg/r | 240/20.9 + 240/20.9 | 220/8.5 + 220/10.0 | 待测 | 阶段2A 初始值（双payload） |
| — | `IRBM_Warhead_Tactical_Stonehenge` | payload[1] dmg/AP/r + payload[2] dmg/r | 1200/800/15.9 + 240/35.9 | 1300/900/10.5 + 260/18.0 | 待测 | 阶段2A 初始值（双payload） |
| — | `IRBM_CIWS_Guns` | groundDamage* | 18 / 2.0 / int 3 | 32 / 30 / int 4 | 待测 | MVP 初始值 |
| — | `IRBM_CIWS_Laser` | groundDamage* | 64 / 2.0 | 52 / 20 | 待测 | MVP 初始值 |
