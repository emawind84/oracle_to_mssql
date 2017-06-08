# oracle_to_mssql
Some regular expressions in order to convert Oracle queries into MS SQL queries


```

# http://www.sqlines.com/oracle-to-sql-server
# oracle to ms sql converter : http://www.sqlines.com/online
# http://regexr.com/

TO_CHAR\s?\(\s?([\._\w\d]*)\s?,\s?['"]YYYY-MM['"]\s?\)  =>   convert(varchar(7), $1, 20)
TO_CHAR\s?\(\s?([\._\w\d]*)\s?,\s?['"]YYYY-MM-DD['"]\s?\)  =>   convert(varchar(10), $1, 20)
TO_CHAR\s?\(\s?([\._\w\d]*)\s?,\s?['"]YYYY-MM-DD HH24:MI['"]\s?\)  =>   convert(varchar(16), $1, 20)
TO_CHAR\s?\(\s?([\._\w\d]*)\s?,\s?['"]YYYY-MM-DD HH24:MI:SS['"]\s?\)  =>   convert(varchar(19), $1, 20)
TO_CHAR\s?\(\s?([\._\w\d]*)\s?,\s?['"]YYYYMMDD['"]\s?\)  =>  replace( convert(varchar(10), $1, 20), '-', '' )
TO_CHAR\s?\(\s?([\._\w\d]*)\s?,\s?['"]YYYY-WW['"]\s?\)  =>  concat(DATEPART( yyyy, $1),'-',DATEPART( wk, $1))
TO_CHAR\s?\(\s?([\._\w\d]*)\s?,\s?['"]D['"]\s?\)  =>  DATEPART( dw, $1)

TO_CHAR\(\s?(convert\(datetime,\s?[#\._\w\d\)\(]*,\s?\d+\)),\s?['"]YYYYMMDD['"]\s?\)  =>  replace( convert(varchar(10), $1, 20), '-', '' )
TO_CHAR\(\s?(convert\(datetime,\s?[#\._\w\d\)\(]*,\s?\d+\)),\s?['"]YYYY-MM['"]\s?\)  =>  convert(varchar(7), $1, 20)
TO_CHAR\(\s?(convert\(datetime,\s?[#\._\w\d\)\(]*,\s?\d+\)),\s?['"]YYYY-MM-DD['"]\s?\)  =>  convert(varchar(10), $1, 20)
TO_CHAR\(\s?(convert\(datetime,\s?[#\._\w\d\)\(]*,\s?\d+\)),\s?['"]YYYY-MM-DD HH24:MI['"]\s?\)  =>  convert(varchar(16), $1, 20)
TO_CHAR\(\s?(convert\(datetime,\s?[#\._\w\d\)\(]*,\s?\d+\)),\s?['"]YYYY-MM-DD HH24:MI:SS['"]\s?\)  =>  convert(varchar(19), $1, 20)

TO_CHAR\s*\(\s*[\d\w\._\-]*\s*\)  =>  convert(varchar(30), $1)

TO_CHAR(DT, 'YYYY')  ???
TO_CHAR(DT, 'MM')  ???
TO_CHAR(DT, 'DD')  ???
TO_CHAR(SUM( PRE_REAL_PER ), 'FM99990D99')
TO_CHAR(COUNT(*))

(TO_CHAR)(\s?\(\s?([#\._\w\d\s]*)\s?,\s*'fm)  =>  ssma_oracle.to_char_numeric$2
(TO_CHAR)(\s*\(SUM\([#\._\w\d\s]*\)\s*,\s*'fm)  =>  ssma_oracle.to_char_numeric$2
TO_CHAR(convert(datetime, MIN(COST_DATE), 20),'YYYY-MM-DD')  ???

TO_DATE\(\s*([#\._\w\d]*)\s*,\s*['"]YYYY-?MM-?DD['"]\s*\)  =>  convert(datetime, $1, 20)
TO_DATE\(\s*replace\(\s*([#\._\w\d]*)\s*,\s*'-'\s*,\s*''\s*\),\s*['"]YYYYMMDD['"]\s*\)  =>  convert(datetime, $1, 20)
TO_DATE\(\s?(replace\(\s?convert\(varchar\(10\),\s?[#\._\w\d\(\)]*\s?, 20\)\s?,\s?'-'\s?,\s?''\s?\)),\s?['"]YYYYMMDD['"]\s?\)  =>  convert(datetime, $1, 20)
TO_DATE\(\s*(ISNULL\([#\._\w\d]*\s*,\s*[#\._\w\d]*\)),\s*['"]YYYY-?MM-?DD['"]\s*\)  =>  convert(datetime, $1, 20)
TO_DATE\(\s?(MAX\([#\._\w\d]*\s?\)),\s?['"]YYYY-?MM-?DD['"]\s?\) =>  convert(datetime, $1, 20)
TO_DATE\(\s?(MIN\([#\._\w\d]*\s?\)),\s?['"]YYYY-?MM-?DD['"]\s?\) =>  convert(datetime, $1, 20)

# replace to_date with with ssma_oracle.to_date2
(\(?)(?<!ssma_oracle\.)TO_DATE\s*\(  =>  $1 ssma_oracle.to_date2(

# the keyworkd DEFAULT must be specified if the function has default values ex. ssma_oracle.lpad_nvarchar('#', 4, default)
LPAD  =>  ssma_oracle.lpad_nvarchar

# add the prefix dbo. to all the functions with prefix 'get_'
(?<!\.)(GET_[_\w]*)\(  =>  dbo.$1(

DECODE\(\s*([#\*\._\w\d\(\)]*)\s*,\s*([#\._\w\d']*)\s*,\s*([#\._\w\d']*)\s*,\s*([#\._\w\d']*)\s*\)  =>  CASE $1 WHEN $2 THEN $3 ELSE $4 END
DECODE\(\s*([#\*\._\w\d\(\)]*)\s*,\s*([#\._\w\d']*)\s*,\s*([#\._\w\d']*)\s*,\s*([#\._\w\d']*)\s*,\s*([#\._\w\d']*)\s*,\s*([#\._\w\d']*)\s*\)  =>  CASE $1 WHEN $2 THEN $3 WHEN $4 THEN $5 ELSE $6 END

# replace nvl2 and isnull2 functions
(?:ISNULL|NVL)2\s*\(\s*([#\._\w\d]*)\s*,\s*([#\._\w\d']*)\s*,\s*([#\._\w\d']*)\s*\)  =>  CASE WHEN $1 IS NOT NULL THEN $2 ELSE $3 END

SYSTIMESTAMP  =>  SYSDATETIMEOFFSET()

# to change manually!
CASE A WHEN NULL THEN B END  =>  CASE WHEN A IS NULL THEN B END
CASE A WHEN NULL THEN B WHEN C THEN D  =>  CASE WHEN A IS NULL THEN B WHEN A = C THEN D

CASE\s*([\d\w_\-\.]*)\s*WHEN\s*NULL\s*THEN\s*([\d\w\-_\|#']*)\s*ELSE\s*([\d\w\-_\|#']*)\s*END  =>  CASE WHEN $1 IS NULL THEN $2 ELSE $3 END

AS\s(SAVE|PRINT)(?=[\s,])  =>  AS "$1"
\.(SAVE|PRINT)(?=[\s,])  =>  ."$1"

SYSDATE  =>  GETDATE()
NVL  =>  ISNULL
chr\(  =>  CHAR(
([#\d\w_-]*)\.nextval  =>  NEXT VALUE FOR $1

['"]%['"]\s*\|\|  => '%' + 
\|\|\s*['"]%['"]  =>  + '%'

trunc ???

RTRIM(A, B)  =>  ssma_oracle.trim_nvarchar(2, A, B)
LTRIM(A, B)  =>  ssma_oracle.trim_nvarchar(1, A, B)
TRIM(A, B)  =>  ssma_oracle.trim_nvarchar(3, A, B)
TRIM(A)  =>  LTRIM(RTRIM(3, A))

ADD_MONTHS\(\s*(?:getdate\(\)|sysdate),\s*([\-\d]+)\s*\)  =>  DATEADD( month, $1, getdate() )
ADD_MONTHS\(\s*(ssma_oracle.to_date2\([#\._\-\|\w\d\s']+,\s*[#\._\-\w\d']+\s*\))\s*,\s*([\-\s\d]+)\s*\)  =>  DATEADD(month, $2, $1 )

(?<=[\s\(])(LENGTHB?)  =>  LEN

INSTR\(\s*([#\._\w\d']*\s*,\s*[#\._\-\w\d'=\|]*)\s*\)  =>  ssma_oracle.instr2_nvarchar($1)
INSTR\(\s*([#\._\w\d']*\s*,\s*[#\._\-\w\d'=\|]*\s*,\s*[\-\d]*\s*,\s*[\-\d]*)\s*\)  =>  ssma_oracle.instr4_nvarchar($1)

TRUNC( DATE1 - DATE2 )  =>  CAST( DATE1 - DATE2 AS INT )

SUBSTR => SUBSTRING
|| => +
STDDEV => STDEV
CEIL => CEILING
MOD(10, 3) => 10%3
MINUS => EXCEPT
ROUND(num) => ROUND(num, 0)
TRUNC(Number) => ROUND(Number, 0, 1)

```
