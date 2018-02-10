
# Step1 Data_engineering 
The climate data for Hawaii is provided through two CSV files. Start by using Python and Pandas to inspect the content of these files and clean the data.

Create a Jupyter Notebook file called data_engineering.ipynb and use this to complete all of your Data Engineering tasks.

Use Pandas to read in the measurement and station CSV files as DataFrames.

Inspect the data for NaNs and missing values. You must decide what to do with this data.

Save your cleaned CSV files with the prefix clean_.


```python
import pandas as pd
import os
import matplotlib.pyplot as plt
import seaborn as sns
 
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Float, Date
from sqlalchemy.orm import Session
from sqlalchemy.ext.automap import automap_base
import pymysql
pymysql.install_as_MySQLdb()

 
```

## Reading in both the data files 'hawaii_measurements.csv' and 'hawaii_stations.csv'


```python
measurements_df = pd.read_csv('Resources/hawaii_measurements.csv')
measurements_df.head()
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
      <th>station</th>
      <th>date</th>
      <th>prcp</th>
      <th>tobs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>USC00519397</td>
      <td>2010-01-01</td>
      <td>0.08</td>
      <td>65</td>
    </tr>
    <tr>
      <th>1</th>
      <td>USC00519397</td>
      <td>2010-01-02</td>
      <td>0.00</td>
      <td>63</td>
    </tr>
    <tr>
      <th>2</th>
      <td>USC00519397</td>
      <td>2010-01-03</td>
      <td>0.00</td>
      <td>74</td>
    </tr>
    <tr>
      <th>3</th>
      <td>USC00519397</td>
      <td>2010-01-04</td>
      <td>0.00</td>
      <td>76</td>
    </tr>
    <tr>
      <th>4</th>
      <td>USC00519397</td>
      <td>2010-01-06</td>
      <td>NaN</td>
      <td>73</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(measurements_df) # lenght 19550
```




    19550




```python
measurements_df['station'].value_counts()
```




    USC00519281    2772
    USC00519397    2724
    USC00513117    2709
    USC00519523    2669
    USC00516128    2612
    USC00514830    2202
    USC00511918    1979
    USC00517948    1372
    USC00518838     511
    Name: station, dtype: int64




```python
#measurements_df.groupby(['station','prcp']).size()
```


```python
measurements_df.dtypes

```




    station     object
    date        object
    prcp       float64
    tobs         int64
    dtype: object




```python
# Inspect the data for Nan and missing data
measurements_df.isnull() # check to see if any data is Nan in the series
measurements_df.isnull().values.any()# just check overall to see if any missing  
measurements_df.isnull().sum().sum() # total number of all the missing values
measurements_df.isnull().sum() # all the columns which have missing data
```




    station       0
    date          0
    prcp       1447
    tobs          0
    dtype: int64



### Cleaning up the data by dropping all NaN's in the dataframe('hawaii_measurements.csv')


```python
# fill missing data Nan with zero 
#fillna_measurements_df = measurements_df.fillna(0)
#fillna_measurements_df.head()
```


```python
# drop all the NaN data
fillna_measurements_df = measurements_df.dropna(how = 'any')
fillna_measurements_df.count()

 
```




    station    18103
    date       18103
    prcp       18103
    tobs       18103
    dtype: int64




```python
fillna_measurements_df.to_csv('Resources/clean_hawaii_measurements.csv', index=True)
```


```python
#cat 'Resources/clean_hawaii_measurements.csv'
```


```python
pd.Series(list(fillna_measurements_df.index))
```




    0            0
    1            1
    2            2
    3            3
    4            5
    5            6
    6            7
    7            8
    8            9
    9           10
    10          11
    11          12
    12          13
    13          14
    14          15
    15          16
    16          17
    17          18
    18          19
    19          20
    20          21
    21          22
    22          23
    23          24
    24          25
    25          27
    26          28
    27          30
    28          31
    29          32
             ...  
    18073    19513
    18074    19514
    18075    19515
    18076    19516
    18077    19517
    18078    19518
    18079    19519
    18080    19520
    18081    19521
    18082    19522
    18083    19523
    18084    19524
    18085    19525
    18086    19526
    18087    19527
    18088    19529
    18089    19530
    18090    19533
    18091    19534
    18092    19535
    18093    19536
    18094    19538
    18095    19540
    18096    19541
    18097    19542
    18098    19543
    18099    19545
    18100    19547
    18101    19548
    18102    19549
    Length: 18103, dtype: int64




```python
fillna_measurements_df.index
```




    Int64Index([    0,     1,     2,     3,     5,     6,     7,     8,     9,
                   10,
                ...
                19536, 19538, 19540, 19541, 19542, 19543, 19545, 19547, 19548,
                19549],
               dtype='int64', length=18103)




```python
fillna_measurements_df.to_csv('Resources/clean_hawaii_measurements.csv', index=True)
new_measurements_df = pd.read_csv('Resources/clean_hawaii_measurements.csv', index_col=0)
new_measurements_df['id'] = pd.Series(list(fillna_measurements_df.index))
new_measurements_df.head()
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
      <th>station</th>
      <th>date</th>
      <th>prcp</th>
      <th>tobs</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>USC00519397</td>
      <td>2010-01-01</td>
      <td>0.08</td>
      <td>65</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>USC00519397</td>
      <td>2010-01-02</td>
      <td>0.00</td>
      <td>63</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>USC00519397</td>
      <td>2010-01-03</td>
      <td>0.00</td>
      <td>74</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>USC00519397</td>
      <td>2010-01-04</td>
      <td>0.00</td>
      <td>76</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>USC00519397</td>
      <td>2010-01-07</td>
      <td>0.06</td>
      <td>70</td>
      <td>6.0</td>
    </tr>
  </tbody>
</table>
</div>



### No need to cleanup any data as there are no missing nor Nan's in the dataframe ('hawaii_stations.csv')


```python
pd.Series(list(new_measurements_df.index))
```




    0            0
    1            1
    2            2
    3            3
    4            5
    5            6
    6            7
    7            8
    8            9
    9           10
    10          11
    11          12
    12          13
    13          14
    14          15
    15          16
    16          17
    17          18
    18          19
    19          20
    20          21
    21          22
    22          23
    23          24
    24          25
    25          27
    26          28
    27          30
    28          31
    29          32
             ...  
    18073    19513
    18074    19514
    18075    19515
    18076    19516
    18077    19517
    18078    19518
    18079    19519
    18080    19520
    18081    19521
    18082    19522
    18083    19523
    18084    19524
    18085    19525
    18086    19526
    18087    19527
    18088    19529
    18089    19530
    18090    19533
    18091    19534
    18092    19535
    18093    19536
    18094    19538
    18095    19540
    18096    19541
    18097    19542
    18098    19543
    18099    19545
    18100    19547
    18101    19548
    18102    19549
    Length: 18103, dtype: int64




```python
stations_df = pd.read_csv('Resources/hawaii_stations.csv')
stations_df.to_csv('Resources/clean_hawaii_stations.csv', index=True)
new_stations_df = pd.read_csv('Resources/clean_hawaii_stations.csv' ,index_col=0)
new_stations_df['id'] = pd.Series(list(stations_df.index))
new_stations_df.head()
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
      <th>station</th>
      <th>name</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>elevation</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>USC00519397</td>
      <td>WAIKIKI 717.2, HI US</td>
      <td>21.2716</td>
      <td>-157.8168</td>
      <td>3.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>USC00513117</td>
      <td>KANEOHE 838.1, HI US</td>
      <td>21.4234</td>
      <td>-157.8015</td>
      <td>14.6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>USC00514830</td>
      <td>KUALOA RANCH HEADQUARTERS 886.9, HI US</td>
      <td>21.5213</td>
      <td>-157.8374</td>
      <td>7.0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>USC00517948</td>
      <td>PEARL CITY, HI US</td>
      <td>21.3934</td>
      <td>-157.9751</td>
      <td>11.9</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>USC00518838</td>
      <td>UPPER WAHIAWA 874.3, HI US</td>
      <td>21.4992</td>
      <td>-158.0111</td>
      <td>306.6</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
l = []
for row in new_measurements_df.iterrows():
    l.append(row)
    
l[0][1]
```




    station    USC00519397
    date        2010-01-01
    prcp              0.08
    tobs                65
    id                   0
    Name: 0, dtype: object




```python
l = []
for row in new_stations_df.iterrows():
    l.append(row)
    
l[0][1]
```




    station               USC00519397
    name         WAIKIKI 717.2, HI US
    latitude                  21.2716
    longitude                -157.817
    elevation                       3
    id                              0
    Name: 0, dtype: object



# Step 2 - Database Engineering
 
Use SQLAlchemy to model your table schemas and create a sqlite database for your tables. You will need one table for measurements and one for stations.

Create a Jupyter Notebook called database_engineering.ipynb and use this to complete all of your Database Engineering work.

Use Pandas to read your cleaned measurements and stations CSV data.

Use the engine and connection string to create a database called hawaii.sqlite.

Use declarative_base and create ORM classes for each table.

You will need a class for Measurement and for Station.
Make sure to define your primary keys.
Once you have your ORM classes defined, create the tables in the database using create_all.


```python
# Import SQL Alchemy
from sqlalchemy import create_engine

# Import and establish Base for which classes will be constructed 
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()

# Import modules to declare columns and column data types
from sqlalchemy import Column, Integer, String, Float
 

```


```python
#Delete the sqlite db before rerunning my code all over again!
#!rm hawaii.sqlite
!rm dropna_hawaii.sqlite
```


```python
# Define a Measurements class and/or model of Measurement
 
class Measurement(Base):
    __tablename__ = "measurements" 
    
    id = Column(Integer, primary_key=True)
    station = Column(String)
    date = Column(String) 
    prcp = Column(Float)  
    tobs = Column(Integer) 
    
 
```


```python
# Define a Stations class and/or model of Stations
 
class Station(Base):
    __tablename__ = "stations" 
    
    id = Column(Integer, primary_key=True)
    station = Column(String)
    name = Column(String(255)) 
    latitude = Column(Float)  
    longitude = Column(Float) 
    elevation = Column(Float)
 
