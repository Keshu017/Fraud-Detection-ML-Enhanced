# 🚀 QUICKSTART GUIDE - Run Fraud Detection ML Project in 10 Minutes

## Prerequisites
- Python 3.9+
- pip or conda
- Git
- Optional: Docker (for containerized deployment)
- Optional: AWS account (for cloud deployment)

---

## Step 1: Clone & Install Dependencies (2 minutes)

```bash
# Clone repository
git clone https://github.com/Keshu017/Fraud-Detection-ML-Enhanced.git
cd Fraud-Detection-ML-Enhanced

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

✅ **Check Installation:**
```bash
python -c "import xgboost, lightgbm, fastapi, mlflow; print('All dependencies installed!')"
```

---

## Step 2: Download Dataset (3 minutes)

### Option A: Using Kaggle API (Recommended)
```bash
# Install kaggle CLI
pip install kaggle

# Download from Kaggle (requires account)
kaggle datasets download -d kartik2112/fraud-detection
unzip fraud-detection.zip -d data/
cd data && gunzip *.gz && cd ..
```

### Option B: Manual Download
1. Visit: https://www.kaggle.com/datasets/kartik2112/fraud-detection
2. Click "Download"
3. Extract to `data/` folder
4. Ensure you have `creditcard.csv` in `data/` directory

✅ **Verify Data:**
```bash
ls -la data/creditcard.csv  # Should show file size ~150MB
```

---

## Step 3: Run Training Pipeline (3 minutes)

### Full Training (All 6 Models)
```bash
python main.py --mode train --all-models
```

**Output:**
- `models/xgboost_model.joblib` - XGBoost trained
- `models/lightgbm_model.joblib` - LightGBM trained
- `models/catboost_model.joblib` - CatBoost trained
- `models/ensemble_v1.joblib` - Final ensemble model
- MLflow tracking URI: http://localhost:5000

### Quick Training (Single Model - 30 seconds)
```bash
python main.py --mode train --model xgboost --quick
```

✅ **Verify Models:**
```bash
ls -lh models/
```

---

## Step 4: View Experiment Results in MLflow (Optional)

```bash
# Start MLflow UI
mlflow ui --port 5000
```

**Then visit:** http://localhost:5000

**You'll see:**
- Experiment runs for each model
- Metrics: Accuracy, Precision, Recall, F1-Score, AUC-ROC
- Parameters: learning_rate, max_depth, n_estimators, etc.
- Model artifacts and versions

---

## Step 5: Run Tests (1 minute)

```bash
# Install test dependencies
pip install pytest pytest-cov

# Run all tests
pytest tests/ -v --cov=src

# Run specific test suite
pytest tests/unit/ -v  # Unit tests only
pytest tests/integration/ -v  # Integration tests
```

✅ **Expected Output:**
```
tests/unit/test_data_validation.py::test_load_data PASSED
tests/unit/test_feature_engineering.py::test_scaling PASSED
tests/unit/test_models.py::test_xgboost_training PASSED
========== 12 passed in 2.34s ==========
```

---

## Step 6: Start FastAPI Server (< 30 seconds)

```bash
# Start the API
uvicorn src.api.main:app --reload --port 8000
```

**Output:**
```
INFO:     Uvicorn running on http://127.0.0.1:8000
INFO:     Application startup complete
```

✅ **Test API Health:**
```bash
curl http://localhost:8000/health
# Response: {"status": "healthy", "version": "1.0"}
```

---

## Step 7: Make Predictions via API

### Single Prediction
```bash
curl -X POST "http://localhost:8000/api/v1/predict" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 100.50,
    "time": 10800,
    "merchant_id": "M001",
    "distance": 50.5
  }'
```

**Response:**
```json
{
  "fraud_probability": 0.0234,
  "prediction": "LEGITIMATE",
  "confidence": 0.9766,
  "processing_time_ms": 45
}
```

### Batch Predictions (CSV Upload)
```bash
# Create sample test file
cat > test_transactions.csv << EOF
amount,time,merchant_id,distance
100.50,10800,M001,50.5
5000.00,3600,M002,1000.0
50.25,43200,M003,10.2
EOF

# Upload for batch prediction
curl -X POST "http://localhost:8000/api/v1/predict/batch" \
  -F "file=@test_transactions.csv"
