
Import Dependencies


```python
import pandas as pd
import numpy as np
import os

```

<li> Setting arbitrary passing_score = 70 for analysis going forward </li>
<li> Create data frames reading Schools data file </li>
 <li>   Rename the name column from 'name' to 'school_name' </li>
    


```python
passing_score = 70
sch_csv_path = os.path.join("raw_data","schools_complete.csv")
schools_df = pd.read_csv(sch_csv_path)
# schools_df = schools_df.loc[schools_df["type"]=="District"]
schools_df = schools_df.reset_index(drop=True)
schools_df = schools_df.rename(columns={"name":"school_name"})
# schools_df

```

Compute number of schools into schools_count


```python
schools_count = schools_df["school_name"].count()
# schools_count
```

Import students data file into students_df data frame
Rename school column as school_name


```python
stu_csv_path = os.path.join("raw_data","students_complete.csv")
students_df = pd.read_csv(stu_csv_path)
students_df = students_df.rename(columns={"school":"school_name"})
# students_df.head()
# students_df.describe()
```

Merge school data frame with type='District' and students data


```python
district_df = pd.merge(schools_df[schools_df.type == "District"],students_df,on="school_name")
# district_df.shape
```

Compute the number of schools into 'district_count'


```python
district_count = len(district_df["school_name"].unique())
district_count
```




    7




```python
budget_total = district_df["budget"].sum()
# budget_total
```


```python
avg_math_score = district_df["math_score"].mean()
# avg_math_score
```


```python
avg_reading_score = district_df["reading_score"].mean()
# avg_reading_score
```


```python
students_count = district_df["Student ID"].count()
# students_count
```


```python
passing_math = district_df.loc[district_df["math_score"] >= passing_score].count() * 100/students_count
passing_math = passing_math["name"]
```


```python
passing_reading = district_df.loc[district_df["reading_score"] >= passing_score].count() * 100/students_count
passing_reading = passing_reading["name"]
```


```python
passing_list = [passing_math,passing_reading]
passing_rate = np.mean(passing_list)
# passing_rate
```

District Summary

Create a high level snapshot (in table form) of the district's key metrics, including:

Total Schools
Total Students
Total Budget
Average Math Score
Average Reading Score
% Passing Math
% Passing Reading
Overall Passing Rate (Average of the above two)


```python
dict = {"Total Schools": district_count, "Total Students": students_count, "Total Budget": budget_total,
       "Average Math Score": avg_math_score, "Average Reading Score": avg_reading_score,
       "% Passing Math": passing_math, "% Passing Reading": passing_reading, "% Overall Passing Rate": passing_rate}
df = pd.DataFrame([dict], columns=dict.keys())
df.reset_index(drop=True)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
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
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7</td>
      <td>26976</td>
      <td>70439053973</td>
      <td>76.987026</td>
      <td>80.962485</td>
      <td>66.518387</td>
      <td>80.905249</td>
      <td>73.711818</td>
    </tr>
  </tbody>
</table>
</div>



create a school and students data from including all the data -- both District and Charter


```python
schoolstu_df = pd.merge(schools_df,students_df,on="school_name")
school_sum = schoolstu_df.groupby(["school_name","type","name"])
```


```python
# school_sum_no_students = district_df.groupby(["school_name"])
# school_sum_no_students['reading_score'].filter(lambda x: x > 70)
```


```python
# t_school_math_sum = pd.DataFrame(district_df.groupby(["school_name"])["math_score"].mean().reset_index())
# t_school_math_sum = pd.DataFrame(district_df.groupby(["school_name"])(["math_score"]).mean().reset_index())
# t_school_math_sum


# pd.DataFrame(district_df.groupby(["school_name","grade"])["math_score"].mean().reset_index())

# t_school_math_pass = school_sum[school_sum["math_score"]> passing_score]
```

Compute the total of math_score greater than the passing score by each school.


```python
school_sum_mp = school_sum.apply(lambda x: x[x["math_score"]>=passing_score])
# school_sum_mp["school_name"].value_counts()
school_sum_mp = pd.DataFrame(school_sum_mp["school_name"].value_counts().reset_index())
school_sum_mp.columns = ["school_name", "math_pass"]
# school_sum_bySchool = pd.DataFrame(school_sum["math_score"].sum())
# school_sum_bySchool
# math = school_sum_bySchool.loc[school_sum_bySchool["math_score"]>=passing_score]
# math.value_count()
```


```python
# school_sum_mp
```

Compute the total of math_score by each school


```python
school_sum_sCounts = school_sum.apply(lambda x: x[x["math_score"]>=0])
school_sum_sCounts = pd.DataFrame(school_sum_sCounts["school_name"].value_counts().reset_index())
school_sum_sCounts.columns = ["school_name", "students_count"]
# school_sum_sCounts
```

Merge school summary table with total of math_score by school


```python
school_sum_all = pd.merge(school_sum_sCounts,school_sum_mp,on="school_name")
# type(school_sum_all)
```

Compute the total reading_score greater then the passing score by each school


```python
school_sum_rp = school_sum.apply(lambda x: x[x["reading_score"]>=passing_score])
# school_sum_mp["school_name"].value_counts()
school_sum_rp = pd.DataFrame(school_sum_rp["school_name"].value_counts().reset_index())
school_sum_rp.columns = ["school_name", "reading_pass"]
# school_sum_rp
```

Merge school summary and reading_score totals for each school


```python
school_sum_all = pd.merge(school_sum_all,school_sum_rp,on="school_name") # run only once
# school_sum_all
```

Compute % Passing Math by school


```python
school_sum_all["% math pass"] = school_sum_all["math_pass"]* 100/school_sum_all["students_count"]

