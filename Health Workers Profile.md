# Health Workers Profile

## Introduction 
[Profil Tenaga Kesehatan at SATUSEHAT Data ](https://satusehat.kemkes.go.id/data/dashboard/c8b80eb9-07bd-4ac9-82c9-13993a360a34)

This is one of the most important project in my early career as a Data Analyst. The Health Workforce Profile Dashboard provides data on the number and distribution of health workers in Community Health Centers (Puskesmas) and General Regional Hospitals (RSUD) across Indonesia's provinces and cities. This data, sourced from the Healthcare Human Resource Information System (SISDMK) that independently inputted by healthcare facilities and regional health departments, is validated by provincial health departments. Developed for the Directorate General of Health Workforce by the Digital Transformation Office and maintained by Almaash, the dashboard is widely used, averaging 2000 views daily. It is embedded in the [Satu Data Indonesia](http://data.go.id) website http://data.go.id and used by organizations like UNICEF, Bappenas, and the media for health worker data.

## Query
### healthworkers_profile
Master table of health workers data, integrating health workers profile and occupation history
````
WITH
  occupation AS(
  SELECT
    healthworkers_profil_id,
    healthworkers_type_level4_id,
    e.name healthworkers_type_level4,
    e.code code_healthworkers_type_level4,
    healthworkers_type_level3_id,
    c.name healthworkers_type_level3,
    c.code code_healthworkers_type_level3,
    healthworkers_type_level2_id,
    d.name healthworkers_type_level2,
    d.code code_healthworkers_type_level2,
    healthworkers_type_level4_alias,
    status_id,
    b.name status,
    group_status_id,
    f.name group_status,
    healthworkers_type_level1_id,
    CASE
      WHEN healthworkers_type_level1_id = 1 THEN 'Tenaga Kesehatan'
      WHEN healthworkers_type_level1_id = 70 THEN 'Asisten Tenaga Kesehatan'
      WHEN healthworkers_type_level1_id = 78 THEN 'Tenaga Penunjang'
    END healthworkers_type_level1,
    CASE
      WHEN c.name IN ('Dokter', 'Dokter Spesialis', 'Dokter Sub Spesialis / Kompetensi Tambahan Lainnya', 'Dokter Sub Spesialis Dasar',  'Dokter Sub Spesialis / Kompetensi Tambahan Penunjang') AND e.name != 'Dokter Gigi Sub Spesialis Lainnya (yang belum tercantum)' THEN 'DOKTER'
      WHEN c.name IN ('Dokter Gigi', 'Dokter Gigi Spesialis') THEN 'DOKTER GIGI'
      WHEN c.name = 'Ahli Teknologi Laboratorium Medik' AND healthworkers_type_level1_id = 1 THEN 'ATLM'
      WHEN d.name = 'Gizi' AND healthworkers_type_level1_id = 1 THEN 'GIZI'
      WHEN d.name = 'Kefarmasian' AND healthworkers_type_level1_id = 1 THEN 'FARMASI'
      WHEN d.name = 'Keperawatan' AND (healthworkers_type_level1_id = 1 OR healthworkers_type_level1_id IS NULL) THEN 'PERAWAT'
      WHEN d.name = 'Kebidanan' AND healthworkers_type_level1_id = 1 THEN 'BIDAN'
      WHEN d.name = 'Kesehatan Masyarakat' AND healthworkers_type_level1_id = 1 THEN 'KESMAS'
      WHEN d.name = 'Kesehatan Lingkungan' AND healthworkers_type_level1_id = 1 THEN 'KESLING'
      WHEN c.name = 'Keuangan' THEN 'KEUANGAN'
      WHEN c.name = 'Teknologi Informasi' THEN 'TEK. INFORMASI'
      ELSE NULL
    END healthworkers_priority_all,
    CASE
      WHEN c.name = 'Dokter' AND healthworkers_type_level1_id = 1 THEN 'DOKTER' 
      WHEN c.name = 'Dokter Gigi' AND healthworkers_type_level1_id = 1 THEN 'DOKTER GIGI'
      WHEN c.name = 'Ahli Teknologi Laboratorium Medik' AND healthworkers_type_level1_id = 1 THEN 'ATLM'
      WHEN d.name = 'Gizi' AND healthworkers_type_level1_id = 1 THEN 'GIZI'
      WHEN d.name = 'Kefarmasian' AND healthworkers_type_level1_id = 1 THEN 'FARMASI'
      WHEN d.name = 'Keperawatan' AND (healthworkers_type_level1_id = 1 OR healthworkers_type_level1_id IS NULL) THEN 'PERAWAT'
      WHEN d.name = 'Kebidanan' AND healthworkers_type_level1_id = 1 THEN 'BIDAN'
      WHEN c.name IN ('Ilmu Perilaku', 'Promosi Kesehatan') AND healthworkers_type_level1_id = 1 THEN 'KESMAS'
      WHEN c.name = 'Sanitasi Lingkungan' AND healthworkers_type_level1_id = 1 THEN 'KESLING'
      WHEN c.name = 'Keuangan' THEN 'KEUANGAN'
      WHEN c.name = 'Teknologi Informasi' THEN 'TEK. INFORMASI'
      ELSE NULL
    END healthworkers_priority,
    CASE
      WHEN c.name = 'Dokter' THEN 'DOKTER'
      WHEN c.name = 'Dokter Gigi' THEN 'DOKTER GIGI'
      WHEN c.name IN ('Dokter Sub Spesialis / Kompetensi Tambahan Lainnya', 'Dokter Sub Spesialis / Kompetensi Tambahan Penunjang', 'Dokter Sub Spesialis Dasar') THEN 'DOKTER SUB Spesialis'
      WHEN c.name = 'Dokter Spesialis' THEN 'DOKTER Spesialis'
      WHEN c.name = 'Dokter Gigi Spesialis' OR e.name = 'Dokter Gigi Sub Spesialis Lainnya (yang belum tercantum)' THEN 'DOKTER GIGI Spesialis'
    END doctor_profession,
    CASE
      WHEN d.name = 'Medis' THEN 'Tenaga Medis'
      WHEN (healthworkers_type_level1_id IS NULL) AND (d.name = 'Keperawatan') THEN 'Tenaga Kesehatan Lainnya'
      WHEN (healthworkers_type_level1_id IN (70,78) OR (healthworkers_type_level1_id IS NULL)) OR ((d.name IN ('Dukungan Manajemen', 'Struktural', 'Pendidikan dan Pelatihan')) AND (d.name IS NULL)) THEN 'Tenaga Penunjang'
      ELSE 'Tenaga Kesehatan Lainnya'
    END AS healthworkers_type_level4_healthworkers,
    CASE 
      WHEN d.name = 'Medis' THEN 'Tenaga Medis'
      WHEN d.name != 'Medis' THEN 'Tenaga Non Medis'
    END flag_healthworkers,
    health_facilities_id,
    health_facilities_type_code,
    health_facilities_type,
    h.code city_code,
    h.name city,
    g.code province_code,
    g.name province,
    healthworkers_practice_permission,
    start_date_healthworkers_practice_permission,
    end_date_healthworkers_practice_permission,
    working_start_date,
    working_end_date,
    work_status_id,
    CASE
      WHEN work_status_id = 1 THEN 'HABIS KONTRAK'
      WHEN work_status_id = 2 THEN 'AKTIF'
      WHEN work_status_id = 3 THEN 'LAINNYA'
      WHEN work_status_id = 4 THEN 'DIPINDAHTUGASKAN'
      WHEN work_status_id = 5 THEN 'MELANJUTKAN PENDIDIKAN'
      WHEN work_status_id = 6 THEN 'DUPLIKASI'
      WHEN work_status_id = 7 THEN 'PENSIUN'
      WHEN work_status_id = 8 THEN 'MENINGGAL DUNIA'
      WHEN work_status_id IS NULL THEN 'UNDEFINED'
      ELSE 'UNDEFINED'
    END work_status,
  FROM
    healthworkers_occupation a
  LEFT JOIN 
    healthworkers_occupation_status b 
  ON 
    a.status_id = b.id
  LEFT JOIN 
    healthworkers_occupation_status f 
    ON a.group_status_id = f.id
  LEFT JOIN 
    healthworkers_healthworkers_type_level2_healthworkers_type_level4 c 
  ON 
    a.healthworkers_type_level3_id = c.id
  LEFT JOIN 
    healthworkers_healthworkers_type_level2_healthworkers_type_level4 d 
  ON 
    a.healthworkers_type_level2_id = d.id
  LEFT JOIN 
    healthworkers_healthworkers_type_level2_healthworkers_type_level4 e 
  ON 
    a.healthworkers_type_level4_id = e.id
  LEFT JOIN 
    healthworkers_res_partner_nuts g 
  ON 
    a.state_id = g.id
  LEFT JOIN 
    healthworkers_res_partner_nuts h 
  ON 
    a.city_id = h.id 
),


  facilities AS(
  SELECT
    id health_facilities_id,
    UPPER(nama) health_facilities_name,
    health_facilities_owner,
    health_facilities_group_owner,
    health_facilities_code,
    health_facilities_new_code,
    health_facilities_type,
    health_facilities_province,
    health_facilities_city,
    id_healthworkers_type_level4_fasyankes,
    healthworkers_type_level4_fasyankes,
    health_facilities_grade_id,
    health_facilities_grade,
    IF(ABS(lat_coord)>11 OR ABS(lat_coord)=0,NULL,lat_coord) lat_coord,
    IF(long_coord<94 OR long_coord>142,NULL,long_coord) long_coord
  FROM
    healthworkers_facilities 
)

SELECT
  a.id,
  UPPER(a.name) nama,
  no_handphone,
  email,
  nik,
  nik_valid,
  gender,
  birth_date,
  b.code province_code,
  b.name province,
  c.code city_code,
  c.name city,
  ektp_name,
  ektp_healthworkers_type_level4_kelamin,
  str,
  str_start_date,
  f.healthworkers_type_level4,
  f.healthworkers_type_level3,
  f.healthworkers_type_level2,
  f.healthworkers_type_level4_alias,
  f.code_healthworkers_type_level4,
  f.code_healthworkers_type_level3,
  f.code_healthworkers_type_level2, 
  f.status,
  f.group_status,
  f.healthworkers_type_level1,
  f.healthworkers_priority_all,
  f.healthworkers_priority,
  f.doctor_profession,
  f.healthworkers_type_level4_healthworkers,
  f.flag_healthworkers,
  CASE
    WHEN doctor_profession IS NOT NULL AND healthworkers_type_level4_alias = 'drs_og' THEN 'Obsgyn'
    WHEN doctor_profession IS NOT NULL AND healthworkers_type_level4_alias = 'drs_bdh' THEN 'Bedah'
    WHEN doctor_profession IS NOT NULL AND healthworkers_type_level4_alias = 'drs_rad'  THEN 'Radiologi'
    WHEN doctor_profession IS NOT NULL AND healthworkers_type_level4_alias = 'drs_pd'  THEN 'Penyakit_Dalam'
    WHEN doctor_profession IS NOT NULL AND healthworkers_type_level4_alias = 'drs_an'  THEN 'Anastesi'
    WHEN doctor_profession IS NOT NULL AND healthworkers_type_level4_alias = 'drs_pk' THEN 'Patologi_Klinik'
    WHEN doctor_profession IS NOT NULL AND healthworkers_type_level4_alias = 'drs_ank' THEN 'Anak'
  END medical_specialist_type,
  f.healthworkers_practice_permission,
  f.start_date_healthworkers_practice_permission,
  f.end_date_healthworkers_practice_permission,
  f.health_facilities_id,
  f.health_facilities_type tipe_health_facilities_skop,
  f.province_code health_facilities_state_id,
  regexp_replace(UPPER(f.province), 'YOGYAKARTA', 'DAERAH ISTIMEWA YOGYAKARTA') health_facilities_province,
  f.city_code health_facilities_city_id,
  f.city health_facilities_city,
  f.work_status,
  g.*,
  a.write_date healthworkers_updated_at,
FROM
  healthworkers_profil a
LEFT JOIN 
  healthworkers_res_partner_nuts b 
ON 
  a.state_id = b.id
LEFT JOIN 
  healthworkers_res_partner_nuts c 
ON 
  a.city_id = c.id
LEFT JOIN 
  healthworkers_res_partner_nuts d 
ON 
  a.ektp_provinsi = d.id
LEFT JOIN 
  healthworkers_res_partner_nuts e 
ON 
  a.ektp_kabkota = e.id
LEFT JOIN 
  occupation f 
ON 
  a.id = f.healthworkers_profil_id
LEFT JOIN 
  facilities g
ON 
  f.health_facilities_id = g.health_facilities_id
WHERE TRUE 
  AND healthworkers_type_level1 != 'Tenaga Penunjang' OR (healthworkers_type_level2 IS NOT NULL AND healthworkers_type_level2 != 'Dukungan Manajemen')
  AND a.active IS TRUE
````

### Health Workers Priority
Individual level table of 9 health workers priority
````
SELECT
  id,
  str,
  work_status,
  healthworkers_priority,
  group_status,
  health_facilities_id,
  health_facilities_name,
  health_facilities_type,
  health_facilities_city_id,
  health_facilities_city,
  health_facilities_state_id,
  health_facilities_province,
  max(healthworkers_updated_at) last_healthworkers_updated_at,
FROM
  healthworkers_profile
WHERE
  healthworkers_priority IS NOT NULL
  AND healthworkers_priority NOT IN ('TEK. INFORMASI', 'KEUANGAN')
  AND health_facilities_name IS NOT null
GROUP BY 
  1,2,3,4,5,6,7,8,9,10,11,12
````

### Puskesmas
Summary table of health workers distribution in Puskesmas
````
WITH 
  healthworkers_priority AS(
    SELECT 
      healthworkers_priority
    FROM 
      healthworkers_profile
    WHERE 
      healthworkers_priority IS NOT NULL
      AND healthworkers_priority NOT IN ('TEK. INFORMASI', 'KEUANGAN')
    GROUP BY
      1)

  ,initial_data AS(
  SELECT 
    health_facilities_state_id,
    health_facilities_province,
    health_facilities_city_id,
    health_facilities_city,
    health_facilities_id,
    health_facilities_name, 
    b.healthworkers_priority
  FROM 
    healthworkers_profile a,
    healthworkers_priority b
  WHERE 
    health_facilities_type = 'PUSKESMAS'
  QUALIFY ROW_NUMBER() OVER (PARTITION BY health_facilities_id, b.healthworkers_priority) = 1
  ORDER BY 
    1,2,3,4,5,6,7)

SELECT 
  a.health_facilities_state_id,
  a.health_facilities_province,
  a.health_facilities_city_id,
  a.health_facilities_city,
  a.health_facilities_id,
  a.health_facilities_name, 
  a.healthworkers_priority,
  COUNT(DISTINCT (CASE WHEN work_status = 'AKTIF' THEN id END)) total_healthworkers,
FROM 
  initial_data a
LEFT JOIN 
  healthworkers_profile b
ON 
  a.health_facilities_id = b.health_facilities_id
  AND a.healthworkers_priority = b.healthworkers_priority
GROUP BY 
  1,2,3,4,5,6,7
ORDER BY 
  1,2,3,4,5,6,7
````

### Medical Specialist Priority
Individual level table of 7 medical specialist priority
````
SELECT
  id,
  str,
  work_status,
  medical_specialist_type,
  health_facilities_id,
  health_facilities_name,
  health_facilities_grade,
  health_facilities_type,
  health_facilities_owner,
  health_facilities_city_id,
  health_facilities_city,
  health_facilities_state_id,
  health_facilities_province,
FROM
  healthworkers_profile
WHERE
  medical_specialist_type IS NOT NULL
  AND health_facilities_name IS NOT null
````

### RSUD
Summary table of medical specialist priority distribution in RSUD
````
WITH 
  medical_specialist_type AS(
    SELECT 
      medical_specialist_type
    FROM 
      healthworkers_profile
    WHERE 
      medical_specialist_type IS NOT NULL
    GROUP BY
      1)

  ,initial_data AS(
  SELECT 
    health_facilities_state_id,
    health_facilities_province,
    health_facilities_city_id,
    health_facilities_city,
    health_facilities_id,
    health_facilities_name, 
    health_facilities_grade,
    b.medical_specialist_type
  FROM 
    healthworkers_profile a,
    medical_specialist_type b
  WHERE 
    health_facilities_type = 'RUMAH SAKIT' 
    AND health_facilities_owner IN ('PEMERINTAH KABUPATEN', 'PEMERINTAH KOTA', 'PEMERINTAH PROVINSI')
  QUALIFY ROW_NUMBER() OVER (PARTITION BY health_facilities_id, b.medical_specialist_type) = 1
  ORDER BY 
    1,2,3,4,5,6,7,8)

  ,master_table AS(
    SELECT
      health_facilities_id,
      health_facilities_name,
      health_facilities_state_id,
      health_facilities_province,
      health_facilities_city_id,
      health_facilities_city,
      health_facilities_grade,
      medical_specialist_type,
      case
        -- RS Tipe A
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Anak' then '+/-'
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Anastesi' then '5'
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Bedah' then '+/-'
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Obsgyn' then '+/-'
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Patologi Klinik' then '3'
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Penyakit Dalam' then '+/-'
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Radiologi' then '3'
        -- RS Tipe B
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Anak' then '4'
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Anastesi' then '3'
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Bedah' then '4'
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Obsgyn' then '4'
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Patologi Klinik' then '2'
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Penyakit Dalam' then '4'
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Radiologi' then '2'
        -- RS Tipe C
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Anak' then '2'
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Anastesi' then '1'
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Bedah' then '2'
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Obsgyn' then '2'
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Patologi Klinik' then '+/-'
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Penyakit Dalam' then '2'
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Radiologi' then '+/-'
        -- RS Tipe D
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Anak' then '1'
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Anastesi' then '+/-'
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Bedah' then '+/-'
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Obsgyn' then '+/-'
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Patologi Klinik' then '+/-'
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Penyakit Dalam' then '1'
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Radiologi' then '+/-'
        -- RS Tipe Null
        else '+/-'
      end target_medical_specialist_type,
    
      case
        -- RS Tipe A
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Anak' then 1
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Anastesi' then 5
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Bedah' then 1
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Obsgyn' then 1
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Patologi Klinik' then 3
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Penyakit Dalam' then 1
        when health_facilities_grade = 'KELAS A' and medical_specialist_type = 'Radiologi' then 3
        -- RS Tipe B
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Anak' then 4
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Anastesi' then 3
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Bedah' then 4
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Obsgyn' then 4
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Patologi Klinik' then 2
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Penyakit Dalam' then 4
        when health_facilities_grade = 'KELAS B' and medical_specialist_type = 'Radiologi' then 2
        -- RS Tipe C
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Anak' then 2
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Anastesi' then 1
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Bedah' then 2
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Obsgyn' then 2
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Patologi Klinik' then 1
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Penyakit Dalam' then 2
        when health_facilities_grade = 'KELAS C' and medical_specialist_type = 'Radiologi' then 1
        -- RS Tipe D
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Anak' then 1
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Anastesi' then 1
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Bedah' then 1
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Obsgyn' then 1
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Patologi Klinik' then 1
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Penyakit Dalam' then 1
        when health_facilities_grade = 'KELAS D' and medical_specialist_type = 'Radiologi' then 1
      
        -- RS Tipe Null
        else 1
      end target_medical_specialist_type_int
    FROM
      initial_data
    ORDER BY 1 )

SELECT 
  a.health_facilities_state_id,
  a.health_facilities_province,
  a.health_facilities_city_id,
  a.health_facilities_city,
  a.health_facilities_id,
  a.health_facilities_name, 
  a.health_facilities_grade,
  a.medical_specialist_type,
  a.target_medical_specialist_type,
  a.target_medical_specialist_type_int,
  COUNT(DISTINCT(CASE WHEN work_status = 'AKTIF' THEN id END)) total_healthworkers,
FROM 
  master_table a
LEFT JOIN 
  healthworkers_profile b
ON 
  a.health_facilities_id = b.health_facilities_id
  AND a.medical_specialist_type = b.medical_specialist_type
GROUP BY 
  1,2,3,4,5,6,7,8,9,10
ORDER BY 
  1,2,3,4,5,6,7,8,9,10
````
