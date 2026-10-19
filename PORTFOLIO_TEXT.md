# Portfolio Presentation Text Assets

This document contains standardized descriptions for showcasing this project on GitHub, CV / resumes, and LinkedIn.

---

## 1. GitHub Repository Description
*(Constraint: Under 160 characters)*

> Geospatial pipeline fusing NASA satellite thermal data, conflict news, and ACLED ground truth to model maritime shipping and fuel rerouting risks.
*(Length: 147 characters)*

---

## 2. Concise CV / Resume Bullet Points

* **Geospatial Data Engineering & Pipeline Design:**  
  Engineered an end-to-end situation-monitoring pipeline aggregating 144k+ NASA FIRMS satellite thermal observations across Ukraine, Yemen, and Iraq; clustered detections into 10,375 discrete events using spatio-temporal DBSCAN and persisted them into PostgreSQL.

* **Machine Learning, Target Leakage Audit & Ground-Truth Validation:**  
  Built supervised classifiers (0.903 F1) predicting conflict association, conducted feature ablation exposing geographic target leakage (retaining 0.819 F1 on physical fire signatures alone), and benchmarked heuristics against 35k+ ACLED ground-truth events (42.5% agreement) to model maritime fleet rerouting risk ($7.91M scenario).

---

## 3. LinkedIn Project Description
*(Target: ~80–120 words | Actual: 110 words)*

Can satellite thermal observations provide objective situation monitoring for commercial shipping corridors?

Collaborating with Ahmet Emre Bingöl on our Boğaziçi University CE49X Data Science final project, we built an end-to-end geospatial monitoring pipeline. Fusing 144,201 NASA FIRMS satellite thermal detections with international news reporting and 35,436 verified ACLED ground-truth conflict incidents, we developed DBSCAN spatio-temporal clustering and a 3-signal industrial flare mask separating petroleum flaring from combat fires. 

Through feature ablation experiments, we audited supervised classifiers for geographic target leakage (0.903 to 0.819 F1) and quantified an illustrative $7.91M commercial fleet fuel-cost exposure across 24 Red Sea alert days.

Full code, figures, and methodology: [GitHub Link]
