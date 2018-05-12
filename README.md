
# PyCity Schools Analysis

* As a whole, schools with higher budgets, did not yield better test results. By contrast, schools with higher spending per student actually (\$645-675) underperformed compared to schools with smaller budgets (<\$585 per student).

* As a whole, smaller and medium sized schools dramatically out-performed large sized schools on passing math performances (89-91% passing vs 67%).

* As a whole, charter schools out-performed the public district schools across all metrics. However, more analysis will be required to glean if the effect is due to school practices or the fact that charter schools tend to serve smaller student populations per school. 
---


```python
import pandas as pd
```


```python
# read data
schools = pd.read_csv('./raw_data/schools_complete.csv')
students = pd.read_csv('./raw_data/students_complete.csv')
```

## District Summary


```python
# District-level school data
num_schools = schools['School ID'].count()
total_budget = schools['budget'].sum()

# Merge data (student-level)
school_students = pd.merge(schools, students, left_on='name', right_on='school', how='left')
school_students.rename(columns={'name_x': 'School Name', 'name_y': 'Student Name', 'grade': 'Grade'}, inplace=True)

# Add passing columns, assuming >= 70 is passing and one must pass both reading and math to pass overall
school_students['Passing Reading'] = school_students['reading_score'] >= 70
school_students['Passing Math'] = school_students['math_score'] >= 70
school_students['Passing Overall'] = school_students['Passing Math'] & school_students['Passing Reading']

# Get district-level passing counts
pass_math = school_students[school_students['Passing Math']]['Student ID'].count()
pass_reading = school_students[school_students['Passing Reading']]['Student ID'].count()
pass_total = school_students[school_students['Passing Overall']]['Student ID'].count()

# Total-level group includes count of students and average math and reading scores.
funcs = {'Student ID': 'count', 'math_score': 'mean', 'reading_score': 'mean'}

# Hacky way to group without a groupby column (aggregate to district level)
ss_total_group = school_students.groupby(by=lambda x: 0)

# aggregate to count students and get means of math and reading scores
ss_total = ss_total_group.agg(funcs)
ss_total.rename(columns={'reading_score': 'Average Reading Score', 'math_score': 'Average Math Score'
                        ,'Student ID': 'Total Students'}, inplace=True)

# Merge student- and school-level data
ss_total['% Passing Math'] = pass_math / ss_total['Total Students']
ss_total['% Passing Reading'] = pass_reading / ss_total['Total Students']
ss_total['% Passing Overall'] = pass_total / ss_total['Total Students']
ss_total['Total Budget'] = total_budget
ss_total['Total Schools'] = num_schools
ss_total_formatted = ss_total
ss_total[['Total Schools', 'Total Students', 'Total Budget', 'Average Math Score', 'Average Reading Score', '% Passing Math', '% Passing Reading', '% Passing Overall']]
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
      <th>% Passing Overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39170</td>
      <td>24649428</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>0.749809</td>
      <td>0.858055</td>
      <td>0.651723</td>
    </tr>
  </tbody>
</table>
</div>



## School Summary


```python
# school-level data
school_result = schools[['name', 'type', 'budget']]

# student-level data by school
school_students_gb = school_students.groupby(by='School ID')
funcs = {'Student ID': 'count', 'math_score': 'mean', 'reading_score': 'mean', 'Passing Math': 'sum', 'Passing Reading': 'sum', 'Passing Overall': 'sum'}#, 'Passing Reading': 'count', 'pass_overall': 'count'}

school_totals = school_students_gb.agg(funcs)

school_result = pd.merge(schools[['name', 'type', 'budget']], school_totals, left_index=True, right_index=True)
school_result['Per Student Budget'] = school_result['budget'] / school_result['Student ID']
school_result['% Passing Math'] = school_result['Passing Math'] / school_result['Student ID']
school_result['% Passing Reading'] = school_result['Passing Reading'] / school_result['Student ID']
school_result['% Passing Overall'] = school_result['Passing Overall'] / school_result['Student ID']
# Make school name the index, rename columns, and show result
school_result.rename(columns={'name': 'Name', 'type': 'Type', 'budget': 'Budget','math_score': 'Math Score',
                              'reading_score': 'Reading Score'}, inplace=True)
