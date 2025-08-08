# 🧑‍💻 Insurance Claims Processing Pipeline - Hands-On Lab Guide

## Preapred by Nived Varma / Microsoft Certified Trainer

## 📋 Project Overview

**Challenge**: Build an end-to-end insurance claims processing pipeline where YOU write the code! Use hints and partial implementations to create a production-ready ML system.

**Your Mission**: Transform raw claims data into actionable fraud predictions and processing time estimates using Azure Databricks + MLflow.

---

## 🎯 What You'll Build

- ✅ **Data Pipeline**: Ingest → Clean → Transform → Feature Engineering
- ✅ **ML Models**: Fraud Detection (Classification) + Processing Time (Regression)
- ✅ **MLflow Integration**: Experiment tracking and model registration
- ✅ **Model Serving**: Deploy via Databricks GUI for real-time inference
- ✅ **API Testing**: Query your deployed models with JSON payloads

---

## 📊 Dataset Setup

**File**: Upload `insurance_claims_data.csv` to `/FileStore/claim_data/`
**Reference**: Use the provided data dictionary for field definitions

---

## 🏗️ Lab Exercises

### **Exercise 1: Data Ingestion & Bronze Layer** ⏱️ _20 minutes_

```python
# Import necessary libraries
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *

# Initialize Spark session
spark = SparkSession.builder.appName("_____").getOrCreate()

# TODO: Read the CSV file from /FileStore/claim_data/insurance_claims_data.csv
# HINT: Use spark.read with proper options for header and schema inference
bronze_df = spark.read._____(_____)._____("_____")

# TODO: Explore the data structure
print(f"Total records: {_____}")
print("Schema:")
_____
print("Sample data:")
_____

# TODO: Save as Delta Bronze table
bronze_df.write.format("_____").mode("_____").saveAsTable("_____")
```

**🎯 Your Tasks:**

- [ ] Complete the CSV reading code
- [ ] Add data exploration commands
- [ ] Save as Delta table named `bronze_claims`

---

### **Exercise 2: Data Cleaning & Silver Layer** ⏱️ _30 minutes_

```python
# TODO: Implement data cleaning pipeline
silver_df = bronze_df \
    .dropna(subset=['_____', '_____'])  # Drop rows with missing critical fields \
    .withColumn('claim_date', to_date('_____')) \
    .withColumn('reported_date', to_date('_____')) \
    .fillna({'customer_segment': '_____', 'witnesses': _____}) \
    .filter(_____)  # Add email validation filter

# TODO: Add data quality checks
print("Data Quality Report:")
print("="*50)

# Count total records
total_records = _____
print(f"Total records after cleaning: {total_records}")

# TODO: Calculate and display null counts for each column
null_counts = silver_df.select([_____ for c in silver_df.columns])
_____

# TODO: Check for duplicate claim_ids
duplicate_claims = _____
print(f"Duplicate claims: {duplicate_claims}")

# TODO: Save partitioned Silver table
# HINT: Partition by location_state and claim_type for better query performance
silver_df.write.format("_____") \
    .partitionBy("_____", "_____") \
    .mode("_____") \
    .saveAsTable("_____")
```

**🎯 Your Tasks:**

- [ ] Complete data cleaning logic
- [ ] Add email validation regex
- [ ] Implement data quality checks
- [ ] Save partitioned Silver table

---

### **Exercise 3: Feature Engineering & Gold Layer** ⏱️ _25 minutes_

```python
# TODO: Create advanced features for ML models
features_df = silver_df \
    .withColumn('days_to_report', _____) \  # Calculate days between incident and report
    .withColumn('claim_severity',
                when(col('claim_amount') > _____, 'High')
                .when(col('claim_amount') > _____, 'Medium')
                .otherwise('_____')) \
    .withColumn('is_weekend', _____) \  # Check if claim_date is weekend
    .withColumn('claim_to_coverage_ratio', _____) \  # claim_amount / coverage_amount
    .withColumn('high_value_claim', (col('claim_amount') > _____).cast('int'))

# TODO: Add seasonal features
features_df = features_df \
    .withColumn('claim_month', _____) \
    .withColumn('claim_quarter', _____) \
    .withColumn('is_storm_season', _____)  # June-November

# TODO: Create customer risk profile
# HINT: Count previous claims per customer (use window functions)
from pyspark.sql.window import Window

window_spec = Window.partitionBy("_____").orderBy("_____")
features_df = features_df \
    .withColumn('customer_claim_history', _____)

# TODO: Display feature summary
print("Feature Engineering Summary:")
print("="*50)
features_df.select("_____", "_____", "_____").show(5)

# TODO: Save Gold layer
features_df.write.format("_____").mode("_____").saveAsTable("_____")
```

