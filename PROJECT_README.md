# Tamweel Lite | مشروعك النهائي

## الملخص التنفيذي
تم اختيار Logistic Regression كنموذج نهائي لأن نماذج التجميع لم تحقق شرط التحسن المحدد مسبقًا. بلغ متوسط Average Precision للنموذج 0.39166 على تنبؤات OOF. تم تثبيت عتبة OOF الخام عند نحو 0.16892، ثم نُقلت بعد المعايرة إلى المقياس المعاير. في بيانات التحدي البالغة 2,500 طلب، تجاوز العتبة 330 طلبًا، لكن سعة المراجعة البالغة 12% سمحت بـ300 طلب فقط، لذلك أزيل 30 طلبًا بواسطة قيد السعة. النتائج تعليمية ولا تمثل اعتمادًا لنظام تمويل حقيقي أو شهادة عدالة.

## Executive summary
Logistic Regression was retained as the final model because none of the ensemble candidates passed the predefined worth-it gate. Its mean OOF Average Precision was 0.39166. The raw OOF decision threshold was frozen at approximately 0.16892 and then transported to the calibrated probability scale. For the 2,500-request unlabeled challenge batch, 330 requests were threshold-eligible, while the 12% review capacity allowed 300 final flags, removing 30 requests through the capacity rule. The results are educational evidence and do not constitute approval for real lending use or a fairness certification.

Decision: KEEP SINGLE / Logistic. Full-batch flags: 300/2500.

اقرأ reports/MODEL_CARD.md والسياسة في artifacts/final_policy.json. الحزمة للتدريب؛ ليست نتيجة تقييم نهائية أو إثبات تسليم. ادمج أدلة أيامك السابقة واحفظ الدفتر المنفذ والعرض.
