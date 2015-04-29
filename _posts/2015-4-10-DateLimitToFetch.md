---
layout: post
title: Limiting dates to fetch the Earthquakes
---

![_config.yml]({{ site.baseurl }}/images/timespan.png)

Since [GeoNet](geonet.org.nz) provides earthquakes data from a certain date, I had to develop a function to limit the time span.

The user can interact with the application using the cellphone and tilting it to the left (back in time) or right (forward in time), the application starts with today's date. To generate the date, I used Python's toOrdinal() function, which transforms a regular date expression (01/01/1950) to a number (711858). This is helpful because I don't have to calculate which month has 30 or 31 days, neither 28 or 29 of February. To change the date, I just have to add or subtract a number.

The initial date I chose is 1950 and the last possible date is Now. To limit the date, I used a DAT execute to retrieve the date from Python, this DAT is execute each frame, because if the application is running and the current day changes, the application just update this information. So TouchDesigner can be kept running forever, and the date will be always updated.
![_config.yml]({{ site.baseurl }}/images/datexecute.png)




Geonet provides a query constructor at [quakesearch.geonet.org.nz](http://quakesearch.geonet.org.nz/):
![_config.yml]({{ site.baseurl }}/images/geonet.png)

The resultant query would be something like this:
`http://quakesearch.geonet.org.nz/services/1.0.0/geojson?startdate=2015-3-26T23:00:00&enddate=2015-3-24T0:00:00`

and the returned JSON will look like this:
![_config.yml]({{ site.baseurl }}/images/json.png)

To fetch the data in TouchDesigner, a 'DAT web' component is used with a 'DAT execute' component to parse the JSON string. The parser then fills a 'DAT table' with all the information about the earthquakes on the requested time span.

###Web network:
![_config.yml]({{ site.baseurl }}/images/webjsonparser.png)

###Parser inside the 'DAT execute' component:

{% highlight python %}
import json
from datetime import datetime

#set up reference to the table in which we'll store the values
eqTable = op('earthquake')

#if the webDat changes, this function is run
def tableChange(dat):

	#clear the table
	eqTable.clear()

	#get the raw json string
	jsonRaw = dat.text
	op('onlinecheck').text = 1

  #create a list with the desired keys
	eqKeys = list(['lat', 'long', 'magnitude', 'depth', 'year', 'month', 'day', 'hour', 'minute'])
	eqTable.appendRow(eqKeys)

	if not jsonRaw:
		jsonRaw = op('offlineEq').text
		print ('no connection with geonet, using predefined data')
		op('onlinecheck').text = 0


	#convert the json string to a python dict
	fullDict = json.loads(jsonRaw)

	eqData = fullDict['features']

	for m in reversed(eqData):
		eqValues = list()

    #append all the values in each row of the table
		eqProperties = dict(m['properties'])
		if 'magnitude' in eqProperties:
			eqValues.append(eqProperties['latitude'])
			eqValues.append(eqProperties['longitude'])
			eqValues.append(eqProperties['magnitude'])
			eqValues.append(eqProperties['depth'])

			#break origin time
			ot = datetime.strptime(eqProperties['origintime'], '%Y-%m-%dT%H:%M:%S.%fZ').timetuple()
      #year
			eqValues.append(ot[0])
			#month
			eqValues.append(ot[1])
			#day
			eqValues.append(ot[2])
			#hour
			eqValues.append(ot[3])
			#minute
			eqValues.append(ot[4])
			eqTable.appendRow(eqValues)
	print ('Json ok')
	return
  {% endhighlight %}

When the JSON is parsed, the data will show up nicely in the earthquake table:
![_config.yml]({{ site.baseurl }}/images/eqtable.png)

Now this table holds all the needed information to display the earthquakes
