## Advanced APIs


> This data can be enhanced by other services offered via APIs such as geocoding or named entity recognition. 
> Some extensions, such as [Refine-NER-Extension](https://github.com/RubenVerborgh/Refine-NER-Extension), help automate enhancing the data by reconciling with remote sources. > However, since many of the APIs are from commercial companies, (often proprietary and restricted, with specific terms of use) the implementation details regularly change, making these extensions difficult to maintain.
> Add to a map using [Google Geocoding API](https://developers.google.com/maps/documentation/geocoding/intro)

Refine's Fetch can only GET, not HTTP POST

to POST in Refine it is possible to use Jython in the expression window instead of GREL.
import urllib2, urllib
url = 'http://sentiment.vivekn.com/api/text/'
params = urllib.urlencode({'txt': 'some angry text test'})
req = urllib2.Request(url,params)
f = urllib2.urlopen(req)
d = f.read()
return d

[urllib2](http://www.jython.org/docs/library/urllib2.html)
urllib2.urlopen(url[, data][, timeout])
urllib2.Request(url[, data][, headers][, origin_req_host][, unverifiable])
- HTTP request will be a POST instead of a GET when the data parameter is provided.
- data should be a buffer in the standard application/x-www-form- urlencoded format. The urllib.urlencode() 

by default, the second parameter is the data that you want to pass along with the request. Instead, you have to specify that you want to pass headers.
import urllib2
url = 'https://api.fitbit.com/1/user/-/activities/heart/date/2016-6-14/1d/1sec/time/00:00/23:59.json'
hdr = {'Authorization': 'Bearer (token)'}
req = urllib2.Request(url,headers=hdr)
f = urllib2.urlopen(req)


[Jython](http://www.jython.org/) is Python implemented for the Java VM, thus it is added to Refine (a Java app) using a simple self-contained `.jar` file.

basic use:
- current version is Jython 2.7 (check in openrefine-2.7-rc.2/webapp/jython/), based on Python 2.7.
- jython in Refine requires `return` in the statement. 
- For example if you were to `print
import urllib2
f = urllib2.urlopen('http://www.python.org/')
return f.read(100)
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<?xml-stylesheet href="./css/ht2html

current version is Jython 2.7 (check in openrefine-2.7-rc.2/webapp/jython/)

the documentation on the Refine wiki is spotty, https://github.com/OpenRefine/OpenRefine/wiki/Jython

can be customized with other libraries using a work around, https://github.com/OpenRefine/OpenRefine/wiki/Extending-Jython-with-pypi-modules
