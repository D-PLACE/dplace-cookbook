# The D-PLACE Cookbook

Recipes for cooking with [D-PLACE data](https://github.com/d-place/dplace-cldf/releases).

> [!IMPORTANT]  
> Note that starting with release 3.0, the aggregated D-PLACE data is distributed as [CLDF dataset](https://github.com/dplace-cldf) only.

## Accessing D-PLACE data

### Overview

D-PLACE data can be accessed in a variety of ways.

- It can be browsed and visualized in the [D-PLACE web application](https://d-place.org).
- Released versions of the aggregated data can be downloaded from [ZENODO](https://zenodo.org/doi/10.5281/zenodo.3935419).
- The constituent datasets and phylogenies can be cloned or forked from repositories at [D-PLACE](https://github.com/d-place) or [Phlorest](https://github.com/phlorest) - thus getting access to work-in-progress data.

Which of these options to choose?

- If you want to get an overview of the kind and extent of data available in D-PLACE, browse the [D-PLACE web application](https://d-place.org).
- If you want to use D-PLACE data for your own analyses, start with a released version(s) from ZENODO.
- If you want to contribute to D-PLACE - e.g. to fix errata - fork and clone the appropriate dataset repositories.


### Working with the CLDF data

The aggregated D-PLACE CLDF data (starting from version 3.0) contains information about
- datasets and phylogenies, as items in a [ContributionTable](https://github.com/cldf/cldf/tree/master/components/contributions)
- societies (from the included datasets) and languoids referenced in the included phylogenies, as items in a [LanguageTable](https://github.com/cldf/cldf/tree/master/components/languages)
- variables (from the included datasets), as items in a [ParameterTable](https://github.com/cldf/cldf/tree/master/components/parameters)
- the summary trees from the included phylogenies, as items in a [TreeTable](https://github.com/cldf/cldf/tree/master/components/trees)

Most recipes in this cookbook assume a local directory looking like https://github.com/D-PLACE/dplace-cldf/tree/master/cldf .
I.e. a clone of https://github.com/D-PLACE/dplace-cldf or an unzipped download of a released version.
