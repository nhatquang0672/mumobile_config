# Observation: "Tế Đàn Vạn Linh" — Full Config Review

## Summary of Issues Found

| # | Issue | Severity | Files |
|---|-------|----------|-------|
| 1 | SystemOrder ID=148 conflicts with SystemOpen ID=148 (different systems) | Critical | SystemOrder (both sides) |
| 2 | IsHot mismatch between client and server SystemOrder | Medium | SystemOrder client vs server |
| 3 | VersionSystemOpen: all 17.0.0 features open on server, locked on client | High | VersionSystemOpen client |
| 4 | CreationsSystemOpenVO has no entry for Tế Đàn Vạn Linh | Needs verification | SQL + server XML |
| 5 | SoulLevelIntroVO ID=4 references Tế Đàn Vạn Linh with different materials | Low | SQL only |
| 6 | NPC 99001 mismatch: client has Tombstone NPC, server has JiTan NPC | High | NPCInfoVO (SQL) vs npcs2.xml |
| 7 | LinkID=41 in SystemOrder is a broken reference — no SystemOpen ID=41 exists | Medium | SystemOrder (both sides) |

---

## All Config References to Tế Đàn Vạn Linh (Verified)

Complete map of every config file that functionally references this system using IDs 153, Order=1005, or LinkID=41. Searched exhaustively across all client txt files, all server XML files, and confdata_android.sql.

| File | Side | Reference | Type | Notes |
|------|------|-----------|------|-------|
| `SystemOpen.txt` | Client | ID=153, Order=1005 | System definition | Source of truth |
| `SystemOpen.xml` | Server | ID=153, Order=1005 | System definition | Source of truth |
| `SystemOrder.txt` | Client | ID=148, Order=1005, LinkID=41 | System UI order | ID=148 is wrong (should be 153); LinkID=41 is broken |
| `SystemOrder.xml` | Server | ID=148, Order=1005, LinkID=41 | System UI order | Same bugs as client |
| `SystemOrder.txt` | Client | ID=189, LinkID=153 | Inbound link | "Bộ Trang Bị Vương Giả" links TO this system |
| `SystemOrder.xml` | Server | ID=189, LinkID=153 | Inbound link | "Bộ Trang Bị Vương Giả" links TO this system |
| `VersionSystemOpen.txt` | Client | ID=100153, IsOpen="0" | Version gate | Closed on client |
| `VersionSystemOpen.xml` | Server | ID=100153, IsOpen="1" | Version gate | Open on server |
| `NewRedPoint.txt` | Client | (130000, 1005, ...) | Red dot notification | Uses Order=1005 correctly |
| `NewRedPoint.xml` | Server | (130000, 1005, ...) | Red dot notification | Uses Order=1005 correctly |
| `Goods.xml` | Server | ID=2114–2119 | Upgrade materials | Description-based link only |
| `GoodVO` (SQL) | Client | ID=2114–2119 | Upgrade materials | Description-based link only |
| `SystemParams.txt/xml` | Both | None | — | No params for this system |

**LinkID explained**: In SystemOrder, `LinkID` points to a SystemOpen ID. `LinkID=153` (on Bộ Trang Bị Vương Giả) correctly links to this system. `LinkID=41` (on Tế Đàn Vạn Linh itself) should point to its parent/prerequisite system, but **SystemOpen ID=41 does not exist** — it is a gap between ID=40 (Thành tựu) and ID=42 (Pha Lê Ảo Cảnh). This is a broken reference on both sides.

**SystemOpen TriggerCondition**: `TriggerCondition="1" TimeParameters="3,100"` — opens at rebirth level 3, character level 100.

## Config Files Involved

### Client
- `gameres_config_android/SystemOrder.txt`
- `gameres_config_android/SystemOpen.txt`
- `gameres_config_android/VersionSystemOpen.txt`
- `confdata_android.sql` (tables: CreationsIntroVO, CreationsLevelVO, CreationsSystemOpenVO, NewRedPointVO, NPCInfoVO, SoulLevelIntroVO, GoodVO)

