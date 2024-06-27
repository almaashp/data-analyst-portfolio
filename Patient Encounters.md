# Patient Encounters

## Introduction 
The Patient Encounters Dashboard is designed to provide a comprehensive overview of patient encounters, demographics, and diagnoses, offering healthcare providers a powerful tool for data-driven decision-making. This dashboard is built to consolidate and visualize key patient information, enabling healthcare professionals to gain quick and meaningful insights into patient care patterns, population health trends, and clinical outcomes.

### Background
In modern healthcare, the ability to efficiently access and analyze patient data is crucial for improving care delivery, enhancing patient outcomes, and optimizing operational efficiency. The Patient Encounters Dashboard addresses these needs by integrating data from various sources, such as electronic health records (EHRs), patient management systems, and clinical databases, into a unified, user-friendly interface.

### Key Components
1. **Patient Encounter: **Volume and Trends: Visualizations showing the number of patient visits over time, identifying peak periods and trends.

2. **Demographics: **Age and Gender Distribution: Charts and graphs illustrating the age and gender makeup of the patient population.

3. **Geographic Distribution: **Maps showing patient locations, highlighting regional patterns in healthcare access and utilization.

4. **Occupation Indicators:** Insights into the socioeconomic status of patients through their occupation, which can inform targeted interventions and resource allocation.

5. **Diagnosis: Common Diagnoses:** Lists and charts of the most frequently encountered diagnoses, helping to identify prevalent health issues.

### Objectives
The primary objective of the Patient Encounters Dashboard is to empower healthcare providers with actionable insights that can improve patient care quality, identify and address health disparities, and streamline clinical workflows. By presenting data in an intuitive and interactive manner, the dashboard facilitates timely and informed decision-making, ultimately contributing to better health outcomes and enhanced operational efficiency.

## Query

### Monitoring Encounter Summary
Aggregate data of patient encounters and diagnoses 
````
WITH rownum AS(
  SELECT 
    *,
    ROW_NUMBER() OVER (PARTITION BY subject_patient_id) rownum_patient,
    ROW_NUMBER() OVER (PARTITION BY subject_patient_id, kode_provinsi_msi) rownum_patient_provinsi_faskes,
    ROW_NUMBER() OVER (PARTITION BY subject_patient_id, faskes_id) rownum_patient_faskes,
    ROW_NUMBER() OVER (PARTITION BY subject_patient_id, id_prov_pasien) rownum_patient_provinsi_pasien,
    ROW_NUMBER() OVER (PARTITION BY encounter_id) rownum_encounter,
    ROW_NUMBER() OVER (PARTITION BY encounter_id, faskes_id) rownum_encounter_faskes,

  FROM
    patient_encounter_diagnosis
)

, faskes_terkoneksi AS(
  SELECT 
    organization_id,
  FROM 
    monev_implementasi_satusehat
  WHERE 
    is_terkoneksi = 1
)

SELECT
  faskes_id ihs_organization,
  faskes_name organization_name,
  jenis_sarana_name,
  kode_provinsi_msi,
  nama_provinsi_msi,
  kode_kabkota_msi,
  nama_kabkota_msi,

  id_prov_pasien,
  nama_prov_pasien,
  id_kabkota_pasien,
  nama_kabkota_pasien,
  
  icd_category_code,
  icd_category_ind,
  jenis_kelamin,
  age_group_primer,
  pekerjaan,
  kategori_10_penyakit_prioritas,
  
  COUNT(DISTINCT encounter_id) encounter_count,
  COUNT(DISTINCT subject_patient_id) patient_count,
  COUNT(DISTINCT CASE WHEN rownum_patient = 1 THEN subject_patient_id END) AS countd_patient,
  COUNT(DISTINCT CASE WHEN rownum_patient_provinsi_faskes = 1 THEN subject_patient_id END) AS countd_patient_provinsi_faskes,
  COUNT(DISTINCT CASE WHEN rownum_patient_faskes = 1 THEN subject_patient_id END) AS countd_patient_faskes,
  COUNT(DISTINCT CASE WHEN rownum_patient_provinsi_pasien = 1 THEN subject_patient_id END) AS countd_patient_provinsi_pasien,
  COUNT(DISTINCT CASE WHEN rownum_encounter = 1 THEN encounter_id END) AS countd_encounter,
  COUNT(DISTINCT CASE WHEN rownum_encounter_faskes = 1 THEN encounter_id END) AS countd_encounter_faskes,
  MAX(encounter_last_updated) last_updated,
  MIN(encounter_last_updated) first_updated,
FROM
  rownum rn
RIGHT JOIN 
  faskes_terkoneksi ft
ON 
  LEFT(rn.faskes_id, 9) = LEFT(ft.organization_id, 9) 
GROUP BY 
  1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
````

### Encounter Trend the last 90 days
Summary table to showcase patient encounters trend for the last 90 days 
````
WITH latest_date AS(
SELECT
  MAX(period_start_date) max_date
FROM
  patient_encounter_diagnosis
WHERE 
  period_start_date <= CURRENT_DATE()
)

,rownum AS(
SELECT 
  *,
  ROW_NUMBER() OVER (PARTITION BY subject_patient_id) rownum_patient,
FROM
  patient_encounter_diagnosis
)

SELECT
  kode_provinsi_msi,
  nama_provinsi_msi,
  kode_kabkota_msi,
  nama_kabkota_msi,
  jenis_sarana_name,
  jenis_kelamin,
  age_group_primer,

  period_start_date,
  COUNT(DISTINCT encounter_id) encounter_count,
  COUNT(DISTINCT subject_patient_id) patient_count,
  COUNT(DISTINCT CASE WHEN rownum_patient = 1 THEN subject_patient_id END) AS countd_patient,
FROM
  rownum,
  latest_date
WHERE 
  period_start_date BETWEEN max_date-90 AND max_date
GROUP BY 
  1,2,3,4,5,6,7,8
````
