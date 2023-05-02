---
title: "Introducing tidyods"
date: 2022-06-12
cover: img/post/2022-06-12-hans-peter-gauster-3y1zF4hIPCg-unsplash.jpg
caption: <a href=\"https://unsplash.com/photos/3y1zF4hIPCg?utm_source=unsplash&utm_medium=referral&utm_content=creditShareLink\">Photo</a> by <a href=\"https://unsplash.com/@sloppyperfectionist?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText\">Hans-Peter Gauster</a> on <a href=\"https://unsplash.com/s/photos/puzzle?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText\">Unsplash</a>"
slug: introducing-tidyods
tags:
  - r
  - package
  - tidyods
---

TLDR: For very good reasons ODS is a horrible data file format.

The OpenDocument Spreadsheet (ODS) format is an increasingly common format for publishing spreadsheets, especially official statistics from UK government departments. I recently encountered [a problem](https://github.com/chainsawriot/readODS/issues/81) when trying to use the [`{readODS}` package](https://cran.r-project.org/package=readODS) to read a published ODS file. More surprsingly, I discovered that {readODS} is the only R package on CRAN for working with ODS files. As a result I've started to develop my own package, [`{tidyods}`](https://mattkerlogue.github.io/tidyods/).

{{< addTOC >}}

## The original problem
The Cabinet Office publishes the Civil Service Statistics[^1], there is an annual publication as well as ad-hoc releases through the year. In the past year, in line with Government Statistical Service guidelines, they have begun publishing their data using the ODS format.

In the course of doing some work I wanted to use the ad-hoc release on [the number of civil servants by postcode](https://www.gov.uk/government/statistics/number-of-civil-servants-by-postcode-department-responsibility-level-and-leaving-cause-2021). This is a fairly beastly spreadsheet, the sheet I'm interested in has 5,544 rows and 11 columns (or 60,984 cells). But in reading in this file I discovered 245 rows that were completely blank. With thanks to [Duncan Garmonsway](https://github.com/nacnudus) the problem was diagnosed as a misinterpretation of the `number-rows-repeated` attribute of the ODS specification. The package author had mistakenly interpretted this as meaning that rows with this attribute are empty[^2]. More interestingly it seems that LibreOffice and Google Sheets do not set this attribute, but Microsoft Excel does. Even if government documents are published in OpenDocument format, the underlying workflows tend to involve Microsoft products in their creation[^3].

### A problem fixed, a problem doubled
I devised a [fix](https://github.com/chainsawriot/readODS/pull/82) for the original problem. In doing so I discovered a separate bug introduced by a recent change to handle the parsing of repeated whitespace. More troublingly this fix has however negatively impacted performance of the package code.

### ODS is a horibble format
As part of the process of developing a fix, I fell down the necessary rabbit hole that is exploring the structure of an ODS file. An ODS file is zip file containing some XML files and other associated files. The main beast is a file called `content.xml` which contains the structure and data of the spreadsheet, other files provide metadata and things such as images. This XML follows the [ODF schema](https://docs.oasis-open.org/office/OpenDocument/v1.3/os/part3-schema/OpenDocument-v1.3-os-part3-schema.html#__RefHeading__1417680_253892949) published by the [Organisation for the Advancement of Structured Information Standards](https://www.oasis-open.org).

XML is a verbose format which has been described as ["the angle bracket tax"](https://blog.codinghorror.com/xml-the-angle-bracket-tax/) and [many](http://wiki.c2.com/?XmlSucks), [many](http://harmful.cat-v.org/software/xml/), [people](https://thecontentwrangler.com/2016/02/23/why-does-xml-suck/) have declared that XML "sucks"[^4]. And therefore if XML sucks, then ODS really sucks.

In theory the strucutre of the ODS XML is fairly simple: header elements, a body element, a spreadsheet element, table elements, row elements, cell elements, text elements, whitespace elements. But XML's verbosity means that ultimately files become large. For example the ODS I was having touble with is only 352KB as an XML document but unzip the ODS container and the underlying `contxt.xml` file is 11.9MB, some 33.8 times larger. The [Journey Time Statistics files](https://www.gov.uk/government/statistical-data-sets/journey-time-statistics-data-tables-jts), table JTS0501 is a 92.7MB file but unzipped its content file is 1.93GB, some 20.8 times larger. There is also major redundancy, the content of cells can appear twice, as an attibute of the cell element.

### If its so terrible why are you writing another package
To be honest, I don't know if I support the use of ODS as a format for publishing government data, but spreadsheet applications are how many people, especially non-analysts, interact with datasets and therefore its better to use an open standard than a proprietary format controlled by a single organisation. Maybe though data, espeically large and complex data, shouldn't be published in ODS format.

### That's not answering the question
Like I said at the top, readODS is the only package listed on CRAN for reading (and writing) ODS file in R. There are however more than 20 packages that work with Excel files. There are various views about whether having multiple ways to do something is desirable or not.

> *There should be one -- and preferably only one -- obvious way to do it.* - The 13th aphorism of the Zen of Python[^5]

But in most programming languages there are usually multiple ways of doing things. So it was a little surprising that there's currently only one package for handling ODS files[^6], and it might help to have some redundancy in the R package ecosystem.

In my investigations of the ODS XML to develop a fix for the bug in {readODS} I realised it would be relatively easy to write my own package to extract information from an ODS file and so I thought why not. Plus if I actually develop this properly it will mean I can go through the CRAN and/or rOpenSci submission processes.

## Introducing tidyods
So without further ado, let me introduce to you the [`{tidyods}` package](https://mattkerlogue.github.io/tidyods/), a package to import cells from ODS files. This package is more an equivalent to Duncan Garmonsway's [`{tidyxl}`](https://nacnudus.github.io/tidyxl/) package for ODS files, but also includes functions to produce similar output to {readODS}.

At present {tidyods} provides four functions:

 - [`read_ods_cells()`](https://mattkerlogue.github.io/tidyods/reference/read_ods_cells.html) to extract cells from an ODS file
 - [`read_ods_sheet()`](https://mattkerlogue.github.io/tidyods/reference/read_ods_sheet.html) to extract the cells as a rectangular dataset
 - [`ods_sheets()`](https://mattkerlogue.github.io/tidyods/reference/ods_sheets.html) to list the sheets in an ODS file
 - [`simple_rectify()`](https://mattkerlogue.github.io/tidyods/reference/simple_rectify.html) to "rectify" cells into a rectangular dataset

In due course I'll write some further blogs on the detailed working of the package, but a brief discussion of these functions now follows.

### Reading cells
The `read_ods_cells()` function is the main workhorse function, `read_ods_sheet()` is a convenience wrapper that combines that function with a call to `simple_rectify()`.

The process for getting cells from the ODS XML file necessitates iterating over rows and then cells (i.e. columns), as a result the underlying process for extract cells builds a dataset that has row and column indices. The ODS specification requires cells have a value type and for non-string value types the value must be stored as an attribute, as well as having a text representation of the value.

`read_ods_cells()` produces output that is similar to the [tidyxl::xlsx_cells()](https://nacnudus.github.io/tidyxl/reference/xlsx_cells.html) function. It returns a tibble with cell location information, values and value type. It also indicates whether a cell is a "proper" cell (i.e. contains a value), is an empty cell (i.e. has no data) or if its a cell covered by a merged cell.

```r
my_ods_cells <- tidyods::read_ods_cells("test.ods", "Sheet1")
```
```
> my_ods_cells |> dplyr::filter(row > 4 & & row < 9)
# A tibble: 20 × 8
     row   col cell_type value_type cell_formula cell_content  base_value          currency_symbol
   <dbl> <int> <chr>     <chr>      <chr>        <chr>         <chr>               <chr>          
 1     5     1 cell      string     NA           "Country"     Country             NA             
 2     5     2 cell      string     NA           "Market"      Market              NA             
 3     5     3 cell      string     NA           "Date"        Date                NA             
 4     5     4 cell      string     NA           "Available"   Available           NA             
 5     5     5 cell      string     NA           "Apple price" Apple price         NA             
 6     6     1 cell      string     NA           "England"     England             NA             
 7     6     2 cell      string     NA           "London"      London              NA             
 8     6     3 cell      date       NA           "2021-06-06"  2021-06-06T00:00:00 NA             
 9     6     4 cell      boolean    NA           "TRUE"        true                NA             
10     6     5 cell      float      NA           "1.8"         1.8                 NA             
11     7     1 cell      string     NA           "England"     England             NA             
12     7     2 cell      string     NA           "London"      London              NA             
13     7     3 merged    NA         NA           ""            NA                  NA             
14     7     4 cell      boolean    NA           "TRUE"        true                NA             
15     7     5 cell      float      NA           "1.9"         1.9                 NA             
16     8     1 cell      string     NA           "England"     England             NA             
17     8     2 cell      string     NA           "Birmingham"  Birmingham          NA             
18     8     3 cell      date       NA           "2021-05-30"  2021-05-30T00:00:00 NA             
19     8     4 cell      boolean    NA           "FALSE"       false               NA             
20     8     5 empty     NA         NA           ""            NA                  NA             
```

### Rectfying cells into a spreadsheet
The {tidyods} package also include a function for coerceing the cells into their original rectangular structure, or taking the notation of the [`{unpivotr}`](https://nacnudus.github.io/unpivotr/) package this is a "rectify" function. The `simple_rectify()` function does not use a row from the sheet for column names, instead is uses the column number (prepended by X). 

```r
tidyods::simple_rectify(my_ods_cells)
```

```
# A tibble: 17 × 5
   X1                                                   X2         X3                  X4        X5   
   <chr>                                                <chr>      <chr>               <chr>     <chr>
 1 Fruit Market Table                                   NA         NA                  NA        NA   
 2 This sheet has some information about fruit markets. NA         NA                  NA        NA   
 3 NA                                                   NA         NA                  NA        NA   
 4 Location                                             NA         NA                  Apple st… NA   
 5 Country                                              Market     Date                Available Appl…
 6 England                                              London     2021-06-06T00:00:00 true      1.8  
 7 England                                              London     NA                  true      1.9  
 8 England                                              Birmingham 2021-05-30T00:00:00 false     NA   
 9 England                                              Manchester 2021-05-30T00:00:00 false     NA   
10 England                                              Manchester 2021-05-29T00:00:00 true      1.3  
11 England                                              Manchester 2021-05-29T00:00:00 true      1.3  
12 Scotland                                             Edinburgh  Thurs 26/05/21      true      1.6  
13 Scotland                                             Edinburgh  NA                  true      1.4  
14 Scotland                                             Glasgow    2021-05-27T00:00:00 true      1.5  
15 Scotland                                             Aberdeen   NA                  true      1.6  
16 Wales                                                Cardiff    2021-05-25T00:00:00 true      1.4  
17 Wales                                                Swansea    2021-05-25T00:00:00 true      1.3  
```

You can see from the example above that there in the `read_ods_cells()` output there are two columns for cell value, the `cell_content` and `base_value`. The `cell_content` column shows the value stored within the text element(s) inside of the cell XML element, whereas `base_value` is derived from the cell attributes, which are used to store the underlying, unformatted data, for non-string values[^7]. By default `simple_rectify()` will use the `base_value`, you can however use the cell_contents by change the `base_values` argument of the function.

## What next
There are some key aspects of package development that are still needed, namely adding examples and tests.

But after that there are a couple of areas that I'm considering for further functionality: a smart rectifier, and performance improvement.

### A smarter rectifier?
The `simple_rectify()` function is a simple pivoting of the cells back to a 2-dimensional structure. However, spreadsheets often aren't purely tabular, for example they may have metadata information in the first couple of rows. The value type information provided by `read_ods_cells()` could also be used to guess and parse the columns of a rectified table into the relevant R datatype, the `simple_rectify()` function leaves all columns as character values.

This might not be a sensible idea, and it wouldn't be intended as a proepr replacement to either the excellent [`{janitor}`](http://sfirke.github.io/janitor/) and `{unpivotr}` packages. It would also likely never be the default option for `read_ods_sheet()`.

### Performance
As might be guessed from earlier commentary about XML being a horrible format for storing data, `{tidyods}` is not a fast performance-optimised procedure. {readODS} is not a particular fast package either, but `{tidyods}` is noticeably slower for larger files. The large postcode data file I mentioned earlier takes around 5-6 seconds to read with `{readODS}`, whereas it takes around 35 seconds with `{tidyods}`.

Both `{readODS}` and `{tidyods}` make use of the [`{xml2}`](http://xml2.r-lib.org) package which offers some memory handling improvements over the older (and no longer actively maintained) [`{XML}`](https://cran.r-project.org/package=XML) package.

The main workhorse of `read_ods_cells()` is a [`purrr::map()`]() call to iterate over all of the row elements. The first potential avenue for performance improvement is to switch to the [`{furrr}`](http://furrr.futureverse.org) package which implements parallel processing versions of the `{purrr}` package. Some initial tests failed, but I've since realised this is because of the interaction between `{furrr}` and `{xml2}`[^8].

The other option would be to develop code that uses the [RapidXML](http://rapidxml.sourceforge.net) C++ library to parse the XML file, which is the approach used by `{tidyxl}`.

## Huzzah

You've got to the end of this post. Well done[^9].

[^1]: Full disclosure, I currently work for the Cabinet Office in the divsion that publishes this data.
[^2]: This has been in the [code base](https://github.com/chainsawriot/readODS/commit/257d333acae1a638647696bf59b8420bbbf290d8#diff-ea7cd63ed9a76967f95c5601f0f117a9cd63258fab8f2dca57f0abc809fdf1c8R492-R494) for ~6 years.
[^3]: The [{a11ytables}](https://co-analysis.github.io/a11ytables/) R package created by my colleague Matt Dray for making spreadsheets that meet the government's accessibility requirements relies on Microsoft Excel in a final workflow step because R packages for writing ODS files don't work as well as they should.
[^4]: "xml sucks" returns [3.1 million Google results](https://www.google.com/search?client=safari&rls=en&q=xml+sucks&ie=UTF-8&oe=UTF-8).
[^5]: Yes this is an R-heavy blog, but sometimes discussion of Python is acceptable.
[^6]: Or perhaps not if you're a cynic that think this is illustrative of just how little ODS is actually used as a format by "real" people.
[^7]: One benefit of base_value is that dates and times are stored in ISO format.
[^8]: Definitely a subject for a separate blog post.
[^9]: Have a GIF {{< figure src="https://media.giphy.com/media/3o6ZtaiPZNzrmRQ6YM/giphy.gif" alt="A GIF og Stewie from Family Guy rocking back and forth in his bed" >}}
