```sql
select COUNT(DISTINCT a11.issue_id) as s2
from (   
 #Дата гарантии
 select a10.issue_id, a10.issue_created, a10.related_issue_id, case when IFNULL(a10.end_guarantee,0)=0 then (case when a10.date_intr = "" then "" else DATE(a10.date_intr + INTERVAL 1 MONTH) end) else a10.end_guarantee end as end_guarantee
 from (
     #Поиск даты окончания внедрения
     select a9.issue_id, a9.issue_created, a9.related_issue_id, a9.end_guarantee, GROUP_CONCAT((case WHEN IFNULL(a9.custom_field_id, 0) = 89 then value else "" end) SEPARATOR "") as date_intr
     from (  
         select a8.issue_id, a8.issue_created, a8.related_issue_id, a8.end_guarantee, custom_values.custom_field_id, custom_values.value, a8.project_id
         from (
             #Поиск последней даты исполнения связанной задачи
             select a7.issue_id, a7.issue_created, a7.related_issue_id, DATE(max(a7.done_date) + INTERVAL 1 MONTH) as end_guarantee, a7.project_id
             from (  
                 select a6.*, case when IFNULL(a6.new_status_value,0) = 23 then a6.last_time else "" end as done_date
                 from (  
                     select a5.*, journal_details.value as new_status_value
                     from (  
                         select a4.*, journals.id as jou_id, journals.created_on as last_time
                         from (  
                             select a3.*, GROUP_CONCAT((case WHEN IFNULL(custom_values.custom_field_id, 0) = 37 then custom_values.customized_id else "" end) SEPARATOR "") related_issue_id
                             from (
                                 #Поиск связанной задачи в исходном виде
                                 select a2.issue_id, a2.issue_created, GROUP_CONCAT((case WHEN IFNULL(custom_values.custom_field_id, 0) = 74 then custom_values.value else "" end) SEPARATOR "") related_issue_sd, a2.project_id
                                 from (
                                     #Поиск задач с типом s2
                                     select a1.issue_id, ANY_VALUE(a1.issue_created) issue_created, GROUP_CONCAT((case WHEN IFNULL(custom_values.custom_field_id, 0) = 19 then custom_values.value else "" end) SEPARATOR "") sup_type, ANY_VALUE(a1.project_id) project_id
                                      from (
                                          #Закрытые и отправленные задачи с датой прихода
                                          (
                                          select a10.issue_id, a10.issue_created, a10.project_id
                                          from (
                                              select a9.issue_id, max(a9.last_time) as done_time, a9.issue_created, a9.project_id
                                              from (
                                                  SELECT a8.*, journal_details.value as new_status_value
                                                  from (
                                                      select a7.*, journals.id as jou_id, journals.created_on as last_time 
                                                      from (
                                                          select a6.issue_id,  a6.max_time, a6.issue_created, a6.project_id
                                                          from (
                                                              SELECT a5.*, journal_details.value as new_status_value
                                                              from (
                                                                  select a4.*, journals.id as jou_id, journals.created_on as last_time 
                                                                  from (
                                                                      select  a3.issue_id, max(a3.last_time) as max_time, a3.issue_created, a3.project_id
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
                                                                              ) a1 LEFT JOIN journals 
                                                                              on journals.journalized_id= a1.issue_id
                                                                              AND journals.created_on >= :date1 
                                                                              AND journals.created_on < :date2
                                                                              ) a2 
                                                                          left join journal_details on a2.jou_id = journal_details.journal_id   
                                                                          and journal_details.prop_key = "status_id" 
                                                                          where journal_id is not null 
                                                                          ) a3
                                                                      GROUP BY a3.issue_id 
                                                                      )a4 
                                                                  LEFT JOIN journals 
                                                                  on journals.journalized_id= a4.issue_id 
                                                                  AND journals.created_on >= :date1
                                                                  AND journals.created_on < :date1
                                                                  ) a5 
                                                              left join journal_details on a5.jou_id = journal_details.journal_id   
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
                                          
                                          UNION
                                          
                                          (
                                          select a6.issue_id, a6.issue_created, a6.project_id
                                          from (SELECT a5.*, journal_details.value as new_status_value
                                          from (
                                                  select a4.*, journals.id as jou_id, journals.created_on as last_time 
                                                  from (
                                                      select  a3.issue_id, max(a3.last_time) as max_time, a3.issue_created, a3.project_id
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
                                                            left join journal_details on a2.jou_id = journal_details.journal_id   
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
                                          where journal_id is not null) a6
                                          where a6.last_time = a6.max_time
                                          and a6.new_status_value = 23
                                          )
                                      )a1 
                                      LEFT JOIN custom_values
                                      on a1.issue_id = custom_values.customized_id  
                                      and custom_values.customized_type = "Issue"
                                      GROUP BY a1.issue_id
                                  )a2
                                  LEFT JOIN custom_values
                                  on a2.issue_id = custom_values.customized_id  
                                  and custom_values.customized_type = "Issue"
                                  where a2.sup_type = "S2 - Гарантийное обслуживание"
                                  GROUP BY a2.issue_id, a2.sup_type
                              )a3
                              LEFT JOIN custom_values
                              on a3.related_issue_sd = custom_values.value
                              and custom_values.customized_type = "Issue"
                              where a3.related_issue_sd !=""
                              GROUP BY a3.issue_id, a3.related_issue_sd
                          )a4
                          LEFT JOIN journals 
                          on journals.journalized_id= a4.related_issue_id
                          and journals.created_on < :date2
                          #where a4.related_issue_id != ""
                      )a5 left join journal_details on a5.jou_id = journal_details.journal_id   
                      and journal_details.prop_key = "status_id" 
                      and journal_details.value = 23
                  )a6
              )a7 
              GROUP BY a7.issue_id, a7.issue_created, a7.related_issue_id
          )a8 
          LEFT JOIN custom_values
          on a8.project_id = custom_values.customized_id  
          and custom_values.customized_type = "Project"
      )a9
      GROUP BY a9.issue_id, a9.issue_created, a9.related_issue_id
  )a10
 )a11
 where a11.end_guarantee != ""
 and a11.issue_created > a11.end_guarantee
```