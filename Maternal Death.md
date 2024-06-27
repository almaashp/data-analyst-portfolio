# Maternal Death

## Introduction 
Maternal mortality is a critical public health issue, reflecting the overall effectiveness of a country's healthcare system. The Maternal Death Analysis Dashboard aims to provide a comprehensive analysis of maternal deaths using data from the national health insurance system. This dashboard leverages extensive health records to identify trends, underlying causes, and demographic factors associated with maternal mortality. By integrating various data sources, including hospital records, insurance claims, and demographic data, the dashboard offers insights into the effectiveness of healthcare interventions and policies.

The primary objectives of the dashboard are:

1. **Trend Analysis:** To visualize the trends in maternal mortality over time, highlighting any significant changes or patterns.
2. **Cause Identification:** To identify the leading causes of maternal deaths, allowing healthcare providers and policymakers to target specific areas for improvement.
3. **Demographic Insights:** To analyze the demographic factors such as age, socioeconomic status, and geographic location, which may influence maternal mortality rates.
4. **Healthcare Access and Quality:** To assess the accessibility and quality of maternal healthcare services across different regions, identifying areas with critical gaps.
5. **Policy Evaluation:** To evaluate the impact of existing healthcare policies and programs on maternal health outcomes, providing data-driven recommendations for future interventions.

By utilizing the national health insurance data, the dashboard aims to support stakeholders, including healthcare providers, policymakers, and researchers, in making informed decisions to reduce maternal mortality and improve maternal health outcomes.

## Query
### Meternal Death Data
Aggregate data of pregnant women, identify their discharge status, condition, and procedure 
````
WITH data_ibu_hamil_meninggal AS(
SELECT
  *
FROM
  national_health_insurance
WHERE
  admission_date_clean <= CURRENT_DATE()
  AND CONTAINS_SUBSTR(diaglist, 'Z34')
  AND discharge_status = 4 )

,data_ibu_hamil AS(
SELECT
  *
FROM
  national_health_insurance
WHERE
  admission_date_clean <= CURRENT_DATE()
  AND CONTAINS_SUBSTR(diaglist, 'Z34'))

,row_numed_ibu_hamil AS(
SELECT 
  *,
  ROW_NUMBER() OVER (PARTITION BY no_kartu ORDER BY discharge_date_clean DESC) rownum
FROM data_ibu_hamil )

,split_diaglist AS (
  SELECT
    sur_key,
    SPLIT(diaglist, ';') AS diaglist_split_values
  FROM
    data_ibu_hamil_meninggal
)

,data_split_diaglist AS(
SELECT
  sur_key,
  diaglists
FROM
  split_diaglist,
  UNNEST(diaglist_split_values) AS diaglists)

,split_proclist AS (
  SELECT
    sur_key,
    SPLIT(proclist, ';') AS proclist_split_values
  FROM
    data_ibu_hamil_meninggal
)

,data_split_proclist AS(
SELECT
  sur_key,
  proclists
FROM
  split_proclist,
  UNNEST(proclist_split_values) AS proclists)

,final AS(
SELECT 
  DISTINCT ihm.sur_key,
  nama_pasien,
  kode_rs,
  msi.nama nama_rs,
  msi.nama_prov,
  msi.nama_kabkota,
  kelas_rs,
  kelas_rawat,
  admission_date,
  admission_date_clean,
  discharge_date,
  discharge_date_clean,
  birth_date,
  birth_date_clean,
  discharge_status,
  sd.diaglists,
  icd.category,
  COALESCE(icd_cat.string_field_0, icd_bi.indonesia_desc) category_bahasa,
  COALESCE(icd_cat.string_field_3, icd_bi_subcat.indonesia_desc) subcategory_bahasa,
  icd.description diaglist_desc,
  icd.indonesian_nm diaglist_desc_bahasa,
  sp.proclists,
  icd9.short_description proclist_short_desc,
  icd9.long_description proclist_long_desc,
  total_tarif,
  umur_tahun,
  case
    when cast(umur_tahun as INT64) <= 15 then '0-15 Tahun'
      when cast(umur_tahun as INT64) <= 25 then '16-25 Tahun'
      when cast(umur_tahun as INT64) <= 35 then '26-35 Tahun'
      when cast(umur_tahun as INT64) <= 45 then '36-45 Tahun'
      when cast(umur_tahun as INT64) <= 55 then '46-55 Tahun'  
      when cast(umur_tahun as INT64) > 55 then '> 55 Tahun'
  end range_umur,
  sep,
  no_kartu,
  if(discharge_status = 4, true, false) ibu_meninggal,
  ihm.silver_updated_at,
FROM 
  data_ibu_hamil_meninggal ihm
LEFT JOIN 
  data_split_diaglist sd
ON 
  ihm.sur_key = sd.sur_key
LEFT JOIN 
  data_split_proclist sp
ON 
  ihm.sur_key = sp.sur_key
LEFT JOIN 
  temp.icd10_complete_dump icd
ON 
  sd.diaglists = icd.code
LEFT JOIN 
  temp.icd9cm_procedure_reference icd9
ON 
  REGEXP_REPLACE(sp.proclists, r'\.', '') = icd9.procedure_code
LEFT JOIN
  temp.faris_icd10_categories icd_cat
ON 
  LEFT(sd.diaglists, 3) = icd_cat.string_field_2
LEFT JOIN 
  (SELECT * FROM temp.icd10_bahasa WHERE subcategory IS NULL) icd_bi
ON 
  LEFT(sd.diaglists, 3) = icd_bi.category
LEFT JOIN 
  (SELECT * FROM temp.icd10_bahasa WHERE subcategory IS NOT NULL) icd_bi_subcat
ON 
  REGEXP_REPLACE(sd.diaglists, r'\.', '') = CONCAT(icd_bi_subcat.category, LEFT(icd_bi_subcat.subcategory ,1))
LEFT JOIN
  (SELECT * FROM master_index.master_sarana_faskes_index WHERE is_valid IS TRUE) msi
ON
  ihm.kode_rs = msi.nomor_id )

