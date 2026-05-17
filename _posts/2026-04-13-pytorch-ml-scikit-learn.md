---
title: "PyTorch_ML_Scikit-Learn"
date: 2026-04-13 10:32:04 
author: shajeebhameed
layout: page
slug: pytorch_ml_scikit-learn
status: publish
---

This is a meaty request — I'll build you a comprehensive interactive wiki/study guide as a React artifact covering all four areas. Let me put this together.Here's your complete offline study wiki. Four tabs, all copy-paste ready:

**Tab 1 — ML & Deployment** covers three full working code templates: a Scikit-Learn Pipeline with feature engineering and evaluation, a Keras/TensorFlow binary classifier with callbacks, and a PyTorch training loop from scratch. Plus an Azure ML deployment pattern and a PSI drift monitoring snippet — all in insurance/fraud contexts that mirror your actual CV.

**Tab 2 — STAR Stories** gives you three polished stories with the specific numbers (24hr→15min, $2M, 100+ hours/month, 40% reduction, 25% optimisation) already woven in. The Copy button on each card pulls clean plain text you can paste into a notes app.

**Tab 3 — EY AI Practice** covers EY.ai, their Ireland sector focus, EU AI Act positioning, and four smart questions to ask your interviewer that signal you've done real research.

**Tab 4 — CV Gap Analysis** flags the TF/PyTorch "foundations" wording as your main risk, and gives you exact language to handle it if pressed. Also includes a ready-to-use opening positioning statement.

One priority before the interview: build one small end-to-end GenAI/RAG prototype (even a simple one over a PDF) so you can speak to it concretely rather than referencing it abstractly from your CV.

## EY Data Scientist — Interview Prep Wiki

> Complete offline study guide for Shajeeb S | EY AI & Data Practice Ireland

* * *