```


```python
# Create Database Connection
engine = create_engine("sqlite:///dropna_hawaii.sqlite")

# fillna(0) decided to compare both the data results, with dropping, and filling 0.
#engine = create_engine("sqlite:///hawaii.sqlite")
```


```python
Base.metadata.create_all(engine)
```


```python
# To push the objects made and query the server we use a Session object
from sqlalchemy.orm import Session
session = Session(bind=engine)
```


```python
session.commit()
```


```python
#Write records stored in a DataFrame to a SQL database.
#with engine.connect() as conn,conn.begin():
new_measurements_df.to_sql(con=engine, index_label='id', name='measurements',if_exists='append',index=False) 
new_stations_df.to_sql(con=engine, index_label='id', name='stations',if_exists='append',index=False)

```


```python
list(map(lambda x: x.id, session.query(Measurement).all()))
```




    [0,
     1,
     2,
     3,
     6,
     7,
     8,
     9,
     10,
     11,
     12,
     13,
     14,
     15,
     16,
     17,
     18,
     19,
     20,
     21,
     22,
     23,
     24,
     25,
     27,
     30,
     31,
     33,
     34,
     35,
     36,
     37,
     38,
     39,
     40,
     41,
     42,
     44,
     45,
     46,
     48,
     49,
     50,
     51,
     52,
     53,
     54,
     55,
     56,
     57,
     58,
     59,
     60,
     62,
     63,
     64,
     65,
     67,
     68,
     69,
     70,
     71,
     73,
     74,
     75,
     76,
     77,
     79,
     80,
     81,
     82,
     83,
     84,
     85,
     86,
     87,
     88,
     89,
     90,
     91,
     92,
     93,
     94,
     95,
     96,
     97,
     98,
     99,
     100,
     101,
     102,
     103,
     104,
     105,
     106,
     107,
     108,
     109,
     110,
     111,
     112,
     113,
     114,
     115,
     116,
     117,
     118,
     119,
     120,
     121,
     123,
     124,
     125,
     126,
     127,
     128,
     130,
     131,
     132,
     133,
     134,
     135,
     136,
     137,
     138,
     139,
     140,
     141,
     142,
     143,
     144,
     145,
     146,
     147,
     148,
     149,
     150,
     151,
     152,
     153,
     154,
     155,
     156,
     157,
     158,
     159,
     160,
     161,
     162,
     163,
     164,
     165,
     166,
     167,
     168,
     169,
     170,
     171,
     172,
     173,
     174,
     175,
     177,
     178,
     179,
     180,
     181,
     182,
     183,
     185,
     186,
     187,
     188,
     189,
     190,
     191,
     192,
     193,
     194,
     195,
     196,
     197,
     198,
     199,
     200,
     201,
     202,
     203,
     204,
     205,
     206,
     207,
     208,
     209,
     210,
     211,
     212,
     213,
     214,
     215,
     216,
     217,
     218,
     219,
     220,
     221,
     222,
     223,
     224,
     225,
     226,
     227,
     228,
     229,
     230,
     231,
     232,
     233,
     234,
     235,
     236,
     237,
     238,
     239,
     240,
     241,
     242,
     243,
     244,
     245,
     246,
     247,
     248,
     249,
     250,
     251,
     252,
     253,
     254,
     255,
     256,
     257,
     258,
     259,
     260,
     261,
     262,
     263,
     264,
     265,
     266,
     267,
     268,
     269,
     270,
     271,
     272,
     273,
     274,
     275,
     276,
     277,
     278,
     279,
     280,
     281,
     283,
     284,
     285,
     286,
     287,
     288,
     289,
     290,
     292,
     293,
     295,
     296,
     297,
     298,
     299,
     300,
     301,
     302,
     303,
     305,
     306,
     307,
     308,
     309,
     310,
     311,
     312,
     313,
     314,
     315,
     316,
     317,
     318,
     319,
     320,
     321,
     322,
     323,
     325,
     326,
     327,
     328,
     329,
     330,
     331,
     332,
     333,
     334,
     336,
     337,
     338,
     339,
     340,
     342,
     343,
     344,
     345,
     346,
     347,
     348,
     349,
     350,
     351,
     352,
     354,
     355,
     356,
     357,
     358,
     359,
     360,
     361,
     362,
     363,
     364,
     365,
     366,
     367,
     368,
     370,
     371,
     372,
     373,
     374,
     375,
     376,
     377,
     378,
     379,
     380,
     381,
     383,
     384,
     385,
     386,
     387,
     388,
     389,
     391,
     392,
     393,
     394,
     395,
     396,
     397,
     398,
     399,
     400,
     401,
     402,
     403,
     405,
     406,
     407,
     408,
     409,
     410,
     411,
     412,
     413,
     414,
     415,
     416,
     417,
     418,
     419,
     420,
     421,
     422,
     423,
     424,
     425,
     426,
     427,
     428,
     429,
     430,
     431,
     432,
     433,
     434,
     435,
     436,
     437,
     438,
     439,
     440,
     441,
     442,
     443,
     444,
     445,
     446,
     447,
     448,
     449,
     450,
     451,
     452,
     453,
     454,
     455,
     456,
     457,
     458,
     459,
     460,
     461,
     462,
     463,
     464,
     465,
     466,
     467,
     468,
     469,
     470,
     471,
     472,
     473,
     474,
     475,
     476,
     477,
     478,
     479,
     480,
     481,
     482,
     483,
     484,
     485,
     486,
     487,
     488,
     489,
     491,
     492,
     493,
     494,
     495,
     496,
     497,
     498,
     499,
     500,
     501,
     502,
     503,
     504,
     506,
     507,
     508,
     509,
     510,
     511,
     512,
     513,
     514,
     515,
     516,
     517,
     518,
     519,
     520,
     521,
     522,
     523,
     524,
     525,
     526,
     527,
     528,
     529,
     530,
     531,
     532,
     533,
     534,
     535,
     536,
     537,
     538,
     539,
     540,
     541,
     542,
     543,
     544,
     545,
     546,
     547,
     548,
     549,
     550,
     551,
     552,
     553,
     554,
     555,
     556,
     557,
     558,
     559,
     560,
     561,
     562,
     563,
     564,
     565,
     566,
     567,
     568,
     569,
     570,
     571,
     572,
     573,
     574,
     575,
     576,
     577,
     578,
     579,
     580,
     581,
     582,
     583,
     584,
     585,
     587,
     588,
     589,
     590,
     591,
     592,
     593,
     594,
     595,
     596,
     597,
     598,
     599,
     600,
     601,
     603,
     604,
     605,
     606,
     607,
     608,
     609,
     610,
     611,
     612,
     613,
     614,
     615,
     616,
     617,
     618,
     619,
     620,
     621,
     622,
     623,
     624,
     625,
     626,
     627,
     628,
     629,
     630,
     631,
     632,
     633,
     634,
     635,
     636,
     637,
     638,
     639,
     640,
     641,
     642,
     643,
     644,
     645,
     646,
     647,
     648,
     649,
     650,
     651,
     652,
     653,
     654,
     655,
     656,
     657,
     658,
     659,
     660,
     661,
     662,
     663,
     664,
     665,
     666,
     667,
     668,
     669,
     670,
     671,
     672,
     673,
     674,
     675,
     676,
     677,
     678,
     679,
     680,
     681,
     682,
     683,
     684,
     685,
     686,
     687,
     688,
     689,
     690,
     691,
     692,
     693,
     694,
     695,
     696,
     697,
     698,
     699,
     700,
     701,
     702,
     703,
     704,
     705,
     706,
     707,
     708,
     709,
     710,
     711,
     712,
     713,
     714,
     715,
     716,
     717,
     718,
     719,
     720,
     721,
     722,
     723,
     724,
     725,
     726,
     727,
     728,
     729,
     730,
     731,
     732,
     733,
     734,
     735,
     736,
     737,
     738,
     739,
     740,
     741,
     742,
     743,
     744,
     745,
     746,
     747,
     748,
     749,
     750,
     751,
     752,
     753,
     754,
     755,
     756,
     757,
     758,
     759,
     760,
     761,
     762,
     763,
     764,
     765,
     766,
     767,
     768,
     769,
     770,
     771,
     772,
     773,
     774,
     775,
     776,
     777,
     778,
     779,
     780,
     781,
     782,
     783,
     784,
     785,
     786,
     787,
     788,
     789,
     790,
     791,
     792,
     793,
     794,
     795,
     796,
     797,
     798,
     799,
     800,
     801,
     802,
     803,
     804,
     805,
     806,
     807,
     808,
     809,
     810,
     811,
     812,
     813,
     814,
     815,
     816,
     817,
     818,
     819,
     820,
     821,
     822,
     823,
     824,
     825,
     826,
     827,
     828,
     829,
     832,
     833,
     834,
     835,
     836,
     837,
     838,
     839,
     840,
     841,
     842,
     843,
     844,
     845,
     846,
     847,
     850,
     851,
     852,
     853,
     854,
     855,
     856,
     857,
     858,
     859,
     860,
     862,
     863,
     864,
     865,
     866,
     867,
     868,
     869,
     870,
     871,
     872,
     873,
     874,
     875,
     876,
     877,
     878,
     879,
     881,
     882,
     883,
     884,
     885,
     886,
     887,
     888,
     889,
     890,
     891,
     892,
     893,
     894,
     895,
     896,
     897,
     898,
     899,
     900,
     903,
     904,
     905,
     906,
     907,
     908,
     909,
     910,
     911,
     912,
     913,
     914,
     915,
     916,
     917,
     918,
     919,
     920,
     921,
     924,
     925,
     926,
     927,
     928,
     929,
     930,
     931,
     932,
     933,
     934,
     935,
     936,
     937,
     938,
     939,
     940,
     941,
     942,
     943,
     944,
     945,
     946,
     947,
     948,
     949,
     950,
     951,
     952,
     953,
     954,
     955,
     956,
     957,
     958,
     959,
     960,
     961,
     962,
     963,
     964,
     965,
     966,
     967,
     968,
     969,
     970,
     971,
     972,
     973,
     974,
     975,
     976,
     977,
     978,
     979,
     980,
     981,
     982,
     983,
     984,
     985,
     986,
     987,
     988,
     989,
     990,
     991,
     992,
     993,
     994,
     995,
     996,
     997,
     998,
     999,
     1000,
     1001,
     1002,
     1003,
     1004,
     1005,
     1006,
     1007,
     1008,
     1009,
     1010,
     1013,
     1014,
     1015,
     1016,
     1017,
     1018,
     1019,
     1020,
     1021,
     1022,
     1023,
     1024,
     1025,
     1026,
     1027,
     1028,
     1029,
     1030,
     1031,
     1032,
     1033,
     1036,
     1037,
     1038,
     1039,
     1040,
     1041,
     1042,
     1043,
     1044,
     1047,
     ...]




```python
list(map(lambda x: x.id, session.query(Station).all()))
```




    [0, 1, 2, 3, 4, 5, 6, 7, 8]




```python
session.query(Measurement).all()
```




    [<__main__.Measurement at 0x10f108978>,
     <__main__.Measurement at 0x10f108d30>,
     <__main__.Measurement at 0x10f108898>,
     <__main__.Measurement at 0x10f108f60>,
     <__main__.Measurement at 0x10f111198>,
     <__main__.Measurement at 0x10f111358>,
     <__main__.Measurement at 0x10f111550>,
     <__main__.Measurement at 0x10f111048>,
     <__main__.Measurement at 0x10f111828>,
     <__main__.Measurement at 0x10f1116a0>,
     <__main__.Measurement at 0x10f111588>,
     <__main__.Measurement at 0x10f1114a8>,
     <__main__.Measurement at 0x10f111390>,
     <__main__.Measurement at 0x10f1112b0>,
     <__main__.Measurement at 0x10f111208>,
     <__main__.Measurement at 0x10f111160>,
     <__main__.Measurement at 0x10f1110b8>,
     <__main__.Measurement at 0x10f111898>,
     <__main__.Measurement at 0x10f111978>,
     <__main__.Measurement at 0x10f111a20>,
     <__main__.Measurement at 0x10f111ac8>,
     <__main__.Measurement at 0x10f111b70>,
     <__main__.Measurement at 0x10f111c18>,
     <__main__.Measurement at 0x10f111cc0>,
     <__main__.Measurement at 0x10f111d68>,
     <__main__.Measurement at 0x10f111e10>,
     <__main__.Measurement at 0x10f111eb8>,
     <__main__.Measurement at 0x10f111f60>,
     <__main__.Measurement at 0x10f119048>,
     <__main__.Measurement at 0x10f1190f0>,
     <__main__.Measurement at 0x10f119198>,
     <__main__.Measurement at 0x10f119240>,
     <__main__.Measurement at 0x10f1192e8>,
     <__main__.Measurement at 0x10f119390>,
     <__main__.Measurement at 0x10f119438>,
     <__main__.Measurement at 0x10f1194e0>,
     <__main__.Measurement at 0x10f119588>,
     <__main__.Measurement at 0x10f119630>,
     <__main__.Measurement at 0x10f1196d8>,
     <__main__.Measurement at 0x10f119780>,
     <__main__.Measurement at 0x10f119828>,
     <__main__.Measurement at 0x10f1198d0>,
     <__main__.Measurement at 0x10f119978>,
     <__main__.Measurement at 0x10f119a20>,
     <__main__.Measurement at 0x10f119ac8>,
     <__main__.Measurement at 0x10f119b70>,
     <__main__.Measurement at 0x10f119c18>,
     <__main__.Measurement at 0x10f119cc0>,
     <__main__.Measurement at 0x10f119d68>,
     <__main__.Measurement at 0x10f119e10>,
     <__main__.Measurement at 0x10f119eb8>,
     <__main__.Measurement at 0x10f119f60>,
     <__main__.Measurement at 0x10f123048>,
     <__main__.Measurement at 0x10f1230f0>,
     <__main__.Measurement at 0x10f123198>,
     <__main__.Measurement at 0x10f123240>,
     <__main__.Measurement at 0x10f1232e8>,
     <__main__.Measurement at 0x10f123390>,
     <__main__.Measurement at 0x10f123438>,
     <__main__.Measurement at 0x10f1234e0>,
     <__main__.Measurement at 0x10f123588>,
     <__main__.Measurement at 0x10f123630>,
     <__main__.Measurement at 0x10f1236d8>,
     <__main__.Measurement at 0x10f123780>,
     <__main__.Measurement at 0x10f123828>,
     <__main__.Measurement at 0x10f1238d0>,
     <__main__.Measurement at 0x10f123978>,
     <__main__.Measurement at 0x10f123a20>,
     <__main__.Measurement at 0x10f123ac8>,
     <__main__.Measurement at 0x10f123b70>,
     <__main__.Measurement at 0x10f123c18>,
     <__main__.Measurement at 0x10f123cc0>,
     <__main__.Measurement at 0x10f123d68>,
     <__main__.Measurement at 0x10f123e10>,
     <__main__.Measurement at 0x10f123eb8>,
     <__main__.Measurement at 0x10f123f60>,
     <__main__.Measurement at 0x10f12b048>,
     <__main__.Measurement at 0x10f12b0f0>,
     <__main__.Measurement at 0x10f12b198>,
     <__main__.Measurement at 0x10f12b240>,
     <__main__.Measurement at 0x10f12b2e8>,
     <__main__.Measurement at 0x10f12b390>,
     <__main__.Measurement at 0x10f12b438>,
     <__main__.Measurement at 0x10f12b4e0>,
     <__main__.Measurement at 0x10f12b588>,
     <__main__.Measurement at 0x10f12b630>,
     <__main__.Measurement at 0x10f12b6d8>,
     <__main__.Measurement at 0x10f12b780>,
     <__main__.Measurement at 0x10f12b828>,
     <__main__.Measurement at 0x10f12b8d0>,
     <__main__.Measurement at 0x10f12b978>,
     <__main__.Measurement at 0x10f12ba20>,
     <__main__.Measurement at 0x10f12bac8>,
     <__main__.Measurement at 0x10f12bb70>,
     <__main__.Measurement at 0x10f12bc18>,
     <__main__.Measurement at 0x10f12bcc0>,
     <__main__.Measurement at 0x10f12bd68>,
     <__main__.Measurement at 0x10f12be10>,
     <__main__.Measurement at 0x10f12beb8>,
     <__main__.Measurement at 0x10f12bf60>,
     <__main__.Measurement at 0x10f134048>,
     <__main__.Measurement at 0x10f1340f0>,
     <__main__.Measurement at 0x10f134198>,
     <__main__.Measurement at 0x10f134240>,
     <__main__.Measurement at 0x10f1342e8>,
     <__main__.Measurement at 0x10f134390>,
     <__main__.Measurement at 0x10f134438>,
     <__main__.Measurement at 0x10f1344e0>,
     <__main__.Measurement at 0x10f134588>,
     <__main__.Measurement at 0x10f134630>,
     <__main__.Measurement at 0x10f1346d8>,
     <__main__.Measurement at 0x10f134780>,
     <__main__.Measurement at 0x10f134828>,
     <__main__.Measurement at 0x10f1348d0>,
     <__main__.Measurement at 0x10f134978>,
     <__main__.Measurement at 0x10f134a20>,
     <__main__.Measurement at 0x10f134ac8>,
     <__main__.Measurement at 0x10f134b70>,
     <__main__.Measurement at 0x10f134c18>,
     <__main__.Measurement at 0x10f134cc0>,
     <__main__.Measurement at 0x10f134d68>,
     <__main__.Measurement at 0x10f134e10>,
     <__main__.Measurement at 0x10f134eb8>,
     <__main__.Measurement at 0x10f134f60>,
     <__main__.Measurement at 0x10f13c048>,
     <__main__.Measurement at 0x10f13c0f0>,
     <__main__.Measurement at 0x10f13c198>,
     <__main__.Measurement at 0x10f13c240>,
     <__main__.Measurement at 0x10f13c2e8>,
     <__main__.Measurement at 0x10f13c390>,
     <__main__.Measurement at 0x10f13c438>,
     <__main__.Measurement at 0x10f13c4e0>,
     <__main__.Measurement at 0x10f13c588>,
     <__main__.Measurement at 0x10f13c630>,
     <__main__.Measurement at 0x10f13c6d8>,
     <__main__.Measurement at 0x10f13c780>,
     <__main__.Measurement at 0x10f13c828>,
     <__main__.Measurement at 0x10f13c8d0>,
     <__main__.Measurement at 0x10f13c978>,
     <__main__.Measurement at 0x10f13ca20>,
     <__main__.Measurement at 0x10f13cac8>,
     <__main__.Measurement at 0x10f13cb70>,
     <__main__.Measurement at 0x10f13cc18>,
     <__main__.Measurement at 0x10f13ccc0>,
     <__main__.Measurement at 0x10f13cd68>,
     <__main__.Measurement at 0x10f13ce10>,
     <__main__.Measurement at 0x10f13ceb8>,
     <__main__.Measurement at 0x10f13cf60>,
     <__main__.Measurement at 0x10f145048>,
     <__main__.Measurement at 0x10f1450f0>,
     <__main__.Measurement at 0x10f145198>,
     <__main__.Measurement at 0x10f145240>,
     <__main__.Measurement at 0x10f1452e8>,
     <__main__.Measurement at 0x10f145390>,
     <__main__.Measurement at 0x10f145438>,
     <__main__.Measurement at 0x10f1454e0>,
     <__main__.Measurement at 0x10f145588>,
     <__main__.Measurement at 0x10f145630>,
     <__main__.Measurement at 0x10f1456d8>,
     <__main__.Measurement at 0x10f145780>,
     <__main__.Measurement at 0x10f145828>,
     <__main__.Measurement at 0x10f1458d0>,
     <__main__.Measurement at 0x10f145978>,
     <__main__.Measurement at 0x10f145a20>,
     <__main__.Measurement at 0x10f145ac8>,
     <__main__.Measurement at 0x10f145b70>,
     <__main__.Measurement at 0x10f145c18>,
     <__main__.Measurement at 0x10f145cc0>,
     <__main__.Measurement at 0x10f145d68>,
     <__main__.Measurement at 0x10f145e10>,
     <__main__.Measurement at 0x10f145eb8>,
     <__main__.Measurement at 0x10f145f60>,
     <__main__.Measurement at 0x10f14e048>,
     <__main__.Measurement at 0x10f14e0f0>,
     <__main__.Measurement at 0x10f14e198>,
     <__main__.Measurement at 0x10f14e240>,
     <__main__.Measurement at 0x10f14e2e8>,
     <__main__.Measurement at 0x10f14e390>,
     <__main__.Measurement at 0x10f14e438>,
     <__main__.Measurement at 0x10f14e4e0>,
     <__main__.Measurement at 0x10f14e588>,
     <__main__.Measurement at 0x10f14e630>,
     <__main__.Measurement at 0x10f14e6d8>,
     <__main__.Measurement at 0x10f14e780>,
     <__main__.Measurement at 0x10f14e828>,
     <__main__.Measurement at 0x10f14e8d0>,
     <__main__.Measurement at 0x10f14e978>,
     <__main__.Measurement at 0x10f14ea20>,
     <__main__.Measurement at 0x10f14eac8>,
     <__main__.Measurement at 0x10f14eb70>,
     <__main__.Measurement at 0x10f14ec18>,
     <__main__.Measurement at 0x10f14ecc0>,
     <__main__.Measurement at 0x10f14ed68>,
     <__main__.Measurement at 0x10f14ee10>,
     <__main__.Measurement at 0x10f14eeb8>,
     <__main__.Measurement at 0x10f14ef60>,
     <__main__.Measurement at 0x10f157048>,
     <__main__.Measurement at 0x10f1570f0>,
     <__main__.Measurement at 0x10f157198>,
     <__main__.Measurement at 0x10f157240>,
     <__main__.Measurement at 0x10f1572e8>,
     <__main__.Measurement at 0x10f157390>,
     <__main__.Measurement at 0x10f157438>,
     <__main__.Measurement at 0x10f1574e0>,
     <__main__.Measurement at 0x10f157588>,
     <__main__.Measurement at 0x10f157630>,
     <__main__.Measurement at 0x10f1576d8>,
     <__main__.Measurement at 0x10f157780>,
     <__main__.Measurement at 0x10f157828>,
     <__main__.Measurement at 0x10f1578d0>,
     <__main__.Measurement at 0x10f157978>,
     <__main__.Measurement at 0x10f157a20>,
     <__main__.Measurement at 0x10f157ac8>,
     <__main__.Measurement at 0x10f157b70>,
     <__main__.Measurement at 0x10f157c18>,
     <__main__.Measurement at 0x10f157cc0>,
     <__main__.Measurement at 0x10f157d68>,
     <__main__.Measurement at 0x10f157e10>,
     <__main__.Measurement at 0x10f157eb8>,
     <__main__.Measurement at 0x10f157f60>,
     <__main__.Measurement at 0x10f160048>,
     <__main__.Measurement at 0x10f1600f0>,
     <__main__.Measurement at 0x10f160198>,
     <__main__.Measurement at 0x10f160240>,
     <__main__.Measurement at 0x10f1602e8>,
     <__main__.Measurement at 0x10f160390>,
     <__main__.Measurement at 0x10f160438>,
     <__main__.Measurement at 0x10f1604e0>,
     <__main__.Measurement at 0x10f160588>,
     <__main__.Measurement at 0x10f160630>,
     <__main__.Measurement at 0x10f1606d8>,
     <__main__.Measurement at 0x10f160780>,
     <__main__.Measurement at 0x10f160828>,
     <__main__.Measurement at 0x10f1608d0>,
     <__main__.Measurement at 0x10f160978>,
     <__main__.Measurement at 0x10f160a20>,
     <__main__.Measurement at 0x10f160ac8>,
     <__main__.Measurement at 0x10f160b70>,
     <__main__.Measurement at 0x10f160c18>,
     <__main__.Measurement at 0x10f160cc0>,
     <__main__.Measurement at 0x10f160d68>,
     <__main__.Measurement at 0x10f160e10>,
     <__main__.Measurement at 0x10f160eb8>,
     <__main__.Measurement at 0x10f160f60>,
     <__main__.Measurement at 0x10f169048>,
     <__main__.Measurement at 0x10f1690f0>,
     <__main__.Measurement at 0x10f169198>,
     <__main__.Measurement at 0x10f169240>,
     <__main__.Measurement at 0x10f1692e8>,
     <__main__.Measurement at 0x10f169390>,
     <__main__.Measurement at 0x10f169438>,
     <__main__.Measurement at 0x10f1694e0>,
     <__main__.Measurement at 0x10f169588>,
     <__main__.Measurement at 0x10f169630>,
     <__main__.Measurement at 0x10f1696d8>,
     <__main__.Measurement at 0x10f169780>,
     <__main__.Measurement at 0x10f169828>,
     <__main__.Measurement at 0x10f1698d0>,
     <__main__.Measurement at 0x10f169978>,
     <__main__.Measurement at 0x10f169a20>,
     <__main__.Measurement at 0x10f169ac8>,
     <__main__.Measurement at 0x10f169b70>,
     <__main__.Measurement at 0x10f169c18>,
     <__main__.Measurement at 0x10f169cc0>,
     <__main__.Measurement at 0x10f169d68>,
     <__main__.Measurement at 0x10f169e10>,
     <__main__.Measurement at 0x10f169eb8>,
     <__main__.Measurement at 0x10f169f60>,
     <__main__.Measurement at 0x10f170048>,
     <__main__.Measurement at 0x10f1700f0>,
     <__main__.Measurement at 0x10f170198>,
     <__main__.Measurement at 0x10f170240>,
     <__main__.Measurement at 0x10f1702e8>,
     <__main__.Measurement at 0x10f170390>,
     <__main__.Measurement at 0x10f170438>,
     <__main__.Measurement at 0x10f1704e0>,
     <__main__.Measurement at 0x10f170588>,
     <__main__.Measurement at 0x10f170630>,
     <__main__.Measurement at 0x10f1706d8>,
     <__main__.Measurement at 0x10f170780>,
     <__main__.Measurement at 0x10f170828>,
     <__main__.Measurement at 0x10f1708d0>,
     <__main__.Measurement at 0x10f170978>,
     <__main__.Measurement at 0x10f170a20>,
     <__main__.Measurement at 0x10f170ac8>,
     <__main__.Measurement at 0x10f170b70>,
     <__main__.Measurement at 0x10f170c18>,
     <__main__.Measurement at 0x10f170cc0>,
     <__main__.Measurement at 0x10f170d68>,
     <__main__.Measurement at 0x10f170e10>,
     <__main__.Measurement at 0x10f170eb8>,
     <__main__.Measurement at 0x10f170f60>,
     <__main__.Measurement at 0x10f17a048>,
     <__main__.Measurement at 0x10f17a0f0>,
     <__main__.Measurement at 0x10f17a198>,
     <__main__.Measurement at 0x10f17a240>,
     <__main__.Measurement at 0x10f17a2e8>,
     <__main__.Measurement at 0x10f17a390>,
     <__main__.Measurement at 0x10f17a438>,
     <__main__.Measurement at 0x10f17a4e0>,
     <__main__.Measurement at 0x10f17a588>,
     <__main__.Measurement at 0x10f17a630>,
     <__main__.Measurement at 0x10f17a6d8>,
     <__main__.Measurement at 0x10f17a780>,
     <__main__.Measurement at 0x10f17a828>,
     <__main__.Measurement at 0x10f17a8d0>,
     <__main__.Measurement at 0x10f17a978>,
     <__main__.Measurement at 0x10f17aa20>,
     <__main__.Measurement at 0x10f17aac8>,
     <__main__.Measurement at 0x10f17ab70>,
     <__main__.Measurement at 0x10f17ac18>,
     <__main__.Measurement at 0x10f17acc0>,
     <__main__.Measurement at 0x10f17ad68>,
     <__main__.Measurement at 0x10f17ae10>,
     <__main__.Measurement at 0x10f17aeb8>,
     <__main__.Measurement at 0x10f17af60>,
     <__main__.Measurement at 0x10f183048>,
     <__main__.Measurement at 0x10f1830f0>,
     <__main__.Measurement at 0x10f183198>,
     <__main__.Measurement at 0x10f183240>,
     <__main__.Measurement at 0x10f1832e8>,
     <__main__.Measurement at 0x10f183390>,
     <__main__.Measurement at 0x10f183438>,
     <__main__.Measurement at 0x10f1834e0>,
     <__main__.Measurement at 0x10f183588>,
     <__main__.Measurement at 0x10f183630>,
     <__main__.Measurement at 0x10f1836d8>,
     <__main__.Measurement at 0x10f183780>,
     <__main__.Measurement at 0x10f183828>,
     <__main__.Measurement at 0x10f1838d0>,
     <__main__.Measurement at 0x10f183978>,
     <__main__.Measurement at 0x10f183a20>,
     <__main__.Measurement at 0x10f183ac8>,
     <__main__.Measurement at 0x10f183b70>,
     <__main__.Measurement at 0x10f183c18>,
     <__main__.Measurement at 0x10f183cc0>,
     <__main__.Measurement at 0x10f183d68>,
     <__main__.Measurement at 0x10f183e10>,
     <__main__.Measurement at 0x10f183eb8>,
     <__main__.Measurement at 0x10f183f60>,
     <__main__.Measurement at 0x10f18b048>,
     <__main__.Measurement at 0x10f18b0f0>,
     <__main__.Measurement at 0x10f18b198>,
     <__main__.Measurement at 0x10f18b240>,
     <__main__.Measurement at 0x10f18b2e8>,
     <__main__.Measurement at 0x10f18b390>,
     <__main__.Measurement at 0x10f18b438>,
     <__main__.Measurement at 0x10f18b4e0>,
     <__main__.Measurement at 0x10f18b588>,
     <__main__.Measurement at 0x10f18b630>,
     <__main__.Measurement at 0x10f18b6d8>,
     <__main__.Measurement at 0x10f18b780>,
     <__main__.Measurement at 0x10f18b828>,
     <__main__.Measurement at 0x10f18b8d0>,
     <__main__.Measurement at 0x10f18b978>,
     <__main__.Measurement at 0x10f18ba20>,
     <__main__.Measurement at 0x10f18bac8>,
     <__main__.Measurement at 0x10f18bb70>,
     <__main__.Measurement at 0x10f18bc18>,
     <__main__.Measurement at 0x10f18bcc0>,
     <__main__.Measurement at 0x10f18bd68>,
     <__main__.Measurement at 0x10f18be10>,
     <__main__.Measurement at 0x10f18beb8>,
     <__main__.Measurement at 0x10f18bf60>,
     <__main__.Measurement at 0x10f193048>,
     <__main__.Measurement at 0x10f1930f0>,
     <__main__.Measurement at 0x10f193198>,
     <__main__.Measurement at 0x10f193240>,
     <__main__.Measurement at 0x10f1932e8>,
     <__main__.Measurement at 0x10f193390>,
     <__main__.Measurement at 0x10f193438>,
     <__main__.Measurement at 0x10f1934e0>,
     <__main__.Measurement at 0x10f193588>,
     <__main__.Measurement at 0x10f193630>,
     <__main__.Measurement at 0x10f1936d8>,
     <__main__.Measurement at 0x10f193780>,
     <__main__.Measurement at 0x10f193828>,
     <__main__.Measurement at 0x10f1938d0>,
     <__main__.Measurement at 0x10f193978>,
     <__main__.Measurement at 0x10f193a20>,
     <__main__.Measurement at 0x10f193ac8>,
     <__main__.Measurement at 0x10f193b70>,
     <__main__.Measurement at 0x10f193c18>,
     <__main__.Measurement at 0x10f193cc0>,
     <__main__.Measurement at 0x10f193d68>,
     <__main__.Measurement at 0x10f193e10>,
     <__main__.Measurement at 0x10f193eb8>,
     <__main__.Measurement at 0x10f193f60>,
     <__main__.Measurement at 0x10f19e048>,
     <__main__.Measurement at 0x10f19e0f0>,
     <__main__.Measurement at 0x10f19e198>,
     <__main__.Measurement at 0x10f19e240>,
     <__main__.Measurement at 0x10f19e2e8>,
     <__main__.Measurement at 0x10f19e390>,
     <__main__.Measurement at 0x10f19e438>,
     <__main__.Measurement at 0x10f19e4e0>,
     <__main__.Measurement at 0x10f19e588>,
     <__main__.Measurement at 0x10f19e630>,
     <__main__.Measurement at 0x10f19e6d8>,
     <__main__.Measurement at 0x10f19e780>,
     <__main__.Measurement at 0x10f19e828>,
     <__main__.Measurement at 0x10f19e8d0>,
     <__main__.Measurement at 0x10f19e978>,
     <__main__.Measurement at 0x10f19ea20>,
     <__main__.Measurement at 0x10f19eac8>,
     <__main__.Measurement at 0x10f19eb70>,
     <__main__.Measurement at 0x10f19ec18>,
     <__main__.Measurement at 0x10f19ecc0>,
     <__main__.Measurement at 0x10f19ed68>,
     <__main__.Measurement at 0x10f19ee10>,
     <__main__.Measurement at 0x10f19eeb8>,
     <__main__.Measurement at 0x10f19ef60>,
     <__main__.Measurement at 0x10f1a5048>,
     <__main__.Measurement at 0x10f1a50f0>,
     <__main__.Measurement at 0x10f1a5198>,
     <__main__.Measurement at 0x10f1a5240>,
     <__main__.Measurement at 0x10f1a52e8>,
     <__main__.Measurement at 0x10f1a5390>,
     <__main__.Measurement at 0x10f1a5438>,
     <__main__.Measurement at 0x10f1a54e0>,
     <__main__.Measurement at 0x10f1a5588>,
     <__main__.Measurement at 0x10f1a5630>,
     <__main__.Measurement at 0x10f1a56d8>,
     <__main__.Measurement at 0x10f1a5780>,
     <__main__.Measurement at 0x10f1a5828>,
     <__main__.Measurement at 0x10f1a58d0>,
     <__main__.Measurement at 0x10f1a5978>,
     <__main__.Measurement at 0x10f1a5a20>,
     <__main__.Measurement at 0x10f1a5ac8>,
     <__main__.Measurement at 0x10f1a5b70>,
     <__main__.Measurement at 0x10f1a5c18>,
     <__main__.Measurement at 0x10f1a5cc0>,
     <__main__.Measurement at 0x10f1a5d68>,
     <__main__.Measurement at 0x10f1a5e10>,
     <__main__.Measurement at 0x10f1a5eb8>,
     <__main__.Measurement at 0x10f1a5f60>,
     <__main__.Measurement at 0x10cff3cc0>,
     <__main__.Measurement at 0x10cff3eb8>,
     <__main__.Measurement at 0x10cff3a20>,
     <__main__.Measurement at 0x10f100160>,
     <__main__.Measurement at 0x10f1002b0>,
     <__main__.Measurement at 0x10f1004a8>,
     <__main__.Measurement at 0x10f1006a0>,
     <__main__.Measurement at 0x10f100898>,
     <__main__.Measurement at 0x10f100a90>,
     <__main__.Measurement at 0x10f100c88>,
     <__main__.Measurement at 0x10f100e80>,
     <__main__.Measurement at 0x10f100dd8>,
     <__main__.Measurement at 0x10f100f28>,
     <__main__.Measurement at 0x10f100e10>,
     <__main__.Measurement at 0x10f100d30>,
     <__main__.Measurement at 0x10f100c18>,
     <__main__.Measurement at 0x10f100b38>,
     <__main__.Measurement at 0x10f100a20>,
     <__main__.Measurement at 0x10f100940>,
     <__main__.Measurement at 0x10f100828>,
     <__main__.Measurement at 0x10f100748>,
     <__main__.Measurement at 0x10f100630>,
     <__main__.Measurement at 0x10f100550>,
     <__main__.Measurement at 0x10f100438>,
     <__main__.Measurement at 0x10f100358>,
     <__main__.Measurement at 0x10f100240>,
     <__main__.Measurement at 0x10f100048>,
     <__main__.Measurement at 0x10f100fd0>,
     <__main__.Measurement at 0x10f1ad048>,
     <__main__.Measurement at 0x10f1ad0f0>,
     <__main__.Measurement at 0x10f1ad198>,
     <__main__.Measurement at 0x10f1ad240>,
     <__main__.Measurement at 0x10f1ad2e8>,
     <__main__.Measurement at 0x10f1ad390>,
     <__main__.Measurement at 0x10f1ad438>,
     <__main__.Measurement at 0x10f1ad4e0>,
     <__main__.Measurement at 0x10f1ad588>,
     <__main__.Measurement at 0x10f1ad630>,
     <__main__.Measurement at 0x10f1ad6d8>,
     <__main__.Measurement at 0x10f1ad780>,
     <__main__.Measurement at 0x10f1ad828>,
     <__main__.Measurement at 0x10f1ad8d0>,
     <__main__.Measurement at 0x10f1ad978>,
     <__main__.Measurement at 0x10f1ada20>,
     <__main__.Measurement at 0x10f1adac8>,
     <__main__.Measurement at 0x10f1adb70>,
     <__main__.Measurement at 0x10f1adc18>,
     <__main__.Measurement at 0x10f1adcc0>,
     <__main__.Measurement at 0x10f1add68>,
     <__main__.Measurement at 0x10f1ade10>,
     <__main__.Measurement at 0x10f1adeb8>,
     <__main__.Measurement at 0x10f1adf60>,
     <__main__.Measurement at 0x10f1b7048>,
     <__main__.Measurement at 0x10f1b70f0>,
     <__main__.Measurement at 0x10f1b7198>,
     <__main__.Measurement at 0x10f1b7240>,
     <__main__.Measurement at 0x10f1b72e8>,
     <__main__.Measurement at 0x10f1b7390>,
     <__main__.Measurement at 0x10f1b7438>,
     <__main__.Measurement at 0x10f1b74e0>,
     <__main__.Measurement at 0x10f1b7588>,
     <__main__.Measurement at 0x10f1b7630>,
     <__main__.Measurement at 0x10f1b76d8>,
     <__main__.Measurement at 0x10f1b7780>,
     <__main__.Measurement at 0x10f1b7828>,
     <__main__.Measurement at 0x10f1b78d0>,
     <__main__.Measurement at 0x10f1b7978>,
     <__main__.Measurement at 0x10f1b7a20>,
     <__main__.Measurement at 0x10f1b7ac8>,
     <__main__.Measurement at 0x10f1b7b70>,
     <__main__.Measurement at 0x10f1b7c18>,
     <__main__.Measurement at 0x10f1b7cc0>,
     <__main__.Measurement at 0x10f1b7d68>,
     <__main__.Measurement at 0x10f1b7e10>,
     <__main__.Measurement at 0x10f1b7eb8>,
     <__main__.Measurement at 0x10f1b7f60>,
     <__main__.Measurement at 0x10f1c0048>,
     <__main__.Measurement at 0x10f1c00f0>,
     <__main__.Measurement at 0x10f1c0198>,
     <__main__.Measurement at 0x10f1c0240>,
     <__main__.Measurement at 0x10f1c02e8>,
     <__main__.Measurement at 0x10f1c0390>,
     <__main__.Measurement at 0x10f1c0438>,
     <__main__.Measurement at 0x10f1c04e0>,
     <__main__.Measurement at 0x10f1c0588>,
     <__main__.Measurement at 0x10f1c0630>,
     <__main__.Measurement at 0x10f1c06d8>,
     <__main__.Measurement at 0x10f1c0780>,
     <__main__.Measurement at 0x10f1c0828>,
     <__main__.Measurement at 0x10f1c08d0>,
     <__main__.Measurement at 0x10f1c0978>,
     <__main__.Measurement at 0x10f1c0a20>,
     <__main__.Measurement at 0x10f1c0ac8>,
     <__main__.Measurement at 0x10f1c0b70>,
     <__main__.Measurement at 0x10f1c0c18>,
     <__main__.Measurement at 0x10f1c0cc0>,
     <__main__.Measurement at 0x10f1c0d68>,
     <__main__.Measurement at 0x10f1c0e10>,
     <__main__.Measurement at 0x10f1c0eb8>,
     <__main__.Measurement at 0x10f1c0f60>,
     <__main__.Measurement at 0x10f1c8048>,
     <__main__.Measurement at 0x10f1c80f0>,
     <__main__.Measurement at 0x10f1c8198>,
     <__main__.Measurement at 0x10f1c8240>,
     <__main__.Measurement at 0x10f1c82e8>,
     <__main__.Measurement at 0x10f1c8390>,
     <__main__.Measurement at 0x10f1c8438>,
     <__main__.Measurement at 0x10f1c84e0>,
     <__main__.Measurement at 0x10f1c8588>,
     <__main__.Measurement at 0x10f1c8630>,
     <__main__.Measurement at 0x10f1c86d8>,
     <__main__.Measurement at 0x10f1c8780>,
     <__main__.Measurement at 0x10f1c8828>,
     <__main__.Measurement at 0x10f1c88d0>,
     <__main__.Measurement at 0x10f1c8978>,
     <__main__.Measurement at 0x10f1c8a20>,
     <__main__.Measurement at 0x10f1c8ac8>,
     <__main__.Measurement at 0x10f1c8b70>,
     <__main__.Measurement at 0x10f1c8c18>,
     <__main__.Measurement at 0x10f1c8cc0>,
     <__main__.Measurement at 0x10f1c8d68>,
     <__main__.Measurement at 0x10f1c8e10>,
     <__main__.Measurement at 0x10f1c8eb8>,
     <__main__.Measurement at 0x10f1c8f60>,
     <__main__.Measurement at 0x10f1d0048>,
     <__main__.Measurement at 0x10f1d00f0>,
     <__main__.Measurement at 0x10f1d0198>,
     <__main__.Measurement at 0x10f1d0240>,
     <__main__.Measurement at 0x10f1d02e8>,
     <__main__.Measurement at 0x10f1d0390>,
     <__main__.Measurement at 0x10f1d0438>,
     <__main__.Measurement at 0x10f1d04e0>,
     <__main__.Measurement at 0x10f1d0588>,
     <__main__.Measurement at 0x10f1d0630>,
     <__main__.Measurement at 0x10f1d06d8>,
     <__main__.Measurement at 0x10f1d0780>,
     <__main__.Measurement at 0x10f1d0828>,
     <__main__.Measurement at 0x10f1d08d0>,
     <__main__.Measurement at 0x10f1d0978>,
     <__main__.Measurement at 0x10f1d0a20>,
     <__main__.Measurement at 0x10f1d0ac8>,
     <__main__.Measurement at 0x10f1d0b70>,
     <__main__.Measurement at 0x10f1d0c18>,
     <__main__.Measurement at 0x10f1d0cc0>,
     <__main__.Measurement at 0x10f1d0d68>,
     <__main__.Measurement at 0x10f1d0e10>,
     <__main__.Measurement at 0x10f1d0eb8>,
     <__main__.Measurement at 0x10f1d0f60>,
     <__main__.Measurement at 0x10f1da048>,
     <__main__.Measurement at 0x10f1da0f0>,
     <__main__.Measurement at 0x10f1da198>,
     <__main__.Measurement at 0x10f1da240>,
     <__main__.Measurement at 0x10f1da2e8>,
     <__main__.Measurement at 0x10f1da390>,
     <__main__.Measurement at 0x10f1da438>,
     <__main__.Measurement at 0x10f1da4e0>,
     <__main__.Measurement at 0x10f1da588>,
     <__main__.Measurement at 0x10f1da630>,
     <__main__.Measurement at 0x10f1da6d8>,
     <__main__.Measurement at 0x10f1da780>,
     <__main__.Measurement at 0x10f1da828>,
     <__main__.Measurement at 0x10f1da8d0>,
     <__main__.Measurement at 0x10f1da978>,
     <__main__.Measurement at 0x10f1daa20>,
     <__main__.Measurement at 0x10f1daac8>,
     <__main__.Measurement at 0x10f1dab70>,
     <__main__.Measurement at 0x10f1dac18>,
     <__main__.Measurement at 0x10f1dacc0>,
     <__main__.Measurement at 0x10f1dad68>,
     <__main__.Measurement at 0x10f1dae10>,
     <__main__.Measurement at 0x10f1daeb8>,
     <__main__.Measurement at 0x10f1daf60>,
     <__main__.Measurement at 0x10f1e1048>,
     <__main__.Measurement at 0x10f1e10f0>,
     <__main__.Measurement at 0x10f1e1198>,
     <__main__.Measurement at 0x10f1e1240>,
     <__main__.Measurement at 0x10f1e12e8>,
     <__main__.Measurement at 0x10f1e1390>,
     <__main__.Measurement at 0x10f1e1438>,
     <__main__.Measurement at 0x10f1e14e0>,
     <__main__.Measurement at 0x10f1e1588>,
     <__main__.Measurement at 0x10f1e1630>,
     <__main__.Measurement at 0x10f1e16d8>,
     <__main__.Measurement at 0x10f1e1780>,
     <__main__.Measurement at 0x10f1e1828>,
     <__main__.Measurement at 0x10f1e18d0>,
     <__main__.Measurement at 0x10f1e1978>,
     <__main__.Measurement at 0x10f1e1a20>,
     <__main__.Measurement at 0x10f1e1ac8>,
     <__main__.Measurement at 0x10f1e1b70>,
     <__main__.Measurement at 0x10f1e1c18>,
     <__main__.Measurement at 0x10f1e1cc0>,
     <__main__.Measurement at 0x10f1e1d68>,
     <__main__.Measurement at 0x10f1e1e10>,
     <__main__.Measurement at 0x10f1e1eb8>,
     <__main__.Measurement at 0x10f1e1f60>,
     <__main__.Measurement at 0x10f1eb048>,
     <__main__.Measurement at 0x10f1eb0f0>,
     <__main__.Measurement at 0x10f1eb198>,
     <__main__.Measurement at 0x10f1eb240>,
     <__main__.Measurement at 0x10f1eb2e8>,
     <__main__.Measurement at 0x10f1eb390>,
     <__main__.Measurement at 0x10f1eb438>,
     <__main__.Measurement at 0x10f1eb4e0>,
     <__main__.Measurement at 0x10f1eb588>,
     <__main__.Measurement at 0x10f1eb630>,
     <__main__.Measurement at 0x10f1eb6d8>,
     <__main__.Measurement at 0x10f1eb780>,
     <__main__.Measurement at 0x10f1eb828>,
     <__main__.Measurement at 0x10f1eb8d0>,
     <__main__.Measurement at 0x10f1eb978>,
     <__main__.Measurement at 0x10f1eba20>,
     <__main__.Measurement at 0x10f1ebac8>,
     <__main__.Measurement at 0x10f1ebb70>,
     <__main__.Measurement at 0x10f1ebc18>,
     <__main__.Measurement at 0x10f1ebcc0>,
     <__main__.Measurement at 0x10f1ebd68>,
     <__main__.Measurement at 0x10f1ebe10>,
     <__main__.Measurement at 0x10f1ebeb8>,
     <__main__.Measurement at 0x10f1ebf60>,
     <__main__.Measurement at 0x10f1f3048>,
     <__main__.Measurement at 0x10f1f30f0>,
     <__main__.Measurement at 0x10f1f3198>,
     <__main__.Measurement at 0x10f1f3240>,
     <__main__.Measurement at 0x10f1f32e8>,
     <__main__.Measurement at 0x10f1f3390>,
     <__main__.Measurement at 0x10f1f3438>,
     <__main__.Measurement at 0x10f1f34e0>,
     <__main__.Measurement at 0x10f1f3588>,
     <__main__.Measurement at 0x10f1f3630>,
     <__main__.Measurement at 0x10f1f36d8>,
     <__main__.Measurement at 0x10f1f3780>,
     <__main__.Measurement at 0x10f1f3828>,
     <__main__.Measurement at 0x10f1f38d0>,
     <__main__.Measurement at 0x10f1f3978>,
     <__main__.Measurement at 0x10f1f3a20>,
     <__main__.Measurement at 0x10f1f3ac8>,
     <__main__.Measurement at 0x10f1f3b70>,
     <__main__.Measurement at 0x10f1f3c18>,
     <__main__.Measurement at 0x10f1f3cc0>,
     <__main__.Measurement at 0x10f1f3d68>,
     <__main__.Measurement at 0x10f1f3e10>,
     <__main__.Measurement at 0x10f1f3eb8>,
     <__main__.Measurement at 0x10f1f3f60>,
     <__main__.Measurement at 0x10f1fc048>,
     <__main__.Measurement at 0x10f1fc0f0>,
     <__main__.Measurement at 0x10f1fc198>,
     <__main__.Measurement at 0x10f1fc240>,
     <__main__.Measurement at 0x10f1fc2e8>,
     <__main__.Measurement at 0x10f1fc390>,
     <__main__.Measurement at 0x10f1fc438>,
     <__main__.Measurement at 0x10f1fc4e0>,
     <__main__.Measurement at 0x10f1fc588>,
     <__main__.Measurement at 0x10f1fc630>,
     <__main__.Measurement at 0x10f1fc6d8>,
     <__main__.Measurement at 0x10f1fc780>,
     <__main__.Measurement at 0x10f1fc828>,
     <__main__.Measurement at 0x10f1fc8d0>,
     <__main__.Measurement at 0x10f1fc978>,
     <__main__.Measurement at 0x10f1fca20>,
     <__main__.Measurement at 0x10f1fcac8>,
     <__main__.Measurement at 0x10f1fcb70>,
     <__main__.Measurement at 0x10f1fcc18>,
     <__main__.Measurement at 0x10f1fccc0>,
     <__main__.Measurement at 0x10f1fcd68>,
     <__main__.Measurement at 0x10f1fce10>,
     <__main__.Measurement at 0x10f1fceb8>,
     <__main__.Measurement at 0x10f1fcf60>,
     <__main__.Measurement at 0x10f205048>,
     <__main__.Measurement at 0x10f2050f0>,
     <__main__.Measurement at 0x10f205198>,
     <__main__.Measurement at 0x10f205240>,
     <__main__.Measurement at 0x10f2052e8>,
     <__main__.Measurement at 0x10f205390>,
     <__main__.Measurement at 0x10f205438>,
     <__main__.Measurement at 0x10f2054e0>,
     <__main__.Measurement at 0x10f205588>,
     <__main__.Measurement at 0x10f205630>,
     <__main__.Measurement at 0x10f2056d8>,
     <__main__.Measurement at 0x10f205780>,
     <__main__.Measurement at 0x10f205828>,
     <__main__.Measurement at 0x10f2058d0>,
     <__main__.Measurement at 0x10f205978>,
     <__main__.Measurement at 0x10f205a20>,
     <__main__.Measurement at 0x10f205ac8>,
     <__main__.Measurement at 0x10f205b70>,
     <__main__.Measurement at 0x10f205c18>,
     <__main__.Measurement at 0x10f205cc0>,
     <__main__.Measurement at 0x10f205d68>,
     <__main__.Measurement at 0x10f205e10>,
     <__main__.Measurement at 0x10f205eb8>,
     <__main__.Measurement at 0x10f205f60>,
     <__main__.Measurement at 0x10f20e048>,
     <__main__.Measurement at 0x10f20e0f0>,
     <__main__.Measurement at 0x10f20e198>,
     <__main__.Measurement at 0x10f20e240>,
     <__main__.Measurement at 0x10f20e2e8>,
     <__main__.Measurement at 0x10f20e390>,
     <__main__.Measurement at 0x10f20e438>,
     <__main__.Measurement at 0x10f20e4e0>,
     <__main__.Measurement at 0x10f20e588>,
     <__main__.Measurement at 0x10f20e630>,
     <__main__.Measurement at 0x10f20e6d8>,
     <__main__.Measurement at 0x10f20e780>,
     <__main__.Measurement at 0x10f20e828>,
     <__main__.Measurement at 0x10f20e8d0>,
     <__main__.Measurement at 0x10f20e978>,
     <__main__.Measurement at 0x10f20ea20>,
     <__main__.Measurement at 0x10f20eac8>,
     <__main__.Measurement at 0x10f20eb70>,
     <__main__.Measurement at 0x10f20ec18>,
     <__main__.Measurement at 0x10f20ecc0>,
     <__main__.Measurement at 0x10f20ed68>,
     <__main__.Measurement at 0x10f20ee10>,
     <__main__.Measurement at 0x10f20eeb8>,
     <__main__.Measurement at 0x10f20ef60>,
     <__main__.Measurement at 0x10f216048>,
     <__main__.Measurement at 0x10f2160f0>,
     <__main__.Measurement at 0x10f216198>,
     <__main__.Measurement at 0x10f216240>,
     <__main__.Measurement at 0x10f2162e8>,
     <__main__.Measurement at 0x10f216390>,
     <__main__.Measurement at 0x10f216438>,
     <__main__.Measurement at 0x10f2164e0>,
     <__main__.Measurement at 0x10f216588>,
     <__main__.Measurement at 0x10f216630>,
     <__main__.Measurement at 0x10f2166d8>,
     <__main__.Measurement at 0x10f216780>,
     <__main__.Measurement at 0x10f216828>,
     <__main__.Measurement at 0x10f2168d0>,
     <__main__.Measurement at 0x10f216978>,
     <__main__.Measurement at 0x10f216a20>,
     <__main__.Measurement at 0x10f216ac8>,
     <__main__.Measurement at 0x10f216b70>,
     <__main__.Measurement at 0x10f216c18>,
     <__main__.Measurement at 0x10f216cc0>,
     <__main__.Measurement at 0x10f216d68>,
     <__main__.Measurement at 0x10f216e10>,
     <__main__.Measurement at 0x10f216eb8>,
     <__main__.Measurement at 0x10f216f60>,
     <__main__.Measurement at 0x10f21f048>,
     <__main__.Measurement at 0x10f21f0f0>,
     <__main__.Measurement at 0x10f21f198>,
     <__main__.Measurement at 0x10f21f240>,
     <__main__.Measurement at 0x10f21f2e8>,
     <__main__.Measurement at 0x10f21f390>,
     <__main__.Measurement at 0x10f21f438>,
     <__main__.Measurement at 0x10f21f4e0>,
     <__main__.Measurement at 0x10f21f588>,
     <__main__.Measurement at 0x10f21f630>,
     <__main__.Measurement at 0x10f21f6d8>,
     <__main__.Measurement at 0x10f21f780>,
     <__main__.Measurement at 0x10f21f828>,
     <__main__.Measurement at 0x10f21f8d0>,
     <__main__.Measurement at 0x10f21f978>,
     <__main__.Measurement at 0x10f21fa20>,
     <__main__.Measurement at 0x10f21fac8>,
     <__main__.Measurement at 0x10f21fb70>,
     <__main__.Measurement at 0x10f21fc18>,
     <__main__.Measurement at 0x10f21fcc0>,
     <__main__.Measurement at 0x10f21fd68>,
     <__main__.Measurement at 0x10f21fe10>,
     <__main__.Measurement at 0x10f21feb8>,
     <__main__.Measurement at 0x10f21ff60>,
     <__main__.Measurement at 0x10f227048>,
     <__main__.Measurement at 0x10f2270f0>,
     <__main__.Measurement at 0x10f227198>,
     <__main__.Measurement at 0x10f227240>,
     <__main__.Measurement at 0x10f2272e8>,
     <__main__.Measurement at 0x10f227390>,
     <__main__.Measurement at 0x10f227438>,
     <__main__.Measurement at 0x10f2274e0>,
     <__main__.Measurement at 0x10f227588>,
     <__main__.Measurement at 0x10f227630>,
     <__main__.Measurement at 0x10f2276d8>,
     <__main__.Measurement at 0x10f227780>,
     <__main__.Measurement at 0x10f227828>,
     <__main__.Measurement at 0x10f2278d0>,
     <__main__.Measurement at 0x10f227978>,
     <__main__.Measurement at 0x10f227a20>,
     <__main__.Measurement at 0x10f227ac8>,
     <__main__.Measurement at 0x10f227b70>,
     <__main__.Measurement at 0x10f227c18>,
     <__main__.Measurement at 0x10f227cc0>,
     <__main__.Measurement at 0x10f227d68>,
     <__main__.Measurement at 0x10f227e10>,
     <__main__.Measurement at 0x10f227eb8>,
     <__main__.Measurement at 0x10f227f60>,
     <__main__.Measurement at 0x10f231048>,
     <__main__.Measurement at 0x10f2310f0>,
     <__main__.Measurement at 0x10f231198>,
     <__main__.Measurement at 0x10f231240>,
     <__main__.Measurement at 0x10f2312e8>,
     <__main__.Measurement at 0x10f231390>,
     <__main__.Measurement at 0x10f231438>,
     <__main__.Measurement at 0x10f2314e0>,
     <__main__.Measurement at 0x10f231588>,
     <__main__.Measurement at 0x10f231630>,
     <__main__.Measurement at 0x10f2316d8>,
     <__main__.Measurement at 0x10f231780>,
     <__main__.Measurement at 0x10f231828>,
     <__main__.Measurement at 0x10f2318d0>,
     <__main__.Measurement at 0x10f231978>,
     <__main__.Measurement at 0x10f231a20>,
     <__main__.Measurement at 0x10f231ac8>,
     <__main__.Measurement at 0x10f231b70>,
     <__main__.Measurement at 0x10f231c18>,
     <__main__.Measurement at 0x10f231cc0>,
     <__main__.Measurement at 0x10f231d68>,
     <__main__.Measurement at 0x10f231e10>,
     <__main__.Measurement at 0x10f231eb8>,
     <__main__.Measurement at 0x10f231f60>,
     <__main__.Measurement at 0x10f23a048>,
     <__main__.Measurement at 0x10f23a0f0>,
     <__main__.Measurement at 0x10f23a198>,
     <__main__.Measurement at 0x10f23a240>,
     <__main__.Measurement at 0x10f23a2e8>,
     <__main__.Measurement at 0x10f23a390>,
     <__main__.Measurement at 0x10f23a438>,
     <__main__.Measurement at 0x10f23a4e0>,
     <__main__.Measurement at 0x10f23a588>,
     <__main__.Measurement at 0x10f23a630>,
     <__main__.Measurement at 0x10f23a6d8>,
     <__main__.Measurement at 0x10f23a780>,
     <__main__.Measurement at 0x10f23a828>,
     <__main__.Measurement at 0x10f23a8d0>,
     <__main__.Measurement at 0x10f23a978>,
     <__main__.Measurement at 0x10f23aa20>,
     <__main__.Measurement at 0x10f23aac8>,
     <__main__.Measurement at 0x10f23ab70>,
     <__main__.Measurement at 0x10f23ac18>,
     <__main__.Measurement at 0x10f23acc0>,
     <__main__.Measurement at 0x10f23ad68>,
     <__main__.Measurement at 0x10f23ae10>,
     <__main__.Measurement at 0x10f23aeb8>,
     <__main__.Measurement at 0x10f23af60>,
     <__main__.Measurement at 0x10f242048>,
     <__main__.Measurement at 0x10f2420f0>,
     <__main__.Measurement at 0x10f242198>,
     <__main__.Measurement at 0x10f242240>,
     <__main__.Measurement at 0x10f2422e8>,
     <__main__.Measurement at 0x10f242390>,
     <__main__.Measurement at 0x10f242438>,
     <__main__.Measurement at 0x10f2424e0>,
     <__main__.Measurement at 0x10f242588>,
     <__main__.Measurement at 0x10f242630>,
     <__main__.Measurement at 0x10f2426d8>,
     <__main__.Measurement at 0x10f242780>,
     <__main__.Measurement at 0x10f242828>,
     <__main__.Measurement at 0x10f2428d0>,
     <__main__.Measurement at 0x10f242978>,
     <__main__.Measurement at 0x10f242a20>,
     <__main__.Measurement at 0x10f242ac8>,
     <__main__.Measurement at 0x10f242b70>,
     <__main__.Measurement at 0x10f242c18>,
     <__main__.Measurement at 0x10f242cc0>,
     <__main__.Measurement at 0x10f242d68>,
     <__main__.Measurement at 0x10f242e10>,
     <__main__.Measurement at 0x10f242eb8>,
     <__main__.Measurement at 0x10f242f60>,
     <__main__.Measurement at 0x10f24a048>,
     <__main__.Measurement at 0x10f24a0f0>,
     <__main__.Measurement at 0x10f24a198>,
     <__main__.Measurement at 0x10f24a240>,
     <__main__.Measurement at 0x10f24a2e8>,
     <__main__.Measurement at 0x10f24a390>,
     <__main__.Measurement at 0x10f24a438>,
     <__main__.Measurement at 0x10f24a4e0>,
     <__main__.Measurement at 0x10f24a588>,
     <__main__.Measurement at 0x10f24a630>,
     <__main__.Measurement at 0x10f24a6d8>,
     <__main__.Measurement at 0x10f24a780>,
     <__main__.Measurement at 0x10f24a828>,
     <__main__.Measurement at 0x10f24a8d0>,
     <__main__.Measurement at 0x10f24a978>,
     <__main__.Measurement at 0x10f24aa20>,
     <__main__.Measurement at 0x10f24aac8>,
     <__main__.Measurement at 0x10f24ab70>,
     <__main__.Measurement at 0x10f24ac18>,
     <__main__.Measurement at 0x10f24acc0>,
     <__main__.Measurement at 0x10f24ad68>,
     <__main__.Measurement at 0x10f24ae10>,
     <__main__.Measurement at 0x10f24aeb8>,
     <__main__.Measurement at 0x10f24af60>,
     <__main__.Measurement at 0x10f254048>,
     <__main__.Measurement at 0x10f2540f0>,
     <__main__.Measurement at 0x10f254198>,
     <__main__.Measurement at 0x10f254240>,
     <__main__.Measurement at 0x10f2542e8>,
     <__main__.Measurement at 0x10f254390>,
     <__main__.Measurement at 0x10f254438>,
     <__main__.Measurement at 0x10f2544e0>,
     <__main__.Measurement at 0x10f254588>,
     <__main__.Measurement at 0x10f254630>,
     <__main__.Measurement at 0x10f2546d8>,
     <__main__.Measurement at 0x10f254780>,
     <__main__.Measurement at 0x10f254828>,
     <__main__.Measurement at 0x10f2548d0>,
     <__main__.Measurement at 0x10f254978>,
     <__main__.Measurement at 0x10f254a20>,
     <__main__.Measurement at 0x10f254ac8>,
     <__main__.Measurement at 0x10f254b70>,
     <__main__.Measurement at 0x10f254c18>,
     <__main__.Measurement at 0x10f254cc0>,
     <__main__.Measurement at 0x10f254d68>,
     <__main__.Measurement at 0x10f254e10>,
     <__main__.Measurement at 0x10f254eb8>,
     <__main__.Measurement at 0x10f254f60>,
     <__main__.Measurement at 0x10f25c048>,
     <__main__.Measurement at 0x10f25c0f0>,
     <__main__.Measurement at 0x10f25c198>,
     <__main__.Measurement at 0x10f25c240>,
     <__main__.Measurement at 0x10f25c2e8>,
     <__main__.Measurement at 0x10f25c390>,
     <__main__.Measurement at 0x10f25c438>,
     <__main__.Measurement at 0x10f25c4e0>,
     <__main__.Measurement at 0x10f25c588>,
     <__main__.Measurement at 0x10f25c630>,
     <__main__.Measurement at 0x10f25c6d8>,
     <__main__.Measurement at 0x10f25c780>,
     <__main__.Measurement at 0x10f25c828>,
     <__main__.Measurement at 0x10f25c8d0>,
     <__main__.Measurement at 0x10f25c978>,
     <__main__.Measurement at 0x10f25ca20>,
     <__main__.Measurement at 0x10f25cac8>,
     <__main__.Measurement at 0x10f25cb70>,
     <__main__.Measurement at 0x10f25cc18>,
     <__main__.Measurement at 0x10f25ccc0>,
     <__main__.Measurement at 0x10f25cd68>,
     <__main__.Measurement at 0x10f25ce10>,
     <__main__.Measurement at 0x10f25ceb8>,
     <__main__.Measurement at 0x10f25cf60>,
     <__main__.Measurement at 0x10f264048>,
     <__main__.Measurement at 0x10f2640f0>,
     <__main__.Measurement at 0x10f264198>,
     <__main__.Measurement at 0x10f264240>,
     <__main__.Measurement at 0x10f2642e8>,
     <__main__.Measurement at 0x10f264390>,
     <__main__.Measurement at 0x10f264438>,
     <__main__.Measurement at 0x10f2644e0>,
     <__main__.Measurement at 0x10f264588>,
     <__main__.Measurement at 0x10f264630>,
     <__main__.Measurement at 0x10f2646d8>,
     <__main__.Measurement at 0x10f264780>,
     <__main__.Measurement at 0x10f264828>,
     <__main__.Measurement at 0x10f2648d0>,
     <__main__.Measurement at 0x10f264978>,
     <__main__.Measurement at 0x10f264a20>,
     <__main__.Measurement at 0x10f264ac8>,
     <__main__.Measurement at 0x10f264b70>,
     <__main__.Measurement at 0x10f264c18>,
     <__main__.Measurement at 0x10f264cc0>,
     <__main__.Measurement at 0x10f264d68>,
     <__main__.Measurement at 0x10f264e10>,
     <__main__.Measurement at 0x10f264eb8>,
     <__main__.Measurement at 0x10f264f60>,
     <__main__.Measurement at 0x10f26e048>,
     <__main__.Measurement at 0x10f26e0f0>,
     <__main__.Measurement at 0x10f26e198>,
     <__main__.Measurement at 0x10f26e240>,
     <__main__.Measurement at 0x10f26e2e8>,
     <__main__.Measurement at 0x10f26e390>,
     <__main__.Measurement at 0x10f26e438>,
     <__main__.Measurement at 0x10f26e4e0>,
     ...]




```python
session.query(Station).all()
```




    [<__main__.Station at 0x10eeb2e48>,
     <__main__.Station at 0x10eeb2eb8>,
     <__main__.Station at 0x10eeb2f28>,
     <__main__.Station at 0x10eeb2f98>,
     <__main__.Station at 0x10eebd048>,
     <__main__.Station at 0x10eebd0b8>,
     <__main__.Station at 0x10eebd128>,
     <__main__.Station at 0x10eebd198>,
     <__main__.Station at 0x10eebd208>]




```python
session.query(Measurement.date, Measurement.prcp)
```




    <sqlalchemy.orm.query.Query at 0x10eeb2d30>




```python
#Saving data from sqlalchemy to pandas dataframe

