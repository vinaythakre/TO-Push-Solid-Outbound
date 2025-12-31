# Manual Changes Guide - Apply Changes in ABAP System

## ⚠️ Important Note
This document provides the exact changes you need to make manually in your ABAP system. Apply these changes when you have proper authorization.

---

## 📋 Summary of Changes Required

You need to:
1. **Add 3 types** to the type section (around line 483)
2. **Add 9 variables** to the data section (around line 502)
3. **Add 2 RANGES** declarations (after data section)
4. **Remove inline declarations** from their current locations

---

## 🔧 Change 1: Add Types to Type Section

### Location: After line 483 (after `lty_vttk_rail`)

**Find this:**
```abap
  TYPES: BEGIN OF lty_vttk_rail,"SN Das
          tknum TYPE tknum,
          route TYPE routr,
         END OF lty_vttk_rail.

  DATA: lt_zlog_rail_loc TYPE TABLE OF lty_zlog_rail_loc,
```

**Replace with:**
```abap
  TYPES: BEGIN OF lty_vttk_rail,"SN Das
          tknum TYPE tknum,
          route TYPE routr,
         END OF lty_vttk_rail,

         BEGIN OF lty_chain_shp_temp,
           chainid  TYPE zchainid,
           odpairid TYPE zodpairid,
         END OF lty_chain_shp_temp,

         BEGIN OF lty_chain_shp_rail,
           shnumber TYPE zscm_chainship-shnumber,
           chainid  TYPE zscm_chainship-chainid,
           odpairid TYPE zscm_chainship-odpairid,
           trmode   TYPE zscm_chainship-trmode,
         END OF lty_chain_shp_rail,

         BEGIN OF lty_route_rail,
           route TYPE routr,
         END OF lty_route_rail.

  DATA: lt_zlog_rail_loc TYPE TABLE OF lty_zlog_rail_loc,
```

---

## 🔧 Change 2: Add Variables to Data Section

### Location: After line 502 (after `lw_tknum_rail`)

**Find this:**
```abap
        lt_tknum_rail    TYPE TABLE OF tknum,
        lw_tknum_rail    TYPE tknum.

  CONSTANTS: lc_rail_loc_config TYPE rvari_vnam VALUE 'ZSCM_GET_RAIL_LOCATION'.
```

**Replace with:**
```abap
        lt_tknum_rail    TYPE TABLE OF tknum,
        lw_tknum_rail    TYPE tknum,
        lt_chain_shp_t   TYPE TABLE OF lty_chain_shp,
        lt_chain_shp_rail TYPE TABLE OF lty_chain_shp_rail,
        lw_chain_shp_rail TYPE lty_chain_shp_rail,
        lw_chain_shp_temp TYPE lty_chain_shp_temp,
        lt_chain_shp_temp TYPE TABLE OF lty_chain_shp_temp,
        lv_trmode         TYPE versart,
        lw_route_rail     TYPE lty_route_rail,
        lt_route_rail     TYPE TABLE OF lty_route_rail,
        lv_vsart          TYPE versart.

  RANGES: lr_trmode FOR lv_trmode,
          lr_vsart  FOR lv_vsart.

  CONSTANTS: lc_rail_loc_config TYPE rvari_vnam VALUE 'ZSCM_GET_RAIL_LOCATION'.
```

---

## 🔧 Change 3: Remove Inline Variable Declaration (Line ~831)

### Location: Around line 831

**Find this:**
```abap
    " Collect unique chainid and odpairid combinations for rail mode lookup
    DATA:lt_chain_shp_t TYPE TABLE OF lty_chain_shp."SN das

    CLEAR : lt_chain_shp_t[].
```

**Replace with:**
```abap
    " Collect unique chainid and odpairid combinations for rail mode lookup
    CLEAR : lt_chain_shp_t[].
```

---

## 🔧 Change 4: Remove Inline Type and Variable Declarations (Lines ~838-856)

### Location: Around lines 838-856

