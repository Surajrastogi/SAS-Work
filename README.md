# SAS-Work
The Basic and common procedure that are used in SAS

/*_all_ will help u sort thhe data by all the variabls */
proc sort data=A;
by descending _all_;
run;

/*Add the data by group by*/
data A;
Input ID $ salary;
cards;
1  2000
1  3000
1  4000
2  5000
2  6000
;run;

Data E;
set A;
by ID;
if first.id then New=0;
new+salary;
if last.id;
drop salary;
run;

proc sql;
create table b2 as select Id,sum(salary) as salary from A group by Id order by id desc;quit;

/*Second highest observation*/

data A;
Input ID salary;
cards;
1  2000
1  3000
1  4000
2  8000
2  6000
;run;

proc sql;
create table y as select * ,(select count(*) from (select distinct Id,Salary from A ) as b 
where b.id=a.id and b.salary LT A.salary) as n from A as a;
quit;



proc sort data=a;
by  ID salary;
run;
Data H;
set A;
by ID;
if first.ID then n=0;
n+1;
if n=2 then output;
run;


/*Nnd highest from 2 class variable*/

Data A;
Input ID $ Class $ Scr;
cards;
1 A 45
1 A 58
1 A 98
1 B 65
1 B 94
1 C 79
1 C 54
1 D 78
2 A 46
2 A 98
;
run;

proc sort data=A ;
by ID Class Scr;
run;

Data B;
set A;
by ID Class;
if first.class then n=1;
else n+1;
run;


proc sql;
create table want1 as
 select *,
  (select count(*) from 
    (select distinct id,scr,class from A) as b 
   where b.id=a.id and b.class=a.class and b.scr lt a.scr) as n
  from A as a
 quit;

/*output*/

1 A 45 0
1 A 58 1
1 A 98 2
1 B 65 0
1 B 94 1
1 C 79 1
1 C 54 0
1 D 78 0
2 A 46 0 
2 A 98 1



/*Every N= 2nd 3rd 4th 5th etc observation*/

Data I;
set A;
if mod(_n_,2)=0 then output;
run;


proc sql;
select * from A where mod(monotonic(salary),2)=0;quit;

/*Work on chracter function*/
Data J;
input Name $ Address $30.;
cards;
su 147averdeehi
St 157bdbdbiharff
sq 154ddnsmssnsnddd
bg 187nscnscns
;run;

data II;
set J;
new=substr(address,4,30);
new2=substr(address,length(Address)-5,3);
new3=catx(Name,address);
new4=cats(Name,address);
new5=cat(Name,address);
new6=compress(address,"",'A');
run;

%symdel x y;
/*macro types*/

%let x=1;
%let y=2;
%let x2=3;
%let x3=4;
%put &x;  1
%put &y;  2
%put &&x; 1
%put &x&y; 12
%put &&x&y; 13
%put &&x&&&x3; 1&4
%put &&x&&x&y;
%put &&x&y&y;


/*Merges and join*/

Data A;
Input ID Name$ Height;
cards;
1 A 1
3 B 2
5 C 2
7 D 2
9 E 2
;
run;

Data B;
Input ID Name$ Weight;
cards;
2 A 2
4 B 3
5 C 4
7 D 5
;
run;

/*We can merge without usinggg the by statmnt and no need to sort the data howwever it overrites the previous observation*/
data C;
Merge A B;
run;
/*Table*/
/*ID Name hgt Wgt*/
/*2   A   1   2*/
/*4   B   2   3 */
/*5   C   2   4*/
/*7   D   2   5*/
/*9   E   2   2*/

data C1;
set a B;
run;
/*it wil just add the data append the data*/

/*Table*/
/*name Name  Hgt wwgt*/
/*1    A     1    .*/
/*3    B     2    .*/
/*5    C     2    .*/
/*7    D     2    .*/
/*9    E     2    .*/
/*2    A     .    2*/
/*4    B     .    3*/
/*5    C     .    4*/
/*7    D     .    5*/

proc sql;
create table  C2 as select a.*,b.* from A as A ,B as b;
quit;

/*Wihout where or on statment It wwil work  as Cartesian product*/
/*table*/
/*ID Name Height Weight*/
/*1	A	1	2*/
/*3	B	2	2*/
/*5	C	2	2*/
/*7	D	2	2*/
/*9	E	2	2*/
/*1	A	1	3*/
/*3	B	2	3*/
/*5	C	2	3*/
/*7	D	2	3*/
/*9	E	2	3*/
/*1	A	1	4*/
/*3	B	2	4*/
/*5	C	2	4*/
/*7	D	2	4*/
/*9	E	2	4*/
/*1	A	1	5*/
/*3	B	2	5*/
/*5	C	2	5*/
/*7	D	2	5*/
/*9	E	2	5*/


