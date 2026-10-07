# بطاقة قرارك — Tamweel Lite

**الحالة:** جاهزة للمراجعة؛ لا تعني اعتمادًا أو درجة
**مصدر الأرقام:** LIVE · **الاستراتيجية:** weighted · **الصفوف:** 5,039 OOF

**المهمة:** الفئة الموجبة `default_within_90d=1` تعني حدث تعثر اصطناعي خلال90 يومًا بعد الطلب. كل طية تحقق طلبات لاحقة، وتستبعد عملاءها من التدريب وتشترط نضج نتيجة التدريب قبل بدايتها. المعرّفات والتاريخ خارج المدخلات.

| الدليل | القيمة |
|---|---:|
| العتبة المقيدة، بالقيمة الكاملة | 0.6583471436014694 |
| عتبة أقل خسارة دون قيد | 0.44863935722081005 |
| Recall | 40.89% |
| Precision | 29.85% |
| AP مجمع منOOF | 0.3100 |
| الإشارات | 526 من 5,039 |
| FN / FP | 227 / 369 |
| الخسارة التعليمية | 2639 وحدة |
| خسارة0.5 | 2403 وحدة؛ ضمن السعة: False |
| التغير عن0.5 | +236 وحدة؛ الموجب زيادة |
| خسارة لكل10,000 طلب، تطبيع حسابي | 5237.15 وحدة |
| فجوة معدل الإنذار الخاطئ بين المناطق | 0.648 نقطة مئوية |

**القاعدة:** درجة ≥ 0.6583471436014694 تعني إشارة مراجعة داخل التمرين؛ غير ذلك بلا إشارة. لا تتخذ موافقة أو رفض تمويل حقيقي. احفظ الدقة الكاملة؛ تقريب العتبة قد يغيّر حجم الطابور.

**السياسة:** FN=10 وFP=1 وحدات تعليمية، وسعة 12% لكل فترة بعد التقريب لأسفل. ليست ريالات فعلية أو رسوم أدوات أو خصمًا من الدرجة.

**دليل السعة:** الفترة 1: 137/195, الفترة 2: 183/200, الفترة 3: 206/207.

## لماذا اخترت هذه العتبة؟
The selected threshold of approximately 0.6583 is defensible because it satisfies the 12% review-capacity constraint in every validation period. It flagged 526 of 5,039 eligible OOF applications (10.44%) overall. At the period level, the flag rates were 8.39%, 10.93%, and 11.89%, all within the 12% capacity limit.

## الخسارة والسعة
The unconstrained minimum-loss threshold was approximately 0.4486 and produced 2,275 simulated loss units, but it flagged 22.64% of applications and therefore violated the capacity constraint. The capacity-constrained threshold of approximately 0.6583 produced 2,639 simulated loss units and flagged 10.44% of applications. Therefore, operational feasibility required accepting a higher simulated loss in exchange for satisfying the review-capacity constraint.

## فرق المناطق وما يحتاج إلى مراجعة
Using the shared threshold, the observed regional false-positive rates were 8.09% for central, 8.29% for western, 7.72% for eastern, and 7.64% for other. The observed FPR gap between the highest and lowest regions was approximately 0.6484 percentage points. This is a descriptive result from synthetic data without confidence intervals or significance tests, so it does not establish fairness or unfairness and requires further validation.

## حدود النتيجة
This is an OOF development analysis rather than a final independent test. OOF coverage includes 5,039 of 10,000 rows because 4,961 observations are warm-up rows without OOF predictions. The current weighted scores are not calibrated probabilities, and the decision policy uses simplified simulated costs and a fixed 12% capacity assumption. Therefore, these results should not be interpreted as final real-world performance.

OOF تغطي 50.39% من التدريب و100% من الصفوف المؤهلة؛ 4,961 صفًا تمهيديًا بلا تنبؤ. اختيار العتبة وتقدير خسارتها هنا يستخدمان أهدافOOF نفسها؛ هذه نتيجة تطوير لا اختبار نهائي. لم نستخدم التحدي. المقارنة الجغرافية وصفية وليست شهادة عدالة، والأوزان لا تضمن معايرة الدرجات.

## سؤالك الأول: لماذا قد تخدعكAccuracy؟
Accuracy can be misleading because the positive class is rare. The pooled OOF positive rate is approximately 7.62%, so a rule that flags nobody achieves about 92.38% accuracy while having zero recall and missing every positive case. Therefore, accuracy alone does not adequately describe performance for this imbalanced classification problem.

## سؤالك الثاني: لماذا تختار علىOOF؟
Threshold selection uses out-of-fold predictions because each OOF prediction is produced for an observation that was not used to fit that fold's model. This provides more honest development evidence for threshold selection than training predictions. Challenge data must remain untouched so that it can provide an independent final evaluation rather than becoming part of model or threshold development.

أدلتك في `artifacts/threshold_metrics.json` و`day3_period_capacity.csv` و`day3_region_audit.csv` و`day3_cost_sensitivity.csv` و`cost_curve.png`. الحساسية سيناريوهات ±20% لخسارةFN، وليست فترات ثقة. راجع السعة والمعايرة عند تغير البيانات؛ لا تفترض ثباتهما مستقبلًا.
