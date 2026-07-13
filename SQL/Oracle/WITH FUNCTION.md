Начиная с Oracle 12c можно использовать конструкцию `WITH ... FUNCTION` прямо внутри SQL:

```sql
WITH FUNCTION mult(a NUMBER, b NUMBER) RETURN NUMBER IS
BEGIN
    return a * b;
END;
SELECT mult(5,6)
FROM dual
```

Но у `WITH FUNCTION` есть важное ограничение: **возвращаемый тип должен быть SQL-совместимым** (`VARCHAR2`, `NUMBER`, `DATE` и т.д.)