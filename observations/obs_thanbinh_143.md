# Observation: "Thần Binh" (ShenBing) — SystemOrder ID=143, Order=998

## Summary of Issues

| # | Issue | Severity |
|---|-------|----------|
| 1 | VersionSystemOpen IsOpen="0" on client, IsOpen="1" on server | High |
| 2 | IsHot="1" on client, IsHot="-1" on server (SystemOrder) | Medium |
| 3 | LinkID=16 points to "NV Ngày-Đấu Trường Quỷ" — valid but unexpected link | Low / verify |

---

## System Entry

| Field | Client | Server |
|-------|--------|--------|
| SystemOrder ID | 148 (bug: should be 143, but wait — the *entry* is at ID=143) | same |
| SystemOrder ID | 143 | 143 |
| Order | 998 | 998 |
| Name | Thần Binh | Thần Binh |
| ImageOne | ShenBing | ShenBing |
| JieMianShunXu | 38 | 38 |
| LinkID | 16 | 16 |
| IsHot | **"1"** | **"-1"** ← mismatch |

**SystemOpen** (both sides identical):
```xml
<System Order="998" ID="143" TriggerCondition="1" TimeParameters="3,1"
  ImageOne="chongsheng_1.png" ImageTwo="chongsheng_2.png"
  Name="Thần Binh" Description="Thần Binh kỳ tích, Vương Giả giáng lâm"
  NotOpenShow="-1" DongHua="-1" />
```
Opens at: rebirth level 3, character level 1 (TimeParameters="3,1").

---

## Issue 1 — VersionSystemOpen mismatch

| ID | Name | Client IsOpen | Server IsOpen |
|----|------|---------------|---------------|
| 100143 | 16.0.0 Shen Bing / 16.0.0-神兵 | `0` | `1` |

Feature is live on server, locked on client.

**Fix**: Set `IsOpen="1"` for ID=100143 in `gameres_config_android/VersionSystemOpen.txt`.

---

## Issue 2 — IsHot mismatch

Client `SystemOrder.txt`: `IsHot="1"` — shows HOT badge on feature icon.
Server `SystemOrder.xml`: `IsHot="-1"` — no badge.

**Fix**: Sync to one canonical value.

---

## Issue 3 — LinkID=16

`LinkID="16"` → SystemOpen ID=16 = **"NV Ngày-Đấu Trường Quỷ"** (Daily Activity - Devil Arena).

Unlike Tế Đàn Vạn Linh's LinkID=41 (broken reference), this ID **exists** in SystemOpen. However, the semantic connection between "Thần Binh" and "Đấu Trường Quỷ" is not obvious — verify whether this prerequisite/link is correct.

---

## All Config References (Verified)

| File | Side | Reference | Notes |
|------|------|-----------|-------|
| `SystemOpen.txt/xml` | Both | ID=143, Order=998 | System definition. Identical both sides. |
| `SystemOrder.txt` | Client | ID=143, Order=998, IsHot="1" | IsHot mismatch |
| `SystemOrder.xml` | Server | ID=143, Order=998, IsHot="-1" | IsHot mismatch |
| `VersionSystemOpen.txt` | Client | ID=100143, IsOpen="0" | Closed on client |
| `VersionSystemOpen.xml` | Server | ID=100143, IsOpen="1" | Open on server |
| `NewRedPoint.txt/xml` | Both | 6 entries, OrderID=998 | Identical both sides |
| `ShenQi.txt/xml` | Both | Core level progression | 2003 entries server, 2005 client (+2 = asset header) |
| `ShenQiRules.txt/xml` | Both | Rules | 7/9 lines (+2 header) |
| `ShenQiShengJie.txt/xml` | Both | Ascension data | 15/17 lines (+2 header) |
| `ShenQiSkill.txt/xml` | Both | Skill data | 803/805 lines (+2 header) |
| `ShenQiSkillUnlock.txt/xml` | Both | Skill unlock | 19/21 lines (+2 header) |
| `ShenQiType.txt/xml` | Both | Types: HP/Phòng/Công/Lôi | 7/9 lines (+2 header) |
| `ShenQiUnlock.txt/xml` | Both | Unlock conditions | 15/17 lines (+2 header) |
| `SystemParams.txt/xml` | Both | 3 ShenBing params | Identical both sides |
| `Goods.xml` / `GoodVO` | Both | Items 23001–23016 | Identical both sides |

**Note on naming**: The client calls this system `ShenBing` (神兵) in SystemOrder/ImageOne; the feature config files are named `ShenQi` (神器) on both sides. Same system, two internal names.

---

## SystemParams Entries

Three params in `SystemParams` (identical on both sides):

| Param | Value | Meaning |
|-------|-------|---------|
| `ZhanShiShenBing` | `2020900–2020911` | Archer class: ShenBing weapon model items (Nỏ Đại Thiên Sứ Lv1–12) |
| `ZhanShiShenBingFuTi` | `2021000–2021011` | Archer class: ShenBing possession buff items |
| `ZhuHunSheDing` | `10,90,100,0.05,2,0.01,5` | Soul infusion settings: 1x cost, 10x cost, base exp, small crit rate, small crit mult, large crit rate, large crit mult |

---

## GoodVO Items (23001–23016)

Functionally linked via ShenQiSkillVO and ShenQiUnlockVO referencing item IDs 23001–23016 directly (verified by table scan). Item definitions are identical between `GoodVO` (SQL) and `Goods.xml` (server).

| ID Range | Type | Purpose |
|----------|------|---------|
| 23001–23003 | HP | Unlock + level up HP Thần Binh (sơ/trung/cao) |
| 23004–23006 | Phòng | Unlock + level up Defense Thần Binh |
| 23007–23009 | Công | Unlock + level up Attack Thần Binh |
| 23010–23012 | Lôi | Unlock + level up Crit Thần Binh |

missing...
| 23013 | Thiên Phú HP | Talent stone for HP Thần Binh | missing
| 23014 | Thiên Phú Phòng | Talent stone for Defense Thần Binh |
| 23015 | Thiên Phú Công | Talent stone for Attack Thần Binh |
| 23016 | Thiên Phú Lôi | Talent stone for Crit Thần Binh |

Unlike Tế Đàn Vạn Linh (description-only link), items 23001–23016 are **functionally referenced** in `ShenQiSkillVO` and `ShenQiUnlockVO` tables — verified by direct ID scan.

---

## What Is In Sync

- `SystemOpen.txt` vs `SystemOpen.xml`: Identical.
- `NewRedPoint.txt` vs `NewRedPoint.xml`: 6 entries, identical.
- All 7 `ShenQi*.txt/xml` file pairs: Content identical (client has +2 Unity header lines).
- `SystemParams`: `ZhanShiShenBing`, `ZhanShiShenBingFuTi`, `ZhuHunSheDing` identical.
- GoodVO items 23001–23016: Identical between SQL and Goods.xml.
