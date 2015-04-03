---
layout: post
title: Fetching Earthquake data from Geonet.org.nz to TouchDesigner
---

Our project is an interactive earthquake visualizer, so information about the earthquakes is the main requirement. In New Zealand the best service that provides data about earthquakes is [GeoNet](geonet.org.nz), the website also provides a very smart API that returns data in several formats including JSON, the format that I'm going to use in the project.

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
```
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
			eqValues.append(ot[0])
			eqValues.append(ot[1])
			eqValues.append(ot[2])
			eqValues.append(ot[3])
			eqValues.append(ot[4])
			eqTable.appendRow(eqValues)
	print ('Json ok')
	return
```
