# Healthcare-Data-Project
use healthcare

ALTER TABLE healthcare_dataset 
CHANGE `Name` name VARCHAR(255),
CHANGE `Age` age INT,
CHANGE `Gender` gender VARCHAR(50),
CHANGE `Blood Type` blood_type VARCHAR(10),
CHANGE `Medical Condition` medical_condition TEXT,
CHANGE `Date of admission` date_of_admission DATE,
CHANGE `Doctor` doctor VARCHAR(255),
CHANGE `Hospital` hospital VARCHAR(255),
CHANGE `Insurance Provider` insurance_provider VARCHAR(255),
CHANGE `Billing Amount` billing_amount DECIMAL(10,2),
CHANGE `Room Number` room_number INT,
CHANGE `Admission Type` admission_type VARCHAR(100),
CHANGE `Discharge Date` discharge_date DATE,
CHANGE `Medication` medication TEXT,
CHANGE `Test Results` test_results TEXT;


-- 1. Query to get then number of patients per doctor.
SELECT 
    doctor, COUNT(name) AS NumberofPatients
FROM
    healthcare_dataset
GROUP BY doctor;
-- select * from healthcare_dataset
-- 2. Query to find the average billing amount for each type of admission.
SELECT 
    Admission_Type, ROUND(AVG(Billing_Amount), 0) AS AvgBilling
FROM
    healthcare_dataset
GROUP BY Admission_Type;

-- 3. Query to find the oldest and youngest patient in the dataset.
 SELECT 
    MAX(age) AS oldest, MIN(age) AS youngest
FROM
    healthcare_dataset;
    
-- 4. Query to find the number of patients who stayed in romm number 101.  
SELECT 
    COUNT(*) AS PatientsInRoom101
FROM
    healthcare_dataset
WHERE
    room_number = 101;

-- 5. Query to identify patients with abnormal test results.
SELECT 
    name, test_results
FROM
    healthcare_dataset
WHERE
	test_results = 'abnormal';

-- 6. Query to get a list of patients who have been admitted more than once.
with PatientAdmissions as (select name, count(*) as AdmissionCount
from healthcare_dataset
group by name)
SELECT 
    name, AdmissionCount
FROM
    PatientAdmissions
WHERE
    AdmissionCount > 1;
    
-- 7. Query to calculate the average age of patients by their gender.
SELECT 
    gender, ROUND(AVG(age), 1) AS AverageAge
FROM
    healthcare_dataset
GROUP BY gender;

-- 8. Query to count the number of patients admitted per month (Date of Admission).
SELECT 
    EXTRACT(MONTH FROM date_of_admission) AS Month_JanToDec,
    COUNT(*) AS Total_Admission
FROM
    healthcare_dataset
GROUP BY EXTRACT(MONTH FROM date_of_admission)
ORDER BY EXTRACT(MONTH FROM date_of_admission) ASC;

-- 9. Query to get the patient with the highest billing amount.
SELECT 
    name, MAX(billing_amount) AS MaxBilling
FROM
    healthcare_dataset
GROUP BY name
ORDER BY MaxBilling DESC
LIMIT 5;

10. Query to find the most common medical condition across all patients.
SELECT 
    medical_condition, COUNT(*) AS Occurrence
FROM
    healthcare_dataset
GROUP BY medical_condition
ORDER BY Occurrence DESC;

-- 11. Query to find all patients with Inconclusive test results who have an insurance provider.
SELECT 
    name, insurance_provider
FROM
    healthcare_dataset
WHERE
    test_results = 'Inconclusive'
        AND insurance_provider IS NOT NULL;
        
-- 12. Query to find the total billling amount by hospital.
SELECT 
    hospital, SUM(billing_amount) AS totalBilling
FROM
    healthcare_dataset
GROUP BY hospital;

-- 13.Query to find the average billing per patient based on admission type.
SELECT 
    admission_type,
    ROUND(AVG(billing_amount), 0) AS avg_billing_per_patient
FROM
    healthcare_dataset
GROUP BY admission_type;

-- 14. Query to calculate the time diffrence between admission and discharge(in days).
SELECT 
    name,
    DATEDIFF(discharge_date, date_of_admission) AS duration_of_stay
FROM
    healthcare_dataset;
    
-- 15. Query to identify doctors who have not discharge any patients yet.
select doctor
from healthcare_dataset  
where doctor not in (
select distinct doctor
from healthcare_dataset
where discharge_date is not null
);
-- Select distinct doctors who have not discharged any patients
SELECT DISTINCT
    d.Doctor
FROM
    healthcare_dataset d
        LEFT JOIN
    healthcare_dataset h ON d.Doctor = h.Doctor
        AND h.Discharge_Date IS NOT NULL
WHERE
    h.Doctor IS NULL;
    
-- 16. Query to list patients who have never been assigned to a room number.
SELECT 
    name
FROM
    healthcare_dataset
