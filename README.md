
# Light-OData

[![CircleCI](https://circleci.com/gh/Soontao/c4codata.svg?style=shield)](https://circleci.com/gh/Soontao/c4codata)
[![codecov](https://codecov.io/gh/Soontao/c4codata/branch/master/graph/badge.svg)](https://codecov.io/gh/Soontao/c4codata)
[![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=Soontao_c4codata&metric=sqale_rating)](https://sonarcloud.io/dashboard?id=Soontao_c4codata)
[![unpkg](https://img.shields.io/github/license/mashape/apistatus.svg)](https://unpkg.com/c4codata?meta)
[![npm version](https://badge.fury.io/js/light-odata.svg)](https://badge.fury.io/js/light-odata)
[![Netlify](https://img.shields.io/netlify/875f9eb3-319e-4dd7-b1eb-b3e5d15656ee?label=doc)](https://tender-dubinsky-6b6899.netlify.app/)

![Size](https://badgen.net/packagephobia/publish/light-odata)
[![Technical Debt](https://sonarcloud.io/api/project_badges/measure?project=Soontao_c4codata&metric=sqale_index)](https://sonarcloud.io/dashboard?id=Soontao_c4codata)
[![Vulnerabilities](https://sonarcloud.io/api/project_badges/measure?project=Soontao_c4codata&metric=vulnerabilities)](https://sonarcloud.io/dashboard?id=Soontao_c4codata)
[![DeepScan grade](https://deepscan.io/api/teams/9408/projects/11929/branches/178297/badge/grade.svg)](https://deepscan.io/dashboard#view=project&tid=9408&pid=11929&bid=178297)

Lightweight OData Client for OData (v2) Service.

## Installation

```bash
npm i -S light-odata
```

Alternative, in build-tool-less environment, just add [unpkg](https://unpkg.com/light-odata) umd link to your page, and `OData` object will be available in `window`.

```html
<script src="https://unpkg.com/light-odata"></script>
```

<!-- ToC start -->
## Table of Contents

   1. [Installation](#installation)
   1. [Table of Contents](#table-of-contents)
   1. [OData Version 2 concepts ](#odata-version-2-concepts-)
      1. [URI Format](#uri-format)
      1. [Pagination](#pagination)
      1. [Filter](#filter)
   1. [OData Client](#odata-client)
      1. [ODataRequest interface](#odatarequest-interface)
      1. [ODataResponse interface](#odataresponse-interface)
   1. [ODataParam](#odataparam)
      1. [pagination](#pagination-1)
      1. [filter](#filter-1)
      1. [inline count](#inline-count)
      1. [orderby](#orderby)
      1. [navigation property](#navigation-property)
      1. [fields select](#fields-select)
      1. [full text search (basic query)](#full-text-search-basic-query)
   1. [ODataFilter](#odatafilter)
      1. [filter by single field value](#filter-by-single-field-value)
      1. [filter by multi fields](#filter-by-multi-fields)
      1. [filter by one field but multi values](#filter-by-one-field-but-multi-values)
      1. [filter by date](#filter-by-date)
   1. [Batch requests](#batch-requests)
   1. [Generator usage](#generator-usage)
   1. [Others](#others)
   1. [CHANGELOG](#changelog)
   1. [LICENSE](#license)
<!-- ToC end -->

## OData Version 2 concepts 

From [OData.org](https://www.odata.org/documentation/odata-version-2-0/uri-conventions/)

**If you are already familiar with OData, please skip this section**

### URI Format

> A URI used by an OData service has up to three significant parts: the service root URI, resource path and query string options. Additional URI constructs (such as a fragment) MAY be present in a URI used by an OData service; however, this specification applies no further meaning to such additional constructs.

### Pagination

> A data service URI with a $skip System Query Option identifies a subset of the Entries in the Collection of Entries identified by the Resource Path section of the URI. That subset is defined by seeking N Entries into the Collection and selecting only the remaining Entries (starting with Entry N+1). N is an integer greater than or equal to zero specified by this query option. If a value less than zero is specified, the URI should be considered malformed.

```
https://services.odata.org/OData/OData.svc/Products?$skip=2&$top=2&$orderby=Rating

The third and fourth Product Entry from the collection of all products when the collection is sorted by Rating (ascending).
```
### Filter

> A URI with a `$filter` System Query Option identifies a subset of the Entries from the Collection of Entries identified by the Resource Path section of the URI. The subset is determined by selecting only the Entries that satisfy the predicate expression specified by the query option.

| Operator             | Description           | Example                                            |
| -------------------- | --------------------- | -------------------------------------------------- |
| Eq                   | Equal                 | /Suppliers?$filter=Address/City eq 'Redmond'       |
| Ne                   | Not equal             | /Suppliers?$filter=Address/City ne 'London'        |
| Gt                   | Greater than          | /Products?$filter=Price gt 20                      |
| Ge                   | Greater than or equal | /Products?$filter=Price ge 10                      |
| Lt                   | Less than             | /Products?$filter=Price lt 20                      |
| Le                   | Less than or equal    | /Products?$filter=Price le 100                     |
| And                  | Logical and           | /Products?$filter=Price le 200 and Price gt 3.5    |
| Or                   | Logical or            | /Products?$filter=Price le 3.5 or Price gt 200     |

## OData Client

Start with a simple query, following code start a `GET` http request, and asks the server to respond to all customers which phone number equals 030-0074321

```javascript
import { OData } from "light-odata"

// odata.org sample odata service
const TestServiceURL = "https://services.odata.org/V2/Northwind/Northwind.svc/$metadata"
const odata = new OData(TestServiceURL)

// Query by filter
//
// GET /V2/Northwind/Northwind.svc/Customers?$format=json&$filter=Phone eq '030-0074321'
const filter = OData.newFilter().field("Phone").eqString("030-0074321");

const result = await odata.newRequest({ // ODataRequest object
  collection: "Customers", // collection name
  params: OData.newParam().filter(filter) // odata param
})
```

### ODataRequest interface

```ts
interface ODataRequest<T> {
  collection: string, /** collection name */
  id?: any, /** object key in READ/UPDATE/DELETE, user could set this field with number/string/object type */
  params?: ODataQueryParam, /** params in QUERY */
  /**
   * GET for QUERY/READ; for QUERY, you can use params to control response data
   * PATCH for UPDATE (partial)
   * PUT for UPDATE (overwrite)
   * POST for CREATE
   * DELETE for DELETE
   */
  method?: HTTPMethod,
  entity?: T /** data object in CREATE/UPDATE */
}
```
### ODataResponse interface


```typescript
interface PlainODataResponse {
  d?: {
    __count?: string; /** $inlinecount values */
    results: any | Array<any>; /** result list/object */
    [key: string]: any;
  };
  error?: { /** if error occurred, node error will have value */
    code: string;
    message: {
      lang: string, 
      value: string /** server error message */
    }
  }
}
```

## ODataParam

use `ODataParam` to control data size, involved fields and order

### pagination

`skip` first 30 records and `top` 10 records

```js
// equal to $format=json&$skip=30&$top=10
OData.newParam().skip(30).top(10)
```

### filter

filter data by fields value

```js
// $format=json&$filter=A eq 'test'
OData.newParam().filter(OData.newFilter().field("A").eqString("test"))
```

### inline count

response with all records count, usefully.

also could set with `filter`, and response with filtered records count.

```js
// equal to $format=json&$inlinecount=allpages
OData.newParam().inlinecount(true).top(1).select("ObjectID")
```

### orderby

sort response data

```javascript
// result is $format=json&$orderby=CreationDateTime desc
OData.newParam().orderby("CreationDateTime")

// result is $format=json&$orderby=A desc,B asc
OData.newParam().orderby([{ field: "A" }, { field: "B", order: "asc" }])
```

### navigation property

expand association data

```javascript
// $expand=Customers
OData.newParam().expand("Customers")
// $expand=Customers,Employees
OData.newParam().expand(["Customers", "Employees"])
```

### fields select

remove unused fields from response

```js
// $format=json&$select=ObjectID,Name
OData.newParam().select("ObjectID").select("Name");
```

### full text search (basic query)

search all **supported** fields with text

**SAP systems feature**

**LOW PERFORMANCE**

```js
// fuzzy
// $format=json&$search=%any word%
OData.newParam().search("any word");
// not fuzzy
// $format=json&$search=any word
OData.newParam().search("any word", false);
```

## ODataFilter

Use the `ODataFilter` to filter data

Most `SAP` systems only support `AND` operator between different fields, and `OR` operator in a same field. (it depends on SAP Netweaver implementation)

So you don't need to specify `AND/OR` operator between fields, `light-odata` will auto process it.

Though C4C only support AND operator in different fields, but for `gt/lt/ge/le`, you can use AND for filtering period data.

just ref following examples

### filter by single field value

```js
// Name eq 'test string'
OData.newFilter().field("Name").eqString("test string")

// ID lt '1024'
OData.newFilter().field("ID").lt("'1024'")

// also support eq/ne/le/lt/gt/ge ...
```

### filter by multi fields

```js
// Name eq 'test string1' and Name2 eq 'test string2'
OData
  .newFilter()
  .field("Name").eq("'test string1'")
  .field("Name2").eqString("test string2")
```

### filter by one field but multi values

```js
// Name eq 'test string1' and (Name2 eq 'test string1' or Name2 eq 'test string2')
OData.newFilter()
  .field("Name").eq("'test string1'")
  .field("Name2").in(["test string3", "test string2"])
```

### filter by date

Depends on field type, use `betweenDateTime` or `betweenDateTimeOffset` to filter date。

Please provide `Date` object in this api.

```js
// Name eq 'test string1' and (CreationDateTime gt datetime'2018-01-24T12:43:31.839Z' and CreationDateTime lt datetime'2018-05-24T12:43:31.839Z')
OData
  .newFilter()
  .field("Name").eq("'test string1'")
  .field("CreationDateTime").betweenDateTime(
    new Date("2018-01-24T12:43:31.839Z"),
    new Date("2018-05-24T12:43:31.839Z"),
    false
  )
  .build()

// > include boundary value

// (CreationDateTime ge datetime'2018-01-24T12:43:31.839Z' and CreationDateTime le datetime'2018-05-24T12:43:31.839Z')
OData
  .newFilter()
  .field("CreationDateTime").betweenDateTime(
    new Date("2018-01-24T12:43:31.839Z"),
    new Date("2018-05-24T12:43:31.839Z")
  )
  .build()
```

 
## Batch requests

use odata `$batch` api for operating multi entities in **single** HTTP request, it will save a lot of time between client & server (In the case of processing a large number of requests).

```javascript

const odata = OData.New({
  metadataUri: `https://services.odata.org/V2/(S(${v4()}))/OData/OData.svc/$metadata`,
  processCsrfToken: false, // set false if you dont need process csrf token
})
const testDesc1 = v4(); // a generated uuid
const testDesc2 = v4();

// execute reqeusts and return mocked responses
const result = await odata.execBatchRequests([
  odata.newBatchRequest({
    collection: "Products",
    entity: {
      ID: 100009,
      Description: testDesc1,
    },
    method: "POST",
    // withContentLength: true, for SAP OData, please set this flag as true
  }),
  odata.newBatchRequest({
    collection: "Products",
    entity: {
      ID: 100012,
      Description: testDesc2,
    },
    method: "POST",
    // withContentLength: true, for SAP OData, please set this flag as true
  })
])


result.map(r => expect(r.status).toEqual(201)) // Created

```

## Others

Use [markdown-toc](https://github.com/sebdah/markdown-toc) to generate table of contents, with following commands: 


```bash
markdown-toc --replace --inline --header "## Table of Contents" --skip-headers=1  README.md
```



## CHANGELOG

[CHANGELOG](./CHANGELOG.md)

## LICENSE

MIT License

Copyright (c) 2018 Theo Sun

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
