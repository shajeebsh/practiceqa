---
title: "code_sql_python"
date: 2026-04-25 10:58:43 
author: shajeebhameed
layout: page
slug: code_sql_python__trashed
status: trash
---

## Synthetic data setup

### Python — generate and load
    
    
    import pandas as pd
    import numpy as np
    from scipy import stats as scipy_stats
    import warnings
    warnings.filterwarnings('ignore')
    
    np.random.seed(42)
    
    # ── Sample records (hardcoded — no file needed) ──────────────
    claims_records = [
        # claim_id, claimant_id, policy_id, claim_type, claim_status,
        # adjuster_id, region, provider_id, claim_amount, reserve_amount,
        # paid_amount, fnol_date, loss_date, closure_date,
        # fraud_flag, num_prev_claims, premium
        ('CLM0001','CUST039','POL007','Liability','Reopened','ADJ004','Galway','PRV002',894.39,3035.46,2600.25,'2023-11-10','2022-11-25','2024-11-02',0,1,4585.54),
        ('CLM0002','CUST052','POL022','Liability','Closed','ADJ003','Cork','PRV004',17907.33,1834.33,4900.81,'2026-08-15','2023-08-01','2023-04-03',0,3,4746.66),  # future fnol
        ('CLM0003','CUST029','POL028','Health','Closed','ADJ004','Limerick',None,6798.11,2977.12,1238.09,'2023-06-01','2024-05-09','2022-01-01',1,2,4820.61),      # closure before fnol
        ('CLM0004','CUST015','POL002','Motor','Closed','ADJ001','Cork','PRV004',14335.58,5579.83,341.31,'2023-01-19','2023-04-03','2023-10-21',0,5,2846.57),
        ('CLM0005','CUST008','POL031','Motor','Closed','ADJ002','Dublin','PRV001',3201.45,2100.22,1800.10,'2023-03-15','2023-02-01','2023-09-12',0,0,1200.00),
        ('CLM0006','CUST017','POL014','Property','Open','ADJ005','Waterford','PRV003',8900.00,9500.00,0.00,'2024-01-20','2023-12-30',None,0,2,3200.00),
        ('CLM0007','CUST041','POL035','Health','Pending','ADJ001','Galway',None,2200.75,2500.00,500.00,'2023-07-08','2023-06-15',None,0,0,890.00),
        ('CLM0008','CUST022','POL018','Motor','Closed','ADJ003','Dublin','PRV002',52341.90,55000.00,48000.00,'2023-05-22','2023-05-20','2024-02-14',1,3,4100.00),
        ('CLM0009','CUST005','POL009','Travel','Open','ADJ002','Cork','PRV004',1100.50,1200.00,0.00,'2024-06-01','2024-05-28',None,0,1,450.00),
        ('CLM0010','CUST033','POL003','Liability','Closed','ADJ004','Limerick','PRV001',29500.00,30000.00,27000.00,'2023-02-14','2023-01-10','2024-01-30',0,4,4800.00),
        ('CLM0011','CUST011','POL025','Motor','Open','ADJ001','Dublin','PRV003',7800.20,8200.00,0.00,'2024-03-10','2024-03-08',None,1,2,2300.00),
        ('CLM0012','CUST047','POL011','Property','Closed','ADJ005','Cork',None,15600.00,16000.00,14800.00,'2023-04-05','2023-03-20','2023-12-15',0,1,3800.00),
        ('CLM0013','CUST019','POL040','Health','Pending','ADJ002','Galway','PRV002',3400.00,3600.00,800.00,'2023-09-18','2023-09-10',None,0,0,1100.00),
        ('CLM0014','CUST055','POL016','Motor','Closed','ADJ003','Waterford','PRV001',9200.50,9800.00,8700.00,'2023-08-22','2023-08-20','2024-04-10',0,2,2700.00),
        ('CLM0015','CUST028','POL007','Liability','Open','ADJ004','Dublin',None,44000.00,46000.00,0.00,'2024-01-05','2023-12-01',None,1,5,4900.00),
        ('CLM0016','CUST039','POL033','Property','Closed','ADJ001','Cork','PRV004',6700.00,7000.00,6200.00,'2023-06-14','2023-06-10','2024-01-20',0,1,2100.00),
        ('CLM0017','CUST003','POL019','Motor','Open','ADJ002','Galway','PRV003',3900.80,4100.00,0.00,'2024-05-03','2024-05-01',None,0,0,1500.00),
        ('CLM0018','CUST044','POL026','Health','Closed','ADJ005','Limerick','PRV001',5500.00,5800.00,5100.00,'2023-10-29','2023-10-25','2024-06-15',0,3,2200.00),
        ('CLM0019','CUST022','POL018','Motor','Open','ADJ003','Dublin','PRV002',11200.00,12000.00,0.00,'2024-02-17','2024-02-15',None,1,4,3900.00),
        ('CLM0020','CUST037','POL004','Property','Closed','ADJ005','Waterford',None,22300.00,23000.00,21000.00,'2023-03-28','2023-03-25','2023-11-10',0,2,4500.00),
        ('CLM0021','CUST009','POL037','Liability','Pending','ADJ004','Cork','PRV004',18500.00,19000.00,3000.00,'2023-12-01','2023-11-20',None,0,1,4700.00),
        ('CLM0022','CUST060','POL012','Motor','Closed','ADJ002','Dublin','PRV001',4200.00,4400.00,3900.00,'2023-07-16','2023-07-14','2024-03-22',0,0,1800.00),
        ('CLM0023','CUST015','POL002','Motor','Open','ADJ003','Cork','PRV003',8100.00,8500.00,0.00,'2024-04-12','2024-04-10',None,0,6,2700.00),
        ('CLM0024','CUST031','POL021','Health','Closed','ADJ005','Galway',None,2800.50,3000.00,2600.00,'2023-05-09','2023-05-05','2023-12-01',0,1,950.00),
        ('CLM0025','CUST048','POL038','Property','Open','ADJ001','Limerick','PRV002',35000.00,37000.00,0.00,'2024-01-30','2024-01-28',None,1,3,4600.00),
        ('CLM0026','CUST006','POL015','Travel','Closed','ADJ004','Dublin','PRV004',900.25,950.00,850.00,'2023-08-05','2023-08-03','2023-10-15',0,0,500.00),
        ('CLM0027','CUST052','POL022','Liability','Open','ADJ002','Cork',None,55000.00,58000.00,0.00,'2024-03-25','2024-03-20',None,1,4,4900.00),
        ('CLM0028','CUST024','POL029','Motor','Closed','ADJ003','Waterford','PRV001',6300.00,6600.00,5900.00,'2023-09-11','2023-09-09','2024-05-08',0,2,2400.00),
        ('CLM0029','CUST013','POL006','Health','Pending','ADJ005','Dublin','PRV003',4100.00,4300.00,900.00,'2023-11-14','2023-11-10',None,0,0,1300.00),
        ('CLM0030','CUST041','POL035','Liability','Closed','ADJ001','Galway','PRV002',31200.00,32000.00,29500.00,'2023-04-20','2023-04-18','2024-02-28',0,2,4800.00),
        # Injected outliers
        ('CLM0041','CUST010','POL008','Motor','Open','ADJ003','Dublin','PRV002',250000.00,260000.00,0.00,'2024-02-10','2024-02-08',None,1,7,4800.00),
        ('CLM0042','CUST057','POL036','Property','Closed','ADJ004','Cork','PRV004',380000.00,390000.00,350000.00,'2023-07-03','2023-06-30','2024-06-20',0,1,4900.00),
        # Injected negative amounts
        ('CLM0043','CUST021','POL020','Health','Open','ADJ002','Galway',None,-500.00,2000.00,0.00,'2024-03-15','2024-03-14',None,0,0,800.00),
        ('CLM0044','CUST046','POL032','Motor','Pending','ADJ005','Limerick','PRV001',-500.00,3500.00,0.00,'2023-09-22','2023-09-20',None,0,1,1700.00),
        # Null claim_amount
        ('CLM0035','CUST002','POL001','Liability','Open','ADJ001','Cork',None,None,42000.00,0.00,'2024-02-28','2024-02-25',None,0,2,4700.00),
        ('CLM0065','CUST055','POL016','Liability','Open','ADJ001','Cork','PRV003',None,35000.00,0.00,'2024-04-18','2024-04-16',None,0,2,4700.00),
    ]
    
    cols = ['claim_id','claimant_id','policy_id','claim_type','claim_status',
            'adjuster_id','region','provider_id','claim_amount','reserve_amount',
            'paid_amount','fnol_date','loss_date','closure_date',
            'fraud_flag','num_prev_claims','premium']
    
    df = pd.DataFrame(claims_records, columns=cols)
    df['fnol_date']    = pd.to_datetime(df['fnol_date'])
    df['loss_date']    = pd.to_datetime(df['loss_date'])
    df['closure_date'] = pd.to_datetime(df['closure_date'], errors='coerce')
    
    # Add 3 duplicate rows
    df = pd.concat([df, df.iloc[[3, 7, 15]]], ignore_index=True)
    
    # Reserve history
    reserve_records = [
        ('CLM0004','2023-02-01',5200.00),('CLM0004','2023-06-20',5600.00),('CLM0004','2023-09-15',5400.00),
        ('CLM0008','2023-06-01',52000.00),('CLM0008','2023-09-10',54000.00),('CLM0008','2024-01-05',55000.00),
        ('CLM0010','2023-03-01',28000.00),('CLM0010','2023-08-15',30500.00),('CLM0010','2024-01-20',30000.00),
        ('CLM0012','2023-05-01',15000.00),('CLM0012','2023-08-20',16200.00),('CLM0012','2023-11-10',16000.00),
        ('CLM0028','2023-10-01',6200.00),('CLM0028','2024-01-10',6500.00),('CLM0028','2024-04-15',6600.00),
        ('CLM0030','2023-05-01',30000.00),('CLM0030','2023-09-20',31500.00),('CLM0030','2024-02-10',32000.00),
        ('CLM0042','2023-08-01',370000.00),('CLM0042','2023-11-20',385000.00),('CLM0042','2024-05-10',390000.00),
    ]
    rh = pd.DataFrame(reserve_records, columns=['claim_id','update_date','reserve_amount'])
    rh['update_date'] = pd.to_datetime(rh['update_date'])
    
    print(f"Claims: {len(df)} rows | Reserve history: {len(rh)} rows")
    print(df[['claim_id','claim_type','claim_status','claim_amount','fnol_date']].head(5).to_string())
    
    

