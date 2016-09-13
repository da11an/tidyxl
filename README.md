
<!-- README.md is generated from README.Rmd. Please edit that file -->
tidyxl
======

[![Travis-CI Build Status](https://travis-ci.org/nacnudus/tidyxl.svg?branch=master)](https://travis-ci.org/nacnudus/tidyxl) [![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/nacnudus/tidyxl?branch=master&svg=true)](https://ci.appveyor.com/project/nacnudus/tidyxl)

[tidyxl](https://github.com/nacnudus/tidyxl) imports non-tabular from Excel files into R. It exposes cell content, position, formatting and comments in a tidy structure for further manipulation, especialy by the [unpivotr](https://github.com/nacnudus/unpivotr) package. It supports the xml-based file formats '.xlsx' and '.xlsm' via the embedded [RapidXML](http://rapidxml.sourceforge.net) C++ library. It does not support the binary file formats '.xlsb' or '.xls'.

Installation
------------

``` r
devtools::install_github("nacnudus/tidyxl")
```

Examples
--------

``` r
library(readxl)
library(tidyxl)
```

The package includes a spreadsheet, 'examples.xlsx', which has a completely unstructured and non-tabular layout. A popular package for importing spreadsheets does the following:

``` r
# This example is not run because readxl currently (2016-09-14) fails where
# any 'headers' are missing
# read_excel("./inst/extdata/examples.xlsx", "Sheet1")
```

[tidyxl](https://github.com/nacnudus/tidyxl) imports the same table into a format suitable for non-tabular processing (see e.g. the [unpivotr](https://github.com/nacnudus/unpivotr) package in 'Similar projects' below).

``` r
(pivoted <- contents("./inst/extdata/examples.xlsx", "Sheet1")[[1]])
#> # A tibble: 105 × 15
#>    address   row   col content formula formula_type formula_ref
#>      <chr> <int> <int>   <chr>   <chr>        <chr>       <chr>
#> 1       A1     1     1    1337    <NA>         <NA>        <NA>
#> 2       B1     1     2    1338    <NA>         <NA>        <NA>
#> 3       C1     1     3   42046    <NA>         <NA>        <NA>
#> 4       D1     1     4   42047    <NA>         <NA>        <NA>
#> 5       E1     1     5   42048    <NA>         <NA>        <NA>
#> 6       F1     1     6       0    <NA>         <NA>        <NA>
#> 7       G1     1     7       1    <NA>         <NA>        <NA>
#> 8       H1     1     8       2    <NA>         <NA>        <NA>
#> 9       I1     1     9       6    <NA>         <NA>        <NA>
#> 10      J1     1    10       4    <NA>         <NA>        <NA>
#> # ... with 95 more rows, and 8 more variables: formula_group <int>,
#> #   type <chr>, character <chr>, height <dbl>, width <dbl>,
#> #   style_format_id <dbl>, local_format_id <dbl>, comment <chr>
```

Formatting is imported separately.

``` r
# Only a small part of the formatting structure is shown here
formats("./inst/extdata/examples.xlsx")$local[1:3] %>% str
#> List of 3
#>  $ number_format: chr [1:75] "General" "d-mmm-yy" "[$-1409]d\\ mmmm\\ yyyy;@" "yyyy\\ mmmm\\ dddd" ...
#>  $ font         :List of 9
#>   ..$ bold     : logi [1:75] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ italic   : logi [1:75] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ underline: logi [1:75] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ strike   : logi [1:75] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ size     : int [1:75] 11 11 11 11 11 11 11 11 11 11 ...
#>   ..$ color    :List of 3
#>   .. ..$ rgb    : chr [1:75] "FFFF0000" "FFFF0000" "FFFF0000" "FFFF0000" ...
#>   .. ..$ indexed: num [1:75] NA NA NA NA NA NA NA NA NA NA ...
#>   .. ..$ theme  : chr [1:75] NA NA NA NA ...
#>   ..$ name     : chr [1:75] "Calibri" "Calibri" "Calibri" "Calibri" ...
#>   ..$ family   : int [1:75] 2 2 2 2 2 2 2 2 2 2 ...
#>   ..$ scheme   : chr [1:75] "minor" "minor" "minor" "minor" ...
#>  $ fill         :List of 3
#>   ..$ patternType: chr [1:75] "none" "none" "none" "none" ...
#>   ..$ bgColor    :List of 3
#>   .. ..$ rgb    : chr [1:75] "FFFFFFFF" "FFFFFFFF" "FFFFFFFF" "FFFFFFFF" ...
#>   .. ..$ indexed: num [1:75] NA NA NA NA NA NA NA NA NA NA ...
#>   .. ..$ theme  : chr [1:75] NA NA NA NA ...
#>   ..$ fgColor    :List of 3
#>   .. ..$ rgb    : chr [1:75] "FFFFFFFF" "FFFFFFFF" "FFFFFFFF" "FFFFFFFF" ...
#>   .. ..$ indexed: num [1:75] NA NA NA NA NA NA NA NA NA NA ...
#>   .. ..$ theme  : chr [1:75] NA NA NA NA ...
```

This enables subsetting of cells by their formats.

``` r
(bold <- formats("./inst/extdata/examples.xlsx")$local$font$bold)
#>  [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE
#> [12] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
#> [23] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
#> [34] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
#> [45] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
#> [56] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
#> [67] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
pivoted[pivoted$local_format_id %in% which(bold), ]
#> # A tibble: 2 × 15
#>   address   row   col content formula formula_type formula_ref
#>     <chr> <int> <int>   <chr>   <chr>        <chr>       <chr>
#> 1      I1     1     9       6    <NA>         <NA>        <NA>
#> 2      T1     1    20      14    <NA>         <NA>        <NA>
#> # ... with 8 more variables: formula_group <int>, type <chr>,
#> #   character <chr>, height <dbl>, width <dbl>, style_format_id <dbl>,
#> #   local_format_id <dbl>, comment <chr>
```

Philosophy
----------

Information in in many spreadsheets cannot be easily imported into R. Why?

Most R packages that import spreadsheets have difficulty unless the layout of the spreadsheet conforms to a strict definition of a 'table', e.g.:

-   observations in rows
-   variables in columns
-   a single header row
-   all information represented by characters, whether textual, logical, or numeric

These rules are designed to eliminate ambiguity in the interpretation of the information. But most spreadsheeting software relaxes these rules in a trade of ambiguity for expression via other media:

-   proximity (other than headers, i.e. other than being the first value at the top of a column)
-   formatting (colours and borders)

Humans can usually resolve the ambiguities with contextual knowledge, but computers are limited by their ignorance. Programmers are hampered by:

-   their language's expressiveness
-   loss of information in transfer from spreadsheet to programming library

Information is lost when software discards it in order to force the data into tabular form. Sometimes date formatting is retained, but mostly formatting is lost, and position has to be inferred again.

[tidyxl](https://github.com/nacnudus/tidyxl) addresses the programmer's problems by not discarding information. It imports the content, position and formatting of cells, leaving it up to the user to associate the different forms of information, and to re-encode them in tabular form without loss. Further packages are planned to assist with that step.

Similar projects
----------------

[tidyxl](https://github.com/nacnudus/tidyxl) was originally derived from [readxl](https://github.com/hadley/readxl) and still contains some of the same code, hence it inherits the GPL-3 licence. [readxl](https://github.com/hadley/readxl) is intended for importing tabular data with a single row of column headers, whereas [tidyxl](https://github.com/nacnudus/tidyxl) is more general, and less clever.

The [rsheets](https://github.com/rsheets) project of several R packages is in the early stages of importing spreadsheet information from Excel and Google Sheets into R, manipulating it, and potentially parsing and processing formulas and writing out to spreadsheet files. In particular, [jailbreaker](https://github.com/rsheets/jailbreakr) attempts to extract non-tabular data from spreadsheets into tabular structures automatically via some clever algorithms.

[tidyxl](https://github.com/nacnudus/tidyxl) differs from [rsheets](https://github.com/rsheets) in scope ([tidyxl](https://github.com/nacnudus/tidyxl) will never import charts, for example), and implementation ([tidyxl](https://github.com/nacnudus/tidyxl) is implemented mainly in C++ and is quite fast, only a little slower than [readxl](https://github.com/hadley/readxl)). [unpivotr](https://github.com/nacnudus/unpivotr) is a package related to [tidyxl](https://github.com/nacnudus/tidyxl) that provides tools for unpivoting complex and non-tabular data layouts using I not AI (intelligence, not artificial intelligence). In this way it corresponds to [jailbreaker](https://github.com/rsheets/jailbreakr), but with a different philosophy.

Roadmap
-------

-   \[ \] Write tests
-   \[ \] Parse shared formulas and propagate to all associated cells.
-   \[ \] Propagate array formulas to all associated cells.
-   \[ \] Parse dates
-   \[ \] Detect cell types (date, boolean, string, number)
-   \[ \] Implement formatting import in C++ for speed.