```

### Get Model Explanation (SHAP)
```bash
curl -X POST "http://localhost:8000/api/v1/explain" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 5000.00,
    "time": 3600,
    "merchant_id": "M002",
    "distance": 1000.0
  }'
```

**Response includes:**
- Base value (model average prediction)
- SHAP values for each feature
- Feature importance ranking
- Feature contribution to fraud probability

---

## Step 8: Interactive API Documentation

**Swagger UI (Auto-Generated):**
```
http://localhost:8000/docs
```

**Features:**
- Try out all endpoints
- See request/response schemas
- Full API documentation
- Interactive testing

---

## Step 9: Docker Deployment (1 minute)

### Build Docker Image
```bash
docker build -t fraud-detection:v1 .
```

### Run Container
```bash
docker run -p 8000:8000 fraud-detection:v1
```

✅ **Test Container:**
```bash
curl http://localhost:8000/health
```

---

## Step 10: Advanced - Train & Deploy Your Own Model

### Train with Custom Hyperparameters
```python
import yaml
from src.models.trainer import ModelTrainer
from src.data.transformation import DataTransformer

# Load data
transformer = DataTransformer()
X_train, y_train = transformer.fit_transform('data/creditcard.csv')

# Train ensemble
trainer = ModelTrainer()
ensemble = trainer.train_ensemble(
    X_train, y_train,
    xgb_params={'max_depth': 8, 'learning_rate': 0.1},
    smote_enabled=True
)

# Save model
trainer.save_model(ensemble, 'models/my_ensemble.joblib')
print(f"Model saved! Accuracy: {ensemble.score(X_train, y_train):.4f}")
```

---

## Complete Workflow (All Steps)

```bash
#!/bin/bash

# Full automated workflow
echo "Step 1: Install dependencies..."
pip install -r requirements.txt

echo "Step 2: Download dataset..."
kaggle datasets download -d kartik2112/fraud-detection
unzip fraud-detection.zip -d data/

echo "Step 3: Train models..."
python main.py --mode train --all-models

echo "Step 4: Run tests..."
pytest tests/ -v --cov=src

echo "Step 5: Start MLflow..."
mlflow ui --port 5000 &

echo "Step 6: Start API server..."
uvicorn src.api.main:app --port 8000

echo "✅ All systems running!"
echo "API: http://localhost:8000/docs"
echo "MLflow: http://localhost:5000"
```

---

## Troubleshooting

### Issue: "ModuleNotFoundError: No module named 'xgboost'"
**Solution:**
```bash
pip install --upgrade xgboost lightgbm catboost
```

### Issue: "Port 8000 already in use"
**Solution:**
```bash
uvicorn src.api.main:app --port 8001  # Use different port
```

### Issue: "Dataset not found"
**Solution:**
```bash
ls -la data/
# Make sure creditcard.csv exists in data/ folder
```

### Issue: "Out of memory during training"
**Solution:**
```bash
# Train single model instead
python main.py --mode train --model xgboost --sample 0.5  # Use 50% of data
```

---

## Performance Benchmarks

| Component | Time | Resource |
|-----------|------|----------|
| Install dependencies | ~2 mins | 500MB disk |
| Download dataset (1M rows) | ~3 mins | 150MB disk |
| Train all 6 models | ~5 mins | 4GB RAM |
| Run tests | ~1 min | 2GB RAM |
| API inference (single) | <100ms | 1GB RAM |
| API inference (batch 1000) | ~50ms per transaction | 2GB RAM |

---

## Next Steps

1. **Experiment with hyperparameters** - Edit `params.yaml`
2. **Add custom features** - Modify `src/data/transformation.py`
3. **Deploy to AWS** - Follow README.md production section
4. **Monitor model drift** - Check `src/monitoring/drift_detection.py`
5. **Integrate with your application** - Use FastAPI endpoints

---

## Key Endpoints Summary

```
POST   /api/v1/predict              → Single fraud prediction
POST   /api/v1/predict/batch        → Batch predictions (CSV)
POST   /api/v1/explain              → SHAP explanation
GET    /health                      → Server health check
GET    /model/version               → Model information
```

---

## Support & Questions

- **GitHub Issues**: github.com/Keshu017/Fraud-Detection-ML-Enhanced/issues
- **Documentation**: README.md in repository
- **API Docs**: http://localhost:8000/docs (after starting API)

---

**🎉 Congratulations! Your fraud detection system is now running!**