```

Compute % Passing Reading by school


```python
school_sum_all["% reading pass"] = school_sum_all["reading_pass"]* 100/school_sum_all["students_count"]


```


```python
# school_sum_all
```


```python
school_sum_all = pd.merge(school_sum_all,schools_df[["school_name","budget","type"]],on="school_name") #run only once
```

Compute budget per student


```python
school_sum_all["budget per student"] = school_sum_all["budget"]/school_sum_all["students_count"]
# school_sum_all
```


```python
school_sum_math = pd.DataFrame(schoolstu_df.groupby(["school_name"])["math_score"].sum().reset_index())
school_sum_math.columns = ["school_name", "math_total"]
# school_sum_math
```


```python
school_sum_all = pd.merge(school_sum_all,school_sum_math,on="school_name")
# school_sum_all
```


```python
school_sum_all["Average Math Score"] = school_sum_all["math_total"]/school_sum_all["students_count"]
# school_sum_all
```


```python
school_sum_reading = pd.DataFrame(schoolstu_df.groupby(["school_name"])["reading_score"].sum().reset_index())
school_sum_reading.columns = ["school_name", "reading_total"]
# school_sum_reading
```


```python
school_sum_all = pd.merge(school_sum_all,school_sum_reading,on="school_name")
# school_sum_all
```


```python
school_sum_all["Average Reading Score"] = school_sum_all["reading_total"]/school_sum_all["students_count"]
# school_sum_all
```


```python
school_sum_all["Overall passing rate"] = (school_sum_all["% math pass"] + school_sum_all["% reading pass"])/2
# school_sum_all.sort(on="Overall passing rate")
```


```python
school_sum_all = school_sum_all.sort_values("Overall passing rate",ascending=False).reset_index(drop=True)
```

School Summary


Create an overview table that summarizes key metrics about each school, including:


School Name
School Type
Total Students
Total School Budget
Per Student Budget
Average Math Score
Average Reading Score
% Passing Math
% Passing Reading
Overall Passing Rate (Average of the above two)


```python
school_sum_all
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school_name</th>
      <th>students_count</th>
      <th>math_pass</th>
      <th>reading_pass</th>
      <th>% math pass</th>
      <th>% reading pass</th>
      <th>budget</th>
      <th>type</th>
      <th>budget per student</th>
      <th>math_total</th>
      <th>Average Math Score</th>
      <th>reading_total</th>
      <th>Average Reading Score</th>
      <th>Overall passing rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Cabrera High School</td>
      <td>1858</td>
      <td>1749</td>
      <td>1803</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>1081356</td>
      <td>Charter</td>
      <td>582.0</td>
      <td>154329</td>
      <td>83.061895</td>
      <td>156027</td>
      <td>83.975780</td>
      <td>95.586652</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thomas High School</td>
      <td>1635</td>
      <td>1525</td>
      <td>1591</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>1043130</td>
      <td>Charter</td>
      <td>638.0</td>
      <td>136389</td>
      <td>83.418349</td>
      <td>137093</td>
      <td>83.848930</td>
      <td>95.290520</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pena High School</td>
      <td>962</td>
      <td>910</td>
      <td>923</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>585858</td>
      <td>Charter</td>
      <td>609.0</td>
      <td>80654</td>
      <td>83.839917</td>
      <td>80851</td>
      <td>84.044699</td>
      <td>95.270270</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Griffin High School</td>
      <td>1468</td>
      <td>1371</td>
      <td>1426</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>917500</td>
      <td>Charter</td>
      <td>625.0</td>
      <td>122360</td>
      <td>83.351499</td>
      <td>123043</td>
      <td>83.816757</td>
      <td>95.265668</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Wilson High School</td>
      <td>2283</td>
      <td>2143</td>
      <td>2204</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>1319574</td>
      <td>Charter</td>
      <td>578.0</td>
      <td>190115</td>
      <td>83.274201</td>
      <td>191748</td>
      <td>83.989488</td>
      <td>95.203679</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Wright High School</td>
      <td>1800</td>
      <td>1680</td>
      <td>1739</td>
      <td>93.333333</td>
      <td>96.611111</td>
      <td>1049400</td>
      <td>Charter</td>
      <td>583.0</td>
      <td>150628</td>
      <td>83.682222</td>
      <td>151119</td>
      <td>83.955000</td>
      <td>94.972222</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Shelton High School</td>
      <td>1761</td>
      <td>1653</td>
      <td>1688</td>
      <td>93.867121</td>
      <td>95.854628</td>
      <td>1056600</td>
      <td>Charter</td>
      <td>600.0</td>
      <td>146796</td>
      <td>83.359455</td>
      <td>147441</td>
      <td>83.725724</td>
      <td>94.860875</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Holden High School</td>
      <td>427</td>
      <td>395</td>
      <td>411</td>
      <td>92.505855</td>
      <td>96.252927</td>
      <td>248087</td>
      <td>Charter</td>
      <td>581.0</td>
      <td>35784</td>
      <td>83.803279</td>
      <td>35789</td>
      <td>83.814988</td>
      <td>94.379391</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Bailey High School</td>
      <td>4976</td>
      <td>3318</td>
      <td>4077</td>
      <td>66.680064</td>
      <td>81.933280</td>
      <td>3124928</td>
      <td>District</td>
      <td>628.0</td>
      <td>383393</td>
      <td>77.048432</td>
      <td>403225</td>
      <td>81.033963</td>
      <td>74.306672</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Hernandez High School</td>
      <td>4635</td>
      <td>3094</td>
      <td>3748</td>
      <td>66.752967</td>
      <td>80.862999</td>
      <td>3022020</td>
      <td>District</td>
      <td>652.0</td>
      <td>358238</td>
      <td>77.289752</td>
      <td>375131</td>
      <td>80.934412</td>
      <td>73.807983</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Ford High School</td>
      <td>2739</td>
      <td>1871</td>
      <td>2172</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>1763916</td>
      <td>District</td>
      <td>644.0</td>
      <td>211184</td>
      <td>77.102592</td>
      <td>221164</td>
      <td>80.746258</td>
      <td>73.804308</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Johnson High School</td>
      <td>4761</td>
      <td>3145</td>
      <td>3867</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>3094650</td>
      <td>District</td>
      <td>650.0</td>
      <td>366942</td>
      <td>77.072464</td>
      <td>385481</td>
      <td>80.966394</td>
      <td>73.639992</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Huang High School</td>
      <td>2917</td>
      <td>1916</td>
      <td>2372</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>1910635</td>
      <td>District</td>
      <td>655.0</td>
      <td>223528</td>
      <td>76.629414</td>
      <td>236810</td>
      <td>81.182722</td>
      <td>73.500171</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Figueroa High School</td>
      <td>2949</td>
      <td>1946</td>
      <td>2381</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>1884411</td>
      <td>District</td>
      <td>639.0</td>
      <td>226223</td>
      <td>76.711767</td>
      <td>239335</td>
      <td>81.158020</td>
      <td>73.363852</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Rodriguez High School</td>
      <td>3999</td>
      <td>2654</td>
      <td>3208</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>2547363</td>
      <td>District</td>
      <td>637.0</td>
      <td>307294</td>
      <td>76.842711</td>
      <td>322898</td>
      <td>80.744686</td>
      <td>73.293323</td>
    </tr>
  </tbody>