### SQL — create tables and insert data
    
    
    -- Create tables
    CREATE TABLE claims (
        claim_id         VARCHAR(10),
        claimant_id      VARCHAR(10),
        policy_id        VARCHAR(10),
        claim_type       VARCHAR(20),
        claim_status     VARCHAR(20),
        adjuster_id      VARCHAR(10),
        region           VARCHAR(20),
        provider_id      VARCHAR(10),
        claim_amount     DECIMAL(12,2),
        reserve_amount   DECIMAL(12,2),
        paid_amount      DECIMAL(12,2),
        fnol_date        DATE,
        loss_date        DATE,
        closure_date     DATE,
        fraud_flag       INT,
        num_prev_claims  INT,
        premium          DECIMAL(12,2)
    );
    
    CREATE TABLE reserve_history (
        claim_id       VARCHAR(10),
        update_date    DATE,
        reserve_amount DECIMAL(12,2)
    );
    
    -- Insert claims (same data as Python above)
    INSERT INTO claims VALUES
    ('CLM0001','CUST039','POL007','Liability','Reopened','ADJ004','Galway','PRV002',894.39,3035.46,2600.25,'2023-11-10','2022-11-25','2024-11-02',0,1,4585.54),
    ('CLM0002','CUST052','POL022','Liability','Closed','ADJ003','Cork','PRV004',17907.33,1834.33,4900.81,'2026-08-15','2023-08-01','2023-04-03',0,3,4746.66),
    ('CLM0003','CUST029','POL028','Health','Closed','ADJ004','Limerick',NULL,6798.11,2977.12,1238.09,'2023-06-01','2024-05-09','2022-01-01',1,2,4820.61),
    ('CLM0004','CUST015','POL002','Motor','Closed','ADJ001','Cork','PRV004',14335.58,5579.83,341.31,'2023-01-19','2023-04-03','2023-10-21',0,5,2846.57),
    ('CLM0005','CUST008','POL031','Motor','Closed','ADJ002','Dublin','PRV001',3201.45,2100.22,1800.10,'2023-03-15','2023-02-01','2023-09-12',0,0,1200.00),
    ('CLM0006','CUST017','POL014','Property','Open','ADJ005','Waterford','PRV003',8900.00,9500.00,0.00,'2024-01-20','2023-12-30',NULL,0,2,3200.00),
    ('CLM0007','CUST041','POL035','Health','Pending','ADJ001','Galway',NULL,2200.75,2500.00,500.00,'2023-07-08','2023-06-15',NULL,0,0,890.00),
    ('CLM0008','CUST022','POL018','Motor','Closed','ADJ003','Dublin','PRV002',52341.90,55000.00,48000.00,'2023-05-22','2023-05-20','2024-02-14',1,3,4100.00),
    ('CLM0009','CUST005','POL009','Travel','Open','ADJ002','Cork','PRV004',1100.50,1200.00,0.00,'2024-06-01','2024-05-28',NULL,0,1,450.00),
    ('CLM0010','CUST033','POL003','Liability','Closed','ADJ004','Limerick','PRV001',29500.00,30000.00,27000.00,'2023-02-14','2023-01-10','2024-01-30',0,4,4800.00),
    ('CLM0011','CUST011','POL025','Motor','Open','ADJ001','Dublin','PRV003',7800.20,8200.00,0.00,'2024-03-10','2024-03-08',NULL,1,2,2300.00),
    ('CLM0012','CUST047','POL011','Property','Closed','ADJ005','Cork',NULL,15600.00,16000.00,14800.00,'2023-04-05','2023-03-20','2023-12-15',0,1,3800.00),
    ('CLM0013','CUST019','POL040','Health','Pending','ADJ002','Galway','PRV002',3400.00,3600.00,800.00,'2023-09-18','2023-09-10',NULL,0,0,1100.00),
    ('CLM0014','CUST055','POL016','Motor','Closed','ADJ003','Waterford','PRV001',9200.50,9800.00,8700.00,'2023-08-22','2023-08-20','2024-04-10',0,2,2700.00),
    ('CLM0015','CUST028','POL007','Liability','Open','ADJ004','Dublin',NULL,44000.00,46000.00,0.00,'2024-01-05','2023-12-01',NULL,1,5,4900.00),
    ('CLM0016','CUST039','POL033','Property','Closed','ADJ001','Cork','PRV004',6700.00,7000.00,6200.00,'2023-06-14','2023-06-10','2024-01-20',0,1,2100.00),
    ('CLM0017','CUST003','POL019','Motor','Open','ADJ002','Galway','PRV003',3900.80,4100.00,0.00,'2024-05-03','2024-05-01',NULL,0,0,1500.00),
    ('CLM0018','CUST044','POL026','Health','Closed','ADJ005','Limerick','PRV001',5500.00,5800.00,5100.00,'2023-10-29','2023-10-25','2024-06-15',0,3,2200.00),
    ('CLM0019','CUST022','POL018','Motor','Open','ADJ003','Dublin','PRV002',11200.00,12000.00,0.00,'2024-02-17','2024-02-15',NULL,1,4,3900.00),
    ('CLM0020','CUST037','POL004','Property','Closed','ADJ005','Waterford',NULL,22300.00,23000.00,21000.00,'2023-03-28','2023-03-25','2023-11-10',0,2,4500.00),
    ('CLM0021','CUST009','POL037','Liability','Pending','ADJ004','Cork','PRV004',18500.00,19000.00,3000.00,'2023-12-01','2023-11-20',NULL,0,1,4700.00),
    ('CLM0022','CUST060','POL012','Motor','Closed','ADJ002','Dublin','PRV001',4200.00,4400.00,3900.00,'2023-07-16','2023-07-14','2024-03-22',0,0,1800.00),
    ('CLM0023','CUST015','POL002','Motor','Open','ADJ003','Cork','PRV003',8100.00,8500.00,0.00,'2024-04-12','2024-04-10',NULL,0,6,2700.00),
    ('CLM0024','CUST031','POL021','Health','Closed','ADJ005','Galway',NULL,2800.50,3000.00,2600.00,'2023-05-09','2023-05-05','2023-12-01',0,1,950.00),
    ('CLM0025','CUST048','POL038','Property','Open','ADJ001','Limerick','PRV002',35000.00,37000.00,0.00,'2024-01-30','2024-01-28',NULL,1,3,4600.00),
    ('CLM0026','CUST006','POL015','Travel','Closed','ADJ004','Dublin','PRV004',900.25,950.00,850.00,'2023-08-05','2023-08-03','2023-10-15',0,0,500.00),
    ('CLM0027','CUST052','POL022','Liability','Open','ADJ002','Cork',NULL,55000.00,58000.00,0.00,'2024-03-25','2024-03-20',NULL,1,4,4900.00),
    ('CLM0028','CUST024','POL029','Motor','Closed','ADJ003','Waterford','PRV001',6300.00,6600.00,5900.00,'2023-09-11','2023-09-09','2024-05-08',0,2,2400.00),
    ('CLM0029','CUST013','POL006','Health','Pending','ADJ005','Dublin','PRV003',4100.00,4300.00,900.00,'2023-11-14','2023-11-10',NULL,0,0,1300.00),
    ('CLM0030','CUST041','POL035','Liability','Closed','ADJ001','Galway','PRV002',31200.00,32000.00,29500.00,'2023-04-20','2023-04-18','2024-02-28',0,2,4800.00),
    -- Outliers
    ('CLM0041','CUST010','POL008','Motor','Open','ADJ003','Dublin','PRV002',250000.00,260000.00,0.00,'2024-02-10','2024-02-08',NULL,1,7,4800.00),
    ('CLM0042','CUST057','POL036','Property','Closed','ADJ004','Cork','PRV004',380000.00,390000.00,350000.00,'2023-07-03','2023-06-30','2024-06-20',0,1,4900.00),
    -- Negative amounts
    ('CLM0043','CUST021','POL020','Health','Open','ADJ002','Galway',NULL,-500.00,2000.00,0.00,'2024-03-15','2024-03-14',NULL,0,0,800.00),
    ('CLM0044','CUST046','POL032','Motor','Pending','ADJ005','Limerick','PRV001',-500.00,3500.00,0.00,'2023-09-22','2023-09-20',NULL,0,1,1700.00),
    -- Nulls
    ('CLM0035','CUST002','POL001','Liability','Open','ADJ001','Cork',NULL,NULL,42000.00,0.00,'2024-02-28','2024-02-25',NULL,0,2,4700.00),
    ('CLM0065','CUST055','POL016','Liability','Open','ADJ001','Cork','PRV003',NULL,35000.00,0.00,'2024-04-18','2024-04-16',NULL,0,2,4700.00),
    -- Duplicates
    ('CLM0004','CUST015','POL002','Motor','Closed','ADJ001','Cork','PRV004',14335.58,5579.83,341.31,'2023-01-19','2023-04-03','2023-10-21',0,5,2846.57),
    ('CLM0008','CUST022','POL018','Motor','Closed','ADJ003','Dublin','PRV002',52341.90,55000.00,48000.00,'2023-05-22','2023-05-20','2024-02-14',1,3,4100.00),
    ('CLM0016','CUST039','POL033','Property','Closed','ADJ001','Cork','PRV004',6700.00,7000.00,6200.00,'2023-06-14','2023-06-10','2024-01-20',0,1,2100.00);
    
    -- Insert reserve history
    INSERT INTO reserve_history VALUES
    ('CLM0004','2023-02-01',5200.00),('CLM0004','2023-06-20',5600.00),('CLM0004','2023-09-15',5400.00),
    ('CLM0008','2023-06-01',52000.00),('CLM0008','2023-09-10',54000.00),('CLM0008','2024-01-05',55000.00),
    ('CLM0010','2023-03-01',28000.00),('CLM0010','2023-08-15',30500.00),('CLM0010','2024-01-20',30000.00),
    ('CLM0012','2023-05-01',15000.00),('CLM0012','2023-08-20',16200.00),('CLM0012','2023-11-10',16000.00),
    ('CLM0028','2023-10-01',6200.00),('CLM0028','2024-01-10',6500.00),('CLM0028','2024-04-15',6600.00),
    ('CLM0030','2023-05-01',30000.00),('CLM0030','2023-09-20',31500.00),('CLM0030','2024-02-10',32000.00),
    ('CLM0042','2023-08-01',370000.00),('CLM0042','2023-11-20',385000.00),('CLM0042','2024-05-10',390000.00);
    
    

