---
author:
- Sadia Mostofa
title: Analysis of Solar Wind Speed and Density Correlation Across Solar
  Cycles 23 and 24 Using OMNI Data
---

# Data Pipeline Code Description

## Introduction

The interaction between solar wind speed and density is a crucial factor
in understanding the dynamics of the interplanetary medium and its
influence on Earth's magnetosphere. The OMNI dataset, sourced from
near-Earth spacecraft, provides comprehensive solar wind measurements,
enabling detailed studies of these parameters. This report examines the
correlation between solar wind speed and density for solar cycles 23 and
24 using daily average data. By analyzing these two distinct periods,
the study aims to highlight variations in solar wind behavior and their
potential implications for space weather phenomena.

## Objectives

-   To develop a robust data collection and processing pipeline that
    retrieves, pre-processes, and analyzes OMNI solar wind data.

-   To develop a program capable of adapting to different selection
    periods for extended analysis.

-   To analyze the correlation between solar wind speed and density
    using daily average data from the OMNI dataset.

-   To compare the characteristics of solar wind parameters during solar
    cycle 23 and solar cycle 24.

-   To visualize the relationship between solar wind speed and density
    through scatter plots for each solar cycle.

## Data Collection and Pre-processing Pipeline

The provided function, *collect_data_from_online_and_preprocess* , is
designed to automate the retrieval and preprocessing of solar wind data
from the **OMNIWeb** database for specified date ranges. It requests
**daily-averaged solar wind speed** and **density data**, processes the
response into a structured format, and saves the data to a CSV file.
Finally, it loads the CSV data into a Pandas Data Frame for further
analysis, while providing key descriptive statistics.

### Function Description

#### Inputs

-   *start_date:* The start date for the data retrieval in the format
    **YYYYMMDD**.

-   *end_date:* The end date for the data retrieval in the format
    **YYYYMMDD**.

#### Steps

-   **Define Parameters for Data Request:**

    -   Constructs a dictionary of request parameters, including data
        resolution (day for daily averages), spacecraft ID (ACE), and
        variable codes (24 for solar wind speed and 30 for sigma-Np).

    -   Uses a POST request to submit these parameters to the
        **OMNIWeb** data retrieval endpoint.

-   **Send Data Request:**

    -   Initiates an HTTP POST request to **OMNIWeb** with the specified
        parameters.

    -   Checks the response status code to ensure a successful retrieval
        and prints the initial portion of the response for validation.

-   **Parse the HTML Response:**

    -   Utilizes **BeautifulSoup** to parse the HTML response and
        extract the data contained within *\<pre\>* tags.

    -   Processes the extracted text to identify headers and data rows.

-   **Reformat Data;** Cleans and organizes the retrieved data by:

    -   Constructing column headers based on extracted metadata (e.g.,
        parameter names).

    -   Parsing the data rows into a structured format.

    -   Writes the processed data to a CSV file named plasma_data.csv.

-   **Load Data to Memory:**

    -   Loads the CSV file into a Pandas DataFrame.

    -   Prints the first few rows and descriptive statistics for the
        dataset to confirm successful processing.

#### Outputs

**Returns:** A Pandas DataFrame containing the retrieved solar wind data
with the following key columns:

-   YEAR

-   DOY (Day of Year)

-   HR (Hour)

-   SW_Plasma_Speed (km/s) and

-   sigma-N (N/cm3̂)

### Function Highlights

The function highlights its capability to automate the retrieval and
preprocessing of solar wind data for any specified date range. By
eliminating manual steps, it ensures a reproducible and scalable
approach to data collection. The function processes the data into a
clean, structured format and saves it as a CSV file, ready for analysis.
Additionally, the function provides essential descriptive statistics,
allowing users to verify the integrity and quality of the retrieved data
immediately. These features make it a reliable tool for handling
extensive datasets from the **OMNIWeb** database.

Scalability and user-friendliness are key strengths of this function. It
can be easily adapted to retrieve different parameters or adjust the
temporal resolution by modifying input parameters. Furthermore, the
function ensures transparency by providing clear progress updates at
each step, from data retrieval to file writing and loading into memory.
This user-centric design and its flexibility make the function an
essential part of a data analysis pipeline for solar wind studies,
particularly for analyzing trends and correlations in **OMNIWeb**
datasets.