</table>
</div>



List the top 5 schools based on 'Overall passing rate'


```python
school_sum_all.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school_name</th>
      <th>students_count</th>
      <th>math_pass</th>
      <th>reading_pass</th>
      <th>% math pass</th>
      <th>% reading pass</th>
      <th>budget</th>
      <th>type</th>
      <th>budget per student</th>
      <th>math_total</th>
      <th>Average Math Score</th>
      <th>reading_total</th>
      <th>Average Reading Score</th>
      <th>Overall passing rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Cabrera High School</td>
      <td>1858</td>
      <td>1749</td>
      <td>1803</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>1081356</td>
      <td>Charter</td>
      <td>582.0</td>
      <td>154329</td>
      <td>83.061895</td>
      <td>156027</td>
      <td>83.975780</td>
      <td>95.586652</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thomas High School</td>
      <td>1635</td>
      <td>1525</td>
      <td>1591</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>1043130</td>
      <td>Charter</td>
      <td>638.0</td>
      <td>136389</td>
      <td>83.418349</td>
      <td>137093</td>
      <td>83.848930</td>
      <td>95.290520</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pena High School</td>
      <td>962</td>
      <td>910</td>
      <td>923</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>585858</td>
      <td>Charter</td>
      <td>609.0</td>
      <td>80654</td>
      <td>83.839917</td>
      <td>80851</td>
      <td>84.044699</td>
      <td>95.270270</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Griffin High School</td>
      <td>1468</td>
      <td>1371</td>
      <td>1426</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>917500</td>
      <td>Charter</td>
      <td>625.0</td>
      <td>122360</td>
      <td>83.351499</td>
      <td>123043</td>
      <td>83.816757</td>
      <td>95.265668</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Wilson High School</td>
      <td>2283</td>
      <td>2143</td>
      <td>2204</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>1319574</td>
      <td>Charter</td>
      <td>578.0</td>
      <td>190115</td>
      <td>83.274201</td>
      <td>191748</td>
      <td>83.989488</td>
      <td>95.203679</td>
    </tr>
  </tbody>
</table>
</div>



List the bottom 5 schools based on 'Overall passing rate'


```python
school_sum_all = school_sum_all.sort_values("Overall passing rate",ascending=True).reset_index(drop=True)
school_sum_all.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school_name</th>
      <th>students_count</th>
      <th>math_pass</th>
      <th>reading_pass</th>
      <th>% math pass</th>
      <th>% reading pass</th>
      <th>budget</th>
      <th>type</th>
      <th>budget per student</th>
      <th>math_total</th>
      <th>Average Math Score</th>
      <th>reading_total</th>
      <th>Average Reading Score</th>
      <th>Overall passing rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Rodriguez High School</td>
      <td>3999</td>
      <td>2654</td>
      <td>3208</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>2547363</td>
      <td>District</td>
      <td>637.0</td>
      <td>307294</td>
      <td>76.842711</td>
      <td>322898</td>
      <td>80.744686</td>
      <td>73.293323</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Figueroa High School</td>
      <td>2949</td>
      <td>1946</td>
      <td>2381</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>1884411</td>
      <td>District</td>
      <td>639.0</td>
      <td>226223</td>
      <td>76.711767</td>
      <td>239335</td>
      <td>81.158020</td>
      <td>73.363852</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Huang High School</td>
      <td>2917</td>
      <td>1916</td>
      <td>2372</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>1910635</td>
      <td>District</td>
      <td>655.0</td>
      <td>223528</td>
      <td>76.629414</td>
      <td>236810</td>
      <td>81.182722</td>
      <td>73.500171</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Johnson High School</td>
      <td>4761</td>
      <td>3145</td>
      <td>3867</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>3094650</td>
      <td>District</td>
      <td>650.0</td>
      <td>366942</td>
      <td>77.072464</td>
      <td>385481</td>
      <td>80.966394</td>
      <td>73.639992</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ford High School</td>
      <td>2739</td>
      <td>1871</td>
      <td>2172</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>1763916</td>
      <td>District</td>
      <td>644.0</td>
      <td>211184</td>
      <td>77.102592</td>
      <td>221164</td>
      <td>80.746258</td>
      <td>73.804308</td>
    </tr>
  </tbody>
