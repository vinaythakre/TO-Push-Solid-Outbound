# Technical Specification: Rail Yard Location Population for TO Push

## 1. Overview

**Function Module:** Z_SCM_SOLIDS_OB_GETDET  
**Enhancement:** Populate Rail Yard Source & Destination Locations  
**Technical Author:** [To be filled]  
**Creation Date:** [To be filled]  
**Change Request:** [To be filled]

## 2. Purpose

Enhance the function module `Z_SCM_SOLIDS_OB_GETDET` to populate `RAILYARDSOURCELOCATION` and `RAILYARDDESTINATIONLOCATION` fields in the `ET_SOB_CN_DATA` structure during Transportation Order (TO) Push rate check processing.

## 3. Technical Design

### 3.1 Data Structures

#### 3.1.1 Existing Types (Already Defined)
- `lty_zlog_rail_loc`: Configuration structure for rail location
  - Fields: `mandt`, `name`, `active`, `bukrs`, `vsart`
- `lty_chain_shp`: Chain shipment structure
  - Fields: `shnumber`, `chainid`, `odpairid`, `trmode`
- `lty_tvrab_rail`: Rail yard location structure
  - Fields: `route`, `vsart`, `knanf`, `knend`
- `lty_ttds`: Transportation planning point structure
  - Fields: `mandt`, `tplst`, `bukrs`

#### 3.1.2 Additional Local Variables Required
```abap
DATA: lv_rail_bukrs TYPE bukrs,
      lv_rail_vsart TYPE vsart,
      lv_rail_route TYPE route,
      lv_rail_shnumber TYPE tknum,
      lv_rail_knanf TYPE knota,
      lv_rail_knend TYPE knotz,
      lv_rail_active TYPE abap_bool.
```

### 3.2 Database Tables

1. **ZLOG_EXEC_VAR**: Configuration table
   - Key fields: `NAME = 'ZSCM_GET_RAIL_LOCATION'`, `ACTIVE = 'X'`
   - Retrieve: `BUKRS`, `VSART`

2. **TTDS**: Transportation Planning Point
   - Key field: `TPLST = VTTK-TPLST`
   - Retrieve: `BUKRS`

3. **ZSCM_CHAINSHIP**: Chain Shipment table
   - Key field: `SHNUMBER = VTTK-TKNUM`
   - Retrieve: `CHAINID`, `ODPAIRID`
   - Secondary select: `CHAINID`, `ODPAIRID`, `TRMODE = VSART`
   - Retrieve: `SHNUMBER`

4. **VTTK**: Shipment Header
   - Key field: `TKNUM = SHNUMBER` (from chain shipment)
   - Retrieve: `ROUTE`

5. **TVRAB**: Route Assignment
   - Key fields: `VSART = VTTK-VSART`, `ROUTE = VTTK-ROUTE`
   - Retrieve: `KNANF`, `KNEND`

### 3.3 Logic Flow

#### Step 1: Configuration Read (Already Implemented)
- Configuration is read once before CN loop (lines 610-627)
- Stored in `lt_zlog_rail_loc` sorted by `bukrs vsart`

#### Step 2: Company Code Determination
- For each shipment in CN processing loop:
  - Read `TTDS` using `VTTK-TPLST` to get `BUKRS`
  - Use existing `lt_ttds` table if available, otherwise read on demand

#### Step 3: Company Code Validation
- Compare `TTDS-BUKRS` with `ZLOG_EXEC_VAR-BUKRS`
- If match found, continue with rail yard derivation
- If no match, skip rail yard logic (fields remain blank)

#### Step 4: Chain Shipment Details
- Read `ZSCM_CHAINSHIP` using `VTTK-TKNUM` as `SHNUMBER`
- Retrieve `CHAINID` and `ODPAIRID`

#### Step 5: Identify Rail Mode Shipment
- Using `CHAINID`, `ODPAIRID`, and `TRMODE = VSART` from configuration
- Read `ZSCM_CHAINSHIP` to find matching shipment
- Retrieve `SHNUMBER` for rail mode shipment

#### Step 6: Retrieve Route
- Read `VTTK` using rail mode `SHNUMBER`
- Retrieve `ROUTE`

#### Step 7: Determine Rail Yard Locations
- Read `TVRAB` using `VSART` and `ROUTE`
- Retrieve `KNANF` (Start Rail Yard) and `KNEND` (End Rail Yard)

#### Step 8: Populate Output Structure
- Before `APPEND gw_cn_data TO et_sob_cn_data`:
  - `gw_cn_data-railyardsourcelocation = lv_rail_knanf`
  - `gw_cn_data-railyarddestinationlocation = lv_rail_knend`

### 3.4 Error Handling

- All database operations must check `sy-subrc`
- If any step fails, fields remain blank (no dump)
- Logic executes only when:
  - Configuration exists (`lt_zlog_rail_loc` is not initial)
  - Company code matches
- Multiple configuration entries processed sequentially until valid match found

### 3.5 Performance Considerations

- Configuration read once before loop (already implemented)
- Use `FOR ALL ENTRIES` for bulk reads where possible
- Use `BINARY SEARCH` for table reads after sorting
- Avoid `SELECT` in loops - use internal tables

## 4. Code Changes

### 4.1 Location of Changes

**Primary Location:** CN data processing section (around line 2534-3148)

**Insertion Point:** Before `APPEND gw_cn_data TO et_sob_cn_data` (line 3148)

### 4.2 Additional Data Declarations

Add to existing data declarations section (around line 490):
```abap
DATA: lv_rail_bukrs TYPE bukrs,
      lv_rail_vsart TYPE vsart,
      lv_rail_route TYPE route,
      lv_rail_shnumber TYPE tknum,
      lv_rail_chainid TYPE zchainid,
      lv_rail_odpairid TYPE zodpairid,
      lv_rail_knanf TYPE knota,
      lv_rail_knend TYPE knotz,
      lv_rail_active TYPE abap_bool,
      lt_chain_shp TYPE TABLE OF lty_chain_shp,
      lw_chain_shp TYPE lty_chain_shp,
      lt_vttk_rail TYPE TABLE OF vttk,
      lw_vttk_rail TYPE vttk.
```

### 4.3 Implementation Logic

Insert rail yard location derivation logic before line 3148 (before APPEND statement).

## 5. Testing Requirements

### 5.1 Unit Test Scenarios

1. **Configuration exists, company code matches:**
   - Verify rail yard locations are populated

2. **Configuration exists, company code does not match:**
   - Verify fields remain blank

3. **Configuration does not exist:**
   - Verify fields remain blank

4. **Chain shipment not found:**
   - Verify fields remain blank

5. **Rail mode shipment not found:**
   - Verify fields remain blank

6. **TVRAB entry not found:**
   - Verify fields remain blank

### 5.2 Integration Test Scenarios

1. End-to-end TO Push with rail mode shipment
2. Multiple shipments with different company codes
3. Mixed rail and non-rail shipments

## 6. Dependencies

- Existing configuration in `ZLOG_EXEC_VAR` with `NAME = 'ZSCM_GET_RAIL_LOCATION'`
- Chain shipment data in `ZSCM_CHAINSHIP`
- Route data in `TVRAB`
- Transportation planning point data in `TTDS`

## 7. Change History

| Date | Author | Description |
|------|--------|-------------|
| [Date] | [Author] | Initial implementation |