**🎯 Your Tasks:**

- [ ] Complete all feature engineering logic
- [ ] Add seasonal and weekend features
- [ ] Implement customer history features
- [ ] Save as `gold_claims_features` table

---

### **Exercise 4: ML Model Development** ⏱️ _45 minutes_

#### **4a. Setup MLflow**

```python
import mlflow
import mlflow.spark
from pyspark.ml import Pipeline
from pyspark.ml.feature import StringIndexer, VectorAssembler, StandardScaler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.regression import RandomForestRegressor
from pyspark.ml.evaluation import BinaryClassificationEvaluator, RegressionEvaluator

# TODO: Set MLflow experiment name
mlflow.set_experiment("_____")
```

#### **4b. Fraud Detection Model**

```python
with mlflow.start_run(run_name="fraud_detection_v1"):

    # TODO: Define feature columns
    categorical_cols = ['_____', '_____', '_____', '_____']
    numeric_cols = ['_____', '_____', '_____', '_____', '_____']

    # TODO: Create preprocessing pipeline
    # HINT: Use StringIndexer for categorical variables
    indexers = [StringIndexer(inputCol=_____, outputCol=_____)
                for col in _____]

    # TODO: Create feature vector
    assembler = VectorAssembler(
        inputCols=_____ + _____,
        outputCol="_____"
    )

    # TODO: Add feature scaling
    scaler = StandardScaler(inputCol="_____", outputCol="_____")

    # TODO: Initialize classifier
    rf_fraud = RandomForestClassifier(
        featuresCol="_____",
        labelCol="_____",
        numTrees=_____,
        maxDepth=_____
    )

    # TODO: Create complete pipeline
    pipeline = Pipeline(stages=_____ + [_____] + [_____] + [_____])

    # TODO: Split data for training
    train_df, test_df = features_df.randomSplit([_____, _____], seed=42)

    # TODO: Fit the model
    model = _____

    # TODO: Make predictions
    predictions = _____

    # TODO: Evaluate model performance
    evaluator = BinaryClassificationEvaluator(labelCol="_____", rawPredictionCol="_____")
    auc = _____

    # TODO: Log metrics to MLflow
    mlflow.log_metric("_____", _____)
    mlflow.log_param("num_trees", _____)
    mlflow.log_param("max_depth", _____)

    # TODO: Log the model
    mlflow.spark.log_model(_____, "_____")

    print(f"✅ Fraud Detection Model AUC: {auc:.3f}")
```

#### **4c. Processing Time Prediction Model**

```python
with mlflow.start_run(run_name="processing_time_v1"):

    # TODO: Filter data with non-null processing days
    processing_df = features_df.filter(_____)

    # TODO: Use same preprocessing pipeline but different target
    rf_time = RandomForestRegressor(
        featuresCol="_____",
        labelCol="_____",
        numTrees=_____
    )

    # TODO: Create pipeline for regression
    pipeline_time = Pipeline(stages=_____)

    # TODO: Train and evaluate
    train_time, test_time = processing_df.randomSplit([_____, _____], seed=42)
    model_time = _____
    predictions_time = _____

    # TODO: Calculate RMSE
    evaluator_time = RegressionEvaluator(
        labelCol="_____",
        predictionCol="_____",
        metricName="_____"
    )
    rmse = _____

    # TODO: Log metrics and model
    mlflow.log_metric("_____", _____)
    mlflow.spark.log_model(_____, "_____")

    print(f"✅ Processing Time Model RMSE: {rmse:.2f} days")
```

**🎯 Your Tasks:**