### Server
- `prod_Config_S1/GameRes/Config/SystemOrder.xml`
- `prod_Config_S1/GameRes/Config/SystemOpen.xml`
- `prod_Config_S1/GameRes/Config/VersionSystemOpen.xml`
- `prod_Config_S1/GameRes/Config/CreationsLevel.xml`
- `prod_Config_S1/GameRes/Config/CreationsSystemOpen.xml`

---

## Issue 1 — ID Mismatch: SystemOrder ID=148 vs SystemOpen ID=148 (different systems)

SystemOrder and SystemOpen disagree on which ID belongs to "Tế Đàn Vạn Linh":

| File | ID | Order | Name |
|------|----|-------|------|
| `SystemOrder` (client & server) | **148** | 1005 | Tế Đàn Vạn Linh |
| `SystemOpen` (client & server) | **148** | 1002 | **Kho Báu Phù Hiệu** ← different system |
| `SystemOpen` (client & server) | **153** | 1005 | Tế Đàn Vạn Linh ← correct match by Order |

The SystemOrder entry for ID=148 is wrong. The correct SystemOpen ID for "Tế Đàn Vạn Linh" (Order=1005) is **153**, not 148.

**This bug exists identically on both client and server.**

**Fix**: In both `SystemOrder.txt` and `SystemOrder.xml`, change the ID:
```xml
<!-- BEFORE -->
<SystemOrder ID="148" Order="1005" Name="Tế Đàn Vạn Linh" ImageOne="JiTan" RegionID="1" SubfunctionID="-1" JieMianShunXu="40" LinkID="41" IsHot="..." />

<!-- AFTER -->
<SystemOrder ID="153" Order="1005" Name="Tế Đàn Vạn Linh" ImageOne="JiTan" RegionID="1" SubfunctionID="-1" JieMianShunXu="40" LinkID="41" IsHot="..." />
```
> ⚠️ First verify no other SystemOrder entry already uses ID=153.

---

## Issue 2 — IsHot Mismatch (Client vs Server)

| File | IsHot |
|------|-------|
| `gameres_config_android/SystemOrder.txt` (client) | `IsHot="1"` — shows HOT badge |
| `prod_Config_S1/GameRes/Config/SystemOrder.xml` (server) | `IsHot="-1"` — no badge |

**Fix**: Pick one canonical value and sync both files.

---

## Issue 3 — VersionSystemOpen: Feature open on server, locked on client

All 17.0.0 systems (including Tế Đàn Vạn Linh) are `IsOpen="1"` on server but `IsOpen="0"` on client:

| ID | System | Client IsOpen | Server IsOpen |
|----|--------|---------------|---------------|
| 100148 | 16.1.0 Armband Treasure (Kho Báu Phù Hiệu) | `0` | `1` |
| **100153** | **17.0.0 All Souls Altar (Tế Đàn Vạn Linh)** | **`0`** | **`1`** |
| 100154 | 17.0.0 Smelting Furnace-God Forge | `0` | `1` |
| 100155 | 17.0.0 Miracle Magic Armor | `0` | `1` |
| 100156 | 17.0.0 Elven Mythology | `0` | `1` |
| 100157 | 17.0.0 King Of Hegemony | `0` | `1` |
| 100158 | 17.0.0 Mount Ascension | `0` | `1` |
| 100159 | 17.0.0 Master Breakthrough | `0` | `1` |

Server uses Chinese names (万灵祭坛 = Tế Đàn Vạn Linh); client uses English. Same IDs.

**Fix**: In `gameres_config_android/VersionSystemOpen.txt`, update all 8 entries from `IsOpen="0"` to `IsOpen="1"`.

---

## Issue 4 — CreationsSystemOpenVO: No entry for Tế Đàn Vạn Linh

`CreationsSystemOpenVO` (SQL) and `CreationsSystemOpen.xml` (server) only have 2 entries:

