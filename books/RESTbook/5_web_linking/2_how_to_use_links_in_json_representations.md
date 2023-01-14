# How to use links in JSON representations

Для ссылок нужно использовать поле `link` или `links` для множеста ссылок, поля такие же как в (5.1).
```json
{
    "link" : {
        "rel" : "alternate",
        "href" : "http://east-nj1.photos.example.org/987/nj1-1234",
    },
    "links" : [
        {
            "rel" : "alternate",
            "href" : "http://east-nj1.photos.example.org/987/nj1-1234"
        },
        {
        "rel" : "http://www.example.org/rels/owner",
        "href" : "http://east-nj1.photos.example.org/987",
        }
    ]
}
```

Также можно использовать прямые URI.
