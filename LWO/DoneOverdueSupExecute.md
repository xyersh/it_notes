```sql
select a7.issue_id, a7.date_created, a7.done_time, 
    case when a7.sup_days = "" then 15 else a7.sup_days end as sup_days
from (
 select a6.issue_id, a6.date_created, a6.done_time, GROUP_CONCAT((case WHEN IFNULL(custom_values.custom_field_id, 0) = 69 then custom_values.value else "" end) SEPARATOR "") sup_days
 from (
     select a5.issue_id, max(a5.date_created) as date_created, a5.done_time, a5.project_id
     from (  
         #Поиск последней даты возврата, если есть
         select a4.*, case when IFNULL(a4.new_status_value,0)=31 then a4.last_time else a4.issue_created end as date_created
         from (  
             select a3.*, journal_details.value as new_status_value
             from (  
                 select a2.issue_id, a2.issue_created, a2.done_time, journals.id as jou_id, journals.created_on as last_time, a2.project_id
                 from (
                     #Поиск задач с типом s1
                     select a1.issue_id, ANY_VALUE(a1.issue_created) issue_created, a1.done_time, GROUP_CONCAT((case WHEN IFNULL(custom_values.custom_field_id, 0) = 19 then custom_values.value else "" end) SEPARATOR "") sup_type, ANY_VALUE(a1.project_id) project_id
                     from (
                     
                        -- region wer
                         #Закрытые и отправленные задачи
                         (select a10.issue_id, a10.issue_created, a10.done_time, a10.project_id
                         from (
                             select a9.issue_id, a9.issue_created, max(a9.last_time) as done_time, a9.project_id
                             from (
                                 SELECT a8.*, journal_details.value as new_status_value
                                 from (
                                     select a7.*, journals.id as jou_id, journals.created_on as last_time 
                                     from (
                                         select a6.issue_id, a6.issue_created,  a6.max_time, a6.project_id
                                         from (
                                             SELECT a5.*, journal_details.value as new_status_value
                                             from (
                                                 select a4.*, journals.id as jou_id, journals.created_on as last_time 
                                                 from (
                                                     select  a3.issue_id, a3.issue_created, max(a3.last_time) as max_time, a3.project_id
                                                     from (
                                                            SELECT a2.*, journal_details.value as new_status_value
                                                            from (
                                                                    select a1.*, journals.id as jou_id, journals.created_on as last_time 
                                                                    from (
																		select issues.id as issue_id, issues.created_on as issue_created, issues.project_id as project_id
																		 from issues, projects
																		 where issues.tracker_id = 7
																								 and issues.project_id = projects.id
																								 and projects.name IN (:proj_list)
                                                                        ) a1 
                                                                    LEFT JOIN journals 
                                                                    on journals.journalized_id= a1.issue_id
                                                                    AND journals.created_on >= :date1
                                                                    AND journals.created_on < :date2
                                                                    ) a2 
                                                            left join journal_details 
                                                            on a2.jou_id = journal_details.journal_id   
                                                            and journal_details.prop_key = "status_id" 
                                                            where journal_id is not null 
                                                            ) a3
                                                            GROUP BY a3.issue_id 
                                                    )a4 
                                                LEFT JOIN journals 
                                                on journals.journalized_id= a4.issue_id 
                                                AND journals.created_on >= :date1 
                                                AND journals.created_on < :date2
                                                ) a5 
                                            left join journal_details 
                                            on a5.jou_id = journal_details.journal_id   
                                            and journal_details.prop_key = "status_id" 
                                            where journal_id is not null
                                            ) a6
                                         where a6.last_time = a6.max_time
                                         and a6.new_status_value = 5 
                                         )a7 
                                     LEFT JOIN journals 
                                     on journals.journalized_id= a7.issue_id 
                                     AND journals.created_on < :date2
                                    ) a8 
                                 left join journal_details 
                                 on a8.jou_id = journal_details.journal_id   
                                 and journal_details.prop_key = "status_id" 
                                 where journal_id is not null
                                 and a8.last_time < a8.max_time
                                )a9
                             group by a9.issue_id
                             )a10
                         where a10.done_time >= :date1
                         AND a10.done_time < :date2
                         )
                         -- endregion 
                         
                         
                         UNION
                         
                         (
                         
                         select a6.issue_id, a6.issue_created, a6.last_time as done_time, a6.project_id
                         from (
                             SELECT a5.*, journal_details.value as new_status_value
                             from (
                                 select a4.*, journals.id as jou_id, journals.created_on as last_time 
                                 from (
                                     select  a3.issue_id, a3.issue_created, max(a3.last_time) as max_time, a3.project_id
                                     from (
                                             SELECT a2.*, journal_details.value as new_status_value
                                             from (
                                                select a1.*, journals.id as jou_id, journals.created_on as last_time 
                                                from (
                                                    select issues.id as issue_id, issues.created_on as issue_created, issues.project_id as project_id
                                                    from issues, projects
                                                    where issues.tracker_id = 7
                                                                     and issues.project_id = projects.id
                                                                     and projects.name IN (:proj_list)
                                                    ) a1 
                                                LEFT JOIN journals 
                                                on journals.journalized_id= a1.issue_id
                                                AND journals.created_on >= :date1
                                                AND journals.created_on < :date2
                                                ) a2 left join journal_details on a2.jou_id = journal_details.journal_id   
                                             and journal_details.prop_key = "status_id" 
                                             where journal_id is not null 
                                            ) a3
                                     GROUP BY a3.issue_id 
                                     )a4 
                                 LEFT JOIN journals 
                                 on journals.journalized_id= a4.issue_id 
                                 AND journals.created_on >= :date1
                                 AND journals.created_on < :date2
                                 ) a5 
                             left join journal_details on a5.jou_id = journal_details.journal_id   
                             and journal_details.prop_key = "status_id" 
                             where journal_id is not null
                             ) a6
                         where a6.last_time = a6.max_time
                         and a6.new_status_value = 23
                         )
                     )a1 
                     LEFT JOIN custom_values
                     on a1.issue_id = custom_values.customized_id  
                     and custom_values.customized_type = "Issue"
                     GROUP BY a1.issue_id, a1.issue_created, a1.done_time
                 )a2 
                 LEFT JOIN journals 
                 on journals.journalized_id= a2.issue_id 
                 and journals.created_on < :date2
                 where a2.sup_type = "S1 - Ошибка ПО"
             )a3 left join journal_details on a3.jou_id = journal_details.journal_id   
             and journal_details.prop_key = "status_id" 
             and journal_details.value = 31
         )a4
     )a5 
     GROUP BY a5.issue_id, a5.done_time
 )a6
 LEFT JOIN custom_values
 on a6.project_id = custom_values.customized_id  
 and custom_values.customized_type = "Project"
 GROUP BY  a6.issue_id, a6.date_created, a6.done_time
)a7
```