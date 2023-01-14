# How to Design XML Representations

В каждом представлении нужно включать self link. 
Если части предсталения содержат текст на разных языках, то стоит добавить xml:lang для идентификации языка.

Пример:
```
# Request
GET /user/smith/address/0 HTTP/1.1
Host: www.example.org

# Response
HTTP/1.1 200 OK
Content-Type: application/xml;charset=UTF-8
<address>
    <id>urn:example:user:smith:address:0</id>
    <atom:link rel="self" href="http://www.example.org/user/smith/address/0"/>
    <street>1, Olympia Dr</street>
    <city>Some City</city>
</address>
```
```
# Response
HTTP/1.1 200 OK
Content-Type: application/xml;charset=UTF-8
Content-Language: en
<content>
    <text>The quick brown fox jumps over the lazy dog.</text>
    <text xml:lang="en-GB">What colour is it?</text>
    <text xml:lang="en-US">What color is it?</text>
    <text xml:lang="de">
        <p>Habe nun, ach! Philosophie,</p>
        <p>Juristerei, und Medizin</p>
        <p>und leider auch Theologie</p>
        <p>durchaus studiert mit heißem Bemüh'n.</p>
    </text>
</content>
```
