# قرار التجميع

KEEP SINGLE — Logistic

The ensemble worth-it gate did not support replacing the single model. Logistic Regression achieved the highest mean OOF Average Precision among the single models (0.39166), while Equal, Weighted, and Stack achieved 0.37170, 0.38942, and 0.38314 respectively. All ensemble candidates failed the predefined worth-it gate, so the final procedure kept Logistic Regression. This avoids adding ensemble complexity without demonstrated OOF improvement.

The model-selection evidence comes from 2,155 nested forward OOF rows across three forward periods. These OOF predictions were generated without using each outer validation period for fitting its corresponding model. However, OOF performance is still development evidence rather than proof of future performance, and temporal or population drift may change the results after deployment.

الدليل: artifacts/ensemble_comparison.csv وday5_ensemble_gate.json. SD وصفي، وليس اختبار دلالة.
