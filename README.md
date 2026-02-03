# VK analysis of cloud gaming

## Everything done in PostgreSQL
 
## Number of unique users

- I used the date format for accuracy and made a grouping
```sql

select
date(session_start) as session_date, 
count(distinct user_id) 
from vk
group by date(session_start)
order by session_date;
```

## Number of sessions and average session time for every platfrom 

- Removed null values, set a constraint so that the end time of the session is always greater than the start time.
- Grouped by platforms, calculated the session values because it is an identifier, and calculated the average session values by converting from varchar to timestamps for correct values

```sql
select platform, round(avg(extract(epoch from session_end::timestamp - session_start::timestamp )/60),1) as average_time, count(session_id) as session_number
from vk
where session_end is not null
and session_start is not null
and session_end::timestamp > session_start::timestamp
group by platform
```

## Percentage of new users

- In the data, each user had null values, and each user could have multiple true values, and they could not be on the very first session. 

- Due to the lack of information, I assumed that the information might not be available immediately. 

- Therefore, all users who had at least one True value were considered new.
```sql

select
    round(
        100.0 * count(distinct case when is_new_user is True then session_id end)
        / count(distinct case when is_new_user is not null then session_id end),
    1) as perofnewal
from vk;
```
## Session duration median for new and old users

- True for new ones and not true for olds users cause i thought null in such data equal to false.
```sql
select
    round(
        percentile_cont(0.5) within group(order by 
            extract(epoch from session_end::timestamp - session_start::timestamp)/60
        )::numeric, 1
    ) as median_session_time
from vk
where is_new_user is not true / True
  and session_start is not null
  and session_end is not null
  and session_end::timestamp > session_start::timestamp;
  ```
## Sum of game time of RU region

- Default rules and sum in timestamps format and answer in hours
```sql
select
    round(sum(extract(epoch from session_end::timestamp - session_start::timestamp))/3600, 1) as total_hours
from vk
where country = 'RU'
  and session_start is not null
  and session_end is not null
  and session_end::timestamp > session_start::timestamp;
```          
