

```python
import pandas as pd
from functools import reduce
```


```python
# Read in school
school = pd.read_csv("raw_data/schools_complete.csv")
school = school.rename(columns={"name": "school"})
school.set_index("school", inplace = True)
```


```python
# Read in students
stud = pd.read_csv("raw_data/students_complete.csv")
```


```python
# DISTRICT SUMMARY
# Assuming passing score is 70% (not specified in HW instructions)

totSch = len(school.index)
totStu = len(stud.index)
totBudg = school["budget"].sum()
avgMath = round(stud["math_score"].mean(),2)
avgRead = round(stud["reading_score"].mean(),2)
passMath = round((stud["math_score"][stud["math_score"] >= 70].count()) / totStu * 100,2)
passRead = round((stud["reading_score"][stud["reading_score"] >= 70].count()) / totStu * 100,2)
passAll = round((float(passMath) + passRead) / 2, 2)

district = pd.DataFrame({
          "Total Schools": [totSch],
          "Total Students": ["{:,}".format(totStu)],
          "Total Budget": ["$" + "{:,}".format(totBudg)],
          "Average Math Score": [avgMath],
          "Average Reading Score": [avgRead],
          "% Passing Math": [passMath],
          "% Passing Reading": [passRead],
          "Overall Passing Rate": [passAll],
          })

district[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]] = district[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]].applymap("{:,.2f}".format) + "%"

# Set order of columns

district = district[["Total Schools","Total Students","Total Budget","Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]
```


```python
# ANSWER - DISTRICT SUMMARY

district
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>74.98%</td>
      <td>85.81%</td>
      <td>80.40%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCHOOL SUMMARY - calc scores

studSchool = stud.groupby("school")
avgMath_School = round(studSchool["math_score"].mean(),2).to_frame()
avgRead_School = round(studSchool["reading_score"].mean(),2).to_frame()
students_School = studSchool["school"].count().to_frame()              # Number of students in school
passMathn = studSchool["math_score"].agg({lambda x: (x >= 70).sum()})  # Number of students who passed math
passReadn = studSchool["reading_score"].agg({lambda x: (x >= 70).sum()})   # Number of students who passed reading

```


```python
# SCHOOL SUMMARY - create df of scores

from functools import reduce

mergeVars = [avgMath_School, avgRead_School, passMathn, passReadn, students_School]
scores_School = reduce(lambda left, right: pd.merge(left, right, left_index = True, right_index = True), mergeVars)
scores_School.columns = ["Average Math Score", "Average Reading Score", "n Passed Math", "n Passed Reading", "# Students"]
scores_School["% Passing Math"] = round(scores_School["n Passed Math"] / scores_School["# Students"] * 100,2)
scores_School["% Passing Reading"] = round(scores_School["n Passed Reading"] / scores_School["# Students"] * 100,2)
scores_School["Overall Passing Rate"] = round((scores_School["% Passing Math"]+scores_School["% Passing Reading"])/2, 2)
scores_School = scores_School[["Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
```


```python
# SCHOOL SUMMARY - merge dfs

schoolSummary = school.merge(scores_School, left_index= True, right_index= True)
schoolSummary.sort_index(inplace = True)            # Sort schools alphabetically

# Format columns
schoolSummary["Total Students"] = schoolSummary["size"].map("{:,}".format)    
schoolSummary["Total Budget"] = "$" + schoolSummary["budget"].map("{:,}".format)
schoolSummary["Per Student Budget"] = "$" + (schoolSummary["budget"]/schoolSummary["size"]).map("{:,.2f}".format)
schoolSummary = schoolSummary.drop(["School ID", "size", "budget"], axis = 1)
schoolSummary = schoolSummary.rename(columns = {"type": "School Type"})

# Set order of columns
schoolSummary = schoolSummary[['School Type',
 'Total Students',
 'Total Budget',
 'Per Student Budget',
 'Average Math Score',
 'Average Reading Score',
 '% Passing Math',
 '% Passing Reading',
 'Overall Passing Rate']]
```


