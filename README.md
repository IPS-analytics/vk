 VK analysis of cloud gaming

Everything done in PostgreSQL
 
Number of unique users

select
date(session_start) as session_date, 
count(distinct user_id) 
from vk
group by date(session_start)
order by session_date;


Number of sessions and average session time for every platfrom 

select platform, round(avg(extract(epoch from session_end::timestamp - session_start::timestamp )/60),1) as average_time, count(session_id) as session_number
from vk
where session_end is not null
and session_start is not null
and session_end::timestamp > session_start::timestamp
group by platform

Percentage of unique users

select
    round(
        100.0 * count(distinct case when is_new_user is True then user_id end)
        / count(distinct user_id),
    1) as perofnewal
from vk;

Session duration median for new and old users

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
  
Sum of game time of RU region

select
    round(sum(extract(epoch from session_end::timestamp - session_start::timestamp))/3600, 1) as total_hours
from vk
where country = 'RU'
  and session_start is not null
  and session_end is not null
  and session_end::timestamp > session_start::timestamp;
 VK analysis of cloud gaming            
