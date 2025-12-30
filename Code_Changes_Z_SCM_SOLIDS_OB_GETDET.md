# Code Changes Documentation: Z_SCM_SOLIDS_OB_GETDET

## Function Module
**Name:** Z_SCM_SOLIDS_OB_GETDET  
**Enhancement:** Populate Rail Yard Source & Destination Locations during TO Push  
**Change Date:** [To be filled]  
**Change Request:** [To be filled]  
**Author:** [To be filled]

---

## Overview
This document details all code changes made to implement rail yard location population logic in the function module `Z_SCM_SOLIDS_OB_GETDET`. The enhancement populates `RAILYARDSOURCELOCATION` and `RAILYARDDESTINATIONLOCATION` fields in the `ET_SOB_CN_DATA` structure during Transportation Order (TO) Push rate check processing.

---

## Change Summary

| Change # | Location | Type | Description |
|----------|----------|------|-------------|
| 1 | Lines 483-492 | Addition | Added data declarations for rail yard location processing |
| 2 | Lines 3155-3238 | Addition | Implemented rail yard location derivation logic |

---

## Detailed Changes

### Change 1: Data Declarations Addition

**Location:** Lines 483-492  
**Type:** Addition  
**Purpose:** Add local variables required for rail yard location processing

#### Code Added:

```abap
DATA: lt_chain_shp     TYPE TABLE OF lty_chain_shp,
      lv_rail_shnumber TYPE tknum,
      lv_rail_chainid  TYPE zchainid,
      lv_rail_odpairid TYPE zodpairid,
      lv_rail_knanf    TYPE knota,
      lv_rail_knend    TYPE knotz,
      lw_vttk_rail     TYPE vttk.
```

#### Context (Before Change):

```abap
DATA: lt_zlog_rail_loc TYPE TABLE OF lty_zlog_rail_loc,
      lw_zlog_rail_loc TYPE lty_zlog_rail_loc,
      lw_chain_shp     TYPE lty_chain_shp,
      lw_tvrab_rail    TYPE lty_tvrab_rail,
      lv_rail_bukrs    TYPE bukrs,
      lv_rail_vsart    TYPE vsart,
      lv_rail_route    TYPE route,
      lv_rail_active   TYPE abap_bool.

CONSTANTS: lc_rail_loc_config TYPE rvari_vnam VALUE 'ZSCM_GET_RAIL_LOCATION'.
" END: Cursor Generated Code
```

#### Context (After Change):

```abap
DATA: lt_zlog_rail_loc TYPE TABLE OF lty_zlog_rail_loc,
      lw_zlog_rail_loc TYPE lty_zlog_rail_loc,
      lw_chain_shp     TYPE lty_chain_shp,
      lw_tvrab_rail    TYPE lty_tvrab_rail,
      lv_rail_bukrs    TYPE bukrs,
      lv_rail_vsart    TYPE vsart,
      lv_rail_route    TYPE route,
      lv_rail_active   TYPE abap_bool,
      lt_chain_shp     TYPE TABLE OF lty_chain_shp,
      lv_rail_shnumber TYPE tknum,
      lv_rail_chainid  TYPE zchainid,
      lv_rail_odpairid TYPE zodpairid,
      lv_rail_knanf    TYPE knota,
      lv_rail_knend    TYPE knotz,
      lw_vttk_rail     TYPE vttk.

CONSTANTS: lc_rail_loc_config TYPE rvari_vnam VALUE 'ZSCM_GET_RAIL_LOCATION'.
" END: Cursor Generated Code
```

#### Variable Descriptions:

| Variable | Type | Purpose |
|----------|------|---------|
| `lt_chain_shp` | Table | Internal table for chain shipment data |
| `lv_rail_shnumber` | tknum | Rail mode shipment number |
| `lv_rail_chainid` | zchainid | Chain ID from chain shipment |
| `lv_rail_odpairid` | zodpairid | Origin-Destination pair ID |
| `lv_rail_knanf` | knota | Start rail yard location (KNANF) |
| `lv_rail_knend` | knotz | End rail yard location (KNEND) |
| `lw_vttk_rail` | vttk | Work area for rail shipment header |