> **Injected quality issues to find:** 2 future/invalid dates · 2 negative amounts · 2 null claim_amounts · 3 duplicate claim_ids · 2 outlier claims (£250k, £380k) · 1 closure before fnol

* * *

## Dimension 1 — Completeness

> Are all expected values present? No missing or null data.

### SQL
    
    
    -- Null count and percentage per column
    SELECT
        'claim_amount'  AS column_name,
        COUNT(*)        AS total_rows,
        SUM(CASE WHEN claim_amount  IS NULL THEN 1 ELSE 0 END) AS null_count,
        ROUND(100.0 * SUM(CASE WHEN claim_amount IS NULL THEN 1 ELSE 0 END)
              / COUNT(*), 2)                                   AS null_pct
    FROM claims
    UNION ALL
    SELECT 'closure_date', COUNT(*),
        SUM(CASE WHEN closure_date IS NULL THEN 1 ELSE 0 END),
        ROUND(100.0 * SUM(CASE WHEN closure_date IS NULL THEN 1 ELSE 0 END) / COUNT(*), 2)
    FROM claims
    UNION ALL
    SELECT 'provider_id', COUNT(*),
        SUM(CASE WHEN provider_id IS NULL THEN 1 ELSE 0 END),
        ROUND(100.0 * SUM(CASE WHEN provider_id IS NULL THEN 1 ELSE 0 END) / COUNT(*), 2)
    FROM claims;
    
    

**Expected output:**

column_name| total_rows| null_count| null_pct  
---|---|---|---  
claim_amount| 39| 2| 5.13  
closure_date| 39| 20| 51.28  
provider_id| 39| 7| 17.95  
  
### Pandas
    
    
    # Null count and percentage for all columns
    null_report = pd.DataFrame({
        'null_count': df.isnull().sum(),
        'null_pct':  (df.isnull().sum() / len(df) * 100).round(2)
    })
    print(null_report[null_report['null_count'] > 0])
    
    # Overall completeness score
    overall = (1 - df.isnull().mean()).mean() * 100
    print(f"Overall completeness: {overall:.1f}%")
    
    

**Expected output:**
    
    
                    null_count  null_pct
    provider_id              7     17.95
    claim_amount             2      5.13
    closure_date            20     51.28
    Overall completeness: 92.4%
    
    

* * *

## Dimension 2 — Uniqueness

> No duplicate records. Each claim_id should appear exactly once.

### SQL
    
    
    -- Step 1: find duplicates
    SELECT claim_id, COUNT(*) AS occurrences
    FROM claims
    GROUP BY claim_id
    HAVING COUNT(*) > 1
    ORDER BY occurrences DESC;
    
    -- Step 2: deduplicate — keep most recent
    WITH ranked AS (
        SELECT *,
               ROW_NUMBER() OVER (
                   PARTITION BY claim_id
                   ORDER BY fnol_date DESC
               ) AS rn
        FROM claims
    )
    SELECT * FROM ranked WHERE rn = 1;
    
    

