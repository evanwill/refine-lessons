## Advanced APIs

*Example 2* demonstrated working with a simple API using Refine's fetch function, essentially using URL patterns to request information from a server. 
Refine's fetch can only use HTTP GET, meaning the query is encoded in the URL string, thus limited in length (2048 ASCII characters), complexity, and security.
However, many API services that could be used to enhance text data, such as geocoding or named entity recognition, use HTTP POST to transfer information to the server for processing. 
They may also require authentication headers send with the request. 

This type of API is often demonstrated using [curl](https://curl.haxx.se/), for example: `curl -d "text=what is sentiment of this sentence?" http://text-processing.com/api/sentiment/`.
GREL does not have a built in function to reproduce this request.
However, the language of the expression window can be changed to [Jython](http://www.jython.org/) or [Clojure](https://clojure.org/) which are more complete scripting languages.
Keep in mind that spending more time writing complex scripts moves away from the strength of Refine. 
If it is necessary to develop a complex Jython routine, it will likely be better to do it directly in Python in Jupyter Notebook which preserves some of the iterative, exploratory interactivity of Refine.
However, it is fairly simple to implement a POST request using Jython, which 



http://text-processing.com/docs/sentiment.html

import urllib2, urllib
url = 'http://text-processing.com/api/sentiment/'
params = urllib.urlencode({'text': 'some angry text test'})
req = urllib2.Request(url,params)
f = urllib2.urlopen(req)
d = f.read()
return d


> This data can be enhanced by other services offered via APIs such as geocoding or named entity recognition. 
> Some extensions, such as [Refine-NER-Extension](https://github.com/RubenVerborgh/Refine-NER-Extension), help automate enhancing the data by reconciling with remote sources. > However, since many of the APIs are from commercial companies, (often proprietary and restricted, with specific terms of use) the implementation details regularly change, making these extensions difficult to maintain.

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
