# 存储过程
```oracle
不建包就用系统自带的my_cur out sys_refcursor

create or replace Package SYSBasic
as
  type cc_cursor is ref cursor;
end SYSBasic;

CREATE OR REPLACE
procedure usp_GetMSG4UserLogin(UserID in number,my_cur out SYSBASIC.cc_CURSOR) 
is
BEGIN
open my_cur for
select * from "ORAPS_Message"."MS_Message" where "ID"=UserID;
END;


```

# GUID
```sh
DECLARE i number;
 guid VARCHAR2(50);
begin
i:=0;
for i in 1..50000 loop
select rawtohex(sys_guid()) into guid from dual;
INSERT INTO "zzz" ("zzz","id") VALUES(guid,i);
end loop;
end;
```