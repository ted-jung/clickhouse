#### =============================================================================
#### Title: dbt(data build tool)
#### Writer: Ted Jung
#### Created: 23, Apr 2025
#### Updated: 23, Apr 2025
#### Description: 
####   Data build tool for ELT(especially, Transformation)
####   Model Schema -> Create a View via dbt
#### =============================================================================


##### Install dbt tool and make a directory and move into the directory

```
> pip install dbt-clickhouse
> mkdir dbt
> cd dbt
```


#### Initialize a directory for new and edit a profile
#### Database to connect and use database(imdb_dbt)
#### Validate the profile

```
> dbt init imdb
> vi ~/.dbt/profiles.yml
```

```
clickhouse_imdb:        <= here, name
  target: dev
  outputs:
    dev:
      type: clickhouse
      schema: imdb_dbt
      host: localhost
      port: 8123
      user: default
      password: ''
      secure: False
```

> dbt debug



#### Init working environment

```
> cd imdb
> vi dbt_project.yml
```

```
# Name your project! Project names should contain only lowercase characters and underscores. 
# A good package name should reflect your organization's name or the intended use of these models
name: 'imdb'
version: '1.0.0'

# This setting configures which "profile" dbt uses for this project.
# the same name at "~/.dbt/profiles.yml"
profile: 'clickhouse_imdb'                       <= here, name

# These configurations specify where dbt should look for different types of files.
# The `model-paths` config, for example, states that models in this project can be
# found in the "models/" directory. You probably won't need to change these!
model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

clean-targets:         #### directories to be removed by `dbt clean`
  - "target"
  - "dbt_packages"


# Configuring models
# Full documentation: https://docs.getdbt.com/docs/configuring-models
# In this example config, we tell dbt to build all models in the example/
# directory as views. These settings can be overridden in the individual model
# files using the `{{ config(...) }}` macro.
models:
  imdb:
    #### Config indicated by + and applies to all files under models/actors/
    actors:
      +materialized: view
```


#### Create two files(schema: yml , dml: sql) for processing
#### schema: refer to database
#### dml: for creating view

```
> rm -rf dbt_root_path/models/examples
> mkdir dbt_root_path/models/actors
> cd dbt_root_path/models/actors
> create schema.yml
```

```
version: 2

sources:
- name: imdb
  tables:
  - name: directors
  - name: actors
  - name: roles
  - name: movies
  - name: genres
  - name: movie_directors
```

```
> create actors_summary.sql
```

```
{{ config(materialized='view') }}

with actor_summary as (
SELECT id,
       any(actor_name)          as name,
       uniqExact(movie_id)      as num_movies,
       avg(rank)                as avg_rank,
       uniqExact(genre)         as genres,
       uniqExact(director_name) as directors,
       max(created_at) as updated_at
FROM (
       SELECT {{ source('imdb', 'actors') }}.id    as id,
               concat({{ source('imdb', 'actors') }}.first_name, ' ', {{ source('imdb', 'actors') }}.last_name) as actor_name,
              {{ source('imdb', 'movies') }}.id   as movie_id,
              {{ source('imdb', 'movies') }}.rank as rank,
               genre,
               concat({{ source('imdb', 'directors') }}.first_name, ' ', {{ source('imdb', 'directors') }}.last_name) as director_name,
               created_at
       FROM {{ source('imdb', 'actors') }}
               JOIN {{ source('imdb', 'roles') }} ON {{ source('imdb', 'roles') }}.actor_id = {{ source('imdb', 'actors') }}.id
               LEFT OUTER JOIN {{ source('imdb', 'movies') }} ON {{ source('imdb', 'movies') }}.id = {{ source('imdb', 'roles') }}.movie_id
               LEFT OUTER JOIN {{ source('imdb', 'genres') }} ON {{ source('imdb', 'genres') }}.movie_id = {{ source('imdb', 'movies') }}.id
               LEFT OUTER JOIN {{ source('imdb', 'movie_directors') }} ON {{ source('imdb', 'movie_directors') }}.movie_id = {{ source('imdb', 'movies') }}.id
               LEFT OUTER JOIN {{ source('imdb', 'directors') }} ON {{ source('imdb', 'directors') }}.id = {{ source('imdb', 'movie_directors') }}.director_id
       )
GROUP BY id
)

select *
from actor_summary
```

```
> vi dbt_project.yml
> dbt run
```


#### Table example (create table)

```
> vi actors_summary.sql
```

