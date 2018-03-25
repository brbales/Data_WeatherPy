
### Observations:

1. A city's weather temperature goes down as its latitude gets farther form the equator (0 degrees). The temperature v. latitude scatter plot supports this. Also, given the high number of samples and the reasonably even spread of those samples between -60 and 80 degrees of latitude we can infer that the observed patter in strong.

2. According to the humidity v. latitude scatter plot, the tightest concentration of 100% humidity weather in the sampled cities occurred for latitudes within +/-20 degrees of the equator (0 degrees). This is to be expected since any city in this region falls within what is known as the tropical zone, characterized by large tropical forests. Note, there are large portions of Africa, Arabia, and Australia within tropical zone which are deserts, as such cities in the sample population near these deserts likely account for some of the significantly lower humidity points in the graph.

3. Based on the wind speed v. latitude plot, there appears to be no correlation between latitude and wind speed. With the exception of a handful of cities, wind speed across all sampled latitudes remained between 0-30 mph. This fact lends itself to support the classification of hurricanes and tornadoes as extreme weather cases, given that wind speeds during these events greatly exceed what appears to be normal rang across much of the planet.



```python
# Dependencies
import requests as req
import json
import gzip
import random
import datetime as dt
import matplotlib.pyplot as plt
from citipy import citipy
import pandas as pd
import numpy as np


```


```python
# API key: Get your own free API Key at OWM
api_key = ""

# set base url for owp current weather requests
base_url = "http://api.openweathermap.org/data/2.5/weather"

units = "imperial"

```

### Build random sample of lat/lng coordinates


```python
# derive random lat/lng coordinates to use ac citipy inputs
lat_sample = []
lng_sample = []

for i in range(750): #will need to be much higher for final run
    i = random.uniform(-90.00, 90.00)
    lat_sample.append(i)

for j in range(750): #will need to be much higher for final run
    j = random.uniform(-180.00, 180.00)
    lng_sample.append(j)

#print(lat_sample)
#print(lng_sample)

# zip lat and lng samples into dict for use with citipy
latlng_dict = dict(zip(lat_sample, lng_sample))
#print(latlng_dict)

```


```python
# create df to be populated: include random coordinates
city_df = pd.DataFrame()
city_df["City"] = ""
city_df["Country"] = ""
city_df["Lat"] = lat_sample
city_df["Lng"] = lng_sample
city_df["Max Temp"] = ""
city_df["Humidity"] = ""
city_df["Cloudiness"] = ""
city_df["Wind Speed"] = ""
city_df["Date"] = ""
city_df = city_df[["City", "Country", "Lat", "Lng", "Date", "Max Temp",
                   "Humidity", "Cloudiness", "Wind Speed"]]
city_df.count()
```




    City            0
    Country         0
    Lat           750
    Lng           750
    Date          750
    Max Temp      750
    Humidity      750
    Cloudiness    750
    Wind Speed    750
    dtype: int64



### Use citipy to match random coordinates to actual city names


```python
# iterrate city_df through citipy to fill city and country
for index, row in city_df.iterrows():
    city_code = citipy.nearest_city(row["Lat"], row["Lng"])
    city_df.set_value(index, "City", city_code.city_name)
    city_df.set_value(index, "Country", city_code.country_code)
    
city_df.count()
```




    City          750
    Country       750
    Lat           750
    Lng           750
    Date          750
    Max Temp      750
    Humidity      750
    Cloudiness    750
    Wind Speed    750
    dtype: int64




```python
# clear randomly generated lat & lng values for replacement by owm values
city_df["Lat"] = ""
city_df["Lng"] = ""
city_df.head()
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
      <th>City</th>
      <th>Country</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Date</th>
      <th>Max Temp</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>barentsburg</td>
      <td>sj</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>saskylakh</td>
      <td>ru</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>amderma</td>
      <td>ru</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>belushya guba</td>
      <td>ru</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>mys shmidta</td>
      <td>ru</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>



### Make owm API call and populate city_df with data from JSON response


```python
# itterate city_df cities to fill empty columns and replace random sample coordinates with owm data

