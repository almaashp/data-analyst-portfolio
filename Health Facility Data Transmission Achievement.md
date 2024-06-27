# Health Facility Data Transmission Achievement

## Introduction 
This dashboard is developed to monitor and evaluate the performance of health facilities in transmitting data to the SATU SEHAT platform. SATU SEHAT is a pivotal initiative by the Ministry of Health in Indonesia aimed at digitizing healthcare and centralizing health data for better decision-making and policy implementation. This dashboard provides key insights into the achievements of health facilities in data transmission, highlighting regional disparities and the correlation between these achievements and the total number of health facilities in each region.

The primary objectives of the dashboard are:

1. **Achievement Distribution:** To visualize the distribution of data transmission achievements across various health facilities, identifying high-performing and underperforming facilities.
2. **Correlation Analysis:** To analyze the correlation between data transmission achievements and the total number of health facilities in each region, providing insights into how facility density impacts data transmission performance.
3. **Regional Digitalization Disparities:** To assess the disparities in digitalization efforts across different regions, identifying areas that require additional support and resources to enhance their data transmission capabilities.

## Query
### Weekly Achievement 
Identifying health facilities weekly data transmission achievement 
````
WITH 
  date_range AS(
  WITH dates AS(
  SELECT
    DATE_TRUNC(MIN(encounter_last_updated), WEEK(MONDAY)) min_week,
    DATE_TRUNC(MAX(encounter_last_updated), WEEK(MONDAY)) max_week,
  FROM
    monev_satusehat_organization_encounter_level_hourly
  )

  SELECT
    weeks AS current_weeks,
    DATE_SUB(weeks, INTERVAL 7 DAY) AS previous_weeks,
  FROM 
    dates,
    UNNEST(GENERATE_DATE_ARRAY(min_week, max_week, INTERVAL 1 WEEK)) AS weeks
)

, organizations AS(
  SELECT 
    gabungan ihs_organization,
    MAX(kode_sarana) kode_sarana,
    MAX(organization_name) organization_name,
    MAX(provinsi) provinsi,
    MAX(provinsi_name) provinsi_name,
    MAX(kabkota) kabkota,
    MAX(kabkota_name) kabkota_name,
    MAX(jenis_sarana_name) jenis_sarana_name, 
  FROM 
    monev_implementasi_satusehat_publik
  WHERE 
    is_terkoneksi = 1
  GROUP BY 
    1
)

, update_time AS(
  SELECT 
    organization_id,
    MAX(encounter_last_updated) last_updated
  FROM 
    fact_encounter
  GROUP BY 
    1
)

, data_sent AS(
  SELECT 
    DATE_TRUNC(encounter_last_updated, WEEK(MONDAY)) week_data_sent,
    ihs_organization,
    SUM(count_encounter_id) count_encounter,
    SUM(count_condition_id) count_condition,
    COUNT(DISTINCT encounter_last_updated) total_days_sent,
    MAX(encounter_last_updated) last_updated,
  FROM 
    monev_satusehat_organization_encounter_level_hourly a
  LEFT JOIN
    master_sarana_index_joined b
  ON
    a.ihs_organization = b.code
  WHERE
    code IS NOT NULL
    AND state_sarana IN ('valid',
      'verified')
    AND name IS NOT NULL
    AND jenis_sarana_name IN ('Unit Transfusi Darah',
      'Laboratorium Kesehatan',
      'Apotek',
      'Rumah Sakit',
      'Pusat Kesehatan Masyarakat',
      'Klinik',
      'Tempat Praktik Mandiri Tenaga Kesehatan')
  GROUP BY 
    1,2
)

, placement_table AS(
  SELECT 
    current_weeks,
    previous_weeks,
    ihs_organization,
    kode_sarana,
    organization_name,
    provinsi,
    provinsi_name,
    kabkota,
    kabkota_name,
    jenis_sarana_name, 
  FROM 
    date_range,
    organizations
)

