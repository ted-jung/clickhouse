## ranking

```
CREATE OR REPLACE VIEW salaries AS 
FROM file('salaries.csv')
SELECT * EXCEPT (weeklySalary), weeklySalary AS salary;


select *,
       row_number() OVER() as rowNum
from salaries
LIMIT 10;


SELECT *,
       row_number() OVER () AS rowNum,
       rank() OVER (ORDER BY salary DESC) AS rank,
       dense_rank() OVER (ORDER BY salary DESC) AS denseRank
FROM salaries
LIMIT 10;


SELECT *,
       rank() OVER (ORDER BY salary DESC) AS rank,
       rank() OVER (PARTITION BY team ORDER BY salary DESC) AS teamRank,
       rank() OVER (PARTITION BY position ORDER BY salary DESC) AS posRank
FROM salaries
ORDER BY salary DESC
LIMIT 10;


WITH windowedSalaries AS
    (
        SELECT
            *,
            rank() OVER (ORDER BY salary DESC) AS rank,
            rank() OVER (PARTITION BY team ORDER BY salary DESC) AS teamRank,
            rank() OVER (PARTITION BY position ORDER BY salary DESC) AS posRank
        FROM salaries
        ORDER BY salary DESC
    )
SELECT
    player,
    position,
    salary,
    bar(salary, 0, (
        SELECT max(salary)
        FROM windowedSalaries
        LIMIT 1
    ), 10) AS plot,
    teamRank,
    posRank,
    rank
FROM windowedSalaries
WHERE team LIKE '%Claireberg Vikings%'
LIMIT 15;
```