for index, row in city_df.iterrows():
    try:
        target_url = "http://api.openweathermap.org/data/2.5/weather?q=%s&APPID=%s&units=%s" % (row["City"], api_key, units)
        weather_data = req.get(target_url).json()
        city_df.set_value(index, "Lng", weather_data["coord"]["lon"])
        city_df.set_value(index, "Lat", weather_data["coord"]["lat"])
        city_df.set_value(index, "Max Temp", weather_data["main"]["temp_max"])
        city_df.set_value(index, "Humidity", weather_data["main"]["humidity"])
        city_df.set_value(index, "Cloudiness", weather_data["clouds"]["all"])
        city_df.set_value(index, "Wind Speed", weather_data["wind"]["speed"])
        city_df.set_value(index, "Date", str(weather_data["dt"]))
    except KeyError:
        if weather_data["cod"] == "404":
            print(weather_data["message"] + ". Skipping...")
    
    print(target_url)

    
#    print(json.dumps(weather_data, indent=4))
# dt.date.fromtimestamp(int(weather_data["dt"])).strftime("%Y-%m-%d")
```

    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=barentsburg&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=amderma&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mys shmidta&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=shunyi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vostok&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=nadym&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kokopo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hami&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=clyde river&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kendari&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=jamestown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=karratha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=tumannyy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dwarka&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=lethem&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=skalistyy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lyngseidet&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dikson&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yeppoon&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saint-philippe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=san cristobal&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sistranda&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cayenne&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kununurra&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=gamba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=krasnyy yar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hilo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cabo san lucas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=meyungs&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vaini&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kodiak&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=verkhnevilyuysk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hermanus&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=nivala&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=northview&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bitung&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mizdah&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=livramento&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=georgetown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tiksi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bilma&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=resistencia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tekeli&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bara&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=nichinan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=avarua&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=falmouth&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=broken hill&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mackenzie&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=waitati&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mahibadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=marzuq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=narsaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dicabisagan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atuona&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=biak&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=khatanga&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port blair&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=messina&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mayumba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vaini&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sao joao dos patos&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=riom&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=avarua&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=torbay&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dikson&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=salalah&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=massakory&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=havoysund&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=aklavik&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=christchurch&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bouar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=upernavik&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vikulovo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bluff&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=augusto correa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kupang&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bengkulu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tena&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lazaro cardenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=petropavlovsk-kamchatskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bambous virieux&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tynda&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=paamiut&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=nenjiang&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lufilufi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=caravelas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto del rosario&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=abu kamal&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto princesa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=castro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=krasnyy yar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=shitanjing&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mar del plata&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=alice springs&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=cockburn harbour&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=salalah&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=asau&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=naryan-mar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bogatyye saby&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lompoc&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=chernyakhovsk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=georgetown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=zhuanghe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=honiara&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cooma&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=baracoa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pisco&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=belaya gora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lumeje&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=biak&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=belyy yar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=novopokrovka&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=satitoa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kodiak&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=katsuura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=talnakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=castro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=alofi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=provideniya&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kruisfontein&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=east london&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lloydminster&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=homagama&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=gorontalo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=benguela&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobyo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tiksi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=belmonte&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=constitucion&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=novyy svit&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pevek&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dikson&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bollnas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=torbay&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atuona&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=santo antonio do sudoeste&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sambava&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=inta&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=digha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=meulaboh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=yanan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=airai&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ancud&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vao&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bubaque&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sinnamary&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=xadani&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kiama&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saint george&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=georgetown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=san patricio&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=airai&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=gamba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=imbituba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kieta&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=iracoubo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kamaishi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sao filipe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=bac lieu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sladkovo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=alamosa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lagoa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=katsuura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=pareora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bentiu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hermanus&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hudson bay&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ribeira grande&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=waipawa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=gangotri&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lorengau&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=stornoway&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=totma&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=khatanga&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=grand gaube&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=coahuayana&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hermanus&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=altay&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=castro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lebu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=sentyabrskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=charlestown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pisco&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bambous virieux&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sambava&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=esperance&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=paraiso&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bluff&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kabompo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=svetlogorsk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lerwick&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rafai&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=noumea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sydney mines&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=nalut&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mayumba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=alibag&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=passagem franca&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ballangen&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=butaritari&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=marcona&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dikson&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=thompson&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vaini&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=tidore&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=shahreza&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vestmannaeyjar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kodiak&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=san quintin&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=shimoda&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=aden&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=olafsvik&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tiksi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=buchanan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=qui nhon&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saldanha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mys shmidta&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=san patricio&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sao filipe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yeppoon&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=crab hill&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port hardy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=butaritari&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=viedma&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=coatzintla&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=jalu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=san patricio&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=east london&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=fortaleza&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atuona&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=klaksvik&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=san carlos de bariloche&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=khor&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=castro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bhuj&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cabo san lucas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=bajram curri&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rupert&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lebu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=nizhneyansk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=manikpur&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=inta&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=broken hill&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vaini&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=fuyu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sitka&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=korla&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dhidhdhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=zhezkazgan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dikson&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bluff&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ode&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=alexandria&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=addi ugri&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ciudad guayana&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=soure&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lebu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saint-georges&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=verkhnyaya inta&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=avarua&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pisco&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hvide sande&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hit&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=castro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=poum&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tiksi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kavieng&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=khatanga&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atuona&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=walvis bay&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=olafsvik&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=avarua&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=geraldton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=nikolskoye&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sarankhola&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=fairbanks&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=jamestown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=pirsagi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bluff&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=moron&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=japura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=zavallya&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sorochinsk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=padang&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=stromness&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=fort frances&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=avarua&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mairana&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lodja&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kendari&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=doha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hilo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=college&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hasaki&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=katherine&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ust-kan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=manutuke&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=beringovskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=nizhneyansk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=grand river south east&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=khatanga&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=camacha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=itatiba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=tarudant&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ixtapa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saldanha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=longlac&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atasu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vila franca do campo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mys shmidta&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bethel&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dong xoai&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cloquet&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qasigiannguit&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=alyangula&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=reconquista&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bandarbeyla&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=nizhneyansk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tiksi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kavieng&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=inirida&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=olga&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=malpe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vyborg&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hun&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=doaba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hermanus&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kungurtug&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=gangapur&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=georgetown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bengkulu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bilma&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sioux lookout&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barkly west&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=kokkola&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=petatlan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vaini&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kota belud&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=alihe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=arrecife&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=donauworth&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vila franca do campo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bethel&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=karasuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sitka&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hilo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rakaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rabo de peixe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=east london&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=taoudenni&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hudson bay&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=guerrero negro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=beyneu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bethel&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=east london&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atuona&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sistranda&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bandarbeyla&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sao joao da barra&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hilo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=makurdi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=terney&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=namatanai&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ilhabela&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=grand river south east&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=karachi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ribeira grande&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=adre&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=adre&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=moerai&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=madimba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=barentsburg&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bluff&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=clyde river&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ixtapa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=airai&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=chuy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vila velha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mandalgovi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=torbay&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port-gentil&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tigil&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=khasan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=botou&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=phan rang&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tiksi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=chhatarpur&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yulara&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vanimo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=poum&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=takoradi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=loreto&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hamilton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=praia da vitoria&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sens&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port hardy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lompoc&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=avarua&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hermanus&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ca mau&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hamadan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pangkalanbuun&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kununurra&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=adrar&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=laguna&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=fougamou&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=esperance&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=jamestown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=atuona&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=narrabri&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=digapahandi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sola&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=grand-santi&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kasongo-lunda&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=khatanga&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kitimat&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=fortuna&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=myanaung&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dicabisagan&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port macquarie&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=we&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=micheweni&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=barrow&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=santa comba&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hovd&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=santa maria&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=lorengau&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ugoofaaru&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=vaitupu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=nouadhibou&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=naze&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=chateaubelair&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kungurtug&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cape town&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=southbridge&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port alfred&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=oranjemund&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saint joseph&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vila franca do campo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=dikson&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=port lincoln&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saldanha&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=te anau&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hermanus&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=anadyr&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=jamestown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=palenque&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=avera&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=crewe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=fortuna&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hilo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=uvat&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=jamestown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=bahia blanca&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=barentsburg&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hobart&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=albany&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=vaini&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=sawtell&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=las vegas&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=flinders&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=homer&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=isangel&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=isangel&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=norman wells&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=tilichiki&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=samalaeulu&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kapaa&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yerbogachen&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=namibe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=arraial do cabo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=kaili&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ginir&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=busselton&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=rikitea&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=georgetown&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=mataura&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=butaritari&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    city not found. Skipping...
    http://api.openweathermap.org/data/2.5/weather?q=kamenskoye&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=meulaboh&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=khatanga&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=kyabe&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo&APPID=f37c799063873e4e92effc4350a3c35b&units=imperial
    

### Clean city_df of any cities owm did not find, leaving empty cells


```python
# replace blank cells witn nan
city_df.replace('', np.nan, inplace=True)

