# FAQ

## GENERAL 

### Can I run PatchFox on my workstation? 

### Can I deploy PatchFox? 

### How do I get data into PatchFox? 

### Is there an enterprise version of PatchFox? 

## TECHNICAL 

### How we reference information and sources therein

For every git commit to every build file PatchFox is instructed to monitor, PatchFox ingests that datum as part of a time series and associates it with one or more "Datasets". 
A "Dataset" is a collection of "Datasources", which is what PatchFox calls a build file it's monitoring. A "Datasource" is the "source" of "events", which PatchFox calls "Datasource Events". 
It enriches the data with information from third party sources like OSS scanners and package indexes. It then tabulates top line metrics describing the state of the "Dataset" 


PatchFox uses the purl spec to refer to software packages - ie "dependencies" 
We also use the same to refer to the sources of said packages and the datums that come out of those sources. 
The purl format looks like this:

```scheme:type/namespace/name@version?qualifiers#subpath```

"scheme" and "type" will always be "pkg" and "generic" respectively 

In PatchFox, in reference to our use of pURL to represent the sources of data, there are three data concepts that are important to undertand: 
* DATASOURCE - is exactly what it sounds like - the name of the thing pumping data into the pipeline. In the pURL it's denoted in the "name" and the "version" fields. It's usually a git-tracked build file. 
  We use "__" to denote the directories in which the build file was discovered. The git branch from which the data eminates after the location of the build file using a :: delimeter. Finally we note the type (according to pURL spec) of the file. 
  * example purl: `pkg:generic/github/codeql__java__ql__integration-tests__java__maven-sample-xml-mode-byname%3A%3Amain@maven`
* DATASET - is a collection of "datasources" This is not denoted as a purl, only by name. In this case - and in keeping with the example above from respository "codeql", the name of the dataset is "github" which occupies the "namespace" pURL field. 
* DATASOURCE_EVENT - is a datum of information that eminates from a "datasource" Usually it's an SBOM, the git-annotated build file, and some metadata representing the state of the dependencies represented by the build file at a given git commit. Note how for a "datasource_event", we use the same pURL identifier as we do for the "datasource" from which the datum eminated with additional qualifiers indicating the commit datetime and the commit hash. In the pURL it occupies the "qualifiers" field.
  * example purl: `pkg:generic/github/codeql__java__ql__integration-tests__java__maven-sample-xml-mode-byname%3A%3Amain@maven?commitdatetime=2024-08-30T08%3A28%3A25%2B00%3A00&commithash=321820e758b86827f6e19ab3d7226d70f9ad982c`


### Metrics 
For every processed DatasourceEvent PatchFox will generate a record describing, in numerical terms, the overall state of the Dataset after the DatasourceEvent was applied. Many of these metrics are ones you would expect. Thigs like "how many packages are in the Dataset" and "how many findings have been in the Dataset for more than a month". These formulate a rich time-series that we use to monitor, forecast, and make recommendations as to how to manage your dependencies. 

Here is an explanation of what those metrics mean to PatchFox

* *Downlevel* refers to a package that is not the most current version offered by the vendor 

* *Stale* refers to a package that has not been updated by the vendor in [x] time. Note this is different than *downlevel* in that a *stale* package can be the most recent version but the vendor may have put out that version a year ago. 

* *Redundant Package Score (RPS)* in essence, this measure indicates the percentage of packages in the dataset that are different versions of the same thing. The measure goes from 0 -> 100. For example, if a dataset is comprised wholly of unique versions of "foo" then the RPS score will be 100. Conversely if every package in the dataset is a unique package type (a package sans version) then the RPS score is 0. RPS score has analytic value because there is a strong correlation between a dataset with a high RPS score and the liklihood that new findings will be associated with the dataset in future. A low RPS score is desired because it's an indication that there's reduced liklihood of findings being present in the dataset in future. TO BE EXPLICITLY CLEAR - THIS IS A MEASURE OF DIFFERENT VERSIONS OF THE SAME THING NOT NUMBER OF INSTANCES
OF THE SAME VERSION OF A THING. For example - if the dataset is comprised of ten INSTANCES of jackson-databind v1.1.1 the RPS score is ZERO. Conversely if the dataset is comprised of ten DIFFERENT VERSIONS of jackson-databind the RPS score is 100. 

* *Patch Effort/Impact/Efficacy* One of the things PatchFox endeavors to do is help organizations work smarter, not harder. The idea is to reduce the amount of effort being expended towards package management and maximize the impact (ie the benefit) of said effort. *Patch Efficacy Score (PES)* is a measure of the ratio of *Patch Impact / Patch Effort*. A high PES score is desireable because it indicates the organization is maximizing positive impact for every patch it makes. 
  * *Patch Effort* is considered to be "1" per patch by default. That number is diminished when multiple patches of the same kind are made within a 90 day period. The rationale is that when an organization performs the same task multiple times in a given 90 day period the task becomes easier with each successive time because humans tend to get better at things when they do it a lot. 
  * *Patch Impact* refers to the benefit of making a given Patch. When a patch reduces findings, avoids future findings, lowers *RPS* score, reduces *stale* or *downlevel* packages, the impact measure goes up.