```python
# ANSWER - SCHOOL SUMMARY

schoolSummary_answer = schoolSummary.reset_index()
schoolSummary_answer[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]] = schoolSummary_answer[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]].applymap("{:,.2f}".format) + "%"
schoolSummary_answer
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>District</td>
      <td>4,976</td>
      <td>$3,124,928</td>
      <td>$628.00</td>
      <td>77.05</td>
      <td>81.03</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.58%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.26%</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020</td>
      <td>$652.00</td>
      <td>77.29</td>
      <td>80.93</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087</td>
      <td>$581.00</td>
      <td>83.80</td>
      <td>83.81</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.30%</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1,761</td>
      <td>$1,056,600</td>
      <td>$600.00</td>
      <td>83.36</td>
      <td>83.73</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.21%</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>Charter</td>
      <td>1,800</td>
      <td>$1,049,400</td>
      <td>$583.00</td>
      <td>83.68</td>
      <td>83.96</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# ANSWER - Top Performing Schools (By Passing Rate)

topSchools = schoolSummary.sort_values("Overall Passing Rate", ascending = False).reset_index()
topSchools = topSchools.loc[0:4,:]
topSchools[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]] = topSchools[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]].applymap("{:,.2f}".format) + "%"
topSchools
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>$582.00</td>
      <td>83.06</td>
      <td>83.98</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.58%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thomas High School</td>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130</td>
      <td>$638.00</td>
      <td>83.42</td>
      <td>83.85</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609.00</td>
      <td>83.84</td>
      <td>84.04</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>$625.00</td>
      <td>83.35</td>
      <td>83.82</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.26%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574</td>
      <td>$578.00</td>
      <td>83.27</td>
      <td>83.99</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.21%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# ANSWER - Bottom Performing Schools (By Passing Rate)

lowSchools = schoolSummary.sort_values("Overall Passing Rate", ascending = True).reset_index()
lowSchools = lowSchools.loc[0:4,:]
lowSchools[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]] = lowSchools[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]].applymap("{:,.2f}".format) + "%"
lowSchools
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363</td>
      <td>$637.00</td>
      <td>76.84</td>
      <td>80.74</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.30%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>$639.00</td>
      <td>76.71</td>
      <td>81.16</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Huang High School</td>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635</td>
      <td>$655.00</td>
      <td>76.63</td>
      <td>81.18</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Johnson High School</td>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650</td>
      <td>$650.00</td>
      <td>77.07</td>
      <td>80.97</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ford High School</td>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916</td>
      <td>$644.00</td>
      <td>77.10</td>
      <td>80.75</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.81%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Math Scores by Grades

#9th grade

math9 = round(stud[["school","math_score"]][stud["grade"] == "9th"]  \
       .groupby(["school"])["math_score"].mean(),2).to_frame()
math9 = math9.rename(columns = {"math_score": "9th"})

#10th grade

math10 = round(stud[["school","math_score"]][stud["grade"] == "10th"]  \
       .groupby(["school"])["math_score"].mean(),2).to_frame()
math10 = math10.rename(columns = {"math_score": "10th"})

#11th grade

math11 = round(stud[["school","math_score"]][stud["grade"] == "11th"]  \
       .groupby(["school"])["math_score"].mean(),2).to_frame()
math11 = math11.rename(columns = {"math_score": "11th"})

#12th grade

math12 = round(stud[["school","math_score"]][stud["grade"] == "12th"]  \
       .groupby(["school"])["math_score"].mean(),2).to_frame()
math12 = math12.rename(columns = {"math_score": "12th"})


# Merge grade dfs together

# ANSWER - Math Scores by Grade

mergemath = [math9, math10, math11, math12]
math_Grades = reduce(lambda left, right: pd.merge(left, right, left_index = True, right_index = True), mergemath)
math_Grades.reset_index()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>77.08</td>
      <td>77.00</td>
      <td>77.52</td>
      <td>76.49</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>83.09</td>
      <td>83.15</td>
      <td>82.77</td>
      <td>83.28</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>76.40</td>
      <td>76.54</td>
      <td>76.88</td>
      <td>77.15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>77.36</td>
      <td>77.67</td>
      <td>76.92</td>
      <td>76.18</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>82.04</td>
      <td>84.23</td>
      <td>83.84</td>
      <td>83.36</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>77.44</td>
      <td>77.34</td>
      <td>77.14</td>
      <td>77.19</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>83.79</td>
      <td>83.43</td>
      <td>85.00</td>
      <td>82.86</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>77.03</td>
      <td>75.91</td>
      <td>76.45</td>
      <td>77.23</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>77.19</td>
      <td>76.69</td>
      <td>77.49</td>
      <td>76.86</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>83.63</td>
      <td>83.37</td>
      <td>84.33</td>
      <td>84.12</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>76.86</td>
      <td>76.61</td>
      <td>76.40</td>
      <td>77.69</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>83.42</td>
      <td>82.92</td>
      <td>83.38</td>
      <td>83.78</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>83.59</td>
      <td>83.09</td>
      <td>83.50</td>
      <td>83.50</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>83.09</td>
      <td>83.72</td>
      <td>83.20</td>
      <td>83.04</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>83.26</td>
      <td>84.01</td>
      <td>83.84</td>
      <td>83.64</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Reading Scores by Grades

#9th grade

read9 = round(stud[["school","reading_score"]][stud["grade"] == "9th"]  \
       .groupby(["school"])["reading_score"].mean(),2).to_frame()
read9 = read9.rename(columns = {"reading_score": "9th"})

#10th grade

read10 = round(stud[["school","reading_score"]][stud["grade"] == "10th"]  \
       .groupby(["school"])["reading_score"].mean(),2).to_frame()
read10 = read10.rename(columns = {"reading_score": "10th"})

#11th grade

read11 = round(stud[["school","reading_score"]][stud["grade"] == "11th"]  \
       .groupby(["school"])["reading_score"].mean(),2).to_frame()
read11 = read11.rename(columns = {"reading_score": "11th"})

#12th grade

read12 = round(stud[["school","reading_score"]][stud["grade"] == "12th"]  \
       .groupby(["school"])["reading_score"].mean(),2).to_frame()
read12 = read12.rename(columns = {"reading_score": "12th"})


# Merge grade dfs together 

# ANSWER - Reading Scores by Grade

mergeReading = [read9, read10, read11, read12]
reading_Grades = reduce(lambda left, right: pd.merge(left, right, left_index = True, right_index = True), mergeReading)
reading_Grades.reset_index()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>81.30</td>
      <td>80.91</td>
      <td>80.95</td>
      <td>80.91</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>83.68</td>
      <td>84.25</td>
      <td>83.79</td>
      <td>84.29</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>81.20</td>
      <td>81.41</td>
      <td>80.64</td>
      <td>81.38</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>80.63</td>
      <td>81.26</td>
      <td>80.40</td>
      <td>80.66</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>83.37</td>
      <td>83.71</td>
      <td>84.29</td>
      <td>84.01</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>80.87</td>
      <td>80.66</td>
      <td>81.40</td>
      <td>80.86</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>83.68</td>
      <td>83.32</td>
      <td>83.82</td>
      <td>84.70</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>81.29</td>
      <td>81.51</td>
      <td>81.42</td>
      <td>80.31</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>81.26</td>
      <td>80.77</td>
      <td>80.62</td>
      <td>81.23</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>83.81</td>
      <td>83.61</td>
      <td>84.34</td>
      <td>84.59</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>80.99</td>
      <td>80.63</td>
      <td>80.86</td>
      <td>80.38</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>84.12</td>
      <td>83.44</td>
      <td>84.37</td>
      <td>82.78</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>83.73</td>
      <td>84.25</td>
      <td>83.59</td>
      <td>83.83</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>83.94</td>
      <td>84.02</td>
      <td>83.76</td>
      <td>84.32</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>83.83</td>
      <td>83.81</td>
      <td>84.16</td>
      <td>84.07</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCORES BY SCHOOL SPENDING

# Create df

schoolBudg = schoolSummary[['Per Student Budget',
                         'Average Math Score',
                         'Average Reading Score',
                         '% Passing Math',
                         '% Passing Reading',
                         'Overall Passing Rate']]       \
            .reset_index()

schoolBudg['Per Student Budget'] = pd.to_numeric(schoolBudg['Per Student Budget'].str.replace("$", ""))
schoolBudg = schoolBudg.drop("school", 1)

                                          
# Find school spending quartiles 

import numpy as np

schoolBudg["Per Student Budget"].quantile(q = [.25,.5,.75])

conditions = [
    (schoolBudg["Per Student Budget"] < 591.5),
    (schoolBudg["Per Student Budget"] >= 591.5) & (schoolBudg["Per Student Budget"] < 628.0),
    (schoolBudg["Per Student Budget"] >= 628.0) & (schoolBudg["Per Student Budget"] < 641.5),
    (schoolBudg["Per Student Budget"] >= 641.5)]

quartiles = ["Q1: < $591.5",
             "Q2: $591.5-628.0",
             "Q3: $628.0-641.5",
             "Q4: >= 641.5"]

schoolBudg["Spending Quartile"] = np.select(conditions, quartiles)

scores_bySpending = schoolBudg.groupby(["Spending Quartile"], as_index = False).mean().round(2)

# Format cols

scores_bySpending["Per Student Budget"] = "$" + scores_bySpending["Per Student Budget"].map("{:,.2f}".format)
scores_bySpending[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]] = scores_bySpending[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]].applymap("{:,.2f}".format) + "%"
```