,final AS(
  SELECT 
    current_weeks,
    previous_weeks,
    pt.ihs_organization,
    pt.kode_sarana,
    pt.organization_name,
    pt.provinsi,
    pt.provinsi_name,
    pt.kabkota,
    pt.kabkota_name,
    pt.jenis_sarana_name,
    IFNULL(ds.total_days_sent, 0) total_days_sent,
    IFNULL(ds.count_encounter, 0) count_encounter,
    IFNULL(ds2.count_encounter, 0) AS target_encounter,
    IFNULL(ds.count_condition, 0) count_condition,
    IFNULL(ds2.count_condition, 0) AS target_condition,
    IFNULL(CASE 
      WHEN ROUND(SAFE_DIVIDE(ds.count_encounter, ds2.count_encounter),2) > 1 THEN 1
      ELSE ROUND(SAFE_DIVIDE(ds.count_encounter, ds2.count_encounter),2)
    END, 0) AS pct_encounter_sent,
    IFNULL(CASE 
      WHEN ROUND(SAFE_DIVIDE(ds.count_condition, ds2.count_condition),2) > 1 THEN 1
      ELSE ROUND(SAFE_DIVIDE(ds.count_condition, ds2.count_condition),2)
    END, 0) AS pct_condition_sent,
    ut.last_updated
  FROM 
    placement_table pt
  LEFT JOIN
    data_sent ds
  ON 
    pt.ihs_organization = ds.ihs_organization
    AND pt.current_weeks = ds.week_data_sent
  LEFT JOIN 
    data_sent ds2
  ON 
    pt.ihs_organization = ds2.ihs_organization
    AND pt.previous_weeks = ds2.week_data_sent
  LEFT JOIN 
    update_time ut
  ON 
    LEFT(pt.ihs_organization, 9) = LEFT(ut.organization_id, 9)
  WHERE 
    ds.total_days_sent IS NOT NULL
  GROUP BY 
    1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18
)

SELECT 
  current_weeks,
  previous_weeks,
  ihs_organization,
  kode_sarana,
  organization_name,
  provinsi,
  provinsi_name,
  kabkota,
  kabkota_name,
  jenis_sarana_name,
  total_days_sent,
  count_encounter,
  target_encounter,
  count_condition,
  target_condition,
  pct_encounter_sent,
  pct_condition_sent,
  ROUND(((pct_encounter_sent+pct_condition_sent)/2),2) capaian_pengiriman_data,
  last_updated
FROM 
  final
````

### Health Facility Classification 
Showcasing health facilities classification based on achievements for the last 4 weeks dan resources utilization
````
WITH initial_table AS(
SELECT
  ihs_organization, 
  current_weeks minggu_capaian,
  kode_sarana, 
  organization_name,
  jenis_sarana_name, 
  provinsi, 
  provinsi_name, 
  kabkota, 
  kabkota_name,
  current_weeks AS w_0,
  current_weeks - 7 AS w_1,
  current_weeks - 14 AS w_2,
  current_weeks - 21 AS w_3,
  capaian_pengiriman_data capaian_mingguan,
FROM
  detail_healthfacilities_classification_weekly
)

, capaian AS(
SELECT 
  ihs_organization,
  minggu_capaian,
  capaian_mingguan,
FROM 
  initial_table
GROUP BY 
  1,2,3
)

, rerata_capaian_fasyankes AS(
SELECT 
  it.ihs_organization, 
  it.minggu_capaian,
  it.kode_sarana,
  it.organization_name,
  it.jenis_sarana_name, 
  it.provinsi, 
  it.provinsi_name, 
  it.kabkota, 
  it.kabkota_name,
  COALESCE(cp0.capaian_mingguan, 0) AS capaian_w0,
  COALESCE(cp1.capaian_mingguan, 0) AS capaian_w1,
  COALESCE(cp2.capaian_mingguan, 0) AS capaian_w2,
  COALESCE(cp3.capaian_mingguan, 0) AS capaian_w3,
  COALESCE((cp0.capaian_mingguan + cp1.capaian_mingguan + cp2.capaian_mingguan + cp3.capaian_mingguan)/4, 0) AS rerata_capaian
FROM 
  initial_table it
LEFT JOIN 
  capaian cp0
ON 
  it.ihs_organization = cp0.ihs_organization
  AND it.w_0 = cp0.minggu_capaian
LEFT JOIN 
  capaian cp1
ON 
  it.ihs_organization = cp1.ihs_organization
  AND it.w_1 = cp1.minggu_capaian
LEFT JOIN 
  capaian cp2
ON 
  it.ihs_organization = cp2.ihs_organization
  AND it.w_2 = cp2.minggu_capaian
LEFT JOIN 
  capaian cp3
ON 
  it.ihs_organization = cp3.ihs_organization
  AND it.w_3 = cp3.minggu_capaian
)

