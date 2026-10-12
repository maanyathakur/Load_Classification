# Power system load classification

Predict `Light_Load`, `Medium_Load`, or `Maximum_Load` for each 15-minute interval using the supplied electrical measurements and calendar features. The notebook covers preprocessing, exploratory data analysis, feature engineering, model selection, and evaluation.

## Files and data

- `load_classification.ipynb`: complete analysis with saved results and plots.
- `load_data.csv`: unchanged supplied dataset, containing 35,041 records from January to December 2018.

The inputs include energy usage, reactive energy, CO2, power factors, and seconds from midnight (`NSM`). `Load_Type` is the target, and the timestamp provides calendar features.

## Run

Use Python 3.12 or 3.13. Open PowerShell in this folder and run the following setup commands once. The environment is created outside the submission folder.

```powershell
py -m venv "$env:USERPROFILE\assignment2-env"
& "$env:USERPROFILE\assignment2-env\Scripts\python.exe" -m pip install pandas==3.0.1 numpy==2.5.4 scipy==1.18.1 scikit-learn==1.7.2 matplotlib==3.10.7 threadpoolctl==3.7.0 joblib==1.6.0 jupyterlab ipykernel==6.30.1
& "$env:USERPROFILE\assignment2-env\Scripts\python.exe" -m ipykernel install --user --name assignment2-env --display-name "Assignment 2"
```

In VS Code, install Microsoft's Python and Jupyter extensions, open the notebook, select the **Assignment 2** kernel, and click **Run All**. Save the notebook after execution.

Alternatively, launch JupyterLab:

```powershell
& "$env:USERPROFILE\assignment2-env\Scripts\python.exe" -m jupyter lab load_classification.ipynb
```

Select the **Assignment 2** kernel and choose **Restart Kernel and Run All Cells**. Keep the CSV beside the notebook in either case.

## Approach

The solution is combining electrical readings with calendar features to classify the current interval. Comparing sensor-only, calendar-only, and combined inputs helps establish what information is useful.

Since the assignment requires the last month for testing, December is being reserved for the final evaluation. Chronological validation is using earlier months for training and later months for model comparison. This reflects the task of classifying subsequent records rather than randomly mixing neighbouring intervals.

## Methodology

1. **Read and split:** Timestamps are being parsed as day-month-year, and December is being set aside before exploring patterns or choosing models.
2. **Clean:** Invalid sensor readings are being marked as missing, NSM is being rebuilt from timestamps, and repeated timestamps are being resolved by retaining the more complete record. Incomplete rows and large positive readings are being kept to avoid discarding useful examples.
3. **Explore:** Class counts, monthly and daily patterns, and sensor distributions are being compared to identify class imbalance and useful relationships.
4. **Build features:** Six sensor readings are being combined with time of day, weekday, weekend status, and month sine/cosine. Calendar features capture the time patterns seen in the plots.
5. **Compare models:** A majority baseline, decision trees, Random Forest, and histogram gradient boosting are being evaluated on September, October, and November, each time training on earlier months. Tree/forest pipelines learn median replacements and missing-value flags within each training fold; boosting handles missing values directly.
6. **Select and investigate:** Selection is using the highest mean macro F1 to give each class equal weight. Input-group comparisons and October-November time-slot checks are helping explain differences in validation performance.
7. **Evaluate:** The selected model is being fitted on January-November and evaluated on December using accuracy, precision, recall, and F1. Its settings remain fixed during testing. The confusion matrix, example errors, and duplicate check provide further context.

## Results

The selected **HGB_small** model is using combined inputs, 100 boosting iterations, up to 7 leaves per tree, and a minimum of 50 samples per leaf.

| December metric | Result |
|---|---:|
| Accuracy | 95.19% |
| Macro precision | 0.9579 |
| Macro recall | 0.9383 |
| Macro F1 | 0.9474 |

The model is making **143 errors across 2,976 unique December intervals**. Its accuracy exceeds the majority baseline's **58.60%**. Medium_Load has the lowest recall at **89.77%**, mainly through confusion with Light_Load. Keeping the single duplicate changes accuracy only slightly, from **95.19% to 95.20%**.

Mean validation macro F1 is **0.8414**, but November's score drops to **0.5957** alongside changes in time-slot label patterns. This suggests sensitivity to changing schedules, so the December result should not be read as consistently strong performance across all months.

The data covers one year from one apparent industrial source; performance at other sites or in future years remains untested. The model requires current interval measurements and is not intended for forecasting before those readings are available.
