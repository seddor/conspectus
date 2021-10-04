# When to use Entity identifiers

Для идентификации ресурсов рекомендуется использовать URN.

Например ИД юзера 1234, а его аддреса 4567, тогда:
```xml
<person xmlns:atom="http://www.w3.org/2005/Atom">
    <atom:link rel="self" href="http://example.org/person/john"/>
    <id>urn:example:user:1234</id>
    <name>John Doe</name>
    <address>
        <id>urn:example:address:4567</id>
        <street>1 Main Street</street>
        <city>Seattle</city>
        <state>WA</state>
    </address>
</person>
```