city_df.count()
```




    City          750
    Country       750
    Lat           653
    Lng           653
    Date          653
    Max Temp      653
    Humidity      653
    Cloudiness    653
    Wind Speed    653
    dtype: int64




```python
# remove any rows rows from sample which containing a nan value
city_df.dropna(axis=0, how="any", inplace=True)
city_df.count()
```




    City          653
    Country       653
    Lat           653
    Lng           653
    Date          653
    Max Temp      653
    Humidity      653
    Cloudiness    653
    Wind Speed    653
    dtype: int64




```python
city_df.head()
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
      <th>City</th>
      <th>Country</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Date</th>
      <th>Max Temp</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>saskylakh</td>
      <td>ru</td>
      <td>71.92</td>
      <td>114.08</td>
      <td>1512795639</td>
      <td>-7.94</td>
      <td>65.0</td>
      <td>44.0</td>
      <td>6.85</td>
    </tr>
    <tr>
      <th>5</th>
      <td>shunyi</td>
      <td>cn</td>
      <td>40.12</td>
      <td>116.65</td>
      <td>1512793800</td>
      <td>46.40</td>
      <td>19.0</td>
      <td>0.0</td>
      <td>4.47</td>
    </tr>
    <tr>
      <th>6</th>
      <td>vostok</td>
      <td>ru</td>
      <td>46.49</td>
      <td>135.88</td>
      <td>1512795970</td>
      <td>-2.36</td>
      <td>68.0</td>
      <td>44.0</td>
      <td>2.53</td>
    </tr>
    <tr>
      <th>7</th>
      <td>nadym</td>
      <td>ru</td>
      <td>65.53</td>
      <td>72.52</td>
      <td>1512795641</td>
      <td>16.95</td>
      <td>86.0</td>
      <td>80.0</td>
      <td>7.85</td>
    </tr>
    <tr>
      <th>8</th>
      <td>kokopo</td>
      <td>pg</td>
      <td>-4.35</td>
      <td>152.26</td>
      <td>1512795971</td>
      <td>79.59</td>
      <td>100.0</td>
      <td>80.0</td>
      <td>5.28</td>
    </tr>
  </tbody>
