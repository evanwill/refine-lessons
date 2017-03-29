## Example: Chronicling America API

Many cultural institutions provide web APIs enabling users to access data about their collections via simple HTTP requests.
Making use of these APIs in Refine allows us to create a targeted dataset by constructing the correct HTTP requests (urls) and parsing the response.
This data can be enhanced by other services offered via APIs such as geocoding or named entity recognition. 
Some extensions, such as [Refine-NER-Extension](https://github.com/RubenVerborgh/Refine-NER-Extension), help automate enhancing the data by reconciling with remote sources. However, since many of the APIs are from commercial companies, (often proprietary and restricted, with specific terms of use) the implementation details regularly change, making these extensions difficult to maintain.
This example will explore a simple API provided by the Chronicling America project and manually construct queries. 

To get started, it is necessary to poke around the web site to understand how the desired content can be harvested.
Chronicling America provides [documentation](http://chroniclingamerica.loc.gov/about/api/) for their simple API. 
However, it is possible to gain further information by viewing the source of the HTML pages. 
Alternate links in the `<head>` element point to other formats available for the data presented on the page (for example, `<link rel="alternate" type="application/json"`).
In this case, the information on each page is available as HTML, Atom-XML, or JSON, depending on the options passed with the URL.
Finally, a search link `<link title="NDNP Page Search" href="/search/pages/opensearch.xml" rel="search" type="application/opensearchdescription+xml" />` points us to an XML document that provides templates for constructing queries. 

This API documentation is a recipe book explaining how to interact with the server using a public URL.  
To demonstrate interacting with the API using Refine, let's construct a small full text data set.

> Chronicling America is full open, no key or account is needed to access the API, and there are no limits on the use. 
> Other API are often proprietary and restricted, with specific terms of use. 

### Construct a query

Although most start by importing a tabular file, pasting values into the clipboard is often the easiest way to start a project.
To get started, open Refine, select `Create project`, and Get Data From `Clipboard`. 
Paste in this CSV:

```
state,year
Idaho,1865
Washington,1865
```

Click next. 
Refine should automatically identify the content as a CSV, so the default parsing options should be correct.
Add a `Project name` at the top right and click `Create project`.

To create a new column with the API query URL, click the arrow on the `state` column > `edit column` > `Add column based on this column`.
Give the new column a name, then click in the `Expression` text box.
We will use GREL (General Refine Expression Language, [docs](https://github.com/OpenRefine/OpenRefine/wiki/General-Refine-Expression-Language)) to create the new value. 
Notice that the current expression is `value`, meaning what ever is in the cell will be copied to the new column exactly. 
If you look at the preview below, this should be reflected.
Delete `value`, and paste this expression:

```
"http://chroniclingamerica.loc.gov/search/pages/results/?state=" + value + "&date1=" + cells['year'].value + "&date2="+ cells['year'].value + "&dateFilterType=yearRange&sequence=1&rows=20&format=json"
```

The preview should update, and you will now see a url as output. 
This is a search query based on the API documentation constructed from our data using GREL.
Look at each component of the expression to understand how it works:
- `http://chroniclingamerica.loc.gov/` base url 
- `search/pages/results` service location (`pages` allows us to search individual newspaper pages, versus `search/titles/results` which searches the newspaper title directory)
- `?` starts a web form *query string*. It is made up of value pairs (`fieldname=value`) separated by `&`. This string is not a hierarchical file location like a normal URL, but is passed to the server or page for processing. 
- `+`, GREL concatenates strings like `"one" + "two"`
- `value` is the GREL variable representing the current value of a cell. 
- `cells['year'].value` is the GREL variable that retrieves the value from a different column in the same row. 

In this case, our first query URL will ask for newspapers from the state `Idaho`, from the year `1865`, only the front page (`sequence=1`), returning a max of 20 (`rows=20`) in JSON (`format=json`).
Our second query URL does the same for `Washington`. 

### Fetch data

Now, click on the `url` column > `edit column` > `Add column by fetching urls`.
Name the new column `json`. 
You set a `throttle delay` to ensure Refine waits a reasonable amount of time between requests. 
The default is conservative. 
Click okay and Refine start requesting the urls.
After a few seconds, you should have a new column filled with a JSON data structure. 

### Parse JSON to get Items

To finish constructing our structured data set, it is necessary to parse the JSON response, split into individual records, and move the values into new columns.
If you look at the JSON, the first elements look like `"totalItems": 52, "endIndex": 20,`. 
This indicates that our search query resulted in 52 total items, but this JSON response contains 20 (since we limited it).
Our data set currently contains only two rows. 
We would like to split the individual records, resulting in a total of 40 rows. 
First however, we need to isolate the individual records. 
They are nested in the JSON response in the `items` element.

GREL has a builtin function to easily parse JSON. 
Click on the `json` column > `edit column` > `Add column based on this column`. 
Name the column `items` and enter the expression:

```
value.parseJson()['items'].join(":::")
```

GREL `parseJson()` allows us to select an element `['items']`, in this case exposing the array of items nested in the JSON record.
The `join()` function concants an array with the given separator. 
Notice that the functions are strung together in sequence using `.`, starting with the raw cell `value`.
I have selected a very unlikely join string `":::"` that I hope is not repeated in the OCR text of the old newspapers represented in the items. 

With the item records isolated, we can split them into individual records.
Click on the `items` column > `edit cells` > `split multivalued cells`, and enter the join used in the last step, `:::`. 

Each of the cells in the `items` column are now split into multiple rows. 
The top of your table should now read 40 rows.
If you click on `records`, it will read 2, representing our original CSV rows.

Click back on `rows`.
Let's clean this up. 
All the information we need is in the `items` column, all others can be deleted. 
Click on the `all` column > `Edit columns` > `Re-order / remove columns`. 
Drag the unnecessary column names to the right side, then click `ok` to remove them. 
With the original columns removed, now both `records` and `rows` will read 40.

### Parse JSON values

To complete the data set, it is necessary to parse the item JSON in to individual columns. 
For each, create a new column from `items`, parse the JSON, and select the element:

- date: `value.parseJson()['date']`
- title: `value.parseJson()['title']`
- city: `value.parseJson()['city'].join(", ")`
- lccn: `value.parseJson()['lccn']`
- text: `value.parseJson()['ocr_eng']`

Each of these columns could be further refined using other GREL transformations.
For example, the `date` column could be converted to a more readable format using `value.toDate("yyyymmdd").toString("yyyy-MM-dd")` or `value.splitByLengths(4,2,2).join("-")`.
A URL could be constructed to retrieve the full item based on `lccn` using `"http://chroniclingamerica.loc.gov/lccn/" + value + "/" + cells.date.value.splitByLengths(4,2,2).join("-") + "/ed-1/"`.