### Function Code: {#function-code .unnumbered}

``` {.python language="Python"}
import requests
from bs4 import BeautifulSoup
import csv
import pandas as pd

def collect_data_from_online_and_preprocess(start_date,end_date):

  # OMNIWeb endpoint for data retrieval
  url = "https://omniweb.gsfc.nasa.gov/cgi/nx1.cgi"

  start_data=str(start_date);
  end_date=str(end_date);
  # Define request parameters
  params = {
      "activity": "retrieve",        # Correct activity value for list data
      "res": "day",              # Daily-averaged data
      "start_date": start_data,  # Start date (YYYYMMDD)
      "end_date": end_date,    # End date (YYYYMMDD)
      "vars": ["24", "30"],      # Flow Speed/ plasma speed(code: 24) and Sigma-Np(code:30)
      "sc_id": "ACE",             # Spacecraft ID (e.g., ACE, WIND)
      "spacecraft": "omni2",
  }

  # Show the request details before sending it
  #print(f"URL: {url}")
  #print(f"Parameters: {params}")

  # Send POST request
  print("1. Data Requesting please wait")
  print("*************************************************")
  response = requests.post(url, data=params)

  # Check if the request was successful
  if response.status_code == 200:
      print(f"2. Request was successful!")
      print(f"Response Status Code: {response.status_code}")
      print("*************************************************")
      print(f"Response Content: \n{response.text[:350]}")  # Print the first 100 characters of the response to prevent overwhelming output
  else:
      print(f"Error: Unable to retrieve data. Status Code: {response.status_code}")
      print(f"Response Content: {response.text}")

  print("3. Data Processing please wait")
  print("*************************************************")
  html_content = response.text  # Use `.text` for string data
  # Parse the HTML content using BeautifulSoup
  soup = BeautifulSoup(html_content, "html.parser")

  # Extract the content inside <pre> tags
  pre_content = soup.find("pre").text

  # Print the extracted content
  #print(pre_content)

  # Process the data into lines
  lines = pre_content.strip().split("\n")

  # Extract the header and data rows
  data_c = ((lines[1].split(" "))[2:4])
  data_colmn1 ="_".join(data_c)
  data_colmn2 = (lines[2].split(" "))[2][:-1]
  header = lines[4].split()
  header.remove("1")
  header.remove("2")
  header.insert(3, data_colmn1)
  header.insert(4, data_colmn2)
  data_rows = [line.split() for line in lines[5:]]

  # Write to a CSV file
  output_file = "plasma_data.csv"
  with open(output_file, mode="w", newline="") as file:
      writer = csv.writer(file)
      writer.writerow(header)  # Write the header row
      writer.writerows(data_rows)  # Write the data rows
  # Load the CSV file
  print(f"4. Data successfully written to {output_file}")
  print("*************************************************")

  # data load to memory as pd
  data = pd.read_csv('plasma_data.csv')
  print("5. Data Loaded Sucessfully to Memory")
  print("*************************************************")
  print("***************First Some Row********************")
  print("*************************************************")
  print(data.head())
  print("*************************************************")
  print("*************Data Description fresh data*********")
  print("*************************************************")
  print(data.describe())
  return data

```

## Data Cleaning Function: `data_cleaning` 

The `data_cleaning` function is designed to pre-process and clean a
dataset by removing invalid or placeholder values. It ensures the
dataset is ready for analysis by eliminating rows with unrepresentative
values in key columns. The primary focus of the function is to improve
data integrity and prepare it for subsequent analytical tasks.

### Key Steps

1.  **Print Cleaning Process Updates**:

    -   Provides clear updates at the start and end of the cleaning
        process, ensuring transparency in the data handling steps.

2.  **Remove Invalid `’SW_Plasma’` Values**:

    -   Filters out rows where the `SW_Plasma` column has a value of
        `9999.0`, which is considered invalid or a placeholder for
        missing data.

3.  **Remove Invalid `’sigma-n’` Values**:

    -   Removes rows where the `sigma-n` column has a value of `999.9`,
        ensuring that only valid entries remain for analysis.

