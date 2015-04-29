---
layout: post
title: Earthquakes In A Day
---

![_config.yml]({{ site.baseurl }}/images/peaks.gif)

After fetching the earthquake data using the [JSON Parser]({{ site.baseurl }}/FetchingEarthquakeData/), I had to create some nodes to show the earthquakes as the time pass by that day.

###Table with all the earthquakes in that particular day:
![_config.yml]({{ site.baseurl }}/images/eqtable.png)
After the information is nicely placed in the table, two 'DAT Select' are used to select the right information at the right time:
![_config.yml]({{ site.baseurl }}/images/selects.gif)
The first select fetches the data based on the hour in the 'CHOP Constant' time. The second select fetches the data from 'select1' based on the minute.
After the process, I created two 'CHOP Trail' to export the data (As seen in the first Post image)
