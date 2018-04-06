*--------------------------------------------------------------------*
* Partner data
*--------------------------------------------------------------------*
DATA:
  lv_partner  TYPE          bu_partner,
  ls_but000   TYPE          but000,
  lv_addrnum  TYPE          ad_addrnum,
  ls_addruse  TYPE          bus021,
  lt_addruse  TYPE TABLE OF bus021.

*--------------------------------------------------------------------*
* SD document data
*--------------------------------------------------------------------*
DATA:
  lv_vbeln    TYPE          bapivbeln-vbeln,
  ls_header   TYPE          bapisdhead,
  lv_posnr    TYPE          posnr_va,
  ls_item     TYPE          bapiitemin,
  lt_item     TYPE TABLE OF bapiitemin,
  ls_schedule TYPE          bapisdhedu,
  lt_schedule TYPE TABLE OF bapisdhedu,
  ls_partner  TYPE          bapipartnr,
  lt_partner  TYPE TABLE OF bapipartnr,
  lv_subrc    TYPE          sy-subrc,
  ls_return   TYPE          bapireturn1.

*--------------------------------------------------------------------*
* Macro: ADD_ITEM
*--------------------------------------------------------------------*
DEFINE add_item.
  "importing
  "  &1 material
  "  &2 target_qty
  "  &3 cond_value

  lv_posnr = lv_posnr + 10.

  CLEAR ls_item.
  ls_item-itm_number = lv_posnr.
  ls_item-plant      = '<..>'.
  ls_item-material   = &1.
  ls_item-target_qty = &2.
  ls_item-target_qu  = '<..>'.
  ls_item-req_qty    = &2.
  ls_item-currency   = '<..>'.
  ls_item-cond_type  = '<..>'.
  ls_item-cond_value = &3.
  ls_item-cond_p_unt = 1.
  APPEND ls_item TO lt_item.

  " Schedule line
  CLEAR ls_schedule.
  ls_schedule-itm_number = lv_posnr.
  ls_schedule-req_qty    = &2.
  APPEND ls_schedule TO lt_schedule.
END-OF-DEFINITION.

*--------------------------------------------------------------------*
* SD document header
*--------------------------------------------------------------------*
CLEAR ls_header.
ls_header-doc_type    = '<..>'.
ls_header-sales_org   = '<..>'.
ls_header-distr_chan  = '<..>'.
ls_header-division    = '<..>'.
ls_header-price_date  = sy-datum.
ls_header-currency    = '<..>'.

*--------------------------------------------------------------------*
* SD document items
*--------------------------------------------------------------------*
*         material  target_qty   cond_value
add_item: '<..>'    '1.000'      <..>.

*--------------------------------------------------------------------*
* SD document partners
*--------------------------------------------------------------------*
lv_partner = '<..>'.

CLEAR ls_but000.

CALL FUNCTION 'BUP_BUT000_SELECT_SINGLE'
  EXPORTING
    i_partner       = lv_partner
  IMPORTING
    e_but000        = ls_but000
  EXCEPTIONS
    not_found       = 1
    internal_error  = 2
    blocked_partner = 3
    OTHERS          = 4.

IF sy-subrc NE 0.
  RETURN.
ENDIF.

CLEAR lt_addruse.

CALL FUNCTION 'BUA_ADDRESS_GET_ALL'
  EXPORTING
    i_partner        = lv_partner
  TABLES
    t_addruse        = lt_addruse
  EXCEPTIONS
    no_address_found = 1
    wrong_parameters = 2
    internal_error   = 3
    date_invalid     = 4
    not_valid        = 5
    partner_blocked  = 6
    OTHERS           = 7.

IF sy-subrc NE 0.
  RETURN.
ENDIF.

LOOP AT lt_addruse INTO ls_addruse.
  lv_addrnum = ls_addruse-addrnumber.
  EXIT. "loop
ENDLOOP.

CLEAR ls_partner.
CLEAR lt_partner.

CASE ls_but000-type.
  WHEN 1.
    ls_partner-name   = ls_but000-name_first.
    ls_partner-name_2 = ls_but000-name_last.
  WHEN 2.
    ls_partner-name   = ls_but000-name_org1.
    ls_partner-name_2 = ls_but000-name_org2.
    ls_partner-name_3 = ls_but000-name_org3.
    ls_partner-name_4 = ls_but000-name_org4.
ENDCASE.

ls_partner-partn_role = 'RE'.
ls_partner-partn_numb = lv_partner.
ls_partner-address    = lv_addrnum.
APPEND ls_partner TO lt_partner.

ls_partner-partn_role = 'AG'.
ls_partner-partn_numb = lv_partner.
ls_partner-address    = lv_addrnum.
APPEND ls_partner TO lt_partner.

*--------------------------------------------------------------------*
* Call BAPI
*--------------------------------------------------------------------*
CLEAR:
  lv_vbeln,
  ls_return.

CALL FUNCTION 'BAPI_SALESDOCU_CREATEFROMDATA'
  EXPORTING
    order_header_in   = ls_header
    business_object   = '<..>' "'BUS2102'
    without_commit    = 'X'
  IMPORTING
    salesdocument     = lv_vbeln
    return            = ls_return
  TABLES
    order_items_in    = lt_item
    order_partners    = lt_partner
    order_schedule_ex = lt_schedule.

*--------------------------------------------------------------------*
* Commit or Rollback
*--------------------------------------------------------------------*
IF ls_return-type CA 'EAX'.
  lv_subrc = 4.
ENDIF.

CASE lv_subrc.
  WHEN 0.
    CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
      EXPORTING
        wait = 'X'.
    MESSAGE s001(00) WITH 'SD document is saved.' lv_vbeln.

  WHEN 4.
    CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
    MESSAGE s001(00) WITH 'SD document is not saved!'.
ENDCASE.
