Случай, когда нам не нужно возвращаемый курсор, и интересно только проведение MDL внутри функции

```plsql
declare
	v_cursor SYS_REFCURSOR; 
begin
	v_cursor:=  cursFunWithDML(params);
end;
```