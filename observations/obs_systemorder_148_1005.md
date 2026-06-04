# Observation: SystemOrder ID=148 / Order=1005 — "Tế Đàn Vạn Linh"

## Summary

The system "Tế Đàn Vạn Linh" has **3 config mismatches** across client and server:
1. **ID conflict in SystemOrder vs SystemOpen** (structural bug, same on both sides)
2. **IsHot value differs** between client and server SystemOrder
3. **VersionSystemOpen IsOpen=0 on client vs IsOpen=1 on server** (feature is live on server, closed on client)

---

## 1. ID Mismatch: SystemOrder says ID=148, but SystemOpen ID=148 is a different system

This is the most critical finding. The two config files disagree on which system ID belongs to "Tế Đàn Vạn Linh":

| File | ID | Order | Name |
|------|----|-------|------|
| `SystemOrder` (client & server) | **148** | 1005 | Tế Đàn Vạn Linh |
| `SystemOpen` (client & server) | **148** | 1002 | **Kho Báu Phù Hiệu** ← different system! |
| `SystemOpen` (client & server) | **153** | 1005 | Tế Đàn Vạn Linh ← correct match by Order |

**Conclusion**: SystemOrder ID=148 entry is likely wrong. The real SystemOpen ID for "Tế Đàn Vạn Linh" (Order=1005) is **153**, not 148.

This is the same bug on both client and server.

---

## 2. IsHot Mismatch (Client vs Server)

| File | IsHot |
|------|-------|
| `gameres_config_android/SystemOrder.txt` (client) | `IsHot="1"` |
| `prod_Config_S1/GameRes/Config/SystemOrder.xml` (server) | `IsHot="-1"` |

Full lines for comparison:

**Client:**
```xml
<SystemOrder ID="148" Order="1005" Name="Tế Đàn Vạn Linh" ImageOne="JiTan" RegionID="1" SubfunctionID="-1" JieMianShunXu="40" LinkID="41" IsHot="1" />
```

**Server:**
```xml
<SystemOrder ID="148" Order="1005" Name="Tế Đàn Vạn Linh" ImageOne="JiTan" RegionID="1" SubfunctionID="-1" JieMianShunXu="40" LinkID="41" IsHot="-1" />
```

**Effect**: `IsHot="1"` shows a "HOT" badge/tag on the feature icon on client. Server has it disabled (`-1`). These should match.

---

## 3. VersionSystemOpen: Feature enabled on server, disabled on client

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

**All 17.0.0 systems (including Tế Đàn Vạn Linh at ID=100153) are live on server but locked on client.**

Note: Server uses Chinese names (万灵祭坛 = Tế Đàn Vạn Linh), client uses English translations. Same IDs.

---

## Fix Instructions

### Fix 1 — Correct the SystemOrder ID for "Tế Đàn Vạn Linh"

In both `gameres_config_android/SystemOrder.txt` and `prod_Config_S1/GameRes/Config/SystemOrder.xml`:

Change the ID from `148` to `153` for the "Tế Đàn Vạn Linh" entry:

```xml
<!-- BEFORE -->
<SystemOrder ID="148" Order="1005" Name="Tế Đàn Vạn Linh" ... />

<!-- AFTER -->
<SystemOrder ID="153" Order="1005" Name="Tế Đàn Vạn Linh" ... />
```

> ⚠️ Verify that ID=153 is not already used by another SystemOrder entry before applying.

### Fix 2 — Sync IsHot value

Decide the canonical value for IsHot on "Tế Đàn Vạn Linh" (ID=148/153) and apply it to both:

- If IsHot should be active: set `IsHot="1"` on server (`SystemOrder.xml`)
- If IsHot should be inactive: set `IsHot="-1"` on client (`SystemOrder.txt`)

### Fix 3 — Enable VersionSystemOpen on client

In `gameres_config_android/VersionSystemOpen.txt`, update IsOpen for **all 17.0.0 features** to match server:

```xml
<!-- Update these from IsOpen="0" to IsOpen="1" -->
<Version ID="100148" SystemName="16.1.0 Armband Treasure" IsOpen="1" />
<Version ID="100153" SystemName="17.0.0 All Souls Altar" IsOpen="1" />
<Version ID="100154" SystemName="17.0.0 Smelting Furnace-God Forge" IsOpen="1" />
<Version ID="100155" SystemName="17.0.0 Miracle Magic Armor" IsOpen="1" />
<Version ID="100156" SystemName="17.0.0 Elven Mythology" IsOpen="1" />
<Version ID="100157" SystemName="17.0.0 King Of Hegemony" IsOpen="1" />
<Version ID="100158" SystemName="17.0.0 Mount Ascension" IsOpen="1" />
<Version ID="100159" SystemName="17.0.0 Master Breakthrough" IsOpen="1" />
```

---

## Files Involved

| File | Side | Issue |
|------|------|-------|
| `gameres_config_android/SystemOrder.txt` | Client | ID=148 should be 153; IsHot="1" mismatches server |
| `prod_Config_S1/GameRes/Config/SystemOrder.xml` | Server | ID=148 should be 153; IsHot="-1" mismatches client |
| `gameres_config_android/SystemOpen.txt` | Client | No change needed (ID=153 is correct here) |
| `prod_Config_S1/GameRes/Config/SystemOpen.xml` | Server | No change needed (ID=153 is correct here) |
| `gameres_config_android/VersionSystemOpen.txt` | Client | IsOpen="0" for all 17.0.0 systems — should be "1" |
| `prod_Config_S1/GameRes/Config/VersionSystemOpen.xml` | Server | IsOpen="1" (correct, reference) |

---

## What is NOT mismatched

- `SystemOpen.txt` vs `SystemOpen.xml`: Both have ID=148→"Kho Báu Phù Hiệu" and ID=153→"Tế Đàn Vạn Linh" with same parameters. **In sync.**
- `confdata_android.sql`: No SystemOrder/SystemOpen table found. Config for "Tế Đàn Vạn Linh" items (GoodVO 2114–2119: Thần Linh Thanh/Tinh/Thánh Huyết, Thần Lực items) exists in SQL but is client-side item data only.
- No dedicated JiTan/altar config file found on either side — feature config lives entirely in the system-level files above.
