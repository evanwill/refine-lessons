## Lesson Goals

OpenRefine is a powerful tool for exploring, cleaning, and transforming data. 
An earlier Programming Historian [lesson](http://programminghistorian.org/lessons/cleaning-data-with-openrefine) introduced the basic functionality of Refine to efficiently discover and correct inconsistency in a data set.
Building on those essential data wrangling skills, this lesson focuses on Refine's ability to fetch urls and parse web content.
Examples introduce some of the advanced features to transform and enhance a data set. 

- construct URLs to retrieve information from a simple web API
- fetch URLs using refine
- parse JSON and HTML response to extract relevant data
- use array functions to manipulate string values

## Why Use OpenRefine?

The ability to create data sets from unstructured documents available on the web opens possibilities for research using digitized primary materials, web archives, and contemporary media streams. 
Programming Historian lessons introduce a number of methods to gather and interact with this content, from [wget](http://programminghistorian.org/lessons/applied-archival-downloading-with-wget) to [Python](http://programminghistorian.org/lessons/intro-to-beautiful-soup).
When working with text documents, Refine is particularly suited for this task, allowing users to fetch urls and directly process the results in an iterative, exploratory manner.

> Refine is:
> 
> - more powerful than a spreadsheet
> - more interactive and visual than scripting
> - more provisional / exploratory / experimental / playful than a database
> 
> [David Huynh](http://web.archive.org/web/20150528125345/http://davidhuynh.net/spaces/nicar2011/tutorial.pdf), the creator of Freebase Gridworks (2009) which became GoogleRefine (2010) and then OpenRefine (2012+)

Refine is a unique tool that combines the power of databases and scripting languages into a more interactive and user friendly visual interface. 
Because of this flexibility it has been embraced by [journalists](https://www.propublica.org/nerds/item/using-google-refine-for-data-cleaning), [librarians](http://data-lessons.github.io/library-openrefine/), [scientists](http://www.datacarpentry.org/OpenRefine-ecology-lesson/), and others needing to wrangle data from diverse sources and formats into structured information.

![1]()

> OpenRefine is a [free](https://www.gnu.org/philosophy/free-sw.en.html) and [open source](https://github.com/OpenRefine/OpenRefine) Java application.
> The user interface is rendered by your web browser, but Refine is not a web application. No information is sent online and no internet connection is necessary.
> Full documentation is available on the [official wiki](https://github.com/OpenRefine/OpenRefine/wiki/).
> For installation, check this [workshop slide](https://evanwill.github.io/clean-your-data/3-start.html).

## Examples

This lesson presents two examples that demonstrate the functions and work flows to harvest and process data accessible from the web.
First, "URL Queries and Parsing JSON" introduces interacting with a simple web API to construct a full text data set of historic newspaper front pages. 
Second, "Fetching and Parsing HTML" introduces functions to navigate and parse HTML documents into a structured data set.