, capaian_resources AS(
  WITH initial_data AS(
  SELECT
    encounter_last_updated_latest_time AS encounter_last_updated,
    ihs_organization,
    nama_fasyankes,
    resources,
    MAX(status) status,
  FROM
    monev_satusehat
  UNPIVOT(
    status
      FOR
        resources IN (
          encounter_id,
          condition_id,
          composition_id,
          procedure_id,
          immunization_id,
          medication_dispense_id,
          medication_request_id,
          observation_id,
          service_request_id,
          specimen_id,
          diagnostic_report_id,
          allergy_intolerance_id,
          clinical_impression_id,
          imaging_study_id,
          care_plan_id,
          medication_statement_id,
          questionnaire_response_id
    ))
  WHERE 
    ihs_organization IS NOT NULL
  GROUP BY 
    1,2,3,4
  )

  , final AS(
  SELECT 
    encounter_last_updated,
    ihs_organization,
    nama_fasyankes,
    CASE 
      WHEN resources = 'encounter_id' THEN 'Encounter'
      WHEN resources = 'condition_id' THEN 'Condition'
      ELSE 'Penunjang'
    END AS resources,
    SUM(status) status,
  FROM 
    initial_data
  GROUP BY 
    1,2,3,4)

SELECT 
  a.ihs_organization,
  MAX(a.nama_fasyankes) nama_fasyankes,
  MAX(IF(a.resources = 'Encounter' AND a.status = 1, 1, 0)) encounter,
  MAX(IF(b.resources = 'Condition' AND b.status = 1, 1, 0)) condition,
  MAX(IF(c.resources = 'Penunjang', c.status, 0)) penunjang,
FROM 
  final a 
LEFT JOIN 
  final b
ON 
  a.ihs_organization = b.ihs_organization 
LEFT JOIN 
  final c 
ON 
  a.ihs_organization = c.ihs_organization 
GROUP BY 
  1  
)

SELECT 
  a.gabungan,
  a.organization_name, 
  a.kode_sarana, 
  a.provinsi, 
  a.provinsi_name, 
  a.kabkota, 
  a.kabkota_name,
  a.jenis_sarana_name,
  IFNULL(a.is_terdaftar, 0) is_terdaftar,
  IFNULL(IF(a.is_terverifikasi = 0, a.is_terkoneksi, a.is_terverifikasi), 0) AS is_terintegrasi,
  IFNULL(a.is_terkoneksi, 0) is_terkoneksi,
  IFNULL(c.encounter+c.condition,2) resource_dasar,
  IFNULL(c.penunjang, 0) resource_penunjang,
  IFNULL(capaian_w0, 0) capaian_w0, 
  IFNULL(capaian_w1, 0) capaian_w1, 
  IFNULL(capaian_w2, 0) capaian_w2, 
  IFNULL(capaian_w3, 0) capaian_w3, 
  IFNULL(rerata_capaian, 0) rerata_capaian,
  CASE
    WHEN is_terdaftar IS NULL THEN '6. Belum menggunakan RME dan terdaftar ke SATUSEHAT'
    WHEN (is_terdaftar = 1 OR IF(a.is_terverifikasi = 0, a.is_terkoneksi, a.is_terverifikasi) = 1) AND (is_terkoneksi = 0) THEN '5. Sudah terdaftar namun belum mengirimkan data ke SATUSEHAT'
    WHEN rerata_capaian <= 0.5 AND (c.encounter = 1 OR c.condition = 1) AND penunjang = 0 THEN '4. Capaian pengiriman kunjungan dan kondisi <= 50% dan jumlah resource dasar saja'
    WHEN rerata_capaian  <= 0.5 AND (c.encounter = 1 OR c.condition = 1) AND penunjang >= 1 THEN '3. Capaian pengiriman kunjungan dan kondisi <= 50% dan jumlah resource dasar + resource penunjang'
    WHEN rerata_capaian  >= 0.5 AND (c.encounter = 1 OR c.condition = 1) AND penunjang = 0 THEN '2. Capaian pengiriman kunjungan dan kondisi > 50% dan jumlah resource dasar saja'
    WHEN rerata_capaian  >= 0.5 AND (c.encounter = 1 OR c.condition = 1) AND penunjang >= 1 THEN '1. Capaian pengiriman kunjungan dan kondisi > 50% dan jumlah resource dasar + resource penunjang'
    ELSE '0. Belum Terklasifikasi'
  END AS klasifikasi
FROM 
  monev_implementasi_satusehat_publik a
LEFT JOIN 
  rerata_capaian_fasyankes b 
ON 
  a.gabungan = b.ihs_organization
LEFT JOIN 
  capaian_resources c
ON 
  a.gabungan = c.ihs_organization
GROUP BY 
  1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19
````