**Expected output (Step 1):**

claim_id| occurrences  
---|---  
CLM0004| 2  
CLM0008| 2  
CLM0016| 2  
  
### Pandas
    
    
    # Find duplicate claim_ids
    dupes = df[df.duplicated(subset='claim_id', keep=False)][
        ['claim_id','claimant_id','claim_type','fnol_date']
    ].sort_values('claim_id')
    print(f"Duplicates found: {df['claim_id'].duplicated().sum()}")
    print(dupes)
    
    # Deduplicate — keep last (most recent load)
    df_dedup = df.drop_duplicates(subset='claim_id', keep='last').copy()
    print(f"Before: {len(df)}  After: {len(df_dedup)}")
    
    # Uniqueness rate
    rate = df['claim_id'].nunique() / len(df) * 100
    print(f"Uniqueness rate: {rate:.1f}%  {'PASS' if rate >= 99 else 'FAIL'}")
    
    

**Expected output:**
    
    
    Duplicates found: 3
      claim_id claimant_id claim_type   fnol_date
       CLM0004     CUST015      Motor  2023-01-19
       CLM0004     CUST015      Motor  2023-01-19
       CLM0008     CUST022      Motor  2023-05-22
       ...
    Before: 39  After: 36
    Uniqueness rate: 92.3%  FAIL
    
    

> **SQL tip:** `ROW_NUMBER()` with `PARTITION BY claim_id ORDER BY fnol_date DESC` is the standard deduplication pattern. `DENSE_RANK` and `RANK` would also work but `ROW_NUMBER` guarantees exactly one row per partition.

* * *

## Dimension 3 — Validity

> Values must conform to business rules. claim_amount > 0, dates must be logical, no future FNOL dates.

### SQL
    
    
    -- Summary: count all violations in one query
    SELECT
        SUM(CASE WHEN claim_amount  <= 0             THEN 1 ELSE 0 END) AS negative_amounts,
        SUM(CASE WHEN fnol_date     > '2025-01-01'   THEN 1 ELSE 0 END) AS future_fnol_dates,
        SUM(CASE WHEN closure_date  <  fnol_date      THEN 1 ELSE 0 END) AS closure_before_fnol,
        SUM(CASE WHEN reserve_amount < paid_amount    THEN 1 ELSE 0 END) AS reserve_below_paid
    FROM claims;
    
    -- Drill: see the actual invalid rows
    SELECT claim_id, claim_type, claim_amount, fnol_date, closure_date
    FROM claims
    WHERE claim_amount  <= 0
       OR fnol_date     >  '2025-01-01'
       OR closure_date  <  fnol_date;
    
    

**Expected output (summary):**

negative_amounts| future_fnol_dates| closure_before_fnol| reserve_below_paid  
---|---|---|---  
2| 1| 1| 5  
  
### Pandas
    
    
    # Summary of all violations
    checks = {
        'negative_claim_amount':  int((df['claim_amount'] <= 0).sum()),
        'future_fnol_date':       int((df['fnol_date'] > pd.Timestamp('2025-01-01')).sum()),
        'closure_before_fnol':    int((df['closure_date'] < df['fnol_date']).sum()),
        'reserve_below_paid':     int((df['reserve_amount'] < df['paid_amount']).sum()),
    }
    for k, v in checks.items():
        print(f"  {'OK  ' if v == 0 else 'FAIL'} | {k}: {v} records")
    
    # Show the invalid rows
    invalid = df[
        (df['claim_amount'] <= 0) |
        (df['fnol_date'] > pd.Timestamp('2025-01-01')) |
        (df['closure_date'] < df['fnol_date'])
    ][['claim_id','claim_amount','fnol_date','closure_date']]
    print(invalid)
    
    

**Expected output:**
    
    
      FAIL | negative_claim_amount: 2 records   (CLM0043, CLM0044 = -500)
      FAIL | future_fnol_date: 1 records        (CLM0002 = 2026-08-15)
      FAIL | closure_before_fnol: 1 records     (CLM0003 closure 2022-01-01 < fnol 2023-06-01)
      FAIL | reserve_below_paid: 5 records
    
    

* * *

## Dimension 4 — Consistency

> Same concept represented the same way across the dataset. Closed claims must have a closure_date. Open claims must not.

### SQL
    
    
    -- Status vs date consistency
    SELECT
        claim_status,
        COUNT(*) AS total,
        SUM(CASE WHEN claim_status = 'Closed' AND closure_date IS NULL     THEN 1 ELSE 0 END) AS closed_no_date,
        SUM(CASE WHEN claim_status = 'Open'   AND closure_date IS NOT NULL THEN 1 ELSE 0 END) AS open_with_date
    FROM claims
    GROUP BY claim_status;
    
    -- Find the specific inconsistent rows
    SELECT claim_id, claim_status, fnol_date, closure_date
    FROM claims
    WHERE (claim_status = 'Closed' AND closure_date IS NULL)
       OR (claim_status = 'Open'   AND closure_date IS NOT NULL);
    
    

**Expected output:**

claim_status| total| closed_no_date| open_with_date  
---|---|---|---  
Closed| 14| 0| 0  
Open| 14| 0| 0  
Pending| 7| 0| 0  
Reopened| 1| 0| 0  
  
### Pandas
    
    
    # Group by status and check date consistency
    by_status = df.groupby('claim_status').agg(
        total=('claim_id','count'),
        missing_closure=('closure_date', lambda x: x.isna().sum()),
        has_closure=('closure_date', lambda x: x.notna().sum())
    )
    by_status['closed_no_date'] = by_status.apply(
        lambda r: r['missing_closure'] if r.name == 'Closed' else 0, axis=1)
    by_status['open_with_date'] = by_status.apply(
        lambda r: r['has_closure'] if r.name == 'Open' else 0, axis=1)
    print(by_status[['total','closed_no_date','open_with_date']])
    
    # Find inconsistent records
    inconsistent = df[
        ((df['claim_status'] == 'Closed') & df['closure_date'].isna()) |
        ((df['claim_status'] == 'Open')   & df['closure_date'].notna())
    ]
    print(f"Inconsistent records: {len(inconsistent)}")
    
    

* * *

## Dimension 5 — Timeliness

> Data available when needed. Open claims older than 90 days breach SLA.

### SQL
    
    
    -- Flag open claims by SLA band
    SELECT
        claim_id,
        claim_type,
        adjuster_id,
        fnol_date,
        DATEDIFF(day, fnol_date, '2025-04-24')        AS days_open,
        -- SQLite: CAST(JULIANDAY('2025-04-24') - JULIANDAY(fnol_date) AS INT)
        -- BigQuery: DATE_DIFF(DATE '2025-04-24', fnol_date, DAY)
        CASE
            WHEN DATEDIFF(day, fnol_date, '2025-04-24') > 180 THEN 'Critical'
            WHEN DATEDIFF(day, fnol_date, '2025-04-24') > 90  THEN 'SLA Breach'
            ELSE 'Within SLA'
        END AS sla_status
    FROM claims
    WHERE claim_status = 'Open'
    ORDER BY days_open DESC;
    
    -- Summary count by SLA band
    SELECT
        CASE
            WHEN DATEDIFF(day, fnol_date, '2025-04-24') > 180 THEN 'Critical'
            WHEN DATEDIFF(day, fnol_date, '2025-04-24') > 90  THEN 'SLA Breach'
            ELSE 'Within SLA'
        END      AS sla_status,
        COUNT(*) AS claim_count
    FROM claims
    WHERE claim_status = 'Open'
    GROUP BY 1
    ORDER BY claim_count DESC;
    
    

