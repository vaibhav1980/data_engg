## Introduction

We've gone over Data Acquisition as of now, so we know how to <i>get</i> our data. But once you have the data, it might not be in the best shape. You might have scraped a bunch of data from a website, but need it in the form of a dataframe to work with it in an easier manner. This process is called data preparation - preparing your data in a format that's easiest to form with.

### Overview

<b> Data Acquisition: </b> Reading and writing with a variety of file formats and databases. <br>
<b> Preparation: </b> Cleaning, munging, combining, normalizing, reshaping, slicing and dicing, and transforming data for analysis. <br>
<b> Transformation: </b> Applying mathematical and statistical operations to groups of data sets to derive new data sets. For example, aggregating a large table by group variables. <br>
<b> Modeling and computation: </b> Connecting your data to statistical models, machine learning algorithms, or other computational tools <br>
<b> Presentation: </b> Creating interactive or static graphical visualizations or textual summaries <br>

### Glossary

Here is some common terminology that we'll encounter throughout the workshop:

<b> Munging/Wrangling: </b> This refers to the overall process of manipulating unstructured or messy data into a structured or clean form. 


## 2.0 Pandas

Pandas allows us to deal with data in a way that us humans can understand it - with labelled columns and indexes. It allows us to effortlessly import data from files such as CSVs, allows us to quickly apply complex transformations and filters to our data and much more. Along with Numpy and Matplotlib, it helps create a really strong base for data exploration and analysis in Python. 

``` python
import pandas as pd 
from pandas import Series, DataFrame
```

### Series

A Series is a one-dimensional array-like object containing an array of data (of any NumPy data type) and an associated array of data labels, called its index. The simplest Series is formed from only an array of data:

``` python
obj = Series([4, 7, -5, 3])
```

Often it will be desirable to create a Series with an index identifying each data point:

``` python
obj2 = Series([4, 7, -5, 3], index=['d', 'b', 'a', 'c'])
```

You can also take a dictionary and convert it to a Series:
``` python
sdata = {'Ohio': 35000, 'Texas': 71000, 'Oregon': 16000, 'Utah': 5000}
obj3 = Series(sdata)
```

### DataFrames

A DataFrame represents a tabular, spreadsheet-like data structure containing an ordered collection of columns, each of which can be a different value type (numeric, string, boolean, etc.).

There are numerous ways to construct a DataFrame, though one of the most common is from a dict of equal-length lists or NumPy arrays:

``` python
data = {'state': ['Ohio', 'Ohio', 'Ohio', 'Nevada', 'Nevada'], 'year': [2000, 2001, 2002, 2001, 2002], 'pop': [1.5, 1.7, 3.6, 2.4, 2.9]}
```

Then we take this and convert it to a DataFrame:
``` python
frame = DataFrame(data)
```
This gets us:
```
   pop   state  year
0  1.5    Ohio  2000
1  1.7    Ohio  2001
2  3.6    Ohio  2002
3  2.4  Nevada  2001
4  2.9  Nevada  2002
```
You can also specify the sequence of columns by:

``` python
DataFrame(data, columns=['year', 'state', 'pop'])
```

### Apply

Lets's generate a random dictionary:

``` python
frame = DataFrame(np.random.randn(4, 3), columns=list('bde'), index=['Utah', 'Ohio', 'Texas', 'Oregon'])
```

With this, we can apply a function on a DataFrame:
``` python
np.abs(frame)
```

We can also apply functions with the `apply()` method:

``` python
f = lambda x: x.max() - x.min()
frame.apply(f)
```

#### Sorting

To sort lexicographically by row or column index, use the sort_index method, which returns a new, sorted object:

``` python
frame.sort_index()
```

Naive Bayes works on Bayes Theorem of probability to predict the class of a given data point. Naive Bayes is extremely fast compared to other classification algorithms and works with an assumption of independence among predictors. 

The Naive Bayes model is easy to build and particularly useful for very large data sets. Along with simplicity, Naive Bayes is known to outperform even highly sophisticated classification methods.


#### Indexing

Given the previous dataframe we created, if we try to access data by its index, we get multiple results, like this: 

``` python
result1.ix[0]
```
```
  First Name Last Name                   Major
0     Lesley   Cordero                     NaN
0     Martin     Perez  Mechanical Engineering
```
This is because the indices weren't result when we did the merge. With a simple parameter change, we can fix this, however:

``` python
names = pd.concat([d1, d3],axis=0,ignore_index=True)
```
Notice the `ignore_index` being set to `True`.

### Challenge

Recall Bayes Theorem, which provides a way of calculating the posterior probability. Its formula is as follows:

![alt text](https://github.com/ByteAcademyCo/stats-programmers/blob/master/bayes.png?raw=true "Logo Title Text 1")

Let's go through an example of how the Naive Bayes Algorithm works using `pandas`. We'll go through a classification problem that determines whether a sports team will play or not based on the weather. 

Let's load the module data:

``` python
import pandas as pd
f1 = pd.read_csv("./weather.csv")
```

#### Frequency Table

The first actual step of this process is converting the dataset into a frequency table. Using the `groupby()` function, we get the frequencies:

``` python
df = f1.groupby(['Weather','Play']).size()
```

Now let's split the frequencies by weather and yes/no. Let's start with the three weather frequencies:

``` python
df2 = f1.groupby('Weather').count()
```

Now let's get the frequencies of yes and no:

``` python
df1 = f1.groupby('Play').count()
```

#### Likelihood Table


Next, you would create a likelihood table by finding the probabilites of each weather condition and yes/no. This will require that we add a new column that takes the play frequency and divides it by the total data occurances. 


``` python
df1['Likelihood'] = df1['Weather']/len(f1)
df2['Likelihood'] = df2['Play']/len(f1)
```

This gets us a dataframe that looks like:

```
          Play  Likelihood
Weather                   
Overcast     4    0.285714
Rainy        5    0.357143
Sunny        5    0.357143
```

Now, we're able to use the Naive Bayesian equation to calculate the posterior probability for each class. The highest posterior probability is the outcome of prediction.

#### Calculation

So now we need a question. Let's propose the following: "Players will play if the weather is sunny. Is this true?"

From this question, we can construct Bayes Theorem. So what's our P(A|B)? P(Yes|Sunny), which gives us:

P(Yes|Sunny) = (P(Sunny|Yes)*P(Yes))/P(Sunny)

Based off the likelihood tables we created, we just grab P(Sunny) and P(Yes). 

``` python
ps = df2['Likelihood']['Sunny']
py = df1['Likelihood']['Yes']
```

That leaves us with P(Sunny|Yes). This is the probability that the weather is sunny given that the players played that day. In `df`, we see that the total number of `yes` days under `sunny` is 3. We take this number and divide it by the total number of `yes` days, which we can get from `df`. 

``` python
psy = df['Sunny']['Yes']/df1['Weather']['Yes']
```

Now, we just have to plug these variables into bayes theorem: 

``` python
p = (psy*py)/ps
```

And we get:

```
0.59999999999999998
```

That means the answer to our original question is yes!

## dplyr

Similar to pandas, dplyr allows us to transform and summarize tabular data with rows and columns. It contains a set of functions that perform common data manipulation operations like filtering rows, selecting specific columns, re-ordering rows, adding new columns, and summarizing data.

First we begin by loading in the needed packages:

``` R
library(dplyr)
library(downloader)
```

Using the data available in [this](https://github.com/lesley2958/data-science-r/blob/master/msleep_ggplot2.csv) repo, we''ll load the data into R:

``` R
url <- "https://raw.githubusercontent.com/genomicsclass/dagdata/master/inst/extdata/msleep_ggplot2.csv" 
filename <- "msleep_ggplot2.csv" 
download(url, filename) # downloads file into local
msleep <- read.csv("msleep_ggplot2.csv") # reads the file
head(msleep) # outputs 6 rows
```

Which gets us the following output:
```
                        name      genus  vore        order conservation
1                    Cheetah   Acinonyx carni    Carnivora           lc
2                 Owl monkey      Aotus  omni     Primates         <NA>
3            Mountain beaver Aplodontia herbi     Rodentia           nt
4 Greater short-tailed shrew    Blarina  omni Soricomorpha           lc
5                        Cow        Bos herbi Artiodactyla domesticated
6           Three-toed sloth   Bradypus herbi       Pilosa         <NA>
  sleep_total sleep_rem sleep_cycle awake brainwt  bodywt
1        12.1        NA          NA  11.9      NA  50.000
2        17.0       1.8          NA   7.0 0.01550   0.480
3        14.4       2.4          NA   9.6      NA   1.350
4        14.9       2.3   0.1333333   9.1 0.00029   0.019
5         4.0       0.7   0.6666667  20.0 0.42300 600.000
6        14.4       2.2   0.7666667   9.6      NA   3.850
```

### Select

To demonstrate how the `select()` method works, we select the name and sleep_total columns.

``` R
sleepData <- select(msleep, name, sleep_total)
head(sleepData)
```

To select all the columns except a specific column, you can use the subtraction sign:

``` R
head(select(msleep, -name))
```

You can also select a range of columns with a colon:

``` R
head(select(msleep, name:order))
```

### Filter

Using the `filter()` method in dplyr we can select rows that meet a certain criterion, such as in the following:

``` R
filter(msleep, sleep_total >= 16)
```
There, we filter out the animals whose sleep total is less than 16 hours. If you want to expand the criteria, you can: 

```R
filter(msleep, sleep_total >= 16, bodywt >= 1)
```

### Pipe Operator

dplyr imports the pipe operator from another package, `magrittr`. This operator allows you to pipe the output from one function to the input of another function. Instead of nesting functions. 

Recall: 

``` R
head(select(msleep, name, sleep_total))
```

Instead, you can rewrite this as: 

``` R
msleep %>% 
    select(name, sleep_total) %>% 
    head
```

This function becomes particularly useful later on. 


### Arrange

To re-order rows by a particular column, you can list the name of the column you want to arrange the rows by: 

``` R
msleep %>% arrange(order) %>% head
```

This outputs the data in alphabetical order:

```
      name     genus  vore        order conservation sleep_total sleep_rem
1   Tenrec    Tenrec  omni Afrosoricida         <NA>        15.6       2.3
2      Cow       Bos herbi Artiodactyla domesticated         4.0       0.7
3 Roe deer Capreolus herbi Artiodactyla           lc         3.0        NA
4     Goat     Capri herbi Artiodactyla           lc         5.3       0.6
5  Giraffe   Giraffa herbi Artiodactyla           cd         1.9       0.4
6    Sheep      Ovis herbi Artiodactyla domesticated         3.8       0.6
  sleep_cycle awake brainwt  bodywt
1          NA   8.4  0.0026   0.900
2   0.6666667  20.0  0.4230 600.000
3          NA  21.0  0.0982  14.800
4          NA  18.7  0.1150  33.500
5          NA  22.1      NA 899.995
6          NA  20.2  0.1750  55.500
```

### Mutate

The `mutate()` function allows us to add new columns to the data frame. In this example, we'll create a new column called `rem_proportion` which is the ratio of rem sleep to total amount of sleep.

``` R
msleep %>% 
    mutate(rem_proportion = sleep_rem / sleep_total) %>%
    head
```

This results in: 

```
                        name      genus  vore        order conservation
1                    Cheetah   Acinonyx carni    Carnivora           lc
2                 Owl monkey      Aotus  omni     Primates         <NA>
3            Mountain beaver Aplodontia herbi     Rodentia           nt
4 Greater short-tailed shrew    Blarina  omni Soricomorpha           lc
5                        Cow        Bos herbi Artiodactyla domesticated
6           Three-toed sloth   Bradypus herbi       Pilosa         <NA>
  sleep_total sleep_rem sleep_cycle awake brainwt  bodywt rem_proportion
1        12.1        NA          NA  11.9      NA  50.000             NA
2        17.0       1.8          NA   7.0 0.01550   0.480      0.1058824
3        14.4       2.4          NA   9.6      NA   1.350      0.1666667
4        14.9       2.3   0.1333333   9.1 0.00029   0.019      0.1543624
5         4.0       0.7   0.6666667  20.0 0.42300 600.000      0.1750000
6        14.4       2.2   0.7666667   9.6      NA   3.850      0.1527778
```

### Summaries

The `summarise()` function creates summary statistics by inputting the column name as a parameter. As an example, to compute the average number of hours of sleep, apply the mean() function to the column `sleep_total` and call the summary value `avg_sleep`. The syntax for this is as follows:

``` R
msleep %>% 
    summarise(avg_sleep = mean(sleep_total))
```
And we learn that the average sleep is `10.43` hours:
```
  avg_sleep
1  10.43373
```

The `summarise()` function can take multiple parameters to output multiple summary statistics:
``` R
msleep %>% 
    summarise(avg_sleep = mean(sleep_total), 
              min_sleep = min(sleep_total),
              max_sleep = max(sleep_total),
              total = n())
```

Notice we now have 4 summary statistics!
```
  avg_sleep min_sleep max_sleep total
1  10.43373       1.9      19.9    83
```

### Group_by

Each order of animals contains multiple rows. If we want to get summary statistics for each order type, we can combine the summarize statistics from the previous section with the `group_by()` function. 

``` R
msleep %>% 
    group_by(order) %>%
    summarise(avg_sleep = mean(sleep_total), 
              min_sleep = min(sleep_total), 
              max_sleep = max(sleep_total),
              total = n())
```

As an output, we get the set of orders with their respective summary statistics:
```
             order avg_sleep min_sleep max_sleep total
            <fctr>     <dbl>     <dbl>     <dbl> <int>
1     Afrosoricida 15.600000      15.6      15.6     1
2     Artiodactyla  4.516667       1.9       9.1     6
3        Carnivora 10.116667       3.5      15.8    12
4          Cetacea  4.500000       2.7       5.6     3
5       Chiroptera 19.800000      19.7      19.9     2
6        Cingulata 17.750000      17.4      18.1     2
7  Didelphimorphia 18.700000      18.0      19.4     2
8    Diprotodontia 12.400000      11.1      13.7     2
9   Erinaceomorpha 10.200000      10.1      10.3     2
10      Hyracoidea  5.666667       5.3       6.3     3
11      Lagomorpha  8.400000       8.4       8.4     1
12     Monotremata  8.600000       8.6       8.6     1
13  Perissodactyla  3.466667       2.9       4.4     3
14          Pilosa 14.400000      14.4      14.4     1
15        Primates 10.500000       8.0      17.0    12
16     Proboscidea  3.600000       3.3       3.9     2
17        Rodentia 12.468182       7.0      16.6    22
18      Scandentia  8.900000       8.9       8.9     1
19    Soricomorpha 11.100000       8.4      14.9     5
```

## Extracting Zipfiles

Oftentimes, you'll have to download a large number of files. They might come in the form of zipfiles. Instead of manually unzipping them, you can use Python to extract these files for you, using the `os` and `zipfile` modules. 

### OS Module

The `os` module provides us a portable way of using operating system dependent functionality. We'll begin exploring it's capabilities now:

``` python
import os
```

With the `os.getcwd()` method, we can get the current directory we're in. This is particularly useful when working with a large number of files in a certain folder. In this case, we'll use `os` to work with the zipfiles:

``` python
cwd = os.getcwd()
dir_path  = os.path.join(cwd, 'Example')
```

With `os`, you can check to see if a certain directory exists. Here, if it doesn't exist, we create that folder, which we can do with the `os.makedirs()` function:

``` python
if not os.path.exists(dir_path):
    os.makedirs(dir_path)
```

Lastly, as we do with the `ls` command on the terminal, we can use `os.listdir()` to get the contents of whatever path we provide as an argument.

``` python
os.listdir(cwd)
```

From there, we can move onto working with zipfiles!


### ZipFile

The `zipfile` module is a powerful module that allows us to extract files from a zipped folder. First, we import the needed modules and assign the name of the zipfile we'll be extracting to a variable:
``` python
import zipfile
import os
zip_name = 'example.zip'
```

Next, we get the current directory path and join it with the zipfile name to get its exact path:

``` python
cwd = os.getcwd()
zip_path = os.path.join(cwd, zip_name)
```

Next, we use the ZipFile class to change the file to a Zipfile object. With this object, we use the `ZipFile.extract()` function to extract all the contents:
``` python
with zipfile.ZipFile(zip_path, 'r') as z:
	z.extractall(cwd)
```

And lastly, we can see what's in the zipfile with: 

``` python
os.listdir(dir_path)
```

## Data Merging

If you encounter two different datasets that contain the same type of information, you might consider merging them for your analyses. This is yet another functionality built into `pandas`. 

Let's go through an example containing student data. `d1` contains 5 of the samples and `d2` contains 2 of them: 

``` python
d1 = pd.read_csv("./names_original.csv")
d2 = pd.read_csv("./names_add.csv")
```

### Concatenation 

Instead of working with two separate datasets, it's much easier to simply merge, so we do this with the `concat()` function:

``` python
result = pd.concat([d1,d2])
```

Now, you might be asking what will happen if one of the datasets has more columns than other - will they still be allowed to merge? Let's try this example with another dataset:

``` python
d3 = pd.read_csv("./names_extra.csv")
```

If we use the same `concat()` function, we get:

``` python
result1 = pd.concat([d1, d3])
```

Notice the `NaN` values - these are undefined values indicating there wasn't any data to be displayed. `pandas` will simply fill in the missing data for each sample where it's unavailable:  

```
  First Name  Last Name                   Major
0     Lesley    Cordero                     NaN
1       Ojas      Sathe                     NaN
2      Helen       Chen                     NaN
3        Eli   Epperson                     NaN
4      Jacob  Greenberg                     NaN
0     Martin      Perez  Mechanical Engineering
1      Menna    Elsayed               Sociology
```

### Merging

Now, how do we merge two datasets with differing columns? Well, let's take a look of our datasets:

``` python
h1 = pd.read_csv("./housing.csv")
h2 = pd.read_csv("./dorms.csv")
```

With the `merge()` function in pandas, we can specify which column to merge on and what kind of join to specify. By default merge does an 'inner' join, but here we set it to a left join:

``` python
house = pd.merge(h1, h2, on="Dorm", how="left")
```
This gets us: 
``` 
          Dorm            Name Street    Cost
0  East Campus      Helen Chen  116th  11,000
1     Broadway   Danielle Jing  114th    9000
2      Shapiro    Craig Rhodes  115th    9500
3         Watt  Lesley Cordero  113th   10500
4  East Campus    Martin Perez  116th  11,000
5     Broadway   Menna Elsayed  114th    9000
6      Wallach   Will Essilfie  114th    9500
```