</table>
</div>



Math Scores by Grade


Table that lists the average Math Score for students of each grade level (9th, 10th, 11th, 12th) at each school.


```python
school_grade_math_sum = pd.DataFrame(schoolstu_df.groupby(["school_name","grade"])["math_score"].mean().reset_index())
school_grade_math_sum.columns = ["school_name", "grade","math_score"]
school_grade_math_sum["dgrade"] = pd.to_numeric(school_grade_math_sum["grade"].str.extract('(^\d*)'))

school_grade_math_sum = school_grade_math_sum.sort_values(["school_name","dgrade"],ascending=True).reset_index(drop=True)
school_grade_math_sum = school_grade_math_sum.rename(columns={'math_score':'Average Math Score'})
school_grade_math_sum = school_grade_math_sum.drop(columns=["dgrade"])
school_grade_math_sum
```

    /Users/rkagathi/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:3: FutureWarning: currently extract(expand=None) means expand=False (return Index/Series/DataFrame) but in a future version of pandas this will be changed to expand=True (return DataFrame)
      This is separate from the ipykernel package so we can avoid doing imports until





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school_name</th>
      <th>grade</th>
      <th>Average Math Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>9th</td>
      <td>77.083676</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bailey High School</td>
      <td>10th</td>
      <td>76.996772</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bailey High School</td>
      <td>11th</td>
      <td>77.515588</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bailey High School</td>
      <td>12th</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cabrera High School</td>
      <td>9th</td>
      <td>83.094697</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Cabrera High School</td>
      <td>10th</td>
      <td>83.154506</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Cabrera High School</td>
      <td>11th</td>
      <td>82.765560</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Cabrera High School</td>
      <td>12th</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Figueroa High School</td>
      <td>9th</td>
      <td>76.403037</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Figueroa High School</td>
      <td>10th</td>
      <td>76.539974</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Figueroa High School</td>
      <td>11th</td>
      <td>76.884344</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Figueroa High School</td>
      <td>12th</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Ford High School</td>
      <td>9th</td>
      <td>77.361345</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Ford High School</td>
      <td>10th</td>
      <td>77.672316</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Ford High School</td>
      <td>11th</td>
      <td>76.918058</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Ford High School</td>
      <td>12th</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Griffin High School</td>
      <td>9th</td>
      <td>82.044010</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Griffin High School</td>
      <td>10th</td>
      <td>84.229064</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Griffin High School</td>
      <td>11th</td>
      <td>83.842105</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Griffin High School</td>
      <td>12th</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Hernandez High School</td>
      <td>9th</td>
      <td>77.438495</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Hernandez High School</td>
      <td>10th</td>
      <td>77.337408</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Hernandez High School</td>
      <td>11th</td>
      <td>77.136029</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Hernandez High School</td>
      <td>12th</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Holden High School</td>
      <td>9th</td>
      <td>83.787402</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Holden High School</td>
      <td>10th</td>
      <td>83.429825</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Holden High School</td>
      <td>11th</td>
      <td>85.000000</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Holden High School</td>
      <td>12th</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Huang High School</td>
      <td>9th</td>
      <td>77.027251</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Huang High School</td>
      <td>10th</td>
      <td>75.908735</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Huang High School</td>
      <td>11th</td>
      <td>76.446602</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Huang High School</td>
      <td>12th</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Johnson High School</td>
      <td>9th</td>
      <td>77.187857</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Johnson High School</td>
      <td>10th</td>
      <td>76.691117</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Johnson High School</td>
      <td>11th</td>
      <td>77.491653</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Johnson High School</td>
      <td>12th</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Pena High School</td>
      <td>9th</td>
      <td>83.625455</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Pena High School</td>
      <td>10th</td>
      <td>83.372000</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Pena High School</td>
      <td>11th</td>
      <td>84.328125</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Pena High School</td>
      <td>12th</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Rodriguez High School</td>
      <td>9th</td>
      <td>76.859966</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Rodriguez High School</td>
      <td>10th</td>
      <td>76.612500</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Rodriguez High School</td>
      <td>11th</td>
      <td>76.395626</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Rodriguez High School</td>
      <td>12th</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Shelton High School</td>
      <td>9th</td>
      <td>83.420755</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Shelton High School</td>
      <td>10th</td>
      <td>82.917411</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Shelton High School</td>
      <td>11th</td>
      <td>83.383495</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Shelton High School</td>
      <td>12th</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Thomas High School</td>
      <td>9th</td>
      <td>83.590022</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Thomas High School</td>
      <td>10th</td>
      <td>83.087886</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Thomas High School</td>
      <td>11th</td>
      <td>83.498795</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Thomas High School</td>
      <td>12th</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Wilson High School</td>
      <td>9th</td>
      <td>83.085578</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Wilson High School</td>
      <td>10th</td>
      <td>83.724422</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Wilson High School</td>
      <td>11th</td>
      <td>83.195326</td>
    </tr>
    <tr>
      <th>55</th>
      <td>Wilson High School</td>
      <td>12th</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Wright High School</td>
      <td>9th</td>
      <td>83.264706</td>
    </tr>
    <tr>
      <th>57</th>
      <td>Wright High School</td>
      <td>10th</td>
      <td>84.010288</td>
    </tr>
    <tr>
      <th>58</th>
      <td>Wright High School</td>
      <td>11th</td>
      <td>83.836782</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Wright High School</td>
      <td>12th</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



