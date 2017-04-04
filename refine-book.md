## Example 1: Fetching and Parsing HTML

This example fetches a single web page and parses it into a structured table. 
Refine has a builtin function to parse HTML and XML based on [jsoup](https://jsoup.org/).
[Array functions](https://github.com/OpenRefine/OpenRefine/wiki/GREL-Array-Functions) from the General Refine Expression Language ([GREL](https://github.com/OpenRefine/OpenRefine/wiki/General-Refine-Expression-Language)) will be introduced for manipulating text. 
A similar workflow can be applied to a list of URLs, creating a flexible web harvesting tool. 

> The raw data for this example is an HTML copy of Shakespeare's [Sonnets](http://www.gutenberg.org/ebooks/1105) from [Project Gutenberg](http://www.gutenberg.org/). 
> Please note that Project Gutenberg provides [feeds](http://www.gutenberg.org/wiki/Gutenberg:Feeds) to bulk download catalog data. 
> Their public website should not be used for web scraping purposes. 

### Start Sonnets Project

Start OpenRefine, select `Create project`, and Get Data From `Clipboard`. 
Paste this URL into the text box: `https://raw.githubusercontent.com/uidaholib/refine-demo/master/pg1105.html`

![Refine clipboard]()

After clicking next, Refine should automatically identify the content as a Line-based text file, the default parsing options should be correct.
Add a descriptive `Project name` at the top right and click `Create project`.
This will result in a new project with one column and one row. 

### Fetch HTML

Refine's builtin function to retrieve a list of URLs is done by creating a new column.
Click on the menu arrow of `Column 1` > `edit column` > `Add column by fetching urls`.

![Add column]()

Name the new column `fetch`. 
A `throttle delay` can be set to ensure Refine waits a reasonable amount of time between requests. 
The default is conservative. 
After clicking okay, Refine will start requesting the URLs in the given column as if you were opening the pages in your browser, and store each response in the cells of the new column.
When the operation completes, the `fetch` column will have the full HTML for the Sonnets web page. 

### Parse HTML

Notice that much of the web page is not sonnet text and must be removed to create a clean data set.
GREL's builtin function `parseHtml()` can read HTML content, allowing elements to be accessed using the jsoup [selector syntax](https://jsoup.org/cookbook/extracting-data/selector-syntax).
First, it is necessary to identify a pattern that can isolate the desired content.
In many cases, the items will be nested in a unique container or given a meaningful class or id.
The sonnets page does not have distinctive semantic markup, but each poem is contained inside a single `<p>` element. 
If all the paragraphs are selected, the sonnets can be extracted from the array.

On the `fetch` column, click on the menu arrow > `edit column` > `Add column based on this column`.
Name the new column `parse`.
Then type `value.parseHtml().select("p")` into the expression box.

![create new column]()

Notice that the preview now shows an [array](https://en.wikipedia.org/wiki/Array_data_type) of all the `<p>` elements found in the page. 
Try the following GREL expressions and watch the preview window to understand how they function.
Adding an index number to the expression selects one element from the array, for example `value.parseHtml().select("p")[0]`.
The beginning of the file contains many paragraphs of license information that is unnecessary for the data set. 
Skipping ahead, the first sonnet is found at `value.parseHtml().select("p")[37]`. 
GREL also supports using negative index numbers, thus `value.parseHtml().select("p")[-1]` will return the last item in the array. 
Working backwards, the last sonnet is at `[-3]`.

Using these index numbers, it is possible to slice the array, extracting only the range of `<p>` that contain sonnets. 
Add the `slice()` function to the expression to preview the sub-set: `value.parseHtml().select("p").slice(37,-2)`.

Clicking `ok` with this expression will result in a blank column, a common cause of confusion when working with arrays.
Refine will not record an array object as a cell value. 
It is necessary to use `toString()` or `join()` to convert the array into a string.
Thus, the final expression to create the `parse` column is:

`value.parseHtml().select("p").slice(37,-2).join("|")`

### Split Cells

The `parse` column now contains all the sonnets separated by "|", but the project still contains only one row. 
Individual rows for each sonnet can be created by splitting the cell.
Click the menu arrow on the `parse` column > `edit cells` > `split multivalued cells`. 
Enter the separator `|` that was used to `join` in the last step.
The top of the project table should now read 154 rows.

![split multivalued cells]()

Each cell in the `parse` column now contains one sonnet surround by a `<p>` tag.
The tags can be cleaned up by parsing the HTML again.
Click on the `parse` column and select `edit cells` > `transform`.
This will bring up a dialog box similar to create new column.
Transform will overwrite the current column cells rather than creating a new column.

![edit cells transform]()

In the expression box, type `value.parseHtml()`.
Notice that the preview shows a complete HTML tree starting with the `<html>` element.
It is important to note that `parseHtml()` will automatically fill in missing tags, which allows it to parse these cell values despite not being valid HTML documents.
To extract the sonnet text, select the `p` tag, add an index number, and the function `innerHtml()`:

`value.parseHtml().select("p")[0].innerHtml()`

### Unescape

Each cell is filled with `&nbsp;`, a HTML entity used to represent "no-break space" since browsers ignore extra white space in the source.
These special characters can be quickly removed using GREL the `unescape()` function.
Select `edit cells` > `transform` and type `value.unescape('html')` in the expression box.
The entities should be replaced with normal whitespace.

### Extract Information with Array Functions

[GREL array functions]((https://github.com/OpenRefine/OpenRefine/wiki/GREL-Array-Functions) provide a powerful way to manipulate text data and can be used to finish processing the sonnets.
Any string value can be turned into an array using the GREL `split()` function by providing the character or expression to split on (this is the opposite of `join()`). 

Each line in the sonnets is separated by `<br />`, providing a convenient separator for splitting. 
Once `split()` creates an array, index numbers and slices can be used to create new columns.
Keep in mind that Refine will not output an array directly to a cell.
It will have to be converted back into a string value with join.

At the same time, we can clean up the unnecessary white space on each line using the `trim()` function.
Trim automatically removes all leading and trailing white space, an essential for data cleaning. 

Create new columns from the `parse` column using `edit column` > `Add column based on this column` with the names and expressions below:

- number: `value.split("<br />")[0].trim()`
- first: `value.split("<br />")[1].trim()`

Trim only works on string values and removes white space from the beginning and end of the cell. 
The expressions above use an index number resulting in a single line.
However, the full sonnet has multiple lines.
Trim will only clean the beginning and end of the cell, leaving unnecessary whitespace in the body of the sonnet.
To trim each line individually use the GREL `forEach` control, a powerful loop that iterates over an array.

Create another new column named `text` from the `parse` column, and type `forEach(value.split("<br />"),lines,lines.trim())` in the expression box.
First, `value.split("<br />")` creates an array from the string value in each cell.
Each item in the array is then represented as a variable specified next, `lines` (it could be anything, usually `v`).
Each item is then evaluated separately with the expression `lines.trim()`.
The results of each separate function is returned as a new array.
Thus, the expression above will trim each individual line in each cell of the `parse` column.
Because the result is a new array, additional functions can be applied to the end of `forEach()`, such as slice and join.

The final expression to extract and clean the full sonnet text is:

`forEach(value.split("<br />"),lines,lines.trim()).slice(1).join("\n")`

And add another new column from `parse` named `last` for the final couplet lines using:

`forEach(value.split("<br />"),lines,lines.trim()).slice(-3).join("\n")`

Finally, numeric columns can be added using the `length()` function.
Create new columns from the `text` column with the names and expressions below:

- characters: `value.length()`
- lines: `value.split(/\n/).length()`

### Cleanup and Export

In this example, many operations result in creating new columns. 
This is a typical workflow with Refine allowing each transformation to be easily checked against the existing data.
At this point the unnecessary columns can be removed. 
Click on the `all` column > `Edit columns` > `Re-order / remove columns`. 
Drag unwanted column names to the right side, in this case `Column 1`, `fetch`, and `parse`. 
Drag the remaining columns in the desired order on the left side.
Click `ok` to remove and reorder the data set. 

![reorder columns]()

Use the export button to download a version of the new sonnet table.
