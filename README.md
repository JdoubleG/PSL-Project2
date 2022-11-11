# **CS 598 Practical Statistical Learning**

### **Project 2: Walmart Store Sales Forecasting**

### James Garijo-Garde (jamesig2)

November 14, 2022

### **Introduction**

For this assignment, we were tasked with building a model to predict the weekly sales by department of 45 Walmart stores based on historical sales data. Each store contains many departments, and the stores were spread out across different regions. This assignment was inspired by [a Kaggle competition](https://www.kaggle.com/c/walmart-recruiting-store-sales-forecasting), however only the training data from the Kaggle competition was used. A challenge of this assignment (and modeling retail data in general) is that predictions regarding sales are generally based on fairly limited history.

The performance of the model is evaluated via weighted mean absolute error (WMAE).

### **Pre-Processing**

The data set contains 421,570 sample data points, each with 5 variables: store, department, date, weekly sales, and a categorical value indicating whether a given week is a holiday week. To pre-process the data, we begin by splitting the provided .csv file into train and test datasets. We separate the weekly sales data from the test dataset and divide the data into 10 time-series folds. The last week in 2010 is numbered as week 53, while the last week of 2011 is numbered 52. I subtracted 1 from the weeks in 2010 to mitigate this issue.

In later iterations of the model, as an additional step of pre-processing, I performed truncated singular value decomposition (SVD) on the data. The goal of this procedure is to reduce noise: the shared top principle components are stronger signals, while the principle components associated with smaller variance are more likely to be noise.

### **Model Approaches**

### Naive Model:

An initial "naive" approach can be taken by predicting future weekly sales with the data from the most recent week. If the most recent week does not have data, data from the next most recent week is used to make the prediction. If the next most recent week does not have data, a zero is assigned as the prediction. This produces a time series for department and store combinations. In this model (and in subsequent models mentioned here), the predictions of the model were appended to the training set to further inform our predictions of weekly sales in future folds. This is not an especially effective model, and WMAE is above 1900. I did not expect the model to perform especially well since there would be no clearly discernible trend from using the previous week; the model does not take into account factors like seasonal spikes in sales or recurring sales.

### Improvements to the Naive Model:

To improve the naive model, I used the same week from the previous year to form the prediction instead of merely the most recent week. I created a new categorical variable (feature) for week, numbered from 1 to 52. This change begins to allow for the potential factors mentioned above -- seasonal spikes and recurring sales -- to influence the model's prediction. This buys an extra point for a lower WMAE.

### Modeling with Year in Addition to Week:

Thus far, our two models have only taken week into account when predicting weekly sales, and have ignored the year. In the next iteration of our model, another feature for year is added. It is a numeric feature. A linear regression model predicting weekly sales off of the week and year features is used (`lm` is used). By adding this feature, I allowed the model to respond to sales trends occurring on a yearly level as opposed to a purely weekly level. We perform this prediction for each department in each store. Coefficients without data have values assigned as zero. I exclude stores that do not appear in the test data or the training data from our model. This brings the WMAE score down to one earning 3 points.

### A More Time-Efficient Model:

By building a set of unique pairs of stores and departments appearing in both the testing and training datasets, it's possible to refactor the above model to be more time-efficient. This yields the same WMAE in 10% of the run time. This refactored model also uses `lm.fit` in place of `lm` since `lm.fit` tends to be more efficient.

### Shifting Fold 5:

Fold 5 has the highest WMAE since it contains two holiday weeks and therefore receives higher weights in WMAE. The winner of the Kaggle competition suggested a post-prediction adjustment to the fifth fold to compensate. This iteration of my model records only 6/7 of the predicted sales for each week in fold 5, reducing overall WMAE to below the threshold for 4 points.

### Pre-Processing the Data to Reduce Noise:

To achieve a meaningful reduction in WMAE, it became necessary to perform additional pre-processing on the data. To do this, I arranged the data from each department in a m-by-n matrix (m denotes the number of stores with the particular department; n denotes the number of weeks) and applied singular value decomposition (SVD), choosing the top d components of the original data (d = 8 was used). If there were less than d rows of sales data to a department, the SVD process was skipped. This brings the WMAE down such that it exceeds the cutoff for full points by a little over 3.

### **Benchmarks**

| System Specifications |                                        |
|-----------------------|----------------------------------------|
| System                | Framework Laptop                       |
| CPU                   | Intel i7 1260P, 12 cores \@ 2.5 GHz    |
| GPU                   | AMD Radeon RX 6600, Thunderbolt 3 eGPU |
| Memory                | 16 GB DDR4 \@ 3200 MHz                 |
| Storage               | Seagate FireCuda 530 SSD, 1 TB         |
| OS                    | Microsoft Windows 11 Education         |
| R Version             | 4.2.1                                  |

| Performance              |          |
|--------------------------|----------|
| Final Model User Time    | 85.38 s  |
| Final Model System Time  | 2.98 s   |
| Final Model Elapsed Time | 90.12 s  |
| Final Model Average WMAE | 1583.395 |
| Fold 1 WMAE              | 1941.581 |
| Fold 2 WMAE              | 1363.462 |
| Fold 3 WMAE              | 1382.497 |
| Fold 4 WMAE              | 1527.280 |
| Fold 5 WMAE              | 2056.657 |
| Fold 6 WMAE              | 1635.783 |
| Fold 7 WMAE              | 1682.747 |
| Fold 8 WMAE              | 1399.604 |
| Fold 9 WMAE              | 1418.078 |
| Fold 10 WMAE             | 1426.258 |

### **Discussion**

Perhaps the biggest challenge to this project was figuring out how to start. The objective was quite broad, so the guidance provided by the professor and teaching assistants were useful in ensuring the assignment could be done well within a reasonable span of time. Dividing the task into iterations and improving my model's performance with each iteration was a good strategy, and ensured the code I have submitted could be provided before the deadline for the project.

While it is not surprising that the naive model was not enough to reach the desired performance with regards to WMAE, some of the steps taken to achieve the target performance are surprising. For instance, it was perhaps easy to anticipate that using the corresponding week from the previous year would lead to a better prediction than the most recent week, as common sense holds that sales will spike around the same weeks each year in anticipation of different holiday seasons, but the need for shifting the fifth fold by a factor of 1/7 would not have been obviously apparent to someone considering how to process the data.

Since the model described in this document is merely a few points shy of the target for the full 6 points, it's worth discussing further improvements that could've been made to make the model achieve a still lower WMAE. It was noted by the professor that adding an additional feature for year squared or converting the year feature created in the second iteration of our model to a categorical variable yielded significant improvements in performance. Ultimately, I did not manage to implement either feature before the project deadline. Another attempt I made at improving the performance of the model was through adjusting the shift on the 5th fold, although I abandoned this given how meager the reductions in WMAE were.

Overall, it's clear that predicting store sales by department is no easy task, and this project has given me a new sense of empathy towards the people responsible for deciding how much of different items to keep in stock.