Reading Scores by Grade


Table that lists the average Reading Score for students of each grade level (9th, 10th, 11th, 12th) at each school.


```python
school_grade_reading_sum = pd.DataFrame(schoolstu_df.groupby(["school_name","grade"])["reading_score"].mean().reset_index())
school_grade_reading_sum.columns = ["school_name", "grade","reading_score"]
school_grade_reading_sum["dgrade"] = pd.to_numeric(school_grade_reading_sum["grade"].str.extract('(^\d*)'))

school_grade_reading_sum = school_grade_reading_sum.sort_values(["school_name","dgrade"],ascending=True).reset_index(drop=True)
school_grade_reading_sum = school_grade_reading_sum.rename(columns={'reading_score':'Average Reading Score'})
school_grade_reading_sum = school_grade_reading_sum.drop(columns=["dgrade"])
school_grade_reading_sum
```

    /Users/rkagathi/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:3: FutureWarning: currently extract(expand=None) means expand=False (return Index/Series/DataFrame) but in a future version of pandas this will be changed to expand=True (return DataFrame)
      This is separate from the ipykernel package so we can avoid doing imports until





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school_name</th>
      <th>grade</th>
      <th>Average Reading Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>9th</td>
      <td>81.303155</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bailey High School</td>
      <td>10th</td>
      <td>80.907183</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bailey High School</td>
      <td>11th</td>
      <td>80.945643</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bailey High School</td>
      <td>12th</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cabrera High School</td>
      <td>9th</td>
      <td>83.676136</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Cabrera High School</td>
      <td>10th</td>
      <td>84.253219</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Cabrera High School</td>
      <td>11th</td>
      <td>83.788382</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Cabrera High School</td>
      <td>12th</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Figueroa High School</td>
      <td>9th</td>
      <td>81.198598</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Figueroa High School</td>
      <td>10th</td>
      <td>81.408912</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Figueroa High School</td>
      <td>11th</td>
      <td>80.640339</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Figueroa High School</td>
      <td>12th</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Ford High School</td>
      <td>9th</td>
      <td>80.632653</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Ford High School</td>
      <td>10th</td>
      <td>81.262712</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Ford High School</td>
      <td>11th</td>
      <td>80.403642</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Ford High School</td>
      <td>12th</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Griffin High School</td>
      <td>9th</td>
      <td>83.369193</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Griffin High School</td>
      <td>10th</td>
      <td>83.706897</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Griffin High School</td>
      <td>11th</td>
      <td>84.288089</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Griffin High School</td>
      <td>12th</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Hernandez High School</td>
      <td>9th</td>
      <td>80.866860</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Hernandez High School</td>
      <td>10th</td>
      <td>80.660147</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Hernandez High School</td>
      <td>11th</td>
      <td>81.396140</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Hernandez High School</td>
      <td>12th</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Holden High School</td>
      <td>9th</td>
      <td>83.677165</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Holden High School</td>
      <td>10th</td>
      <td>83.324561</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Holden High School</td>
      <td>11th</td>
      <td>83.815534</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Holden High School</td>
      <td>12th</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Huang High School</td>
      <td>9th</td>
      <td>81.290284</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Huang High School</td>
      <td>10th</td>
      <td>81.512386</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Huang High School</td>
      <td>11th</td>
      <td>81.417476</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Huang High School</td>
      <td>12th</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Johnson High School</td>
      <td>9th</td>
      <td>81.260714</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Johnson High School</td>
      <td>10th</td>
      <td>80.773431</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Johnson High School</td>
      <td>11th</td>
      <td>80.616027</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Johnson High School</td>
      <td>12th</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Pena High School</td>
      <td>9th</td>
      <td>83.807273</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Pena High School</td>
      <td>10th</td>
      <td>83.612000</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Pena High School</td>
      <td>11th</td>
      <td>84.335938</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Pena High School</td>
      <td>12th</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Rodriguez High School</td>
      <td>9th</td>
      <td>80.993127</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Rodriguez High School</td>
      <td>10th</td>
      <td>80.629808</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Rodriguez High School</td>
      <td>11th</td>
      <td>80.864811</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Rodriguez High School</td>
      <td>12th</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Shelton High School</td>
      <td>9th</td>
      <td>84.122642</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Shelton High School</td>
      <td>10th</td>
      <td>83.441964</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Shelton High School</td>
      <td>11th</td>
      <td>84.373786</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Shelton High School</td>
      <td>12th</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Thomas High School</td>
      <td>9th</td>
      <td>83.728850</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Thomas High School</td>
      <td>10th</td>
      <td>84.254157</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Thomas High School</td>
      <td>11th</td>
      <td>83.585542</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Thomas High School</td>
      <td>12th</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Wilson High School</td>
      <td>9th</td>
      <td>83.939778</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Wilson High School</td>
      <td>10th</td>
      <td>84.021452</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Wilson High School</td>
      <td>11th</td>
      <td>83.764608</td>
    </tr>
    <tr>
      <th>55</th>
      <td>Wilson High School</td>
      <td>12th</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Wright High School</td>
      <td>9th</td>
      <td>83.833333</td>
    </tr>
    <tr>
      <th>57</th>
      <td>Wright High School</td>
      <td>10th</td>
      <td>83.812757</td>
    </tr>
    <tr>
      <th>58</th>
      <td>Wright High School</td>
      <td>11th</td>
      <td>84.156322</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Wright High School</td>
      <td>12th</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>