### Pandas
    
    
    today = pd.Timestamp('2025-04-24')
    open_c = df[df['claim_status'] == 'Open'].copy()
    open_c['days_open'] = (today - open_c['fnol_date']).dt.days
    
    # pd.cut equivalent of SQL CASE WHEN
    open_c['sla_status'] = pd.cut(
        open_c['days_open'],
        bins=[-np.inf, 90, 180, np.inf],
        labels=['Within SLA', 'SLA Breach', 'Critical']
    )
    
    print("SLA summary:")
    print(open_c['sla_status'].value_counts())
    
    print("\nTop 5 oldest open claims:")
    print(open_c[['claim_id','claim_type','adjuster_id','days_open','sla_status']]
          .sort_values('days_open', ascending=False).head(5).to_string(index=False))
    
    

**Expected output:**
    
    
    SLA summary:
    Critical      9
    SLA Breach    3
    Within SLA    2
    
    claim_id  claim_type adjuster_id  days_open sla_status
     CLM0027   Liability      ADJ002        400   Critical
     CLM0015   Liability      ADJ004        474   Critical
     CLM0025    Property      ADJ001        450   Critical
    
    

* * *

## Dimension 6 — Accuracy

> Values reflect reality. Distribution profiling reveals impossible or suspicious figures.

### SQL
    
    
    -- Full distribution profile by claim type
    SELECT
        claim_type,
        COUNT(*)                                                              AS count,
        ROUND(AVG(claim_amount),   2)                                         AS mean_amount,
        ROUND(MIN(claim_amount),   2)                                         AS min_amount,
        ROUND(MAX(claim_amount),   2)                                         AS max_amount,
        ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY claim_amount), 2)  AS median,
        ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY claim_amount), 2)  AS p95,
        ROUND(PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY claim_amount), 2)  AS p99
    FROM claims
    WHERE claim_amount IS NOT NULL AND claim_amount > 0
    GROUP BY claim_type
    ORDER BY mean_amount DESC;
    
    

### Pandas
    
    
    clean = df[df['claim_amount'].notna() & (df['claim_amount'] > 0)]
    
    # Overall distribution
    print(clean['claim_amount'].describe(percentiles=[.25,.5,.75,.95,.99]).round(2))
    
    # By claim type — compare mean vs median (skew indicator)
    profile = clean.groupby('claim_type')['claim_amount'].agg(
        count='count',
        mean=lambda x: round(x.mean(), 2),
        median=lambda x: round(x.median(), 2),
        p95=lambda x: round(x.quantile(.95), 2),
        max='max'
    )
    print(profile.sort_values('mean', ascending=False))
    
    

> **Key insight:** When `mean >> median`, data is right-skewed — common in insurance. A few large claims inflate the mean. Always report both in assessments.

* * *

## DQ Scorecard — all 6 dimensions combined

### SQL
    
    
    SELECT check_name, score_pct, threshold_pct,
           CASE WHEN score_pct >= threshold_pct THEN 'PASS' ELSE 'FAIL' END AS result
    FROM (
        SELECT 'Completeness — claim_amount' AS check_name,
               ROUND(100.0 * SUM(CASE WHEN claim_amount IS NOT NULL THEN 1 ELSE 0 END) / COUNT(*), 1) AS score_pct,
               95.0 AS threshold_pct
        FROM claims
        UNION ALL
        SELECT 'Uniqueness — claim_id',
               ROUND(100.0 * COUNT(DISTINCT claim_id) / COUNT(*), 1), 99.0
        FROM claims
        UNION ALL
        SELECT 'Validity — positive amounts',
               ROUND(100.0 * SUM(CASE WHEN claim_amount > 0 OR claim_amount IS NULL THEN 1 ELSE 0 END) / COUNT(*), 1), 98.0
        FROM claims
        UNION ALL
        SELECT 'Consistency — closed has closure date',
               ROUND(100.0 * SUM(CASE WHEN claim_status <> 'Closed' OR closure_date IS NOT NULL THEN 1 ELSE 0 END) / COUNT(*), 1), 95.0
        FROM claims
        UNION ALL
        SELECT 'Validity — closure after fnol',
               ROUND(100.0 * SUM(CASE WHEN closure_date IS NULL OR closure_date >= fnol_date THEN 1 ELSE 0 END) / COUNT(*), 1), 99.0
        FROM claims
    ) checks;
    
    

### Pandas
    
    
    total = len(df)
    
    scorecard = pd.DataFrame([
        {'check': 'Completeness — claim_amount',
         'score': round(df['claim_amount'].notna().sum() / total * 100, 1), 'threshold': 95.0},
        {'check': 'Uniqueness — claim_id',
         'score': round(df['claim_id'].nunique() / total * 100, 1),         'threshold': 99.0},
        {'check': 'Validity — positive amounts',
         'score': round(((df['claim_amount'] > 0) | df['claim_amount'].isna()).sum() / total * 100, 1), 'threshold': 98.0},
        {'check': 'Consistency — closed has date',
         'score': round(((df['claim_status'] != 'Closed') | df['closure_date'].notna()).sum() / total * 100, 1), 'threshold': 95.0},
        {'check': 'Validity — closure after fnol',
         'score': round((df['closure_date'].isna() | (df['closure_date'] >= df['fnol_date'])).sum() / total * 100, 1), 'threshold': 99.0},
    ])
    scorecard['result'] = scorecard.apply(
        lambda r: 'PASS' if r['score'] >= r['threshold'] else 'FAIL', axis=1)
    print(scorecard.to_string(index=False))
    print(f"\n{(scorecard['result']=='PASS').sum()}/{len(scorecard)} checks passing")
    
    

**Expected output:**

check| score| threshold| result  
---|---|---|---  
Completeness — claim_amount| 94.9| 95.0| FAIL  
Uniqueness — claim_id| 92.3| 99.0| FAIL  
Validity — positive amounts| 94.9| 98.0| FAIL  
Consistency — closed has date| 100.0| 95.0| PASS  
Validity — closure after fnol| 97.4| 99.0| FAIL  
  
* * *

## Outlier detection

### IQR method (recommended for skewed claims data)

#### SQL
    
    
    WITH quartiles AS (
        SELECT
            claim_type,
            PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY claim_amount) AS q1,
            PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY claim_amount) AS q3
        FROM claims
        WHERE claim_amount IS NOT NULL AND claim_amount > 0
        GROUP BY claim_type
    ),
    bounds AS (
        SELECT claim_type, q1, q3,
               ROUND(q1 - 1.5*(q3-q1), 2) AS lower_fence,
               ROUND(q3 + 1.5*(q3-q1), 2) AS upper_fence
        FROM quartiles
    )
    SELECT c.claim_id, c.claim_type,
           ROUND(c.claim_amount, 2) AS claim_amount,
           b.upper_fence,
           'OUTLIER' AS flag
    FROM claims c
    JOIN bounds b ON c.claim_type = b.claim_type
    WHERE c.claim_amount > b.upper_fence
       OR c.claim_amount < b.lower_fence
    ORDER BY c.claim_amount DESC;
    
    

#### Pandas
    
    
    clean = df[df['claim_amount'].notna() & (df['claim_amount'] > 0)].copy()
    
    Q1, Q3  = clean['claim_amount'].quantile([0.25, 0.75])
    IQR     = Q3 - Q1
    lower   = Q1 - 1.5 * IQR
    upper   = Q3 + 1.5 * IQR
    
    print(f"Q1={Q1:,.0f}  Q3={Q3:,.0f}  IQR={IQR:,.0f}")
    print(f"Lower fence={lower:,.0f}  Upper fence={upper:,.0f}")
    
    outliers = clean[(clean['claim_amount'] < lower) | (clean['claim_amount'] > upper)]
    print(f"\nOutliers: {len(outliers)}")
    print(outliers[['claim_id','claim_type','claim_amount']].sort_values('claim_amount', ascending=False))
    
    

**Expected output:**
    
    
    Q1=3,201  Q3=29,500  IQR=26,299
    Lower fence=-36,248  Upper fence=68,949
    
    Outliers: 2
      claim_id claim_type  claim_amount
       CLM0042   Property     380000.00
       CLM0041      Motor     250000.00
    
    

### Z-score method (for normally distributed data)