4.  **Display Cleaned Data Description**:

    -   Outputs descriptive statistics of the cleaned dataset, allowing
        users to validate that the cleaning process has been performed
        correctly.

### Purpose and Benefits

The function ensures that the dataset used for analysis is free from
anomalies and placeholder values, thereby improving the reliability and
accuracy of downstream analyses. By providing detailed feedback and
descriptions of the cleaned data, it also offers users confidence in the
quality of their dataset.

### Function Code: {#function-code-1 .unnumbered}

``` {.python language="Python"}
def data_cleaning(data):
  print("*************************************************")
  print("**********************Cleaning Data**************")
  print("*************************************************")

  # Remove rows where 'SW_Plasma' is 9999.0
  print("Remove rows where 'SW_Plasma' is 9999.0")
  data_cleaned = data[data['SW_Plasma'] != 9999.0]

  # Remove rows where 'sigma-n' is 999.9
  print("Remove rows where 'sigma-n' is 999.9")
  data_cleaned = data[data['sigma-n'] != 999.9]

  print("*************************************************")
  print("*************Data Description Cleaned Data*******")
  print("*************************************************")
  print(data_cleaned.describe())
  return data_cleaned
```

## Scatter Plotting Function : `scatter_plot`

The `scatter_plot` function is designed to visually represent the
relationship between solar wind speed (`SW_Plasma`) and density
(`sigma-n`) by creating a scatter plot. This visualization provides
insights into the correlation, patterns, or distribution of these
parameters, enabling users to better understand their relationship.

### Function Details

##### 1. Purpose:

To generate a scatter plot that visually depicts the relationship
between solar wind speed and density for a given dataset.

##### 2. Parameters:

-   **`data`**: A pandas DataFrame containing at least two columns:

    -   `SW_Plasma`: Represents the solar wind speed in km/s.

    -   `sigma-n`: Represents the plasma density in $N/\text{cm}^3$.

-   **`type_data`**: A string that specifies the type or subset of the
    data (e.g., "Fast Solar Wind", "Medium Solar Wind"). This is
    appended to the plot title for better context.

##### 3. Key Steps:

1.  **Figure Initialization:** The figure is created using Matplotlib
    with a defined size of $8 \times 6$, ensuring the plot is clearly
    visible.

2.  **Scatter Plot Creation:** A scatter plot is generated with
    `sigma-n` on the $x$-axis (Density) and `SW_Plasma` on the $y$-axis
    (Solar Wind Speed). Data points are displayed in blue with partial
    transparency (`alpha=0.5`) to enhance visibility when points
    overlap.

3.  **Title and Axes Labels:**

    -   The plot is titled *"Scatter Plot: Solar Wind Speed vs
        Density"*, followed by the specific `type_data` for context.

    -   The $x$-axis is labeled *"Density ($N/\text{cm}^3$)"*.

    -   The $y$-axis is labeled *"Solar Wind Speed ($\text{km/s}$)"*.

4.  **Grid Addition:** A grid is included in the background to make it
    easier to interpret the relative positions of the points.

5.  **Plot Display:** The `plt.show()` function displays the plot.

##### 4. Output:

A scatter plot that helps users visually assess the relationship between
solar wind speed and plasma density for the specified dataset and
category.

##### 5. Applications:

-   The function is useful for scientific research and analysis,
    enabling users to explore the correlation between solar wind
    properties.

-   It can help identify trends for specific solar wind conditions
    (e.g., fast, medium, or slow solar wind).

### Advantages:

-   **Customizability:** The title adapts dynamically to the provided
    `type_data`, ensuring the plot is descriptive and context-specific.

-   **Clarity:** The use of grids, labeled axes, and transparency makes
    the visualization intuitive and easy to interpret.

-   **Versatility:** Suitable for datasets of varying sizes and can be
    reused for different solar wind conditions or parameter subsets.

### Function Code: {#function-code-2 .unnumbered}

``` {.python language="Python"}
import matplotlib.pyplot as plt

def scatter_plot(data,type_data ):
  print("*************************************************")
  print("*************Scatter plotting *******************")
  print("*************************************************")
  # Create a scatter plot of Solar Wind Speed vs Density
  plt.figure(figsize=(8, 6))
  plt.scatter(y=data['SW_Plasma'],x=data['sigma-n'], color='blue', alpha=0.5)
  plt.title('Scatter Plot: Solar Wind Speed vs Density'+type_data)
  plt.ylabel('Solar Wind Speed (km/s)')
  plt.xlabel('Density (N/cm^3)')
  plt.grid(True)
  plt.show()
```

