# Extracting multiple variables in wide format

Building on the [recipe for one variable](README.md), extracting multiple variables to a CSV file in [wide format](https://en.wikipedia.org/wiki/Wide_and_narrow_data#Wide)
(as opposed to the "narrow" or "long" format in `data.csv`) can be done as follows.

We'll use [`csvsql`](https://csvkit.readthedocs.io/en/1.0.3/scripts/csvsql.html) again, and since typing long SQL queries on the
command line is no fun, exploit its functionality to pass the query in a file. So with an SQL file `query.sql` with the following
content
```sql
select 
    s.id, s.pref_name_for_society,
    d1.code as EA006,
    d2.code as EA007 
from 
    data as d1,
    data as d2,
    societies as s
where
    s.id = d1.soc_id and s.id = d2.soc_id
    and d1.var_id = 'EA006'
    and d2.var_id = 'EA007'
```
we can run
```bash
csvsql --query query.sql \
dplace-data/datasets/EA/data.csv \
dplace-data/datasets/EA/societies.csv > EA_006_007.csv
```
to create a CSV file looking like
```
id,pref_name_for_society,EA006,EA007
Aa1,!Kung,2,8
Aa2,Dorobo,1,8
Aa3,Nama,4,1
Aa4,Bergdama,2,8
Aa5,Mbuti,5,1
```
