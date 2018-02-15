# utl_enumerate_all_zero_sums_that_are_at_most_2_quarters_apart_np_complete
Enumerate all zero sums that are at most 2 quarters apart np complete. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    Enumerate all zero sums that are at most 2 quarters apart np complete

    see github
    https://goo.gl/QMBkPL
    https://github.com/rogerjdeangelis/utl_enumerate_all_zero_sums_that_are_at_most_2_quarters_apart_np_complete

    https://goo.gl/LHWQiQ
    https://stackoverflow.com/questions/48767036/find-the-nearest-match-that-could-add-up-to-zero-in-sas

    ALGORITHM

     Enumerate all zero sums where
     time between observations is 2 quaters or less

       -2 -1 0 1 +2

    INPUT
    =====

     WORK.HAVE total obs=20

        FIRM    YRQTR     VAL

         AA     20001    -100   Year 2000 4 quaters
         AA     20002      50
         AA     20003      50   Nite 2000Q1, Q2 and Q3 add to zero
         AA     20004       0

         AA     20011       0   Year 2001 4 quaters
         AA     20012     -50
         AA     20013     100
         AA     20014     -50

         AA     20021       0   Year 2001 4 quaters
         AA     20022      50
         AA     20023    -150
         AA     20024     100

         BB     20001       0
         BB     20002       0
         BB     20003       0
         BB     20004      50
         BB     20011     150
         BB     20012       0
         BB     20013    -100
         BB     20014    -100


     Example output

     WORK.HAVAPN total obs=130

       Here we have combinatons of size two

     NORMALIZED

        FIRM    COMBINATIONS   SET    YRQUARTER   VALUES

         AA           2          1      20002        50    +/- 2 Quarters from 2000 Q4
         AA           2          1      20012       -50    50-50=0

         AA           2          2      20003        50
         AA           2          2      20012       -50


     DENORMALIZED

     WORK.WANT total obs=40

        FIRM    COMBINATIONS    SET   _20002    _20012    _20003

         AA           2           1     50        -50        .
         AA           2           2      .        -50       50


    PROCESS  (messy code but it works. please clean it up)
    =======================================================

      proc datasets lib=work;
        delete havGrp havApn;
      run;quit;

      %symdel yr vals cnt firm k / nowarn;

      data _null_;
           length vals $200;
           retain cnt 0 vals;
           set have;
           by firm;
           cnt=cnt+1;
           vals=catx(" ",vals,put(val,best.));
           if first.firm then call symputx("yr",substr(put(yrQtr,5.),1,4));
           if last.firm then do;
              call symputx("vals",vals);
              call symputx("cnt",cnt);
              call symputx("firm",firm);
              cnt=0;
              vals="";
              do k=2 to 5;
                 call symputx('k',k);
                 rc=dosubl('
                      data havgrp(rename=(k=combinations yr1=yrQuarter cmb=values));
                       retain yr &yr firm "&firm" set 0 q 0;
                       array x[&cnt] (&vals);
                       array c[&k] ;
                       array i[&k];
                       n=dim(x);
                       k=dim(i);
                       i[1]=0;
                       ncomb=comb(n,k);
                       do j=1 to ncomb+1;
                          rc=lexcombi(n, k, of i[*]);
                          do h=1 to k;
                             c[h]=x[i[h]];
                          end;
                          z= max(of i[*]) - min(of i[*]);
                          if z<=4 and sum(of c[*])=0 then do;
                             set=set+1;
                             do idx=1 to dim(i);
                                cmb=c[idx];
                                cnt=i[idx];
                                qtr=mod(cnt-1,4) +1;
                                if mod(cnt,4)=0 then yr1=yr+cnt/4-.25;
                                else yr1=yr+(cnt/4) - .25;
                                yr1=10*int(yr1) + (yr1-int(yr1))*4 + 1;
                                keep k set firm yr1 cmb;
                                output;
                             end;
                          end;
                       end;
                   run;quit;
                   proc append data=havGrp base=havApn;
                   run;quit;
                 ');
              end;
           end;
      run;quit;

      proc transpose data=havApn out=want(drop=_name_);;
      by firm combinations set;
      id yrquarter;
      var values;
      run;quit;

    OUTPUT   (Full output below)
    ===============================

    All Obs(130) from dataset havApn

    Obs    FIRM    SET    COMBINATIONS    VALUES    YRQUARTER

      1     AA       1          2            50       20002
      2     AA       1          2           -50       20012
      3     AA       2          2            50       20003
      4     AA       2          2           -50       20012
      5     AA       3          2             0       20004
      6     AA       3          2             0       20011
      7     AA       4          2             0       20011
      8     AA       4          2             0       20021
      9     AA       5          2           -50       20012
     10     AA       5          2            50       20022
     11     AA       6          2           -50       20014
     12     AA       6          2            50       20022
     13     AA       1          3          -100       20001
     14     AA       1          3            50       20002
     15     AA       1          3            50       20003
     16     AA       2          3            50       20002
     17     AA       2          3             0       20004
     18     AA       2          3           -50       20012
    .......................................................
    126     BB       2          5            50       20004    50+150-100-100-0 = 0
    127     BB       2          5           150       20011
    128     BB       2          5             0       20012
    129     BB       2          5          -100       20013
    130     BB       2          5          -100       20014


    DENORMALIZED
    ==========

    All Obs(40) from dataset want

       FIRM    COMBINATIONS    SET    _20002    _20012    _20003    _20004    _20011    _20021    _20022    _20014    _20001    _20013    _20023    _20024

        AA           2           1      50        -50        .         .          .        .         .          .         .         .         .         .
        AA           2           2       .        -50       50         .          .        .         .          .         .         .         .         .
        AA           2           3       .          .        .         0          0        .         .          .         .         .         .         .
        BB           4           1       .          .        .        50        150        .         .       -100         .      -100         .         .
    ............................
        BB           5           1       .          0        .        50        150        .         .       -100         .      -100         .         .
        BB           5           2       .          0        .        50        150        .         .       -100         .      -100         .         .


    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    data have;
      input firm$ yrQtr val;
    cards4;
    AA 20001 -100
    AA 20002 50
    AA 20003 50
    AA 20004 0
    AA 20011 0
    AA 20012 -50
    AA 20013 100
    AA 20014 -50
    AA 20021 0
    AA 20022 50
    AA 20023 -150
    AA 20024 100
    BB 20001 0
    BB 20002 0
    BB 20003 0
    BB 20004 50
    BB 20011 150
    BB 20012 0
    BB 20013 -100
    BB 20014 -100
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    proc datasets lib=work;
      delete havGrp havApn;
    run;quit;

    %symdel yr vals cnt firm k / nowarn;

    data _null_;
         length vals $200;
         retain cnt 0 vals;
         set have;
         by firm;
         cnt=cnt+1;
         vals=catx(" ",vals,put(val,best.));
         if first.firm then call symputx("yr",substr(put(yrQtr,5.),1,4));
         if last.firm then do;
            call symputx("vals",vals);
            call symputx("cnt",cnt);
            call symputx("firm",firm);
            cnt=0;
            vals="";
            do k=2 to 5;
               call symputx('k',k);
               rc=dosubl('
                    data havgrp(rename=(k=combinations yr1=yrQuarter cmb=values));
                     retain yr &yr firm "&firm" set 0 q 0;
                     array x[&cnt] (&vals);
                     array c[&k] ;
                     array i[&k];
                     n=dim(x);
                     k=dim(i);
                     i[1]=0;
                     ncomb=comb(n,k);
                     do j=1 to ncomb+1;
                        rc=lexcombi(n, k, of i[*]);
                        do h=1 to k;
                           c[h]=x[i[h]];
                        end;
                        z= max(of i[*]) - min(of i[*]);
                        if z<=4 and sum(of c[*])=0 then do;
                           set=set+1;
                           do idx=1 to dim(i);
                              cmb=c[idx];
                              cnt=i[idx];
                              qtr=mod(cnt-1,4) +1;
                              if mod(cnt,4)=0 then yr1=yr+cnt/4-.25;
                              else yr1=yr+(cnt/4) - .25;
                              yr1=10*int(yr1) + (yr1-int(yr1))*4 + 1;
                              keep k set firm yr1 cmb;
                              output;
                           end;
                        end;
                     end;
                 run;quit;
                 proc append data=havGrp base=havApn;
                 run;quit;
               ');
            end;
         end;
    run;quit;

    proc transpose data=havApn out=want (drop=_name_);;
    by firm combinations set;
    id yrquarter;
    var values;
    run;quit;

    NORMALIZED
    ==========

    All Obs(130) from dataset havApn

    Obs    FIRM    SET    COMBINATIONS    VALUES    YRQUARTER

      1     AA       1          2            50       20002
      2     AA       1          2           -50       20012
      3     AA       2          2            50       20003
      4     AA       2          2           -50       20012
      5     AA       3          2             0       20004
      6     AA       3          2             0       20011
      7     AA       4          2             0       20011
      8     AA       4          2             0       20021
      9     AA       5          2           -50       20012
     10     AA       5          2            50       20022
     11     AA       6          2           -50       20014
     12     AA       6          2            50       20022
     13     AA       1          3          -100       20001
     14     AA       1          3            50       20002
     15     AA       1          3            50       20003
     16     AA       2          3            50       20002
     17     AA       2          3             0       20004
     18     AA       2          3           -50       20012
     19     AA       3          3            50       20002
     20     AA       3          3             0       20011
     21     AA       3          3           -50       20012
     22     AA       4          3            50       20003
     23     AA       4          3             0       20004
     24     AA       4          3           -50       20012
     25     AA       5          3            50       20003
     26     AA       5          3             0       20011
     27     AA       5          3           -50       20012
     28     AA       6          3           -50       20012
     29     AA       6          3           100       20013
     30     AA       6          3           -50       20014
     31     AA       7          3           -50       20012
     32     AA       7          3             0       20021
     33     AA       7          3            50       20022
     34     AA       8          3           100       20013
     35     AA       8          3            50       20022
     36     AA       8          3          -150       20023
     37     AA       9          3           -50       20014
     38     AA       9          3             0       20021
     39     AA       9          3            50       20022
     40     AA      10          3            50       20022
     41     AA      10          3          -150       20023
     42     AA      10          3           100       20024
     43     AA      11          3            50       20022
     44     AA      11          3          -150       20023
     45     AA      11          3           100       20024
     46     AA       1          4          -100       20001
     47     AA       1          4            50       20002
     48     AA       1          4            50       20003
     49     AA       1          4             0       20004
     50     AA       2          4          -100       20001
     51     AA       2          4            50       20002
     52     AA       2          4            50       20003
     53     AA       2          4             0       20011
     54     AA       3          4            50       20002
     55     AA       3          4             0       20004
     56     AA       3          4             0       20011
     57     AA       3          4           -50       20012
     58     AA       4          4            50       20003
     59     AA       4          4             0       20004
     60     AA       4          4             0       20011
     61     AA       4          4           -50       20012
     62     AA       5          4             0       20004
     63     AA       5          4           -50       20012
     64     AA       5          4           100       20013
     65     AA       5          4           -50       20014
     66     AA       6          4             0       20011
     67     AA       6          4           -50       20012
     68     AA       6          4           100       20013
     69     AA       6          4           -50       20014
     70     AA       7          4           -50       20012
     71     AA       7          4           100       20013
     72     AA       7          4           -50       20014
     73     AA       7          4             0       20021
     74     AA       8          4           100       20013
     75     AA       8          4             0       20021
     76     AA       8          4            50       20022
     77     AA       8          4          -150       20023
     78     AA       9          4             0       20021
     79     AA       9          4            50       20022
     80     AA       9          4          -150       20023
     81     AA       9          4           100       20024
     82     AA      10          4             0       20021
     83     AA      10          4            50       20022
     84     AA      10          4          -150       20023
     85     AA      10          4           100       20024
     86     AA       1          5          -100       20001
     87     AA       1          5            50       20002
     88     AA       1          5            50       20003
     89     AA       1          5             0       20004
     90     AA       1          5             0       20011
     91     AA       2          5             0       20004
     92     AA       2          5             0       20011
     93     AA       2          5           -50       20012
     94     AA       2          5           100       20013
     95     AA       2          5           -50       20014
     96     AA       3          5             0       20011
     97     AA       3          5           -50       20012
     98     AA       3          5           100       20013
     99     AA       3          5           -50       20014
    100     AA       3          5             0       20021
    101     BB       1          2             0       20001
    102     BB       1          2             0       20002
    103     BB       2          2             0       20001
    104     BB       2          2             0       20003
    105     BB       3          2             0       20002
    106     BB       3          2             0       20003
    107     BB       4          2             0       20002
    108     BB       4          2             0       20012
    109     BB       5          2             0       20003
    110     BB       5          2             0       20012
    111     BB       1          3             0       20001
    112     BB       1          3             0       20002
    113     BB       1          3             0       20003
    114     BB       2          3             0       20002
    115     BB       2          3             0       20003
    116     BB       2          3             0       20012
    117     BB       1          4            50       20004
    118     BB       1          4           150       20011
    119     BB       1          4          -100       20013
    120     BB       1          4          -100       20014
    121     BB       1          5            50       20004
    122     BB       1          5           150       20011
    123     BB       1          5             0       20012
    124     BB       1          5          -100       20013
    125     BB       1          5          -100       20014
    126     BB       2          5            50       20004
    127     BB       2          5           150       20011
    128     BB       2          5             0       20012
    129     BB       2          5          -100       20013
    130     BB       2          5          -100       20014


    DENORMALIZED
    ==========

    All Obs(40) from dataset want

       FIRM    COMBINATIONS    SET    _20002    _20012    _20003    _20004    _20011    _20021    _20022    _20014    _20001    _20013    _20023    _20024

        AA           2           1      50        -50        .         .          .        .         .          .         .         .         .         .
        AA           2           2       .        -50       50         .          .        .         .          .         .         .         .         .
        AA           2           3       .          .        .         0          0        .         .          .         .         .         .         .
        AA           2           4       .          .        .         .          0        0         .          .         .         .         .         .
        AA           2           5       .        -50        .         .          .        .        50          .         .         .         .         .
        AA           2           6       .          .        .         .          .        .        50        -50         .         .         .         .
        AA           3           1      50          .       50         .          .        .         .          .      -100         .         .         .
        AA           3           2      50        -50        .         0          .        .         .          .         .         .         .         .
        AA           3           3      50        -50        .         .          0        .         .          .         .         .         .         .
        AA           3           4       .        -50       50         0          .        .         .          .         .         .         .         .
        AA           3           5       .        -50       50         .          0        .         .          .         .         .         .         .
        AA           3           6       .        -50        .         .          .        .         .        -50         .       100         .         .
        AA           3           7       .        -50        .         .          .        0        50          .         .         .         .         .
        AA           3           8       .          .        .         .          .        .        50          .         .       100      -150         .
        AA           3           9       .          .        .         .          .        0        50        -50         .         .         .         .
        AA           3          10       .          .        .         .          .        .        50          .         .         .      -150       100
        AA           3          11       .          .        .         .          .        .        50          .         .         .      -150       100
        AA           4           1      50          .       50         0          .        .         .          .      -100         .         .         .
        AA           4           2      50          .       50         .          0        .         .          .      -100         .         .         .
        AA           4           3      50        -50        .         0          0        .         .          .         .         .         .         .
        AA           4           4       .        -50       50         0          0        .         .          .         .         .         .         .
        AA           4           5       .        -50        .         0          .        .         .        -50         .       100         .         .
        AA           4           6       .        -50        .         .          0        .         .        -50         .       100         .         .
        AA           4           7       .        -50        .         .          .        0         .        -50         .       100         .         .
        AA           4           8       .          .        .         .          .        0        50          .         .       100      -150         .
        AA           4           9       .          .        .         .          .        0        50          .         .         .      -150       100
        AA           4          10       .          .        .         .          .        0        50          .         .         .      -150       100
        AA           5           1      50          .       50         0          0        .         .          .      -100         .         .         .
        AA           5           2       .        -50        .         0          0        .         .        -50         .       100         .         .
        AA           5           3       .        -50        .         .          0        0         .        -50         .       100         .         .
        BB           2           1       0          .        .         .          .        .         .          .         0         .         .         .
        BB           2           2       .          .        0         .          .        .         .          .         0         .         .         .
        BB           2           3       0          .        0         .          .        .         .          .         .         .         .         .
        BB           2           4       0          0        .         .          .        .         .          .         .         .         .         .
        BB           2           5       .          0        0         .          .        .         .          .         .         .         .         .
        BB           3           1       0          .        0         .          .        .         .          .         0         .         .         .
        BB           3           2       0          0        0         .          .        .         .          .         .         .         .         .
        BB           4           1       .          .        .        50        150        .         .       -100         .      -100         .         .
        BB           5           1       .          0        .        50        150        .         .       -100         .      -100         .         .
        BB           5           2       .          0        .        50        150        .         .       -100         .      -100         .         .









