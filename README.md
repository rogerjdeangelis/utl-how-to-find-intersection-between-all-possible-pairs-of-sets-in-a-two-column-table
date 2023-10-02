# utl-how-to-find-intersection-between-all-possible-pairs-of-sets-in-a-two-column-table
How to find intersection between all possible pairs of sets in a two column table
    %let pgm=utl-how-to-find-intersection-between-all-possible-pairs-of-sets-in-a-two-column-table;

    How to find intersection between all possible pairs of sets in a two column table;

      SOLUTIONS

          1 wps sql
          2 wps r no sql
          3 wps r sql
            https://stackoverflow.com/users/12158757/thomasiscoding
          4 wps python sql

    'I would like to end up with a table that intersects all possible pairs of sets,
    and specifies the size of the smaller set in each pair.
    This will give rise to calculating an overlap coefficient for each pair of sets'

    github
    https://tinyurl.com/4763msb5
    https://github.com/rogerjdeangelis/utl-how-to-find-intersection-between-all-possible-pairs-of-sets-in-a-two-column-table

    stackoverflow
    https://tinyurl.com/454p32ue
    https://stackoverflow.com/questions/77215428/how-to-find-intersection-between-all-possible-pairs-of-sets-in-a-two-column-table

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=any;
    libname sd1 "d:/sd1";
    data sd1.have;
      input my_group$ cities$;
    cards4;
    foo london
    foo paris
    foo rome
    foo tokyo
    foo oslo
    bar paris
    bar nyc
    bar rome
    bar munich
    bar warsaw
    bar sf
    baz milano
    baz oslo
    baz sf
    baz paris
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                            |                                        |                                                  */
    /*       INPUT                |            PROCESS                     |                 OUTPUT                           */
    /*                            |                                        |                                                  */
    /*  MY_GROUP    CITIES        |    EXAMPLE (bar-foo combo output)      |      COMBO  INTERSECT    SIZE     COEFF          */
    /*                            |                                        |                                                  */
    /*    foo       london        |          INTERSECTIONS                 |                                                  */
    /*    foo       paris match   |                           (in both  )  |     bar-foo     2          5       0.4           */
    /*    foo       rome  match   |   bar        foo    paris (intersect)  |                                                  */
    /*    foo       tokyo         |   bar        foo    rome               |                                                  */
    /*    foo       oslo          |                                        |  FULL OUTPUT                                     */
    /*                            |   Two cities are in both bar and foo   |                                                  */
    /*    bar       paris match   |                                        |                     r_my_                  min   */
    /*    bar       sf            |   bar has 6 records                    |  Obs    my_group    group    intersect    Size   */
    /*    bar       nyc           |   foo has 5 records use this one       |                                                  */
    /*    bar       rome  match   |                                        |   1       bar        baz         2          4    */
    /*    bar       munich        |   So we have an intersection of 2      |   2       bar        foo         2          5    */
    /*    bar       warsaw        |   and min(bar count,foo count)= 5      |   3       baz        foo         2          4    */
    /*                            |                                        |                                                  */
    /*    baz       paris         |   coef=2/5                             |                                                  */
    /*    baz       sf            |                                        |                                                  */
    /*    baz       milano        |                                        |                                                  */
    /*    baz       oslo          |                                        |                                                  */
    /*                            |                                        |                                                  */
    /**************************************************************************************************************************/

    /*                                  _
    / | __      ___ __  ___   ___  __ _| |
    | | \ \ /\ / / `_ \/ __| / __|/ _` | |
    | |  \ V  V /| |_) \__ \ \__ \ (_| | |
    |_|   \_/\_/ | .__/|___/ |___/\__, |_|
                 |_|                 |_|
    */

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    options validvarname=any;
    proc sql;

      create
         table sd1.want as
      select
         l.my_group
        ,r.my_group as r_my_group
        ,count(*) as intersect
        ,( min((select count(my_group) as size from sd1.have where my_group=l.my_group),
            (select count(my_group) as size from sd1.have where my_group=r.my_group))) as minSize
        ,calculated intersect/calculated minSize as coef
      from
        sd1.have as l left join sd1.have as r
      on
             l.cities = r.cities
        and l.my_group ne r.my_group
      where
        l.my_group < r.my_group
      group
        by l.my_group, r.my_group

    ;quit;
    ');

    proc print data=sd1.want;
    run;quit;

     /**************************************************************************************************************************/
     /*                                                                                                                        */
     /*                    r_my_                  min                                                                          */
     /* Obs    my_group    group    intersect    Size    coef                                                                  */
     /*                                                                                                                        */
     /*  1       bar        baz         2          4      0.5                                                                  */
     /*  2       bar        foo         2          5      0.4                                                                  */
     /*  3       baz        foo         2          4      0.5                                                                  */
     /*                                                                                                                        */
     /**************************************************************************************************************************/

    /*___
    |___ \  __      ___ __  ___   _ __
      __) | \ \ /\ / / `_ \/ __| | `__|
     / __/   \ V  V /| |_) \__ \ | |
    |_____|   \_/\_/ | .__/|___/ |_|
                     |_|
    */


    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    options validvarname=any;
    libname sd1 "d:/sd1";

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    want<-do.call(
        rbind,
        combn(
            with(
                have,
                split(CITIES, MY_GROUP)
            ),
            2,
            \(x)
            transform(
                data.frame(
                    combo = paste0(names(x), collapse = "-"),
                    nrIntersect = sum(x[[1]] %in% x[[2]]),
                    szSmallSet = min(lengths(x))
                ),
                olCoeff = nrIntersect / szSmallSet
            ),
            simplify = FALSE
        )
    );
    want;
    endsubmit;
    import data=sd1.want r=want;
    run;quit;
    ');

    proc print data=sd1.want width=min;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*                                                                                                                        */
    /*  The WPS proc R                                                                                                        */
    /*                                                                                                                        */
    /*      combo nrIntersect szSmallSet olCoeff                                                                              */
    /*                                                                                                                        */
    /*  1 bar-baz           2          4     0.5                                                                              */
    /*  2 bar-foo           2          5     0.4                                                                              */
    /*  3 baz-foo           2          4     0.5                                                                              */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /* WPS                                                                                                                    */
    /*                                                                                                                        */
    /* Obs     COMBO     NRINTERSECT    SZSMALLSET    OLCOEFF                                                                 */
    /*                                                                                                                        */
    /*  1     bar-baz         2              4          0.5                                                                   */
    /*  2     bar-foo         2              5          0.4                                                                   */
    /*  3     baz-foo         2              4          0.5                                                                   */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____                                  _
    |___ /  __      ___ __  ___   ___  __ _| |
      |_ \  \ \ /\ / / `_ \/ __| / __|/ _` | |
     ___) |  \ V  V /| |_) \__ \ \__ \ (_| | |
    |____/    \_/\_/ | .__/|___/ |___/\__, |_|
                     |_|                 |_|
    */

    /*----                                                                   ----*/
    /*---- SQLlite3 does no support                                          ----*/
    /*---- calculated intersect/calculated minSize as coef                   ----*/
    /*---- However calculating using the statement below may be just as fast ----*/
    /*---- count(l.my_group)/(select min((select count(my_group) as size..    ---*/
    /*----                                                                   ----*/
    /*----                                                                   ----*/
    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    library(sqldf);
    want <- sqldf("
      select
         l.my_group
        ,r.my_group as r_my_group
        ,count(l.my_group) as xsect
        ,(select min((select count(my_group) as size from have where my_group=l.my_group),
            (select count(my_group) as size from have where my_group=r.my_group))) as minSize
      from
        have as l left join have as r
      on
             l.cities = r.cities
        and l.my_group <> r.my_group
      where
        l.my_group < r.my_group
      group
        by l.my_group, r.my_group
    ");
    want;
    endsubmit;
    import data=sd1.want r=want;
    run;quit;
    ');

    proc print data=sd1.want width=min;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS proc R                                                                                                         */
    /*                                                                                                                        */
    /*   MY_GROUP r_my_group  xsect minSize                                                                                   */
    /* 1      bar        baz      2       4                                                                                   */
    /* 2      bar        foo      2       5                                                                                   */
    /* 3      baz        foo      2       4                                                                                   */
    /*                                                                                                                        */
    /* WPS                                                                                                                    */
    /*                    R_MY_                                                                                               */
    /* Obs    MY_GROUP    GROUP    XSECT     MINSIZE                                                                          */
    /*                                                                                                                        */
    /*  1       bar        baz        2         4                                                                             */
    /*  2       bar        foo        2         5                                                                             */
    /*  3       baz        foo        2         4                                                                             */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _                                      _   _                             _
    | || |   __      ___ __  ___   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
    | || |_  \ \ /\ / / `_ \/ __| | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
    |__   _|  \ V  V /| |_) \__ \ | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
       |_|     \_/\_/ | .__/|___/ | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
                      |_|         |_|    |___/                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    %utl_submit_wps64x("
    options validvarname=any lrecl=32756;
    libname sd1 'd:/sd1';
    proc python;
    export data=sd1.have python=have;
    submit;
    print(have);
    from os import path;
    import pandas as pd;
    import numpy as np;
    from pandasql import sqldf;
    mysql = lambda q: sqldf(q, globals());
    from pandasql import PandaSQL;
    pdsql = PandaSQL(persist=True);
    sqlite3conn = next(pdsql.conn.gen).connection.connection;
    sqlite3conn.enable_load_extension(True);
    sqlite3conn.load_extension('c:/temp/libsqlitefunctions.dll');
    mysql = lambda q: sqldf(q, globals());
    want = pdsql('''
      select
         l.my_group
        ,r.my_group as r_my_group
        ,count(l.my_group) as xsect
        ,(select min((select count(my_group) as size from have where my_group=l.my_group),
            (select count(my_group) as size from have where my_group=r.my_group))) as minSize
      from
        have as l left join have as r
      on
             l.cities = r.cities
        and l.my_group <> r.my_group
      where
        l.my_group < r.my_group
      group
        by l.my_group, r.my_group
    ''');
    print(want);
    endsubmit;
    import data=sd1.want python=want;
    run;quit;
    "));

    proc print data=sd1.want width=min;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS proc R                                                                                                         */
    /*                                                                                                                        */
    /*   MY_GROUP r_my_group   xsect  minSize                                                                                 */
    /* 0      bar        baz       2        4                                                                                 */
    /* 1      bar        foo       2        5                                                                                 */
    /* 2      baz        foo       2        4                                                                                 */
    /*                                                                                                                        */
    /* WPS                                                                                                                    */
    /*                    R_MY_                                                                                               */
    /* Obs    MY_GROUP    GROUP    XSECT     MINSIZE                                                                          */
    /*                                                                                                                        */
    /*  1       bar        baz        2         4                                                                             */
    /*  2       bar        foo        2         5                                                                             */
    /*  3       baz        foo        2         4                                                                             */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