### Entities

### package
Contains every package that has ever been detected by PatchFox. User may also refer to this as a "dependency". For record spec see [package Entity Reference](./package.md).

### finding
Pairs [findingReporter](./findingReporter.md) entity(ies) with a [findingData](./findingData.md) entity. Additionally are package records indicating which packages are tied to this finding. For record spec see [finding Entity Reference](./finding.md).

### findingData
Represents finding data including identifier, description, cpes, and summary text. For record spec see [findingData Entity Reference](./findingData.md).

### findingReporter
Represents sources if [findingData](./findingData.md). Usually an OSS scanner like Grype, Snyk, etc. For record spec see [findingReporter Entity Reference](./findingReporter.md).

### dataset
Represents a named collection of datasources. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [dataset Entity Reference](./dataset.md)

### datasetMetrics
Represents macro level metrics for a [dataset](./dataset.md) at the point the point in time an update represented by a [datasourceEvent](./datasourceEvent.md) is applied. For record spec see [datasetMetrics Entity Reference](./datasetMetrics.md)

### datasource
Represents a single datasource - ie - a single source controlled build file PatchFox is tracking. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [datasource Entity Reference](./datasource.md)

### datasourceEvent
Represents a single commit to a datasource (ie - a source controlled build file). See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [datasourceEvent Entity Reference](./datasourceEvent.md)

### edit
Represents a package change, CREATE, UPDATE, DELETE, resultant from a datasourceEvent (ie - a source controlled build file). Edit objects are additionally used by PatchFox to indicate recommended changes to a Dataset. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [edit Entity Reference](./edit.md).

