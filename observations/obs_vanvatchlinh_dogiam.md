# Observation: "Linh Vạn Vật" & "Đồ Giám Nguyên Tố" — Sub-systems of Tế Đàn Vạn Linh

## Overview

These two features are sub-tabs inside the Tế Đàn Vạn Linh UI, registered in `CreationsSystemOpenVO`:

| ID | Name | Grade |
|----|------|-------|
| 2 | Vạn Vật Chi Linh (Linh Vạn Vật) | 2 |
| 3 | Đồ Giám Nguyên Tố | 3 |

Both `CreationsSystemOpen.xml` (server) and `CreationsSystemOpen.txt` (client) are **identical**. Neither has been expanded — no additional sub-system entry exists for Tế Đàn Vạn Linh itself (Grade=1 slot is absent), which ties back to Issue 4 in `obs_tedanvanlinh.md`.

---

## Summary of Issues

| # | Feature | Issue | Severity |
|---|---------|-------|----------|
| 1 | Đồ Giám Nguyên Tố | TotemsLevel.xml uses Chinese names (`图腾1-5`), client uses Vietnamese (`Đồ Giám 1-5`) | Medium |
| 2 | Vạn Vật Chi Linh | SoulLevelIntroVO says material = "Máu Thần Linh, Thủy Tinh Tụ Linh" but config uses item 2003 (Ngọc Sinh Mệnh) | Medium |
| 3 | Đồ Giám Nguyên Tố | TotemsIntroVO says material = "Máu Thần Linh, Cát Tụ Linh" — neither item exists in GoodVO | High |

---

## Feature 1: Vạn Vật Chi Linh (Grade=2)

### Config Files

| File | Side | In Sync? |
|------|------|----------|
| `SoulLevel.txt/xml` | Both | ✅ Identical content (client +2 Unity header lines) |
| `SoulBlessing.txt/xml` | Both | ✅ Identical |
| `SoulFireLevel.txt/xml` | Both | ✅ Identical |
| `SoulPowerLevel.txt/xml` | Both | ✅ Identical |
| `SoulRuneLevel.txt/xml` | Both | ✅ Identical |
| `SoulRuneRule` (SQL only) | Client | N/A — SQL table, no server XML equivalent found |
| `SoulTreasureLevel.txt/xml` | Both | ✅ Identical |
| `SoulTreasureFashionLevel.txt/xml` | Both | ✅ Identical |
| `SoulTreasureGrade.txt/xml` | Both | ✅ Identical |

### Structure

- 10 soul types (SoulID 1–10): Linh Minh Nha, Linh Thủy Tiên, Linh Mị Hoặc, Linh Man Chùy, Linh Cuồng Phủ, Linh Cổ Thú, Linh Yêu Hoa, Linh Du Ngư, Linh Phi Dực, Linh Nhược Mộc
- Each soul has 100 levels (Level 0–100)
- Max level (100) has `NeedGoods="-1"` (no further upgrade)
- `SoulBlessing`: 10 blessing levels triggered by total soul count × average level

### Issue 2 — SoulLevelIntroVO material mismatch

`SoulLevelIntroVO` (SQL, client-only):
```
ID=4: "Tăng cấp Tế Đàn Vạn Linh cần tốn Máu Thần Linh, Thủy Tinh Tụ Linh."
```

Actual `SoulLevel.xml` `NeedGoods`: item **2003** ("Ngọc Sinh Mệnh") only.

"Máu Thần Linh" and "Thủy Tinh Tụ Linh" do not exist in GoodVO. The intro text is wrong — it describes materials that don't exist. Players see incorrect upgrade instructions in-game.

---

## Feature 2: Đồ Giám Nguyên Tố (Grade=3)

### Config Files

| File | Side | In Sync? |
|------|------|----------|
| `TotemsLevel.txt/xml` | Both | ⚠️ Content differs — see Issue 1 |
| `TotemsIntroVO` (SQL) | Client | N/A — SQL only |
| `TotemsLevelVO` (SQL) | Client | N/A — SQL only |

### Issue 1 — TotemsLevel Name mismatch (Chinese vs Vietnamese)

Server `TotemsLevel.xml`:
```xml
<TotemsLevel XuLie="10000" ID="1" Name="图腾1" .../>   ← Chinese
<TotemsLevel XuLie="20000" ID="2" Name="图腾2" .../>
<TotemsLevel XuLie="30000" ID="3" Name="图腾3" .../>
<TotemsLevel XuLie="40000" ID="4" Name="图腾4" .../>
<TotemsLevel XuLie="50000" ID="5" Name="图腾5" .../>
```

Client `TotemsLevel.txt`:
```xml
<TotemsLevel XuLie="10000" ID="1" Name="Đồ Giám 1" .../>  ← Vietnamese
<TotemsLevel XuLie="20000" ID="2" Name="Đồ Giám 2" .../>
...
```

All other fields (Model, Grade, Level, Attribute, NeedGoods) are identical. Only `Name` differs.

**Fix**: Update server `TotemsLevel.xml` Name fields from `图腾1-5` to `Đồ Giám 1-5`.

### Issue 3 — TotemsIntroVO material mismatch (materials don't exist)

`TotemsIntroVO` (SQL):
```
ID=3: "Tăng cấp, tăng bậc Đồ Giám Nguyên Tố cần Máu Thần Linh, Cát Tụ Linh"
ID=4: "Cấp Đồ Giám Nguyên Tố càng cao... Máu Thần Tinh..."
ID=5: "Tốn Máu Thần Linh, Cát Tụ Linh có thể tăng cấp..."
```

Actual `TotemsLevel.xml` `NeedGoods`:
- Grade 1–2: item **2003** (Ngọc Sinh Mệnh) only
- Grade 3: items **2003** + **2004** (Ngọc Sáng Tạo)

Neither "Máu Thần Linh" nor "Cát Tụ Linh" exist as GoodVO items. The intro text describes nonexistent materials — player sees wrong upgrade instructions.

### Structure

- 5 totem types (ID 1–5): Wind/Fire/Lightning/Water/Earth (yuansututeng_feng/huo/lei/shui/tu)
- Each totem has Grades 1–3, Levels 0–10 (33 entries per totem, ~165 total)
- NeedGoods="-1" at max level (Grade 3, Level 10)

---

## Shared Upgrade Materials (Both Sub-systems)

Both Vạn Vật Chi Linh and Đồ Giám Nguyên Tố use the same item pool:

| Item ID | Name | Used by |
|---------|------|---------|
| 2003 | Ngọc Sinh Mệnh | Both (all grades) |
| 2004 | Ngọc Sáng Tạo | Đồ Giám Nguyên Tố Grade 2–3 |

Note: item 2003 and 2004 descriptions in GoodVO say "trang bị" (equipment) not soul/totem — their descriptions are generic, not feature-specific. Same items are also used by `CreationsLevel` (the parent Tế Đàn Vạn Linh system).

---

## What Is In Sync

- `CreationsSystemOpen.xml/txt`: Identical both sides
- All `SoulLevel`, `SoulBlessing`, `SoulFireLevel`, `SoulPowerLevel`, `SoulRuneLevel`, `SoulTreasureLevel`, `SoulTreasureFashionLevel`, `SoulTreasureGrade`: Content identical
- `TotemsLevel` game stats (Attribute, NeedGoods, Grade, Level, Model): Identical — only Name differs