WHERE
    room_number IS NULL;
    
-- 17. Query to find the total number of patients per insurance provider.    
SELECT 
    insurance_provider, COUNT(*) AS Total_Patients
FROM
    healthcare_dataset
GROUP BY insurance_provider;

-- 18. Query to get the patients admitted within the last 30 days.
SELECT 
    name, date_of_admission
FROM
    healthcare_dataset
WHERE
    date_of_admission >= CURDATE() - INTERVAL 30 DAY;
    
-- 19. Query to calculate the total billing amount per patient over time.
SELECT 
    name, SUM(billing_amount) AS total_billing
FROM
    healthcare_dataset
GROUP BY name;

-- 20. Query to find patients who have been admitted more than once and have abnormal test results.
with Reapeated_Admission as (select name, count(*) as Admission_count
from healthcare_dataset
group by name)
SELECT 
    hd.name, hd.test_results
FROM
    healthcare_dataset AS hd
        JOIN
    Reapeated_Admission AS ra ON hd.name = ra.name
WHERE
    ra.Admission_count > 1
        AND hd.test_results = 'Abnormal';
        
-- Another Approach    
SELECT 
    hd.name, hd.test_results
FROM
    healthcare_dataset hd
WHERE
    hd.test_results = 'Abnormal'
        AND hd.name IN (SELECT 
            name
        FROM
            healthcare_dataset
        GROUP BY name
        HAVING COUNT(name) > 1);
        
-- 21. Query to list the top 5 patients with the highest billing amounts, along with their admission type.
SELECT 
    name, admission_type, billing_amount
FROM
    healthcare_dataset
ORDER BY billing_amount DESC
LIMIT 5;      

-- 22. Query to find doctors who have treated the most number of patients in a given year.
SELECT 
    doctor, COUNT(*) AS patient_count
FROM
    healthcare_dataset
WHERE
    EXTRACT(YEAR FROM date_of_admission) = 2021
GROUP BY doctor
ORDER BY patient_count DESC
LIMIT 3;

-- 23. Query to find the number of patients with abnormal test results, grouped by hospital.
SELECT 
    hospital, COUNT(*) AS Number_of_Patients
FROM
    healthcare_dataset
WHERE
    test_results = 'abnormal'
GROUP BY hospital;

-- 24. Query to identify patients who have not been discharged yet (i.e., null discharge date).
SELECT 
    name, date_of_admission
FROM
    healthcare_dataset
WHERE
    discharge_date is null;
        
-- 25. Query to calculate the percentage of patients with each test result.
SELECT
    test_results,
    COUNT(*) AS patient_count,
    ROUND((COUNT(*) * 100.0) / (SELECT COUNT(*) FROM healthcare_dataset), 2) AS percentage
FROM
    healthcare_dataset
GROUP BY
    test_results;

-- 26. Query to retrieve patients whose medical condition matches a specific keyword (e.g., "Cancer")
SELECT 
    name, medical_condition
FROM
    healthcare_dataset
WHERE
    medical_condition = 'cancer';

-- Another way
SELECT 
    name, medical_condition
FROM
    healthcare_dataset
WHERE
    medical_condition LIKE '%Cancer%';

-- 27. Query to find the rank of doctors based on the number of patients they have treated.
select doctor, count(*) as Number_Of_Patients,
rank() over(order by count(*) desc) as Doctor_Rank
from healthcare_dataset
group by doctor;
    
-- 28. Query to find the total billing amount for each doctor, including the percentage contribution per doctor.
SELECT 
    doctor,
    SUM(billing_amount) AS Total_Billing,
    ROUND((SUM(billing_amount) * 100) / (SELECT 
                    SUM(billing_amount)
                FROM
                    healthcare_dataset),
            2) AS Percentage_Contrubution
FROM
    healthcare_dataset
GROUP BY doctor;

-- Another way
with doctor_billing as (
select doctor, sum(billing_amount) as Total_Billing
from healthcare_dataset
group by doctor
limit 5
)
select doctor, Total_Billing,
round((Total_Billing * 100.0) / sum(Total_Billing) over(), 2) as Billing_Percentage
from doctor_billing;

-- 29. Query to calculate the moving average of the billing amount over the last 3 months.
select name, date_of_admission, billing_amount,
avg(billing_amount) over(partition by name order by date_of_admission rows between 2 preceding and current row) as Moving_Avg_Billing
from healthcare_dataset;
     
-- 30. Query to identify patients who have been admitted, discharged, and readmitted.    
with Patients_Admissions as (select name, date_of_admission, discharge_date,
lead(date_of_admission) over (partition by name order by date_of_admission) as Next_admission_date
from healthcare_dataset)
SELECT 
    name,
    date_of_admission AS initial_admission,
    discharge_date AS initial_discharge,
    Next_admission_date,
    DATEDIFF(Next_admission_date, discharge_date) AS Days_between
FROM
    Patients_Admissions
