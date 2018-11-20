# How to extract data for selected variables from D-PLACE?

Data in D-PLACE is organized in [datasets](https://github.com/D-PLACE/dplace-data/blob/master/datasets/README.md). Thus, the first step
towards extracting data for a variable is locating the dataset the variable belongs to. If you found a variable of interest to you
in the web app, e.g. [EA006](https://d-place.org/parameters/EA006), you already have all you need: The variable ID `EA006` and a
link to the dataset it belongs to, the [Ethnographic Atlas](https://d-place.org/contributions/EA) with ID `EA`. So the datapoints
for variable `EA006` must be extracted from all datapoints of the dataset `EA`, located at [`datasets/EA/data.csv`](https://github.com/D-PLACE/dplace-data/blob/master/datasets/EA/data.csv).

Since all data for a dataset is available as a set of [CSV files](https://en.wikipedia.org/wiki/Comma-separated_values), you can
use your favorite CSV tool to extract and further manipulate the data you are interested in. In the following, we describe how to
proceed with the tools from [csvkit](https://csvkit.readthedocs.io/en/1.0.3/index.html) - a suite of [command line tools](http://swcarpentry.github.io/shell-novice/)
to work with CSV.

So how to extract a table of values for a variable like [the one displayed in the web app](https://d-place.org/parameters/EA006#table-container)
using csvkit?

1. Let's inspect what `data.csv` looks like:
   ```bash
   $ head -n 2 dplace-data/datasets/EA/data.csv 
   soc_id,sub_case,year,var_id,code,comment,references,source_coded_data,admin_comment
   Aa1,Nyai Nyae region,1950,EA001,8,,biesele1972; biesele1975; biesele1976; draper1972; draper1975; drapernd; hansenetal1969; harpending1971; howell1979; howellnd; konner1971; konner1972; konner1973; konner1977; lee1966; lee1968; lee1972a; lee1974; lee1979; leeanddevore1976; marshall1956; marshall1957; marshall1957a; marshall1957b; marshall1958; marshall1959; marshall1960; marshall1961; marshall1962; marshall1965; marshall1976; marshallandmarshall1956; schapera1930; shostak1981; thomas1959; tobias1978,EthnographicAtlas_1967_p62,
   ```
   `var_id` is the name of the column holding the variable IDs. Thus, we can extract the subset we are interested in running
   ```bash
   $ csvgrep -c var_id -m EA006 dplace-data/datasets/EA/data.csv > EA006.csv
   ```
2. Inspecting the CSV file we just created with `csvstat` yields
```bash
$ csvstat EA006.csv   
  1. "soc_id"

	Type of data:          Text
	Contains null values:  False
	Unique values:         1291
	...

  2. "sub_case"

	Type of data:          Text
	Contains null values:  True (excluded from calculations)
	Unique values:         439
	Most common values:    None (852x)
	                       Those in contact with mission (2x)
	                       Nyai Nyae region (1x)
	                       with special reference to Central Dorobo (1x)
	                       Gei/Khauan tribe (1x)

  3. "year"

	Type of data:          Number
	Contains null values:  True (excluded from calculations)
	Unique values:         68
	Smallest value:        -2.000
	Largest value:         1.965
	Most common values:    1.920 (193x)
	                       1.950 (166x)
	                       1.930 (166x)
	                       1.900 (120x)
	                       1.910 (112x)

  ...

  5. "code"

	Type of data:          Number
	Contains null values:  True (excluded from calculations)
	Unique values:         8
	Smallest value:        1
	Largest value:         7
	Most common values:    1 (657x)
	                       6 (275x)
	                       2 (125x)
	                       3 (68x)
	                       4 (65x)

  6. "comment"

	Type of data:          Text
	Contains null values:  True (excluded from calculations)

  7. "references"

	Type of data:          Text
	Contains null values:  True (excluded from calculations)
	
  ...

Row count: 1291
```
Notes:
- Since the `soc_id` column contains 1291 unique values (in 1291 rows), there's exactly one value per society for this variable. 
  (This is not generally the case for D-PLACE variables.)
- Our file contains more rows than th table in the web app, but this discrepancy can be explained by the web app omitting `NA`
  values:
  ```
  $ csvgrep -c code -m NA EA006.csv 
  soc_id,sub_case,year,var_id,code,comment,references,source_coded_data,admin_comment
  Ca27,,,EA006,NA,,,,
  Cc15,,,EA006,NA,,,,
  ...
  ```
- Our file only contains numeric codes, not the labels (e.g. "Bride-wealth").

Adding the code labels is easy, though, using the really powerful [`csvsql`](https://csvkit.readthedocs.io/en/1.0.3/scripts/csvsql.html) command:
```bash
$ csvsql --query "select d.soc_id, d.sub_case, d.year, d.code, c.name, c.description, d.comment from 'EA006' as d, 'codes' as c where d.var_id = c.var_id and d.code = c.code" EA006.csv dplace-data/datasets/EA/codes.csv > EA006_with_code_names.csv
```
Here, we join two CSV files on matching `var_id` and `code`, and select the columns we are interested in.

Note that `csvsql` can be used to perfom 1. and 2. above, and additionally pulling in society names:
```bash
$ csvsql --query "select d.soc_id, s.pref_name_for_society, d.sub_case, d.year, d.code, c.name from data as d, codes as c, societies as s where d.var_id = c.var_id and d.code = c.code and s.id = d.soc_id and d.var_id = 'EA006'" \
dplace-data/datasets/EA/data.csv \
dplace-data/datasets/EA/codes.csv \
dplace-data/datasets/EA/societies.csv > EA006.csv
```

