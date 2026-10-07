# تقريرك: التفسير والمعايرة — Tamweel Lite

**الحالة:** جاهز للمراجعة؛ لا يعني اعتمادًا أو درجة

**مصدر التفسير:** LIVE. **النموذج والمعايرة:** LIVE. **السعة:** CAPACITY_REVIEW_REQUIRED.

## النموذج والأدوار
LightGBM موزون، 80 شجرة. الهدف حدث تعثر اصطناعي خلال90 يومًا بعد الطلب. الأدوار منفصلة زمنيًا وبالعملاء: تدريب 2516، معايرة 584 (40 موجب)، سياسة 589، تقييم 1733. الفجوات والتداخلات مستبعدة. سبق استخدام بيانات التقييم في الدورة، فهي ليست اختبارًا نهائيًا لم يمسّ.

## التفسير العام والمحلي
Permutation يقيس انخفاضAP على التقييم؛ إشارات المنطقة تُبدّل معًا. SHAP يفسر النموذج الخام بوحدةlog-odds وخلفية مسارات أشجار التدريب. base+sum(SHAP)=raw margin، ثمsigmoid للمجموع فقط. القيم ليست نقاط احتمال ولا تفسيرًا مباشرًا للنموذج المعاير.

Global and local explanations were consistent in identifying bureau_score and dti as major model drivers. On the 1,733-request held-out evaluation set, permutation importance showed the largest AP drops for bureau_score (0.1270) and dti (0.0690). In the global SHAP analysis of 300 sampled evaluation requests, bureau_score had the largest mean absolute SHAP value (0.9042 log-odds), followed by dti (0.5437). For the high-score request TR-009585, bureau_score=497 contributed about +2.27 log-odds and dti=1.281 contributed about +1.20 log-odds, making them the strongest positive local drivers.

The SHAP outputs are expressed in raw log-odds, not probability points. The global SHAP beeswarm therefore shows how each feature moves the model's raw log-odds output. For TR-009585, the SHAP contributions move the prediction from the baseline E[f(X)] of about -1.79 to f(x)=2.232. The resulting raw probability was 0.9031, while the separately calibrated probability was 0.4795.

الطلب الاصطناعي TR-009585: الدرجة الخام 0.90308 والاحتمال المعاير 0.47952. اختير أعلى درجة داخل عينةSHAP دون استخدام النتيجة الفعلية.
- استخدم النموذج درجة ائتمانية اصطناعية عند الطلب بالقيمة 497 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+2.2693 log-odds؛ قيمة معوضة: False)
- استخدم النموذج نسبة الالتزام مع القسط المقترح إلى الدخل بالقيمة 1.2806 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+1.1981 log-odds؛ قيمة معوضة: False)
- استخدم النموذج مبلغ التمويل المطلوب بالقيمة 93437.3 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+0.2007 log-odds؛ قيمة معوضة: False)

The explanations describe model associations and should not be interpreted as causal reasons. Correlated features can share predictive signal, so individual feature importance should be interpreted cautiously. For example, permutation importance and SHAP both highlight bureau_score and dti, but this does not establish that either feature causes default. Local reason statements explain this model prediction for a particular request and are not causal or universally applicable explanations.

## دليل المعايرة
على 1733 صفًا و139 موجب: Brier 0.113027 → 0.067112؛ ECE 0.146871 → 0.022486. عشر حاويات متساوية العرض مع أعدادها فيday4_reliability_bins.csv. AP 0.258677 → 0.258677؛ ROC-AUC 0.770804 → 0.770804. هذه نتائج هذه العينة وليست ضمانًا لتحسن مستقبلي.

Calibration was evaluated on a separate held-out period containing 1,733 requests. Sigmoid calibration improved probability reliability: the measured Brier-score change (after - before) was -0.04592 and the ECE change was -0.12438, where negative changes indicate improvement. The reliability curve also moved the calibrated probabilities closer to the observed event rates than the raw probabilities.

## الاستقرار
200 تكرارbootstrap صالح بسحب العملاء؛ فترات مئينية95% مع تثبيت النموذج والمعاير. لا تشمل تعلم النموذج أو المعايرة أو الانجراف المستقبلي، ولا تصف احتمال فرد. انحرافAP بين ربعي التقييم وصفي فقط. اختبارbureau_score±1 نُفذ؛ راجع day4_local_stability.csv.

Stability was assessed using 200 customer-cluster bootstrap replicates with the model and calibrator held fixed. The paired 95% percentile interval for Average Precision was 0.197442 to 0.337854 for both raw and calibrated scores. The Brier-score change interval was -0.054177 to -0.037409, remaining below zero across the interval. These intervals quantify sampling variability under customer-cluster resampling; they do not cover model retraining uncertainty, alternative model specifications, future distribution shift, or causal validity.

## العتبة ومنطقة المراجعة
العتبة الخام 0.5881953696965011 اختيرت علىpolicy بخسارة10×FN+FP وسقف12% ثم نُقلت إلى 0.17331013263107387. لم تعدل باستخدام التقييم. المنطقة[0.15331, 0.19331] تشخيصية بعرض±0.02 وليست فترة ثقة. الاتحاد يحسب الطلب مرة واحدة.
- 2024Q3: السقف 100، الإشارات 97، اتحاد المراجعة 109.
- 2024Q4: السقف 107، الإشارات 109، اتحاد المراجعة 122.

The transported calibrated threshold was 0.1733. Capacity was assessed separately by evaluation period. In 2024Q3, capacity was 100 requests: 97 risk flags were within capacity, but adding 23 near-threshold cases produced 109 candidate reviews, exceeding capacity. In 2024Q4, capacity was 107 requests, while there were 109 risk flags and 122 candidate reviews after including 21 near-threshold cases. Therefore the result was CAPACITY_REVIEW_REQUIRED. This is an evaluation finding and the threshold should not be retuned on the evaluation data.

عند تجاوز السعة، وثّق الحاجة إلى تصميم سياسة جديدة على بيانات تطوير وتقييمها بدليل جديد. لا ترفع السقف ولا تقص الحالات بعد رؤية النتيجة. التفسير ليس سببية أو شهادة عدالة، والخسارة وحدات تعليمية لا رسوم أو خصم درجات. لا يستخدم هذا التمرين لتمويل حقيقي.
