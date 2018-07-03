# utl_use_table2_to_select_and_stack_elements_in_table1
Use table2 to select and stack elements in table1. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.


    Use table2 to select and stack elements in table1

      Same result for Base SAS and Base WPS

      TWO SOLUTIONS

         1. Datastep
         2. WPS RPROC R


    INPUT
    =====

     SD1.HAV1ST total obs=5

        Y1    Y2    Y3    Y4    Y5

        11    12    13    14    15
        21    22    23    24    25
        31    32    33    34    35
        41    42    43    44    45
        51    52    53    54    55


     SD1.HAV2ND  total obs=5

      IDX1    IDX2    IDX3    IDX4    IDX5

        1       1       0       0       1
        0       0       1       0       1
        0       1       0       0       1
        1       0       1       0       0
        1       1       1       1       1


    EXAMPLE OUTPUT
    --------------

    WORK.WANT total obs=14|  RULES
                          |
       VAR    VAL         |         IDX1 IDX2 IDX3 IDX4 IDX5
                          |
       Y1      11         |           1    1    0    0    1
       Y2      12         | keep ys  11   12             15
       Y5      15         |
                          |
       Y3      23         |           0    0    1    0     1
       Y5      25         |                    23         25


    PROCESS
    =======

     1. datastep

        data want;

          merge sd1.hav1st sd1.hav2nd;

          array ys y:;
          array idxs idx:;

          do over ys;
             if idxs =1 then do;
                var= vname(ys);
                val= ys;
                output;
             end;
          end;

          keep var val;

        run;quit;


     2. WPS RPROC R (working code)

        want<-gather(hav1st*hav2nd,key=var,value=val);
        want<-want[which(want$val > 0),];


    OUTPUT
    ======

     WORK.WANT total obs=14

        VAR    VAL

        Y1      11
        Y2      12
        Y5      15
        Y3      23
        Y5      25
        Y2      32
        Y5      35
        Y1      41
        Y3      43
        Y1      51
        Y2      52
        Y3      53
        Y4      54
        Y5      55

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    libname sd1 "d:/sd1";
    options validvarname=upcase;
    data sd1.hav1st;

      array ys[5]  y1-y5;
      do rows=1 to 5;
         rowc=put(rows,1.);
         do cols=1 to 5;
            ys[cols]=10*rows+cols;
         end;
         output;
      end;
      keep y:;

    run;quit;


    data sd1.hav2nd;
      call streaminit(1234);
      array idxs[5]  idx1-idx5;
      do rows=1 to 5;
         rowc=put(rows,1.);
         do cols=1 to 5;
            idxs[cols]=int(2*(rand('uniform')));
         end;
         output;
      end;
      keep id:;

    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    * SAS see process;


    * WPS;

    %utl_submit_wps64('
    libname sd1 "d:/sd1";
        data want;

          merge sd1.hav1st sd1.hav2nd;

          array ys y:;
          array idxs idx:;

          do over ys;
             if idxs =1 then do;
                var= vname(ys);
                val= ys;
                output;
             end;
          end;

          keep var val;

        run;quit;
        proc print;
        run;quit;
    ');


    %utl_submit_wps64('
    libname sd1 "d:/sd1";
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk  sas7bdat "%sysfunc(pathname(work))";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(haven);
    library(tidyverse);
    hav1st<-read_sas("d:/sd1/hav1st.sas7bdat");
    hav2nd<-read_sas("d:/sd1/hav2nd.sas7bdat");
    head(hav1st);
    head(hav2nd);
    hav1st*hav2nd;
    want<-gather(hav1st*hav2nd,key=var,value=val);
    want<-want[which(want$val > 0),];
    endsubmit;
    import r=want  data=wrk.wantwps;
    run;quit;
    ');

    proc print data=wantwps;
    run;quit;


