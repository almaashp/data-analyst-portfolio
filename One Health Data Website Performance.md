# One Health Data Website Performance

## Introduction

The One Health Data Platform Monitoring and Analysis Dashboard is designed to oversee and assess the performance and utilization of the One Health Data platform, an initiative by the Ministry of Health to centralize and streamline data management across the healthcare sector. This platform consolidates various dashboards and datasets created within the Ministry, promoting efficient data sharing, accessibility, and integration. The One Health Data platform aims to enhance data-driven decision-making processes, ensuring that accurate and timely information is available to support healthcare policies and interventions.

The dashboard serves multiple purposes:

1. **Platform Utilization Monitoring:** Tracks the usage of the One Health Data platform, including the number of active users, frequency of access, and engagement with different datasets and dashboards.

2. **Performance Metrics:** Measures key performance indicators (KPIs) such as dashboard utilization and account approval latency.

3. **User Feedback and Satisfaction:** Collects and analyzes user feedback to identify areas for improvement and enhance user satisfaction.

4. **Impact Analysis:** Assesses the impact of the One Health Data platform on healthcare decision-making and policy implementation, highlighting success stories and areas for further development.

## Query

### Summary Dashboard
Baseline of dashboard information such as total visit and shared per dashboard, dashboard type, etc.
````
WITH dashboard AS(
SELECT 
  *
FROM 
  dashboard
WHERE TRUE
  AND is_deleted IS FALSE )

SELECT
  DISTINCT a.id,
  a.title,
  b.description,
  a.is_public,
  a.shared_count,
  a.visitor_count,
  a.vendor,
  a.level,
  c.message report_message,
  c.created_at report_created_at,
  a.created_at,
  a.updated_at,
  a.execution_timestamp
FROM
  dashboard a
LEFT JOIN 
  dashboard_item b
ON 
  a.id = b.dashboard_id
LEFT JOIN 
  dashboard_report c
ON 
  a.id = c.dashboard_id
ORDER BY 
  visitor_count DESC
````

### Summary User
Baseline of user information such as user type, user registration process, etc.  
````
WITH user AS(
SELECT
  *
FROM
  user
WHERE 
  is_deleted IS FALSE)

,final as(
SELECT
  a.id,
  a.name,
  a.created_at user_created_at,
  a.email,
  a.verification_status, 
  case 
    when verification_status = 'DITERIMA' then 3
    when verification_status = 'MENUNGGU' then 2
    when verification_status = 'DITOLAK' then 1
  end as rank_verification,
  a.work_unit_id,
  b.name work_unit,
  b.scope,
  b.level,
  c.code province_code,
  c.name province_name,
  regexp_replace(region_code, '[^0-9]', '') region_code,
  g.name region_name,
  g.level region_level,
  e.type role_user,
  COUNT(DISTINCT f.id) count_login,
  MAX(login_at) last_login_at,
  MIN(login_at) first_login_at,
FROM
  user a
FULL OUTER JOIN
  work_unit b
ON 
  a.work_unit_id = b.id 
LEFT JOIN 
  region c
ON
  LEFT(b.region_code, 2) = c.code
LEFT JOIN 
  region g
ON
  b.region_code = g.code
LEFT JOIN 
  role_on_users d
ON 
  a.id = d.user_id
LEFT JOIN 
  role e
ON 
  d.role_id = e.id
LEFT JOIN 
  login_record f
ON 
  a.id = f.user_id
GROUP BY 
  1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16)

select 
  *,
  case 
    when rank_verification = 3 and last_login_at is not null then '5'
    when rank_verification = 3 and last_login_at is null then '4'
    when rank_verification = 2 then '3'
    when rank_verification = 1 then '2'
    when rank_verification is null then '1'
  end as flag
from 
  final
````

### Funnel
Summary of user funnel to create sankey diagram in Tableau
````
WITH user AS(
SELECT
  *
FROM
  user
WHERE 
  is_deleted IS FALSE)

SELECT
  CASE 
    WHEN c.level = 'KABUPATEN_KOTA' THEN 'Dinkes Kabupaten/Kota'
    WHEN c.level = 'PROVINSI' THEN 'Dinkes Provinsi'
    WHEN scope = 'PUSAT' AND NOT b.name IN ('Menteri Kesehatan', 'Wakil Menteri Kesehatan') THEN 'Satker Kemenkes' 
  END as group_user,
  'Target Users' step_1,
  if (verification_status IS NOT NULL, 'Registered', 'Not Yet Registered') step_2,
  case 
    when verification_status = 'DITERIMA' then 'Approved'
    when verification_status = 'DITOLAK' then 'Rejected'
    when verification_status = 'MENUNGGU' then 'Waiting for Approval'
  end as step_3,
  case 
    when verification_status = 'DITERIMA' and f.user_id IS NOT NULL  then 'Activated'
    when verification_status = 'DITERIMA' and f.user_id IS NULL then 'Not Yet Activated'
  end as step_4,
  count(distinct b.name) count_work_unit
FROM
  work_unit b
LEFT JOIN
  user a
ON 
  a.work_unit_id = b.id 
LEFT JOIN 
  region c
ON
  b.region_code = c.code
LEFT JOIN 
  role_on_users d
ON 
  a.id = d.user_id
LEFT JOIN 
  role e
ON 
  d.role_id = e.id
LEFT JOIN 
  login_record f
ON 
  a.id = f.user_id
WHERE 
  scope IN ('DAERAH', 'PUSAT')
group by 
  1,2,3,4,5
ORDER BY 
  1,2,3,4,5
````
