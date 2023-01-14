# Использование подзапросов

## Как источник данных

Подзапросы можно использовать в блоке `FROM`:
```SQL
SELECT d.dept_id, d.name, e_cnt.how_many num_employees
    FROM department d INNER JOIN
    (SELECT dept_id, COUNT(*) how_many
    FROM employee
    GROUP BY dept_id) e_cnt
    ON d.dept_id = e_cnt.dept_id;
```

### Подзапросы в условиях фильтрации

Одно из основных применений подзапросов.
