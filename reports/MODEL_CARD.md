# بطاقة النموذج | Model Card
## الحالة
READY_FOR_REVIEW — جودة التفسير تحتاج مراجعة بشرية، وليست درجة آلية.
## الغرض والاستخدام | Purpose and use
This project is an educational synthetic credit-risk classification exercise. The model is intended to demonstrate a reproducible machine-learning workflow including temporal validation, model selection, calibration, threshold selection, capacity control, and evidence generation. It is not intended for real lending, financing, or other high-stakes production decisions.
بيانات Tamweel Lite اصطناعية؛ الهدف حدث خلال 90 يومًا بعد الطلب. 22 خاصية متاحة وقت الطلب؛ لا معرفات أو تواريخ أو هدف في المدخلات. الاستخدام التعليمي فقط؛ لا قرارات تمويل فعلية.
## البيانات والتحقق | Data and validation
حوض التدريب والاختيار: 6576 طلبًا. المعايرة: 836 طلبًا و78 موجبًا. حجز عملاء المعايرة، 90 يومًا لنضج التسميات، وOOF أمامي متداخل مع فصل العملاء في المستويين. طيات المقارنة: 2023Q1 و2023Q3 و2024Q1. نستبعد النتائج التي لم تنضج قبل الأدوار التالية؛ آخر بيانات التدريب لا تستخدم تلقائيًا.
The model-selection evidence comes from 2,155 nested forward OOF rows across three forward periods. These OOF predictions were generated without using each outer validation period for fitting its corresponding model. However, OOF performance is still development evidence rather than proof of future performance, and temporal or population drift may change the results after deployment.
## النموذج والقرار | Model selection
KEEP SINGLE / Logistic. انحراف AP المرجعي: 0.029808. ارجع إلى artifacts/ensemble_comparison.csv وday5_fold_scores.csv للأرقام الكاملة.
The ensemble worth-it gate did not support replacing the single model. Logistic Regression achieved the highest mean OOF Average Precision among the single models (0.39166), while Equal, Weighted, and Stack achieved 0.37170, 0.38942, and 0.38314 respectively. All ensemble candidates failed the predefined worth-it gate, so the final procedure kept Logistic Regression. This avoids adding ensemble complexity without demonstrated OOF improvement.
## المعايرة | Calibration
Sigmoid على عينة محجوزة من التدريب والاختيار؛ الرسم والمقاييس تشخيص على عينة تعلم المعاير، وليسا اختبارًا مستقلاً. لا ادعاء بتحسن على تحدٍّ مجهول التسميات.
Calibration used a separate calibration-only set of 836 rows containing 78 positive cases (9.33% prevalence). On these calibration-fit rows, the raw Brier score was 0.076473 and ECE was 0.021121, while the sigmoid values were 0.078058 and 0.034871. Therefore, sigmoid calibration did not improve these diagnostic metrics on the calibration-fit sample. These are calibration-fit diagnostics and should not be interpreted as independent final-test performance.
## السياسة والسعة والمناطق | Policy and regions
خسارة 10 FN + FP، عتبة OOF الخام 0.16892161427109176 والمنقولة 0.12225843144286948. سعة الدفعة 300؛ المرشحون 330؛ الإشارات النهائية 300. كتلة الدرجات المتساوية لا تقسم. 1=إشارة مراجعة تعليمية، 0=عدم رفع الإشارة.
The frozen OOF policy used a raw threshold of approximately 0.16892 and satisfied the configured 12% review-capacity constraint, with a maximum period flag fraction of approximately 11.75%. On the 2,500-row unlabeled challenge batch, 330 requests were threshold-eligible, but capacity was 300, so 30 requests were removed and 300 were finally flagged. Regional OOF false-positive rates varied from approximately 6.31% to 10.29%. These regional comparisons are diagnostics only and do not constitute a fairness certification.
## التفسير وحدوده | Explanation scope
تفسير اليوم الرابع يخص نموذج اليوم الرابع؛ لا يُنسب تلقائيًا إلى هذه النسخة. تغيير النموذج أو خصائصه أو معايرته يستلزم مراجعة التفسير.
The Day 4 explanation evidence helps explain model behavior and probability calibration, but it does not directly explain the same final model selected on Day 5. Day 5 selected Logistic Regression, whereas the earlier Day 4 interpretability analysis was produced for its Day 4 fitted model. Therefore, feature-level explanations for the final Day 5 model should be recomputed and rechecked before they are presented as explanations of the final model.
## المتابعة والقيود | Monitoring and limitations
If this procedure were evaluated in a controlled future setting, monitoring should include predictive and calibration drift, score and feature-distribution changes, review-capacity utilization, threshold-eligible and final flag rates, Brier score and ECE when labels become available, and regional diagnostics such as false-positive rate and recall. Material drift, capacity violations, or deterioration in calibration or regional behavior should trigger review and new development evidence rather than silent threshold changes.
OOF يستخدم للاختيار، وثلاث فترات ليست اختبار دلالة. العتبة قد تتغير سعتها عند نقلها إلى نموذج معاد التدريب. لا تسميات للتحدي، ولا مقاييس أداء أو شهادة عدالة له. البيانات لا تمثل أشخاصًا أو مناطق حقيقية.
## إعادة الإنتاج | Reproducibility
seed=211; trees=80; CPU مجاني. الإصدارات في artifacts/environment.json. المصادر/بصماتها في artifacts/day5_run.json. النموذج artifacts/final_model؛ inference.predict يعيد ID واحتمالًا؛ السياسة تطبق بعد جمع الدفعة. replay_final يعيد التنبؤ المحفوظ؛ rebuild_final يعيد التدريب. لا تدرب النموذج بعد تثبيت المعاير.
## ملكيتك للتسليم | Submission ownership
أكمل أدلة الأيام السابقة والعرض، واحفظ الدفتر المنفذ، ثم سجل SHA وtag مستودعك في قناة التسليم الخاصة. دعم الدورة f486fc50dd9ac8403016facc58cf6a62beb4abf4 ليس SHA تسليمك.
