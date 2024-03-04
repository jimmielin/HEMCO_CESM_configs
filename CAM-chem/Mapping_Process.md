# Process to map GEOS-Chem HEMCO_Config.rc to CAM-chem
Haipeng Lin, September 2022

Mapping also in supplementary material of *Fritz et al., 2022*.

## Scaling factors for mapping in HEMCO-CESM: Install these first

```
#==============================================================================
# --- CEDS to CAM-Chem scale factors ---
#
# NOTES:
#  Reference: Table S4. Emmons, L. K., Schwantes, R. H., Orlando, J. J.,
#    Tyndall, G., Kinnison, D., Lamarque, J.‐F., et al. (2020).
#    The Chemistry Mechanism in the Community Earth System Model version 2 (CESM2).
#    Journal of Advances in Modeling Earth Systems, 12, e2019MS001882.
#    https://doi.org/10.1029/2019MS001882
#
# CEDS sectors: (from CEDS readme)
# sector    Description                                                              SO4_aX SF#         SO4_NUM SF#     BC_NUM SF#    POM_NUM SF#
#   0: AGR  Non-combustion agricultural sector                                          8902                8014
#   1: ENE  Energy transformation and extraction                                        8907                8011
#   2: IND  Industrial combustion and processes                                         8908                8011
#   3: TRA  Surface Transportation (Road, Rail, Other)                                  8906                8022         --8041--      --8042--
#   4: RCO  Residential, commercial, and other                                          8905                8022
#   5: SLV  Solvents                                                                    8904                8014
#   6: WST  Waste disposal and handling                                                 8903                8014
#   7: SHP  International shipping (including VOCs from oil tanker loading/leakage)     8909                8013
#=============================================================================
8902 CESM_SO4a1_AGR  0.025    - - - xy 1 1
8903 CESM_SO4a1_WST  0.025    - - - xy 1 1
8904 CESM_SO4a1_SLV  0.025    - - - xy 1 1
8905 CESM_SO4a2_RCO  0.025    - - - xy 1 1
8906 CESM_SO4a2_TRA  0.025    - - - xy 1 1
8907 CESM_SO4a1_ENE  0.025    - - - xy 1 1
8908 CESM_SO4a1_IND  0.025    - - - xy 1 1
8909 CESM_SO4a1_SHP  0.025    - - - xy 1 1

8910 CESM_CH3CHOCH3  0.2      - - - xy 1 1
8911 CESM_MEK        0.8      - - - xy 1 1
8912 CESM_HCOOH      0.5      - - - xy 1 1
8913 CESM_CH3COOH    0.5      - - - xy 1 1

# BC/OC
8920 CESM_BCa4_ANTH  1.0      - - - xy 1 1
8921 CESM_BCa4_BB    1.0      - - - xy 1 1
8922 CESM_BCa4_AIR   1.0      - - - xy 1 1

8930 CESM_POMa4_ANTH 1.4      - - - xy 1 1
8931 CESM_POMa4_BB   1.4      - - - xy 1 1

# Does not seem to be in CEDS, have to look closer later (hplin, 8/6/20)
8985 CESM_SO4a1_BB   0.025    - - - xy 1 1
8986 CESM_SO4a1_VOL  0.5      - - - xy 1 1
8987 CESM_SO4a2_VOL  0.5      - - - xy 1 1

# Convert CEDS total alcohols to methanol and ethanol following Emmons et al. (2020, JAMES)
8990 CESM_VOC1toMOH 0.15 - - - xy 1 1
8991 CESM_VOC1toEOH 0.85 - - - xy 1 1

#-----------------------------------------
# AEROSOL NUMBER EMISSIONS FOR CEDS
# Based on Emmons et al., 2020 CESM2
#-----------------------------------------
# sector    Description
#   0: AGR  Non-combustion agricultural sector
#   1: ENE  Energy transformation and extraction
#   2: IND  Industrial combustion and processes
#   3: TRA  Surface Transportation (Road, Rail, Other)
#   4: RCO  Residential, commercial, and other
#   5: SLV  Solvents
#   6: WST  Waste disposal and handling
#   7: SHP  International shipping (including VOCs from oil tanker loading/leakage)
# The scaling is performed ON TOP of original bins to convert aerosol mass to aerosol number
#
#     Enumber = Emass * (1 / ( pi/6 * rho * D_emit^3 ))
#
#              kg/m2/s               kg/m3   um^3=10^-18 m^3
#             =  1/m2/s
#
# where pi = 3.1415926
#       rho = specdens_amode(SM,M) --> species corresponding to the given aerosol mode
#       D_emit is dependent by sector
#
#  WARNING: THE FOLLOWING FACTORS ARE APPLIED ON TOP OF THE EXISTING CESM SCALE FACTORS FOR CEDS (i.e. 0.025)
#  THUS THEY SHOULD ALWAYS BE USED IN COMBINATION, e.g., 908/8001
#
# Refer to the parameter table in Emmons et al., 2020 SUPPLEMENT
#
#     1/(3.1415926/6*1770*(0.261e-6)**3)
8011  CESM_SO4a1toNUM_ENE_IND 6.068853e16  - - - xy 1 1
8013  CESM_SO4a1toNUM_SHP     6.068853e16  - - - xy 1 1
#     1/(3.1415926/6*1770*(0.134e-6)**3)
#  BB, Volcano, Agr, Wst, Solvents
8013  CESM_SO4a1toNUM_BB_VOL  4.484497e17  - - - xy 1 1
8014  CESM_SO4a1toNUM_AG_WS   4.484497e17  - - - xy 1 1
#     1/(3.1415926/6*1770*(0.0504e-6)**3)
8021  CESM_SO4a2toNUM_VOL     8.428233e18  - - - xy 1 1
#     1/(3.1415926/6*1770*(0.0504e-6)**3)
8022  CESM_SO4a2toNUM_RES_TRA 8.428233e18  - - - xy 1 1

# BC total anthro, BB, aircraft
#     1/(3.1415926/6*1770*(0.134e-6)**3)
8041  CESM_BCa4toNUM          4.484497e17  - - - xy 1 1
#     1/(3.1415926/6*1000*(0.134e-6)**3)
8042  CESM_OCa4toNUM          7.937559e17  - - - xy 1 1
```

