# Artificial Intelligence for Security, 2024/25: Group Assignment (previous year, reference only)

> Transcribed from the previous year's assignment PDF (dated 18/11/2023 in the document). Kept only for comparison: the current assignment is [assignment-2025-26.md](assignment-2025-26.md).

In this assignment you will analyse recent and large cyber-security datasets. Details are as follows:

- You must complete the assignment in groups of 3 (minimum) or 4 (maximum) students.
- The assignment involves writing a Jupyter notebook to analyse the data as described below. Your notebook should be self-explanatory, with clear descriptions of the analysis performed and the conclusions drawn.
- The assignment is due on Thursday the 19th of December at 23:50 (via WeBeep).
- On Friday the 20th of December, each group will have around 10 minutes to present their notebook during the practical session. (No need for any presentation slides, we just want to see your notebook.)

## Marking

1. the appropriate and correct use of the methods applied,
2. the depth and insightfulness of the analysis,
3. the clarity of the description in the notebook, and
4. the quality of the presentation.

## Datasets

Each group should choose ONE of the following datasets. A maximum of 4 groups can work on the same dataset ([registration file](https://docs.google.com/document/d/17Bs11S_6QJKZwpH8nPfX9Zaquc2k8Kkj5PfkFLGMQJ0/edit?usp=sharing), first come first served). Some datasets can be very large: you may need to sample a random subset of the data to perform certain types of analysis.

1. **PhiUSIIL Phishing URL (2024)**: <https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+dataset>. 134,850 legitimate and 100,945 phishing URLs. DO NOT open the links in a browser. The dataset contains features extracted from the URLs themselves; you could programmatically download the content of the webpages (without ever opening them) and extract features from the text of the page.
2. **CIC Internet of Medical Things (IoMT) dataset 2024**: <https://www.unb.ca/cic/datasets/iomt-dataset-2024.html>. Network traffic from 40 medical devices communicating through different protocols, exposed to 18 different attacks (organised in a hierarchy available on the web page), more than 1M samples.
3. **CIC Internet of Vehicles (IoV) dataset 2024**: <https://www.unb.ca/cic/datasets/iov-dataset-2024.html>. Communication between components of a Ford car, either in normal conditions or while applying one of five different attacks.
4. **SMS Spam Collection Dataset (2022)**: <https://archive.ics.uci.edu/dataset/228/sms+spam+collection>. Over 5 thousand SMS messages, around 700 of which are spam. Small (200KB), so you might look for additional spam datasets to train a better classifier. Paper: <https://dl.acm.org/doi/10.1145/2034691.2034742>

## Tasks (suggestions)

The tasks listed below are only suggestions. You are strongly encouraged to investigate further and present all your insights in your notebook.

### A) Compute summary statistics

- How many rows and how many columns are there in the data?
- What are the names and datatypes in each column?
- Do any columns contain categorical values? If so, what values can they take? Which value is most frequent?
- For numeric columns calculate summary statistics (min/max, mean/median, standard deviation, etc.). What do these values tell you?
- Is there a target variable that needs to be predicted? How frequent/balanced are its values?

### B) Visualise the data

- Create bar plots and histograms for various features. Are there any obvious outliers?
- If there is a categorical target variable, plot summary statistics for different values of the target variable. Are there differences across the groups? What does that mean?
- Plot a heatmap of the correlations. Which features are correlated with one another? Do the correlations make sense?

### C) Perform supervised learning

- Do any of the features contain missing values? If so, how should you handle them?
- Which features are important for the classification?
- Which classifiers give the best performance?
- What evaluation methods are most appropriate for this problem?

### D) Perform unsupervised learning

- Plot the clusters in 2 dimensions, by selecting pairs of attributes or by performing dimensionality reduction.
- Do the clusters look reasonable? Do they correspond to intuitive subsets of the data? Could they be useful for defining features for a classifier?
- Try running the clustering separately for different values of the target variable.
- Run multivariate anomaly detection techniques. Do the results tell you anything interesting? Can you interpret the outliers or are they noise?

### E) Optional: investigate other datasets

If you finish the analysis on the chosen dataset, look for other related datasets online.
