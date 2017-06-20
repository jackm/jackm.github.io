---
layout: post
title:  "Thermostat Data Collection and Graphing"
date:   2016-12-07 00:00:00
---

About a year ago a new furnace was installed in the house and along with it came a fancy new thermostat,
the [Lennox iComfort WiFi Touchscreen thermostat][lennox-icomfort].
This particular thermostat happened to have WiFi built into it and the ability to use a smartphone app to control the
thermostat settings remotely with the use of personal account credentials. Up until now, I've had the WiFi disabled because
I saw no use for it, but I discovered that the API used in the app for this thermostat has been reverse engineered and that
it is possible to retreive data from it for other purposes.

I decided that I wanted to be able to retreive the temperature data from my thermostat and graph it using Cacti/rrdtool.
Using simple HTTP GET and PUT requests, you can retreive and even set some parameters for the thermostat.
A summary of the Lennox iComfort API can be found [here][lennox-api].

All that was needed was to create a simple script using a language of choice, such as Python,
and make it perform a few HTTP GET requests using my Lennox account credentials which is linked to my thermostat.
The response back is an easy to parse JSON object. After that you can pull out whichever data you are interested in,
and then pass it on to Cacti to record and graph.

The final result is something like the following where I graph the indoor and outdoor temperature,
as well as the heat-to and cool-to programmed setpoints.

{% include image.html name="thermostat_graph.png" %}

## References

<https://www.iwasdot.com/extending-a-lennox-icomfort-thermostat/>

<https://github.com/ut666/LennoxThermoPi-II>

[lennox-icomfort]: http://www.lennox.com/products/comfort-controls/thermostats/icomfortwi-fi
[lennox-api]: http://htmlpreview.github.io/?https://github.com/ut666/LennoxThermoPi-II/blob/master/lennoxAPI.html
