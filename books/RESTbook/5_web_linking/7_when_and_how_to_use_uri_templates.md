When and How to Use URI Templates

Для значений в щаблонах ссылок нужно использовать `{}`:
* http://www.example.org/segment1/{token1}/segment2
* http://www.example.org/path?param1={p1}&param2={p2}
* http://www.example.org/path;param1={p1};param2={p2}

Для XML стоит использовать элемент `link-template`:
```xml
<link-template 
    href="http://www.example.org/customers/{customer-id}"
    title="View customer detail"
    rel="http://www.example.org/rels/detail"
/>
<link-template 
    href="http://www.example.org/search/k={keyword}&p={page-number}&r={results-per-page}"
    title="Search results"
    rel="http://www.example.org/rels/search"
/>
```
Для JSON:
```json
"link-templates": [
    {
        "rel": "http://www.example.org/rels/detail",
        "href": "http://www.example.org/customers/{customer-id}",
        "title": "View customer detail"
    },
    {
        "rel" : "http://www.example.org/rels/search",
        "href" : "http://www.example.org/search/k={keyword}&p={page-number}&r={results-per-page}",
        "title" : "Search results"
    }
]
```
