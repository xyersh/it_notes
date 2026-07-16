
#### Создание XMLType
- **xmlType(** VARCHAR **)** XMLType - преобразует строковые данные (VARCHAR, CLOB) в XML-тип
```sql

with data as (
select
'<product>
<product_id>1</product_id>
<name>Товар 1</name>
<price>100</price>
</product>' as str
from dual
)

select xmlType(str)
from data
```

- **XMLElement(** VARCHAR,[xmlAttribute], [xmlElement], [node_value] **)** XMLType - создает xml-ноду из строкового значения
```sql
select xmlElement("branch",
                   xmlELement("id", d.id),
                   xmlElement("parent_id", d.parent_id),
                   xmlELement("dimcode", d.dimcode),
                   xmlELement("name", d.name)
       )
from dimension d
where id = 41623630
```
Вернет:
```xml
<branch>
    <id>41623630</id>
    <parent_id>40002121</parent_id>
    <dimcode>739-900</dimcode>
    <name>Дирекция по Минску и Минской области г. Минск, ул. Коллекторная, 11-2</name>
</branch>
```


- **XMLAttributes(** 'value' as "tag_name", ... **)** - создает один или несколько атрибутов "tag_name" со значением 'value' в xml-ноде. Используется в качестве параметра функции xmlElement
```sql
select XMLElement(
    "USER",
    XMLAttributes('Alex' as name, 'Ivanov' as surname, '42' as age)
    )
from dual 
```
Вернет:
```xml
<USER NAME="Alex" SURNAME="Ivanov" AGE="42"></USER>
```

- **XMLForest(** 'value1' as "node_name1", 'value2' as "node_name2" ... **)** - позволяет преобразовывать множество значений в единый XML-фрагмент.  Она особенно полезна при создании XML-структур из реляционных данных.
   допустимо использование **XMLForest()** в качестве параметра функции  **XMLElement**
```sql
select XMLForest(
        100 as "INCOME",
        500 as "EXPENCE"
    )
from dual 
```
Вернет:
```xml
<INCOME>100</INCOME>
<EXPENCE>500</EXPENCE>
```

- **XMLConcat(** XMLType, ... **)** XMLType - позволяет объединять несколько XML-фрагментов в один. Она особенно полезна при необходимости  конкатенировать результаты нескольких запросов или функций, возвращающих XML-данные.  
Ее действие несколько похоже на работу xmlELement за тем исключением, что xmlConcat при объединение Xml-фрагментов не создает корневую xml-ноду
```sql
select XMLConcat(
       XMLElement("ID", 123213),
       XMLElement("NAME", 'Natasha'),
       XMLElement("NAME", 'Ivanova')
       
    )
from dual
```
Вернет:
```xml
<ID>123213</ID>
<NAME>Natasha</NAME>
<NAME>Ivanova</NAME>
```

#### Извлечение данных из XML

- **extractValue(** XMLType, 'xpath' **)** VARCHAR - извлекает скалярное значение из XML-документа
```sql
with data as (
select xmlType(
'<ERIP_Response>
  <ServiceInfo>
    <ServiceInfoId>00000000010394660329-00-67A940</ServiceInfoId>
    <ParameterList Count="5">
      <Parameter Idx="1" Name="Универсальный">по счётчику: 1</Parameter>
      <Parameter Idx="2" Name="- проживающих (чел.)">4</Parameter>
      <Parameter Idx="3" Name="-- по показаниям (всего)">0 BYN</Parameter>
      <Parameter Idx="4" Name="Начислено">0 BYN</Parameter>
      <Parameter Idx="5" Name="Начислено/Перерасчет/Вх.сальдо">61,49 BYN</Parameter>
    </ParameterList>
    <Amount Amount_Budget="0" FineAmount="0" Currency="933" Visible="Y" Editable="Y" MinAmount="0,01" MaxAmount="99999,99" AmountPrecision="0,01">61,49</Amount>
    <ExtraInfo Count="1"  xml:space="preserve">
      <ExtraInfoText Idx="1">Сч-к 1 Холодная вода/98562 расход: 8</ExtraInfoText>
    </ExtraInfo>
    <NextRequestType>TransactionStart</NextRequestType>
  </ServiceInfo>
</ERIP_Response>') as root
from dual
)

select extractValue(root, '//Parameter[1]/text()' ) as Parameter_1, -- извлечение значения ноды
    extractValue(root, '//Amount/@Currency') as currency -- извлечение атрибута ноды
from data
```
Вернет:


| PARAMETER_1    | CURRENCY |
| -------------- | -------- |
| по счётчику: 1 | 933      |



