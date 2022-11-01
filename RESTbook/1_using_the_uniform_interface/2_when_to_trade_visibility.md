# When to Trade Visibility

В некоторых ситуация явностью приходится жертвовать. Например, один ресурс может содержать данные другого ресурса, и при изменении одного ресраса зависимый должен тоже обновится, однако со стороны HTTP все ресурсы не зависмы, об их зависимости знает только сервер. 

Также ясностью жертвуют в следующих случаев:

* Большее удобство для клиента, при работе со сложными ресурсами
* Для сложный операций, которые требует бизнес логика
* При необходимости оптимизации тяжёлых запросов