---

### Change 2: Rail Yard Location Derivation Logic

**Location:** Lines 3155-3238  
**Type:** Addition  
**Purpose:** Implement complete rail yard location population logic before appending CN data

#### Code Added:

```abap
" BEGIN: Cursor Generated Code
" Rail Yard Location Population Logic
CLEAR: lv_rail_bukrs, lv_rail_vsart, lv_rail_route,
       lv_rail_shnumber, lv_rail_chainid, lv_rail_odpairid,
       lv_rail_knanf, lv_rail_knend, lw_zlog_rail_loc,
       lw_chain_shp, lw_vttk_rail, lw_tvrab_rail.

" Step 2: Determine Company Code from TTDS
CLEAR lw_ttds.
READ TABLE lt_ttds INTO lw_ttds WITH KEY tplst = gw_vttk-tplst BINARY SEARCH.
IF sy-subrc = 0.
  lv_rail_bukrs = lw_ttds-bukrs.

  " Step 3: Company Code Validation and Rail Yard Derivation
  " Loop through rail location configuration to find matching company code
  LOOP AT lt_zlog_rail_loc INTO lw_zlog_rail_loc
    WHERE bukrs = lv_rail_bukrs.
    lv_rail_vsart = lw_zlog_rail_loc-vsart.

    " Step 4: Retrieve Chain Shipment Details
    CLEAR lw_chain_shp.
    SELECT SINGLE shnumber
                  chainid
                  odpairid
          FROM zscm_chainship
          CLIENT SPECIFIED
          INTO (lw_chain_shp-shnumber,
                lw_chain_shp-chainid,
                lw_chain_shp-odpairid)
          WHERE mandt = sy-mandt
            AND shnumber = gw_vttk-tknum.
    IF sy-subrc = 0.
      lv_rail_chainid = lw_chain_shp-chainid.
      lv_rail_odpairid = lw_chain_shp-odpairid.

      " Step 5: Identify Shipment for Rail Mode
      CLEAR lw_chain_shp.
      SELECT SINGLE shnumber
            FROM zscm_chainship
            CLIENT SPECIFIED
            INTO lw_chain_shp-shnumber
            WHERE mandt = sy-mandt
              AND chainid = lv_rail_chainid
              AND odpairid = lv_rail_odpairid
              AND trmode = lv_rail_vsart.
      IF sy-subrc = 0.
        lv_rail_shnumber = lw_chain_shp-shnumber.

        " Step 6: Retrieve Route from Shipment
        CLEAR lw_vttk_rail.
        SELECT SINGLE route
              FROM vttk
              CLIENT SPECIFIED
              INTO lw_vttk_rail-route
              WHERE mandt = sy-mandt
                AND tknum = lv_rail_shnumber.
        IF sy-subrc = 0.
          lv_rail_route = lw_vttk_rail-route.

          " Step 7: Determine Rail Yard Locations
          CLEAR lw_tvrab_rail.
          SELECT SINGLE knanf
                      knend
                FROM tvrab
                CLIENT SPECIFIED
                INTO (lw_tvrab_rail-knanf,
                      lw_tvrab_rail-knend)
                WHERE mandt = sy-mandt
                  AND vsart = gw_vttk-vsart
                  AND route = lv_rail_route.
          IF sy-subrc = 0.
            lv_rail_knanf = lw_tvrab_rail-knanf.
            lv_rail_knend = lw_tvrab_rail-knend.

            " Step 8: Populate Output Structure
            gw_cn_data-railyardsourcelocation = lv_rail_knanf.
            gw_cn_data-railyarddestinationlocation = lv_rail_knend.
            EXIT. " Exit loop once valid match found
          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.
  ENDLOOP.
ENDIF.
" END: Cursor Generated Code
```

#### Context (Before Change):

```abap
* Added by SN Das CD:8086353(End)
              APPEND gw_cn_data TO et_sob_cn_data.
              CLEAR : gw_vbrp,lw_subusiness_id.
```

