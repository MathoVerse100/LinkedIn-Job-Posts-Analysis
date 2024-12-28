Loading the datasets:

```python
df = pd.read_csv("./datasets/linkedin_job_posts_insights.csv")
world = pd.read_csv("./datasets/world_cities.csv")
```

```python
df.head()
```
| Job Title                                      | Company Name    | Location                             | Hiring Status         | Date                | Seniority Level   | Job Function                           | Employment Type   | Industry                                                                              |
|:----------------------------------------------|:----------------|:-------------------------------------|:----------------------|:--------------------|:------------------|:---------------------------------------|:------------------|:-------------------------------------------------------------------------------------|
| Store Business Manager - DAVID JONES CHERMSIDE | M.J. Bale       | Brisbane, Queensland, Australia      | Be an early applicant | 2023-04-13 00:00:00 | Not Applicable    | Sales and Business Development         | Full-time         | Government Administration                                                             |
| Senior Machine Learning Engineer               | Redwolf + Rosch | Adelaide, South Australia, Australia | Be an early applicant | 2023-04-25 00:00:00 | Mid-Senior level  | Engineering and Information Technology | Part-time         | Internet Publishing                                                                   |
| Senior Data Scientist                          | Bupa            | Melbourne, Victoria, Australia       | Be an early applicant | 2023-04-29 00:00:00 | Entry level       | Engineering and Information Technology | Full-time         | Technology, Information and Internet                                                  |
| Solution Architect                             | Xybion Digital  | Chennai, Tamil Nadu, India           | Be an early applicant | 2023-01-26 00:00:00 | Mid-Senior level  | Engineering and Information Technology | Full-time         | IT Services and IT Consulting, Software Development, and Computer Networking Products |
| Lead Data Scientist                            | Spinny          | Gurugram, Haryana, India             | Actively Hiring       | 2023-02-24 00:00:00 | Mid-Senior level  | Information Technology                 | Full-time         | Advertising Services                                                                  |

```python
df.info()
```
```python
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 27972 entries, 0 to 27971
Data columns (total 9 columns):
 #   Column           Non-Null Count  Dtype         
---  ------           --------------  -----         
 0   job_title        27972 non-null  object        
 1   company_name     27972 non-null  object        
 2   location         27972 non-null  object        
 3   hiring_status    27972 non-null  object        
 4   date             27972 non-null  datetime64[ns]
 5   seniority_level  27972 non-null  object        
 6   job_function     27972 non-null  object        
 7   employment_type  27972 non-null  object        
 8   industry         27972 non-null  object
```

I will remove all null values from the dataset. Since the number of nulls is relatively small and the data is primarily nominal, this approach should be acceptable.

```python
df.dropna(inplace=True)
df.reset_index(drop=True, inplace=True)
```

Now we need to clean the data itself.

```python
df["seniority_level"].unique()

df["employment_type"].unique()
```

```python
array(['Not Applicable', 'Mid-Senior level', 'Entry level',
       '\n            Mid-Senior level\n          ', 'Associate',
       '\n            Not Applicable\n          ',
       '\n            Associate\n          ',
       '\n            Entry level\n          ',
       '\n            Director\n          ', 'Executive', 'Director',
       '\n            Internship\n          ',
       '\n            Executive\n          ', 'Internship',
       '\n            Senior level\n          '], dtype=object)

array(['Full-time', 'Part-time', '\n          Full-time\n        ',
       '\n        \n          Employment type\n        \n        \n          Full-time\n        \n      ',
       'Contract', '\n          Contract\n        ',
       '\n        \n          Employment type\n        \n        \n          Part-time\n        \n      ',
       '\n        \n          Employment type\n        \n        \n          Internship\n        \n      ',
       '\n          Part-time\n        ', 'Internship',
       '\n          Internship\n        ', 'Volunteer', 'Temporary',
       'Other', '\n          Temporary\n        ',
       '\n          Other\n        ',
       '\n        \n          Employment type\n        \n        \n          Contract\n        \n      ',
       '\n        \n          Employment type\n        \n        \n          Other\n        \n      '],
      dtype=object)
```

It appears that line breaks in the data have been improperly formatted as strings. To address this, we need to remove unnecessary characters such as "Employment type\n", 
"\n", and any leading or trailing whitespace from the seniority_level and employment_type columns. To ensure consistency, we should apply this cleanup process to all 
columns in the dataset.

```python
def line_break_cleaner(word):
    return word.strip("\n ")

def employment_type_cleaner(employment_type):
    return employment_type.lstrip("Employment type\n")

for df_column in df.columns:
    try:
        df[df_column] = df[df_column].apply(func=line_break_cleaner)
    except AttributeError:
        continue

df["employment_type"] = df["employment_type"].apply(employment_type_cleaner)

print(df["seniority_level"].unique())
print(df["employment_type"].unique())
```