/*Full join*/

proc sql;
create table D6 as select coalesce(a.ID,b.ID) as ID, coalesce(a.name,b.name) as Name,Height,Weight from A Full join B on A.id=B.id;
run;

Data D;
merge A B;
by ID;
run;

/*Table*/
/*ID	Name Height	Weight*/
/*1	A	1	.*/
/*2	A	.	2*/
/*3	B	2	.*/
/*4	B	.	3*/
/*5	C	2	4*/
/*7	D	2	5*/
/*9	E	2	.*/

/*Inner Join with data step merge and sql, In Sql when wwe are not using stattment join then we have to use on instead of where and vice versa*/
Data D1 D2;
merge A (in=a) B(in=b);
by ID;
If a and B then output D1;
else output D2;
run;

Data D3;
merge A (in=a) B(in=b);
by ID;
If a=1 and B=1;
run;

proc sql;
create table E as select a.*,b.* from A ,B where a.ID=B.ID;
quit;

proc sql;
create table F as select a.*,b.* from A inner join B on a.ID=B.ID;
quit;

/*Table*/
/*ID Name Hgt Wgt*/
/**/
/*5   C   2  4*/
/*7   D   2  5*/
/**/


/*left join*/
Data D2;
merge A (in=a) B(in=b);
by ID;
If a;
run;

proc sql;
create table D3 as select a.*,b.* from A left join B on A.id=B.id;
run;

/*table */
/*ID	Name	Height	Weight*/
/*1	A	1	.*/
/*2	A	.	2*/
/*3	B	2	.*/
/*4	B	.	3*/
/*5	C	2	4*/
/*7	D	2	5*/
/*9	E	2	.*/


/*Right join*/

Data D4;
merge A (in=a) B(in=b);
by ID;
If b;
run;

proc sql;
create table D5 as select coalesce(a.ID,b.ID) as ID, coalesce(a.name,b.name) as Name,Height,Weight from A Right join B on A.id=B.id;
run;

/*Table*/
/*ID	Name Height	Weight*/
/*2	A	.	2*/
/*4	B	.	3*/
/*5	C	2	4*/
/*7	D	2	5*/


/*one to many merge and many to many merge */
Data C;
Input ID Name$ Height;
cards;
1 A 1
5 B 2
5 C 3
7 D 2
7 M 8
9 E 2
;
run;

Data D1;
Input ID Name$ 3-4 Weight 5-6;
cards;
2 G .
5 R 3
5   4
5 Q 5
7 O 4 
;
run;

data D7;
merge C D; 
by ID;
run;

/*table*/
/*ID name Hgt wgt*/
/*1 A     1   .*/
/*2 G     .   .*/
/*5 R     2   3*/
/*5       3   4*/
/*5 Q     3   5*/
/*7 O     2   4*/
/*7 M     8   4*/
/*9 E     2   .*/


/*Append types*/

Proc append base= E data=B force;
run;

data D8;
set A B c;
run;

proc datasets Nolist;
append base=F data=A Force;
append base=F data=B Force;
run;

proc sql;
create table D10 as select * from C
OUTER UNION CORR
select * from D;
Quit;

/*Here we can append the data but the problem in using union and union all is that it ignores the variable and overwrites in the other variable one dataset have x and Y other  daatset have x and z so it will add the z into y variable*/
/*the on difference between union and uunion all is that unioin can simply avoid duplicates*/

proc sql;
create table D9 as select * from C
Union all
select * from D;
Quit;

/*How to savve datasets i wwant*/
Proc datasets lib=work;
save D1-D10;
Delete D1;
run;

/*Count number of observation*/

data _null_;
set D nobs= nobs;
put one= nobs;
stop;
run;

proc means data=D3 N;
run;

proc freq data=D1;
run;

proc sql;
select nobs into: S from dictionary.tables where upper(libname)='WORK' and upper(memname)='D4';
run;

%put &s;
Data A2;
set A End=Eof;
if EOF;
run;

/*Count missing values*/

proc freq;

proc means data=A2 Nmiss;
run;

proc sql;
select nmiss(name) as one from D2;
run;


/*Repalce missing value*/

options missing=0;  /*will change all numeric to 0*/

Data A;
set B;
if Nu = . then nu=0;
run;

/*count coulumn*/

Proc sql;create table new as select name from dictionary.columns where upper(libname)='WORK' and upper(memname)='D4';
run;
proc contents;


/*Join with more than 2 tables*/
Data C1;
input Name $ age hgt;
cards;
S 29 175
A 07 148
Q 41 150
D 54 150
;

Data C2;
input Name $ Wgt;
cards;
S 175
A 148
Q 150
D 150
E 471
;