school_result.set_index(keys='Name', inplace=True)
school_result
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
      <th>Type</th>
      <th>Budget</th>
      <th>Reading Score</th>
      <th>Passing Math</th>
      <th>Student ID</th>
      <th>Passing Overall</th>
      <th>Math Score</th>
      <th>Passing Reading</th>
      <th>Per Student Budget</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>1910635</td>
      <td>81.182722</td>
      <td>1916.0</td>
      <td>2917</td>
      <td>1561.0</td>
      <td>76.629414</td>
      <td>2372.0</td>
      <td>655.0</td>
      <td>0.656839</td>
      <td>0.813164</td>
      <td>0.535139</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>1884411</td>
      <td>81.158020</td>
      <td>1946.0</td>
      <td>2949</td>
      <td>1569.0</td>
      <td>76.711767</td>
      <td>2381.0</td>
      <td>639.0</td>
      <td>0.659885</td>
      <td>0.807392</td>
      <td>0.532045</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1056600</td>
      <td>83.725724</td>
      <td>1653.0</td>
      <td>1761</td>
      <td>1583.0</td>
      <td>83.359455</td>
      <td>1688.0</td>
      <td>600.0</td>
      <td>0.938671</td>
      <td>0.958546</td>
      <td>0.898921</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>3022020</td>
      <td>80.934412</td>
      <td>3094.0</td>
      <td>4635</td>
      <td>2481.0</td>
      <td>77.289752</td>
      <td>3748.0</td>
      <td>652.0</td>
      <td>0.667530</td>
      <td>0.808630</td>
      <td>0.535275</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>917500</td>
      <td>83.816757</td>
      <td>1371.0</td>
      <td>1468</td>
      <td>1330.0</td>
      <td>83.351499</td>
      <td>1426.0</td>
      <td>625.0</td>
      <td>0.933924</td>
      <td>0.971390</td>
      <td>0.905995</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>1319574</td>
      <td>83.989488</td>
      <td>2143.0</td>
      <td>2283</td>
      <td>2068.0</td>
      <td>83.274201</td>
      <td>2204.0</td>
      <td>578.0</td>
      <td>0.938677</td>
      <td>0.965396</td>
      <td>0.905826</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1081356</td>
      <td>83.975780</td>
      <td>1749.0</td>
      <td>1858</td>
      <td>1697.0</td>
      <td>83.061895</td>
      <td>1803.0</td>
      <td>582.0</td>
      <td>0.941335</td>
      <td>0.970398</td>
      <td>0.913348</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>3124928</td>
      <td>81.033963</td>
      <td>3318.0</td>
      <td>4976</td>
      <td>2719.0</td>
      <td>77.048432</td>
      <td>4077.0</td>
      <td>628.0</td>
      <td>0.666801</td>
      <td>0.819333</td>
      <td>0.546423</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>248087</td>
      <td>83.814988</td>
      <td>395.0</td>
      <td>427</td>
      <td>381.0</td>
      <td>83.803279</td>
      <td>411.0</td>
      <td>581.0</td>
      <td>0.925059</td>
      <td>0.962529</td>
      <td>0.892272</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>585858</td>
      <td>84.044699</td>
      <td>910.0</td>
      <td>962</td>
      <td>871.0</td>
      <td>83.839917</td>
      <td>923.0</td>
      <td>609.0</td>
      <td>0.945946</td>
      <td>0.959459</td>
      <td>0.905405</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1049400</td>
      <td>83.955000</td>
      <td>1680.0</td>
      <td>1800</td>
      <td>1626.0</td>
      <td>83.682222</td>
      <td>1739.0</td>
      <td>583.0</td>
      <td>0.933333</td>
      <td>0.966111</td>
      <td>0.903333</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>2547363</td>
      <td>80.744686</td>
      <td>2654.0</td>
      <td>3999</td>
      <td>2119.0</td>
      <td>76.842711</td>
      <td>3208.0</td>
      <td>637.0</td>
      <td>0.663666</td>
      <td>0.802201</td>
      <td>0.529882</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>3094650</td>
      <td>80.966394</td>
      <td>3145.0</td>
      <td>4761</td>
      <td>2549.0</td>
      <td>77.072464</td>
      <td>3867.0</td>
      <td>650.0</td>
      <td>0.660576</td>
      <td>0.812224</td>
      <td>0.535392</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>1763916</td>
      <td>80.746258</td>
      <td>1871.0</td>
      <td>2739</td>
      <td>1487.0</td>
      <td>77.102592</td>
      <td>2172.0</td>
      <td>644.0</td>
      <td>0.683096</td>
      <td>0.792990</td>
      <td>0.542899</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1043130</td>
      <td>83.848930</td>
      <td>1525.0</td>
      <td>1635</td>
      <td>1487.0</td>
      <td>83.418349</td>
      <td>1591.0</td>
      <td>638.0</td>
      <td>0.932722</td>
      <td>0.973089</td>
      <td>0.909480</td>
    </tr>
  </tbody>