```python
['Not Applicable' 'Mid-Senior level' 'Entry level' 'Associate' 'Director'
 'Executive' 'Internship' 'Senior level']

['Full-time' 'Part-time' 'Contract' 'Internship' 'Volunteer' 'Temporary'
 'Other']
```

```python
df[df.duplicated() == True]
```

| index   | job_title   | company_name   | location   | hiring_status   | date   | seniority_level   | job_function   | employment_type   | industry   |\n|---------|-------------|----------------|------------|-----------------|--------|-------------------|----------------|-------------------|------------|

There are no duplicates in the dataset, and unnecessary line breaks and spaces have been successfully addressed. 

Upon reviewing the job_title column, I noticed that many entries contain nonsensical characters at the beginning and end, such as numbers, hashes, dashes without context, 
unclosed or extraneous quotation marks, and similar issues. We must get rid of these inconsistencies by stripping them from the data.

Furthermore, whilst inspecting the location column, US states have been abbreviated by ISO 3166-2 codes, and I did not find any other states from other countries 
abbreviated the same way. So I just added "United States" to any string in the column with such codes.

```python
def job_title_cleaner(job):
    return job.strip('#1234567890 -"')

df["job_title"] = df["job_title"].apply(func=job_title_cleaner)
df = df.drop(columns="index")

def united_states(sentence):
    return sentence.__add__(", United States") if sentence[-2:] == sentence[-2:].upper() else sentence

df["location"] = df["location"].apply(func=united_states)
```
Ok now let's view the world_cities dataset:

```python
world.head()
```
| City      | City ASCII   |     Lat |      Lng | Country   | ISO2   | ISO3   | Admin Name   | Capital   |   Population |
|:----------|:-------------|--------:|---------:|:----------|:-------|:-------|:-------------|:----------|-------------:|
| Tokyo     | Tokyo        | 35.6897 | 139.692  | Japan     | JP     | JPN    | Tōkyō        | primary   |   3.7732e+07 |
| Jakarta   | Jakarta      | -6.175  | 106.828  | Indonesia | ID     | IDN    | Jakarta      | primary   |   3.3756e+07 |
| Delhi     | Delhi        | 28.61   |  77.23   | India     | IN     | IND    | Delhi        | admin     |   3.2226e+07 |
| Guangzhou | Guangzhou    | 23.13   | 113.26   | China     | CN     | CHN    | Guangdong    | admin     |   2.694e+07  |
| Mumbai    | Mumbai       | 19.0761 |  72.8775 | India     | IN     | IND    | Mahārāshtra  | admin     |   2.4973e+07 |



The city column contains city names, some of which are in their native non-English form or use alphabets other than the English alphabet, whereas the city_ascii column 
provides the city names in English. Therefore, I will remove the city column and rename the city_ascii column to city. Additionally, I will retain only the country 
and iso3 columns, the latter of which will be renamed to iso_alpha3. This makes it much easier to plot maps.

```python
world = world.drop(columns=["city", "iso2", "admin_name", "capital", "lat", "lng", "population"]).rename(columns={'city_ascii':'city', 'iso3':'iso_alpha3'})

for column in world.columns:
    try:
        world[column].apply(func=line_break_cleaner)
    except AttributeError:
        continue

world.info()
```
```python
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 44691 entries, 0 to 44690
Data columns (total 3 columns):
 #   Column      Non-Null Count  Dtype 
---  ------      --------------  ----- 
 0   city        44691 non-null  object
 1   country     44691 non-null  object
 2   iso_alpha3  44691 non-null  object
dtypes: object(3)
```

The data is good now. As an extra, I would like to identify the country of origin for each job posting. Using the world_cities dataset and leveraging regular expressions, 
I will create a dictionary containing dataframes, each segmented by country. Each dataframe includes the count of job postings grouped by date.

```python
world_countries = sorted(world["country"].unique().tolist())
jobs_by_country = dict()
for country in world_countries:
    pattern = "|".join([rf"\b{x}\b" for x in world["city"][world["country"] == country].unique()])
    pattern_we_dont_want = "|".join([rf"\b{x}\b" for x in world_countries if x != country])
    df_country = df[df["location"].str.match(pattern)][~df["location"].str.contains(pattern_we_dont_want, regex=True)]
    df_country = df_country.groupby("date").size().reset_index(name="count").sort_values(by="date",ascending=True)
    df_country["country"] = country
    jobs_by_country[country] = df_country
```

The data can now be aggregated, analyzed and visualized properly using Tableau, which is my preferred visualization tool.