,row_numed AS(
SELECT 
  *,
  ROW_NUMBER() OVER (PARTITION BY no_kartu ORDER BY discharge_date_clean DESC) rownum,
  ROW_NUMBER() OVER (PARTITION BY diaglists, no_kartu) rownum_diaglist,
  ROW_NUMBER() OVER (PARTITION BY proclists, no_kartu) rownum_proclist,
FROM final )

SELECT 
  admission_date_clean,
  discharge_date_clean,
  ibu_meninggal,
  kode_rs,
  nama_rs,
  nama_prov,
  nama_kabkota,
  kelas_rs,
  diaglists,
  category_bahasa,
  subcategory_bahasa,
  proclists,
  proclist_short_desc,
  range_umur,
  SUM(DISTINCT total_tarif) total_tarif,
  IF(rownum = 1, count(distinct no_kartu), 0) unique_pasien,
  IF(rownum_diaglist = 1, count(distinct no_kartu), 0) unique_diaglist,
  IF(rownum_proclist = 1, count(distinct no_kartu), 0) unique_proclist,
  COUNT(DISTINCT no_kartu) jumlah_pasien
FROM
  row_numed
GROUP BY 
  discharge_date_clean,
  admission_date_clean,
  ibu_meninggal,
  kode_rs,
  nama_rs,
  nama_prov,
  nama_kabkota,
  kelas_rs,
  diaglists,
  category_bahasa,
  subcategory_bahasa,
  proclists,
  proclist_short_desc,
  range_umur,
  rownum,
  rownum_diaglist,
  rownum_proclist
UNION ALL
SELECT
  admission_date_clean,
  discharge_date_clean,
  null ibu_meninggal,
  null kode_rs,
  null nama_rs,
  null nama_prov,
  null nama_kabkota,
  null kelas_rs,
  null diaglists,
  null category_bahasa,
  null subcategory_bahasa,
  null proclists,
  null proclist_short_desc,
  null range_umur,
  null total_tarif,
  IF(rownum = 1, count(distinct no_kartu), 0) unique_pasien,
  null unique_diaglist,
  null unique_proclist,
  null jumlah_pasien
FROM
  row_numed_ibu_hamil
WHERE
  admission_date_clean <= CURRENT_DATE()
  AND CONTAINS_SUBSTR(diaglist, 'Z34')
GROUP BY 
  admission_date_clean,
  discharge_date_clean,
  rownum
````

### Maternal Death Claim
	
Total claim cost for maternal death case  
````
SELECT
  admission_date_clean,
  discharge_date_clean,
  no_kartu,
  kode_rs,
  msi.nama nama_rs,
  msi.nama_prov,
  msi.nama_kabkota,
  MAX(silver_updated_at) last_update,
  SUM(total_tarif) total_tarif
FROM
  national_health_insurance ihm
LEFT JOIN (
  SELECT
    *
  FROM
    health_facilities
  WHERE
    is_valid IS TRUE) msi
ON
  ihm.kode_rs = msi.nomor_id
WHERE
  admission_date_clean <= CURRENT_DATE()
  AND CONTAINS_SUBSTR(diaglist, 'Z34')
  AND discharge_status = 4
GROUP BY
  1,2,3,4,5,6,7
````
