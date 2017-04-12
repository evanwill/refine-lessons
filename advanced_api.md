## Example 3: Advanced APIs

*Example 2* demonstrated working with a simple API using Refine's fetch function, essentially utilizing URL patterns to request information from a server. 
Refine's fetch can only use the HTTP GET protocol, meaning the query is encoded in the URL string, thus limited in length (2048 ASCII characters), complexity, and security.
Many API services that could be used to enhance text data, such as [geocoding](https://en.wikipedia.org/wiki/Geocoding) or [named entity recognition](https://en.wikipedia.org/wiki/Named-entity_recognition), use HTTP POST to transfer information to the server for processing. 
GREL does not have a built in function to use this type of API.
However, the expression window language can be changed to [Jython](http://www.jython.org/), providing a more complete scripting environment where it is fairly simple to implement a POST request.

> [Jython](http://www.jython.org/) is Python implemented for the Java VM and comes bundled with Refine (look for the `.jar` file in `openrefine-2.7-rc.2/webapp/extensions/jython/`).
> The [official documentation](https://github.com/OpenRefine/OpenRefine/wiki/Jython) is sparse.
> The current version is Jython 2.7, based on [Python 2.7](https://docs.python.org/2.7/), and can be extended with non-standard libraries using a [work around](https://github.com/OpenRefine/OpenRefine/wiki/Extending-Jython-with-pypi-modules).
> Keep in mind that spending time writing complex scripts moves away from the strengths of Refine. 
> If it is necessary to develop a lengthy Jython routine, it will likely be more efficient to process the data directly in Python. 
> On the other hand, if you know a handy method to process data in Python 2, Jython is an easy way to apply it in a Refine project.

### Jython in the expression window

Return to the Sonnets project completed in *Example 1*. 
If the tab was closed, click *Open* > *Open Project* and find the Sonnets example. 

On the *first* column > *Edit column* > *Add column based on this column*.
On the right side of the *Expression* box is a drop down to change the expression language.
Select *Python / Jython* from the list.

Notice that the preview now shows `null` for the output. 
A Jython expression in Refine must have a `return` statement to add the output to the new cells in the transformation.
Replace `value` with `return value`, simply copying the current cells. 
The basic [GREL variables](https://github.com/OpenRefine/OpenRefine/wiki/Variables) can be used in Jython by substituting brackets instead of periods. 
For example, the GREL `cells.last.value` would be Jython `cells['last']['value']`. 

![jython expression](images/refine-jython.png)

### POST request

To create a HTTP request in Jython, use the standard libraries [urllib2](http://www.jython.org/docs/library/urllib2.html) and [urllib](http://www.jython.org/docs/library/urllib.html).
The fetch column function can be recreated with Jython to demonstrate an HTTP GET request. 
In the expression box, type:

```
import urllib2
url = 'http://www.jython.org/'
page = urllib2.urlopen(url).read()
return page
```

The preview should display the HTML source of the Jython home page.
The `url` variable could use `value` to construct a query similar to the fetch used in *Example 2*.

> A throttle delay can be added by importing `time` and adding `time.sleep(15)` to the script, replacing 15 with the number of seconds to delay. 

To create a POST request, add data to the urllib2 request object. 
For example, [text-processing.com](http://text-processing.com/) provides free natural language processing APIs based on the [Python NLTK](http://www.nltk.org/).
The documentation for the [Sentiment Analysis](http://text-processing.com/docs/sentiment.html) service says to send an HTTP POST request "with form encoded data containing the text you want to analyze." 
This type of API is often demonstrated using [curl](https://curl.haxx.se/) on the commandline, for example: `curl -d "text=what is the sentiment of this sentence?" http://text-processing.com/api/sentiment/`. 
The curl example can be recreated in Jython to test the service:

```
import urllib2
url = 'http://text-processing.com/api/sentiment/'
return urllib2.urlopen(url,"text=what is the sentiment of this sentence?").read()
```

The preview should show a JSON response with sentiment probability values.
To retrieve sentiment analysis data for the first lines of the sonnets, put this basic Jython pattern together with the values of the cells.
The API documentation gave us the base URL and the name for the key (`text`) used for the data.
The code is in a non-compact form to demonstrate each step.

```
import urllib2, urllib
url = "http://text-processing.com/api/sentiment/"
data = urllib.urlencode({"text": value})
query = urllib2.Request(url,data)
request = urllib2.urlopen(query)
response = request.read()
return response
```

![jython request](images/refine-jython-request.png)

Click *Ok* to run the Jython script for every row in the column.
The JSON response can then be parsed using the methods demonstrated in *Example 2*.
this API allows 1000 per day per IP address. 

Compare with another sentiment analysis service, http://sentiment.vivekn.com/docs/api/
In this case the docs say base url http://sentiment.vivekn.com/api/text/ and key `txt`. 

```
import urllib2
url = "http://sentiment.vivekn.com/api/text/"
return urllib2.urlopen(url,"txt="+value).read()
```

These are very simple free APIs for demonstration purposes. 
pay attention to what the models are trained on. both are trained on movie reviews, thus optimized for small bits of text with language similar to a review. 
OpenCalais is trained on news.
We should be asking questions about the alogrithms to understand the metrics they are capable of producing, and bias and limitations built in.

> Some extensions, such as [Refine-NER-Extension](https://github.com/RubenVerborgh/Refine-NER-Extension), help automate enhancing the data by reconciling with remote sources. 
> However, since many of the APIs are from commercial companies, (often proprietary and restricted, with specific terms of use) the implementation details regularly change, making these extensions difficult to maintain.

> if you have errors, you may need to `value.escape('xml')` first

They may also require authentication headers send with the request. 

by default, the second parameter is the data that you want to pass along with the request. Instead, you have to specify that you want to pass headers.
import urllib2
url = 'https://api.fitbit.com/1/user/-/activities/heart/date/2016-6-14/1d/1sec/time/00:00/23:59.json'
hdr = {'Authorization': 'Bearer (token)'}
req = urllib2.Request(url,headers=hdr)
f = urllib2.urlopen(req)