```python
bins = [560,600,620,640,680]

# Create the names for the four bins
group_names = ['Low', 'Medium', 'High', 'Very High']
```


```python
# Cut budget and place the scores into bins
pd.cut(school_sum_all["budget per student"], bins, labels=group_names)
```




    0          High
    1          High
    2     Very High
    3     Very High
    4     Very High
    5     Very High
    6          High
    7           Low
    8           Low
    9           Low
    10          Low
    11         High
    12       Medium
    13         High
    14          Low
    Name: budget per student, dtype: category
    Categories (4, object): [Low < Medium < High < Very High]




```python
school_sum_all["Budget bin"]= pd.cut(
    school_sum_all["budget per student"], bins, labels=group_names)
# school_sum_all
```

Scores by School Spending


Table that breaks down school performances based on average Spending Ranges (Per Student). Use 4 reasonable bins to group school spending. Include in the table each of the following:


Average Math Score
Average Reading Score
% Passing Math
% Passing Reading
Overall Passing Rate (Average of the above two)


```python
school_sum_all = school_sum_all.sort_values("Overall passing rate",ascending=False).reset_index(drop=True)
school_sum_all
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school_name</th>
      <th>students_count</th>
      <th>math_pass</th>
      <th>reading_pass</th>
      <th>% math pass</th>
      <th>% reading pass</th>
      <th>budget</th>
      <th>type</th>
      <th>budget per student</th>
      <th>math_total</th>
      <th>Average Math Score</th>
      <th>reading_total</th>
      <th>Average Reading Score</th>
      <th>Overall passing rate</th>
      <th>Budget bin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Cabrera High School</td>
      <td>1858</td>
      <td>1749</td>
      <td>1803</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>1081356</td>
      <td>Charter</td>
      <td>582.0</td>
      <td>154329</td>
      <td>83.061895</td>
      <td>156027</td>
      <td>83.975780</td>
      <td>95.586652</td>
      <td>Low</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thomas High School</td>
      <td>1635</td>
      <td>1525</td>
      <td>1591</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>1043130</td>
      <td>Charter</td>
      <td>638.0</td>
      <td>136389</td>
      <td>83.418349</td>
      <td>137093</td>
      <td>83.848930</td>
      <td>95.290520</td>
      <td>High</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pena High School</td>
      <td>962</td>
      <td>910</td>
      <td>923</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>585858</td>
      <td>Charter</td>
      <td>609.0</td>
      <td>80654</td>
      <td>83.839917</td>
      <td>80851</td>
      <td>84.044699</td>
      <td>95.270270</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Griffin High School</td>
      <td>1468</td>
      <td>1371</td>
      <td>1426</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>917500</td>
      <td>Charter</td>
      <td>625.0</td>
      <td>122360</td>
      <td>83.351499</td>
      <td>123043</td>
      <td>83.816757</td>
      <td>95.265668</td>
      <td>High</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Wilson High School</td>
      <td>2283</td>
      <td>2143</td>
      <td>2204</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>1319574</td>
      <td>Charter</td>
      <td>578.0</td>
      <td>190115</td>
      <td>83.274201</td>
      <td>191748</td>
      <td>83.989488</td>
      <td>95.203679</td>
      <td>Low</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Wright High School</td>
      <td>1800</td>
      <td>1680</td>
      <td>1739</td>
      <td>93.333333</td>
      <td>96.611111</td>
      <td>1049400</td>
      <td>Charter</td>
      <td>583.0</td>
      <td>150628</td>
      <td>83.682222</td>
      <td>151119</td>
      <td>83.955000</td>
      <td>94.972222</td>
      <td>Low</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Shelton High School</td>
      <td>1761</td>
      <td>1653</td>
      <td>1688</td>
      <td>93.867121</td>
      <td>95.854628</td>
      <td>1056600</td>
      <td>Charter</td>
      <td>600.0</td>
      <td>146796</td>
      <td>83.359455</td>
      <td>147441</td>
      <td>83.725724</td>
      <td>94.860875</td>
      <td>Low</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Holden High School</td>
      <td>427</td>
      <td>395</td>
      <td>411</td>
      <td>92.505855</td>
      <td>96.252927</td>
      <td>248087</td>
      <td>Charter</td>
      <td>581.0</td>
      <td>35784</td>
      <td>83.803279</td>
      <td>35789</td>
      <td>83.814988</td>
      <td>94.379391</td>
      <td>Low</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Bailey High School</td>
      <td>4976</td>
      <td>3318</td>
      <td>4077</td>
      <td>66.680064</td>
      <td>81.933280</td>
      <td>3124928</td>
      <td>District</td>
      <td>628.0</td>
      <td>383393</td>
      <td>77.048432</td>
      <td>403225</td>
      <td>81.033963</td>
      <td>74.306672</td>
      <td>High</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Hernandez High School</td>
      <td>4635</td>
      <td>3094</td>
      <td>3748</td>
      <td>66.752967</td>
      <td>80.862999</td>
      <td>3022020</td>
      <td>District</td>
      <td>652.0</td>
      <td>358238</td>
      <td>77.289752</td>
      <td>375131</td>
      <td>80.934412</td>
      <td>73.807983</td>
      <td>Very High</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Ford High School</td>
      <td>2739</td>
      <td>1871</td>
      <td>2172</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>1763916</td>
      <td>District</td>
      <td>644.0</td>
      <td>211184</td>
      <td>77.102592</td>
      <td>221164</td>
      <td>80.746258</td>
      <td>73.804308</td>
      <td>Very High</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Johnson High School</td>
      <td>4761</td>
      <td>3145</td>
      <td>3867</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>3094650</td>
      <td>District</td>
      <td>650.0</td>
      <td>366942</td>
      <td>77.072464</td>
      <td>385481</td>
      <td>80.966394</td>
      <td>73.639992</td>
      <td>Very High</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Huang High School</td>
      <td>2917</td>
      <td>1916</td>
      <td>2372</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>1910635</td>
      <td>District</td>
      <td>655.0</td>
      <td>223528</td>
      <td>76.629414</td>
      <td>236810</td>
      <td>81.182722</td>
      <td>73.500171</td>
      <td>Very High</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Figueroa High School</td>
      <td>2949</td>
      <td>1946</td>
      <td>2381</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>1884411</td>
      <td>District</td>
      <td>639.0</td>
      <td>226223</td>
      <td>76.711767</td>
      <td>239335</td>
      <td>81.158020</td>
      <td>73.363852</td>
      <td>High</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Rodriguez High School</td>
      <td>3999</td>
      <td>2654</td>
      <td>3208</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>2547363</td>
      <td>District</td>
      <td>637.0</td>
      <td>307294</td>
      <td>76.842711</td>
      <td>322898</td>
      <td>80.744686</td>
      <td>73.293323</td>
      <td>High</td>
    </tr>
  </tbody>
</table>
</div>




```python
bins = [0,2000,4000,6000]

