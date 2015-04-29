---
layout: post
title: Controlling Time
---

![_config.yml]({{ site.baseurl }}/images/timespan.png)

Since [GeoNet](geonet.org.nz) provides earthquakes data from a certain date, I had to develop a function to limit the time span.

The user can interact with the application using the cellphone and tilting it to the left (back in time) or right (forward in time), the application starts with today's date. To generate the date, I used Python's toOrdinal() function, which transforms a regular date expression (01/01/1950) to a number (711858). This is helpful because I don't have to calculate which month has 30 or 31 days, neither 28 or 29 of February. To change the date, I just have to add or subtract a number.

The initial date I chose is 1950 and the last possible date is Now. To limit the date, I used a DAT execute to retrieve the date from Python, this DAT is execute each frame, because if the application is running and the current day changes, the application just update this information. So TouchDesigner can be kept running forever, and the date will be always updated.
![_config.yml]({{ site.baseurl }}/images/datexecute.png)

###Code to get today's date, inside the 'DAT execute' component:
{% highlight python %}
def frameEnd(frame):
	table = op('table2')
	firstdate = date(1950, 1, 1).toordinal()
	today = date.today().toordinal()
	table.clear()
	# date constraints, (1950 to Today)
	table.appendRow(firstdate)
	table.appendRow(today)
	table.appendRow(today-firstdate)
	return
{% endhighlight %}

I also created manual user input, this allows the user to set a specific date without tilting the cellphone.
###Manual User Input:
![_config.yml]({{ site.baseurl }}/images/manualinput.png)
