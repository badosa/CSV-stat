# The JSON-stat Comma-Separated Values Format

[JSON-stat](https://json-stat.org/) is a simple lightweight JSON dissemination format for statistics best suited for data visualizations, mobile apps or open data initiatives.

The **JSON-stat Comma-Separated Values** format (CSV-stat, or JSV for short) is CSV plus an extra metadata header.
CSV-stat supports all the JSON-stat dataset core semantics.

JSON-stat can be converted to CSV-stat and CSV-stat can be converted back to JSON-stat without loss of information (only the [note](https://json-stat.org/format/#note), [link](https://json-stat.org/format/#link), [child](https://json-stat.org/format/#child), [coordinates](https://json-stat.org/format/#coordinates) and [extension](https://json-stat.org/format/#extension) properties in the JSON-stat format are not currently supported by CSV-stat).

***

**Sample file**: https://json-stat.org/samples/galicia.jsv

***

* [The Format](#the-format)
  * [The Extra Metadata Header](#the-extra-metadata-header)
    * [The *jsonstat* line](#the-jsonstat-line)
    * [The *data* line](#the-data-line)
    * [The *href*, *label*, *source* and *updated* lines](#the-href-label-source-and-updated-lines)
    * [The *dimension* lines](#the-dimension-lines)
  * [The CSV Lines](#the-csv-lines)
    * [The CSV header line](#the-csv-header-line)
    * [The CSV data records](#the-csv-data-records)
* [Conversion Tools](#conversion-tools)
  * [Command Line](#command-line)
  * [Client JavaScript](#client-javascript)
  * [Node.js](#nodejs)

***

## The Format

### The Extra Metadata Header

CSV-stat adds extra lines at the beginning of a regular CSV file. These extra lines contain metadata organized in columns like in a regular CSV. Like in a regular CSV, the column delimiter is a comma, but can be any character.

The first column in each line of the extra metadata header is the *tag column*. The following columns are *content columns*.

The tag column defines the meaning of the line and can only contain one of the following reserved words:

* *data*
* *dimension*
* *href*
* *jsonstat*
* *label*
* *source*
* *updated*

The first line in the metadata header must be a *jsonstat* line. The last one must be a *data* line. Lines in between can be in any order.

#### The *jsonstat* line

The *jsonstat* line contains two content columns: one for the decimal delimiter and one for the unit separator.

```
jsonstat,.,|
```

(All the examples assume that the column delimiter is the comma.)

#### The *data* line

The *data* line is an empty line: it does not contain any content column.

```
data
```

#### The *href*, *label*, *source* and *updated* lines

These lines are optional. When present, they contain a single content column to store the associated JSON-stat dataset property.

```
label,Unemployment rate in the OECD countries 2003-2014
source,Economic Outlook No 92 - December 2012 - OECD Annual Projections
updated,2012-11-27
href,https://json-stat.org/samples/oecd.json
```

#### The *dimension* lines

There must be as many *dimension* lines as dimensions in the dataset. Order is not important: dimensions will be sorted taking into account the CSV header line.

A *dimension* line must have the following mandatory content columns:

* dimension id
* dimension label
* dimension size (number of categories)

And then for each category:

* category id
* category label

```
dimension,sex,gender,3,T,total,M,male,F,female
```

After that, a *role* column can be optionally included when [role](https://json-stat.org/format/#role) information ("geo", "time", "metric") is available.

```
dimension,residence,province of residence,5,T,total,15,A Coruña,27,Lugo,32,Ourense,36,Pontevedra,geo
```

Dimensions with role "metric" can optionally include a *unit* column for each category in the dimension.

A unit column is made of four possible fields:

1. Number of [decimals](https://json-stat.org/format/#decimals)
2. Unit [label](https://json-stat.org/format/#label)
3. Unit [symbol](https://json-stat.org/format/#symbol)
4. Symbol [position](https://json-stat.org/format/#position)

These fields in the unit column must be delimited with the unit separator specified in the *jsonstat* line. The unit separator (usually, "|") must be different from the column delimiter.

```
dimension,measure,concepts,2,gsp,Gross State Product,pop,population,metric,0|million|$|start,1|million
```

### The CSV lines

CSV-stat's extra metadata header ends with a *data* line. After that, CSV regular lines are present.

#### The CSV header line

This line must have as many columns as dimensions, plus a *status* column (only when [status](https://json-stat.org/format/#status) information is available), plus a *value* column. Content of these columns in the header line are dimension ids, "status" (when status information is available) and "value".

```
concept,area,year,status,value
```

The order of the columns determines the dimension order. The last column is always the *value* column. When a *status* column is present, it must be immediately before the last one.

#### The CSV data records

Data records follow the header line. Columns contain category ids, status strings (when available) and values. Missing values can be indicated in the *value* column with any string not convertible to a number.

```
UNR,AU,2003,,5.943826289
UNR,AU,2004,m,n/a
UNR,AU,2005,,5.044790587
UNR,AU,2006,,4.789362794
UNR,AU,2007,,4.379649386
UNR,AU,2008,,4.249093453
UNR,AU,2009,,5.592226603
UNR,AU,2010,,5.230660289
UNR,AU,2011,,5.099422942
UNR,AU,2012,,5.224336088
UNR,AU,2013,e,5.50415003
UNR,AU,2014,e,5.462866231
```

Data records order is not relevant. While in a regular CSV, category order must be inferred from the records order, in CSV-stat this information is derived from *dimension* lines.

## Conversion Tools

### Command Line

You can use the [JSON-stat Command Line Conversion Tools](https://github.com/badosa/JSON-stat-conv) to convert to and from CSV-stat.

To convert to CSV-stat from JSON-stat, use [jsonstat2csv](https://github.com/badosa/JSON-stat-conv#jsonstat2csv):

```
jsonstat2csv oecd.json oecd.jsv --rich
```

To convert to JSON-stat from CSV-stat, use [csv2jsonstat](https://github.com/badosa/JSON-stat-conv#csv2jsonstat).

```
csv2jsonstat oecd.jsv oecd.json
```

### Client JavaScript

Include the [JSON-stat Javascript Toolkit](https://json-stat.com) and the [JSON-stat Javascript Utilities Suite](https://github.com/badosa/JSON-stat/tree/master/utils) in your webpage.

Use [toCSV()](https://github.com/badosa/JSON-stat/blob/master/utils/tocsv.md) to convert to CSV-stat from JSON-stat.

```js
JSONstat(
  "https://json-stat.org/samples/oecd.json",
  function(){
    var csv=JSONstatUtils.toCSV(
      this,
      {
        rich: true
      }
    );
    document.getElementsByTagName("body")[0].innerHTML="<pre>"+csv+"</pre>";
  }
);
```

Use [fromCSV()](https://github.com/badosa/JSON-stat/blob/master/utils/fromcsv.md) to convert to JSON-stat from CSV-stat.

```js
fetch( "https://json-stat.org/samples/galicia.jsv" )
  .then(function(resp) {
    resp.text().then(function(jsv){
      var json=JSONstatUtils.fromCSV( jsv );
      document.getElementsByTagName("body")[0].innerHTML=JSON.stringify(json);
    });
  })
;
```

### Node.js

Use the [jsonstat-utils](https://www.npmjs.com/package/jsonstat-utils) module.

```js
const
  utils = require("jsonstat-utils"),
  csvString = utils.toCSV( jsonstatObject, { rich: true } ),
  newJsonstatObject = utils.fromCSV( csvString )
;
```