Data C3;
input Name $ sex $;
cards;
S F
A M
Q M
D F
;
run;

proc sql;
create table c4 as select * from C1 as A, C2 as b,C3 as c where a.name=b.name and a.name=c.name
order by Name;
run;

/*Common Id  wwith all the variable*/

proc sql;
create table c5 as select * from C1 as A left join C2 as b on a.name=b.name left join C3 as c on a.name=c.name;
run;

/*Self Join*/
data C6;
input Name $ ID ManagerID;
cards;
Smith 123 456
Robert 456  .
William 222 456
Daniel . 222
Cook 383 222
;
run;

proc sql;
create table c8 as select b.name as Mgr from C6 as b  ;
select b.ID as Mgr1 from C6 as b  ;

run;

proc sql;
create table c7 as select a.* ,b.name as Mgr ,b.id from C6 as a left join C6 as b on a.ManagerID=b.Id ;
run;

data C8;
input Parent $ Child $;
cards;
A1  B1
A2  B3
B1  C1
C1  D2
B3  C3
;
run;

proc Sql;
create table c9 as select a.*,b.child as Grandchild from C8 as  a left join c8 as b on a.child=b.parent;
run;


/*Impicit and expiciit conversions*/

/*A character variable WEIGHT_KG is created and assigned the value “75?, the subsequent statement attempts to use this character 
variable in a numeric calculation. In this instance SAS will perform an implicit conversion 
(i.e. SAS automatically converts the variable from character to numeric so that the calculation can take place)*/

/*example: when it is changing in a calculation*/

DATA weight;
  weight_kg = "75";
  weigth_st = 0.157473 * weight_kg;
RUN;

/*Explicit*/

/*With thhe hepl of input and put function*/

DATA weight;
  weight_kg = "75";
  weigth_st = 0.157473 * INPUT(weight_kg,BEST.);
RUN;


/*output function*/


Data input;
Input var1 $ var2 $;
cards;
A        one
A        two
B        three
C        four
A        five
;run;

data WORK.ONE WORK.TWO;
  set WORK.INPUT;
  if Var1='A' then output WORK.ONE;
  output;
run;

/*When we are using output function in an other statement it will append the original data with the condiona data as in here it should give 3 obs with A*/
/*however it is appending the  obs with he old data set*/

/*tables*/
/*A        one*/
/*A        one*/
/*A        two*/
/*A        two*/
/*B        three*/
/*C        four*/
/*A        five*/
/*A        five*/

Data example2;
input pre $ post $ count;
cards;
Yes Yes 30
Yes No 10
No Yes 40
No No 20
;
run;
proc freq data=example2;
tables pre*post;
weight count;run;


data C6;
input Name $ ID ManagerID;
cards;
Smith 123 456
Robert 456  .
William 222 456
Daniel . 222
Cook 383 222
;
run;

data C9;
input Name $ ID ManagerID;
cards;
Smith 123 456
Robert 456  .
William 222 456
Daniel . 222
Cook 383 222
;
run;

data c10;
set c9;
call missing (ID,0);
run;



data oldfile;
input id First$;
cards;
5463 Olsen
6574 Hogan
7896 Bridge
4352 Anson
5674 Leach
7902 Wilson
9786 Fabes
;

data newfile;
input id First$;
cards;
5463 Olsen
6574 Hogan
7896 Bridge
4352 Anson
5671 Leach
7900 Wilson
9786 Sampo
2112 Ramav
;
proc sql;
title "Updated Rows";
select * from newfile
except
select * from oldfile;
quit;


/*types of macro creation*/

%let x=12;

data _null_;
call symput ('Y',13);
run;

proc sql;
select names into: Z from dictionary.tables where libname='Work' and memname=New;
run;

%macro one(Ds,ss);
data &ds;
set ss;
run;
%mend;

data On&i;
do i= 1 to 7;
set New;
end;
run;

/*Import macro*/

Options mlogic mprint symbolgen;
%macro import;


%do i= 1 %to 3;
proc import datafie="D:\SAS Dumps\Car_sales&i..csv" out=work.car_sales&i. Dbms=csv replace;
getnames=yes;
datarow=2;
run;
%end;
%mend;

%import;

proc sql;
select count (distinct memname) into: one from dictionary.tables where upper(libname)='WORK';
quit;

%put &one;

libname  UU "D:\SAS Dumps\Exp";
libname UU Clear;

proc datasets lib=Work;
save Car_sales1 Car_sales2 Car_sales3;
run;

/*Export macro*/

Options mlogic mprint symbolgen;
%macro export;
 
proc sql;
select count (distinct memname) into: one from dictionary.tables where upper(libname)='WORK';
quit;

%do i= 1 %to &one.;

proc export data=Work.Car_sales&i. outfile="D:\SAS Dumps\Exp\Car_sales&i..csv"  dbms=csv replace;
run;