#### SQL
    
    
    WITH stats AS (
        SELECT AVG(claim_amount)   AS mean_amt,
               STDEV(claim_amount) AS std_amt   -- SQLite: manual formula
        FROM claims
        WHERE claim_amount IS NOT NULL AND claim_amount > 0
    )
    SELECT c.claim_id, c.claim_type,
           ROUND(c.claim_amount, 2)                                               AS claim_amount,
           ROUND(ABS(c.claim_amount - s.mean_amt) / NULLIF(s.std_amt, 0), 2)     AS z_score
    FROM claims c CROSS JOIN stats s
    WHERE ABS(c.claim_amount - s.mean_amt) / NULLIF(s.std_amt, 0) > 3
    ORDER BY z_score DESC;
    
    

#### Pandas
    
    
    from scipy import stats as scipy_stats
    
    clean['z_score'] = np.abs(scipy_stats.zscore(clean['claim_amount']))
    outliers_z = clean[clean['z_score'] > 3]
    print(outliers_z[['claim_id','claim_type','claim_amount','z_score']]
          .sort_values('z_score', ascending=False))
    
    print(f"\nIQR method: {len(outliers)} | Z-score: {len(outliers_z)}")
    print("Use IQR for insurance — claims are right-skewed, z-score is less reliable")
    
    

### Three handling strategies
    
    
    # Strategy 1: FLAG — best for insurance (preserve data, add signal)
    df['outlier_flag'] = ((df['claim_amount'].notna()) & (df['claim_amount'] > upper)).astype(int)
    
    # Strategy 2: CAP / WINSORISE — replace at fence, preserves row count
    df['claim_amount_capped'] = df['claim_amount'].clip(lower=lower, upper=upper)
    
    # Strategy 3: REMOVE — avoid in insurance (a £380k claim may be legitimate)
    df_removed = df[(df['claim_amount'].isna()) |
                    ((df['claim_amount'] >= lower) & (df['claim_amount'] <= upper))]
    
    print(f"Flagged:  {df['outlier_flag'].sum()} records")
    print(f"Max after cap:  {df['claim_amount_capped'].max():,.2f}")
    print(f"Rows after remove: {len(df_removed)}")
    
    

> **Assessment phrase:** "I prefer to flag outliers in insurance rather than remove them — a £380k claim is statistically unusual but may be entirely legitimate. Removing it would distort loss ratio calculations."

* * *

## Window functions — 8 insurance scenarios

### 3A. ROW_NUMBER — top claim per claimant

#### SQL
    
    
    -- ROW_NUMBER: always unique, no ties. Best for "pick exactly one row per group"
    WITH ranked AS (
        SELECT claim_id, claimant_id, claim_type, claim_amount,
               ROW_NUMBER() OVER (
                   PARTITION BY claimant_id
                   ORDER BY claim_amount DESC
               ) AS rn
        FROM claims
        WHERE claim_amount IS NOT NULL AND claim_amount > 0
    )
    SELECT claim_id, claimant_id, claim_type, ROUND(claim_amount,2) AS claim_amount
    FROM ranked
    WHERE rn = 1
    ORDER BY claim_amount DESC;
    
    

#### Pandas
    
    
    # Pandas equivalent of ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)
    top = (df[df['claim_amount'] > 0]
           .sort_values('claim_amount', ascending=False)
           .groupby('claimant_id').first()
           .reset_index()
           [['claimant_id','claim_id','claim_type','claim_amount']]
           .sort_values('claim_amount', ascending=False))
    print(top.head(10).to_string(index=False))
    
    

* * *

### 3B. RANK vs DENSE_RANK — tie handling

Function| Ties get same rank?| Gaps after tie?| Example (1,1,?,4)  
---|---|---|---  
`ROW_NUMBER`| No — always unique| —| 1, 2, 3, 4  
`RANK`| Yes| Yes| 1, 1, **3** , 4  
`DENSE_RANK`| Yes| No| 1, 1, **2** , 3  
  
#### SQL
    
    
    WITH counts AS (
        SELECT adjuster_id, COUNT(*) AS claim_count
        FROM claims GROUP BY adjuster_id
    )
    SELECT adjuster_id, claim_count,
           RANK()       OVER (ORDER BY claim_count DESC) AS rank_with_gap,
           DENSE_RANK() OVER (ORDER BY claim_count DESC) AS rank_no_gap
    FROM counts
    ORDER BY claim_count DESC;
    
    

#### Pandas
    
    
    adj = df.groupby('adjuster_id').size().reset_index(name='claim_count')
    adj['rank_with_gap'] = adj['claim_count'].rank(method='min',   ascending=False).astype(int)
    adj['rank_no_gap']   = adj['claim_count'].rank(method='dense', ascending=False).astype(int)
    print(adj.sort_values('claim_count', ascending=False).to_string(index=False))
    
    

> **Key mapping:** `rank(method='min') = RANK()` | `rank(method='dense') = DENSE_RANK()` | `rank(method='first') = ROW_NUMBER()`

* * *

### 3C. LAG & LEAD — fraud detection

#### SQL
    
    
    -- LAG: detect claimants who filed again within 30 days
    WITH ordered AS (
        SELECT claim_id, claimant_id, fnol_date, claim_amount,
               LAG(fnol_date)    OVER (PARTITION BY claimant_id ORDER BY fnol_date) AS prev_fnol,
               LAG(claim_amount) OVER (PARTITION BY claimant_id ORDER BY fnol_date) AS prev_amount
        FROM claims WHERE fnol_date IS NOT NULL
    )
    SELECT claim_id, claimant_id, fnol_date, prev_fnol,
           DATEDIFF(day, prev_fnol, fnol_date) AS days_since_last,
           ROUND(claim_amount, 2) AS claim_amount
    FROM ordered
    WHERE prev_fnol IS NOT NULL
      AND DATEDIFF(day, prev_fnol, fnol_date) <= 30
    ORDER BY days_since_last;
    
    -- LEAD: look ahead to next claim
    SELECT claim_id, claimant_id, fnol_date,
           LEAD(fnol_date,  1) OVER (PARTITION BY claimant_id ORDER BY fnol_date) AS next_claim_date,
           LEAD(claim_type, 1) OVER (PARTITION BY claimant_id ORDER BY fnol_date) AS next_claim_type
    FROM claims WHERE fnol_date IS NOT NULL
    ORDER BY claimant_id, fnol_date;
    
    

#### Pandas
    
    
    lag_df = df[df['claim_amount'] > 0].sort_values(['claimant_id','fnol_date']).copy()
    
    # shift(1) = LAG(1) | shift(-1) = LEAD(1)
    lag_df['prev_fnol']    = lag_df.groupby('claimant_id')['fnol_date'].shift(1)
    lag_df['next_fnol']    = lag_df.groupby('claimant_id')['fnol_date'].shift(-1)
    lag_df['days_since']   = (lag_df['fnol_date'] - lag_df['prev_fnol']).dt.days
    
    # Fraud signal: repeat claim within 30 days
    repeat = lag_df[lag_df['days_since'].notna() & (lag_df['days_since'] <= 30)]
    print(f"Repeat claims within 30 days: {len(repeat)}")
    print(repeat[['claim_id','claimant_id','fnol_date','prev_fnol','days_since']].to_string(index=False))
    
    

> **Key mapping:** `shift(1) = LAG(1)` | `shift(-1) = LEAD(1)` | `shift(n) = LAG(n)`

* * *

### 3D. Running total

#### SQL
    
    
    WITH monthly AS (
        SELECT FORMAT(fnol_date, 'yyyy-MM') AS month,      -- SQLite: STRFTIME('%Y-%m',...)
               ROUND(SUM(claim_amount), 2)  AS monthly_total,
               COUNT(*)                     AS claim_count
        FROM claims
        WHERE claim_amount > 0 AND fnol_date IS NOT NULL
        GROUP BY FORMAT(fnol_date, 'yyyy-MM')
    )
    SELECT month, monthly_total, claim_count,
           ROUND(SUM(monthly_total) OVER (
               ORDER BY month
               ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
           ), 2) AS running_total
    FROM monthly
    ORDER BY month;
    
    

