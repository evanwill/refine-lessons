# harvest book

The Sonnets by William Shakespeare
http://www.gutenberg.org/ebooks/1105

start from clip board:
http://www.gutenberg.org/cache/epub/1105/pg1105-images.html

create new column by fetch, html

want to get rid of junk at top, looking at code notice that the first <h1> is the title before the sonnets start, so we can use that as a break point:
split column html on "<h1" 2 columns

now "html 2" has the text we are interested in, each sonnet is in a "<p>"
we can parse the html to cleanly extract each sonnet.
transform cells, type in box: 
value.parseHtml().select("p")[0].innerHtml()
note the index number [0] is necessary since our selector returns many items. change the index number and check the preview to see if you are getting the poems. 
we need to create a loop to parse them all. 
forEach(value.parseHtml().select("p"),e,e.innerHtml()).join("|")

now we need to split into multiple cells using the separator we used
edit cells > split mutlivalued cells > |

note that the first row is now all stuff we don't want, so lets clean it up. 
click the star. 
click to last rows. note that the last two rows are also stuff we dont want. star those as well. 
facet by star > true,  All > edit rows > remove all matching. close the facet.
remove the empty columns. 

notice that since we used "innerHtml" you see a lot of &nbsp;, these are html entities to represent special characters on the web. let't get rid of them safely using refines unescape, on the column edit cells > transform :
value.unescape('html')

now lets extract the sonnet number from the text. 
add column based on this column, number 
there is a few ways to do this, but lets look at using [array functions]((https://github.com/OpenRefine/OpenRefine/wiki/GREL-Array-Functions), which are a powerful way to manipulate text data . 
We can create an array from any string using the split function, providing the character or expression we want to split on. In the sonnets, each line is separated by "<br />", so we can use that. type:
split(value, "<br />")
in your expression box and look at the preview. note that there is now an array, a list of values enclosed by [] and separated by a comma. 
Let select the first item in the array by adding an index number:
split(value, "<br />")[0]
Click ok, now we have a column with the sonnet number. 

lets create a column with the first line. 
html, create column based on this column, first:
split(value, "<br />")[1]

lets remove the number from the text using the array function slice.
We need to give slice an array, so we can use the expression from above. slice then asks for a start and end point. The number was at index [0], so our start is [1]. We can leave the end point out, and slice will assume until the end: 
slice(split(value, "<br />"),1)
if you look at the preview, you see an array with the lines we want. however, if you click ok, nothing will happen. Refine doesn't allow you to output an array directly to a cell. we need to convert it back to a string value. in the above examples we did that by using an index number, thus selecting an actual string value from the array. 
Since we want more than one of the array values, we need to join them together with the join() function. we need to specify the join character:
slice(split(value, "<br />"),1).join("\n") 

Now that we have cleaned it up, lets rename the column. edit column > rename > sonnet 
 
lets grab the last two lines which are usually a final couplet in the sonnets. we need to create an array again, but now all the <br /> are gone, so we need a different separator. of course we just used \n to join, so that is the separator to break it back apart.  edit column add column based > last 
split(value, /\n/)
how to grab the last? lets quickly count the lines, change the new column name to lines:
length(split(value, /\n/))
check the range with a text facet. most are 14 lines long, but one is 12 and another 15. this means we can not just split using a positive index number. luckily slice allows accepts negative index numbers counting from the last item in the array: 
slice(split(value, /\n/),-2).join("\n") 

clean up white space:
first column, use built in 
more complicated for sonnet,
trim(string s) 
forEach(split(value, /\n/),e,e.trim()).join("\n")

reorder columns 

explore data with numeric facet 

export 