## Correlation Function Description: `compute_correlation`

The `compute_correlation` function calculates and displays the Pearson
correlation coefficient between solar wind speed (`SW_Plasma`) and
plasma density (`sigma-n`) from a given dataset. This metric is crucial
for quantifying the strength and direction of the linear relationship
between these two variables.

### Function Details

##### 1. Purpose:

To compute and display the Pearson correlation coefficient for assessing
the linear relationship between solar wind speed and plasma density.

##### 2. Parameters:

-   **`data`**: A pandas DataFrame containing at least two columns:

    -   `SW_Plasma`: Represents the solar wind speed in km/s.

    -   `sigma-n`: Represents the plasma density in $N/\text{cm}^3$.

##### 3. Key Steps:

1.  The function utilizes the `pandas.DataFrame.corr` method to compute
    the Pearson correlation coefficient between the `SW_Plasma` and
    `sigma-n` columns in the provided dataset.

2.  The correlation value is printed with a precision of two decimal
    places, providing an easily interpretable result for the user.

##### 4. Output:

-   Prints the Pearson correlation coefficient with the following
    format:

    > `Correlation between Solar Wind Speed and Density: <value>`

-   Returns no explicit value; the correlation coefficient is presented
    as a printed output.

### Function Code: {#function-code-3 .unnumbered}

``` {.python language="Python"}
# Compute the correlation coefficient between 'SW Plasma Speed' and 'sigma-n'
def compute_correlation(data):
  print("*************************************************")
  print("*************Describing Correlation**************")
  print("*************************************************")
  correlation = data['SW_Plasma'].corr(data['sigma-n'])
  print(f"Correlation between Solar Wind Speed and Density: {correlation:.2f}")
```

## Separate Data based on Solar wind Speed and Correlation Study Function Description:

The `separte_corelation_and_polt_based_on_solar_wind` function is
designed to categorize solar wind data into three distinct
groups---fast, medium, and slow solar winds---and analyze the
correlation between solar wind speed and plasma density for each group.
It also visualizes the relationship using scatter plots.

### Function Details

##### 1. Purpose:

To categorize solar wind data based on solar wind speed and compute
correlation coefficients alongside scatter plots for each category.

##### 2. Parameters:

-   **`data`**: A pandas DataFrame containing at least two columns:

    -   `SW_Plasma`: Represents the solar wind speed in km/s.

    -   `sigma-n`: Represents the plasma density in $N/\text{cm}^3$.

##### 3. Key Steps:

1.  **Categorization:**

    -   Fast solar wind: Rows where `SW_Plasma` $>$ 700.

    -   Slow solar wind: Rows where `SW_Plasma` $<$ 445.

    -   Medium solar wind: Rows where 445 $\leq$ `SW_Plasma` $\leq$ 700.

2.  **Scatter Plots and Correlations:**

    -   For each category, call `scatter_plot()` to visualize the data.

    -   Call `compute_correlation()` to calculate and display the
        Pearson correlation coefficient.

##### 4. Outputs:

-   **Scatter Plots:** Visualizes the relationship between solar wind
    speed and plasma density for each category.

-   **Correlation Coefficients:** Computes and prints the strength and
    direction of the linear relationship for each category.

### Applications

-   Provides insights into how solar wind speed affects plasma density
    in different regimes.

-   Supports targeted studies of solar wind dynamics for specific
    categories (e.g., fast wind events during solar storms).

### Advantages

-   **Segmentation:** Allows detailed analysis by separating the data
    into meaningful categories.

-   **Visualization:** Includes scatter plots for visual interpretation
    of trends.

-   **Automation:** Combines categorization, visualization, and
    correlation analysis in a single automated pipeline.

### Function Code: {#function-code-4 .unnumbered}