#### Pandas
    
    
    # cumsum() = SUM() OVER (ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
    monthly = (df[df['claim_amount'] > 0]
               .groupby(df['fnol_date'].dt.to_period('M'))['claim_amount']
               .agg(monthly_total='sum', claim_count='count')
               .reset_index())
    monthly.columns = ['month','monthly_total','claim_count']
    monthly['running_total']  = monthly['monthly_total'].cumsum().round(2)
    monthly['running_claims'] = monthly['claim_count'].cumsum()
    print(monthly.to_string(index=False))
    
    

* * *

### 3E. Rolling average

#### SQL
    
    
    WITH monthly AS (
        SELECT FORMAT(fnol_date, 'yyyy-MM') AS month,
               ROUND(AVG(claim_amount), 2)  AS avg_monthly
        FROM claims
        WHERE claim_amount > 0 AND fnol_date IS NOT NULL
        GROUP BY FORMAT(fnol_date, 'yyyy-MM')
    )
    SELECT month, avg_monthly,
           ROUND(AVG(avg_monthly) OVER (
               ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
           ), 2) AS rolling_3mo_avg,
           ROUND(AVG(avg_monthly) OVER (
               ORDER BY month ROWS BETWEEN 11 PRECEDING AND CURRENT ROW
           ), 2) AS rolling_12mo_avg
    FROM monthly ORDER BY month;
    
    

#### Pandas
    
    
    # rolling(3) = ROWS BETWEEN 2 PRECEDING AND CURRENT ROW (current + 2 previous = 3 months)
    monthly['avg_monthly']      = (df[df['claim_amount'] > 0]
                                   .groupby(df['fnol_date'].dt.to_period('M'))['claim_amount']
                                   .mean().values)
    monthly['rolling_3mo_avg']  = monthly['avg_monthly'].rolling(3).mean().round(2)
    monthly['rolling_12mo_avg'] = monthly['avg_monthly'].rolling(12).mean().round(2)
    print(monthly[['month','avg_monthly','rolling_3mo_avg','rolling_12mo_avg']].to_string(index=False))
    
    

> **Key mapping:** `rolling(n) = ROWS BETWEEN (n-1) PRECEDING AND CURRENT ROW`

* * *

### 3F. NTILE — risk banding

#### SQL
    
    
    WITH spend AS (
        SELECT claimant_id, ROUND(SUM(claim_amount),2) AS total_spend,
               COUNT(*) AS claim_count, MAX(fraud_flag) AS any_fraud
        FROM claims WHERE claim_amount > 0
        GROUP BY claimant_id
    )
    SELECT claimant_id, total_spend, claim_count, any_fraud,
           NTILE(4) OVER (ORDER BY total_spend DESC) AS spend_quartile,
           CASE NTILE(4) OVER (ORDER BY total_spend DESC)
               WHEN 1 THEN 'High Risk'   WHEN 2 THEN 'Medium-High'
               WHEN 3 THEN 'Medium-Low'  WHEN 4 THEN 'Low Risk'
           END AS risk_band
    FROM spend ORDER BY total_spend DESC;
    
    

#### Pandas
    
    
    spend = df[df['claim_amount'] > 0].groupby('claimant_id').agg(
        total_spend=('claim_amount','sum'),
        claim_count=('claim_id','count'),
        fraud_count=('fraud_flag','sum')
    ).reset_index()
    
    # pd.qcut(q=4) = NTILE(4) — equal-frequency bins
    # pd.cut = equal-width bins — DIFFERENT, don't confuse them
    spend['risk_band'] = pd.qcut(
        spend['total_spend'], q=4,
        labels=['Low Risk','Medium-Low','Medium-High','High Risk']
    )
    print(spend.sort_values('total_spend', ascending=False).to_string(index=False))
    
    print("\nAverage by risk band:")
    print(spend.groupby('risk_band')[['total_spend','claim_count','fraud_count']].mean().round(2))
    
    

> **Key mapping:** `pd.qcut(q=4) = NTILE(4)` (equal frequency) | `pd.cut = equal-width` (different concept)

* * *

### 3G. FIRST_VALUE & LAST_VALUE — reserve development

#### SQL
    
    
    -- CRITICAL GOTCHA: without the explicit frame, LAST_VALUE = CURRENT ROW
    SELECT claim_id, update_date, ROUND(reserve_amount,2) AS reserve_amount,
           ROUND(FIRST_VALUE(reserve_amount) OVER (
               PARTITION BY claim_id ORDER BY update_date
               ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
           ), 2) AS initial_reserve,
           ROUND(LAST_VALUE(reserve_amount) OVER (
               PARTITION BY claim_id ORDER BY update_date
               ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
               -- WITHOUT this frame = LAST_VALUE returns CURRENT ROW (common mistake!)
           ), 2) AS final_reserve
    FROM reserve_history
    ORDER BY claim_id, update_date;
    
    

#### Pandas
    
    
    # Pandas .first() and .last() always give the true first/last — no frame gotcha
    res = rh.sort_values('update_date').groupby('claim_id').agg(
        initial_reserve=('reserve_amount','first'),
        final_reserve  =('reserve_amount','last'),
        num_updates    =('reserve_amount','count')
    ).reset_index()
    res['change_pct'] = ((res['final_reserve'] - res['initial_reserve'])
                          / res['initial_reserve'] * 100).round(1)
    res['movement'] = pd.cut(
        res['change_pct'],
        bins=[-np.inf, -20, 0, 20, np.inf],
        labels=['Significantly released','Released','Strengthened','Significantly strengthened']
    )
    print(res.to_string(index=False))
    
    

**Expected output:**

claim_id| initial_reserve| final_reserve| change_pct| movement  
---|---|---|---|---  
CLM0004| 5200.00| 5400.00| +3.8| Strengthened  
CLM0008| 52000.00| 55000.00| +5.8| Strengthened  
CLM0042| 370000.00| 390000.00| +5.4| Strengthened  
  
> **SQL gotcha to mention in your assessment:** "LAST_VALUE requires `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` — without it the frame stops at the current row, so LAST_VALUE simply returns the current row's value. This is the most common window function mistake."

* * *

### 3H. PERCENT_RANK — claim percentile within type

#### SQL
    
    
    -- Flag top 5% within each claim type for SIU review
    SELECT * FROM (
        SELECT claim_id, claim_type, ROUND(claim_amount,2) AS claim_amount,
               ROUND(PERCENT_RANK() OVER (
                   PARTITION BY claim_type ORDER BY claim_amount
               ) * 100, 1) AS percentile_in_type,
               ROUND(CUME_DIST() OVER (
                   PARTITION BY claim_type ORDER BY claim_amount
               ) * 100, 1) AS cumulative_dist_pct
        FROM claims
        WHERE claim_amount IS NOT NULL AND claim_amount > 0
    ) ranked
    WHERE percentile_in_type >= 95
    ORDER BY claim_type, claim_amount DESC;
    
    

#### Pandas
    
    
    # rank(pct=True) = PERCENT_RANK() / CUME_DIST() combined
    work = df[df['claim_amount'] > 0].copy()
    work['pct_rank_in_type'] = (
        work.groupby('claim_type')['claim_amount']
        .rank(pct=True).mul(100).round(1)
    )
    
    top5 = work[work['pct_rank_in_type'] >= 95][
        ['claim_id','claim_type','claim_amount','pct_rank_in_type','fraud_flag']
    ].sort_values(['claim_type','claim_amount'], ascending=[True,False])
    print(f"Top 5% SIU candidates: {len(top5)}")
    print(top5.to_string(index=False))
    
    

* * *

## Combined — fraud risk scoring query

Combines LAG, PERCENT_RANK, ROW_NUMBER, and a composite score in one query.