```python
## ANSWER - Scores by School Spending

scores_bySpending
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Spending Quartile</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Q1: &lt; $591.5</td>
      <td>$581.00</td>
      <td>83.45</td>
      <td>83.94</td>
      <td>93.46%</td>
      <td>96.61%</td>
      <td>95.04%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Q2: $591.5-628.0</td>
      <td>$611.33</td>
      <td>83.52</td>
      <td>83.86</td>
      <td>93.95%</td>
      <td>96.31%</td>
      <td>95.13%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Q3: $628.0-641.5</td>
      <td>$635.50</td>
      <td>78.50</td>
      <td>81.69</td>
      <td>73.08%</td>
      <td>85.05%</td>
      <td>79.07%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Q4: &gt;= 641.5</td>
      <td>$650.25</td>
      <td>77.02</td>
      <td>80.96</td>
      <td>66.70%</td>
      <td>80.68%</td>
      <td>73.69%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCORES BY SCHOOL SIZE

# Create df

schoolSize = schoolSummary[['Total Students',
                         'Average Math Score',
                         'Average Reading Score',
                         '% Passing Math',
                         '% Passing Reading',
                         'Overall Passing Rate']]       \
            .reset_index()

schoolSize = schoolSize.drop("school", 1)

schoolSize["Total Students"] = pd.to_numeric(schoolSize["Total Students"].str.replace(",", ""))

                                          
# Find school sizes 

import numpy as np

schoolSize["Total Students"].quantile(q = [.25,.5,.75])

conditions = [
    (schoolSize["Total Students"] < 1698),
    (schoolSize["Total Students"] >= 1698) & (schoolSize["Total Students"] < 3474),
    (schoolSize["Total Students"] >= 3474)]
sizes = ["Small: < 1,698", "Medium: 1,698-3,474", "Large: > 3,474"]

schoolSize["Size"] = np.select(conditions, sizes)


scores_bySize = schoolSize.groupby(["Size"], as_index = False).mean() \
                .round(2)                         \
                .sort_values(["Total Students"])    \
                .drop(["Total Students"], 1)

# # Format cols

scores_bySize[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]] = scores_bySize[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]].applymap("{:,.2f}".format) + "%"
```


