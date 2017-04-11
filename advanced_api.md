## Example 3: Advanced APIs

*Example 2* demonstrated working with a simple API using Refine's fetch function, essentially utilizing URL patterns to request information from a server. 
Refine's fetch can only use the HTTP GET protocol, meaning the query is encoded in the URL string, thus limited in length (2048 ASCII characters), complexity, and security.
Many API services that could be used to enhance text data, such as [geocoding](https://en.wikipedia.org/wiki/Geocoding) or [named entity recognition](https://en.wikipedia.org/wiki/Named-entity_recognition), use HTTP POST to transfer information to the server for processing. 
GREL does not have a built in function to use this type of API.
However, the language of the expression window can be changed to [Jython](http://www.jython.org/), providing a more complete scripting environment where it is fairly simple to implement a POST request.

> [Jython](http://www.jython.org/) is Python implemented for the Java VM and comes bundled with Refine (see the `.jar` file in `openrefine-2.7-rc.2/webapp/jython/`).
> The current version is Jython 2.7, based on [Python 2.7](https://docs.python.org/2.7/).
> Keep in mind that spending time writing complex scripts moves away from the strengths of Refine. 
> If it is necessary to develop a lengthy Jython routine for Refine, it will likely be more efficient to process the data directly in Python. 
> On the other hand, if you know a handy method to process data in Python 2, Jython is an easy way to apply to a Refine project.

### Jython in the expression window

Return to the Sonnets project completed in *Example 1*. 
If the tab is still open, simply switch to the tab. 
Otherwise click *Open* > *Open Project* and find the Sonnets example. 

On the *first* column > *Edit column* > *Add column based on this column*.
On the right side of the *Expression* box is a drop down to change the expression language.
Select *Python / Jython* from the list.

Notice that the preview now shows `null` for the output. 
A Jython expression in Refine must have a `return` statement to add the output to the new cells in the transformation.
Replace `value` with `return value`, simply copying the current cells. 
The basic GREL variables can be used, however use brackets instead of periods in Jython, for example GREL `cells['last'].value` would be Jython `cells['last']['value']`. 

![jython expression](images/refine-jython.png)

### POST request

This type of API is often demonstrated using [curl](https://curl.haxx.se/) on the commandline, for example: `curl -d "text=what is the sentiment of this sentence?" http://text-processing.com/api/sentiment/`.


http://text-processing.com/docs/sentiment.html

import urllib2, urllib
url = 'http://text-processing.com/api/sentiment/'
params = urllib.urlencode({'text': 'some angry text test'})
req = urllib2.Request(url,params)
f = urllib2.urlopen(req)
d = f.read()
return d

 
> Some extensions, such as [Refine-NER-Extension](https://github.com/RubenVerborgh/Refine-NER-Extension), help automate enhancing the data by reconciling with remote sources. 
> However, since many of the APIs are from commercial companies, (often proprietary and restricted, with specific terms of use) the implementation details regularly change, making these extensions difficult to maintain.

> pay attention to what the models are trained on. NLTK is trained on movie reviews, thus optimized for small bits of text with language simalar to a review. OpenCalais is trained on news.

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


They may also require authentication headers send with the request. 

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


the documentation on the Refine wiki is spotty, https://github.com/OpenRefine/OpenRefine/wiki/Jython

can be customized with other libraries using a work around, https://github.com/OpenRefine/OpenRefine/wiki/Extending-Jython-with-pypi-modules