### SQL
    
    
    SELECT c.claim_id, c.claimant_id, c.claim_type,
           ROUND(c.claim_amount, 2) AS claim_amount,
    
           -- Percentile rank within claim type (high = expensive vs peers)
           ROUND(PERCENT_RANK() OVER (
               PARTITION BY c.claim_type ORDER BY c.claim_amount) * 100, 1) AS pct_rank_in_type,
    
           -- Days since this claimant's previous claim
           DATEDIFF(day,
               LAG(c.fnol_date) OVER (PARTITION BY c.claimant_id ORDER BY c.fnol_date),
               c.fnol_date) AS days_since_last_claim,
    
           -- Running claim count per claimant
           COUNT(*) OVER (
               PARTITION BY c.claimant_id
               ORDER BY c.fnol_date
               ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
           ) AS claim_sequence,
    
           -- Composite fraud risk score
           CASE WHEN PERCENT_RANK() OVER (
                   PARTITION BY c.claim_type ORDER BY c.claim_amount) >= 0.95
                THEN 3 ELSE 0 END
           + CASE WHEN DATEDIFF(day,
                   LAG(c.fnol_date) OVER (PARTITION BY c.claimant_id ORDER BY c.fnol_date),
                   c.fnol_date) <= 30
                THEN 3 ELSE 0 END
           + CASE WHEN c.fraud_flag = 1 THEN 2 ELSE 0 END
           + CASE WHEN c.num_prev_claims > 4 THEN 1 ELSE 0 END AS fraud_risk_score
    
    FROM claims c
    WHERE c.claim_amount IS NOT NULL AND c.claim_amount > 0
    ORDER BY fraud_risk_score DESC, claim_amount DESC;
    
    

### Pandas
    
    
    scored = df[df['claim_amount'] > 0].sort_values(['claimant_id','fnol_date']).copy()
    
    # LAG
    scored['prev_fnol']       = scored.groupby('claimant_id')['fnol_date'].shift(1)
    scored['days_since_last'] = (scored['fnol_date'] - scored['prev_fnol']).dt.days
    
    # PERCENT_RANK
    scored['pct_rank'] = scored.groupby('claim_type')['claim_amount'].rank(pct=True).mul(100)
    
    # ROW_NUMBER / claim sequence
    scored['claim_seq'] = scored.groupby('claimant_id').cumcount() + 1
    
    # Composite score
    scored['fraud_risk_score'] = (
        (scored['pct_rank'] >= 95).astype(int)                    * 3 +
        (scored['days_since_last'].fillna(999) <= 30).astype(int) * 3 +
        scored['fraud_flag']                                       * 2 +
        (scored['num_prev_claims'] > 4).astype(int)                * 1
    )
    
    result = (scored.sort_values('fraud_risk_score', ascending=False)
              [['claim_id','claimant_id','claim_type','claim_amount',
                'pct_rank','days_since_last','fraud_flag','fraud_risk_score']]
              .head(10))
    print(result.to_string(index=False))
    
    

* * *

## Insurance metrics

### Loss ratio
    
    
    -- SQL
    SELECT claim_type, COUNT(*) AS claims,
           ROUND(SUM(paid_amount),  2) AS total_paid,
           ROUND(SUM(premium),      2) AS total_premium,
           ROUND(100.0 * SUM(paid_amount) / NULLIF(SUM(premium),0), 1) AS loss_ratio_pct,
           CASE WHEN 100.0 * SUM(paid_amount) / NULLIF(SUM(premium),0) > 100 THEN 'Unprofitable'
                WHEN 100.0 * SUM(paid_amount) / NULLIF(SUM(premium),0) >  70 THEN 'Watch'
                ELSE 'Healthy' END AS status
    FROM claims
    GROUP BY claim_type ORDER BY loss_ratio_pct DESC;
    
    
    
    
    # Pandas
    lr = df.groupby('claim_type').apply(lambda g: pd.Series({
        'claims':        len(g),
        'total_paid':    round(g['paid_amount'].sum(), 2),
        'total_premium': round(g['premium'].sum(), 2),
        'loss_ratio':    round(g['paid_amount'].sum() / g['premium'].sum() * 100, 1)
    })).reset_index()
    lr['status'] = lr['loss_ratio'].apply(
        lambda x: 'Unprofitable' if x > 100 else ('Watch' if x > 70 else 'Healthy'))
    print(lr.sort_values('loss_ratio', ascending=False).to_string(index=False))
    
    

### Cycle time (FNOL to closure)
    
    
    -- SQL: always show mean AND median — claims are right-skewed
    SELECT claim_type, COUNT(*) AS closed_claims,
           ROUND(AVG(DATEDIFF(day, fnol_date, closure_date)), 1)                       AS avg_days,
           ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP
                 (ORDER BY DATEDIFF(day, fnol_date, closure_date)), 1)                 AS median_days,
           MAX(DATEDIFF(day, fnol_date, closure_date))                                 AS max_days
    FROM claims
    WHERE claim_status = 'Closed' AND closure_date IS NOT NULL AND fnol_date IS NOT NULL
    GROUP BY claim_type ORDER BY avg_days DESC;
    
    
    
    
    # Pandas
    closed = df[(df['claim_status']=='Closed') & df['closure_date'].notna()].copy()
    closed['cycle_days'] = (closed['closure_date'] - closed['fnol_date']).dt.days
    print(closed.groupby('claim_type')['cycle_days'].agg(
        count='count',
        mean=lambda x: round(x.mean(),1),
        median=lambda x: round(x.median(),1),
        max='max'
    ).sort_values('mean', ascending=False).to_string())
    
    

* * *

## Quick reference — SQL to Pandas mapping

SQL window function| Pandas equivalent| Notes  
---|---|---  
`ROW_NUMBER() OVER (PARTITION BY a ORDER BY b)`| `.sort_values(b).groupby(a).cumcount() + 1`| Unique rank, no ties  
`RANK() OVER (ORDER BY x DESC)`| `.rank(method='min', ascending=False)`| Gaps after ties  
`DENSE_RANK() OVER (ORDER BY x DESC)`| `.rank(method='dense', ascending=False)`| No gaps after ties  
`LAG(x, 1) OVER (PARTITION BY a ORDER BY b)`| `.groupby(a)[x].shift(1)`| Look back 1 row  
`LEAD(x, 1) OVER (PARTITION BY a ORDER BY b)`| `.groupby(a)[x].shift(-1)`| Look forward 1 row  
`SUM(x) OVER (ORDER BY b ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`| `.cumsum()`| Running total  
`AVG(x) OVER (ORDER BY b ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)`| `.rolling(3).mean()`| 3-month rolling avg  
`NTILE(4) OVER (ORDER BY x)`| `pd.qcut(x, q=4)`| Equal-frequency bands  
`PERCENT_RANK() OVER (PARTITION BY a ORDER BY x)`| `.groupby(a)[x].rank(pct=True)`| 0 to 1 percentile  
`FIRST_VALUE(x) OVER (PARTITION BY a ORDER BY b ...)`| `.groupby(a)[x].transform('first')`| First value in group  
`LAST_VALUE(x) OVER (PARTITION BY a ORDER BY b ROWS BETWEEN UNBOUNDED ... FOLLOWING)`| `.groupby(a)[x].transform('last')`| **SQL needs explicit frame!**  
  
* * *

## Key phrases for your assessment

> "IQR is my preferred outlier method for insurance claims — the data is right-skewed and z-score gets distorted by the very extreme values I'm trying to detect."

> "LAST_VALUE in SQL requires `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` — without this frame it defaults to the current row, which is the most common window function mistake."

> "I always report both mean and median for claims cycle time — complex litigation claims inflate the mean significantly and the median gives a more representative view of typical performance."

> "I flag outliers in insurance rather than remove them — a £380k claim is statistically unusual but entirely legitimate. Silently removing it would distort loss ratio and reserving calculations."

> "Data quality has 6 dimensions: Completeness, Uniqueness, Validity, Consistency, Timeliness, and Accuracy. I embed checks at every layer of the pipeline rather than validating only at the end."

> "`pd.qcut(q=4)` is the Pandas equivalent of `NTILE(4)` — equal-frequency bins. `pd.cut` creates equal-width bins which is a completely different concept."