``` {.python language="Python"}
def separte_corelation_and_polt_based_on_solar_wind(data):
  # Categorize into fast, medium, and slow
  fast_solar_wind = data[data['SW_Plasma'] > 700]
  slow_solar_wind = data[data['SW_Plasma'] < 445]
  medium_solar_wind = data[(data['SW_Plasma'] >= 445) & (data['SW_Plasma'] <= 700)]
  
  print("*************************************************")
  print("**Describing Corelation for Fast Solar Wind**")
  print("*************************************************")
  scatter_plot(fast_solar_wind,"(Fast Solar Wind)")
  compute_correlation(fast_solar_wind)

  print("*************************************************")
  print("**Describing Corelation for Medium Solar Wind**")
  print("*************************************************")
  scatter_plot(medium_solar_wind,"(Medium Solar Wind)")
  compute_correlation(medium_solar_wind)

  print("*************************************************")
  print("**Describing Corelation for Slow Solar Wind**")
  print("*************************************************")
  scatter_plot(slow_solar_wind,"(Slow Solar Wind)")
  compute_correlation(slow_solar_wind)
```

# Data Analysis 

## Solar Cycle 23 (January 1996-December 2008)

### Descriptive Statistics Summary

The dataset comprises 109,609 data points spanning from 1996 to 2008,
covering parameters like Year, Day of Year (DOY), Hour, Solar Wind Speed
(SW_Plasma), and Density. The Solar Wind Speed has a mean of 447.07 km/s
with a standard deviation of 105.11 km/s, while Density has a mean of
0.69 N/cm³ and a standard deviation of 1.07 N/cm³. Percentiles for Solar
Wind Speed and Density are provided, showing data distribution across
different ranges.

::: tabular
\|l\|lllll\| & & & & &\
**Total Data Point** &\
**Mean** & & & & & 0.69\
**Standard Deviation** & & & & & 1.07\
**Minimum** & & & & & 0\
**25th Percentile** & & & & & 0.2\
**50th Percentile (Median)** & & & & & 0.4\
**75th Percentile** & & & & & 0.7\
**Maximum** & & & & & 45\
:::

### Correlation Analysis

The overall correlation between Solar Wind Speed (SW_Plasma) and Density
(sigma-n) is -0.14, which indicates a very weak negative correlation.
This suggests that as solar wind speed increases, the density slightly
decreases, but the relationship is not strong enough to draw significant
conclusions. Let's examine this correlation further for the three
different categories of solar wind speed: Fast, Medium, and Slow.

![Scatter plot of Solar cycle 23; Solar wind speed vs Density; X axis
shows Density in $N/cm^3$ and Y axis shows Solar Wind Speed in $Km/s$
](sw23_overal_plt.png){width="0.75\\linewidth"}

#### Fast Solar Wind (SW_Plasma $>$ 700 km/s) {#fast-solar-wind-sw_plasma-700-kms .unnumbered}

The correlation between Solar Wind Speed and Density for fast solar wind
is 0.23, which represents a weak positive correlation. This suggests
that as the solar wind speed increases in the fast solar wind category,
the density tends to slightly increase as well. Although the
relationship is weak, it is a positive one, meaning there is a slight
tendency for higher density values with higher solar wind speeds.

#### Medium Solar Wind (445 km/s $<=$ SW_Plasma $<=$ 700 km/s) {#medium-solar-wind-445-kms-sw_plasma-700-kms .unnumbered}

For the medium solar wind category, the correlation is -0.10, which is a
very weak negative correlation. This indicates that there is a slight
decrease in density as the solar wind speed increases, but again, the
relationship is weak and not statistically significant. This result
shows that within this range of solar wind speeds, there is no clear
pattern linking speed and density.

   **Solar Wind Category**   **Correlation between Solar Wind Speed and Density**
  ------------------------- ------------------------------------------------------
       Fast Solar Wind                               0.23
      Medium Solar Wind                             -0.10
       Slow Solar Wind                              -0.04

  : Correlation of Solar Wind Speed and Density for Different Solar Wind
  Speeds For Solar Cycle 23

![Scatter Plot for Different types of Solar wind Speed Vs Density for
solar cycle 23](fast23.png){#fig:solar_wind_comparison
width="\\linewidth"}

![Scatter Plot for Different types of Solar wind Speed Vs Density for
solar cycle 23](medium23.png){#fig:solar_wind_comparison
width="\\linewidth"}

