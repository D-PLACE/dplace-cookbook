# How to extract data for selected variables from D-PLACE?

The first step towards extracting data for a variable is finding its identifier. If you found a variable of interest to you
in the web app, e.g. [EA006](https://d-place.org/parameters/EA006), you already have all you need: The variable ID `EA006`. 

Since CLDF data is available as a set of [CSV files](https://en.wikipedia.org/wiki/Comma-separated_values), you can
use your favorite CSV tool to extract and further manipulate the data you are interested in. In the following, we describe how to
proceed with the tools from [csvkit](https://csvkit.readthedocs.io/en/1.0.3/index.html) - a suite of [command line tools](http://swcarpentry.github.io/shell-novice/)
to work with CSV.

So how to extract a table of values for a variable like [the one displayed in the web app](https://d-place.org/parameters/EA006#table-container)
using csvkit?

Let's inspect what `cldf/data.csv` looks like:
```shell
$ head -n 2 cldf/data.csv
ID,Soc_ID,Var_ID,Value,Code_ID,Comment,Source,sub_case,year,source_coded_data,admin_comment
binford-1,B1,B001,65,,,avadhani1975;brosius1986;harrison1949;kedit1982;oldrey1975;sellato1994;urquhart1951,,1970,Binford_2001_Table_5.01,
```

`Var_ID` is the name of the column holding the variable IDs, so
we can get an initial overview of the data for the variable running a command such as
```shell
$ csvgrep -c Var_ID -m EA006 cldf/data.csv | csvstat -c Value -y 0 --freq-count 10
  4. "Value"

        Type of data:          Text
        Contains null values:  True (excluded from calculations)
        Unique values:         8
        Longest value:         18, characters
        Most common values:    Bride-wealth (657x)
                               Insignificant (275x)
                               Bride-service (125x)
                               Token bride-wealth (68x)
                               Gift exchange (65x)
                               Dowry (43x)
                               Woman exchange (39x)
                               None (19x)

Row count: 1291
```

Note that the 19 rows with `NULL` values (aka `None` or `NA`) are not loaded into the web app. Thus,
the web app shows 1272 rows for the values of `EA006`.

`Soc_ID` is the name of the column linking values to societies. Let's have a look at it:
```shell
$ csvgrep -c Var_ID -m EA006 ../dplace-cldf/cldf/data.csv | csvstat -c Soc_ID -y 0
  2. "Soc_ID"

        Type of data:          Text
        Contains null values:  False
        Unique values:         1291
        Longest value:         5, characters
        Most common values:    Aa1 (1x)
                               Aa2 (1x)
                               Aa3 (1x)
                               Aa4 (1x)
                               Aa5 (1x)

Row count: 1291
```
Since the `Soc_ID` column contains 1291 unique values (in 1291 rows), there's exactly one value per society for this variable. 
(This is not generally the case for D-PLACE variables.)

Let's use the really powerful [`csvsql`](https://csvkit.readthedocs.io/en/1.0.3/scripts/csvsql.html) command to
pull in society names:
```shell
$ csvsql --query "select d.Value, s.Name, d.year, d.sub_case from data as d, societies as s where d.Value is not null and d.Soc_ID = s.ID and d.Var_ID = 'EA006' order by d.Code_ID, s.Name" cldf/data.csv cldf/societies.csv 
```
The resulting rows look already a lot like the table in the web app:

Value | Name | year | sub_case
--- | --- | --- | ---
Bride-wealth | Ababda | 1920 | 
Bride-wealth | Abarambo | 1890 | 
Bride-wealth | Abelam | 1930 | Kalabu; Northern Abelam
Bride-wealth | Abipón | 1800 | Those in contact with mission
Bride-wealth | Abron | 1900 | 
Bride-wealth | Acholi | 1920 | 
Bride-wealth | Achumawi | 1860 | 
Bride-wealth | Adangme | 1940 | with special reference to the Krobo
Bride-wealth | Adi | 1940 | 
Bride-wealth | Afar | 1880 | 

The only piece of data we are missing is the name of the top-level language family associated with the
language spoken by the society.

We can pull in this information by looking it up in the [Glottolog CLDF dataset](https://doi.org/10.5281/zenodo.8131091).
After downloading and unzipping the release data, we can look up the family names with a query like
```sql
SELECT
    d.Value, s.Name, l2.Name, d.year, d.sub_case 
FROM 
    data AS d, 
    societies AS s, 
    languages AS l1 
LEFT JOIN languages AS l2 
    ON l1.Family_ID = l2.ID 
WHERE 
    s.Glottocode = l1.ID 
    AND d.Value IS NOT NULL 
    AND d.Soc_ID = s.ID 
    AND d.Var_ID = 'EA006' 
ORDER BY d.Code_ID, s.Name
```
as
```shell
$ csvsql --query "select d.Value, s.Name, l2.Name, d.year, d.sub_case from data as d, societies as s, languages as l1 left join languages as l2 on l1.Family_ID = l2.ID where s.Glottocode = l1.ID and d.Value is not null and d.Soc_ID = s.ID and d.Var_ID = 'EA006' order by d.Code_ID, s.Name limit 20" ../dplace-cldf/cldf/data.csv ../dplace-cldf/cldf/societies.csv glottolog-cldf/cldf/languages.csv
```

Value | Name               | Name | year | sub_case
--- |--------------------| --- | --- | ---
Bride-wealth | Ababda             | Afro-Asiatic | 1920 | 
Bride-wealth | Abarambo           | Atlantic-Congo | 1890 | 
Bride-wealth | Abelam             | Ndu | 1930 | Kalabu; Northern Abelam
Bride-wealth | Abipón             | Guaicuruan | 1800 | Those in contact with mission
Bride-wealth | Abron              | Atlantic-Congo | 1900 | 
Bride-wealth | Acholi             | Nilotic | 1920 | 
Bride-wealth | Achumawi           | Palaihnihan | 1860 | 
Bride-wealth | Adangme            | Atlantic-Congo | 1940 | with special reference to the Krobo
Bride-wealth | Adi                | Sino-Tibetan | 1940 | 
Bride-wealth | Afar               | Afro-Asiatic | 1880 | 
Bride-wealth | Afikpo             | Atlantic-Congo | 1950 | with special reference to the village of Mgbom
Bride-wealth | Akyem              | Atlantic-Congo | 1930 | with special reference to the Kwahu
Bride-wealth | Algerians          | Afro-Asiatic | 1870 | 
Bride-wealth | Alorese            | Timor-Alor-Pantar | 1940 | Abui of Atimelang Village
Bride-wealth | Alsea              |  | 1860 | 
Bride-wealth | Alur               | Nilotic | 1890 | 
Bride-wealth | Amarar             | Afro-Asiatic | 1930 | 
Bride-wealth | Ambo               | Atlantic-Congo | 1910 | with special reference to the Kuanyama
Bride-wealth | Ambonese           | Austronesian | 1950 | Allang
Bride-wealth | Ancient Egyptians  | Afro-Asiatic | -1400 | with special reference to the New Empire


Note that we had to join Glottolog's `languages.csv` twice - one time to find the family Glottocode
associated with a language and then to look up the name of the family. The `LEFT JOIN` used for the
second join makes sure we don't drop isolates, i.e. rows where the language does not belong to any
family (e.g. Alsea).
