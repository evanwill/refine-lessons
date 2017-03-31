## Example 1: URL Queries and Parsing JSON

Many cultural institutions provide web APIs enabling users to access data about their collections via simple HTTP requests.
This example will explore a simple API provided by the [Chronicling America](http://chroniclingamerica.loc.gov/) project. 

To get started, it is necessary to poke around the web site to understand how the desired content can be harvested.
Chronicling America provides [documentation](http://chroniclingamerica.loc.gov/about/api/) for their API. 
However, it is possible to gain further information by viewing the source of the HTML pages. 
Alternate links in the `<head>` element point to other available formats in addition to the HTML document (for example, `<link rel="alternate" type="application/json" href="/lccn/sn86088527/1917-03-29/ed-1/seq-1.json" />`).
In this case, the information on each page is available as HTML, Atom-XML, or JSON, depending on the options passed with the URL.
Finally, a search link `<link title="NDNP Page Search" href="/search/pages/opensearch.xml" rel="search" type="application/opensearchdescription+xml" />` points us to an XML document that provides templates for constructing HTTP search queries. 

> Chronicling America is fully open, no key or account is needed to access the API and there are no limits on the use. 
> Other APIs are often proprietary and restricted.
> Please review the specific terms of use before web scraping or using the information in research.

This API documentation is a recipe book explaining how to interact with the server using a public URL.
To demonstrate interacting with the Chronicling America API using Refine, this example constructs a small full text data set of newspaper front pages.
Construct the query URL, fetch the information, parse the response.

### Start Chronicling America project

To get started, open Refine, select `Create project`, and Get Data From `Clipboard`. 
Paste this CSV into the text box:

```
state,year
Idaho,1865
Washington,1865
```

Click next. 
Refine should automatically identify the content as a CSV, so the default parsing options should be correct.
Add a descriptive `Project name` at the top right and click `Create project`.

### Construct a query

To construct the API query URL, create a new column by clicking the menu arrow on the `state` column > `edit column` > `Add column based on this column`.
Give the new column the name `url`, then click in the `Expression` text box.
This box accepts functions written in GREL (General Refine Expression Language, [docs](https://github.com/OpenRefine/OpenRefine/wiki/General-Refine-Expression-Language)) that will be applied to each cell in the existing column when creating values for the new column.

Notice that the default expression is `value`, the GREL variable representing the current value of a cell. 
This means that each cell will be copied to the new column exactly. 
The preview below the expression box should reflex this.
Delete `value`, and paste this expression:

```
"http://chroniclingamerica.loc.gov/search/pages/results/?state=" + value + "&date1=" + cells['year'].value + "&date2="+ cells['year'].value + "&dateFilterType=yearRange&sequence=1&rows=20&format=json"
```

The preview should update, and you will now see a URL as output. 
This is a search query based on the API documentation constructed from our data using GREL.
Examine each component of the expression to understand how it works:

- `http://chroniclingamerica.loc.gov/` is the base URL given in the API documentation.
- `search/pages/results` is the search service location for individual newspaper pages, given in the API documentation.
- `?` starts a web form *query string*. A query string is made up of value pairs (`fieldname=value`) separated by `&`. This string is not a hierarchical file location like a normal URL, but is passed to the server or page for processing. 
- `+` GREL concatenates string (text) values with the plus sign. For example the expression `"one" + "two"` would result in "onetwo".
- `value` is the current value of each cell. In this case it is the state value listed in the `state` column, thus in row 1 value = Idaho and in row 2 value = Washington. 
- `cells['year'].value` is the GREL variable that retrieves the value from a different column in the same row. In this case it is the value listed in the `year` column. 

Much like using the [advanced search form](http://chroniclingamerica.loc.gov/#tab=tab_advanced_search), the value pairs of the query string set the options for the Chronicling America search. 
The first query URL will ask for newspapers from the Idaho (`state=Idaho`), from the year `1865`, only the front pages (`sequence=1`), returning a max of 20 (`rows=20`) in JSON (`format=json`).
Our second query URL does the same for `Washington`. 

### Fetch URLs

The `url` column is now a list of web queries that could be accessed with a browser.
Refine's builtin function to retrieve a list of URLs is done by creating a new column.  

Click on the `url` column > `edit column` > `Add column by fetching urls`.
Name the new column `json`. 
A `throttle delay` can be set to ensure Refine waits a reasonable amount of time between requests. 
The default is conservative. 
Click okay.
Refine will start requesting the URLs as if you were opening the pages in your browser, and store each response in the cells of the new column.
After a few seconds, the `json` column should be filled with a JSON data structure. 

### Parse JSON to get Items

The first elements of the JSON response look like `"totalItems": 52, "endIndex": 20`. 
This indicates that our search resulted in 52 total items, but the response contains only 20 (since we limited it).
The Refine project currently contains only two rows, but the two `json` responses contain information about a total of 40 newspaper pages nested in the JSON `items` element. 
Thus, to construct a orderly data set, it is necessary to parse the JSON and split each item into its own row.

GREL has a builtin function to easily parse JSON. 
Click on the `json` column > `edit column` > `Add column based on this column`. 
Name the column `items` and enter the expression:

```
value.parseJson()['items'].join(":::")
```

GREL `parseJson()` allows us to select an element `['items']`, in this case exposing the array of newspaper records nested inside the JSON response.
The `join()` function concatenates an array with the given separator. For example, the expression `[1,2,3].join(";")` will result in "1;2;3".  
Since the newspaper items contain an OCR text field, the strange separator ":::" is necessary to ensure that it is unique.

> Notice that variables and functions are strung together in sequence using `.`, starting with the raw cell `value`.
> This allows complex operations to be constructed by passing the results of each function to the next.

With the newspaper records isolated, individual rows can be created by splitting the cells.
Click on the `items` column > `edit cells` > `split multivalued cells`, and enter the join used in the last step, `:::`. 

Each of the cells in the `items` column are now split into multiple rows. 
The top the table should read 40 rows.
If you click on Show as `records`, it will read 2, representing our original CSV rows.
Click back on `rows`.

So far in this example, most operations result in creating new columns. 
This is a typical workflow with Refine allowing each transformation to be easily checked against the existing data.
However, at this point all the necessary information is contained in the `items` column. 
It is a good time to clean up the unnecessary columns.
Click on the `all` column > `Edit columns` > `Re-order / remove columns`. 
Drag the unnecessary column names to the right side, then click `ok` to remove them. 
With the original columns removed, both `records` and `rows` will read 40.

### Parse JSON values

To complete the data set, it is necessary to parse each newspaper's JSON record into individual columns. 
This is a very common task, as most web APIs return information in JSON format.
Again, GREL's `parseJson()` function make this easy. 
For each JSON key, create a new column from `items`, parse the JSON, and select the key:

- date: `value.parseJson()['date']`
- title: `value.parseJson()['title']`
- city: `value.parseJson()['city'].join(", ")`
- lccn: `value.parseJson()['lccn']`
- text: `value.parseJson()['ocr_eng']`

Each of these columns could be further refined using other GREL transformations.
For example, the `date` column could be converted to a more readable format using `value.toDate("yyyymmdd").toString("yyyy-MM-dd")` or `value.splitByLengths(4,2,2).join("-")`.
A URL could be constructed to retrieve the full item based on `lccn` using `"http://chroniclingamerica.loc.gov/lccn/" + value + "/" + cells.date.value.splitByLengths(4,2,2).join("-") + "/ed-1/"`.

### Automate



### Going further

This data can be enhanced by other services offered via APIs such as geocoding or named entity recognition. 
Some extensions, such as [Refine-NER-Extension](https://github.com/RubenVerborgh/Refine-NER-Extension), help automate enhancing the data by reconciling with remote sources. However, since many of the APIs are from commercial companies, (often proprietary and restricted, with specific terms of use) the implementation details regularly change, making these extensions difficult to maintain.
