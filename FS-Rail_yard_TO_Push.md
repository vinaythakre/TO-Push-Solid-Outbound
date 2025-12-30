Functional Specification FM: Z_SCM_SOLIDS_OB_GETDET Enhancement:
Populate Rail Yard Source & Destination Locations during TO Push

1.  Purpose The purpose of this enhancement is to populate
    RAILYARDSOURCELOCATION and RAILYARDDESTINATIONLOCATION fields during
    Rate Check performed as part of TO Push in FM
    Z_SCM_SOLIDS_OB_GETDET. Currently, these fields are passed as blank,
    which leads to incorrect or incomplete rate determination. This
    functional specification defines the logic to derive and populate
    these values dynamically based on configuration and shipment data.

2.  Business Background During Transportation Order (TO) push, rate
    check is executed. For rail transportation, rail yard source and
    destination locations are mandatory to correctly determine freight
    rates. Due to missing logic, these fields are not being populated,
    causing rate check issues.

3.  Scope of Change In Scope Enhancement in FM Z_SCM_SOLIDS_OB_GETDET
    Populate fields in structure ET_SOB_CN_DATA Logic applicable during
    TO Push only Rail mode--specific logic based on configuration Out of
    Scope Any change to rate determination logic itself Changes to
    database tables or configuration tables UI or report changes

4.  Affected Objects

5.  Current Behavior (As-Is) During TO Push, Rate Check is triggered.
    Fields below are passed as blank:
    ET_SOB_CN_DATA-RAILYARDSOURCELOCATION
    ET_SOB_CN_DATA-RAILYARDDESTINATIONLOCATION This results in incorrect
    or failed rate determination for rail shipments.

6.  Proposed Behavior (To-Be) During TO Push, system derives and
    populates: Rail Yard Source Location Rail Yard Destination Location
    Logic is executed only when company code conditions are met. Values
    are derived based on route, transport mode, and chain shipment
    configuration.

7.  Detailed Functional Logic Step 1: Read Configuration from
    ZLOG_EXEC_VAR Select records from ZLOG_EXEC_VAR where: NAME =
    'ZSCM_GET_RAIL_LOCATION' ACTIVE = 'X' Retrieve: BUKRS VSART Note:
    Multiple records may exist for the same NAME.

Step 2: Determine Company Code from TTDS Pass VTTK-TPLST to table TTDS
as TTDS-TPLST Retrieve: TTDS-BUKRS

Step 3: Company Code Validation Compare: TTDS-BUKRS ZLOG_EXEC_VAR-BUKRS
If values do not match: Skip the entire rail yard derivation logic If
values match: Continue with Step 4

Step 4: Retrieve Chain Shipment Details Pass VTTK-TKNUM to
ZSCM_CHAINSHIP as: SHNUMBER Retrieve: CHAINID ODPAIRID

Step 5: Identify Shipment for Rail Mode Using values from Step 4 and
configuration: CHAINID ODPAIRID TRMODE = ZLOG_EXEC_VAR-VSART Select from
ZSCM_CHAINSHIP Retrieve: SHNUMBER

Step 6: Retrieve Route from Shipment Pass retrieved SHNUMBER to VTTK
Retrieve: VTTK-ROUTE

Step 7: Determine Rail Yard Locations Pass below values to TVRAB:
TVRAB-VSART = VTTK-VSART TVRAB-ROUTE = VTTK-ROUTE Retrieve: KNANF (Start
Rail Yard) KNEND (End Rail Yard)

Step 8: Populate Output Structure Before appending data to
ET_SOB_CN_DATA, assign values as follows:
ET_SOB_CN_DATA-RAILYARDSOURCELOCATION = TVRAB-KNANF
ET_SOB_CN_DATA-RAILYARDDESTINATIONLOCATION = TVRAB-KNEND

8.  Error Handling & Assumptions If any intermediate step does not
    return data: System should not dump Fields should remain blank Logic
    is executed only when configuration exists and company code matches
    Multiple entries in ZLOG_EXEC_VAR should be processed sequentially
    until a valid match is found