- **extract(** XMLType, 'xpath' **)** XMLType - возвращает xml-фрагмент из XML-документа
```sql
with data as (
select xmlType(
'<ERIP_Response>
  <ServiceInfo>
    <ServiceInfoId>00000000010394660329-00-67A940</ServiceInfoId>
    <ParameterList Count="5">
      <Parameter Idx="1" Name="Универсальный">по счётчику: 1</Parameter>
      <Parameter Idx="2" Name="- проживающих (чел.)">4</Parameter>
      <Parameter Idx="3" Name="-- по показаниям (всего)">0 BYN</Parameter>
      <Parameter Idx="4" Name="Начислено">0 BYN</Parameter>
      <Parameter Idx="5" Name="Начислено/Перерасчет/Вх.сальдо">61,49 BYN</Parameter>
    </ParameterList>
    <Amount Amount_Budget="0" FineAmount="0" Currency="933" Visible="Y" Editable="Y" MinAmount="0,01" MaxAmount="99999,99" AmountPrecision="0,01">61,49</Amount>
    <ExtraInfo Count="1"  xml:space="preserve">
      <ExtraInfoText Idx="1">Сч-к 1 Холодная вода/98562 расход: 8</ExtraInfoText>
    </ExtraInfo>
    <NextRequestType>TransactionStart</NextRequestType>
  </ServiceInfo>
</ERIP_Response>') as root
from dual
)

select extract(root, '//ExtraInfoText')
from data
```
Вернет:
```xml
<ExtraInfoText Idx="1">Сч-к 1 Холодная вода/98562 расход: 8</ExtraInfoText>
```

- **XMLTable(** params **)** - позволяет преобразовывать XML-данные, хранящиеся в базе данных, в реляционную таблицу. Это позволяет вам работать с  
XML-данными, используя привычные SQL-запросы, как если бы они были обычными табличными данными.

xmlTable используется внутри блока **FROM** в связке с таблицей, из которой производится извлечение данных из XML.  
общая сигyатура функции выглядит так:  
> xmlTable(
    'xpath'                                         --определяем корневую XML-ноду для дальнейшей обработки  
    PASSING TABLE_COLUMN_NAME,   --указываем столбец таблицы с XML-содержимым, именно в нем будет производиться поиск данных  
    COLUMNS                                    -- в этом блоке укажем названия новых столбцов и путь к значениям для них  
        new_col_name1 TYPE PATH 'xpath_to_node1'     
        new_col_name2 TYPE PATH 'xpath_to_node2'  
        ...  
)

далее полученные значения в столбцах new_col_name1, new_col_name2,... можно использовать в блоке **SELECT**

```sql
WITH data AS(
SELECT
XMLType('<DATA>
    <ROW ID="1000">
        <NAME>Vashia</NAME>
        <SURNAME>Pupkin</SURNAME>
    </ROW>
    <ROW ID="1001">
        <NAME>Petya</NAME>
        <SURNAME>Zalupkin</SURNAME>
    </ROW>
</DATA>') as data
from dual
)

select  t.id, t.name, t.surname
from data d,
xmlTable(
        '//ROW'
        PASSING d.data
        COLUMNS
            id VARCHAR2(100)     PATH '@ID', 
            name VARCHAR(100)    PATH '//NAME',
            surname VARCHAR(100) PATH '//SURNAME'
    ) t
```
Вернет:

| ID   | NAME   | SURNAME  |
| ---- | ------ | -------- |
| 1000 | Vashia | Pupkin   |
| 1001 | Petya  | Zalupkin |


#### Манипулирование данными XML
**XMLAGG(** XMLparams ... **)** **XMLType** - позволяет сгруппировать множество строк в единый XML-документ.  
Это особенно полезно при необходимости объединить XML-данные из нескольких строк в одну XML-структуру.

Сценарий:  
Представим, что у нас есть таблица employees со следующей структурой:  
Задача: Сформировать XML-документ, в котором для каждого отдела будет список сотрудников.
```sql
with employees as (
select 100 as id, 10 as department_id, 'Иван' as name, 'Иванов'as last_name from dual 
union all
select 101 as id, 10 as department_id, 'Петр' as name, 'Петров'as last_name from dual 
union all
select 102 as id, 10 as department_id, 'Сидр' as name, 'Содоров'as last_name from dual 
union all
select 103 as id, 20 as department_id, 'Александр' as name, 'Александров'as last_name from dual union all
select 104 as id, 20 as department_id, 'Андрей' as name, 'Андреев'as last_name from dual 
union all
select 105 as id, 20 as department_id, 'Алексей' as name, 'Алексеев'as last_name from dual union all
select 106 as id, 20 as department_id, 'Сергей' as name, 'Сергеев'as last_name from dual 
union all
select 107 as id, 30 as department_id, 'Георгий' as name, 'Георгиев'as last_name from dual 
)

select department_id,
           XMLElement("department",
                 XMLAttributes(department_id AS "id"),
                 XMLAGG(XMLElement("employee", name || ' ' || last_name))
               ) AS employees_xml
from employees
GROUP BY department_id
```

