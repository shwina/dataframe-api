# Purpose and scope

## Introduction

This document defines a Python dataframe API.

A dataframe is a programming interface for expressing data manipulations over a
data structure consisting of rows and columns. Columns are named, and values in a
column share a common data type. This definition is intentionally left broad.

## History and dataframe implementations

Dataframe libraries in several programming language exist, such as
[R](https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/data.frame),
[Scala](https://docs.databricks.com/spark/latest/dataframes-datasets/introduction-to-dataframes-scala.html),
[Julia](https://juliadata.github.io/DataFrames.jl/stable/) and others.

In Python, the most popular dataframe library is [pandas](https://pandas.pydata.org/).
pandas was initially developed at a hedge fund, with a focus on
[panel data](https://en.wikipedia.org/wiki/Panel_data) and financial time series.
It was open sourced in 2009, and since then it has been growing in popularity, including
many other domains outside time series and financial data. While still rich in time series
functionality, today is considered a general-purpose dataframe library. The original
`Panel` class that gave name to the library was deprecated in 2017 and removed in 2019,
to focus on the main `DataFrame` class.

Internally, pandas is implemented on top of NumPy, which is used to store the data
and to perform many of the operations. Some parts of pandas are written in Cython.

As of 2020 the pandas website has around one million and a half visitors per month.

Other libraries emerged in the last years, to address some of the limitations of pandas.
But in most cases, the libraries implemented a public API very similar to pandas, to
make the transition to their libraries easier. Next, there is a short description of
the main dataframe libraries in Python.

[Dask](https://dask.org/) is a task scheduler built in Python, which implements a
dataframe interface. Dask dataframe uses pandas internally in the workers, and it provides
an API similar to pandas, adapted to its distributed and lazy nature.

[Vaex](https://vaex.io/) is an out-of-core alternative to pandas. Vaex uses hdf5 to
create memory maps that avoid loading data sets to memory. Some parts of Vaex are
implemented in C++.

[Modin](https://github.com/modin-project/modin) is a distributed dataframe
library originally built on [Ray](https://github.com/ray-project/ray), but has
a more modular way, that allows it to also use Dask as a scheduler, or replace the
pandas-like public API by a SQLite-like one.

[cuDF](https://github.com/rapidsai/cudf) is a GPU dataframe library built on top
of Apache Arrow and RAPIDS. It provides an API similar to pandas.

[PySpark](https://spark.apache.org/docs/latest/api/python/index.html) is a
dataframe library that uses Spark as a backend. PySpark public API is based on the
original Spark API, and not in pandas.

[Koalas](https://github.com/databricks/koalas) is a dataframe library built on
top of PySpark that provides a pandas-like API.

[Ibis](https://ibis-project.org/) is a dataframe library with multiple SQL backends.
It uses SQLAlchemy and a custom SQL compiler to translate its pandas-like API to
SQL statements, executed by the backends. It supports conventional DBMS, as well
as big data systems such as Apache Impala or BigQuery.

Given the growing Python dataframe ecosystem, and its complexity, this document provides
a standard Python dataframe API. Until recently, pandas has been a de-facto standard for
Python dataframes. But currently there are a growing number of not only dataframe libraries,
but also libraries that interact with dataframes (visualization, statistical or machine learning
libraries for example). Interactions among libraries are becoming complex, and the pandas
public API is suboptimal as a standard, for its size, complexity, and implementation details
it exposes (for example, using NumPy data types or `NaN`).


## Scope

In the first iteration of the API standard, the scope is limited to create a data exchange
protocol. In future iterations the scope will be broader, including elements to operate with
the data.

It is in the scope of this document the different elements of the API. This includes signatures
and semantics. To be more specific:

- Data structures and Python classes
- Functions, methods, attributes and other API elements
- Expected returns of the different operations
- Data types (Python and low-level types)

The scope of this document is limited to generic dataframes, and not dataframes specific to
certain domains.


### Goals

The goal of the first iteration is to provide a data exchange protocol, so consumers of dataframes
can interact with a standard interface to access their data.

The goal of future iterations will be to provide a standard interface that encapsulates
implementation details of dataframe libraries. This will allow users and third-party libraries to
write code that interacts and operates with a standard dataframe, and not with specific implementations.

The main goals for the API defined in this document are:

- Make conversion of data among different implementations easier
- Let third party libraries consume dataframes from any implementations

In the future, besides a data exchange protocol, the standard aims to include common operations
done with dataframe, with the next goals in mind:

- Provide a common API for dataframes so software using dataframes can work with all
  implementations
- Provide a common API for dataframes to build user interfaces on top of it, for example
  libraries for interactive use or specific domains and industries
- Help user transition from one dataframe library to another

See the [use cases](02_use_cases.html) section for details on the exact use cases considered.


### Out-of-scope

#### Execution details

Implementation details of the dataframes and execution of operations. This includes:

- How data is represented and stored (whether the data is in memory, disk, distributed)
- Expectations on when the execution is happening (in an eager or lazy way)
- Other execution details

**Rationale:** The API defined in this document needs to be used by libraries as diverse as Ibis,
Dask, Vaex or cuDF. The data can live in databases, distributed systems, disk or GPU memory.
Any decision that involves assumptions on where the data is stored, or where execution happens
could prevent implementation from adopting the standard.

#### High level APIs

It is out of scope to provide an API designed for interactive use. While interactive use
is a key aspect of dataframes, an API designed for interactive use can be built on top
of the API defined in this document.

Domain or industry specific APIs are also out of scope, but can benefit from the standard
to better interact with the different dataframe implementation.

**Rationale:** Interactive or domain specific users are key in the Python dataframe ecosystem.
But the amount and diversity of users makes it unfeasible to standardize every dataframe feature
that is currently used. In particular, functionality built as syntactic sugar for convenience in
interactive use, or heavily overloaded create very complex APIs. For example, the pandas dataframe
constructor, which accepts a huge number of formats, or its `__getitem__` (e.g. `df[something]`)
which is heavily overloaded. Implementations can provide convenient functionality like this one
for the users they are targeting, but it is out-of-scope for the standard, so the standard is
simple and easy to adopt.


### Non-goals

- Build an API that is appropriate to all users
- Have a unique dataframe implementation for Python
- Standardize functionalities specific to a domain or industry


## Stakeholders

This section provides the list of stakeholders considered for the definition of this API.


### Dataframe library authors

We encourage dataframe libraries in Python to implement the API defined in this document
in their libraries.

The list of known Python dataframe libraries at the time of writing this document is next:

- [cuDF](https://github.com/rapidsai/cudf)
- [Dask](https://dask.org/)
- [datatable](https://github.com/h2oai/datatable)
- [dexplo](https://github.com/dexplo/dexplo/)
- [Eland](https://github.com/elastic/eland)
- [Grizzly](https://github.com/weld-project/weld#grizzly)
- [Ibis](https://ibis-project.org/)
- [Koalas](https://github.com/databricks/koalas)
- [Mars](https://docs.pymars.org/en/latest/)
- [Modin](https://github.com/modin-project/modin)
- [pandas](https://pandas.pydata.org/)
- [PySpark](https://spark.apache.org/docs/latest/api/python/index.html)
- [StaticFrame](https://static-frame.readthedocs.io/en/latest/)
- [Turi Create](https://github.com/apple/turicreate)
- [Vaex](https://vaex.io/)


### Downstream library authors

Authors of libraries that consume dataframes. They can use the API defined in this document
to know how the data contained in a dataframe can be consumed, and which operations are implemented.

A non-exhaustive list of downstream library categories is next:

- Plotting and visualization (e.g. Matplotlib, Bokeh, Altair, Plotly)
- Statistical libraries (e.g. statsmodels)
- Machine learning libraries (e.g. scikit-learn)


### Upstream library authors

Authors of libraries that provide functionality used by dataframes.

A non-exhaustive list of upstream categories is next:

- Data formats, protocols and libraries for data analytics (e.g. Apache Arrow, NumPy)
- Task schedulers (e.g. Dask, Ray, Mars)
- Big data systems (e.g. Spark, Hive, Impala, Presto)
- Libraries for database access (e.g. SQLAlchemy)


### Dataframe power users


This group considers developers of reusable code that use dataframes. For example, developers of
applications that use dataframes. Or authors of libraries that provide specialized dataframe
APIs to be built on top of the standard API.

People using dataframes in an interactive way are considered out of scope. These users include data
analysts, data scientists and other users that are key for dataframes. But this type of user may need
shortcuts, or libraries that take decisions for them to save them time. For example automatic type
inference, or excessive use of very compact syntax like Python squared brackets / `__getitem__`.
Standardizing on such practices can be extremely difficult, and it is out of scope.

With the development of a standard API that targets developers writing reusable code we expected
to also serve data analysts and other interactive users. But in an indirect way, by providing a
standard API where other libraries can be built on top. Including libraries with the syntactic
sugar required for fast analysis of data.


## High-level API overview




## How to read this document




## How to adopt this API




## Definitions




## References
