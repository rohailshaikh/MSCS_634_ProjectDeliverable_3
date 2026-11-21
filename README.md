# MSCS 634 Project Deliverable 3

## Classification, Clustering, and Pattern Mining

**Student:** Rohail Shaikh , Phaneendra Vanapamula, Abhilash Marthala  
**Course:** Advanced Big Data and Data Mining (MSCS-634-M50)

## Dataset and preparation

This continues Deliverables 1 and 2 using the same UCI Bike Sharing dataset: **17,379 hourly
records and 17 cleaned columns**, covering 2011–2012 in Washington, D.C. The included CSV
is unchanged. The notebook recreates the 13 calendar features from Deliverable 2.

**High demand means at least 213 rentals/hour**, the 75th percentile of January–September 2011.
This threshold is fixed before every validation and test period. All rental count columns
are excluded from classifier and clustering inputs.

Training uses **13,747 records through July 2012**; testing uses **3,632 records from
August–December 2012**. Five expanding validation folds cover two calendar months each.
Scaling and encoding are fitted inside training folds.

## Classification and tuning

- **Decision Tree:** Default baseline plus a tuned version.
- **k-NN:** 15 Euclidean-distance neighbors, with closer neighbors weighted more heavily.

Numeric inputs are standardized and categories are one-hot encoded. Grid search compares
**24 tree settings** using high-demand F1:

| Parameter | Values | Purpose |
|---|---|---|
| `max_depth` | 4, 6, 8, None | Control tree complexity. |
| `min_samples_leaf` | 1, 10, 30 | Reduce noisy splits. |
| `class_weight` | None, balanced | Check whether class weighting improves F1. |

The selected settings are **unrestricted depth, 10 samples per leaf, no class reweighting**.
Mean CV F1 rises from **0.7705 to 0.8256**. The larger leaf size constrains the tree.

## Classification results

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | CV F1 |
|---|---:|---:|---:|---:|---:|---:|
| Decision Tree baseline | 0.7494 | 0.8079 | 0.6408 | 0.7147 | 0.7473 | 0.7705 |
| k-NN | 0.8926 | 0.9122 | 0.8640 | 0.8874 | 0.9629 | 0.8220 |
| Decision Tree tuned | 0.8706 | 0.8540 | 0.8876 | 0.8705 | 0.9447 | 0.8256 |

F1, precision, and recall refer to high demand. Higher values are better. ROC-AUC uses
probabilities. CV F1 averages the five calendar folds.

The **tuned tree leads CV F1**, narrowly ahead of k-NN, and remains the validation-selected
model. **k-NN has the best test F1 (0.8874), accuracy (89.26%), and ROC-AUC (0.9629)**.
The tuned tree has higher recall, missing 200 high-demand hours versus 242 for k-NN.
k-NN gives fewer false warnings: 148 versus 270. The small CV difference does not establish
a consistent future winner. Test results do not change parameter selection.

## K-Means clustering

K-Means uses cyclic hour, temperature, humidity, wind, working-day status, and encoded
weather. Numeric inputs are standardized; rental counts and demand classes are excluded.
The model and preprocessing are fitted on training hours.

Among k = 2–6, **k = 2** has the highest silhouette on a fixed 2,000-hour training sample:
**0.1946**. Separation is modest. PCA shows the groups in two dimensions, covering **46.9%**
of transformed-feature variance; clustering uses the full feature space.

| Cluster | Hours | Temperature | Humidity | Daytime | Mean rentals/hour |
|---|---:|---:|---:|---:|---:|
| 0 | 7,429 | 0.448 | 0.744 | 38.5% | 98.7 |
| 1 | 6,318 | 0.553 | 0.480 | 82.4% | 261.4 |

Temperature and humidity remain in normalized units. Cluster 0 is cooler and more humid;
Cluster 1 is warmer, less humid, and mostly daytime. Working-day rates are similar at around
68%. Counts are added after clustering to describe demand. The groups overlap, but their
different demand profiles can inform condition-based planning.

## FP-Growth association rules

Each hour is a transaction of time period, day type, weather, temperature band, and demand
class. Temperature bands use training tertiles: cool ≤0.38, mild >0.38 to 0.62, warm >0.62.

Rules require **support ≥3%, confidence ≥70%, lift ≥1.2**, and only high demand as the outcome.
Itemsets contain at most three items. **Six rules qualify**; two useful examples are:

| Conditions → high demand | Train support | Train confidence | Train lift | Test confidence |
|---|---:|---:|---:|---:|
| period=evening & temperature=warm | 6.45% | 96.52% | 3.01 | 99.10% |
| day=nonworking & period=midday | 5.70% | 71.70% | 2.24 | 85.37% |

Support measures the frequency of both conditions and high demand. Confidence is the
high-demand rate among matching hours. Lift compares confidence with the overall rate.
Warm evenings (16:00–19:59) provide the strongest rule, supported by **887 training hours**;
later confidence uses **221 matching hours**. Non-working midday means 10:00–15:59.
Both patterns identify windows to review for extra bicycles or staff.

Rules are selected on training data, then checked on later records. The high-demand base
rate rises from **32.1% to 49.0%**, so confidence and lift must be interpreted against that
change. Rules describe associations, not causes or guaranteed forecasts.

## Visualizations and challenges

The notebook includes confusion matrices, ROC curves, F1 comparisons, silhouette scores,
PCA clusters, demand boxplots, and rule-confidence comparisons, with explanations.

| Challenge | Response |
|---|---|
| Changing demand and unequal class proportions. | Fixed the target threshold and used calendar validation, F1, recall, and ROC-AUC. |
| Different feature scales. | Fitted scaling on training records. |
| Count columns reveal the outcome. | Excluded counts from predictive and clustering inputs. |
| Tiny tree leaves and trivial rules. | Tuned leaf size and required supported, demand-related rules. |

These historical system-wide results support planning analysis. Advance forecasts require
weather forecasts available at prediction time. Deliverable 1 already explored the full
dataset, so testing is a later-period check. Deliverable 4 can combine these findings with
the earlier cleaning, EDA, and regression results.

## Files and running

- `ProjectDeliverable_3_Rohail_Shaikh.ipynb`: Executed notebook.
- `bike_sharing_cleaned.csv`: Same cleaned dataset.
- `README.md` and `requirements.txt`: Documentation and dependencies.
- `results/`: Classification metrics, cluster profiles, and all six association rules as CSVs.

Extract the ZIP. In Anaconda Prompt, open the extracted folder and run:

```bash
python -m pip install -r requirements.txt
jupyter notebook ProjectDeliverable_3_Rohail_Shaikh.ipynb
```

Choose **Restart Kernel and Run All Cells**. Keep the dataset beside the notebook.

## Submission

Upload the four root files and `results` folder to `MSCS_634_ProjectDeliverable_3` on GitHub.
Ensure public or instructor access, then submit the repository URL on Blackboard.

## References

Fanaee-T, H. (2013). *Bike sharing* [Data set]. UCI Machine Learning Repository.
https://doi.org/10.24432/C5W894. [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Cleaned names and calendar features were prepared in this project.

Wu, D. (2024). *Data mining with Python: Theory, application, and case studies*. CRC Press.

Implementation documentation: [GridSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html),
[KMeans](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html),
and [mlxtend association rules](https://rasbt.github.io/mlxtend/user_guide/frequent_patterns/association_rules/).