### datasourceMetrics
For every commit to every [datasource](#datasource) a record is made to this table indicating the deltas of all the metrics tracked for all datasources in the [dataset](#dataset). In other words, dataset level metrics are tracked in the [datasetMetrics](#datasetMetrics) table and in this table are the deltas for every commit as to how those datasetMetrics numbers were impacted by any given commit. For record spec see [datasourceMetrics Entity Reference](./datasourceMetrics.md).

### datasourceMetricsCurrent
Represents the current state of every datasource from a metrics perspective in the dataset. This table contains a subset of the metrics contained in [datasetMetrics](#datasetMetrics) and [datasourceMetrics](#datasourceMetrics). Use this table to understand how the datasources connected to PatchFox comprise current datasetMetrics numbers. For record spec see [datasourceMetricsCurrent Entity Reference](./datasourceMetricsCurrent.md).


## PatchFox Data Service Database API Documentation

### Overview

The data-service provides an HTTP API for quering the database and receiving results as paginated collections of JSON encoded database records. The service itself is a spring-boot microservice that implements Spring Data REST paging and sorting capabilities WITH THE FOLLOWING EXCEPTIONS:  
* the `sort` parameter uses of a period - NOT A COMMA! The correct way to use the `sort` parameter is `sort=commitDateTime.desc`
---

### Table Entity Overview
The system manages software package data with security vulnerability information. See entity description for a list and short description of each. 
---

### Database Query API 

**Endpoint Pattern:** `{HOST}/api/v1/db/{tableName}/query`
- **Table names are case-insensitive** (e.g., `PACKAGE`, `package`, `Package` all work)


#### Capabilities

**Exact Match for Boolean, Numeric, and Dates:**
```http
GET /api/v1/db/datasetMetrics/query?isCurrent=true
GET /api/v1/db/datasetMetrics/query?totalFindings=100
GET /api/v1/db/datasetMetrics/query?commitDateTime=2024-01-01T00:00:00Z
```

**Numeric Comparisons:**
```http
GET /api/v1/db/datasetMetrics/query?totalFindings=gt.100        # Greater than 100
GET /api/v1/db/package/query?numberVersionsBehindHead=lt.5      # Less than 5
GET /api/v1/db/datasetMetrics/query?criticalFindings=gte.10     # Greater than or equal
GET /api/v1/db/datasetMetrics/query?lowFindings=lte.50          # Less than or equal
GET /api/v1/db/datasetMetrics/query?totalFindings=gte.10&gt.100 # Between 10 and 100
```

**Date Range Filtering:**
```http
GET /api/v1/db/datasetMetrics/query?commitDateTime=gt.2024-01-01T00:00:00Z
GET /api/v1/db/datasourceEvent/query?eventDateTime=lt.2024-12-31T23:59:59Z
GET /api/v1/db/datasetMetrics/query?commitDateTime=gte.2024-01-01T00:00:00Z&gt.2024-12-31T23:59:59Z
```

**Pattern Matching For String Values:**
!!! IMPORTANT -- DOES NOT SUPPORT `*` OR ANY OTHER FORM OF WILDCARD VALUE. MATCHES ARE FUZZY BY DEFAULT !!!

```http
GET /api/v1/db/findingData/query?description=SQL # Contains "SQL"
```

**Multiple Values (OR logic for same field):**
```http
GET /api/v1/db/package/query?type=npm,maven,pypi   # type is npm OR maven OR pypi
```

**Boolean Fields:**
```http
GET /api/v1/db/datasetMetrics/query?isCurrent=true&isForecastRecommendationsTaken=false
GET /api/v1/db/edit/query?isPfRecommendedEdit=true&isUserEdit=false
```

#### Complex Query Examples

**Find vulnerable npm packages:**
```http
GET /api/v1/db/package/query?type=npm&totalFindings=gt.0
```

**Find current dataset metrics with high critical findings:**
```http
GET /api/v1/db/datasetMetrics/query?isCurrent=true&criticalFindings=gt.10
```

**Find packages with versions:**
```http
GET /api/v1/db/package/query?version=1.0.0,1.1.0,2.0.0
```


### special queryDSL endpoints 

There are many times when the question being asked is tied in with a given Dataset at a given time. For questions involving Packages, Findings, or Edits associated with a given Dataset at a given time, there are the following four endpoints to help. 

#### packages 

**find packages associated wtih a given dataset.**
```http
/api/v1/db/datasetMetrics/package/query
```

**find package types (dedup of find packages) associated wtih a given dataset**
```http
/api/v1/db/datasetMetrics/packageType/query
```

**find packages associated with a set of datasources that are associated with a given dataset.** 
```http
/api/v1/db/datasetMetrics/datasource/package/query?datasetName=reddit&isCurrent=true&datasources.purl=go__src&commitDateTime=gt.1999-12-29T00:24:26Z
```

#### findings

**find finding types associated wtih a given dataset**
```http
/api/v1/db/datasetMetrics/package/findingType/query
```

**find findings associated wtih a set of datasources that are associated with a given dataset**
```http
/api/v1/db/datasetMetrics/datasource/package/finding/query
```

#### edits

**find edits associated wtih a given dataset**
```http
/api/v1/db/datasetMetrics/edit/query
```

**find edits associated wtih a given set of datasources that are associated with a given dataset**
```http
/api/v1/db/datasetMetrics/datasource/edit/query
```


Every one of these takes a small set of query string arguments intended for a DatasetMetrics query. Any remaining arguments will be passed along to the subsequent query to the Package, Finding, or Edit datastores respectively. These arguments are (and they must be camelCased):

* __datasetName__ (required)
  * the name of the Dataset or Datasets you want to include in the query
  * single name ex `datasetName=foo` 
  * multiple names ex `datasetName=foo,bar`
* __datasources.purl__ (required for datasetMetrics/datasource queries)
  * comma delimited list of datasource names (or full purls) used to scope the query to only datasources in the dataset that match the provided datasource names.
* __commitDateTime__ (optional)
  * the specific commitDateTime(s) you want records for. If not supplied you will get the latest record 
  * single date exact ex `commitDateTime=2025-04-09T01:08:40.648Z`
  * on or after date ex `commitDateTime=gte.2025-04-09T01:08:40.648Z`
  * range ex `commitDateTime.gte=2025-04-09T01:08:40.648Z&commitDateTime=lt.2025-05-09T01:08:40.648Z`
* __{isCurrent, isForecastSameCourse, isForecastRecommendationsTaken}__ (one of required)
  * indicates what kind of Dataset information you want: the actual data, the forecast based on the actual data, or the recommendations made based on the actual and forecasted data. 
  * just the actual data ex `isCurrent=true`
  * the actual and forecast data `isCurrent=true&isForecastSameCourse=true`

For example, if you make the following call 

```
/api/v1/db/datasetMetrics/package/query?datasetName=foo&isCurrent=true&purl=bar
```

PatchFox will retrieve the most recent metrics record for dataset "foo" marked "is_current". It will then look at all the Packages associated with that record for anything with a field "purl" that contains the text "bar" and return those to the caller. 


### special argument available only to genAI callers

Because the PatchFox genAI agent is making queries by way of tool middleware to the data-service it has access to a special argument called `select` that serves the same purpose as a SELECT statment in SQL. The effect is to reduce the fields returned by the API to only those specified by the `select` argument. 

#### ex. select only the status field from the dataset table where the "id" field equals "1"
```
https://{DATA_SERVICE_HOST}/api/v1/db/dataset/query?id=1&select=status
```

#### ex. select the status and name fields from the dataset table where the "id" field equals "1"
```
https://{DATA_SERVICE_HOST}/api/v1/db/dataset/query?id=1&select=status,name
```

#### ex select the nested field dataset.datasources.type field from the dataset collection "datasources"
```
https://{DATA_SERVICE_HOST}/api/v1/db/dataset/query?select=dataset.datasources.type
```

