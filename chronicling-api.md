API def, web services
refine extensions, https://github.com/RubenVerborgh/Refine-NER-Extension 
problem = changing API companies, parameters, difficult to maintain and update
solution = learn apis, manually construct queries

we are going to look at the Chronicling America api, http://chroniclingamerica.loc.gov/about/api/ 
and use it to construct a topical corpus of newspaper articles.
the API documentation is a like a recipe book, telling us how we can interact with with server from a public URL. 
Chronam is full open, no key or account is needed to access the API, and no terms on the use. 
Other API are often proprietary and restricted, with specific terms of use. 

The documentation tells us the base url is ` http://chroniclingamerica.loc.gov/`. We add additional elements to this url to interact with the API. 

build a newspaper corpus:
- Chronicling America api, http://chroniclingamerica.loc.gov/about/api/ 
- Start with query, build url, fetch
- parse individual resources
- grab item metadata
- parse to grab full text
- geocode 

view source, hints in code, `<link rel="alternate" type="application/json"`

http://chroniclingamerica.loc.gov/search/pages/results/?state=&date1=1789&date2=1924&proxtext=moscow+id&x=0&y=0&dateFilterType=yearRange&rows=20&searchType=basic