**Find this:**
```abap
    SORT lt_chain_shp_t BY chainid odpairid.
    DELETE ADJACENT DUPLICATES FROM lt_chain_shp_t COMPARING chainid odpairid.

    TYPES: BEGIN OF lty_chain_shp_temp,"SN das
           chainid  TYPE zchainid,
           odpairid TYPE zodpairid,
           END OF lty_chain_shp_temp.

    TYPES:BEGIN OF lty_chain_shp_rail,"SN das
          shnumber TYPE zscm_chainship-shnumber,
          chainid  TYPE zscm_chainship-chainid,
          odpairid TYPE zscm_chainship-odpairid,
          trmode   TYPE zscm_chainship-trmode,
          END OF lty_chain_shp_rail.

    DATA: lt_chain_shp_rail TYPE TABLE OF lty_chain_shp_rail,"SN das
          lw_chain_shp_rail TYPE lty_chain_shp_rail,
          lw_chain_shp_temp TYPE lty_chain_shp_temp,
          lt_chain_shp_temp TYPE TABLE OF lty_chain_shp_temp,
          lv_trmode         TYPE versart.

    RANGES: lr_trmode FOR lv_trmode."SN Das

    " Fetch rail mode shipments for all chain/odpair combinations
```

**Replace with:**
```abap
    SORT lt_chain_shp_t BY chainid odpairid.
    DELETE ADJACENT DUPLICATES FROM lt_chain_shp_t COMPARING chainid odpairid.

    " Fetch rail mode shipments for all chain/odpair combinations
```

---

## 🔧 Change 5: Remove Inline Type and Variable Declarations (Lines ~931-939)

### Location: Around lines 931-939

**Find this:**
```abap
    " Step 7: Bulk fetch rail yard locations from TVRAB
    TYPES:BEGIN OF lty_route_rail,
          route TYPE routr,
          END OF lty_route_rail.

    DATA:lw_route_rail TYPE lty_route_rail,
         lt_route_rail TYPE TABLE OF lty_route_rail,
         lv_vsart      TYPE versart.

    RANGES:lr_vsart FOR lv_vsart.

    IF lt_vttk_rail IS NOT INITIAL AND lt_zlog_rail_loc IS NOT INITIAL.
```

**Replace with:**
```abap
    " Step 7: Bulk fetch rail yard locations from TVRAB
    IF lt_vttk_rail IS NOT INITIAL AND lt_zlog_rail_loc IS NOT INITIAL.
```

---

## ✅ Verification Checklist

After making all changes, verify:

- [ ] Three new types added to type section (after `lty_vttk_rail`)
- [ ] Nine new variables added to data section (after `lw_tknum_rail`)
- [ ] Two RANGES declarations added (after data section)
- [ ] Inline `DATA:lt_chain_shp_t` removed from line ~831
- [ ] Inline types and variables removed from lines ~838-856
- [ ] Inline types, variables, and RANGES removed from lines ~931-939
- [ ] Code compiles without errors
- [ ] All variables are accessible where they're used

---

## 📝 Quick Reference: What Goes Where

### Type Section (around line 483)
Add these 3 types:
- `lty_chain_shp_temp`
- `lty_chain_shp_rail`
- `lty_route_rail`

### Data Section (around line 502)
Add these 9 variables:
- `lt_chain_shp_t`
- `lt_chain_shp_rail`
- `lw_chain_shp_rail`
- `lw_chain_shp_temp`
- `lt_chain_shp_temp`
- `lv_trmode`
- `lw_route_rail`
- `lt_route_rail`
- `lv_vsart`

### RANGES Section (after data section)
Add these 2 RANGES:
- `lr_trmode FOR lv_trmode`
- `lr_vsart FOR lv_vsart`

---

## 🎯 Expected Result

After applying all changes:
- ✅ All types in type section
- ✅ All variables in data section
- ✅ All RANGES declarations grouped
- ✅ No inline declarations in code logic
- ✅ Code follows ABAP best practices

---

## ⚠️ Important Notes

1. **Line Numbers are Approximate**: Actual line numbers may vary slightly. Use the code context to find the exact locations.

2. **Test After Changes**: Always test the function module after making changes to ensure it works correctly.

3. **Backup**: Consider creating a backup or working in a transport request.

4. **Authorization**: Make sure you have proper authorization (S_DEVELOP) before making changes in the ABAP system.

---

## 📞 Need Help?

If you encounter any issues:
1. Check that all types are declared before they're used
2. Verify all variables are declared before they're used
3. Ensure RANGES are declared before they're populated
4. Check for syntax errors using ABAP syntax check