</table>
</div>



## Top Performing Schools (By Passing Rate)


```python
# sort by passing overall descending to get top performing
top_performing = school_result.sort_values(by='% Passing Overall', ascending=False).head()
top_performing
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
      <th>Type</th>
      <th>Budget</th>
      <th>Reading Score</th>
      <th>Passing Math</th>
      <th>Student ID</th>
      <th>Passing Overall</th>
      <th>Math Score</th>
      <th>Passing Reading</th>
      <th>Per Student Budget</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
      <th>School Size</th>
      <th>Budget per Student Range</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1081356</td>
      <td>83.975780</td>
      <td>1749.0</td>
      <td>1858</td>
      <td>1697.0</td>
      <td>83.061895</td>
      <td>1803.0</td>
      <td>582.0</td>
      <td>0.941335</td>
      <td>0.970398</td>
      <td>0.913348</td>
      <td>Medium (1000 - 2000)</td>
      <td>&lt;$585</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1043130</td>
      <td>83.848930</td>
      <td>1525.0</td>
      <td>1635</td>
      <td>1487.0</td>
      <td>83.418349</td>
      <td>1591.0</td>
      <td>638.0</td>
      <td>0.932722</td>
      <td>0.973089</td>
      <td>0.909480</td>
      <td>Medium (1000 - 2000)</td>
      <td>$615-645</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>917500</td>
      <td>83.816757</td>
      <td>1371.0</td>
      <td>1468</td>
      <td>1330.0</td>
      <td>83.351499</td>
      <td>1426.0</td>
      <td>625.0</td>
      <td>0.933924</td>
      <td>0.971390</td>
      <td>0.905995</td>
      <td>Medium (1000 - 2000)</td>
      <td>$615-645</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>1319574</td>
      <td>83.989488</td>
      <td>2143.0</td>
      <td>2283</td>
      <td>2068.0</td>
      <td>83.274201</td>
      <td>2204.0</td>
      <td>578.0</td>
      <td>0.938677</td>
      <td>0.965396</td>
      <td>0.905826</td>
      <td>Large (2000 - 5000)</td>
      <td>&lt;$585</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>585858</td>
      <td>84.044699</td>
      <td>910.0</td>
      <td>962</td>
      <td>871.0</td>
      <td>83.839917</td>
      <td>923.0</td>
      <td>609.0</td>
      <td>0.945946</td>
      <td>0.959459</td>
      <td>0.905405</td>
      <td>Small (&lt; 1000)</td>
      <td>$585-615</td>
    </tr>
  </tbody>
</table>
</div>



## Bottom Performing Schools (By Passing Rate)


```python
# sort by passing overall ascending to get bottom performing
bottom_performing = school_result.sort_values(by='% Passing Overall').head()
bottom_performing
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
      <th>Type</th>
      <th>Budget</th>
      <th>Reading Score</th>
      <th>Passing Math</th>
      <th>Student ID</th>
      <th>Passing Overall</th>
      <th>Math Score</th>
      <th>Passing Reading</th>
      <th>Per Student Budget</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>2547363</td>
      <td>80.744686</td>
      <td>2654.0</td>
      <td>3999</td>
      <td>2119.0</td>
      <td>76.842711</td>
      <td>3208.0</td>
      <td>637.0</td>
      <td>0.663666</td>
      <td>0.802201</td>
      <td>0.529882</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>1884411</td>
      <td>81.158020</td>
      <td>1946.0</td>
      <td>2949</td>
      <td>1569.0</td>
      <td>76.711767</td>
      <td>2381.0</td>
      <td>639.0</td>
      <td>0.659885</td>
      <td>0.807392</td>
      <td>0.532045</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>1910635</td>
      <td>81.182722</td>
      <td>1916.0</td>
      <td>2917</td>
      <td>1561.0</td>
      <td>76.629414</td>
      <td>2372.0</td>
      <td>655.0</td>
      <td>0.656839</td>
      <td>0.813164</td>
      <td>0.535139</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>3022020</td>
      <td>80.934412</td>
      <td>3094.0</td>
      <td>4635</td>
      <td>2481.0</td>
      <td>77.289752</td>
      <td>3748.0</td>
      <td>652.0</td>
      <td>0.667530</td>
      <td>0.808630</td>
      <td>0.535275</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>3094650</td>
      <td>80.966394</td>
      <td>3145.0</td>
      <td>4761</td>
      <td>2549.0</td>
      <td>77.072464</td>
      <td>3867.0</td>
      <td>650.0</td>
      <td>0.660576</td>
      <td>0.812224</td>
      <td>0.535392</td>
    </tr>
  </tbody>