</table>
</div>



### Send final city_df data to .csv


```python
# save the final df as a csv
city_df.to_csv("city_weather_data.csv", encoding="utf-8", index=False)
```

### Build a scatter plots for each required comparison


```python
# temp v. lat
plt.scatter(city_df["Lat"], 
            city_df["Max Temp"],
            edgecolor="k", linewidths=1, marker="o", 
            alpha=0.9)

# format graph 
plt.grid()
plt.xlim([-90, 90])
plt.ylim([-40, 120])

plt.xlabel("Latitude")
plt.ylabel("Max Temperature (F)")
plt.title("City Max Temperature (F) vs. Latitude")

# save figure
plt.savefig("temp_lat.png")

# show plot
plt.show()
```


![png](output_18_0.png)



```python
# humid v. lat
plt.scatter(city_df["Lat"], 
            city_df["Humidity"],
            edgecolor="k", linewidths=1, marker="o", 
            alpha=0.9)

# format graph 
plt.grid()
plt.xlim([-90, 90])
plt.ylim([0, 110])

plt.xlabel("Latitude")
plt.ylabel("Humidity (%)")
plt.title("City Humidity (%) vs. Latitude")

# save figure
plt.savefig("humidity_lat.png")

# show plot
plt.show()
```


![png](output_19_0.png)



```python
# clouds v. lat
plt.scatter(city_df["Lat"], 
            city_df["Cloudiness"],
            edgecolor="k", linewidths=1, marker="o", 
            alpha=0.9)

# format graph 
plt.grid()
plt.xlim([-90, 90])
plt.ylim([-10, 110])

plt.xlabel("Latitude")
plt.ylabel("Cloudiness (%)")
plt.title("City Cloudiness (%) vs. Latitude")

# Save the figure
plt.savefig("cloudy_lat.png")

# show plot
plt.show()
```


![png](output_20_0.png)



```python
# wind v. lat
plt.scatter(city_df["Lat"], 
            city_df["Wind Speed"],
            edgecolor="k", linewidths=1, marker="o", 
            alpha=0.9)

# format graph 
plt.grid()
plt.xlim([-90, 90])
plt.ylim([-10, 65])

plt.xlabel("Latitude")
plt.ylabel("Wind Speed (mph)")
plt.title("City Wind Speed (mph) vs. Latitude")

# save figure
plt.savefig("wind_lat.png")

# show plot
plt.show()
```


![png](output_21_0.png)