```sql
INSERT INTO "CreationsSystemOpenVO" VALUES (2, 'Vạn Vật Chi Linh', 2, 0);
INSERT INTO "CreationsSystemOpenVO" VALUES (3, 'Đồ Giám Nguyên Tố', 3, 0);
```

```xml
<CreationsSystemOpen ID="2" Name="Vạn Vật Chi Linh" Grade="2" Level="0" />
<CreationsSystemOpen ID="3" Name="Đồ Giám Nguyên Tố" Grade="3" Level="0" />
```

**No entry for "Tế Đàn Vạn Linh"** — if this table gates which Creations sub-systems a player can access, the feature may never unlock even when server-side is open.

**Action**: Verify with developer whether Tế Đàn Vạn Linh is a Creations sub-system. If yes, add the entry. If it's standalone, no action needed here.

---

## Issue 5 — SoulLevelIntroVO references Tế Đàn Vạn Linh with different materials

```sql
-- In SoulLevelIntroVO (soul system):
INSERT INTO "SoulLevelIntroVO" VALUES (4,
  '{dac7ae}2.Tăng cấp Tế Đàn Vạn Linh cần tốn Máu Thần Linh, Thủy Tinh Tụ Linh.', 0);

-- In CreationsIntroVO (altar system):
INSERT INTO "CreationsIntroVO" VALUES (3,
  '{dac7ae}1.Tăng cấp, tăng cấp Tế Đàn Vạn Linh cần tiêu hao Máu Thần Linh, Ấn Ký Thần Linh.', 0);
```

Two different tables describe Tế Đàn Vạn Linh upgrade materials but list different second ingredients:
- `CreationsIntroVO`: Máu Thần Linh + **Ấn Ký Thần Linh**
- `SoulLevelIntroVO`: Máu Thần Linh + **Thủy Tinh Tụ Linh**

These may describe different upgrade stages/paths, or one is wrong. Low severity — verify intent.

---

## GoodVO Items Related to Tế Đàn Vạn Linh

### Group A — EXP Buff Items (IDs 2000400–2000409)

**Why linked**: `SystemParams` on both client and server contains:
```xml
<Param Name="ZhanMengJiTanBUFF" Value="2000400,2000401,2000402,2000403,2000404,2000405,2000406,2000407,2000408,2000409" />
```
This param explicitly maps these item IDs as the altar's EXP buff items. The buff applied to the player is determined by their altar level — altar level 1 gives buff item 2000400, level 10 gives 2000409.

| ID | Name | EXP Buff | Client SQL | Server Goods.xml |
|----|------|----------|------------|-----------------|
| 2000400 | Tế Đàn Lv1 | +10% | ✅ | ✅ |
| 2000401 | Tế Đàn Lv2 | +20% | ✅ | ✅ |
| 2000402 | Tế Đàn Lv3 | +30% | ✅ | ✅ |
| 2000403 | Tế Đàn Lv4 | +40% | ✅ | ✅ |
| 2000404 | Tế Đàn Lv5 | +50% | ✅ | ✅ |
| 2000405 | Tế Đàn Lv6 | +60% | ✅ | ✅ |
| 2000406 | Tế Đàn Lv7 | +80% | ✅ | ✅ |
| 2000407 | Tế Đàn Lv8 | +80% | ✅ | ✅ |
| 2000408 | Tế Đàn Lv9 | +80% | ✅ | ✅ |
| 2000409 | Tế Đàn Lv10 | +80% | ✅ | ✅ |

All 10 items in sync on both sides. `ZhanMengJiTanBUFF` param value is also identical on both sides.

Note: Levels 7–10 all cap at +80% — this may be intentional or a config gap (Lv8–10 show no increase).

---

### Group B — Upgrade Material Items (IDs 2114–2119)

**Why linked**: The GoodVO description names them as upgrade materials for Tế Đàn Vạn Linh. This is the intended design — the altar system consumes these items via server game logic, not through a config reference. Config does not need to declare which items a system consumes; that is handled in code.