```
{{ config(order_by='(updated_at, id, name)', engine='MergeTree()', materialized='table') }}

with actor_summary as (
SELECT id,
       any(actor_name)          as name,
       uniqExact(movie_id)      as num_movies,
       avg(rank)                as avg_rank,
       uniqExact(genre)         as genres,
       uniqExact(director_name) as directors,
       max(created_at) as updated_at
FROM (
       SELECT {{ source('imdb', 'actors') }}.id    as id,
               concat({{ source('imdb', 'actors') }}.first_name, ' ', {{ source('imdb', 'actors') }}.last_name) as actor_name,
              {{ source('imdb', 'movies') }}.id   as movie_id,
              {{ source('imdb', 'movies') }}.rank as rank,
               genre,
               concat({{ source('imdb', 'directors') }}.first_name, ' ', {{ source('imdb', 'directors') }}.last_name) as director_name,
               created_at
       FROM {{ source('imdb', 'actors') }}
               JOIN {{ source('imdb', 'roles') }} ON {{ source('imdb', 'roles') }}.actor_id = {{ source('imdb', 'actors') }}.id
               LEFT OUTER JOIN {{ source('imdb', 'movies') }} ON {{ source('imdb', 'movies') }}.id = {{ source('imdb', 'roles') }}.movie_id
               LEFT OUTER JOIN {{ source('imdb', 'genres') }} ON {{ source('imdb', 'genres') }}.movie_id = {{ source('imdb', 'movies') }}.id
               LEFT OUTER JOIN {{ source('imdb', 'movie_directors') }} ON {{ source('imdb', 'movie_directors') }}.movie_id = {{ source('imdb', 'movies') }}.id
               LEFT OUTER JOIN {{ source('imdb', 'directors') }} ON {{ source('imdb', 'directors') }}.id = {{ source('imdb', 'movie_directors') }}.director_id
       )
GROUP BY id
)

select *
from actor_summary
```

```
> dbt run
```

#### Another table incremental
#### Run this to test

```
> edit actors_summay.sql
```

```
{{ config(order_by='(updated_at, id, name)', engine='MergeTree()', materialized='incremental', unique_key='id') }}
with actor_summary as (
    SELECT id,
        any(actor_name) as name,
        uniqExact(movie_id)    as num_movies,
        avg(rank)                as avg_rank,
        uniqExact(genre)         as genres,
        uniqExact(director_name) as directors,
        max(created_at) as updated_at
    FROM (
        SELECT {{ source('imdb', 'actors') }}.id as id,
            concat({{ source('imdb', 'actors') }}.first_name, ' ', {{ source('imdb', 'actors') }}.last_name) as actor_name,
            {{ source('imdb', 'movies') }}.id as movie_id,
            {{ source('imdb', 'movies') }}.rank as rank,
            genre,
            concat({{ source('imdb', 'directors') }}.first_name, ' ', {{ source('imdb', 'directors') }}.last_name) as director_name,
            created_at
    FROM {{ source('imdb', 'actors') }}
        JOIN {{ source('imdb', 'roles') }} ON {{ source('imdb', 'roles') }}.actor_id = {{ source('imdb', 'actors') }}.id
        LEFT OUTER JOIN {{ source('imdb', 'movies') }} ON {{ source('imdb', 'movies') }}.id = {{ source('imdb', 'roles') }}.movie_id
        LEFT OUTER JOIN {{ source('imdb', 'genres') }} ON {{ source('imdb', 'genres') }}.movie_id = {{ source('imdb', 'movies') }}.id
        LEFT OUTER JOIN {{ source('imdb', 'movie_directors') }} ON {{ source('imdb', 'movie_directors') }}.movie_id = {{ source('imdb', 'movies') }}.id
        LEFT OUTER JOIN {{ source('imdb', 'directors') }} ON {{ source('imdb', 'directors') }}.id = {{ source('imdb', 'movie_directors') }}.director_id
    )
    GROUP BY id
)
select *
from actor_summary

{% if is_incremental() %}

-- this filter will only be applied on an incremental run
where id > (select max(id) from {{ this }}) or updated_at > (select max(updated_at) from {{this}})

{% endif %}
```

```
> dbt run
```

#### insert a new data  like below

```
INSERT INTO imdb.actors VALUES (845466, 'Clicky', 'McClickHouse', 'M');

INSERT INTO imdb.roles (created_at, actor_id, movie_id, `role`)
SELECT now() as created_at, 845466 as actor_id, id as movie_id, 'Himself' as `role`
FROM imdb.movies
LIMIT 910 OFFSET 10000;
```


#### Test - Append only mode

```
{{ config(order_by='(updated_at, id, name)', engine='MergeTree()', materialized='incremental', unique_key='id', incremental_strategy='append') }}
```

```
> dbt run
```

```
INSERT INTO imdb.actors VALUES (845467, 'Danny', 'DeBito', 'M');

INSERT INTO imdb.roles (created_at, actor_id, movie_id, `role`)
SELECT now() as created_at, 845467 as actor_id, id as movie_id, 'Himself' as role
FROM imdb.movies
LIMIT 920 OFFSET 10000;

SELECT * FROM imdb_dbt.actors_summary ORDER BY num_movies DESC LIMIT 3;
```

```
> dbt run
```