```python
# ANSWER - Scores by School Size

scores_bySize
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Size</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Small: &lt; 1,698</td>
      <td>83.60</td>
      <td>83.88</td>
      <td>93.44%</td>
      <td>96.66%</td>
      <td>95.05%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Medium: 1,698-3,474</td>
      <td>80.54</td>
      <td>82.68</td>
      <td>82.17%</td>
      <td>89.63%</td>
      <td>85.90%</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Large: &gt; 3,474</td>
      <td>77.06</td>
      <td>80.92</td>
      <td>66.46%</td>
      <td>81.06%</td>
      <td>73.76%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCORES BY SCHOOL TYPE

# Create df

schoolType = schoolSummary[['School Type',
                         'Average Math Score',
                         'Average Reading Score',
                         '% Passing Math',
                         '% Passing Reading',
                         'Overall Passing Rate']]       \
            .reset_index()

schoolType = schoolType.drop("school", 1)

scores_byType = schoolType.groupby(["School Type"], as_index = False).mean().round(2)
                                          
# Format cols

scores_byType[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]] = scores_byType[["% Passing Math", "% Passing Reading", "Overall Passing Rate"]].applymap("{:,.2f}".format) + "%"
```


```python
# ANSWER - Scores by School Type

scores_byType
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Charter</td>
      <td>83.47</td>
      <td>83.90</td>
      <td>93.62%</td>
      <td>96.59%</td>
      <td>95.10%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>District</td>
      <td>76.96</td>
      <td>80.97</td>
      <td>66.55%</td>
      <td>80.80%</td>
      <td>73.68%</td>
    </tr>
  </tbody>
</table>
</div>


