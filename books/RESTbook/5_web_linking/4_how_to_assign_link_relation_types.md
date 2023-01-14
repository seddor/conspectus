# How to Assign Link Relation Types

Основные типы:
* self — сслыка на представление ресурса
* alternate — используется для ссылки на альтернативную версию представления этого же ресурса
* edit — сслыка на редактирование ресурса
* related — ссылка на связанный ресурс
* previous/next — сслыка на предыдущий/следующий ресерс в упорядоенной коллекции ресурсов(например для переключения страниц)
* first/last — сслыка на первый/последний ресерс в упорядоенной коллекции ресурсов(например для переключения страниц)

HTML 4.01 определяет такие типа: alternate, stylesheet, start, next, prev, contents, index, glossary, copyright, chapter, section, subsection, appendix, help и bookmark, HTML 5 добавляет ещё archives, feed, pingback и т.п. 

Также можно использовать несколько типов в одной rel(`rel="alternate help"`)
