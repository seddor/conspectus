# When to use PUT to create new resources

PUT стоит использовать для создания новых ресурсов, только когда клиент может определять URI ресурса, в остальных случаях нужно использовать POST.

Пример:
```
# Request
PUT /user/smith/address/home_address HTTP/1.1
Host: www.example.org
Content-Type: application/xml;charset=UTF-8
<address>
    <street>1, Main Street</street>
    <city>Some City</city>
</address>

# Response
HTTP/1.1 201 Created
Location: http://www.example.org/user/smith/address/home_address
Content-Location: http://www.example.org/user/smith/address/home_address
Content-Type: application/xml;charset=UTF-8
<address>
    <id>urn:example:user:smith:address:1</id>
    <atom:link rel="self" href="http://www.example.org/user/smith/address/home_address"/>
    <street>1, Main Street</street>
    <city>Some City</city>
</address>
```