## Straightforward species

Map the following species directly to CAM-chem names. **Do not find and replace directly - some netCDF variable names also use GEOS-Chem species names.**

* ACET -> CH3COCH3
* ALD2 -> CH3CHO
* BENZ -> BENZENE
* TOLU -> TOLUENE
* XYLE -> XYLENES

## Nonexact replacement

These replacements come with caveats.

* PRPE -> C3H6. Actually, GEOS-Chem PRPE is lumped >= C3 alkenes and not exactly C3H6.
* ALK4 -> BIGALK. Note that the number of carbons does not exactly align between GEOS-Chem ALK4 (Lumped >= C4 Alkanes) and CAM-chem BIGALK.
* SO4 -> `so4_a1` or `so4_a2`. For residential and transportation and volcanoes, use `so4_a2`. Otherwise, `so4_a1`. Careful not to use `63` and `890x` at the same time, they serve the same purpose to scale SO2 to SO4 in CEDS and CEDSv2.

### MOH, EOH, ROH

ROH is not in CAM-chem. For MOH (CH3OH), EOH (C2H5OH)

* MOH -> CH3OH
* EOH -> C2H5OH

In CAM-chem (Emmons et al., 2020), MOH and EOH are 0.15 and 0.85 of total VOC01-alcohols. This is handled in GEOS-Chem HEMCO-config through scaling factors 90/91/92 for MOH/EOH/ROH split. In CAM-chem, use `8990/8991` to make the 0.15/0.85 split.

### Taking care of aerosols

Note that we are using MAM4 aerosols. e.g., SO4 emissions need to be emitted both as aerosols (`so4_aX`) and into the corresponding aerosol number bin (`num_aX`) using conversion factors 8011 to 8042.

For GEOS-Chem, only the numbers need to be emitted separately (`num_aX`) as the species themselves (`SO4` -> `so4_aX`, `OCPI` -> `pom_aX` ..) will be mapped into MAM4 inside `chemistry.F90`.

## Dust bins

Generally not needed because dust is handled by CLM, not HEMCO. However, Fritz et al. 2022 indicates that `DST1` is to be mapped to `dst_a1 + dst_a2`, and `DST4` is `dst_a3`. This may be needed for some inventories which directly emit dust.

## BC, OC

* Map BCPO **and** BCPI to `bc_a4`.
* Map OCPO **and** OCPI to `pom_a4`.

## Extensions handling

### GFED, FINN fire emissions

Map BC, OC to `a4`-size bin, and apply scaling:
```
    --> Scaling_bc_a4          :       1.0
    --> Scaling_pom_a4         :       1.4
```

### Dust emissions

Disable, unsupported. Handled by CLM.

### Biogenic emissions

Disable, unsupported. Handled by CLM.

### Seasalt emissions

Disable, handled by CLM. If seasalt debromination is desired (`BrSALA`, `BrSALC`) emissions, they should be scaled according to the `SALA` and `SALC` fluxes for consistency, to avoid inaccuracy. Refer to Fritz et al., 2022 for the reason.

The offline GEOS-Chem seasalt emissions uses these factors:

```
# NOTES:
# - Sea salt alkalinity and chloride values obtained from hcox_seasalt_mod.F90
# - BrContent obtained from '--> Br- mass ratio' in SeaSalt extension above
615 SSAlkalinity 1.0     - - - xy 1 1
616 SSChloride   0.5504  - - - xy 1 1
617 BrContent    2.11e-3 - - - xy 1 1
```

Which are used to scale the `SALA_TOTAL` and `SALC_TOTAL` to `SALAAL/SALCAL` (`615`), `SALACL/SALCCL` (`616`), and `BrSALA/BrSALC` (`617`), respectively. This scaling must be applied to the CESM on-line seasalt emissions as well.

## Additional handling to ketones and acids for CMIP6 emissions (only for reference run)

If not comparing with GEOS-Chem and running HEMCO for direct replacement for the CMIP6 emissions in CAM-chem, the following indications must be observed:

* Ketones are split 0.2:0.8 into CH3COCH3 and MEK. Use `8910` and `8911`.
* Acids are split 0.5:0.5 into HCOOH and CH3COOH. Use `8912` and `8913`.

## Citations

Emmons, L. K., Schwantes, R. H., Orlando, J. J., Tyndall, G., Kinnison, D., Lamarque, J.-F., et al. (2020). The Chemistry Mechanism in the Community Earth System Model version 2 (CESM2). Journal of Advances in Modeling Earth Systems, 12, e2019MS001882. https://doi.org/10.1029/2019MS001882

Supplement: https://agupubs.onlinelibrary.wiley.com/action/downloadSupplement?doi=10.1029%2F2019MS001882&file=jame21103-sup-0001-2019MS001882+Text_SI-S01.pdf

Liu, X., Ma, P.-L., Wang, H., Tilmes, S., Singh, B., Easter, R. C., Ghan, S. J., and Rasch, P. J.: Description and evaluation of a new four-mode version of the Modal Aerosol Module (MAM4) within version 5.3 of the Community Atmosphere Model, Geosci. Model Dev., 9, 505–522, https://doi.org/10.5194/gmd-9-505-2016, 2016.

