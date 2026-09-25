# new
 METHOD get_relevant_charges.
************************************************************************
* Author      : Saurabh Singh Net Id: CG22512
* Date        : 18.05.2026
* Reference   : New method
* Transport/RT: ARDK943771 / RT 14417
* FS No       : D1316
* Description : Filter relevant credits based on Regime and charge type input
************************************************************************

   DATA: lt_excl_receiv TYPE SORTED TABLE OF zexcl_receivable WITH NON-UNIQUE KEY main_trans sub_trans.

   et_dfkkop = it_dfkkop.
* Get Contract account category for given regime
   IF gv_regime IS NOT INITIAL.
     SELECT vktyp UP TO 1 ROWS  FROM ztregime2cacorev  INTO  @DATA(lv_vktyp)
     WHERE regime = @gv_regime ORDER BY regime, vktyp.
     ENDSELECT.
     IF sy-subrc = 0 .
       DELETE et_dfkkop WHERE vktyp_ps <> lv_vktyp.
     ENDIF.
   ENDIF.
   IF lv_vktyp IS NOT INITIAL AND et_dfkkop IS NOT INITIAL.
* Select everything from the exclusion table
     SELECT SINGLE receivables_ex_variant FROM ztetmp_ui5_ca WHERE ca_category = @lv_vktyp INTO @DATA(lv_tabstrip_variant).
     IF sy-subrc EQ 0.
       SELECT mandt, variant, func_seq, doc_type, main_trans,  sub_trans FROM zexcl_receivable
         WHERE variant EQ @lv_tabstrip_variant ORDER BY main_trans, sub_trans
         INTO TABLE @lt_excl_receiv .
       IF sy-subrc EQ 0.
         LOOP AT et_dfkkop ASSIGNING FIELD-SYMBOL(<fs_dfkkop>).
           READ TABLE lt_excl_receiv   WITH KEY main_trans = <fs_dfkkop>-hvorg
                                                sub_trans  = <fs_dfkkop>-tvorg TRANSPORTING NO FIELDS.
           IF sy-subrc = 0.
             DELETE et_dfkkop.
           ENDIF.
         ENDLOOP.
       ENDIF.
     ENDIF.
   ENDIF.

 ENDMETHOD.
