# How to Design URIs for queries

Query-параметры используются для

* фильтрации 

  ```
  http://www.example.org/book/978-0374292881/reviews?limit=5
  http://www.example.org/book/978-0374292881/reviews?after=2009-08-15
  ```

  

*  сортировки

  ```
  http://www.example.org/book/978-0374292881/reviews?sortbyAsc=date
  http://www.example.org/book/978-0374292881/reviews?sortbyDesc=created
  ```

  

*  указания списка полей ресурса, которые должны быть включены в ответ

  ```
  http://www.example.org/book/978-0374292881/reviews?sortbyAsc=date&fields=title
  ```

  

Query-параметры оцианальны и использую дефолтные значения. 