with engine.connect() as conn,conn.begin():
    data1=pd.read_sql_table('measurements',conn)
    data2=pd.read_sql_table('stations',conn)
    
     
```


```python
data1.head()
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
      <th>id</th>
      <th>station</th>
      <th>date</th>
      <th>prcp</th>
      <th>tobs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>USC00519397</td>
      <td>2010-01-01</td>
      <td>0.08</td>
      <td>65</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>USC00519397</td>
      <td>2010-01-02</td>
      <td>0.00</td>
      <td>63</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>USC00519397</td>
      <td>2010-01-03</td>
      <td>0.00</td>
      <td>74</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>USC00519397</td>
      <td>2010-01-04</td>
      <td>0.00</td>
      <td>76</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6</td>
      <td>USC00519397</td>
      <td>2010-01-07</td>
      <td>0.06</td>
      <td>70</td>
    </tr>
  </tbody>
</table>
</div>




```python
data2.head()
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
      <th>id</th>
      <th>station</th>
      <th>name</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>elevation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>USC00519397</td>
      <td>WAIKIKI 717.2, HI US</td>
      <td>21.2716</td>
      <td>-157.8168</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>USC00513117</td>
      <td>KANEOHE 838.1, HI US</td>
      <td>21.4234</td>
      <td>-157.8015</td>
      <td>14.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>USC00514830</td>
      <td>KUALOA RANCH HEADQUARTERS 886.9, HI US</td>
      <td>21.5213</td>
      <td>-157.8374</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>USC00517948</td>
      <td>PEARL CITY, HI US</td>
      <td>21.3934</td>
      <td>-157.9751</td>
      <td>11.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>USC00518838</td>
      <td>UPPER WAHIAWA 874.3, HI US</td>
      <td>21.4992</td>
      <td>-158.0111</td>
      <td>306.6</td>
    </tr>
  </tbody>
</table>
</div>




```python
# To inspect and see if my database has the tables 
from sqlalchemy import create_engine
from sqlalchemy.engine import reflection

engine = create_engine("sqlite:///dropna_hawaii.sqlite")
insp = reflection.Inspector.from_engine(engine)
print(insp.get_table_names())
```

    ['measurements', 'stations']



```python
session.query(Measurement.date, Measurement.prcp).filter(Measurement.date.between('2016-08-23', '2017-08-23'))
```




    <sqlalchemy.orm.query.Query at 0x10f5e5f98>




```python
Measurement.date.between('2016-08-23', '2017-08-23')
```




    <sqlalchemy.sql.elements.BinaryExpression object at 0x10f1080f0>