%end;

%mend;

%export;


/*Append Macro*/

%macro Appe;
proc sql;
select count (distinct memname) into: one from dictionary.tables where upper(libname)='WORK';
quit;
%do i= 1 %to &one.;
proc append base=Car_sales  data=car_sales&i. Force;
run;
%end;%mend;

%Appe;


/*merge macro*/

%macro Merge;
proc sql;
select count (distinct memname) into: one from dictionary.tables where upper(libname)='WORK';
quit;

%do i= 2 %to &one.;
data New;
Merge Car_sales1 car_sales&i.;
run;
%end;%mend;

%merge;

/*Output*/


Options mlogic mprint symbolgen;

%macro KK;
proc sql;
select distinct Vehicle_type into: two separated by '#' from Car_sales1;
run;

proc sql;
select count(distinct Vehicle_type) into : three from Car_sales1;
run;
%put &two.;

%do i= 1 %to &three.;


Data test_&i.;
set Car_sales1;
%if Vehicle_type ='%scan(&two.,&i.,'#')' %then output test_&i.;
run;
%end;
%mend; 

%kk;



Data A;
input Name $ Salary 8.;
cards;
A 5555
B 4000
C 5544
D 68146
E 1641
F 545
;
run;

Data B;
input Name $ Salary 8.;
cards;
A 5555
A 4000
A 5544
C 1641
C 545
;
run;

proc sql;
select * from A where Salary GE (select avg(salary) from b);Quit;


proc sql;
select * from B where salary in (select max(salary) from B where salary not in (select max(salary) from B )group by Name)group by name ;
quit;

proc sort data=B ;
by name descending salary;run;

Data C;
set B;
by name;
if first.name then n=1;
else n+1;
if n=2 then output;
run;

proc sort data=B out=One Nodupkey;
by Name; run;



Data B;
input Name $ Salary 8.;
cards;
A 5555
A 4000
A 5544
C 1641
C 545
;
run;

proc sql;
select * from B group by name order by Salary desc;
quit;

proc sql;
select name into: oo separated by '#' from dictionary.columns where upper(libname)="WORK" and upper(memname)="B";
Quit; 

%put  &oo;
data _null_;
set b nobs=nobs;
put one=nobs;
stop;
run;

data D;
set B End=Eof;
if Eof then output;
run;

proc means data=B N;
run;
proc freq data=b;
run;

data _null_;
set B nobs=nobs;
call symput('New',nobs);
run;

proc sql;
select count(*) into: one from B;
quit;

%put &one.;

data example2;
input Name $ ID ManagerID;
cards;
Smith 123 456
Robert 456  .
William 222 456
Daniel 777 222
Cook 383 222
;
run;

Proc sql ;
select a.*,b.name as  mgrna from example2 as a left join example2 as b on a.ManagerID=b.ID;
quit;

proc sort data=example2 out=x;
by ManagerID;
run;
proc sort data=example2 out=y (rename=(Name=Manager ID=ManagerID ManagerID=ID));
by ID;
run;
data want;
merge x (in= a) y (in=b);
by managerid;
if a;
run;

data example22;
input Parent $ Child $;
cards;
A1  B1
A2  B3
B1  C1
C1  D2
B3  C3
;
run;

proc sql;
select a.*,b.Parent as Grand from example22 as a left join example22 as b on a.parent=b.child;quit;



proc sort data=example22 out= X ;
by parent ;run;

proc sort data=example22 out= Y (rename=(parent=Grand Child=parent))  ;
by parent ;run;

	data want;
	merge x (in= a) y (in=b);
	by parent;
	if a;
	run;


To see the size of the datasets

proc sql;
select Memname,NlOBS,Filesize format=Sizekb10.2 from dictionary.tables where Upper(libname)='SASHELP' and upper(MEMNAME)='AIR';
quit;

To Adding sum group by


Data A;
Input BID Time $ Temp;
cards;
1  AM 15
2  PM 78
1 AM 45
5 AM 98
6 PM 105
;
run;

proc sort data=A;
by time;
Data B(keep=Time Temp1);
set A;
by Time;
if First.time then Temp1=0;
else Temp1+temp;
if last.time;
run;

Data A;
Input BID Time $ Temp;
cards;
1  10:AM 15
2  20:PM 78
1 15:AM 45
5 05:AM 98
6 9:PM 105
;
run;


data A1;
set A;
time2=scan(time,2,':');
Time1=substr(Time,(length(time)-1),2);
run;



Connection string 

proc sql;
connect to oracle as AB( userid= paswoord= dsn=);
create table abc as select * from AB 
(select * from ABC );
quit;

libname odbclib odbc uid=id pwd=password dsn=test;




 






