#### Context (After Change):

```abap
* Added by SN Das CD:8086353(End)
              " BEGIN: Cursor Generated Code
              " Rail Yard Location Population Logic
              [Complete logic as shown above]
              " END: Cursor Generated Code

              APPEND gw_cn_data TO et_sob_cn_data.
              CLEAR : gw_vbrp,lw_subusiness_id.
```

---

## Logic Flow

The implementation follows the 8-step process defined in the functional specification:

1. **Step 1**: Configuration Read (Already implemented at lines 610-627)
   - Reads `ZLOG_EXEC_VAR` with `NAME = 'ZSCM_GET_RAIL_LOCATION'` and `ACTIVE = 'X'`
   - Stores in `lt_zlog_rail_loc` sorted by `bukrs vsart`

2. **Step 2**: Determine Company Code from TTDS
   - Reads `lt_ttds` table using `VTTK-TPLST` to get `BUKRS`
   - Uses existing table populated earlier in the function module

3. **Step 3**: Company Code Validation
   - Loops through `lt_zlog_rail_loc` to find matching `BUKRS`
   - If match found, continues with rail yard derivation
   - If no match, fields remain blank

4. **Step 4**: Retrieve Chain Shipment Details
   - Reads `ZSCM_CHAINSHIP` using `VTTK-TKNUM` as `SHNUMBER`
   - Retrieves `CHAINID` and `ODPAIRID`

5. **Step 5**: Identify Shipment for Rail Mode
   - Uses `CHAINID`, `ODPAIRID`, and `TRMODE = VSART` from configuration
   - Reads `ZSCM_CHAINSHIP` to find matching rail mode shipment
   - Retrieves `SHNUMBER` for rail mode shipment

6. **Step 6**: Retrieve Route from Shipment
   - Reads `VTTK` using rail mode `SHNUMBER`
   - Retrieves `ROUTE`

7. **Step 7**: Determine Rail Yard Locations
   - Reads `TVRAB` using `VSART` and `ROUTE`
   - Retrieves `KNANF` (Start Rail Yard) and `KNEND` (End Rail Yard)

8. **Step 8**: Populate Output Structure
   - Assigns values to:
     - `gw_cn_data-railyardsourcelocation = lv_rail_knanf`
     - `gw_cn_data-railyarddestinationlocation = lv_rail_knend`
   - Exits loop once valid match is found

---

## Database Tables Accessed

| Table | Purpose | Key Fields | Retrieved Fields |
|-------|---------|------------|------------------|
| `TTDS` | Get company code | `TPLST = VTTK-TPLST` | `BUKRS` |
| `ZSCM_CHAINSHIP` | Get chain shipment details | `SHNUMBER = VTTK-TKNUM` | `CHAINID`, `ODPAIRID` |
| `ZSCM_CHAINSHIP` | Find rail mode shipment | `CHAINID`, `ODPAIRID`, `TRMODE = VSART` | `SHNUMBER` |
| `VTTK` | Get route from shipment | `TKNUM = rail mode SHNUMBER` | `ROUTE` |
| `TVRAB` | Get rail yard locations | `VSART`, `ROUTE` | `KNANF`, `KNEND` |

---

## Error Handling

- All database operations check `sy-subrc` immediately after execution
- If any step fails, processing continues but fields remain blank
- No dumps occur - all errors are handled gracefully
- Logic executes only when:
  - Configuration exists (`lt_zlog_rail_loc` is not initial)
  - Company code matches configuration entry
- Multiple configuration entries processed sequentially until valid match found
- Loop exits immediately once valid match is found (using `EXIT`)

---

## Performance Considerations

1. **Configuration Read**: Already optimized - read once before CN loop (lines 610-627)
2. **Company Code Lookup**: Uses existing `lt_ttds` table with `BINARY SEARCH`
3. **SELECT in Loop**: 
   - Acceptable here because configuration table is expected to be small
   - Functional specification requires sequential processing until match found
   - Loop exits immediately once valid match is found
4. **Database Access**: Uses `CLIENT SPECIFIED` for all SELECT statements
5. **Index Usage**: All SELECT statements use key fields for optimal performance