</table>
</div>



## Math Scores by Grade


```python
# Pivot to get passing math by school name and grade
pd.pivot_table(school_students, index='School Name', columns='Grade', values='Passing Math')
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
      <th>Grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>0.663438</td>
      <td>0.684253</td>
      <td>0.642996</td>
      <td>0.671468</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>0.939914</td>
      <td>0.923237</td>
      <td>0.950262</td>
      <td>0.952652</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>0.665793</td>
      <td>0.653032</td>
      <td>0.685990</td>
      <td>0.641355</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>0.693503</td>
      <td>0.687405</td>
      <td>0.654917</td>
      <td>0.689076</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>0.940887</td>
      <td>0.941828</td>
      <td>0.928082</td>
      <td>0.924205</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>0.667482</td>
      <td>0.668199</td>
      <td>0.667377</td>
      <td>0.667149</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>0.929825</td>
      <td>0.912621</td>
      <td>0.951807</td>
      <td>0.913386</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>0.634941</td>
      <td>0.647712</td>
      <td>0.661538</td>
      <td>0.681280</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>0.651182</td>
      <td>0.669449</td>
      <td>0.650641</td>
      <td>0.667857</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>0.944000</td>
      <td>0.960938</td>
      <td>0.950276</td>
      <td>0.930909</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>0.658654</td>
      <td>0.654076</td>
      <td>0.681876</td>
      <td>0.664089</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>0.933036</td>
      <td>0.949029</td>
      <td>0.940701</td>
      <td>0.933962</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>0.940618</td>
      <td>0.925301</td>
      <td>0.928994</td>
      <td>0.934924</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>0.942244</td>
      <td>0.936561</td>
      <td>0.944072</td>
      <td>0.933439</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>0.932099</td>
      <td>0.926437</td>
      <td>0.932249</td>
      <td>0.941176</td>
    </tr>
  </tbody>
</table>
</div>



## Reading Score by Grade 


```python
# Pivot to get passing reading by school name and grade
pd.pivot_table(school_students, index='School Name', columns='Grade', values='Passing Reading')
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
      <th>Grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>School Name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>0.835351</td>
      <td>0.805755</td>
      <td>0.813230</td>
      <td>0.821674</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>0.974249</td>
      <td>0.970954</td>
      <td>0.968586</td>
      <td>0.967803</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>0.812582</td>
      <td>0.781382</td>
      <td>0.819646</td>
      <td>0.815421</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>0.790960</td>
      <td>0.799697</td>
      <td>0.782931</td>
      <td>0.795918</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>0.972906</td>
      <td>0.975069</td>
      <td>0.979452</td>
      <td>0.960880</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>0.806846</td>
      <td>0.817096</td>
      <td>0.797441</td>
      <td>0.811143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>0.964912</td>
      <td>0.961165</td>
      <td>0.987952</td>
      <td>0.944882</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>0.821382</td>
      <td>0.805825</td>
      <td>0.811966</td>
      <td>0.812796</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>0.814996</td>
      <td>0.796327</td>
      <td>0.819444</td>
      <td>0.818571</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>0.956000</td>
      <td>0.949219</td>
      <td>0.944751</td>
      <td>0.981818</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>0.802885</td>
      <td>0.808151</td>
      <td>0.792142</td>
      <td>0.803265</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>0.937500</td>
      <td>0.973301</td>
      <td>0.954178</td>
      <td>0.967925</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>0.976247</td>
      <td>0.971084</td>
      <td>0.961538</td>
      <td>0.980477</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>0.962046</td>
      <td>0.966611</td>
      <td>0.964206</td>
      <td>0.968304</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>0.969136</td>
      <td>0.972414</td>
      <td>0.964770</td>
      <td>0.958824</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Spending


