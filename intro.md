## Lesson Goals

OpenRefine is a powerful tool for exploring, cleaning, and transforming data. 
An earlier Programming Historian [lesson](http://programminghistorian.org/lessons/cleaning-data-with-openrefine) introduced the basic functionality of Refine to efficiently discover and correct inconsistency in a data set.
Building on those essential data wrangling skills, this lesson focuses on Refine's ability to fetch urls and parse web content.
Examples introduce some of the advanced features to transform and enhance a data set including: 

- fetch URLs using Refine
- construct URLs to retrieve information from a simple web API
- parse HTML and JSON responses to extract relevant data
- use array functions to manipulate string values

### Why Use OpenRefine?

The ability to create data sets from unstructured documents available on the web opens possibilities for research using digitized primary materials, web archives, and contemporary media streams. 
Programming Historian lessons introduce a number of methods to gather and interact with this content, from [wget](http://programminghistorian.org/lessons/applied-archival-downloading-with-wget) to [Python](http://programminghistorian.org/lessons/intro-to-beautiful-soup).
When working with text documents, Refine is particularly suited for this task, allowing users to fetch urls and directly process the results in an iterative, exploratory manner.

> David Huynh, the creator of Freebase Gridworks (2009) which became GoogleRefine (2010) and then OpenRefine (2012+), says Refine is:
> 
> - more powerful than a spreadsheet
> - more interactive and visual than scripting
> - more provisional / exploratory / experimental / playful than a database [^huynh]

[^huynh]: David Huynh, "Google Refine", Computer-Assisted Reporting Conference 2011, http://web.archive.org/web/20150528125345/http://davidhuynh.net/spaces/nicar2011/tutorial.pdf.

Refine is a unique tool that combines the power of databases and scripting languages into an interactive and user friendly visual interface. 
Because of this flexibility it has been embraced by [journalists](https://www.propublica.org/nerds/item/using-google-refine-for-data-cleaning), [librarians](http://data-lessons.github.io/library-openrefine/), [scientists](http://www.datacarpentry.org/OpenRefine-ecology-lesson/), and others needing to wrangle data from diverse sources and formats into structured information.

![OpenRefine](images/openrefine.png)

> OpenRefine is a [free](https://www.gnu.org/philosophy/free-sw.en.html) and [open source](https://github.com/OpenRefine/OpenRefine) Java application.
> The user interface is rendered by your web browser, but Refine is not a web application. No information is sent online and no internet connection is necessary.
> Full documentation is available on the [official wiki](https://github.com/OpenRefine/OpenRefine/wiki/).
> For installation and staring Refine check this [workshop page](https://evanwill.github.io/clean-your-data/3-start.html).

### Examples

This lesson presents two examples demonstrating workflows to harvest and process data from the web.
First, "Fetching and Parsing HTML" introduces functions to parse an HTML document into a structured data set.
Second, "URL Queries and Parsing JSON" introduces interacting with a simple web API to construct a full text data set of historic newspaper front pages. 
