# utl-setting-flags-within-nested-groups-on-multiple-conditions
Setting flags within nested groups on multiple conditions 
    Setting flags within nested groups on multiple conditions                                                            
                                                                                                                         
    Problem                                                                                                              
                                                                                                                         
       within the same ID/Treatment/Linenumber                                                                           
           if gap > 90 days  then GT90_within_line =1;                                                                   
                                                                                                                         
       within the same ID/Treatment                                                                                      
           if gap < 90 days, then LT90_btw_line_within_trt = 1                                                           
                                                                                                                         
    github                                                                                                               
    https://tinyurl.com/y2hcfc69                                                                                         
    https://github.com/rogerjdeangelis/utl-setting-flags-within-nested-groups-on-multiple-conditions                     
                                                                                                                         
    sas forum                                                                                                            
    https://tinyurl.com/y69q4lcr                                                                                         
    https://communities.sas.com/t5/SAS-Programming/Remove-group-by-conditions/m-p/564056                                 
                                                                                                                         
    *_                   _                                                                                               
    (_)_ __  _ __  _   _| |_                                                                                             
    | | '_ \| '_ \| | | | __|                                                                                            
    | | | | | |_) | |_| | |_                                                                                             
    |_|_| |_| .__/ \__,_|\__|                                                                                            
            |_|                                                                                                          
    ;                                                                                                                    
                                                                                                                         
    data have;                                                                                                           
    input id $ treatment $ linenumber date yymmdd10. ;                                                                   
    format date yymmdd10.;                                                                                               
    cards;                                                                                                               
    001 A 1 2014-08-15                                                                                                   
    001 A 1 2015-06-20                                                                                                   
    001 A 2 2015-08-29                                                                                                   
    001 A 2 2018-02-28                                                                                                   
    001 A 2 2018-05-07                                                                                                   
    001 B 3 2018-08-01                                                                                                   
    001 B 4 2019-02-01                                                                                                   
    001 C 5 2019-03-03                                                                                                   
    ;                                                                                                                    
    run;                                                                                                                 
                                                                                                                         
                                                              | RULES                                                    
                                                              | =====                                                    
    WORK.HAVMRG total obs=8                                   |   GT90_          LT90_BTW_                               
                                                              |  WITHIN_       LINE_WITHIN_                              
      ID   TREATMENT  LINENUMBER     DATE    NXTDTE    DTEDIF |    LINE             TRT                                  
                                                              |  Within Lines   Within Treatments                        
                                                                                                                         
      001      A           1        19950     20259      309  |     1 >90            .                                   
      001      A           1        20259     20329       70  |     .                1 <90                               
                                                                                                                         
      001      A           2        20329     21243      914  |     1 >90            .                                   
      001      A           2        21243     21311       68  |     .                1 <90                               
      001      A           2        21311     21397       86  |     .                1 <90                               
                                                                                                                         
      001      B           3        21397     21581      184  |     1 >90            .                                   
      001      B           4        21581     21611       30  |     .                1 <90                               
      001      C           5        21611         .        .  |     .                1 <90  (no next date)               
                                                                                                                         
                                                                                                                         
    *            _               _                                                                                       
      ___  _   _| |_ _ __  _   _| |_                                                                                     
     / _ \| | | | __| '_ \| | | | __|                                                                                    
    | (_) | |_| | |_| |_) | |_| | |_                                                                                     
     \___/ \__,_|\__| .__/ \__,_|\__|                                                                                    
                    |_|                                                                                                  
    ;                                                                                                                    
                                                                                                                         
    WORK.WANT total obs=8                                                                                                
                                                                                                                         
                                                   GT90_BTW_       LT90_BTW_                                             
                                                 LINE_WITHIN_    LINE_WITHIN_                                            
      ID     TREATMENT    LINENUMBER     DATE         TRT             TRT                                                
                                                                                                                         
      001        A             1        19950          1               .                                                 
      001        A             1        20259          .               1                                                 
      001        A             2        20329          1               .                                                 
      001        A             2        21243          .               1                                                 
      001        A             2        21311          .               1                                                 
      001        B             3        21397          1               .                                                 
      001        B             4        21581          .               1                                                 
      001        C             5        21611          .               1                                                 
                                                                                                                         
    *                                                                                                                    
     _ __  _ __ ___   ___ ___  ___ ___                                                                                   
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|                                                                                  
    | |_) | | | (_) | (_|  __/\__ \__ \                                                                                  
    | .__/|_|  \___/ \___\___||___/___/                                                                                  
    |_|                                                                                                                  
    ;                                                                                                                    
                                                                                                                         
    proc datasets lib=work mt=view mt=data;                                                                              
      delete havMrg want;                                                                                                
    run;quit;                                                                                                            
                                                                                                                         
    data want;                                                                                                           
                                                                                                                         
      if _n_ = 0 then do; %let rc=%sysfunc(dosubl('                                                                      
                                                                                                                         
        data havMrg/view=havMrg;                                                                                         
          merge have have(firstobs=2 keep=date rename=date=nxtDte);                                                      
          dteDif=nxtDte - date;                                                                                          
        run;quit;                                                                                                        
                                                                                                                         
        '));                                                                                                             
      end;                                                                                                               
                                                                                                                         
      set havmrg;                                                                                                        
      by id  treatment linenumber;                                                                                       
                                                                                                                         
      if dteDif > 90 then                                                                                                
                gt90_btw_line_within_trt =1;                                                                             
                                                                                                                         
      if dteDif < 90 then                                                                                                
                lt90_btw_line_within_trt = 1 ;                                                                           
      output;                                                                                                            
                                                                                                                         
      if last.linenumber then gt90_btw_line_within_trt = . ;                                                             
      if last.treatment  then lt90_btw_line_within_trt = . ;                                                             
                                                                                                                         
      drop nxtDte dteDif;                                                                                                
                                                                                                                         
    run;quit;                                                                                                            
                                                                                                                         
                                                                                                                         