---

## Testing Scenarios

### Unit Test Cases

1. **Configuration exists, company code matches**
   - Expected: Rail yard locations populated correctly
   - Verify: `RAILYARDSOURCELOCATION` and `RAILYARDDESTINATIONLOCATION` have values

2. **Configuration exists, company code does not match**
   - Expected: Fields remain blank
   - Verify: Both fields are initial

3. **Configuration does not exist**
   - Expected: Fields remain blank
   - Verify: Both fields are initial

4. **Chain shipment not found**
   - Expected: Fields remain blank
   - Verify: Both fields are initial

5. **Rail mode shipment not found**
   - Expected: Fields remain blank
   - Verify: Both fields are initial

6. **TVRAB entry not found**
   - Expected: Fields remain blank
   - Verify: Both fields are initial

### Integration Test Cases

1. **End-to-end TO Push with rail mode shipment**
   - Test complete flow with valid configuration and data
   - Verify rate check receives populated rail yard locations

2. **Multiple shipments with different company codes**
   - Test batch processing with mixed company codes
   - Verify only matching company codes get rail yard locations

3. **Mixed rail and non-rail shipments**
   - Test processing of both rail and non-rail shipments
   - Verify rail yard locations only populated for rail shipments

---

## Code Quality Compliance

✅ **ABAP Coding Rules Compliance:**
- All variables properly declared with correct prefixes (`lv_`, `lw_`, `lt_`)
- Uses `CLIENT SPECIFIED` for all SELECT statements
- Proper error handling with `sy-subrc` checks
- Code marked with "BEGIN: Cursor Generated Code" / "END: Cursor Generated Code"
- No hard-coded values (uses constants and variables)
- Proper CLEAR statements before variable usage

✅ **NetWeaver 7.31 Compatibility:**
- No inline declarations
- No constructor operators
- No string templates
- Classic OpenSQL syntax
- All variables declared upfront

---

## Dependencies

### Existing Code Dependencies
- `lt_ttds`: Table populated earlier in function module (line 766-777)
- `lt_zlog_rail_loc`: Configuration table populated earlier (lines 610-627)
- `gw_vttk`: Shipment header work area from main processing loop
- `gw_cn_data`: CN data structure being populated

### Configuration Dependencies
- `ZLOG_EXEC_VAR` table must have entries with:
  - `NAME = 'ZSCM_GET_RAIL_LOCATION'`
  - `ACTIVE = 'X'`
  - `BUKRS` and `VSART` fields populated

### Data Dependencies
- Chain shipment data must exist in `ZSCM_CHAINSHIP`
- Route data must exist in `TVRAB`
- Transportation planning point data must exist in `TTDS`

---

## Change Impact Analysis

### Affected Areas
- **CN Data Processing Section**: Logic added before APPEND statement
- **Output Structure**: Two new fields populated (`RAILYARDSOURCELOCATION`, `RAILYARDDESTINATIONLOCATION`)

### No Impact On
- Header data processing
- Customer document data processing
- Existing CN data fields
- Other function module logic

### Backward Compatibility
- ✅ Fully backward compatible
- ✅ If configuration doesn't exist, behavior unchanged (fields remain blank)
- ✅ No changes to existing field population logic
- ✅ No changes to function module interface

---

## Rollback Plan

To rollback these changes:

1. **Remove Change 2** (Lines 3155-3238): Delete the rail yard location derivation logic
2. **Remove Change 1** (Lines 487-492): Remove the additional data declarations

The function module will continue to work normally, but rail yard locations will not be populated (original behavior).

---

## Notes

- Code follows existing coding patterns in the function module
- Error handling consistent with rest of the codebase
- Performance optimized where possible
- All database operations use proper client specification
- Code is self-contained and doesn't affect other processing logic

---

## Approval

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Developer | [To be filled] | [Date] | |
| Reviewer | [To be filled] | [Date] | |
| Approver | [To be filled] | [Date] | |

---

## Revision History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 1.0 | [Date] | [Author] | Initial implementation |



