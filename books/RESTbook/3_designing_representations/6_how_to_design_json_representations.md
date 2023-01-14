# How to Design JSON Representations

Рекомендации сходи с XML:
В каждом представлении нужно вклбючать self link. 
Для полей на нескольких языках стоит добавлять поле с языком.
```json
{
    "name" : "John",
    "id" : "urn:example:user:1234",
    "link" : {
        "rel : "self",
        "href" : "http://www.example.org/person/john"
    },
    "address" : {
        "id" : "urn:example:address:4567",
        "link" : {
            "rel : "self",
            "href" : "http://www.example.org/person/john/address"
        }
    }
}
```
```json
{
    "content" : {
        "text" : [
            {
                "value" : "The quick brown fox jumps over the lazy dog."
            },
            {
                "lang" : "en-GB",
                "value" : "What colour is it"
            },
            {
                "lang" : "en-US",
                "value" : "What color is it"
            }
        ]
    }
}
```