- [ ] Complete fraud detection model pipeline
- [ ] Implement processing time regression model
- [ ] Add proper MLflow logging
- [ ] Achieve AUC > 0.8 for fraud model

---

### **Exercise 5: Model Registration via GUI** ⏱️ _15 minutes_

**Manual Steps in Databricks UI:**

1. **Navigate to MLflow**:

   - Go to `Machine Learning` → `Experiments`
   - Find your experiment: `/Claims-Processing-Models`

2. **Register Fraud Model**:

   - Click on your best fraud detection run
   - Click `Register Model`
   - Model name: `fraud_detection_model`
   - Description: "Insurance claims fraud detection model"

3. **Register Processing Time Model**:

   - Click on your processing time run
   - Click `Register Model`
   - Model name: `processing_time_model`
   - Description: "Claims processing time prediction model"

4. **Transition to Production**:
   - Go to `Models` tab
   - For each model: `Stage` → `Transition to` → `Production`

**🎯 Your Tasks:**

- [ ] Register both models via Databricks GUI
- [ ] Transition models to Production stage
- [ ] Verify model artifacts are saved

---

### **Exercise 6: Model Serving Setup** ⏱️ _20 minutes_

**Manual Steps in Databricks UI:**

1. **Create Model Serving Endpoint**:

   - Go to `Machine Learning` → `Model Serving`
   - Click `Create serving endpoint`
   - Endpoint name: `claims-fraud-detector`
   - Select model: `fraud_detection_model`
   - Version: `Production`
   - Click `Create`

2. **Create Processing Time Endpoint**:

   - Create another endpoint: `claims-processing-timer`
   - Select model: `processing_time_model`
   - Version: `Production`

3. **Wait for Deployment** (5-10 minutes):
   - Monitor status until "Ready"

**🎯 Your Tasks:**

- [ ] Create serving endpoints for both models
- [ ] Verify endpoints are in "Ready" state
- [ ] Note endpoint URLs for testing

---

### **Exercise 7: API Testing with JSON** ⏱️ _25 minutes_

#### **Test Data Preparation**

```python
# TODO: Create test samples for API calls
test_claims = [
    {
        "claim_id": "CLM-2024-TEST001",
        "claim_amount": 85000,
        "claim_type": "Fire",
        "customer_segment": "Premium",
        "policy_type": "Home",
        "fraud_score": 0.75,
        "days_to_report": 5,
        "witnesses": 1,
        "claim_severity": "High",
        "is_weekend": 0,
        "claim_to_coverage_ratio": 0.17,
        "high_value_claim": 1
    },
    # TODO: Add 2 more test cases - one normal, one suspicious
    {
        "claim_id": "_____",
        "claim_amount": _____,
        # Complete this test case
    },
    {
        "claim_id": "_____",
        "claim_amount": _____,
        # Complete this test case
    }
]

# Display test data
import json
for i, claim in enumerate(test_claims):
    print(f"Test Case {i+1}:")
    print(json.dumps(claim, indent=2))
    print("-" * 40)
```

#### **JSON Payloads for Model Testing**

**Fraud Detection API Call:**

```json
{
  "instances": [
    {
      "claim_type": "Fire",
      "customer_segment": "Premium",
      "policy_type": "Home",
      "claim_severity": "High",
      "claim_amount": 85000,
      "fraud_score": 0.75,
      "days_to_report": 5,
      "witnesses": 1,
      "claim_to_coverage_ratio": 0.17,
      "high_value_claim": 1,
      "is_weekend": 0
    }
  ]
}
```

**Processing Time API Call:**

```json
{
  "instances": [
    {
      "claim_type": "Fire",
      "customer_segment": "Premium",
      "policy_type": "Home",
      "claim_severity": "High",
      "claim_amount": 85000,
      "days_to_report": 5,
      "witnesses": 1,
      "claim_to_coverage_ratio": 0.17,
      "high_value_claim": 1,
      "is_weekend": 0
    }
  ]
}
```

#### **Testing in Databricks UI**

**Steps to Test Models:**

1. **Navigate to Model Serving**:

   - Go to `Machine Learning` → `Model Serving`
   - Click on `claims-fraud-detector` endpoint

