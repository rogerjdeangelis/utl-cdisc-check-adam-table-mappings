# utl-cdisc-check-adam-table-mappings
Given production tables sdtm.dm, sdtm.vs and production adam sl check the mappings programatically
    %let pgm=utl-cdisc-check-adam-table-mappings;

    /*

    Given production CDISC tables sdtm.dm, sdtm.vs and production adam sl check the mappings programatically.
    Thi is done using reverse engineering the mapping without using the specs.

    github
    https://tinyurl.com/mrf28b9j
    https://github.com/rogerjdeangelis/utl-cdisc-check-adam-table-mappings

    You need  download this base64 encoded zip file and, run the macro at the end of this readme to decode.
    Github does not work well with binary data.
    https://github.com/rogerjdeangelis/utl-cdisc-check-adam-table-mappings/blob/main/smpFolDer.b64

    and then run this to create the 7z zip file, then unzip for the package.

    %utl_b64decode(d:/smp/b64/smpFolDer.b64,d:/smp/zip/smp.7z);

    The voodoo macro will provide the mappings.

    The macro can find

        1. Exact mappings of sdtm variables to adam variables(sdtm variable that equals adam variable on all records).
        2. Finds all 1:1 mappings
        3. Can find adam variable equal to a linear combination of two sdtm variables (bmi ~ height weight)
        4. Finds all very high correlations especially 100% correlations
        5. Finds all charcater variables with very high correspondence Cramer V statistics ( like test and test code)

    Process flow
        1. Identify the sdtm domains needed to create the adam dataset. ( Iam missing a couple, this is just a sample).
        2. Rename all the veriables in each sdtm with the prefix domain name ie dm_age
        3. Rename all variables in the adam datasets ie adsl rename sl_studyid ...
        4. Run voodoo macro
        5  Examine voodoo output particularly the findings above see .\smp\vdo\smp_010adsl_dm_vs.txt
     _                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|

    *x 'tree "d:/smp" /F /A | clip';


    D:\SMP

    +---adm
    |       smpadm_adsl.sas7bdat       ==> adam table to find and check mapping
    |
    +---sas
    |       smp_voodoo.sas   (also autocall library)
    |       ...
    +---vdo
    |       smp_010adsl_dm_vs.txt    ==> voodoo listing output
    |       smp_010adsl_dm_vs.log    ==> voodoo output
    |
    \---sdm
            smpsdm_dm.sas7bdat        ===> sdtm input datasets
            smpsdm_vs.sas7bdat
                 _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|

    d:/smp/vdo/smp_010adsl_dm_vs.txt

    The listing output from the smp_voodoo macro can be used as a check mappings.
    This is just a simple example. If you add all the SDTMs that are input to ADAM.ADSL,
    you should be able to get almost all the ADSL vatriables= mappings.

    Using just inputs SDTM_VS and DM.

    The following variables are uniformly evaluated with NO missing values
    a single non-missing value is present for all observations

          #    Variable       Value                             Label
        ---    -----------    ------------------------------    ----------------------------------------

          1    SL_AGEU        YEARS                             Age Units
          2    SL_ITTFL       Y                                 Intent-To-Treat Population Flag
          3    SL_SAFFL       Y                                 Safety Population Flag
          4    SL_STUDYID     CDISCPILOT01                      Study Identifier

     
    These character variables have equal values for all observations (you cn add to this by adding more sdtm tables)

    Obs    BATCH

      1    Variables with All Equal Values
      2
      3    Variable     Type  Len   Compare      Len   Label                                Compare Label
      4
      5    SL_STUDYID   CHAR   12   DM_STUDYID    12   Study Identifier                     Study Identifier
      8    SL_USUBJID   CHAR   11   VS_USUBJID    11   Unique Subject Identifier            Unique Subject Identifier
      9    SL_SUBJID    CHAR    4   DM_SUBJID      4   Subject Identifier for the Study     Subject Identifier for the Study
     10    SL_SITEID    CHAR    3   DM_SITEID      3   Study Site Identifier                Study Site Identifier
     13    SL_ARM       CHAR   20   DM_ARM        20   Description of Planned Arm           Description of Planned Arm
     15    SL_TRT01P    CHAR   20   DM_ARM        20   Planned Treatment for Period 01      Description of Planned Arm
     16    SL_TRT01A    CHAR   20   DM_ARM        20   Actual Treatment for Period 01       Description of Planned Arm
     17    SL_AGEU      CHAR    5   DM_AGEU        5   Age Units                            Age Units
     18    SL_RACE      CHAR   32   DM_RACE       32   Race                                 Race
     19    SL_SEX       CHAR    1   DM_SEX         1   Sex                                  Sex
     20    SL_ETHNIC    CHAR   22   DM_ETHNIC     22   Ethnicity                            Ethnicity
     24    SL_DTHFL     CHAR    1   DM_DTHFL       1   Subject Died?                        Subject Death Flag
     25    SL_RFSTDTC   CHAR   20   DM_RFSTDTC    10   Subject Reference Start Date/Time    Subject Reference Start Date/Time
     26    SL_RFSTDTC   CHAR   20   DM_RFXSTDTC   10   Subject Reference Start Date/Time    Date/Time of First Study Treatment
     28    SL_RFENDTC   CHAR   20   DM_RFENDTC    10   Subject Reference End Date/Time      Subject Reference End Date/Time
     22    SL_SAFFL     CHAR    1   VS_VSBLFL      1   Safety Population Flag               Baseline Flag
     23    SL_ITTFL     CHAR    1   VS_VSBLFL      1   Intent-To-Treat Population Flag      Baseline Flag
     
    These numeric variables have equal values for all observations

    Obs    BATCH

     1     Variables with All Equal Values
     2
     3     Variable     Type  Len   Compare      Len   Label                                 Compare Label
     4
     5     SL_TRT01PN   NUM     8   SL_TRT01AN     8   Planned Treatment for Period 01 (N)   Actual Treatment for Period 01 (N)
     6     SL_AGE       NUM     8   DM_AGE         8   Age                                   Age
                             _       _   _
      ___ ___  _ __ _ __ ___| | __ _| |_(_) ___  _ __  ___
     / __/ _ \| `__| `__/ _ \ |/ _` | __| |/ _ \| `_ \/ __|
    | (_| (_) | |  | | |  __/ | (_| | |_| | (_) | | | \__ \
     \___\___/|_|  |_|  \___|_|\__,_|\__|_|\___/|_| |_|___/

    These num variables have equal values for all observations

    Variable Correlations (Spearman)

                   Correlated     Correlation    Number    Spearman
    Variable       With               Coef       of Obs       P

    DM_AGE         SL_AGE           1.00000        254      0.0127
    SL_TRT01AN     SL_TRT01PN       1.00000        254      0.9766

    Variable Correlations (Spearman)

       Plot of DM_AGE*SL_AGE.  Legend: A = 1 obs, B = 2 obs, etc.

         -+-------------+-------------+-------------+-------------+-
         |                                                         |
         |                                                         |
      90 +                                                         +
         |                                                     D A |
         |                                                  G F    |
         |                                                 F       |
         |                                              H P        |
         |                                           S J           |
      80 +                                          K              +
         |                                       M N               |
         |                                    L N                  |
         |                                   H                     |
    A    |                                I M                      |
    g    |                             I I                         |
    e 70 +                            E                            +
         |                         G G                             |
         |                      A H                                |
         |                     D                                   |
         |                  D E                                    |
         |               E B                                       |
      60 +              C                                          +
         |             B                                           |
         |        F C                                              |
         |                                                         |
         |      A                                                  |
         | A A                                                     |
      50 +                                                         +
         |                                                         |
         -+-------------+-------------+-------------+-------------+-
         50            60            70            80            90

                                     Age
                                                        _
      ___ ___  _ __ _ __ ___  ___ _ __   ___  _ __   __| | ___ _ __   ___ ___
     / __/ _ \| `__| `__/ _ \/ __| `_ \ / _ \| `_ \ / _` |/ _ \ `_ \ / __/ _ \
    | (_| (_) | |  | | |  __/\__ \ |_) | (_) | | | | (_| |  __/ | | | (_|  __/
     \___\___/|_|  |_|  \___||___/ .__/ \___/|_| |_|\__,_|\___|_| |_|\___\___|
                                 |_|
    Cramer V
    ALL PAIRS OF VARIABLES WHERE MAX LEVELS IS 2000 AND MAX NUMBER OF VARIABLES IS 30
    DM_ETHNIC * DM_RACE

    Obs                 TABLE                 STATISTIC     VALUE

      1    Table DM_AGE * SL_AGE              Cramer's V    1.0000
      2    Table SL_RFENDT * VS_VSSEQ         Cramer's V    1.0000
      3    Table SL_DISONSDT * DM_DTHDTC      Cramer's V    1.0000
      4    Table SL_TRTSDT * DM_DTHDTC        Cramer's V    1.0000
      5    Table SL_VISIT1DT * DM_DTHDTC      Cramer's V    1.0000
      6    Table DM_AGE * SL_AGEGR1N          Cramer's V    1.0000
      7    Table SL_AGE * SL_AGEGR1N          Cramer's V    1.0000
      8    Table SL_AVGDD * DM_ACTARM         Cramer's V    1.0000
      9    Table SL_AVGDD * DM_ACTARMCD       Cramer's V    1.0000
     10    Table SL_TRT01AN * SL_TRT01PN      Cramer's V    1.0000
     11    Table SL_TRT01AN * DM_ARM          Cramer's V    1.0000
     12    Table SL_TRT01AN * DM_ARMCD        Cramer's V    1.0000
     13    Table SL_TRT01PN * DM_ARM          Cramer's V    1.0000
     14    Table SL_TRT01PN * DM_ARMCD        Cramer's V    1.0000
     15    Table DM_ACTARM * DM_ACTARMCD      Cramer's V    1.0000
     16    Table DM_ARM * DM_ARMCD            Cramer's V    1.0000
     _       _
    / |  _  / |
    | | (_) | |
    | |  _  | |
    |_| (_) |_|


     
    Relationship OF VARIABLES WHERE MAX LEVELS IS 2000 AND MAX NUMBER OF VARIABLES IS 100
    One to One  -- One to Many  --  Many to One


       2    One to One      SL_AGE to DM_AGE
     237    One to One      SL_AGEGR1N to SL_AGEGR1
     807    One to One      SL_RACEN to SL_RACE
     890    One to One      SL_TRT01AN to SL_TRT01PN
     901    One to One      SL_TRT01AN to DM_ARM
     902    One to One      SL_TRT01AN to DM_ARMCD
     909    One to One      SL_TRT01AN to SL_ARM
     924    One to One      SL_TRT01AN to SL_TRT01A
     925    One to One      SL_TRT01AN to SL_TRT01P
    1157    One to One      SL_TRTSDT to VS_VSDTC
    1153    One to One      SL_TRTSDT to SL_RFSTDTC

    */

    /*   _             _
     ___| |_ __ _ _ __| |_
    / __| __/ _` | `__| __|
    \__ \ || (_| | |  | |_
    |___/\__\__,_|_|   \__|
              _
     ___  ___| |_ _   _ _ __
    / __|/ _ \ __| | | | `_ \
    \__ \  __/ |_| |_| | |_) |
    |___/\___|\__|\__,_| .__/
                       |_|
    */

    filename sasautos clear;
    filename sasautos "d:/smp/sas";

    %put %sysfunc(pathname(sasautos));

    libname smpSdm "d:/smp/sdm";  * delete;
    libname smpAdm "d:/smp/adm";

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|        _                             _     _
      __ _  __| | __ _ _ __ ___     __ _  __| |___| |
     / _` |/ _` |/ _` | `_ ` _ \   / _` |/ _` / __| |
    | (_| | (_| | (_| | | | | | | | (_| | (_| \__ \ |
     \__,_|\__,_|\__,_|_| |_| |_|  \__,_|\__,_|___/_|

    */

    proc datasets lib=work kill mt=data nodetails nolist;
    run;quit;

    %utlfkil(d:/smp/vdo/smp_010adsl_dm_vs.txt);
    %utlfkil(d:/smp/vdo/smp_010adsl_dm_vs.log);

    %utlnopts; /* turn off log */

    * rename all variables in input datasets;

    * get adls variable names;
    %array(_sl,values=%utl_varlist(smpAdm.smpAdm_adsl));

    %put &=_sl1; /* studyid */
    %put &=_sln; /* 49 variables */

    data adsl;
      set smpAdm.smpAdm_adsl (rename=(%do_over(_sl,phrase=%str(?=sl_?))));
    run;quit;

    * delete macro array;
    %arraydelete(_sl);

    /*                                       _             _     _
     _ __ ___ _ __   __ _ _ __ ___   ___  __| |   __ _  __| |___| |
    | `__/ _ \ `_ \ / _` | `_ ` _ \ / _ \/ _` |  / _` |/ _` / __| |
    | | |  __/ | | | (_| | | | | | |  __/ (_| | | (_| | (_| \__ \ |
    |_|  \___|_| |_|\__,_|_| |_| |_|\___|\__,_|  \__,_|\__,_|___/_|

    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  Middle Observation(127 ) of Last dataset = WORK.ADSL - Total Obs 254                                                  */
    /*                                                                                                                        */
    /*   -- CHARACTER --                                                                                                      */
    /*  SL_STUDYID                       C12      CDISCPILOT01        Study Identifier                                        */
    /*  SL_USUBJID                       C11      01-708-1353         Unique Subject Identifier                               */
    /*  SL_SUBJID                        C4       1353                Subject Identifier for the Study                        */
    /*  SL_SITEID                        C3       708                 Study Site Identifier                                   */
    /*  SL_SITEGR1                       C3       708                 Pooled Site Group 1                                     */
    /*  SL_ARM                           C20      Xanomeline Low D    Description of Planned Arm                              */
    /*  SL_TRT01P                        C20      Xanomeline Low D    Planned Treatment for Period 01                         */
    /*  SL_TRT01A                        C20      Xanomeline Low D    Actual Treatment for Period 01                          */
    /*  SL_AGEGR1                        C5       >80                 Pooled Age Group 1                                      */
    /*  SL_AGEU                          C5       YEARS               Age Units                                               */
    /*  SL_RACE                          C32      BLACK OR AFRICAN    Race                                                    */
    /*  SL_SEX                           C1       F                   Sex                                                     */
    /*  SL_ETHNIC                        C22      NOT HISPANIC OR     Ethnicity                                               */
    /*  SL_SAFFL                         C1       Y                   Safety Population Flag                                  */
    /*  SL_ITTFL                         C1       Y                   Intent-To-Treat Population Flag                         */
    /*  SL_EFFFL                         C1       Y                   Efficacy Population Flag                                */
    /*  SL_COMP8FL                       C1       Y                   Completers of Week 8 Population Flag                    */
    /*  SL_COMP16FL                      C1       N                   Completers of Week 16 Population Flag                   */
    /*  SL_COMP24FL                      C1       N                   Completers of Week 24 Population Flag                   */
    /*  SL_DISCONFL                      C1       Y                   Did the Subject Discontinue the Study?                  */
    /*  SL_DSRAEFL                       C1       Y                   Discontinued due to AE?                                 */
    /*  SL_DTHFL                         C1                           Subject Died?                                           */
    /*  SL_BMIBLGR1                      C6       <25                 Pooled Baseline BMI Group 1                             */
    /*  SL_DURDSGR1                      C4       >=12                Pooled Disease Duration Group 1                         */
    /*  SL_RFSTDTC                       C20      2013-07-04          Subject Reference Start Date/Time                       */
    /*  SL_RFENDTC                       C20      2013-09-10          Subject Reference End Date/Time                         */
    /*  SL_DCDECOD                       C27      ADVERSE EVENT       Standardized Disposition Term                           */
    /*  SL_EOSSTT                        C12      DISCONTINUED        End of Study Status                                     */
    /*  SL_DCSREAS                       C18      Adverse Event       Reason for Discontinuation from Study                   */
    /*                                                                                                                        */
    /*   -- NUMERIC --                                                                                                        */
    /*  SL_TRT01PN                       N8       54                  Planned Treatment for Period 01 (N)                     */
    /*  SL_TRT01AN                       N8       54                  Actual Treatment for Period 01 (N)                      */
    /*  SL_TRTSDT                        N8       19543               Date of First Exposure to Treatment                     */
    /*  SL_TRTEDT                        N8       19598               Date of Last Exposure to Treatment                      */
    /*  SL_TRTDURD                       N8       56                  Total Treatment Duration (Days)                         */
    /*  SL_AVGDD                         N8       54                  Avg Daily Dose (as planned)                             */
    /*  SL_CUMDOSE                       N8       3024                Cumulative Dose (as planned)                            */
    /*  SL_AGE                           N8       87                  Age                                                     */
    /*  SL_AGEGR1N                       N8       3                   Pooled Age Group 1 (N)                                  */
    /*  SL_RACEN                         N8       2                   Race (N)                                                */
    /*  SL_BMIBL                         N8       20.3                Baseline BMI (kg/m^2)                                   */
    /*  SL_HEIGHTBL                      N8       157.5               Baseline Height (cm)                                    */
    /*  SL_WEIGHTBL                      N8       50.4                Baseline Weight (kg)                                    */
    /*  SL_EDUCLVL                       N8       16                  Years of Education                                      */
    /*  SL_DISONSDT                      N8       18480               Date of Onset of Disease                                */
    /*  SL_DURDIS                        N8       34.4                Duration of Disease (Months)                            */
    /*  SL_VISIT1DT                      N8       19526               Date of Visit 1                                         */
    /*  SL_VISNUMEN                      N8       8                   End of Trt Visit (Vis 12 or Early Term.)                */
    /*  SL_RFENDT                        N8       19611               Date of Discontinuation/Completion                      */
    /*  SL_MMSETOT                       N8       16                  MMSE Total                                              */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*       _ _                   _
     ___  __| | |_ _ __ ___     __| |_ __ ___
    / __|/ _` | __| `_ ` _ \   / _` | `_ ` _ \
    \__ \ (_| | |_| | | | | | | (_| | | | | | |
    |___/\__,_|\__|_| |_| |_|  \__,_|_| |_| |_|

    */

    * get adls variable names;
    %array(_dm,values=%utl_varlist(smpSdm.smpSdm_dm));

    %put &_dm1; /* studyid */
    %put &_dmn; /* 25 variables */

    data dm ;
      set smpSdm.smpSdm_dm(rename=(%do_over(_dm,phrase=%str(?=dm_?))));
    run;quit;
    /*----- Number of unique levels=306 for dm_usubjid from dm (obs=306) 06SEP2022:11:57:49 -----*/

    * delete macro array;
    %arraydelete(_dm);

    /*                                         _
     _ __ ___ _ __   __ _ _ __ ___   ___    __| |_ __ ___
    | `__/ _ \ `_ \ / _` | `_ ` _ \ / _ \  / _` | `_ ` _ \
    | | |  __/ | | | (_| | | | | | |  __/ | (_| | | | | | |
    |_|  \___|_| |_|\__,_|_| |_| |_|\___|  \__,_|_| |_| |_|

    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Middle Observation(153 ) of Last dataset = WORK.DM - Total Obs 306                                                     */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  -- CHARACTER --                                                                                                       */
    /* DM_STUDYID                       C12      CDISCPILOT01        Study Identifier                                         */
    /* DM_DOMAIN                        C2       DM                  Domain Abbreviation                                      */
    /* DM_USUBJID                       C11      01-708-1353         Unique Subject Identifier                                */
    /* DM_SUBJID                        C4       1353                Subject Identifier for the Study                         */
    /* DM_RFSTDTC                       C10      2013-07-04          Subject Reference Start Date/Time                        */
    /* DM_RFENDTC                       C10      2013-09-10          Subject Reference End Date/Time                          */
    /* DM_RFXSTDTC                      C10      2013-07-04          Date/Time of First Study Treatment                       */
    /* DM_RFXENDTC                      C10      2013-08-28          Date/Time of Last Study Treatment                        */
    /* DM_RFICDTC                       C1                           Date/Time of Informed Consent                            */
    /* DM_RFPENDTC                      C16      2013-09-10T13:15    Date/Time of End of Participation                        */
    /* DM_DTHDTC                        C10                          Date/Time of Death                                       */
    /* DM_DTHFL                         C1                           Subject Death Flag                                       */
    /* DM_SITEID                        C3       708                 Study Site Identifier                                    */
    /* DM_AGEU                          C5       YEARS               Age Units                                                */
    /* DM_SEX                           C1       F                   Sex                                                      */
    /* DM_RACE                          C32      BLACK OR AFRICAN    Race                                                     */
    /* DM_ETHNIC                        C22      NOT HISPANIC OR     Ethnicity                                                */
    /* DM_ARMCD                         C8       Xan_Lo              Planned Arm Code                                         */
    /* DM_ARM                           C20      Xanomeline Low D    Description of Planned Arm                               */
    /* DM_ACTARMCD                      C8       Xan_Lo              Actual Arm Code                                          */
    /* DM_ACTARM                        C20      Xanomeline Low D    Description of Actual Arm                                */
    /* DM_COUNTRY                       C3       USA                 Country                                                  */
    /* DM_DMDTC                         C10      2013-06-17          Date/Time of Collection                                  */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  -- NUMERIC --                                                                                                         */
    /* DM_AGE                           N8       87                  Age                                                      */
    /* DM_DMDY                          N8       -17                 Study Day of Collection                                  */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*       _ _
     ___  __| | |_ _ __ ___   __   _____
    / __|/ _` | __| `_ ` _ \  \ \ / / __|
    \__ \ (_| | |_| | | | | |  \ V /\__ \
    |___/\__,_|\__|_| |_| |_|   \_/ |___/

    */


    * get adls variable names;
    %array(_vs,values=%utl_varlist(smpSdm.smpSdm_vs));

    %put &=_vs5; /* VSTESTCD */
    %put &=_vsn; /* 25 variables */

    data vs ;
      set smpSdm.smpSdm_vs(rename=(%do_over(_vs,phrase=%str(?=vs_?))));
    run;quit;
    /*----- Number of unique levels=254 for vs_usubjid from vs (obs=29,643) 06SEP2022:11:56:50 -----*/

    * delete macro array;
    %arraydelete(_vs);

    /*
     _ __ ___ _ __   __ _ _ __ ___   ___  __   _____
    | `__/ _ \ `_ \ / _` | `_ ` _ \ / _ \ \ \ / / __|
    | | |  __/ | | | (_| | | | | | |  __/  \ V /\__ \
    |_|  \___|_| |_|\__,_|_| |_| |_|\___|   \_/ |___/

    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Middle Observation(14821 ) of Last dataset = WORK.VS - Total Obs 29643                                                 */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  -- CHARACTER --                                                                                                       */
    /* VS_STUDYID                       C12      CDISCPILOT01        Study Identifier                                         */
    /* VS_DOMAIN                        C2       VS                  Domain Abbreviation                                      */
    /* VS_USUBJID                       C11      01-708-1428         Unique Subject Identifier                                */
    /* VS_VSTESTCD                      C6       PULSE               Vital Signs Test Short Name                              */
    /* VS_VSTEST                        C24      Pulse Rate          Vital Signs Test Name                                    */
    /* VS_VSPOS                         C8       STANDING            Vital Signs Position of Subject                          */
    /* VS_VSORRES                       C5       100                 Result or Finding in Original Units                      */
    /* VS_VSORRESU                      C9       beats/min           Original Units                                           */
    /* VS_VSSTRESC                      C6       100                 Character Result/Finding in Std Format                   */
    /* VS_VSSTRESU                      C9       beats/min           Standard Units                                           */
    /* VS_VSSTAT                        C8                           Completion Status                                        */
    /* VS_VSLOC                         C11                          Location of Vital Signs Measurement                      */
    /* VS_VSBLFL                        C1                           Baseline Flag                                            */
    /* VS_VISIT                         C19      SCREENING 1         Visit Name                                               */
    /* VS_EPOCH                         C9       SCREENING           Epoch                                                    */
    /* VS_VSDTC                         C10      2013-11-02          Date/Time of Measurements                                */
    /* VS_VSTPT                         C30      AFTER STANDING F    Planned Time Point Name                                  */
    /* VS_VSELTM                        C4       PT3M                Planned Elapsed Time from Time Point Ref                 */
    /* VS_VSTPTREF                      C16      PATIENT STANDING    Time Point Reference                                     */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  -- NUMERIC --                                                                                                         */
    /* VS_VSSEQ                         N8       31                  Sequence Number                                          */
    /* VS_VSSTRESN                      N8       100                 Numeric Result/Finding in Standard Units                 */
    /* VS_VISITNUM                      N8       1                   Visit Number                                             */
    /* VS_VISITDY                       N8       -7                  Planned Study Day of Visit                               */
    /* VS_VSDY                          N8       -7                  Study Day of Vital Signs                                 */
    /* VS_VSTPTNUM                      N8       817                 Planned Time Point Number                                */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /* _       _                   _     _       _
      (_) ___ (_)_ __     __ _  __| |___| |   __| |_ __ ___   __   _____
      | |/ _ \| | `_ \   / _` |/ _` / __| |  / _` | `_ ` _ \  \ \ / / __|
      | | (_) | | | | | | (_| | (_| \__ \ | | (_| | | | | | |  \ V /\__ \
     _/ |\___/|_|_| |_|  \__,_|\__,_|___/_|  \__,_|_| |_| |_|   \_/ |___/
    |__/
    */

    %utlopts;

    proc sql;

      create
        table smp_010adsl_dm_vs as
      select
        sl.*
       ,dm.*
       ,vs.*
      from
        adsl as sl
         left join dm as dm                        on sl.sl_usubjid = dm.dm_usubjid
         left join vsUnq as vs %dosubl('
           proc sort data=vs out=vsUnq nodupkey;
             where vs_vsblfl="Y";
             by vs_usubjid;
           run;quit;
           ')                                      on sl.sl_usubjid = vs.vs_usubjid

    ;quit;
    /*----- Number of unique levels=254 for sl_usubjid from smp_010adsl_dm_vs (obs=254) 06SEP2022:11:56:05 ----*/

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Middle Observation(127 ) of Last dataset = WORK.SMP_010ADSL_DM_VS - Total Obs 254                                      */
    /*                                                                                                                        */
    /*  -- CHARACTER --                                                                                                       */
    /* SL_STUDYID                       C12      CDISCPILOT01        Study Identifier                                         */
    /* SL_USUBJID                       C11      01-708-1353         Unique Subject Identifier                                */
    /* SL_SUBJID                        C4       1353                Subject Identifier for the Study                         */
    /* SL_SITEID                        C3       708                 Study Site Identifier                                    */
    /* SL_SITEGR1                       C3       708                 Pooled Site Group 1                                      */
    /* SL_ARM                           C20      Xanomeline Low D    Description of Planned Arm                               */
    /* SL_TRT01P                        C20      Xanomeline Low D    Planned Treatment for Period 01                          */
    /* SL_TRT01A                        C20      Xanomeline Low D    Actual Treatment for Period 01                           */
    /* SL_AGEGR1                        C5       >80                 Pooled Age Group 1                                       */
    /* SL_AGEU                          C5       YEARS               Age Units                                                */
    /* SL_RACE                          C32      BLACK OR AFRICAN    Race                                                     */
    /* SL_SEX                           C1       F                   Sex                                                      */
    /* SL_ETHNIC                        C22      NOT HISPANIC OR     Ethnicity                                                */
    /* SL_SAFFL                         C1       Y                   Safety Population Flag                                   */
    /* SL_ITTFL                         C1       Y                   Intent-To-Treat Population Flag                          */
    /* SL_EFFFL                         C1       Y                   Efficacy Population Flag                                 */
    /* SL_COMP8FL                       C1       Y                   Completers of Week 8 Population Flag                     */
    /* SL_COMP16FL                      C1       N                   Completers of Week 16 Population Flag                    */
    /* SL_COMP24FL                      C1       N                   Completers of Week 24 Population Flag                    */
    /* SL_DISCONFL                      C1       Y                   Did the Subject Discontinue the Study?                   */
    /* SL_DSRAEFL                       C1       Y                   Discontinued due to AE?                                  */
    /* SL_DTHFL                         C1                           Subject Died?                                            */
    /* SL_BMIBLGR1                      C6       <25                 Pooled Baseline BMI Group 1                              */
    /* SL_DURDSGR1                      C4       >=12                Pooled Disease Duration Group 1                          */
    /* SL_RFSTDTC                       C20      2013-07-04          Subject Reference Start Date/Time                        */
    /* SL_RFENDTC                       C20      2013-09-10          Subject Reference End Date/Time                          */
    /* SL_DCDECOD                       C27      ADVERSE EVENT       Standardized Disposition Term                            */
    /* SL_EOSSTT                        C12      DISCONTINUED        End of Study Status                                      */
    /* SL_DCSREAS                       C18      Adverse Event       Reason for Discontinuation from Study                    */
    /*                                                                                                                        */
    /* DM_STUDYID                       C12      CDISCPILOT01        Study Identifier                                         */
    /* DM_DOMAIN                        C2       DM                  Domain Abbreviation                                      */
    /* DM_USUBJID                       C11      01-708-1353         Unique Subject Identifier                                */
    /* DM_SUBJID                        C4       1353                Subject Identifier for the Study                         */
    /* DM_RFSTDTC                       C10      2013-07-04          Subject Reference Start Date/Time                        */
    /* DM_RFENDTC                       C10      2013-09-10          Subject Reference End Date/Time                          */
    /* DM_RFXSTDTC                      C10      2013-07-04          Date/Time of First Study Treatment                       */
    /* DM_RFXENDTC                      C10      2013-08-28          Date/Time of Last Study Treatment                        */
    /* DM_RFICDTC                       C1                           Date/Time of Informed Consent                            */
    /* DM_RFPENDTC                      C16      2013-09-10T13:15    Date/Time of End of Participation                        */
    /* DM_DTHDTC                        C10                          Date/Time of Death                                       */
    /* DM_DTHFL                         C1                           Subject Death Flag                                       */
    /* DM_SITEID                        C3       708                 Study Site Identifier                                    */
    /* DM_AGEU                          C5       YEARS               Age Units                                                */
    /* DM_SEX                           C1       F                   Sex                                                      */
    /* DM_RACE                          C32      BLACK OR AFRICAN    Race                                                     */
    /* DM_ETHNIC                        C22      NOT HISPANIC OR     Ethnicity                                                */
    /* DM_ARMCD                         C8       Xan_Lo              Planned Arm Code                                         */
    /* DM_ARM                           C20      Xanomeline Low D    Description of Planned Arm                               */
    /* DM_ACTARMCD                      C8       Xan_Lo              Actual Arm Code                                          */
    /* DM_ACTARM                        C20      Xanomeline Low D    Description of Actual Arm                                */
    /* DM_COUNTRY                       C3       USA                 Country                                                  */
    /* DM_DMDTC                         C10      2013-06-17          Date/Time of Collection                                  */
    /*                                                                                                                        */
    /* VS_STUDYID                       C12      CDISCPILOT01        Study Identifier                                         */
    /* VS_DOMAIN                        C2       VS                  Domain Abbreviation                                      */
    /* VS_USUBJID                       C11      01-708-1353         Unique Subject Identifier                                */
    /* VS_VSTESTCD                      C6       DIABP               Vital Signs Test Short Name                              */
    /* VS_VSTEST                        C24      Diastolic Blood     Vital Signs Test Name                                    */
    /* VS_VSPOS                         C8       SUPINE              Vital Signs Position of Subject                          */
    /* VS_VSORRES                       C5       60                  Result or Finding in Original Units                      */
    /* VS_VSORRESU                      C9       mmHg                Original Units                                           */
    /* VS_VSSTRESC                      C6       60                  Character Result/Finding in Std Format                   */
    /* VS_VSSTRESU                      C9       mmHg                Standard Units                                           */
    /* VS_VSSTAT                        C8                           Completion Status                                        */
    /* VS_VSLOC                         C11                          Location of Vital Signs Measurement                      */
    /* VS_VSBLFL                        C1       Y                   Baseline Flag                                            */
    /* VS_VISIT                         C19      BASELINE            Visit Name                                               */
    /* VS_EPOCH                         C9       TREATMENT           Epoch                                                    */
    /* VS_VSDTC                         C10      2013-07-04          Date/Time of Measurements                                */
    /* VS_VSTPT                         C30      AFTER LYING DOWN    Planned Time Point Name                                  */
    /* VS_VSELTM                        C4       PT5M                Planned Elapsed Time from Time Point Ref                 */
    /* VS_VSTPTREF                      C16      PATIENT SUPINE      Time Point Reference                                     */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  -- NUMERIC --                                                                                                         */
    /* SL_TRT01PN                       N8       54                  Planned Treatment for Period 01 (N)                      */
    /* SL_TRT01AN                       N8       54                  Actual Treatment for Period 01 (N)                       */
    /* SL_TRTSDT                        N8       19543               Date of First Exposure to Treatment                      */
    /* SL_TRTEDT                        N8       19598               Date of Last Exposure to Treatment                       */
    /* SL_TRTDURD                       N8       56                  Total Treatment Duration (Days)                          */
    /* SL_AVGDD                         N8       54                  Avg Daily Dose (as planned)                              */
    /* SL_CUMDOSE                       N8       3024                Cumulative Dose (as planned)                             */
    /* SL_AGE                           N8       87                  Age                                                      */
    /* SL_AGEGR1N                       N8       3                   Pooled Age Group 1 (N)                                   */
    /* SL_RACEN                         N8       2                   Race (N)                                                 */
    /* SL_BMIBL                         N8       20.3                Baseline BMI (kg/m^2)                                    */
    /* SL_HEIGHTBL                      N8       157.5               Baseline Height (cm)                                     */
    /* SL_WEIGHTBL                      N8       50.4                Baseline Weight (kg)                                     */
    /* SL_EDUCLVL                       N8       16                  Years of Education                                       */
    /* SL_DISONSDT                      N8       18480               Date of Onset of Disease                                 */
    /* SL_DURDIS                        N8       34.4                Duration of Disease (Months)                             */
    /* SL_VISIT1DT                      N8       19526               Date of Visit 1                                          */
    /* SL_VISNUMEN                      N8       8                   End of Trt Visit (Vis 12 or Early Term.)                 */
    /* SL_RFENDT                        N8       19611               Date of Discontinuation/Completion                       */
    /* SL_MMSETOT                       N8       16                  MMSE Total                                               */
    /*                                                                                                                        */
    /* DM_AGE                           N8       87                  Age                                                      */
    /* DM_DMDY                          N8       -17                 Study Day of Collection                                  */
    /*                                                                                                                        */
    /* VS_VSSEQ                         N8       7                   Sequence Number                                          */
    /* VS_VSSTRESN                      N8       60                  Numeric Result/Finding in Standard Units                 */
    /* VS_VISITNUM                      N8       3                   Visit Number                                             */
    /* VS_VISITDY                       N8       1                   Planned Study Day of Visit                               */
    /* VS_VSDY                          N8       1                   Study Day of Vital Signs                                 */
    /* VS_VSTPTNUM                      N8       815                 Planned Time Point Number                                */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*
     _ __ ___   __ _ _ __    _     _ __ ___   __ _  ___ _ __ ___
    | `_ ` _ \ / _` | `_ \ _| |_  | `_ ` _ \ / _` |/ __| `__/ _ \
    | | | | | | (_| | |_) |_   _| | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_| .__/  |_|   |_| |_| |_|\__,_|\___|_|  \___/
                    |_|
    */

    %utlopts;

    /* for 3 min test
    data smp_010adsl_dm_vs_tst ;
      set smp_010adsl_dm_vs(obs=10 keep=sl_:);
    run;quit ;
    */

    proc printto log="d:/smp/vdo/smp_010adsl_dm_vs.log" new;
    run;quit;

    %inc "d:/smp/sas/oto_voodoo.sas";

    * this can take 20+ minutes - for all these options;

    %utlvdoc
        (
        libname        = work                /* libname of input dataset                                         */
        ,data          = smp_010adsl_dm_vs   /* name of input dataset                                            */
        ,key           = sl_usubjid          /* 0 or variable                                                    */
        ,ExtrmVal      = 30           /* display top and bottom 10 frequencies                                   */
        ,UniPlot       = 1            /* 0 or univariate plots                                                   */
        ,UniVar        = 1            /* 0 or univariate analysis                                                */
        ,misspat       = 1            /* 0 or 1 missing patterns                                                 */
        ,chart         = 1            /* 0 or 1 line printer chart                                               */
        ,taball        = SL_ARM SL_SITEID   /* variable with all other variables 0                               */
        ,tabone        = SL_ARM SL_AGEU SL_RACE SL_SEX SL_ETHNIC   /* 0 or  variable vs all other variables      */
        ,mispop        = 1            /* 0 or 1  missing vs populated                                            */
        ,mispoptbl     = 1            /* 0 or 1  missing vs populated                                            */
        ,dupcol        = 1            /* 0 or 1  columns duplicated                                              */
        ,unqtwo        = 0            /* AREACODES DST STATECODE STATENAME ZIP_CLASS STATE Y COUNTY all pairs0   */
        ,vdocor        = 1            /* 0 or 1  correlation of numeric variables                                */
        ,oneone        = 1            /* 0 or 1  one to one - one to many - many to many                         */
        ,cramer        = 1            /* 0 or 1  association of character variables                              */
        ,optlength     = 1            /* 0 optimum length for character and numeric variables                    */
        ,maxmin        = 1            /* 0 or max min for every varuiable                                        */
        ,unichr        = 0            /* 0 univariate analysis of character variiables                           */
        ,outlier       = 1            /* 0 robust regression determination of outliers                           */
        ,printto       = d:\smp\vdo\&data..txt        /* file or output if output window                         */
        ,rsquare       = 0            /* sl_bmi with all dm_height dm_weight                                     */
        ,Cleanup       = 0            /* 0 or 1 delete intermediate datasets                                     */
        );

    proc printto;
    run;quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

/*                                    _                    _
 _ __ ___   __ _  ___ _ __ ___     __| | ___  ___ ___   __| | ___
| `_ ` _ \ / _` |/ __| `__/ _ \   / _` |/ _ \/ __/ _ \ / _` |/ _ \
| | | | | | (_| | (__| | | (_) | | (_| |  __/ (_| (_) | (_| |  __/
|_| |_| |_|\__,_|\___|_|  \___/   \__,_|\___|\___\___/ \__,_|\___|

*/
%macro utl_b64decode(inp,out);

   /*
   %let inp=d:/sd1/carsEnc.b64;
   %let out=d:/sd1/carsDec.zip;
   */

data _null_;
  length b64 $ 76 byte $ 1;
  infile "&inp" lrecl= 76 truncover length=b64length;
  input @1 b64 $base64X76.;
  if _N_=1 then putlog "NOTE: Detected Base64 Line Length of " b64length;
  file "&out" recfm=F lrecl= 1;
  do i=1 to (b64length/4)*3;
    byte=byte(rank(substr(b64,i, 1)));
    put byte $char1.;
  end;
run;quit;

%mend utl_b64decode;