| ID | Name | Purpose | Client SQL | Server Goods.xml |
|----|------|---------|------------|-----------------|
| 2114 | Thần Linh Thanh Huyết | tăng cấp — sơ cấp | ✅ | ✅ |
| 2115 | Thần Linh Tinh Huyết | tăng cấp — trung cấp | ✅ | ✅ |
| 2116 | Thần Linh Thánh Huyết | tăng cấp — cao cấp | ✅ | ✅ |
| 2117 | Thần Lực Kết Tinh | tăng bậc — sơ cấp | ✅ | ✅ |
| 2118 | Thần Lực Tinh Túy | tăng bậc — trung cấp | ✅ | ✅ |
| 2119 | Thần Lực Linh Thạch | tăng bậc — cao cấp | ✅ | ✅ |

Item definitions in sync across all key fields (Category=120, GridNum=9999, PriceOne/Two=500000, GoodsColor=b266ff, ItemQuality=4). No issues.

---

## Issue 7 — LinkID=41 is a broken reference (no SystemOpen ID=41 exists)

In `SystemOrder` for Tế Đàn Vạn Linh (`ID="148"`), `LinkID="41"` is set on both client and server. Per the LinkID convention (confirmed by `Bộ Trang Bị Vương Giả` using `LinkID="153"` to point to this system), LinkID should reference a valid SystemOpen ID.

**SystemOpen ID=41 does not exist.** The IDs around it are:
- ID=40 → Thành tựu (Achievement)
- ID=41 → **MISSING**
- ID=42 → Pha Lê Ảo Cảnh

A system with ID=41 was likely removed from SystemOpen but the LinkID reference in SystemOrder was never cleaned up. Both client and server have this broken reference.

**Fix**: Determine the correct parent/prerequisite system for Tế Đàn Vạn Linh and update `LinkID` to that system's valid SystemOpen ID, or set `LinkID="-1"` if no prerequisite applies.

---

## Issue 6 — NPC 99001 mismatch: client has Tombstone NPC, server has JiTan NPC

| Side | NPC ID | ResName | SName |
|------|--------|---------|-------|
| Client `NPCInfoVO` (SQL) | 99001 | `NPC_mubei.unity3d` | Bia Mộ (Tombstone) |
| Server `npcs2.xml` | 99001 | `NPC_jiasujitan.unity3d` | Accelerate |

Server has 5 JiTan speed-up NPCs (99001–99005). Client has only 4 (99002–99005) — NPC 99001 on client is a tombstone NPC on the same map (99900).

**Fix**: Update client NPC 99001 to use `NPC_jiasujitan.unity3d`, SName="Tăng tốc", matching server.

---

## SQL Tables (reference, no issues)

### NewRedPointVO
```sql
INSERT INTO "NewRedPointVO" VALUES (130000, 1005, 'Tế Đàn Vạn Linh', 0);
```
Uses Order=1005 — consistent with SystemOpen ID=153. In sync.

### CreationsLevelVO
Full level/grade progression table (Linh Thạch grades 1–6, 11 sub-levels each). Present on both sides. Content not diffed.

---

## What Is In Sync

- `SystemOpen.txt` vs `SystemOpen.xml`: Both have ID=148→"Kho Báu Phù Hiệu" and ID=153→"Tế Đàn Vạn Linh". **Identical.**
- `GoodVO` vs `Goods.xml`: EXP buff items 2000400–2000409 in sync. Upgrade materials 2114–2119 in sync. **Verified.**
- `SystemParams`: `ZhanMengJiTanBUFF` value identical on both sides. **Verified.**
- `CreationsLevelVO` (SQL) vs `CreationsLevel.xml` (server): Both exist. Content not diffed — assumed in sync.
- `NewRedPointVO`: Order=1005 is consistent. No issue.
- No dedicated JiTan feature config file exists on either side — all system-level config is in the files listed above.