![Scatter Plot for Different types of Solar wind Speed Vs Density for
solar cycle 23](slow23.png){#fig:solar_wind_comparison
width="\\linewidth"}

#### Slow Solar Wind (SW_Plasma $<$ 445 km/s) {#slow-solar-wind-sw_plasma-445-kms .unnumbered}

The correlation for slow solar wind is -0.04, indicating an extremely
weak negative correlation. This suggests that there is almost no
relationship between solar wind speed and density for slow solar winds.
The minimal negative correlation implies that as the solar wind speed
decreases, the density remains relatively unchanged or slightly
increases, but the effect is negligible.

## Solar Cycle 24 (December 2008-December 2019)

### Descriptive Statistics Summary

Table 2.3 presents a statistical summary of the data collected during
Solar Cycle 24, covering key parameters such as the year, day of year
(DOY), hour (HR), solar wind speed (SW Plasma), and density (sigma-n).
The data spans from 2008 to 2019, with a total of 95,673 data points.
The solar wind speed (SW Plasma) has a mean value of 412.7 km/s and
shows significant variability, with a standard deviation of 91.86 km/s.
The minimum recorded solar wind speed is 233 km/s, and the maximum is
878 km/s. The density of the solar wind (sigma-n) has a mean of 0.66
N/cm³, with a standard deviation of 0.97. The density values range from
0 to 62.5 N/cm³. The 25th, 50th, and 75th percentiles indicate that the
solar wind speed typically falls around 392 km/s, and the density is
centered around 0.4 N/cm³, with a relatively low average.

::: tabular
\|l\|lllll\| & & & & &\
**Total Data Point** &\
**Mean** & & & & & 0.66\
**Std Dev** & & & & & 0.97\
**Min** & & & & & 0\
**25%** & & & & & 0.2\
**50%** & & & & & 0.4\
**75%** & & & & & 0.7\
**Max** & & & & & 62.5\
:::

\

### Correlation Analysis

The data analysis for Solar Cycle 24 explores the correlation between
Solar Wind Speed (SW_Plasma) and Density (sigma-n) during different
solar wind conditions. The overall correlation between solar wind speed
and density is found to be -0.14, indicating a weak negative
relationship, suggesting that as solar wind speed increases, density
tends to decrease, although this relationship is not strong.

![Scatter plot of Solar cycle 24; Solar wind speed vs Density; X axis
shows Density in $N/cm^3$ and Y axis shows Solar Wind Speed in $Km/s$
](sw24_overal_plt.png){width="0.75\\linewidth"}

For the Fast Solar Wind, the correlation is slightly positive at 0.02,
suggesting almost no relationship between speed and density during these
periods. This suggests that the characteristics of fast solar wind in
this cycle do not show a notable trend when it comes to density
variations.

For the Medium Solar Wind, the correlation is -0.10, indicating a weak
negative relationship between the speed of the medium solar wind and its
density. This suggests that, although a slight inverse relationship
exists, it is not strong enough to draw definitive conclusions.

For the Slow Solar Wind, the correlation is -0.05, indicating a very
weak negative correlation, with little to no significant relationship
between the speed and density of slow solar wind during this cycle.

In summary, the data reveals weak to negligible correlations between
solar wind speed and density across all solar wind categories (fast,
medium, and slow) during Solar Cycle 24, with the strongest correlation
being negative but very weak for the overall cycle.

   **Solar Wind Category**   **Correlation between Solar Wind Speed and Density**
  ------------------------- ------------------------------------------------------
       Fast Solar Wind                               0.02
      Medium Solar Wind                             -0.10
       Slow Solar Wind                              -0.05

  : Correlation of Solar Wind Speed and Density for Different Solar Wind
  Speeds For Solar Cycle 24

![Scatter Plot for Different types of Solar wind Speed Vs Density for
solar cycle 24](fast24.png){#fig:solar_wind_comparison
width="\\linewidth"}

![Scatter Plot for Different types of Solar wind Speed Vs Density for
solar cycle 24](medium24.png){#fig:solar_wind_comparison
width="\\linewidth"}

![Scatter Plot for Different types of Solar wind Speed Vs Density for
solar cycle 24](slow24.png){#fig:solar_wind_comparison
width="\\linewidth"}