# Create the names for the four bins
group_names = ['Small', 'Medium', 'Large']
```


```python
# Cut school size and place the scores into bins
pd.cut(school_sum_all["students_count"], bins, labels=group_names)
```




    0      Small
    1      Small
    2      Small
    3      Small
    4     Medium
    5      Small
    6      Small
    7      Small
    8      Large
    9      Large
    10    Medium
    11     Large
    12    Medium
    13    Medium
    14    Medium
    Name: students_count, dtype: category
    Categories (3, object): [Small < Medium < Large]



Scores by School Size


Repeat of the above breakdown, but this time group schools based on a reasonable approximation of school size (Small, Medium, Large).


```python
school_sum_all["School size bin"]= pd.cut(
    school_sum_all["students_count"], bins, labels=group_names)
school_sum_all
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>school_name</th>
      <th>students_count</th>
      <th>math_pass</th>
      <th>reading_pass</th>
      <th>% math pass</th>
      <th>% reading pass</th>
      <th>budget</th>
      <th>type</th>
      <th>budget per student</th>
      <th>math_total</th>
      <th>Average Math Score</th>
      <th>reading_total</th>
      <th>Average Reading Score</th>
      <th>Overall passing rate</th>
      <th>Budget bin</th>
      <th>School size bin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Cabrera High School</td>
      <td>1858</td>
      <td>1749</td>
      <td>1803</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>1081356</td>
      <td>Charter</td>
      <td>582.0</td>
      <td>154329</td>
      <td>83.061895</td>
      <td>156027</td>
      <td>83.975780</td>
      <td>95.586652</td>
      <td>Low</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Thomas High School</td>
      <td>1635</td>
      <td>1525</td>
      <td>1591</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>1043130</td>
      <td>Charter</td>
      <td>638.0</td>
      <td>136389</td>
      <td>83.418349</td>
      <td>137093</td>
      <td>83.848930</td>
      <td>95.290520</td>
      <td>High</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pena High School</td>
      <td>962</td>
      <td>910</td>
      <td>923</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>585858</td>
      <td>Charter</td>
      <td>609.0</td>
      <td>80654</td>
      <td>83.839917</td>
      <td>80851</td>
      <td>84.044699</td>
      <td>95.270270</td>
      <td>Medium</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Griffin High School</td>
      <td>1468</td>
      <td>1371</td>
      <td>1426</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>917500</td>
      <td>Charter</td>
      <td>625.0</td>
      <td>122360</td>
      <td>83.351499</td>
      <td>123043</td>
      <td>83.816757</td>
      <td>95.265668</td>
      <td>High</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Wilson High School</td>
      <td>2283</td>
      <td>2143</td>
      <td>2204</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>1319574</td>
      <td>Charter</td>
      <td>578.0</td>
      <td>190115</td>
      <td>83.274201</td>
      <td>191748</td>
      <td>83.989488</td>
      <td>95.203679</td>
      <td>Low</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Wright High School</td>
      <td>1800</td>
      <td>1680</td>
      <td>1739</td>
      <td>93.333333</td>
      <td>96.611111</td>
      <td>1049400</td>
      <td>Charter</td>
      <td>583.0</td>
      <td>150628</td>
      <td>83.682222</td>
      <td>151119</td>
      <td>83.955000</td>
      <td>94.972222</td>
      <td>Low</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Shelton High School</td>
      <td>1761</td>
      <td>1653</td>
      <td>1688</td>
      <td>93.867121</td>
      <td>95.854628</td>
      <td>1056600</td>
      <td>Charter</td>
      <td>600.0</td>
      <td>146796</td>
      <td>83.359455</td>
      <td>147441</td>
      <td>83.725724</td>
      <td>94.860875</td>
      <td>Low</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Holden High School</td>
      <td>427</td>
      <td>395</td>
      <td>411</td>
      <td>92.505855</td>
      <td>96.252927</td>
      <td>248087</td>
      <td>Charter</td>
      <td>581.0</td>
      <td>35784</td>
      <td>83.803279</td>
      <td>35789</td>
      <td>83.814988</td>
      <td>94.379391</td>
      <td>Low</td>
      <td>Small</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Bailey High School</td>
      <td>4976</td>
      <td>3318</td>
      <td>4077</td>
      <td>66.680064</td>
      <td>81.933280</td>
      <td>3124928</td>
      <td>District</td>
      <td>628.0</td>
      <td>383393</td>
      <td>77.048432</td>
      <td>403225</td>
      <td>81.033963</td>
      <td>74.306672</td>
      <td>High</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Hernandez High School</td>
      <td>4635</td>
      <td>3094</td>
      <td>3748</td>
      <td>66.752967</td>
      <td>80.862999</td>
      <td>3022020</td>
      <td>District</td>
      <td>652.0</td>
      <td>358238</td>
      <td>77.289752</td>
      <td>375131</td>
      <td>80.934412</td>
      <td>73.807983</td>
      <td>Very High</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Ford High School</td>
      <td>2739</td>
      <td>1871</td>
      <td>2172</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>1763916</td>
      <td>District</td>
      <td>644.0</td>
      <td>211184</td>
      <td>77.102592</td>
      <td>221164</td>
      <td>80.746258</td>
      <td>73.804308</td>
      <td>Very High</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Johnson High School</td>
      <td>4761</td>
      <td>3145</td>
      <td>3867</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>3094650</td>
      <td>District</td>
      <td>650.0</td>
      <td>366942</td>
      <td>77.072464</td>
      <td>385481</td>
      <td>80.966394</td>
      <td>73.639992</td>
      <td>Very High</td>
      <td>Large</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Huang High School</td>
      <td>2917</td>
      <td>1916</td>
      <td>2372</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>1910635</td>
      <td>District</td>
      <td>655.0</td>
      <td>223528</td>
      <td>76.629414</td>
      <td>236810</td>
      <td>81.182722</td>
      <td>73.500171</td>
      <td>Very High</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Figueroa High School</td>
      <td>2949</td>
      <td>1946</td>
      <td>2381</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>1884411</td>
      <td>District</td>
      <td>639.0</td>
      <td>226223</td>
      <td>76.711767</td>
      <td>239335</td>
      <td>81.158020</td>
      <td>73.363852</td>
      <td>High</td>
      <td>Medium</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Rodriguez High School</td>
      <td>3999</td>
      <td>2654</td>
      <td>3208</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>2547363</td>
      <td>District</td>
      <td>637.0</td>
      <td>307294</td>
      <td>76.842711</td>
      <td>322898</td>
      <td>80.744686</td>
      <td>73.293323</td>
      <td>High</td>
      <td>Medium</td>
    </tr>
  </tbody>