## Table of Contents

  1. [CV Gap Analysis vs Job Spec](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#1-cv-gap-analysis-vs-job-spec>)
  2. [ML & Deployment — End-to-End Projects](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#2-ml--deployment--end-to-end-projects>)
     * [2A. Scikit-Learn Pipeline](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#2a-scikit-learn-end-to-end-pipeline>)
     * [2B. TensorFlow / Keras](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#2b-tensorflow--keras-end-to-end>)
     * [2C. PyTorch](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#2c-pytorch-end-to-end>)
     * [2D. Azure ML Deployment](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#2d-azure-ml-deployment-pattern>)
     * [2E. Model Monitoring & Drift](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#2e-model-monitoring--drift-detection>)
  3. [STAR Stories](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#3-star-stories>)
     * [Story 1 — AXA Fraud Detection](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#story-1--axa-fraud-detection-platform>)
     * [Story 2 — J&J Cost Savings](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#story-2--jj-cost-savings-analysis>)
     * [Story 3 — Google Street View](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#story-3--google-street-view-anomaly-detection>)
  4. [EY AI & Data Practice Research](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#4-ey-ai--data-practice-research>)
  5. [Key Concepts Cheat Sheet](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#5-key-concepts-cheat-sheet>)
  6. [One-Line Positioning Statement](<https://claude.ai/chat/fbe832ba-627c-4af8-a0ff-2e5ba89f5283#6-one-line-positioning-statement>)



* * *

## 1\. CV Gap Analysis vs Job Spec

### Strong matches — lead with these

Your Experience| Job Spec Requirement  
---|---  
AXA fraud detection ML pipeline| ML model end-to-end management  
PySpark anomaly detection models| Machine learning and AI  
Azure Fabric, Azure ML, Purview| Cloud expertise (Azure)  
BigQuery, Vertex AI| Cloud expertise (GCP)  
Delta Lake feature store| Data pipeline management  
CDC streams (24hr → 15min latency)| Data pipeline management  
Snowflake Bronze-Silver-Gold lakehouse| Big data analysis  
FCA/CBI model explainability/lineage| Regulatory compliance  
C-suite Power BI dashboards| Communication competency  
LLM/RAG/Agent workflows (CV listed)| Generative AI  
$2M cost savings via regression analysis| Statistical modelling  
  
### Gaps to address proactively

**1\. TensorFlow/PyTorch listed as "foundations" in your CV**

The spec requires proficiency. Study Modules 2B and 2C below. In interview, use this framing:

> "My primary framework has been Scikit-Learn for tabular ML — which dominates insurance use cases — but I have built classification networks in Keras/TensorFlow and I understand the PyTorch training loop pattern. I'm actively deepening this."

**2\. MLOps tooling (MLflow not mentioned)**

You have Azure ML deployment experience, which is valid. Know the concept: experiment tracking, model registry, deployment pipelines. In interview:

> "In Azure ML I used the model registry and managed online endpoints — that's Microsoft's integrated MLOps layer, handling versioning, deployment, and monitoring in one platform."

**3\. GenAI listed in CV but no project detail**

If asked, be specific about what you've done. If you haven't built a RAG prototype yet, build a simple one before the interview (PDF over Azure OpenAI + vector search). You need to speak to it concretely.

### How to handle "only 3 years required" when you have 11+

EY specifies 3 years minimum but is hiring for Senior Consultant grade. Frame it positively:

> "I bring senior-level depth — I've operated as the technical lead across full project lifecycles, not just as a contributor. That maps to EY's expectation that Senior Consultants can independently structure and deliver AI workstreams with minimal oversight."

* * *

## 2\. ML & Deployment — End-to-End Projects

* * *

### 2A. Scikit-Learn End-to-End Pipeline

This is the most common "show your work" expectation. Know this pattern cold. Context: fraud detection (mirrors your AXA work).
    
    
    import pandas as pd
    import numpy as np
    from sklearn.model_selection import train_test_split, cross_val_score
    from sklearn.pipeline import Pipeline
    from sklearn.compose import ColumnTransformer
    from sklearn.preprocessing import StandardScaler, OneHotEncoder
    from sklearn.impute import SimpleImputer
    from sklearn.ensemble import GradientBoostingClassifier
    from sklearn.metrics import classification_report, roc_auc_score
    import joblib
    
    # --- 1. LOAD & SPLIT ---
    df = pd.read_csv("claims.csv")
    X = df.drop("fraud_flag", axis=1)
    y = df["fraud_flag"]
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, stratify=y, random_state=42
    )
    
    # --- 2. FEATURE ENGINEERING ---
    num_features = ["claim_amount", "days_to_report", "prior_claims"]
    cat_features = ["policy_type", "region", "channel"]
    
    num_pipe = Pipeline([
        ("impute", SimpleImputer(strategy="median")),
        ("scale",  StandardScaler())
    ])
    cat_pipe = Pipeline([
        ("impute", SimpleImputer(strategy="most_frequent")),
        ("encode", OneHotEncoder(handle_unknown="ignore", sparse_output=False))
    ])
    preprocessor = ColumnTransformer([
        ("num", num_pipe, num_features),
        ("cat", cat_pipe, cat_features)
    ])
    
    # --- 3. MODEL ---
    model = Pipeline([
        ("prep", preprocessor),
        ("clf",  GradientBoostingClassifier(
                    n_estimators=300,
                    max_depth=4,
                    learning_rate=0.05,
                    random_state=42
                ))
    ])
    model.fit(X_train, y_train)
    
    # --- 4. EVALUATE ---
    y_pred = model.predict(X_test)
    y_prob = model.predict_proba(X_test)[:, 1]
    print(classification_report(y_test, y_pred))
    print("ROC-AUC:", roc_auc_score(y_test, y_prob))
    
    # Cross-validation sanity check
    cv_scores = cross_val_score(model, X_train, y_train, cv=5, scoring="roc_auc")
    print(f"CV AUC: {cv_scores.mean():.4f} ± {cv_scores.std():.4f}")
    
    # --- 5. PERSIST ---
    joblib.dump(model, "fraud_model_v1.pkl")
    
    

#### Key concepts to explain out loud

**Pipeline** — Chains preprocessing and model so the same transforms apply to train and test. This prevents data leakage (a common interview trap question).

**ColumnTransformer** — Applies different transforms to numeric vs categorical columns in one step. Interviewers like to hear you mention this because it's more production-realistic than separate manual transforms.

**stratify=y in train_test_split** — Preserves class ratio in both splits. Critical for fraud detection (imbalanced datasets). Without it, your test set might have zero fraud cases.

**ROC-AUC vs Accuracy** — For fraud detection, accuracy is misleading. A model that predicts "no fraud" for every claim gets 99% accuracy but catches nothing. AUC measures ranking ability (how well the model separates fraud vs legitimate) regardless of threshold.

**Cross-validation** — k-fold CV gives a more reliable estimate of generalisation than a single train/test split. Always quote mean ± std dev. Large std dev signals unstable model.

**GradientBoosting hyperparameters** :

  * `n_estimators=300` — number of trees; more = lower bias, higher compute
  * `max_depth=4` — tree depth; lower = more regularisation
  * `learning_rate=0.05` — shrinks each tree's contribution; lower rate + more trees = better generalisation (but slower)



* * *

### 2B. TensorFlow / Keras End-to-End

Tabular binary classification in the insurance/fraud context.
    
    
    import tensorflow as tf
    from tensorflow import keras
    from sklearn.preprocessing import StandardScaler
    from sklearn.model_selection import train_test_split
    import numpy as np
    
    # Assume X_train, X_test, y_train, y_test are already split and numpy arrays
    
    scaler = StandardScaler()
    X_train_s = scaler.fit_transform(X_train)
    X_test_s  = scaler.transform(X_test)   # fit only on train — no leakage
    
    # --- MODEL ARCHITECTURE ---
    model = keras.Sequential([
        keras.layers.Input(shape=(X_train_s.shape[1],)),
        keras.layers.Dense(128, activation="relu"),
        keras.layers.BatchNormalization(),
        keras.layers.Dropout(0.3),
        keras.layers.Dense(64, activation="relu"),
        keras.layers.Dropout(0.2),
        keras.layers.Dense(1, activation="sigmoid")   # binary output: fraud probability
    ])
    
    model.compile(
        optimizer=keras.optimizers.Adam(learning_rate=1e-3),
        loss="binary_crossentropy",
        metrics=["AUC", "accuracy"]
    )
    
    model.summary()
    
    # --- CALLBACKS ---
    early_stop = keras.callbacks.EarlyStopping(
        monitor="val_auc",
        patience=10,
        restore_best_weights=True   # restores the best epoch's weights automatically
    )
    lr_reduce = keras.callbacks.ReduceLROnPlateau(
        monitor="val_loss",
        factor=0.5,
        patience=5,
        min_lr=1e-6
    )
    
    # --- TRAINING ---
    history = model.fit(
        X_train_s, y_train,
        validation_split=0.2,
        epochs=100,
        batch_size=256,
        callbacks=[early_stop, lr_reduce],
        class_weight={0: 1, 1: 10}   # penalise missing fraud (minority class) 10x more
    )
    
    # --- EVALUATE ---
    loss, auc, acc = model.evaluate(X_test_s, y_test, verbose=0)
    print(f"Test AUC: {auc:.4f} | Accuracy: {acc:.4f}")
    
    # Get probabilities
    y_prob = model.predict(X_test_s).flatten()
    
    # --- SAVE ---
    model.save("fraud_nn.keras")
    
    # Load back
    loaded_model = keras.models.load_model("fraud_nn.keras")
    
    

#### TensorFlow concepts to explain

**BatchNormalization** — Normalises activations across each mini-batch. Speeds up training, reduces sensitivity to weight initialisation, acts as mild regulariser.

**Dropout(0.3)** — Randomly zeros 30% of neurons during training. Forces network to learn redundant representations. Switched OFF automatically during inference.

**class_weight** — Tells Keras to weight the loss from fraud cases 10x higher. Equivalent to oversampling the minority class but computationally cheaper.

**EarlyStopping + restore_best_weights=True** — Stops training when validation AUC stops improving for `patience` epochs, then restores the best checkpoint. Prevents overfitting without manual epoch selection.

**ReduceLROnPlateau** — Halves the learning rate when loss plateaus. Allows fine-grained convergence later in training.

**activation="sigmoid" for binary output** — Outputs a value between 0 and 1. Use 0.5 as default threshold, or tune threshold using precision-recall curve for fraud use cases.

**binary_crossentropy loss** — The standard loss for binary classification. Penalises confident wrong predictions heavily.

**Why not softmax for binary?** — Sigmoid is equivalent to softmax with 2 classes but uses one output neuron instead of two. Both work; sigmoid is simpler.

* * *

### 2C. PyTorch End-to-End

PyTorch requires you to write the training loop manually — more control, more code. Interviewers often ask about the difference from Keras.
    
    
    import torch
    import torch.nn as nn
    from torch.utils.data import DataLoader, TensorDataset
    from sklearn.metrics import roc_auc_score
    import numpy as np
    
    # --- PREPARE TENSORS ---
    X_t = torch.tensor(X_train_s, dtype=torch.float32)
    y_t = torch.tensor(y_train.values, dtype=torch.float32).unsqueeze(1)  # shape: [N, 1]
    
    X_val_t = torch.tensor(X_test_s, dtype=torch.float32)
    y_val_t  = torch.tensor(y_test.values, dtype=torch.float32).unsqueeze(1)
    
    dataset = TensorDataset(X_t, y_t)
    loader  = DataLoader(dataset, batch_size=256, shuffle=True)
    
    # --- MODEL DEFINITION ---
    class FraudNet(nn.Module):
        def __init__(self, n_features):
            super().__init__()
            self.net = nn.Sequential(
                nn.Linear(n_features, 128),
                nn.ReLU(),
                nn.BatchNorm1d(128),
                nn.Dropout(0.3),
                nn.Linear(128, 64),
                nn.ReLU(),
                nn.Dropout(0.2),
                nn.Linear(64, 1),
                nn.Sigmoid()
            )
    
        def forward(self, x):
            return self.net(x)
    
    device = "cuda" if torch.cuda.is_available() else "cpu"
    model  = FraudNet(X_train_s.shape[1]).to(device)
    print(model)
    
    # --- TRAINING LOOP ---
    criterion = nn.BCELoss()
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer, patience=5, factor=0.5)
    
    best_auc = 0
    for epoch in range(100):
        # TRAINING PHASE
        model.train()   # activates Dropout and BatchNorm training behaviour
        epoch_loss = 0
        for xb, yb in loader:
            xb, yb = xb.to(device), yb.to(device)
            optimizer.zero_grad()       # 1. clear gradients from previous step
            preds = model(xb)           # 2. forward pass
            loss  = criterion(preds, yb) # 3. compute loss
            loss.backward()             # 4. backpropagate (compute gradients)
            optimizer.step()            # 5. update weights
            epoch_loss += loss.item()
    
        # VALIDATION PHASE
        model.eval()    # deactivates Dropout, uses running stats for BatchNorm
        with torch.no_grad():
            val_preds = model(X_val_t.to(device)).cpu().numpy().flatten()
        val_auc = roc_auc_score(y_test, val_preds)
        scheduler.step(1 - val_auc)    # reduce LR if AUC stops improving
    
        if val_auc > best_auc:
            best_auc = val_auc
            torch.save(model.state_dict(), "fraud_net_best.pt")
    
        if epoch % 10 == 0:
            print(f"Epoch {epoch:3d} | Loss: {epoch_loss/len(loader):.4f} | Val AUC: {val_auc:.4f}")
    
    print(f"Best Val AUC: {best_auc:.4f}")
    
    # --- LOAD BEST MODEL ---
    model.load_state_dict(torch.load("fraud_net_best.pt"))
    model.eval()
    
    

#### PyTorch vs Keras mental model (common interview comparison)

Concept| Keras| PyTorch  
---|---|---  
Training loop| Hidden (model.fit)| Explicit — you write it  
Gradient clearing| Automatic| optimizer.zero_grad() required  
Backprop| Automatic| loss.backward() explicit  
Weight update| Automatic| optimizer.step() explicit  
Train/eval modes| Automatic| model.train() / model.eval() required  
Flexibility| Lower| Higher (research preferred)  
Debugging| Harder| Easier (standard Python debugger works)  
  
**Why optimizer.zero_grad() matters** — PyTorch accumulates gradients by default. If you don't zero them each batch, gradients from previous batches add up, corrupting your updates.

**model.train() vs model.eval()** — Dropout and BatchNorm behave differently during training vs inference. Always set the correct mode. This is a common interview trap.

**torch.no_grad()** — Disables gradient tracking during validation. Saves memory and speeds up inference. Always use for eval loops.

**state_dict vs full model save**

  * `torch.save(model.state_dict(), path)` — saves only weights (preferred, portable)
  * `torch.save(model, path)` — saves entire model (fragile, path-dependent)



* * *

### 2D. Azure ML Deployment Pattern

Relevant to your CV and EY's Microsoft partnership.
    
    
    from azure.ai.ml import MLClient
    from azure.identity import DefaultAzureCredential
    from azure.ai.ml.entities import (
        Model, ManagedOnlineEndpoint, ManagedOnlineDeployment, Environment
    )
    import requests, json
    
    # --- 1. AUTHENTICATE ---
    ml_client = MLClient(
        DefaultAzureCredential(),
        subscription_id="<your-subscription-id>",
        resource_group_name="<your-rg>",
        workspace_name="<your-workspace>"
    )
    
    # --- 2. REGISTER MODEL ---
    model = ml_client.models.create_or_update(
        Model(
            path="fraud_model_v1.pkl",
            name="fraud-detector",
            version="1",
            description="GBM fraud detection model trained on AXA claims data"
        )
    )
    print(f"Registered: {model.name} v{model.version}")
    
    # --- 3. CREATE ENDPOINT ---
    endpoint = ManagedOnlineEndpoint(
        name="fraud-scoring-endpoint",
        description="Real-time fraud scoring",
        auth_mode="key"
    )
    ml_client.online_endpoints.begin_create_or_update(endpoint).result()
    
    # --- 4. DEPLOY ---
    deployment = ManagedOnlineDeployment(
        name="blue",
        endpoint_name="fraud-scoring-endpoint",
        model=model,
        instance_type="Standard_DS3_v2",
        instance_count=1,
        environment=Environment(
            image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04",
            conda_file="conda.yml"
        )
    )
    ml_client.online_deployments.begin_create_or_update(deployment).result()
    
    # Set 100% traffic to blue deployment
    endpoint.traffic = {"blue": 100}
    ml_client.online_endpoints.begin_create_or_update(endpoint).result()
    
    # --- 5. SCORE ---
    keys = ml_client.online_endpoints.get_keys("fraud-scoring-endpoint")
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {keys.primary_key}"
    }
    payload = {
        "input_data": {
            "columns": ["claim_amount", "days_to_report", "policy_type"],
            "data": [[5000, 3, "comprehensive"], [150, 45, "third_party"]]
        }
    }
    response = requests.post(endpoint.scoring_uri, json=payload, headers=headers)
    print(response.json())
    
    

#### MLOps lifecycle — what to know conceptually
    
    
    Code → Experiment → Model Registry → Deployment → Monitoring → Retrain
    
    Experiment tracking : Log params, metrics, artifacts per run (MLflow or Azure ML)
    Model registry      : Version control for trained models — promotion gates
                          (dev → staging → prod) with approval workflows
    Deployment          : Blue/green deployments — route traffic gradually
                          to new model version, rollback if metrics degrade
    Monitoring          : Watch input feature distributions (PSI), prediction
                          score drift, downstream business metrics
    Retrain trigger     : PSI > 0.2, or model AUC drops below threshold on
                          labelled production data
    
    

* * *

### 2E. Model Monitoring & Drift Detection
    
    
    import numpy as np
    import pandas as pd
    
    # --- POPULATION STABILITY INDEX (PSI) ---
    # Detects feature or score distribution drift between train and production
    # PSI < 0.10 → stable
    # PSI 0.10–0.20 → minor shift, monitor
    # PSI > 0.20 → significant shift, consider retraining
    
    def compute_psi(expected: np.ndarray, actual: np.ndarray, buckets: int = 10) -> float:
        """
        expected : distribution at training time (reference)
        actual   : current production distribution
        """
        breakpoints = np.percentile(expected, np.linspace(0, 100, buckets + 1))
        breakpoints[0]  = -np.inf
        breakpoints[-1] =  np.inf
    
        def bucket_pct(arr):
            counts, _ = np.histogram(arr, bins=breakpoints)
            pct = counts / len(arr)
            return np.where(pct == 0, 1e-8, pct)  # avoid log(0)
    
        e = bucket_pct(expected)
        a = bucket_pct(actual)
        return float(np.sum((a - e) * np.log(a / e)))
    
    # Usage
    train_scores = model.predict_proba(X_train)[:, 1]
    prod_scores  = model.predict_proba(X_production)[:, 1]
    psi = compute_psi(train_scores, prod_scores)
    print(f"Score PSI: {psi:.4f}")
    if psi > 0.2:
        print("WARNING: Significant score drift — evaluate retraining")
    
    # --- FEATURE DRIFT (per-feature PSI) ---
    drift_report = {}
    for col in num_features:
        drift_report[col] = compute_psi(
            X_train[col].values,
            X_production[col].values
        )
    
    drift_df = pd.DataFrame.from_dict(drift_report, orient="index", columns=["PSI"])
    drift_df["status"] = drift_df["PSI"].apply(
        lambda x: "stable" if x < 0.1 else ("monitor" if x < 0.2 else "DRIFT")
    )
    print(drift_df.sort_values("PSI", ascending=False))
    
    

#### What to monitor in production — summary

Signal| Method| Threshold  
---|---|---  
Score distribution| PSI on model output probabilities| > 0.2 = retrain  
Feature distributions| PSI per input feature| > 0.2 = investigate  
Null rates| % missing values per feature| > 2x baseline = data quality alert  
Model performance| AUC/precision on labelled data (with delay)| Drop > 5% = retrain  
Business metrics| Fraud catch rate, false positive rate| Agreed SLAs  
  
* * *

## 3\. STAR Stories

**Interview tip:** Before each story say "I'll give you a concrete example from my work at [company]." Keep Situation + Task under 60 seconds. Spend most time on Action (your thinking process) and Result (the numbers). EY interviewers will probe with "what was your specific personal contribution?" — be precise.

* * *

### Story 1 — AXA Fraud Detection Platform

**Competency addressed:** ML end-to-end management | Fraud domain expertise | Regulatory compliance

* * *

**SITUATION:** AXA's fraud detection ML pipeline was fed by a monolithic overnight batch process with 24-hour data latency. Claims flagged at midnight weren't actionable until the following day, meaning fraud patterns could propagate undetected for 36–48 hours. There was also no systematic data quality framework — the ML models were consuming unvalidated inputs, degrading the reliability of loss reserving outputs.

**TASK:** I was tasked with redesigning the data infrastructure layer feeding the fraud ML pipeline: reduce latency, introduce systematic data quality gates, and build a feature store that downstream models could consume reliably. I also had to ensure the solution met FCA and CBI audit and model explainability requirements.

**ACTION:** I designed and deployed CDC (Change Data Capture) streams in Microsoft Fabric to replace the overnight batch, cutting ingestion lag to under 15 minutes. I built a data quality scoring framework with 40+ automated validation checks covering null rates, statistical range violations, and referential integrity across claims, policy, and customer tables. I engineered a Delta Lake feature store consolidating all model-ready attributes into a single, versioned source of truth consumed by downstream ML models. I also built anomaly detection models in PySpark using statistical thresholds (z-scores and IQR methods) to flag upstream data irregularities before they reached the ML layer. Finally, I established end-to-end data lineage using Microsoft Purview, mapping every feature from source to model input, to satisfy FCA/CBI audit requirements on model explainability.

**RESULT:** Data latency dropped from 24 hours to under 15 minutes — a 96% reduction — directly accelerating ML model refresh cycles for fraud detection. Manual data review effort fell by 40%. Actuarial loss reserving models received measurably higher-quality inputs. The lineage framework passed FCA/CBI audit without remediation. I communicated model outputs — fraud risk scores, anomaly counts, reserving projections — to C-suite via Power BI Direct Lake dashboards, translating statistical findings into business language for non-technical stakeholders.

* * *

### Story 2 — J&J Cost Savings Analysis

**Competency addressed:** Statistical modelling | Translating data insights to business value | Communication

* * *

**SITUATION:** Johnson & Johnson's Global Services function operated across 4 disparate legacy data sources with no single source of truth. Finance and procurement leadership were making resourcing and outsourcing decisions based on inconsistent, manually reconciled data. There was no statistical model to identify inefficiency or cost optimisation opportunities — decisions were made on intuition rather than data.

**TASK:** My task was to consolidate these data sources into a unified analytical platform, apply statistical modelling to identify cost drivers, and present findings to senior leadership in a way that drove executive decisions — not just produced reports.

**ACTION:** I architected a C# service layer to automate Salesforce-to-data-warehouse integration, eliminating manual exports. I then consolidated all 4 legacy sources into a single BigQuery model — a canonical source of truth serving 150+ analysts. I applied regression and cost attribution modelling to decompose spend drivers across the function, ran A/B scenario analyses modelling efficiency under different process configurations, and documented 200+ business attributes with feature transformation rules in a structured STTM (source-to-target mapping). I translated the statistical outputs into board-ready Power BI visualisations, deliberately framing all findings in revenue impact and FTE efficiency terms rather than model coefficients.

**RESULT:** The analysis surfaced $2M+ in identifiable cost savings through process optimisation. The unified BigQuery model eliminated reconciliation overhead for 150+ analysts. Leadership adopted three of five modelled efficiency scenarios in the following quarter. This project sharpened my ability to translate complex statistical findings into business language — a skill I understand is central to EY's client-facing AI & Data practice.

* * *

### Story 3 — Google Street View Anomaly Detection

**Competency addressed:** ML-assisted anomaly detection | Automation | Cloud data pipelines

* * *

**SITUATION:** Google's Street View data pipeline was experiencing latency discrepancies in image ingestion that were difficult to diagnose manually. Research and compliance teams were spending 100+ hours per month manually inspecting pipeline outputs. There was no automated anomaly detection to identify which pipeline stages were causing delays, making root cause analysis slow and reactive.

**TASK:** I was asked to build ML-assisted anomaly detection for Street View image latency discrepancies and automate the data pipeline orchestration that compliance and research teams depended on.

**ACTION:** I built ML-assisted anomaly detection using statistical threshold models — z-score and seasonal decomposition — applied to image latency metrics at each pipeline stage in BigQuery. The model surfaced a specific bottleneck in the ingestion stage that had been invisible to manual review. In parallel, I automated feature extraction and pipeline orchestration using Python and Google Workflow, eliminating the manual inspection cycle. I engineered geospatial feature layers (Churn Buildings, Churn POIs, Churn Forest Scope) using DMS for transformation and downstream model consumption. Real-time Looker Studio dashboards gave senior leadership R&D performance metrics without any manual reporting cycle.

**RESULT:** The anomaly detection model surfaced the pipeline bottleneck that drove a 25% process optimisation in the subsequent quarter. Automation eliminated 100+ manual hours per month for research and compliance teams. Executive dashboards accelerated decision-making for global senior leadership and replaced a manual weekly reporting cycle entirely.

* * *

## 4\. EY AI & Data Practice Research

### EY Ireland — Practice Overview

**Practice name:** EY Consulting — AI & Data (part of Technology Consulting in Ireland)

**Ireland offices:** Two Haddington Buildings, Dublin 2. Also Belfast, Cork, Limerick, Galway.

**Scale:** EY globally has ~4,000 AI specialists. The Ireland practice serves both domestic Irish clients and EMEA engagements, positioning Dublin as a regional delivery hub.

**Key sectors in Ireland relevant to your background:**

Sector| Your Proof Point  
---|---  
Financial Services (P&C insurance, bancassurance)| AXA fraud detection platform  
Life Sciences & Pharma (ICON, Pfizer, BioMarin)| 11 years cross-industry delivery  
Technology (LinkedIn, Meta, Google EMEA centres)| Google Street View project  
Retail & Consumer| Listed in your CV  
Public Sector| Listed in your CV  
  
* * *

### EY's AI Positioning & Current Priorities

**EY.ai platform** — EY's unified AI platform, announced in 2023 with a $1.4B investment. It integrates AI across all service lines — audit, tax, consulting, strategy. EY positions it as the backbone of AI-enabled client services. In conversations, demonstrate awareness of this investment.

**Trusted AI / Responsible AI** — EY emphasises AI governance, explainability, and regulatory compliance as core differentiators. Their AI assurance and TPRM (Third Party Risk Management) services are growing fast as enterprise AI adoption increases. Your Purview/FCA/CBI lineage work at AXA is a perfect proof point here.

**GenAI focus areas** — EY is investing heavily in GenAI for client delivery: RAG-based enterprise search, LLM-powered document extraction for audit and contract review, AI agents for process automation. Your listed LLM/RAG/Agent experience is directly relevant. Prepare a concrete example.

**Cloud partnerships** — EY is a top-tier partner of Microsoft (Azure), AWS, and Google Cloud. The Ireland practice delivers on all three. Your Azure Fabric + GCP BigQuery breadth across two major cloud platforms is a genuine differentiator.

**EU AI Act** — In force from 2024/2025, this is creating demand for AI governance and compliance services across all EY sectors. FCA/CBI model explainability and lineage work (which you delivered at AXA) maps directly to EU AI Act high-risk AI obligations.

* * *

### EY Ireland — Recent Context to Reference

**AI Ireland** — EY is a founding partner of AI Ireland, the national industry body for AI in Ireland. Shows genuine investment in the local ecosystem. Referencing this signals you've done research.

**Insurance AI** — EY has published extensively on AI in P&C insurance claims automation, fraud detection, and reserving. Search "EY insurance claims AI" for specific reports to reference by name in the interview.

**EY NextWave** — EY's global data and AI transformation programme, pitching end-to-end data modernisation including lakehouse architecture (which you delivered at Travelers) and MLOps maturity frameworks.

**EY wavespace** — EY's innovation centres. Dublin has a wavespace location used for client workshops and AI prototyping engagements. Showing awareness of how EY structures innovation delivery is a positive signal.

* * *

### Questions to Ask Your Interviewer

These signal genuine research, not generic curiosity:

  1. "How is EY Ireland's AI & Data practice positioned around the EU AI Act — are you seeing demand for model governance and explainability services from financial services clients specifically?"
  2. "With EY.ai as a platform investment, how does the Ireland team contribute to or consume that capability in client engagements?"
  3. "What does the typical engagement look like for a Senior Consultant in AI & Data — is it more embedded long-term in client delivery or structured as shorter advisory sprints?"
  4. "How does EY Ireland's practice interact with the global AI Centre of Excellence, especially for Generative AI use cases?"
  5. "Given the growth in EU AI Act compliance work, is model governance and explainability a significant part of current client engagements?"



* * *

## 5\. Key Concepts Cheat Sheet

### Statistics & ML fundamentals (likely interview topics)

**Bias-variance tradeoff**

  * High bias = underfitting (model too simple, ignores signal)
  * High variance = overfitting (model too complex, memorises noise)
  * GBM with small depth = high bias; GBM with large depth = high variance
  * Regularisation (Dropout, L2, early stopping) reduces variance



**Class imbalance — techniques**

  * class_weight (Keras, Scikit-Learn) — penalise minority class in loss
  * SMOTE — synthetic oversampling of minority class
  * Threshold tuning — lower decision threshold to catch more fraud (increases recall, reduces precision)
  * Evaluation: use AUC-ROC or F1/precision-recall curve, not accuracy



**Feature importance — methods**

  * Tree-based: built-in `feature_importances_` (mean decrease in impurity)
  * SHAP values: game-theoretic, model-agnostic, shows direction of effect
  * Permutation importance: shuffle each feature, measure AUC drop



**Regularisation**

  * L1 (Lasso): drives some weights to exactly zero — feature selection
  * L2 (Ridge): shrinks weights towards zero but not exactly — reduces overfitting
  * Dropout: neural network equivalent — randomly zeros neurons during training
  * Early stopping: stop training when validation loss starts increasing



**Gradient descent variants**

  * Batch GD: all data each update — stable but slow
  * Stochastic GD: one sample per update — noisy, fast
  * Mini-batch GD: subset per update — balance of both (standard practice)
  * Adam optimiser: adaptive per-parameter learning rates — default for neural networks



**Evaluation metrics**

  * AUC-ROC: ranking metric, threshold-independent, 0.5 = random, 1.0 = perfect
  * Precision: of all predicted fraud, how many were real fraud
  * Recall (sensitivity): of all real fraud, how many did we catch
  * F1: harmonic mean of precision and recall
  * For fraud: recall is usually more important (missing fraud is costly)



**Cross-validation**

  * k-fold: split data into k folds, train on k-1, test on 1, rotate
  * Stratified k-fold: preserves class ratio in each fold
  * Time-series: always split chronologically — no future data in training



### Data engineering concepts

**CDC (Change Data Capture)** — Captures only the rows that changed since the last load. Much more efficient than full table reloads. Implemented via database transaction logs. Used to achieve the 24hr → 15min latency improvement at AXA.

**Bronze-Silver-Gold lakehouse**

  * Bronze: raw ingested data, no transforms, immutable
  * Silver: cleaned, validated, joined — business-usable
  * Gold: aggregated, feature-engineered — ML and BI ready



**Feature store** — Centralised repository of pre-computed, versioned features. Ensures training and serving use identical feature definitions (prevents train/serve skew).

**Data lineage** — Tracks every feature from source system to model input. Required for model explainability under FCA/CBI and EU AI Act.

**PSI (Population Stability Index)** — Measures distribution shift between two datasets (typically train vs production). PSI < 0.1 = stable, > 0.2 = significant drift, retrain.

* * *

## 6\. One-Line Positioning Statement

Use this in "tell me about yourself" to open every interview:

* * *

> "I'm a data scientist and AI consultant with 11 years building the full stack — from data pipelines and feature engineering through to ML model deployment and executive communication. Most recently at Cognizant I delivered an AI-enabled fraud detection platform for AXA Ireland, cutting data latency by 96% and meeting FCA model explainability requirements. I'm now looking to bring that end-to-end delivery experience into EY's AI & Data practice, where the combination of technical depth and client communication is exactly the profile the role demands."

* * *

**Core positioning message:** You are "the person who closes the loop between data engineering and ML delivery." You build the pipelines, build the models, deploy them, and can explain them to a CFO. That full-stack profile is what a consulting firm monetises — it's not just a data scientist, it's a billable end-to-end practitioner.

* * *

_End of wiki. Last updated for EY Senior Consultant AI & Data interview preparation._

> Claide [Link](<https://claude.ai/share/c5adf679-838b-46d2-b4c8-0c53cdebd99d>) for Core Technical Scenarios

I'll build you five visual diagrams covering the core technical scenarios from the job spec — each one showing how the technologies you know fit in. Let me walk through them one by one.

**Diagram 1 — The ML model lifecycle** (the core competency EY is testing for)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-30.png?w=1024)

This is the lifecycle EY is testing you on — the "end-to-end management" requirement. Click any box to go deeper. Notice the retrain loop at the bottom — that's what separates a production ML system from a notebook.

**Diagram 2 — Your tech stack mapped to the job spec requirements**

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-31.png?w=1024)

Green = solid, amber = needs active prep before the interview. TF/PyTorch and GenAI are your two amber items.

**Diagram 3 — The fraud detection data architecture** (your AXA STAR story, visualised)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-32.png?w=1024)

This is your STAR Story 1 made visual. If an EY interviewer asks "walk me through a data architecture you've built," you can describe exactly this diagram from memory — Bronze/Silver/Gold layers, CDC streams, feature store, Purview governance, and the Power BI output.

**Diagram 4 — The bias-variance tradeoff** (core statistics concept in the spec)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-33.png?w=1024)

Drag the slider left to right — you can see bias drop as variance rises. The sweet spot is the minimum of total error. This is a foundational statistics concept the spec calls out, and being able to explain it fluently (not just name it) impresses interviewers.

**Diagram 5 — GenAI / RAG architecture** (the "preferred" requirement you need to prepare)

![](https://practiceiq.wordpress.com/wp-content/uploads/2026/04/image-34.png?w=1024)

This is the GenAI architecture EY is actively building for clients — RAG over enterprise documents. The two paths are: an offline ingestion path (documents → chunks → embeddings → vector store) and an online query path (user query → embed → similarity search → LLM with context → grounded answer). Click any node for a deeper explanation.

Here's a quick summary of what each diagram teaches you for the interview:

The lifecycle diagram gives you the framework to answer "describe how you manage ML models end to end." The tech map shows you exactly where your two gaps are. The AXA architecture is your STAR Story 1 made visual — describe it left to right. The bias-variance widget lets you internalise the tradeoff interactively rather than just memorise the definition. And the RAG diagram covers the GenAI requirement — study the two paths (ingestion vs query) so you can draw it on a whiteboard if asked.
