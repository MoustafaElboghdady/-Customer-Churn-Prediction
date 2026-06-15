# Customer Churn Prediction — Technical Notebook

---

## 1. Import Libraries

```python
import pandas as pd
import numpy as np
from scipy import stats
import statsmodels.formula.api as smf
import statsmodels.api as sm
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, roc_auc_score, ConfusionMatrixDisplay
from sklearn import metrics
from statsmodels.stats.proportion import proportion_confint, proportions_ztest
import matplotlib.pyplot as plt
import seaborn as sns
from sqlalchemy import create_engine
from datetime import datetime, timedelta
```

---

## 2. Load Data

```python
# Load the raw dataset exported from the source system
df = pd.read_csv(r'F:\WORK\Orient Group\Sources\الكل 11-05-2026.csv')
```

---

## 3. Initial Exploration

```python
# Inspect data shape, numeric distributions, and column types
df.describe()
df.columns
print(df.dtypes)
```

---

## 4. Data Cleaning & Feature Engineering

```python
# Remove duplicates and convert date columns to proper datetime format
df = df.drop_duplicates()
df['DayDate']    = pd.to_datetime(df['DayDate'])
df['winning_date'] = pd.to_datetime(df['DeliveryDate'])
df['travel_date']  = pd.to_datetime(df['DeliveryDate2'])
df['DeliveryDate']  = pd.to_datetime(df['DeliveryDate'])
df['DeliveryDate2'] = pd.to_datetime(df['DeliveryDate2'])

# Derive travel and winning status from date values
# 1900-01-01 is used as a null placeholder in the source system
df['travel_status']  = df['DeliveryDate2'].apply(lambda x: "travelled" if x > pd.Timestamp('1900-01-01') else "not_travelled")
df['winning_status'] = df['DeliveryDate'].apply(lambda x: "winner" if x > pd.Timestamp('1900-01-01') else "not_winner")

# Aggregate arrears and payment delay indicators from winner/non-winner sub-columns
df['total_arears']  = df['قيمة المتأخرات - فائز'] + df['قيمة المتأخرات - غير فائز']
df['pay_status']    = df['عدد متأخر - غير فائز'] + df['عدد متأخر - فائز']
df['customer_status'] = df['عدد سحب غير فائز'] + df['عدد سحب فائز']
df['refund_amount'] = df['قيمة سداد السحب غير فائز'] + df['قيمة سداد السحب فائز']
```

---

## 5. Select Relevant Columns

```python
# Keep only the columns needed for analysis and modeling
df_clean = df[[
    'اسم الفرع', 'DayDate',
    'عدد الربط', 'عدد الفردي', 'عدد الربط - صافي', 'عدد الفردي - صافي',
    'قيمة السداد الفردي - صافي', 'كود العميل', 'اسم العميل', 'عدد الأقساط',
    'total_arears', 'pay_status', 'customer_status', 'refund_amount',
    'winning_date', 'travel_date', 'winning_status', 'travel_status'
]]
```

---

## 6. Encode Business Labels

```python
# Convert numeric flags to meaningful categorical labels
df_clean['customer_status'] = df_clean['customer_status'].apply(lambda x: "active" if x == 1 else "lost")
df_clean['pay_status']      = df_clean['pay_status'].apply(lambda x: "delay" if x == 1 else "committed")
df_clean['customer_type']   = df_clean['عدد الفردي'].apply(lambda x: "sinle" if x == 1 else "group")

# Drop the net payment column (not needed for modeling)
df_clean.drop(columns='قيمة السداد الفردي - صافي', inplace=True)
```

---

## 7. Rename Columns to English

```python
# Standardize column names for easier downstream use
df_clean['branch_name']   = df_clean['اسم الفرع']
df_clean['customer_name'] = df_clean['اسم العميل']
df_clean['customer_code'] = df_clean['كود العميل']
df_clean['instal_amount'] = df['قيمة القسط']
df_clean['instal_count']  = df_clean['عدد الأقساط']
df_clean['total_paid']    = df['إجمالي قيمة السداد']

# Remove the original Arabic columns that have been replaced
df_clean.drop(columns=['اسم الفرع', 'كود العميل', 'اسم العميل', 'عدد الأقساط'], inplace=True)
```

---

## 8. Prepare Regression Dataset

```python
# Select only the features needed for the logistic regression model
df_regression = df_clean[[
    'customer_code', 'customer_status', 'customer_type',
    'instal_amount', 'instal_count', 'winning_status'
]]
```

---

## 9. Label Encoding

```python
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()

# Encode all categorical variables to numeric for model compatibility
df_regression = df_regression.dropna()
df_regression['customer_status'] = le.fit_transform(df_regression['customer_status'])
df_regression['customer_type']   = le.fit_transform(df_regression['customer_type'])
df_regression['winning_status']  = le.fit_transform(df_regression['winning_status'])

# Separate features (X) from the target variable (Y)
X = df_regression.drop(columns=['customer_code', 'customer_status']).dropna()
Y = df_regression['customer_status'].dropna()
```

---

## 10. Train / Test Split

```python
# Split data: 70% for training, 30% for testing
x_train, x_test, y_train, y_test = train_test_split(X, Y, test_size=0.3, random_state=42)
```

---

## 11. Train Logistic Regression Model

```python
# Train the logistic regression model to predict customer churn
model = LogisticRegression(max_iter=1000)
model.fit(x_train, y_train)
```

---

## 12. Model Evaluation

```python
# Evaluate using Precision, Recall, and F1-Score
precision = metrics.precision_score(y_test, model.predict(x_test))
print(f"Precision: {precision*100:.2f}%")

recall = metrics.recall_score(y_test, model.predict(x_test))
print(f"Recall: {recall*100:.2f}%")

f1 = metrics.f1_score(y_test, model.predict(x_test))
print(f"F1-Score: {f1*100:.2f}%")

# Full classification report
# Class 0 = active customer | Class 1 = lost customer
CLF = classification_report(y_test, model.predict(x_test))
print(CLF)

# AUC Test 
from sklearn.metrics import roc_auc_score
print("AUC:", roc_auc_score(y_test, predicted_probabilities))
#
```

**Results:**
| Metric    | Value |
|-----------|-------|
| Precision | 97.47% |
| Recall    | 99.96% |
| F1-Score  | 98.70% |
| Accuracy  | 98% |
| AUC       | 96% |

---

## 13. Confusion Matrix

```python
# Visualize the classification performance as a confusion matrix
ConfusionMatrixDisplay.from_estimator(model, x_test, y_test)
plt.show()
```

---

## 14. Churn Probability Scores

```python
# Generate probability score for each customer in the test set
predicted_probabilities = model.predict_proba(x_test)[:, 1]

# Attach the probability score (scaled to 0–100) back to the main dataframe
df_clean.loc[y_test.index, 'Churn Probability'] = predicted_probabilities * 100

# Remove rows where probability could not be calculated (NaN)
df_clean = df_clean[df_clean['Churn Probability'].notnull()]

df_clean.sample(10)
```
