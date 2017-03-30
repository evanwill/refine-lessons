## Example 2: Parsing HTML

Example 1 demonstrated how to fetch a list of URLs and parse JSON. 
GREL also has a builtin function to easily parse HTML and XML based on [jsoup](https://jsoup.org/).
This example parses a single web page into a structured table and introduces array functions that are useful for working with strings. 
A similar workflow could be applied to a full list of URLs. 

### Start Sonnets project

This example will create an ordered data set from an HTML copy of Shakespeare's [Sonnets](http://www.gutenberg.org/ebooks/1105) from [Project Gutenberg](http://www.gutenberg.org/). Project Gutenberg provides [feeds](http://www.gutenberg.org/wiki/Gutenberg:Feeds) to bulk download their catalog data, so please do not use their public website for web scraping purposes. In this case we are accessing the text of a single HTML book, not scraping the site.

To get started, click the `Open` button in the upper right, which will open a new Refine tab.
Select `Create project` and Get Data From `Clipboard`. 
Paste this URL into the text box: `http://www.gutenberg.org/cache/epub/1105/pg1105-images.html`

Click next.
Refine should automatically identify the content as a Line-based text file, so the default parsing options should be correct.
Add a descriptive `Project name` at the top right and click `Create project`.

### Fetch HTML

From `Column 1`, create a new column using `edit column` > `Add column by fetching urls`.
Name the column `fetch`.

When the operation completes, the `fetch` column will have the full HTML for the Sonnets web page. 
To make it easier to look at, click the URL in `Column 1` and view-source of the page in a new tab. 
Much of the code is not sonnets and must be removed. 

### Parse HTML

GREL's builtin function `parseHtml()` can read HTML content, allowing elements to be accessed using the jsoup [selector syntax](https://jsoup.org/cookbook/extracting-data/selector-syntax).
First, it is necessary to identify a pattern that can isolate the desired content from the HTML.
In many cases, the items will be nested in a unique container or given a meaningful class or id.
The sonnets page does not have distinctive semantic markup, but each poem is contained inside a single `<p>` element. 
If all the paragraphs are selected, the sonnets can be extracted from the array.

On `fetch`, create a new column using `edit column` > `Add column based on this column`.
Name the new column `slice`.
Then type `value.parseHtml().select("p")` into the expression box.
Notice that the preview now shows an array of all the `<p>` elements found in the page. 
Watch the preview window as the GREL expression changes to understand how it functions.

Adding an index number to the expression selects one element from the array, for example `value.parseHtml().select("p")[0]`.
The beginning of the file contains many paragraphs of license information that is unnecessary for the data set. 
Skipping ahead, the first sonnet is found at `value.parseHtml().select("p")[37]`. 
GREL also supports using negative index numbers, thus `value.parseHtml().select("p")[-1]` will return the last item in the array. 
Working backwards, the last sonnet is at `[-3]`.
Using these index numbers, it is possible to slice the array, extracting only the range of `<p>` that contain sonnets. 
Add the `slice()` function to the expression to preview the sub-set, `value.parseHtml().select("p").slice(37,-2)`.

Clicking `ok` with this expression will result in a blank column, a common cause of confusion when working with arrays.
Refine will not record an array object as a cell value. 
It is necessary to use `toString()` or `join()` to convert the array into a string.
Thus, the final expression to create the `slice` column is:

`value.parseHtml().select("p").slice(37,-2).join("|")`

### Split cells

The `slice` column now contains all the sonnets separated by "|". 
To put each sonnet on its own row, `edit cells` > `split multivalued cells`, with separator `|`.
This should result in 154 rows.

The `<p>` tags around each sonnet can be cleaned up by parsing the HTML again.
Click on the `slice` column and select `edit cells` > `transform`.
In the expression box, type `value.parseHtml()`.
Notice that the preview shows a complete HTML tree starting with the `<html>` element.
It is important to note that `parseHtml()` will automatically fill in missing tags, which allows it to parse these cell values despite not being valid HTML documents.
Now select the `p` tag, add an index number, and the function `innerHtml()` to extract the sonnet text:

`value.parseHtml().select("p")[0].innerHtml()`

### Unescape

Each cell is filled with `&nbsp;`, a HTML entity used to represent "no-break space" since browsers ignore extra white space in the source.
These special characters can be quickly removed using GREL `unescape()`.
Select `edit cells` > `transform` and type `value.unescape('html')` in the expression box.
The entities should be replaced with normal whitespace.

### Extract information with array functions

To finish processing the sonnets, we will continue to use [array functions]((https://github.com/OpenRefine/OpenRefine/wiki/GREL-Array-Functions), which are a powerful way to manipulate text data.
We can create an array from any string value using the GREL `split()` function, providing the character or expression to split on (this is the opposite of `join()`). 

Each line in the sonnets is separated by `<br />`, providing a convenient separator for splitting. 
Once `split()` creates an array, index numbers and slices can be used to create new columns.
Keep in mind that Refine won't output an array directly to a cell.
It will have to be converted back into a string value with join.

At the same time, we can clean up the unnecessary white space on each line using the `trim()` function.
Trim automatically removes all leading and trailing white space, an essential function for data cleaning. 

Create new columns from the `slice` column using `edit column` > `Add column based on this column` with the names and expressions below:

- number: `value.split("<br />")[0].trim()`
- first: `value.split("<br />")[1].trim()`

Trim only works on string values and removes white space from the beginning and end of the cell. 
In the expressions above, the results are single lines.
However, the full sonnet has multiple lines so trim will not touch the unnecessary whitespace.
To trim each line individually we can use the `forEach` control, a powerful loop that iterates over an array.

Create another new column from the `slice` column, and type `forEach(value.split("<br />"),lines,lines.trim())` in the expression box.
`value.split("<br />")` creates an array from the string value in each cell.
each item in the array is then represented as a variable specified next, `lines` (it could be anything, usually `v`).
each item is then evaluated separately with the expression `lines.trim()`.
The results of each separate function is returned as a new array.
Thus we can apply any normal array functions to the end of forEach(), such as slice and join.

Extract the full sonnet text with the expression:

`forEach(value.split("<br />"),lines,lines.trim()).slice(1).join("\n")`

And add a column for the last lines (couplet) using:

`forEach(value.split("<br />"),lines,lines.trim()).slice(-3).join("\n")`

Finally, we can add a few numeric columns.
- characters: `value.length()`
- lines: `value.split(/\n/).length()`

remove and reorder columns 

explore data with numeric facet 

export 