WHERE
    Next_admission_date IS NOT NULL
        AND DATEDIFF(Next_admission_date, discharge_date) <=30
ORDER BY name , date_of_admission;

-- 31. Query to retrieve a patient’s total billing amount by admission type and their rank among all patients in the same category.
select name, admission_type, sum(billing_amount) as Total_billing,
rank() over(partition by admission_type order by sum(billing_amount) desc) as Rank_in_category
from healthcare_dataset
group by name, admission_type;

-- 32. Query to find the longest length of stay in the hospital for a patient, along with the doctor's name
select hd.name, hd.doctor, datediff(hd.discharge_date, hd.date_of_admission) as length_of_stay
from healthcare_dataset as hd
where datediff(hd.discharge_date, hd.date_of_admission) = (
select max(datediff(discharge_date, date_of_admission))
from healthcare_dataset
);

-- 33. Query to calculate the percentile of billing amounts for each patient.
select name, billing_amount,
percent_rank() over(order by billing_amount desc) as Billing_Percentile
from healthcare_dataset;

-- 34. Query to find all patients with the same medical condition as a specific patient
SELECT hd1.name, hd1.medical_condition
FROM healthcare_dataset hd1
JOIN healthcare_dataset hd2 ON hd1.medical_condition = hd2.medical_condition
WHERE hd2.name = 'John Doe' AND hd1.name != 'John Doe';

-- Another Approach
SELECT 
    name, medical_condition
FROM
    healthcare_dataset
WHERE
    medical_condition = (SELECT 
            medical_condition
        FROM
            healthcare_dataset
        WHERE
            name = 'John Doe')
        AND name != 'John Doe';
     
-- 35. Query to find the patient with the highest total billing amount within each hospital
with Hospital_Billing as (select name, hospital, sum(billing_amount) as Total_Billing
from healthcare_dataset
group by name, hospital)
select hospital, name, Total_Billing from (
rank() over(partition by hospital order by Total_Billing desc) as Billing_Rank
from Hospital_Billing) as Ranked_Billing
where Billing_Rank= 1;        

-- 36. Query to find the average length of stay per medical condition
SELECT 
    medical_condition,
    AVG(DATEDIFF(discharge_date, date_of_admission)) AS avg_length_of_stay
FROM
    healthcare_dataset
GROUP BY medical_condition;

-- 37. Query to find the percentage of patients with each test result by hospital
select hospital, test_results, count(*) as Patient_count,
round((count(*) * 100.0 / sum(count(*)) over (partition by hospital)), 2) as percentage
from healthcare_dataset
group by hospital, test_results
order by hospital, test_results;

-- 38. Query to find patients with abnormal test results and the most common medication prescribed to them.
SELECT 
    hd.name, hd.medication, COUNT(*) AS medication_count
FROM
    healthcare_dataset hd
WHERE
    hd.test_results = 'Abnormal'
GROUP BY hd.name , hd.medication
ORDER BY medication_count DESC
LIMIT 1;

-- Another approach
with Abnormal_Patients as (select name, medication
from healthcare_dataset
where test_results= 'Abnormal'),
Medication_Counts as (
select medication, count(*) as prescription_count
from Abnormal_Patients
group by medication
)
select name, medication 
from Abnormal_Patients
where medication = (
select medication
from Medication_Counts
order by prescription_count
limit 1
);

-- 39. Query to identify patients who have not had a discharge date but have been admitted for more than 90 days
SELECT * FROM
    healthcare_dataset
WHERE
    discharge_date IS NULL
    AND date_of_admission <= CURRENT_DATE - INTERVAL '90' DAY;
    
-- Another approach
SELECT Name, DATEDIFF(CURDATE(), Date_of_Admission) AS LengthOfStay
FROM healthcare_dataset
WHERE Discharge_Date IS NULL
AND DATEDIFF(CURDATE(), Date_of_Admission) > 90;

-- 40. Query to calculate the total billing for each patient and classify them into billing tiers.
select name, sum(billing_amount) as Total_Billing,
case
	when sum(billing_amount) < 1000 then 'low'
    when sum(billing_amount) between 1000 and 5000 then 'medium'
    else 'high'
    end as Billing_Tier
    from healthcare_dataset
    group by name
    order by Total_Billing desc;
    
    -- another way
    WITH PatientBilling AS (
    SELECT
        name,
        SUM(billing_amount) AS total_billing
    FROM
        healthcare_dataset
    GROUP BY
        name
),
BillingTiers AS (
    SELECT
        name,
        total_billing,
        CASE
            WHEN total_billing < 1000 THEN 'Low'
            WHEN total_billing BETWEEN 1000 AND 5000 THEN 'Medium'
            ELSE 'High'
        END AS billing_tier
    FROM
        PatientBilling
)
SELECT
    name,
    total_billing,
    billing_tier
FROM
    BillingTiers
ORDER BY
    total_billing DESC;

    

