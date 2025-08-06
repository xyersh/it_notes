```sql
select COUNT(DISTINCT a2.issue_id) as done_without_plan
from (
 #Поиск исполненных в периоде задач и плана
 select a1.issue_id, ANY_VALUE(a1.done_time) AS done_time, GROUP_CONCAT((case WHEN IFNULL(custom_values.custom_field_id, 0) = 76 then custom_values.value  else "" end) SEPARATOR "") plan_date
 from (  
     #Закрытые и отправленные задачи
     (select a10.issue_id, a10.done_time
     from (select a9.issue_id, a9.issue_created, max(a9.last_time) as done_time
     from (SELECT a8.*, journal_details.value as new_status_value
     from (select a7.*, journals.id as jou_id, journals.created_on as last_time 
     from (select a6.issue_id, a6.issue_created,  a6.max_time
     from (SELECT a5.*, journal_details.value as new_status_value
     from (select a4.*, journals.id as jou_id, journals.created_on as last_time 
     from (select  a3.issue_id, a3.issue_created, max(a3.last_time) as max_time
     from (SELECT a2.*, journal_details.value as new_status_value
     from (select a1.*, journals.id as jou_id, journals.created_on as last_time 
     from (select issues.id as issue_id, issues.created_on as issue_created
     from issues, projects
     where issues.tracker_id = 5
         and projects.name IN (:proj_list)
     and issues.project_id = projects.id
     ) a1 LEFT JOIN journals 
     on journals.journalized_id= a1.issue_id
     AND journals.created_on >= :date1
     AND journals.created_on < :date2
     ) a2 left join journal_details on a2.jou_id = journal_details.journal_id   
     and journal_details.prop_key = "status_id" 
     where journal_id is not null ) a3
     GROUP BY a3.issue_id )a4 LEFT JOIN journals 
     on journals.journalized_id= a4.issue_id 
     AND journals.created_on >= :date1
     AND journals.created_on < :date2
     ) a5 left join journal_details on a5.jou_id = journal_details.journal_id   
     and journal_details.prop_key = "status_id" 
     where journal_id is not null) a6
     where a6.last_time = a6.max_time
     and a6.new_status_value = 5 )a7 LEFT JOIN journals 
     on journals.journalized_id= a7.issue_id 
     AND journals.created_on < :date2
     ) a8 left join journal_details on a8.jou_id = journal_details.journal_id   
     and journal_details.prop_key = "status_id" 
     where journal_id is not null
     and a8.last_time < a8.max_time)a9
     group by a9.issue_id)a10
     where a10.done_time >= :date1
     AND a10.done_time < :date2
     )UNION(
     select a6.issue_id, a6.last_time as done_time
     from (SELECT a5.*, journal_details.value as new_status_value
     from (select a4.*, journals.id as jou_id, journals.created_on as last_time 
     from (select  a3.issue_id, a3.issue_created, max(a3.last_time) as max_time
     from (SELECT a2.*, journal_details.value as new_status_value
     from (select a1.*, journals.id as jou_id, journals.created_on as last_time 
     from (select issues.id as issue_id, issues.created_on as issue_created
     from issues, projects
     where issues.tracker_id = 5
         and projects.name IN (:proj_list)
     and issues.project_id = projects.id
     ) a1 LEFT JOIN journals 
     on journals.journalized_id= a1.issue_id
     AND journals.created_on >= :date1
     AND journals.created_on < :date2
     ) a2 left join journal_details on a2.jou_id = journal_details.journal_id   
     and journal_details.prop_key = "status_id" 
     where journal_id is not null ) a3
     GROUP BY a3.issue_id )a4 LEFT JOIN journals 
     on journals.journalized_id= a4.issue_id 
     AND journals.created_on >= :date1
     AND journals.created_on < :date2
     ) a5 left join journal_details on a5.jou_id = journal_details.journal_id   
     and journal_details.prop_key = "status_id" 
     where journal_id is not null) a6
     where a6.last_time = a6.max_time
     and a6.new_status_value = 23)
 )a1 
 LEFT JOIN custom_values
 on a1.issue_id = custom_values.customized_id  
 and custom_values.customized_type = "Issue"
 GROUP BY a1.issue_id
)a2
where a2.plan_date = ""

```