</table>
</div>



School Type, Budget and Overall Passing Rate summary


```python
school_type_budget_sum = pd.DataFrame(school_sum_all.groupby(["type","Budget bin"])["Overall passing rate"].mean().reset_index())
school_type_budget_sum
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>Budget bin</th>
      <th>Overall passing rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Charter</td>
      <td>Low</td>
      <td>95.000564</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Charter</td>
      <td>Medium</td>
      <td>95.270270</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Charter</td>
      <td>High</td>
      <td>95.278094</td>
    </tr>
    <tr>
      <th>3</th>
      <td>District</td>
      <td>High</td>
      <td>73.654616</td>
    </tr>
    <tr>
      <th>4</th>
      <td>District</td>
      <td>Very High</td>
      <td>73.688113</td>
    </tr>
  </tbody>
</table>
</div>



School Type, School Size and Overall Passing Rate summary


```python
school_typ_size_sum= pd.DataFrame(school_sum_all.groupby(["type","School size bin"])["Overall passing rate"].mean().reset_index())
school_typ_size_sum
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>School size bin</th>
      <th>Overall passing rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Charter</td>
      <td>Small</td>
      <td>95.089371</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Charter</td>
      <td>Medium</td>
      <td>95.203679</td>
    </tr>
    <tr>
      <th>2</th>
      <td>District</td>
      <td>Medium</td>
      <td>73.490414</td>
    </tr>
    <tr>
      <th>3</th>
      <td>District</td>
      <td>Large</td>
      <td>73.918215</td>
    </tr>
  </tbody>
</table>
</div>



Three observable trends based on the data.
<li> Charter Schools fare well in Overall passing rate compared to District Schools </li>
<li> Smaller school size results in higher Overall passing rate </li>
<li> Budget per student is less in Charter Schools but still Overall passing rate is high compared to District Schools </li>
