API def, web services
refine extensions, https://github.com/RubenVerborgh/Refine-NER-Extension 
problem = changing API companies, parameters, difficult to maintain and update
solution = learn apis, manually construct queries
"A RESTful API is an application program interface (API) that uses HTTP requests to do things with external data."

we are going to look at the Chronicling America api, http://chroniclingamerica.loc.gov/about/api/ 
and use it to construct a topical corpus of newspaper articles.
the API documentation is a like a recipe book, telling us how we can interact with with server from a public URL. 
Chronam is full open, no key or account is needed to access the API, and no terms on the use. 
Other API are often proprietary and restricted, with specific terms of use. 

The documentation tells us the base url is `http://chroniclingamerica.loc.gov/`. We add additional elements to this url to interact with the API. 

build a newspaper corpus:
- Chronicling America api, http://chroniclingamerica.loc.gov/about/api/ 
- help, http://chroniclingamerica.loc.gov/help/ 
- Start with query, build url, fetch
- parse individual resources
- grab item metadata
- parse to grab full text
- geocode 

view source, hints in code, `<link rel="alternate" type="application/json"`

http://chroniclingamerica.loc.gov/search/pages/results/?state=&date1=1789&date2=1924&proxtext=moscow+id&x=0&y=0&dateFilterType=yearRange&rows=20&searchType=basic

advance search
http://chroniclingamerica.loc.gov/search/pages/results/?
date range:
dateFilterType=range&date1=1850&date2=1870
any of words:
&ortext=anyof
with all of:
&andtext=allof
with phrase:
&phrasetext=thephrase
with the words:
&proxtext=withwords
with in _ words of each other:
&proxdistance=5

&rows=20

&searchType=advanced

http://chroniclingamerica.loc.gov/search/pages/results/?dateFilterType=range&date1=1850&date2=1870&language=&ortext=muffin+dog&andtext=allegator+breath&phrasetext=dogs+and+cats&proxtext=chunky+lick&proxdistance=5&rows=20&searchType=advanced

http://chroniclingamerica.loc.gov/search/pages/results/?state=&date1=1789&date2=1924&proxtext=chunky+lick&x=0&y=0&dateFilterType=yearRange&rows=20&searchType=basic

built in documenation listed in the html <head>:
<link title="NDNP Page Search" href="/search/pages/opensearch.xml"
          rel="search" type="application/opensearchdescription+xml" />

Gives this document:
<?xml version="1.0" encoding="UTF-8"?>
<OpenSearchDescription 
    xmlns="http://a9.com/-/spec/opensearch/1.1/"
    xmlns:chronam="http://chroniclingamerica.loc.gov">
    <ShortName>Chronicling America Pages Search</ShortName>
    <Description>Search digital newspaper content from the Library of Congress</Description>
    <InputEncoding>UTF-8</InputEncoding>
    <Image width="16" height="16" type="image/x-icon">http://loc.gov/favicon.ico</Image>
    <Url type="text/html" template="http://chroniclingamerica.loc.gov/search/pages/results/?andtext={searchTerms}&amp;page={startPage?}&amp;ortext={chronam:booleanOrText?}&amp;year={chronam:year?}&amp;date1={chronam:date?}&amp;date2={chronam:date?}&amp;phrasetext={chronam:phraseText?}&amp;proxText={chronam:proxText?}&amp;proximityValue={chronam:proximityValue?}" />
    <Url type="application/atom+xml" template="http://chroniclingamerica.loc.gov/search/pages/results/?andtext={searchTerms}&amp;page={startPage?}&amp;ortext={chronam:booleanOrText?}&amp;year={chronam:year?}&amp;date1={chronam:date?}&amp;date2={chronam:date?}&amp;phrasetext={chronam:phraseText?}&amp;proxText={chronam:proxText?}&amp;proximityValue={chronam:proximityValue?}&amp;format=atom" />
    <Url type="application/json" template="http://chroniclingamerica.loc.gov/search/pages/results/?andtext={searchTerms}&amp;page={startPage?}&amp;ortext={chronam:booleanOrText?}&amp;year={chronam:year?}&amp;date1={chronam:date?}&amp;date2={chronam:date?}&amp;phrasetext={chronam:phraseText?}&amp;proxText={chronam:proxText?}&amp;proximityValue={chronam:proximityValue?}&amp;format=json" />
</OpenSearchDescription>

http://chroniclingamerica.loc.gov/search/pages/results/?date1=1864&amp;date2=1866&amp;phrasetext=boise+capitol&amp;proxText=boise+capitol&amp;proximityValue=6&amp;format=json

idaho front pages 1865
http://chroniclingamerica.loc.gov/search/pages/results/?state=Idaho&dateFilterType=yearRange&date1=1865&date2=1865&sequence=1&language=&ortext=&andtext=&phrasetext=&proxtext=&proxdistance=5&rows=20&searchType=advanced

http://chroniclingamerica.loc.gov/search/pages/results/?state=Idaho&year=1865&sequence=1

http://chroniclingamerica.loc.gov/search/pages/results/?state=Idaho&date1=1865&date2=1865&proxtext=&x=10&y=10&dateFilterType=yearRange&rows=100&searchType=basic&format=json

create via clipboard:

state,year
Idaho,1865
Washington,1865

new column url:
"http://chroniclingamerica.loc.gov/search/pages/results/?state=" + value + "&date1=" + cells.year.value + "&date2="+ cells.year.value + "&dateFilterType=yearRange&sequence=1&rows=100&format=json"

escape(value, 'url')

new column by fetch json_search.

value.parseJson()["Writer"]

split the items ( parseJson(string s).get("a") ), new column items:
value.parseJson().items.join(":::")
split multivalue cells :::
remove extra columns, state fill down

new columns with values:
date: value.parseJson().date
title: value.parseJson().title
city: value.parseJson().city.join(", ")
lccn: value.parseJson().lccn
text: value.parseJson().ocr_eng

new column issue_url based on lccn:
"http://chroniclingamerica.loc.gov/lccn/" + value + "/" + cells.date.value.splitByLengths(4,2,2).join("-") + "/ed-1/"