2. **Test Fraud Detection**:

   - Click `Query endpoint`
   - Paste fraud detection JSON payload
   - Click `Send Request`
   - Expected response: `{"predictions": [{"prediction": 1.0, "probability": [0.2, 0.8]}]}`

3. **Test Processing Time**:
   - Go to `claims-processing-timer` endpoint
   - Paste processing time JSON payload
   - Expected response: `{"predictions": [45.2]}`

#### **Programmatic Testing**

```python
# TODO: Test your deployed models programmatically
import requests
import json

# Get your endpoint URLs from Databricks UI
fraud_endpoint_url = "https://your-workspace.databricks.com/serving-endpoints/claims-fraud-detector/invocations"
time_endpoint_url = "https://your-workspace.databricks.com/serving-endpoints/claims-processing-timer/invocations"

# TODO: Create headers with authentication
headers = {
    "Authorization": f"Bearer {_____}",  # Your Databricks token
    "Content-Type": "application/json"
}

# TODO: Test fraud detection
fraud_payload = {
    "instances": [_____]  # Use your test data
}

response = requests.post(_____, headers=_____, json=_____)
print(f"Fraud Detection Response: {_____}")

# TODO: Test processing time prediction
time_payload = {
    "instances": [_____]
}

response_time = requests.post(_____, headers=_____, json=_____)
print(f"Processing Time Response: {_____}")
```

**🎯 Your Tasks:**

- [ ] Complete test data creation
- [ ] Test both models via Databricks GUI
- [ ] Implement programmatic API testing
- [ ] Verify model responses are reasonable

---

## 🎯 Success Checklist

### **Data Pipeline** ✅

- [ ] CSV data loaded into Bronze table
- [ ] Data cleaning implemented in Silver layer
- [ ] Feature engineering completed in Gold layer
- [ ] All Delta tables properly partitioned

### **ML Models** 🤖

- [ ] Fraud detection model trained with AUC > 0.8
- [ ] Processing time model trained with RMSE < 20
- [ ] MLflow experiments properly logged
- [ ] Models registered via GUI

### **Model Serving** 🚀

- [ ] Both models deployed to serving endpoints
- [ ] Endpoints show "Ready" status
- [ ] API calls return valid predictions
- [ ] JSON payloads properly formatted

### **Testing** 🧪

- [ ] Created realistic test cases
- [ ] Tested via Databricks GUI successfully
- [ ] Programmatic API calls working
- [ ] Model responses validated

---

## 🎁 Bonus Challenges

If you finish early, try these advanced features:

1. **Hyperparameter Tuning**:

   ```python
   # TODO: Implement grid search for optimal parameters
   from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
   ```

2. **Feature Importance Analysis**:

   ```python
   # TODO: Extract and visualize feature importance
   feature_importance = model.stages[-1].featureImportances
   ```

3. **Model Monitoring**:

   ```python
   # TODO: Create prediction drift monitoring
   def monitor_model_drift(new_predictions, baseline_predictions):
       # Implement drift detection logic
       pass
   ```

4. **Batch Inference Pipeline**:
   ```python
   # TODO: Create automated batch scoring job
   def batch_score_claims(input_table, output_table):
       # Load models and score in batch
       pass
   ```

---

## 💡 Hints & Tips

**Common Debugging Steps:**

- Use `.show()` frequently to inspect DataFrames
- Check schema with `.printSchema()`
- Count records at each pipeline stage
- Use `.explain()` for query optimization

**Performance Tips:**

- Cache DataFrames used multiple times
- Use broadcast joins for small lookup tables
- Optimize partition sizes (aim for 100MB-1GB per partition)

**MLflow Best Practices:**

- Use descriptive run names and tags
- Log both training and validation metrics
- Save feature importance and model artifacts
- Use model signatures for input validation

---

## 🎉 Completion

**Demo Requirements:**

- Show end-to-end pipeline execution
- Demonstrate live model predictions via API
- Explain your feature engineering choices
- Present model performance metrics

**What You've Built:**

- Production-ready data pipeline
- ML models with proper experiment tracking
- Deployed model serving endpoints
- API testing framework

---

_Ready to build your insurance claims processing system? Let's code! 🚀_
