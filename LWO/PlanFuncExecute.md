```sql
SELECT COUNT(DISTINCT a3.issue_id) as plan
from (
 select a2.issue_id, CONCAT(a2.plan_date," 23:59:58") as plan_date
 from (
     select a1.issue_id, GROUP_CONCAT((case WHEN IFNULL(custom_values.custom_field_id, 0) = 76 then custom_values.value  else "" end) SEPARATOR "") plan_date
     from (
         select issues.id as issue_id from issues, projects
         where issues.tracker_id = 5
         and issues.project_id = projects.id
                 and projects.name IN (:proj_list)
     )a1
     LEFT JOIN custom_values
     on a1.issue_id = custom_values.customized_id  
     and custom_values.customized_type = "Issue"
     GROUP BY a1.issue_id
 )a2
 WHERE a2.plan_date != ""
)a3
where a3.plan_date >= :date1
and a3.plan_date < :date2
```