```python
# create new column based on Per Student Budget bins
bins = [0, 585, 615, 645, 675]
labels = ['<$585', '$585-615', '$615-645', '$645-675']
spending_series = pd.cut(school_result['Per Student Budget'], bins, labels=labels)
school_result['Budget per Student Range'] = spending_series
budget_groups = school_result.groupby(by='Budget per Student Range')
budget_groups['Math Score', 'Reading Score', '% Passing Math', '% Passing Reading', '% Passing Overall'].mean()
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
      <th>Math Score</th>
      <th>Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Budget per Student Range</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>0.934601</td>
      <td>0.966109</td>
      <td>0.903695</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>0.942309</td>
      <td>0.959003</td>
      <td>0.902163</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>79.079225</td>
      <td>81.891436</td>
      <td>0.756682</td>
      <td>0.861066</td>
      <td>0.661121</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>76.997210</td>
      <td>81.027843</td>
      <td>0.661648</td>
      <td>0.811340</td>
      <td>0.535269</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Size


```python
# Create new column based on school size bins
bins = [0, 1000, 2000, 5000]
names = ['Small (< 1000)', 'Medium (1000 - 2000)', 'Large (2000 - 5000)']

size_series = pd.cut(school_result['Student ID'], bins, labels=names)
school_result['School Size'] = size_series
size_groups = school_result.groupby(by='School Size')
size_groups['Math Score', 'Reading Score', '% Passing Math', '% Passing Reading', '% Passing Overall'].mean()
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
      <th>Math Score</th>
      <th>Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt; 1000)</th>
      <td>83.821598</td>
      <td>83.929843</td>
      <td>0.935502</td>
      <td>0.960994</td>
      <td>0.898839</td>
    </tr>
    <tr>
      <th>Medium (1000 - 2000)</th>
      <td>83.374684</td>
      <td>83.864438</td>
      <td>0.935997</td>
      <td>0.967907</td>
      <td>0.906215</td>
    </tr>
    <tr>
      <th>Large (2000 - 5000)</th>
      <td>77.746417</td>
      <td>81.344493</td>
      <td>0.699634</td>
      <td>0.827666</td>
      <td>0.582860</td>
    </tr>
  </tbody>
</table>
</div>



## Scores by School Type


```python
# Simple group by on Type (Charter/District)
size_groups = school_result.groupby(by='Type')
size_groups['Math Score', 'Reading Score', '% Passing Math', '% Passing Reading', '% Passing Overall'].mean()
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
      <th>Math Score</th>
      <th>Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>0.936208</td>
      <td>0.965865</td>
      <td>0.904322</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>0.665485</td>
      <td>0.807991</td>
      <td>0.536722</td>
    </tr>
  </tbody>
</table>
</div>


