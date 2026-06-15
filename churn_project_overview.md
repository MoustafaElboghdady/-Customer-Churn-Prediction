# مشروع التنبؤ بفقد العملاء — وصف المشروع والنتائج
# Customer Churn Prediction — Project Overview & Results

---

## وصف المشروع | Project Description

يهدف هذا المشروع إلى بناء نموذج تنبؤي قادر على تحديد العملاء الأكثر عرضة للانسحاب (الفقد) من خدمات الشركة. تم تطوير المشروع بالكامل باستخدام بيانات حقيقية مستخرجة من النظام الداخلي للشركة، وتغطي تاريخ حركة العملاء منذ عام 2023 حتى مايو 2026.

This project aims to build a predictive model capable of identifying customers most at risk of churning from the company's services. The project was developed entirely using real data extracted from the company's internal system, covering customer activity from 2023 through May 2026.

---

## المشكلة | The Problem

 
تعاني الشركة من فقد مستمر في قاعدة عملائها دون وجود آلية استباقية للكشف المبكر عن العملاء المعرضين للانسحاب. كان الاعتماد على المتابعة اليدوية بعد وقوع الفقد فعلياً، مما يجعل التدخل متأخراً وغير فعّال في أغلب الأحيان.

  
The company has been experiencing continuous customer loss without a proactive mechanism for early detection of at-risk customers. Reliance on manual follow-up after churn had already occurred made interventions late and largely ineffective.

---

## البيانات المستخدمة | Data Used

   
تم استخدام قاعدة بيانات تحتوي على **20,914 سجل عميل** تشمل المتغيرات التالية:

  
A dataset containing **20,914 customer records** was used, including the following variables:

| المتغير / Variable | الوصف / Description |
|---|---|
| `customer_status` | حالة العميل (active / lost) — Target Variable |
| `customer_type` | نوع العميل (فردي / مجموعة) — single / group |
| `instal_amount` | قيمة القسط الشهري — Monthly installment amount |
| `instal_count` | عدد الأقساط المسددة — Number of installments paid |
| `winning_status` | هل العميل فائز برحلة — Whether the customer won a trip |
| `total_paid` | إجمالي ما دفعه العميل — Total amount paid by customer |
| `pay_status` | حالة الالتزام بالسداد — Payment commitment status |

---

## منهجية العمل | Methodology

   
تم تنفيذ المشروع عبر المراحل التالية بالتسلسل:

  
The project was executed through the following sequential stages:

1. **تنظيف البيانات / Data Cleaning** — إزالة التكرارات، تحويل أنواع البيانات، معالجة القيم الخاصة (1900-01-01 كـ null placeholder)
2. **هندسة المتغيرات / Feature Engineering** — اشتقاق متغيرات جديدة (winning_status, travel_status, pay_status, customer_type)
3. **الترميز / Label Encoding** — تحويل المتغيرات النصية إلى قيم رقمية
4. **تقسيم البيانات / Train-Test Split** — 70% تدريب / 30% اختبار
5. **بناء النموذج / Model Training** — Logistic Regression مع 1000 iteration
6. **التقييم / Evaluation** — Precision, Recall, F1-Score, Confusion Matrix
7. **حساب الاحتمالية / Probability Scoring** — إضافة Churn Probability لكل عميل بمقياس 0–100

---

## النتائج | Results

   
حقق النموذج أداءً استثنائياً في التنبؤ بالعملاء المعرضين للفقد:

  
The model achieved exceptional performance in predicting at-risk customers:

| المقياس / Metric | القيمة / Value |
|---|---|
| **Accuracy**      | **98%** |
| **Precision**     | **97.47%** |
| **Recall**        | **99.96%** |
| **F1-Score**      | **98.70%** |
| **AUC**           | **96%** |

   

قبل وقوع الانسحاب فعلياً، مما يتيح للفريق التدخل المبكر والاحتفاظ بهم.**99.96% من العملاء المعرضين للفقد** يعني ذلك أن النموذج قادر على اكتشاف 


  
This means the model can identify **99.96% of at-risk customers** before they actually churn, enabling the team to intervene early and retain them.

---

## مخرجات النموذج | Model Output

   
ينتج النموذج لكل عميل نسبه مئويه تستخدم لتصميف العميل حسب مستوي الخطر 

  
The model generates a **Churn Probability Score (0–100)** for each customer, used to classify them by risk level:

| مستوى الخطر / Risk Level    | نطاق الاحتمالية / Probability Range | التوصية / Recommendation |

| 🔴 خطر عالي / High Risk     |      > 80%                          | تدخل فوري — Immediate intervention |
| 🟡 خطر متوسط / Medium Risk  |      40% – 80%                      | متابعة مكثفة — Close monitoring |
| 🟢 آمن / Low Risk           |              < 40%                  | متابعة عادية — Routine follow-up |

---

## التوصيات | Recommendations


Based on the model results, the following is recommended:

1.  تطبيق النموذج دورياً (أسبوعياً) على قاعدة العملاء الكاملة لاستخراج قائمة العملاء عالي الخطر.
    Run the model weekly on the full customer base to generate a high-risk customer list.

2. تطوير النموذج مستقبلاً بإضافة متغيرات إضافية (عمر العميل مع الشركة، عدد الشكاوى، آخر تفاعل).
   Enhance the model by adding more features (customer tenure, complaints count, last interaction date).

3. ربط المخرجات بلوحة تحكم تفاعلية (Power BI) لتسهيل المتابعة من قبل الإدارة.
   Connect the output to an interactive dashboard (Power BI) for management-level monitoring.

---

## التقنيات المستخدمة | Technologies Used

| الأداة / Tool           | الاستخدام / Purpose |
|---|---|
| Python (pandas, numpy) | تنظيف البيانات / Data Cleaning |
| scikit-learn           | بناء النموذج / Model Training |
| statsmodels            | التحليل الإحصائي / Statistical Analysis |
| matplotlib / seaborn   | التصور البياني / Visualization |
| VS Code + Jupyter      | بيئة التطوير / Development Environment |

---

*تاريخ المشروع: مايو 2026 — Project Date: May 2026*
