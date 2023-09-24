# Pymaceuticals Data Visualization Assignment
## Introduction
In this assignment, we'll work with Python, Pandas, and Matplotlib to analyze and visualize data related to a pharmaceutical company's study on cancer treatments. We will perform various data manipulation and visualization tasks to analyze and visualize the data to provide valuable insights.  

### Step 1: Prepare the Data
`import` the necessary Python libraries, then use `pd.merge("")` to combine data from `mouse_metadata.csv` and `study_results.csv` into a single DataFrame. Find the duplicate mice by finding the duplication Mouse ID and Timepoint from the merged data, then create a clean DataFrame by dropping duplicate mice by their ID. We used an inner join to keep only the rows with matching 'Mouse ID'.
```python
duplicate_mice = combined_data[combined_data.duplicated(subset=["Mouse ID", "Timepoint"])]
...
cleaned_data = combined_data.drop_duplicates(subset=['Mouse ID', 'Timepoint'])
cleaned_data = cleaned_data[cleaned_data["Mouse ID"]!="g989"]
```

### Step 2: Generate Summary Statistics
Generate a summary statistics table of mean, median, variance, standard deviation, and SEM of the tumor volume for each regimen using `groupby`. An alternative method, using the aggregation method, produces the same summary statistics in a single line
```python
summary_stats = cleaned_data.groupby('Drug Regimen')['Tumor Volume (mm3)'].agg(['mean', 'median', 'var', 'std', 'sem'])
```

### Step 3: Create Bar and Pie Charts
Generate a bar plot showing the total number of rows (Mouse ID/Timepoints) for each drug regimen using Panda `.plot(kind='bar')` and pyplot `plt.bar(VARIABLE.index, VARIABLE.values)`. Generate a pie plot showing the distribution of female versus male mice using Pandas `.plot(kind='pie', autopct='%1.1f%%')` and pyplot `plt.pie(VARIABLE, labels=VARIABLE.index, autopct='%1.1f%%')`
    
### Step 4: Calculate quartiles, find outliers, and create a box plot
Start by getting the last timepoint for each mouse, then merge this with `cleaned_data` to get the tumor volume at the last timepoint. Put treatments' names into a list for a for loop and create an empty list to fill with tumor vol data for plotting. Loop through each drug regimen to collect tumor volume data by locating the rows that contain mice from each drug. Then, get the tumor vol from the subset and calculate the outliers using upper and lower bounds by finding the quartiles.
```python
for regimen in regimens_list:
    regimen_data = final_tumor_data[final_tumor_data['Drug Regimen'] == regimen]
    
    tumor_volumes = regimen_data['Tumor Volume (mm3)']
    tumor_volume_data.append(tumor_volumes)
    
    quartiles = tumor_volumes.quantile([0.25, 0.75])
    lowerq = quartiles[0.25]
    upperq = quartiles[0.75]
    iqr = upperq - lowerq
    lower_bound = lowerq - 1.5 * iqr
    upper_bound = upperq + 1.5 * iqr
    outliers = regimen_data[(regimen_data['Tumor Volume (mm3)'] < lower_bound) | (regimen_data['Tumor Volume (mm3)'] > upper_bound)]
```
Using the information found above, generate a box plot that shows the distribution of the tumor volume for each treatment group.
```python
plt.boxplot(tumor_volume_data, labels=regimens_list, flierprops={'marker': 'o', 'markerfacecolor': 'red'})
```

### Step 5: Create a line and scatter plot for the drug Capomulin and Mouse L509
Generate a line plot of tumor volume vs. time point for a single mouse treated with Capomulin. From `clean_data`, select rows where the 'Drug Regimen' is 'Capomulin' and the 'Mouse ID' is 'l509'.
```python
capomulin_mouse = cleaned_data[(cleaned_data['Drug Regimen'] == 'Capomulin') & (cleaned_data['Mouse ID'] == 'l509')]
plt.plot(capomulin_mouse['Timepoint'], capomulin_mouse['Tumor Volume (mm3)'])
```
Generate a scatter plot of mouse weight vs. the average observed tumor volume for the entire Capomulin regimen. From `cleaned_data`, select and create a subset of the data containing only mice that received the 'Capomulin' treatment. Then, calculate the average tumor volume and weight for each mouse that received the 'Capomulin' treatment. 
```python
capomulin_data = cleaned_data[cleaned_data['Drug Regimen'] == 'Capomulin']
average_tumor_volume = capomulin_data.groupby('Mouse ID')['Tumor Volume (mm3)'].mean()
mouse_weight = capomulin_data.groupby('Mouse ID')['Weight (g)'].mean()
```

### Step 6: Calculate correlation and regression
Calculate the correlation coefficient and a linear regression model for mouse weight and average observed tumor volume for the entire Capomulin regimen
```python
capomulin_data = cleaned_data[cleaned_data['Drug Regimen'] == 'Capomulin']
average_tumor_volume = capomulin_data.groupby('Mouse ID')['Tumor Volume (mm3)'].mean()
mouse_weight = capomulin_data.groupby('Mouse ID')['Weight (g)'].mean()
correlation_coefficient = round(average_tumor_volume.corr(mouse_weight), 2)
slope, intercept, r_value, p_value, std_err = st.linregress(mouse_weight, average_tumor_volume)
predicted_tumor_volume = slope * mouse_weight + intercept
```
To calculate the correlation coefficient, first filter the data for the Capomulin regimen, then group each mouse and calculate the average 'Tumor Volume (mm3)' and average 'Weight (g)'.   

Then, use `.corr()` to calculate the correlation coefficient between the average tumor volume and average mouse weight. A positive value suggests as one goes up, the other tends to go up, while a negative value suggests as one goes up, the other tends to go down.  

To calculate linear regression between average mouse weight and average tumor volume, we need to calculate:
 - `slope`: the slope of the best-fit line
 -  `intercept`: the best-fit line intersecting the vertical axis
 -  `r_value`: indicating the strength and direction of the relationship
 -  `p_value`: measure the statistical significance of the relationship
 - `std_err`: standard error of the estimate  

### Step 7:  Final analysis
Among the four-drug regimens, Capomulin and Ramicane appear to be more effective in reducing tumor volume than Infubinol and Ceftamin since there is a lower median tumor volume for Capomulin and Ramicane. There is a positive correlation between mouse weight and the average tumor volume. This indicates that as mouse weight increases, the average tumor volume tends to increase as well. This correlation suggests that heavier mice develop larger tumors. Ceftamin, Capomulin, and Ramicane do not have any outliers, indicating that the data points for these drug regimens are consistent and do not deviate significantly.  

### Reference
[agg()](https://sparkbyexamples.com/pandas/pandas-groupby-sum-examples/)  
[matplotlib.pyplot](https://matplotlib.org/stable/api/pyplot_summary.html)  
[autopct](https://stackoverflow.com/questions/6170246/how-do-i-use-matplotlib-autopct)  
[scipy.stats.linregress](https://github.com/scipy/scipy/issues/2962)  
[Filter Rows by Conditions](https://sparkbyexamples.com/pandas/pandas-filter-rows-by-conditions/)
