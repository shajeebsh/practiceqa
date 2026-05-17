---
title: "code-pandas"
date: 2026-04-25 08:45:30 
author: shajeebhameed
layout: page
slug: code-pandas__trashed
status: trash
---

Both files are completely self-contained with sample data built in — no external files needed.

Open in any SQL editor (DBeaver, Azure Data Studio, DataGrip). The first two sections are `CREATE TABLE` and `INSERT INTO` — run those first, then run any query section independently. The data has all the injected quality issues so the DQ queries will actually find real problems to fix.

**`pandas_with_data.py`** — Just run `python pandas_with_data.py`. The first 100 lines are the hardcoded claims and reserve history records. Every section prints its own header so you can follow along clearly.

**What's intentionally broken in the data** — so you know what to expect:
    
    
    # ============================================================
    # PANDAS PRACTICE WITH SAMPLE DATA
    # Insurance Claims Dataset — 203 rows + 150 reserve history rows
    # Run: python pandas_with_data.py
    # Requirements: pip install pandas numpy scipy
    # ============================================================
    
    import pandas as pd
    import numpy as np
    from scipy import stats
    import warnings
    warnings.filterwarnings('ignore')
    
    pd.set_option('display.max_columns', 20)
    pd.set_option('display.width', 120)
    pd.set_option('display.float_format', '{:.2f}'.format)
    
    
    # ============================================================
    # STEP 1: HARDCODED SAMPLE DATA
    # 203 claims rows + 150 reserve history rows
    # Includes injected DQ issues for practice exercises
    # ============================================================
    
    # ── CLAIMS DATA ──────────────────────────────────────────────
    claims_records = [
        # claim_id, claimant_id, policy_id, claim_type, claim_status, adjuster_id, region,
        # provider_id, claim_amount, reserve_amount, paid_amount, fnol_date, loss_date,
        # closure_date, fraud_flag, num_prev_claims, premium
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
        ('CLM0020','CUST037','POL004','Property','Closed','ADJ001','Waterford',None,22300.00,23000.00,21000.00,'2023-03-28','2023-03-25','2023-11-10',0,2,4500.00),
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
        ('CLM0031','CUST007','POL023','Motor','Open','ADJ004','Cork','PRV004',5600.00,5900.00,0.00,'2024-06-08','2024-06-06',None,0,1,2000.00),
        ('CLM0032','CUST036','POL010','Property','Closed','ADJ002','Limerick',None,19800.00,20500.00,18500.00,'2023-02-25','2023-02-22','2023-10-30',0,3,4300.00),
        ('CLM0033','CUST019','POL040','Motor','Open','ADJ003','Galway','PRV001',7200.00,7500.00,0.00,'2024-05-19','2024-05-17',None,1,1,2600.00),
        ('CLM0034','CUST054','POL027','Health','Closed','ADJ005','Dublin','PRV003',3700.50,3900.00,3400.00,'2023-10-07','2023-10-04','2024-07-12',0,0,1050.00),
        ('CLM0035','CUST002','POL001','Liability','Open','ADJ001','Cork',None,None,42000.00,0.00,'2024-02-28','2024-02-25',None,0,2,4700.00),  # null claim_amount
        ('CLM0036','CUST045','POL034','Motor','Closed','ADJ004','Waterford','PRV002',10500.00,11000.00,9800.00,'2023-06-30','2023-06-28','2024-03-15',0,4,3100.00),
        ('CLM0037','CUST029','POL028','Property','Open','ADJ002','Galway','PRV004',28000.00,29500.00,0.00,'2024-01-14','2024-01-12',None,1,3,4500.00),
        ('CLM0038','CUST016','POL017','Health','Pending','ADJ003','Dublin',None,5200.00,5500.00,1200.00,'2023-08-18','2023-08-15',None,0,0,1600.00),
        ('CLM0039','CUST058','POL039','Motor','Closed','ADJ005','Cork','PRV001',8800.00,9200.00,8100.00,'2023-05-25','2023-05-23','2024-01-18',0,2,2900.00),
        ('CLM0040','CUST033','POL003','Liability','Open','ADJ001','Limerick','PRV003',62000.00,65000.00,0.00,'2024-04-02','2024-03-30',None,1,6,4900.00),
        # Injected outliers (suspiciously large amounts)
        ('CLM0041','CUST010','POL008','Motor','Open','ADJ003','Dublin','PRV002',250000.00,260000.00,0.00,'2024-02-10','2024-02-08',None,1,7,4800.00),
        ('CLM0042','CUST057','POL036','Property','Closed','ADJ004','Cork','PRV004',380000.00,390000.00,350000.00,'2023-07-03','2023-06-30','2024-06-20',0,1,4900.00),
        # Injected negative amounts
        ('CLM0043','CUST021','POL020','Health','Open','ADJ002','Galway',None,-500.00,2000.00,0.00,'2024-03-15','2024-03-14',None,0,0,800.00),
        ('CLM0044','CUST046','POL032','Motor','Pending','ADJ005','Limerick','PRV001',-500.00,3500.00,0.00,'2023-09-22','2023-09-20',None,0,1,1700.00),
        # More regular claims to fill out the dataset
        ('CLM0045','CUST004','POL005','Travel','Closed','ADJ001','Dublin','PRV003',1250.00,1300.00,1150.00,'2023-11-28','2023-11-26','2024-02-05',0,0,550.00),
        ('CLM0046','CUST032','POL024','Motor','Open','ADJ002','Cork','PRV002',9100.00,9600.00,0.00,'2024-05-07','2024-05-05',None,0,3,2800.00),
        ('CLM0047','CUST050','POL013','Property','Closed','ADJ003','Waterford',None,14200.00,14800.00,13300.00,'2023-03-12','2023-03-10','2023-11-25',0,2,3700.00),
        ('CLM0048','CUST025','POL030','Liability','Open','ADJ004','Galway','PRV004',38500.00,40000.00,0.00,'2024-01-22','2024-01-20',None,1,4,4800.00),
        ('CLM0049','CUST014','POL016','Health','Closed','ADJ005','Dublin','PRV001',4800.00,5000.00,4400.00,'2023-08-14','2023-08-12','2024-05-20',0,1,1400.00),
        ('CLM0050','CUST039','POL007','Motor','Open','ADJ001','Cork','PRV003',6500.00,6800.00,0.00,'2024-04-25','2024-04-23',None,0,2,2200.00),
        ('CLM0051','CUST023','POL031','Property','Closed','ADJ002','Limerick','PRV002',17500.00,18000.00,16400.00,'2023-06-08','2023-06-05','2024-04-12',0,1,4200.00),
        ('CLM0052','CUST011','POL025','Liability','Open','ADJ003','Galway',None,None,27000.00,0.00,'2024-03-18','2024-03-16',None,1,5,4700.00),  # null
        ('CLM0053','CUST059','POL038','Motor','Closed','ADJ004','Dublin','PRV004',7400.00,7700.00,6900.00,'2023-10-16','2023-10-14','2024-07-08',0,0,2500.00),
        ('CLM0054','CUST036','POL010','Health','Pending','ADJ005','Cork','PRV001',3100.00,3300.00,700.00,'2023-12-20','2023-12-18',None,0,2,1000.00),
        ('CLM0055','CUST018','POL020','Liability','Closed','ADJ001','Waterford','PRV003',25600.00,26500.00,24000.00,'2023-04-28','2023-04-25','2024-03-05',0,3,4600.00),
        ('CLM0056','CUST043','POL037','Motor','Open','ADJ002','Dublin','PRV002',8300.00,8700.00,0.00,'2024-06-14','2024-06-12',None,0,1,2700.00),
        ('CLM0057','CUST027','POL027','Property','Closed','ADJ003','Cork',None,21000.00,21800.00,19500.00,'2023-07-22','2023-07-20','2024-05-18',0,2,4400.00),
        ('CLM0058','CUST005','POL009','Health','Open','ADJ004','Galway','PRV004',2600.00,2800.00,0.00,'2024-02-05','2024-02-03',None,0,0,870.00),
        ('CLM0059','CUST048','POL038','Liability','Closed','ADJ005','Limerick','PRV001',41000.00,43000.00,38500.00,'2023-09-05','2023-09-03','2024-08-01',1,5,4900.00),
        ('CLM0060','CUST030','POL004','Motor','Open','ADJ001','Dublin','PRV003',5100.00,5300.00,0.00,'2024-05-28','2024-05-26',None,0,0,1900.00),
        ('CLM0061','CUST001','POL001','Travel','Closed','ADJ002','Cork','PRV002',750.00,800.00,700.00,'2023-10-10','2023-10-08','2023-12-20',0,0,480.00),
        ('CLM0062','CUST049','POL029','Motor','Open','ADJ003','Waterford','PRV004',12600.00,13200.00,0.00,'2024-01-08','2024-01-06',None,1,3,3500.00),
        ('CLM0063','CUST034','POL015','Property','Closed','ADJ004','Galway',None,16800.00,17500.00,15600.00,'2023-05-15','2023-05-12','2024-02-20',0,1,4100.00),
        ('CLM0064','CUST012','POL021','Health','Pending','ADJ005','Dublin','PRV001',4500.00,4700.00,1000.00,'2023-11-25','2023-11-22',None,0,1,1250.00),
        ('CLM0065','CUST055','POL016','Liability','Open','ADJ001','Cork','PRV003',None,35000.00,0.00,'2024-04-18','2024-04-16',None,0,2,4700.00),  # null
        ('CLM0066','CUST020','POL033','Motor','Closed','ADJ002','Limerick','PRV002',9700.00,10100.00,9100.00,'2023-08-28','2023-08-26','2024-06-10',0,4,3000.00),
        ('CLM0067','CUST038','POL026','Property','Open','ADJ003','Dublin',None,24500.00,25500.00,0.00,'2024-03-05','2024-03-03',None,1,2,4400.00),
        ('CLM0068','CUST051','POL039','Health','Closed','ADJ004','Cork','PRV004',3300.00,3500.00,3000.00,'2023-12-12','2023-12-10','2024-08-15',0,0,1050.00),
        ('CLM0069','CUST007','POL023','Liability','Open','ADJ005','Waterford','PRV001',48000.00,50000.00,0.00,'2024-02-22','2024-02-20',None,1,6,4900.00),
        ('CLM0070','CUST026','POL006','Motor','Closed','ADJ001','Galway','PRV003',6800.00,7100.00,6300.00,'2023-07-10','2023-07-08','2024-04-25',0,1,2300.00),
        ('CLM0071','CUST044','POL034','Travel','Open','ADJ002','Dublin','PRV002',1400.00,1500.00,0.00,'2024-06-20','2024-06-18',None,0,0,520.00),
        ('CLM0072','CUST003','POL019','Property','Closed','ADJ003','Cork',None,13500.00,14000.00,12700.00,'2023-04-10','2023-04-07','2023-12-28',0,2,3600.00),
        ('CLM0073','CUST056','POL014','Motor','Open','ADJ004','Limerick','PRV004',10200.00,10700.00,0.00,'2024-01-28','2024-01-26',None,0,3,3200.00),
        ('CLM0074','CUST009','POL005','Health','Closed','ADJ005','Galway','PRV001',5900.00,6200.00,5500.00,'2023-09-24','2023-09-22','2024-07-30',0,1,1700.00),
        ('CLM0075','CUST042','POL030','Liability','Open','ADJ001','Dublin','PRV003',58000.00,61000.00,0.00,'2024-04-08','2024-04-06',None,1,7,4900.00),
        ('CLM0076','CUST017','POL017','Motor','Closed','ADJ002','Cork','PRV002',7300.00,7600.00,6800.00,'2023-06-25','2023-06-23','2024-03-30',0,0,2400.00),
        ('CLM0077','CUST035','POL024','Property','Open','ADJ003','Waterford',None,None,19000.00,0.00,'2024-05-14','2024-05-12',None,0,1,4000.00),  # null
        ('CLM0078','CUST060','POL040','Health','Closed','ADJ004','Galway','PRV004',4200.00,4400.00,3900.00,'2023-11-06','2023-11-04','2024-09-05',0,2,1300.00),
        ('CLM0079','CUST023','POL031','Liability','Open','ADJ005','Dublin','PRV001',None,32000.00,0.00,'2024-03-30','2024-03-28',None,1,4,4800.00),  # null
        ('CLM0080','CUST028','POL007','Motor','Closed','ADJ001','Cork','PRV003',8600.00,9000.00,8000.00,'2023-08-02','2023-07-31','2024-05-14',0,3,2800.00),
        # Additional rows to complete 200-row dataset
        ('CLM0081','CUST015','POL002','Travel','Open','ADJ002','Limerick','PRV002',1600.00,1700.00,0.00,'2024-06-28','2024-06-26',None,0,0,590.00),
        ('CLM0082','CUST053','POL013','Motor','Closed','ADJ003','Dublin','PRV004',11800.00,12300.00,11000.00,'2023-03-20','2023-03-18','2023-12-08',0,2,3400.00),
        ('CLM0083','CUST030','POL004','Property','Open','ADJ004','Cork',None,32000.00,33500.00,0.00,'2024-02-14','2024-02-12',None,1,3,4600.00),
        ('CLM0084','CUST046','POL032','Health','Closed','ADJ005','Galway','PRV001',3800.00,4000.00,3500.00,'2023-10-22','2023-10-20','2024-08-18',0,0,1150.00),
        ('CLM0085','CUST001','POL001','Liability','Open','ADJ001','Waterford','PRV003',43500.00,45500.00,0.00,'2024-04-22','2024-04-20',None,0,1,4800.00),
        ('CLM0086','CUST040','POL036','Motor','Closed','ADJ002','Dublin','PRV002',6200.00,6500.00,5800.00,'2023-07-18','2023-07-16','2024-04-04',0,4,2100.00),
        ('CLM0087','CUST016','POL017','Property','Open','ADJ003','Cork',None,None,16000.00,0.00,'2024-05-24','2024-05-22',None,0,2,3900.00),  # null
        ('CLM0088','CUST057','POL036','Health','Closed','ADJ004','Limerick','PRV004',5100.00,5400.00,4700.00,'2023-12-04','2023-12-02','2024-09-20',0,1,1550.00),
        ('CLM0089','CUST024','POL029','Liability','Open','ADJ005','Galway','PRV001',51000.00,54000.00,0.00,'2024-01-16','2024-01-14',None,1,5,4900.00),
        ('CLM0090','CUST038','POL026','Motor','Closed','ADJ001','Dublin','PRV003',9400.00,9800.00,8800.00,'2023-09-28','2023-09-26','2024-06-22',0,2,2900.00),
        ('CLM0091','CUST010','POL008','Travel','Open','ADJ002','Cork','PRV002',1800.00,1900.00,0.00,'2024-07-04','2024-07-02',None,0,0,610.00),
        ('CLM0092','CUST047','POL011','Motor','Closed','ADJ003','Waterford','PRV004',14700.00,15300.00,13800.00,'2023-04-15','2023-04-13','2024-01-28',0,3,4000.00),
        ('CLM0093','CUST020','POL033','Property','Open','ADJ004','Galway',None,27000.00,28500.00,0.00,'2024-03-12','2024-03-10',None,1,2,4500.00),
        ('CLM0094','CUST006','POL015','Health','Closed','ADJ005','Dublin','PRV001',4400.00,4600.00,4100.00,'2023-11-18','2023-11-16','2024-10-01',0,0,1350.00),
        ('CLM0095','CUST050','POL013','Liability','Open','ADJ001','Cork','PRV003',37000.00,39000.00,0.00,'2024-04-30','2024-04-28',None,0,4,4700.00),
        ('CLM0096','CUST032','POL024','Motor','Closed','ADJ002','Limerick','PRV002',7600.00,7900.00,7100.00,'2023-08-08','2023-08-06','2024-05-30',0,1,2500.00),
        ('CLM0097','CUST043','POL037','Property','Open','ADJ003','Dublin',None,None,22000.00,0.00,'2024-06-06','2024-06-04',None,0,3,4200.00),  # null
        ('CLM0098','CUST014','POL016','Health','Closed','ADJ004','Cork','PRV004',3600.00,3800.00,3300.00,'2023-12-28','2023-12-26','2024-10-15',0,1,1100.00),
        ('CLM0099','CUST027','POL027','Liability','Open','ADJ005','Waterford','PRV001',None,29000.00,0.00,'2024-02-08','2024-02-06',None,1,6,4800.00),  # null
        ('CLM0100','CUST058','POL039','Motor','Closed','ADJ001','Galway','PRV003',10800.00,11300.00,10100.00,'2023-05-05','2023-05-03','2024-02-12',0,2,3300.00),
        # 3 duplicate rows (CLM0004, CLM0008, CLM0016) — injected at end
        ('CLM0004','CUST015','POL002','Motor','Closed','ADJ001','Cork','PRV004',14335.58,5579.83,341.31,'2023-01-19','2023-04-03','2023-10-21',0,5,2846.57),
        ('CLM0008','CUST022','POL018','Motor','Closed','ADJ003','Dublin','PRV002',52341.90,55000.00,48000.00,'2023-05-22','2023-05-20','2024-02-14',1,3,4100.00),
        ('CLM0016','CUST039','POL033','Property','Closed','ADJ001','Cork','PRV004',6700.00,7000.00,6200.00,'2023-06-14','2023-06-10','2024-01-20',0,1,2100.00),
    ]
    
    cols_claims = [
        'claim_id','claimant_id','policy_id','claim_type','claim_status',
        'adjuster_id','region','provider_id','claim_amount','reserve_amount',
        'paid_amount','fnol_date','loss_date','closure_date',
        'fraud_flag','num_prev_claims','premium'
    ]
    
    df = pd.DataFrame(claims_records, columns=cols_claims)
    df['fnol_date']    = pd.to_datetime(df['fnol_date'])
    df['loss_date']    = pd.to_datetime(df['loss_date'])
    df['closure_date'] = pd.to_datetime(df['closure_date'], errors='coerce')
    
    # ── RESERVE HISTORY DATA ─────────────────────────────────────
    reserve_records = [
        ('CLM0001','2023-03-01',2900.00), ('CLM0001','2023-07-15',3100.00), ('CLM0001','2024-01-10',2950.00),
        ('CLM0004','2023-02-01',5200.00), ('CLM0004','2023-06-20',5600.00), ('CLM0004','2023-09-15',5400.00),
        ('CLM0008','2023-06-01',52000.00),('CLM0008','2023-09-10',54000.00),('CLM0008','2024-01-05',55000.00),
        ('CLM0010','2023-03-01',28000.00),('CLM0010','2023-08-15',30500.00),('CLM0010','2024-01-20',30000.00),
        ('CLM0012','2023-05-01',15000.00),('CLM0012','2023-08-20',16200.00),('CLM0012','2023-11-10',16000.00),
        ('CLM0014','2023-09-01',9200.00), ('CLM0014','2023-12-10',9600.00), ('CLM0014','2024-03-05',9800.00),
        ('CLM0016','2023-07-01',6500.00), ('CLM0016','2023-10-15',6900.00), ('CLM0016','2024-01-05',7000.00),
        ('CLM0018','2023-11-01',5200.00), ('CLM0018','2024-02-15',5700.00), ('CLM0018','2024-05-20',5800.00),
        ('CLM0020','2023-04-01',21500.00),('CLM0020','2023-07-10',22800.00),('CLM0020','2023-10-20',23000.00),
        ('CLM0022','2023-08-01',4000.00), ('CLM0022','2023-11-15',4300.00), ('CLM0022','2024-03-01',4400.00),
        ('CLM0024','2023-06-01',2700.00), ('CLM0024','2023-09-05',2900.00), ('CLM0024','2023-11-20',3000.00),
        ('CLM0028','2023-10-01',6200.00), ('CLM0028','2024-01-10',6500.00), ('CLM0028','2024-04-15',6600.00),
        ('CLM0030','2023-05-01',30000.00),('CLM0030','2023-09-20',31500.00),('CLM0030','2024-02-10',32000.00),
        ('CLM0032','2023-03-01',19000.00),('CLM0032','2023-06-15',20200.00),('CLM0032','2023-10-05',20500.00),
        ('CLM0034','2023-11-01',3600.00), ('CLM0034','2024-03-05',3800.00), ('CLM0034','2024-06-20',3900.00),
        ('CLM0036','2023-07-01',10200.00),('CLM0036','2023-10-10',10800.00),('CLM0036','2024-02-25',11000.00),
        ('CLM0039','2023-06-01',8600.00), ('CLM0039','2023-09-15',9000.00), ('CLM0039','2024-01-05',9200.00),
        ('CLM0042','2023-08-01',370000.00),('CLM0042','2023-11-20',385000.00),('CLM0042','2024-05-10',390000.00),
        ('CLM0045','2023-12-01',1200.00), ('CLM0045','2024-01-15',1280.00), ('CLM0045','2024-02-01',1300.00),
        ('CLM0047','2023-04-01',13800.00),('CLM0047','2023-07-20',14500.00),('CLM0047','2023-11-10',14800.00),
        ('CLM0049','2023-09-01',4600.00), ('CLM0049','2023-12-15',4900.00), ('CLM0049','2024-05-01',5000.00),
        ('CLM0051','2023-07-01',17000.00),('CLM0051','2023-10-20',17800.00),('CLM0051','2024-03-25',18000.00),
        ('CLM0053','2023-11-01',7200.00), ('CLM0053','2024-02-10',7500.00), ('CLM0053','2024-06-20',7700.00),
        ('CLM0055','2023-05-01',24500.00),('CLM0055','2023-09-15',26000.00),('CLM0055','2024-02-25',26500.00),
        ('CLM0057','2023-08-01',20000.00),('CLM0057','2023-11-20',21200.00),('CLM0057','2024-05-01',21800.00),
        ('CLM0059','2023-10-01',40000.00),('CLM0059','2024-01-15',42000.00),('CLM0059','2024-07-20',43000.00),
        ('CLM0063','2023-05-01',13000.00),('CLM0063','2023-08-15',13800.00),('CLM0063','2024-02-01',14000.00),
        ('CLM0066','2023-09-01',9300.00), ('CLM0066','2023-12-10',9800.00), ('CLM0066','2024-05-25',10100.00),
        ('CLM0068','2024-01-01',3200.00), ('CLM0068','2024-04-15',3400.00), ('CLM0068','2024-08-01',3500.00),
        ('CLM0070','2023-08-01',6500.00), ('CLM0070','2023-11-20',6900.00), ('CLM0070','2024-04-10',7100.00),
        ('CLM0072','2023-05-01',12800.00),('CLM0072','2023-08-15',13500.00),('CLM0072','2023-12-20',14000.00),
        ('CLM0074','2023-10-01',5600.00), ('CLM0074','2024-01-15',6000.00), ('CLM0074','2024-07-15',6200.00),
        ('CLM0076','2023-07-01',6900.00), ('CLM0076','2023-10-20',7300.00), ('CLM0076','2024-03-15',7600.00),
        ('CLM0078','2023-12-01',4000.00), ('CLM0078','2024-03-10',4300.00), ('CLM0078','2024-08-25',4400.00),
        ('CLM0080','2023-09-01',8200.00), ('CLM0080','2023-12-15',8700.00), ('CLM0080','2024-05-01',9000.00),
        ('CLM0082','2023-04-01',11200.00),('CLM0082','2023-07-20',11900.00),('CLM0082','2023-11-25',12300.00),
        ('CLM0084','2023-11-01',3700.00), ('CLM0084','2024-02-15',3900.00), ('CLM0084','2024-07-28',4000.00),
        ('CLM0086','2023-08-01',5900.00), ('CLM0086','2023-11-10',6200.00), ('CLM0086','2024-03-20',6500.00),
        ('CLM0088','2024-01-01',4900.00), ('CLM0088','2024-04-20',5200.00), ('CLM0088','2024-09-05',5400.00),
        ('CLM0090','2023-10-01',9000.00), ('CLM0090','2024-01-20',9500.00), ('CLM0090','2024-06-08',9800.00),
        ('CLM0092','2023-05-01',14200.00),('CLM0092','2023-08-15',14900.00),('CLM0092','2024-01-10',15300.00),
        ('CLM0094','2023-12-01',4200.00), ('CLM0094','2024-03-15',4500.00), ('CLM0094','2024-09-16',4600.00),
        ('CLM0096','2023-09-01',7300.00), ('CLM0096','2023-12-20',7700.00), ('CLM0096','2024-05-15',7900.00),
        ('CLM0098','2024-01-01',3500.00), ('CLM0098','2024-04-10',3700.00), ('CLM0098','2024-10-01',3800.00),
        ('CLM0100','2023-06-01',10300.00),('CLM0100','2023-09-15',10900.00),('CLM0100','2024-01-28',11300.00),
    ]
    
    rh = pd.DataFrame(reserve_records, columns=['claim_id','update_date','reserve_amount'])
    rh['update_date'] = pd.to_datetime(rh['update_date'])
    
    print("=" * 65)
    print(f"Dataset loaded  |  Claims: {len(df)} rows  |  Reserve history: {len(rh)} rows")
    print("=" * 65)
    print("\nFirst 5 rows:")
    print(df[['claim_id','claim_type','claim_status','claim_amount','fnol_date','fraud_flag']].head(5).to_string())
    
    
    
    
    
    
    # ============================================================
    # SECTION 1: DATA QUALITY — 6 DIMENSIONS
    # ============================================================
    
    print("\n\n" + "=" * 65)
    print("SECTION 1: DATA QUALITY DIMENSIONS")
    print("=" * 65)
    
    
    # ── 1A. COMPLETENESS ────────────────────────────────────────
    print("\n── 1A. COMPLETENESS ──")
    print("Which columns have nulls? What percentage?\n")
    
    null_report = pd.DataFrame({
        'null_count': df.isnull().sum(),
        'null_pct':  (df.isnull().sum() / len(df) * 100).round(2)
    })
    print(null_report[null_report['null_count'] > 0].to_string())
    print(f"\nOverall completeness: {(1 - df.isnull().mean()).mean()*100:.1f}%")
    # Expected: claim_amount ~9 nulls, closure_date ~many nulls, provider_id some nulls
    
    
    # ── 1B. UNIQUENESS ──────────────────────────────────────────
    print("\n── 1B. UNIQUENESS ──")
    print("Find duplicate claim_ids (3 injected: CLM0004, CLM0008, CLM0016)\n")
    
    dupes = df[df.duplicated(subset='claim_id', keep=False)][
        ['claim_id','claimant_id','claim_type','fnol_date']
    ].sort_values('claim_id')
    print(f"Duplicate claim_ids: {df['claim_id'].duplicated().sum()}")
    print(dupes.to_string(index=False))
    
    df_dedup = df.drop_duplicates(subset='claim_id', keep='last').copy()
    print(f"\nAfter dedup: {len(df_dedup)} rows (removed {len(df) - len(df_dedup)})")
    
    
    # ── 1C. VALIDITY ────────────────────────────────────────────
    print("\n── 1C. VALIDITY ──")
    print("Business rule violations:\n")
    
    checks = {
        'negative_claim_amount':  int((df['claim_amount'] <= 0).sum()),
        'future_fnol_date':       int((df['fnol_date'] > pd.Timestamp('2025-01-01')).sum()),
        'closure_before_fnol':    int((df['closure_date'] < df['fnol_date']).sum()),
        'reserve_below_paid':     int((df['reserve_amount'] < df['paid_amount']).sum()),
    }
    for k, v in checks.items():
        print(f"  {'OK ' if v==0 else 'FAIL'} | {k}: {v}")
    # Expected: 2 negative, 1 future (CLM0002 2026-08-15), 1 closure before fnol (CLM0003)
    
    
    # ── 1D. CONSISTENCY ─────────────────────────────────────────
    print("\n── 1D. CONSISTENCY ──")
    print("Closed claims must have closure_date; Open claims should not\n")
    
    by_status = df.groupby('claim_status').agg(
        total=('claim_id','count'),
        missing_closure=('closure_date', lambda x: x.isna().sum()),
        has_closure=('closure_date', lambda x: x.notna().sum())
    )
    by_status['closed_no_date'] = by_status.apply(
        lambda r: r['missing_closure'] if r.name == 'Closed' else 0, axis=1)
    by_status['open_with_date'] = by_status.apply(
        lambda r: r['has_closure'] if r.name == 'Open' else 0, axis=1)
    print(by_status[['total','closed_no_date','open_with_date']].to_string())
    
    
    # ── 1E. TIMELINESS ──────────────────────────────────────────
    print("\n── 1E. TIMELINESS ──")
    print("SLA ageing for open claims (reference date: 2025-04-24)\n")
    
    today = pd.Timestamp('2025-04-24')
    open_c = df[df['claim_status'] == 'Open'].copy()
    open_c['days_open'] = (today - open_c['fnol_date']).dt.days
    open_c['sla_status'] = pd.cut(
        open_c['days_open'],
        bins=[-np.inf, 90, 180, np.inf],
        labels=['Within SLA', 'SLA Breach', 'Critical']
    )
    print("SLA summary:"); print(open_c['sla_status'].value_counts().to_string())
    print("\nTop 10 oldest open claims:")
    print(open_c[['claim_id','claim_type','adjuster_id','days_open','sla_status']]
          .sort_values('days_open', ascending=False).head(10).to_string(index=False))
    
    
    # ── 1F. ACCURACY ────────────────────────────────────────────
    print("\n── 1F. ACCURACY — Distribution profile ──\n")
    
    clean = df[df['claim_amount'].notna() & (df['claim_amount'] > 0)]
    print(clean['claim_amount'].describe(percentiles=[.25,.5,.75,.95,.99]).round(2).to_string())
    print("\nBy claim type:")
    print(clean.groupby('claim_type')['claim_amount'].agg(['count','mean','median','max']).round(2).to_string())
    
    
    # ── 1G. DQ SCORECARD ────────────────────────────────────────
    print("\n── 1G. FULL DQ SCORECARD ──\n")
    
    total = len(df)
    scorecard = pd.DataFrame([
        {'check': 'Completeness — claim_amount',   'score': round(df['claim_amount'].notna().sum()/total*100,1), 'threshold': 95.0},
        {'check': 'Uniqueness — claim_id',          'score': round(df['claim_id'].nunique()/total*100,1),         'threshold': 99.0},
        {'check': 'Validity — positive amounts',    'score': round(((df['claim_amount']>0)|df['claim_amount'].isna()).sum()/total*100,1), 'threshold': 98.0},
        {'check': 'Consistency — closed has date',  'score': round(((df['claim_status']!='Closed')|df['closure_date'].notna()).sum()/total*100,1), 'threshold': 95.0},
        {'check': 'Validity — closure after fnol',  'score': round((df['closure_date'].isna()|(df['closure_date']>=df['fnol_date'])).sum()/total*100,1), 'threshold': 99.0},
    ])
    scorecard['result'] = scorecard.apply(lambda r: 'PASS' if r['score'] >= r['threshold'] else 'FAIL', axis=1)
    print(scorecard.to_string(index=False))
    print(f"\nResult: {(scorecard['result']=='PASS').sum()}/{len(scorecard)} checks passing")
    
    
    # ============================================================
    # SECTION 2: OUTLIER DETECTION
    # ============================================================
    
    print("\n\n" + "=" * 65)
    print("SECTION 2: OUTLIER DETECTION")
    print("=" * 65)
    
    clean = df[df['claim_amount'].notna() & (df['claim_amount'] > 0)].copy()
    
    
    # ── 2A. IQR ─────────────────────────────────────────────────
    print("\n── 2A. IQR METHOD (best for right-skewed claims data) ──\n")
    
    Q1, Q3  = clean['claim_amount'].quantile([0.25, 0.75])
    IQR     = Q3 - Q1
    lower   = Q1 - 1.5 * IQR
    upper   = Q3 + 1.5 * IQR
    print(f"Q1={Q1:,.0f}  Q3={Q3:,.0f}  IQR={IQR:,.0f}  Lower={lower:,.0f}  Upper={upper:,.0f}")
    
    outliers_iqr = clean[(clean['claim_amount'] < lower) | (clean['claim_amount'] > upper)]
    print(f"Outliers detected: {len(outliers_iqr)}")
    print(outliers_iqr[['claim_id','claim_type','claim_amount']].sort_values('claim_amount',ascending=False).to_string(index=False))
    # Expected: CLM0042 (380k), CLM0041 (250k) and others
    
    
    # ── 2B. Z-SCORE ─────────────────────────────────────────────
    print("\n── 2B. Z-SCORE METHOD ──\n")
    
    from scipy import stats as scipy_stats
    clean['z_score'] = np.abs(scipy_stats.zscore(clean['claim_amount']))
    outliers_z = clean[clean['z_score'] > 3]
    print(f"Z-score outliers (|z|>3): {len(outliers_z)}")
    print(outliers_z[['claim_id','claim_type','claim_amount','z_score']].sort_values('z_score',ascending=False).to_string(index=False))
    print(f"\nIQR={len(outliers_iqr)} vs Z-score={len(outliers_z)}  (IQR catches more for skewed data)")
    
    
    # ── 2C. THREE HANDLING STRATEGIES ───────────────────────────
    print("\n── 2C. HANDLING STRATEGIES ──\n")
    
    df['outlier_flag']         = ((df['claim_amount'].notna()) & (df['claim_amount'] > upper)).astype(int)
    df['claim_amount_capped']  = df['claim_amount'].clip(lower=lower, upper=upper)
    df_removed                 = df[(df['claim_amount'].isna()) | ((df['claim_amount'] >= lower) & (df['claim_amount'] <= upper))]
    
    print(f"Flag   — {df['outlier_flag'].sum()} records flagged (data preserved) ← best for insurance")
    print(f"Cap    — max after capping: {df['claim_amount_capped'].max():,.2f}")
    print(f"Remove — {len(df)-len(df_removed)} rows dropped (avoid: £500k may be legitimate)")
    
    
    # ── 2D. KS TEST — DRIFT ─────────────────────────────────────
    print("\n── 2D. DATA DRIFT — KS TEST ──\n")
    
    early  = df[df['fnol_date'] < pd.Timestamp('2023-07-01')]['claim_amount'].dropna()
    recent = df[df['fnol_date'] > pd.Timestamp('2024-06-01')]['claim_amount'].dropna()
    if len(early) > 5 and len(recent) > 5:
        stat, pval = scipy_stats.ks_2samp(early, recent)
        print(f"Early cohort n={len(early)}  |  Recent cohort n={len(recent)}")
        print(f"KS stat={stat:.3f}  p-value={pval:.4f}")
        print(f"Drift: {'YES — investigate' if pval < 0.05 else 'No significant drift'}")
    
    
    # ============================================================
    # SECTION 3: WINDOW FUNCTION EQUIVALENTS IN PANDAS
    # ============================================================
    
    print("\n\n" + "=" * 65)
    print("SECTION 3: WINDOW FUNCTION EQUIVALENTS IN PANDAS")
    print("=" * 65)
    
    work = df[df['claim_amount'].notna() & (df['claim_amount'] > 0)].copy()
    
    
    # ── 3A. ROW_NUMBER ──────────────────────────────────────────
    print("\n── 3A. ROW_NUMBER — Top claim per claimant ──")
    print("SQL: ROW_NUMBER() OVER (PARTITION BY claimant_id ORDER BY claim_amount DESC)\n")
    
    top = (work.sort_values('claim_amount', ascending=False)
               .groupby('claimant_id').first().reset_index()
               [['claimant_id','claim_id','claim_type','claim_amount']]
               .sort_values('claim_amount', ascending=False))
    print(top.head(10).to_string(index=False))
    
    
    # ── 3B. RANK & DENSE_RANK ───────────────────────────────────
    print("\n── 3B. RANK vs DENSE_RANK — Adjuster rankings ──")
    print("SQL: RANK() / DENSE_RANK() OVER (ORDER BY claim_count DESC)\n")
    
    adj = work.groupby('adjuster_id').size().reset_index(name='claim_count')
    adj['rank_with_gap'] = adj['claim_count'].rank(method='min',   ascending=False).astype(int)
    adj['rank_no_gap']   = adj['claim_count'].rank(method='dense', ascending=False).astype(int)
    print(adj.sort_values('claim_count', ascending=False).to_string(index=False))
    print("\nNote: rank(method='min') = RANK()  |  rank(method='dense') = DENSE_RANK()")
    
    
    # ── 3C. LAG & LEAD ──────────────────────────────────────────
    print("\n── 3C. LAG — Repeat claims within 30 days (fraud signal) ──")
    print("SQL: LAG(fnol_date) OVER (PARTITION BY claimant_id ORDER BY fnol_date)\n")
    
    lag_df = work.sort_values(['claimant_id','fnol_date']).copy()
    lag_df['prev_fnol']   = lag_df.groupby('claimant_id')['fnol_date'].shift(1)
    lag_df['days_since']  = (lag_df['fnol_date'] - lag_df['prev_fnol']).dt.days
    
    repeat = lag_df[lag_df['days_since'].notna() & (lag_df['days_since'] <= 30)]
    print(f"Repeat claims within 30 days: {len(repeat)}")
    if len(repeat):
        print(repeat[['claim_id','claimant_id','fnol_date','prev_fnol','days_since','claim_amount']].to_string(index=False))
    
    print("\n── 3C. LEAD — Next claim date per claimant ──")
    print("SQL: LEAD(fnol_date) OVER (PARTITION BY claimant_id ORDER BY fnol_date)\n")
    lag_df['next_fnol']  = lag_df.groupby('claimant_id')['fnol_date'].shift(-1)
    lag_df['next_type']  = lag_df.groupby('claimant_id')['claim_type'].shift(-1)
    has_next = lag_df[lag_df['next_fnol'].notna()][['claim_id','claimant_id','fnol_date','next_fnol','next_type']]
    print(has_next.head(10).to_string(index=False))
    
    
    # ── 3D. RUNNING TOTAL ───────────────────────────────────────
    print("\n── 3D. RUNNING TOTAL — Cumulative claims by month ──")
    print("SQL: SUM(x) OVER (ORDER BY month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)\n")
    
    monthly = (work.groupby(work['fnol_date'].dt.to_period('M'))['claim_amount']
                   .agg(monthly_total='sum', claim_count='count').reset_index())
    monthly.columns = ['month','monthly_total','claim_count']
    monthly['running_total']  = monthly['monthly_total'].cumsum().round(2)
    monthly['running_claims'] = monthly['claim_count'].cumsum()
    print(monthly.to_string(index=False))
    
    
    # ── 3E. ROLLING AVERAGE ─────────────────────────────────────
    print("\n── 3E. ROLLING AVERAGE — 3-month rolling avg ──")
    print("SQL: AVG(x) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)\n")
    
    monthly['avg_monthly']      = work.groupby(work['fnol_date'].dt.to_period('M'))['claim_amount'].mean().values
    monthly['rolling_3mo_avg']  = monthly['avg_monthly'].rolling(3).mean().round(2)
    monthly['rolling_12mo_avg'] = monthly['avg_monthly'].rolling(12).mean().round(2)
    print(monthly[['month','avg_monthly','rolling_3mo_avg','rolling_12mo_avg']].to_string(index=False))
    print("\nNote: rolling(3) = ROWS BETWEEN 2 PRECEDING AND CURRENT ROW")
    
    
    # ── 3F. NTILE ───────────────────────────────────────────────
    print("\n── 3F. NTILE(4) — Risk quartile banding ──")
    print("SQL: NTILE(4) OVER (ORDER BY total_spend DESC)  ←→  Pandas: pd.qcut(q=4)\n")
    
    spend = work.groupby('claimant_id').agg(
        total_spend=('claim_amount','sum'),
        claim_count=('claim_id','count'),
        fraud_count=('fraud_flag','sum')
    ).reset_index()
    spend['spend_quartile'] = pd.qcut(spend['total_spend'], q=4,
                                       labels=['Low Risk','Medium-Low','Medium-High','High Risk'])
    print(spend.sort_values('total_spend', ascending=False).head(15).to_string(index=False))
    print("\nAverage by risk band:")
    print(spend.groupby('spend_quartile')[['total_spend','claim_count','fraud_count']].mean().round(2).to_string())
    print("\nNote: pd.qcut = equal-frequency (NTILE)  |  pd.cut = equal-width (different!)")
    
    
    # ── 3G. FIRST_VALUE & LAST_VALUE ────────────────────────────
    print("\n── 3G. FIRST_VALUE & LAST_VALUE — Reserve development ──")
    print("SQL: FIRST_VALUE/LAST_VALUE OVER (PARTITION BY claim_id ORDER BY update_date")
    print("     ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)\n")
    
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
    print("\nKEY GOTCHA: In SQL, LAST_VALUE without frame = current row. Always add")
    print("  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING")
    print("In Pandas: .groupby().last() always gives the true last value — no gotcha.")
    
    
    # ── 3H. PERCENT_RANK ────────────────────────────────────────
    print("\n── 3H. PERCENT_RANK — Claim percentile within type ──")
    print("SQL: PERCENT_RANK() OVER (PARTITION BY claim_type ORDER BY claim_amount)")
    print("Pandas: .groupby().rank(pct=True)\n")
    
    work['pct_rank_in_type'] = (
        work.groupby('claim_type')['claim_amount']
        .rank(pct=True).mul(100).round(1)
    )
    # Top 5% per type — flag for SIU review
    top5 = work[work['pct_rank_in_type'] >= 95][
        ['claim_id','claim_type','claim_amount','pct_rank_in_type','fraud_flag']
    ].sort_values(['claim_type','claim_amount'], ascending=[True,False])
    print(f"Claims in top 5% within type (SIU candidates): {len(top5)}")
    print(top5.to_string(index=False))
    
    
    # ── 3I. COMBINED FRAUD SCORE ────────────────────────────────
    print("\n── 3I. COMBINED: Fraud risk score ──\n")
    
    scored = work.sort_values(['claimant_id','fnol_date']).copy()
    scored['prev_fnol']       = scored.groupby('claimant_id')['fnol_date'].shift(1)
    scored['days_since_last'] = (scored['fnol_date'] - scored['prev_fnol']).dt.days
    scored['pct_rank']        = scored.groupby('claim_type')['claim_amount'].rank(pct=True).mul(100)
    scored['claim_seq']       = scored.groupby('claimant_id').cumcount() + 1
    
    scored['fraud_risk_score'] = (
        (scored['pct_rank'] >= 95).astype(int)              * 3 +
        (scored['days_since_last'].fillna(999) <= 30).astype(int) * 3 +
        scored['fraud_flag']                                 * 2 +
        (scored['num_prev_claims'] > 4).astype(int)          * 1
    )
    top_fraud = (scored.sort_values('fraud_risk_score', ascending=False)
                 [['claim_id','claimant_id','claim_type','claim_amount',
                   'pct_rank','days_since_last','fraud_flag','fraud_risk_score']]
                 .head(15))
    print(top_fraud.to_string(index=False))
    
    
    # ============================================================
    # SECTION 4: INSURANCE METRICS
    # ============================================================
    
    print("\n\n" + "=" * 65)
    print("SECTION 4: INSURANCE METRICS")
    print("=" * 65)
    
    
    # ── 4A. LOSS RATIO ──────────────────────────────────────────
    print("\n── 4A. LOSS RATIO by claim type ──")
    print("Formula: paid_amount / premium × 100  |  >100% = unprofitable\n")
    
    lr = df.groupby('claim_type').apply(lambda g: pd.Series({
        'claims':        len(g),
        'total_paid':    round(g['paid_amount'].sum(), 2),
        'total_premium': round(g['premium'].sum(), 2),
        'loss_ratio':    round(g['paid_amount'].sum() / g['premium'].sum() * 100, 1)
    })).reset_index()
    lr['status'] = lr['loss_ratio'].apply(lambda x: 'Unprofitable' if x>100 else ('Watch' if x>70 else 'Healthy'))
    print(lr.sort_values('loss_ratio', ascending=False).to_string(index=False))
    
    
    # ── 4B. CYCLE TIME ──────────────────────────────────────────
    print("\n── 4B. CYCLE TIME — FNOL to closure ──")
    print("Always show mean AND median — claims data is right-skewed\n")
    
    closed = df[(df['claim_status']=='Closed') & df['closure_date'].notna() & df['fnol_date'].notna()].copy()
    closed['cycle_days'] = (closed['closure_date'] - closed['fnol_date']).dt.days
    cycle = closed.groupby('claim_type')['cycle_days'].agg(
        count='count',
        mean=lambda x: round(x.mean(),1),
        median=lambda x: round(x.median(),1),
        max='max'
    ).reset_index()
    print(cycle.sort_values('mean', ascending=False).to_string(index=False))
    
    
    # ── 4C. COHORT ──────────────────────────────────────────────
    print("\n── 4C. COHORT — Monthly resolution rate ──\n")
    
    df['cohort_month'] = df['fnol_date'].dt.to_period('M')
    cohort = df.groupby('cohort_month').agg(
        total=('claim_id','count'),
        closed=('claim_status', lambda x: (x=='Closed').sum())
    ).reset_index()
    cohort['resolution_pct'] = (cohort['closed'] / cohort['total'] * 100).round(1)
    cohort['running_total']  = cohort['total'].cumsum()
    print(cohort.to_string(index=False))
    
    
    # ── 4D. PROVIDER FRAUD ──────────────────────────────────────
    print("\n── 4D. PROVIDER FRAUD ANALYSIS ──\n")
    
    prov = df[df['provider_id'].notna()].groupby('provider_id').agg(
        total_claims =('claim_id','count'),
        fraud_count  =('fraud_flag','sum'),
        avg_claim    =('claim_amount', lambda x: round(x.mean(),2)),
        total_claimed=('claim_amount', lambda x: round(x.sum(),2))
    ).reset_index()
    prov['fraud_rate_pct'] = (prov['fraud_count']/prov['total_claims']*100).round(1)
    prov['spend_rank']     = prov['total_claimed'].rank(ascending=False, method='dense').astype(int)
    print(prov.sort_values('total_claimed', ascending=False).to_string(index=False))
    
    
    # ============================================================
    print("\n\n" + "=" * 65)
    print("ALL EXERCISES COMPLETE")
    print("=" * 65)
    print("""
    WEEKEND CHALLENGE EXERCISES:
      1. Completeness: add 'num_prev_claims' and 'premium' null checks to 1A
      2. Validity: add a check for claim_amount > premium (possible overbilling)
      3. Window: add a COUNT(*) OVER (PARTITION BY region) column to the fraud score
      4. Outliers: run IQR separately for each claim_type (not overall)
      5. Metrics: calculate combined_ratio = loss_ratio + 30% (assume fixed expense ratio)
    
    KEY PHRASES FOR YOUR ASSESSMENT:
      shift(1) = LAG(1)  |  shift(-1) = LEAD(1)
      pd.qcut(q=4) = NTILE(4)  |  pd.cut = equal-width bins (different!)
      rank(method='min') = RANK()  |  rank(method='dense') = DENSE_RANK()
      rolling(3) = ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
      In SQL, LAST_VALUE needs explicit frame — Pandas .last() does not.
    """)
