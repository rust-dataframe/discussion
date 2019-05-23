# Work-In-Progress Roundup

Let's look at some dataframe works-in-progress!

The purpose of this document is to generate discussion around initial design for a dataframe library, based on how the existing projects implement some of the [use cases](#use-cases) we've [discussed](https://github.com/rust-dataframe/discussion/issues/3).

The works-in-progress being analyzed are:
* [Utah](https://github.com/kernelmachine/utah)
* [Rust-Dataframe](https://github.com/nevi-me/rust-dataframe) by [@nevi-me](https://github.com/nevi-me)
* [Agnes](https://github.com/agnes-rs/agnes) by [@jblondin](https://github.com/jblondin)
* [Frames](https://github.com/jesskfullwood/frames) by [@jesskfullwood](https://github.com/jesskfullwood)
* [Black-jack](https://github.com/milesgranger/black-jack) by [@milesgranger](https://github.com/milesgranger)

## Use cases

To summarize the discussion [here](https://github.com/rust-dataframe/discussion/issues/3), the use cases we are targeting include:
* **Static heterogeneous typing**: Dataframe types are statically checked, and a dataframe can hold different (arbitrary) types in different columns
* **Static axis labels**: Compile-time (non-stringy) checks on accessing columns in the dataframe
* **Ergonomic data access**: Data is able to be easily processed -- iteration (by row or by column), map, fold, direct indexing
* **Arrow compatibility**: Supporting Apache Arrow at the storage level would improve interoperability
* **Basic SQL manipulation**: i.e. join, concatenate, merge, unique values, filtering, sorting
* **Reshaping**: i.e. melt, pivot, aggregation
* **Input / Output**: Loading from and writing to different formats, preferably in an easily extensible manner
* **Missing value handling**

This list, and the analysis of each library's approach to each use case, are not intended to act as a quantitative comparison of the merits of the individual libraries, but merely as context for further discussion -- a library not covering a specific use case could mean that feature is doable with a particular design but just hasn't been implemented yet.

One other consideration is general ergonomics / usability. These questions will be a bit harder to answer, requiring more experience with the library to fully examine than was initially performed for this analysis:
* How terse is the library usage? How much boilerplate code is necessary?
* How readable is the library code?
* How readable is client code (code that uses the library)?
* How interpretable are compile errors?
* How extensible is the library?

## Project Analysis

### [Utah](https://github.com/kernelmachine/utah)

**Status**: Abandoned (Last code update: Dec 28, 2016)

A `ndarray` `Array2`-backed dataframe library with intriguing 'combinator' (row- or column- iterators) functionality.

Since it's backed by `ndarray`, it assumes all data members are a single type (no heterogeneous columns). `utah` does provide an enum-based (thus non-static) approach to mixed types, with a small set of native supported types (`f64`, `i64`, `i32`, `String`). This also provides some support for missing values (as an enum variant). Arbitrary typing is supported.

Data access in `utah` is handled through a set of iterators called 'combinators'. Iteration can operate over columns or rows, and provides a lazily-evaluated data access interface that should be familiar to users of Rust's standard library. Columns or rows can be referenced by `String`-based labels, which can be specified by user.

No innate support for missing values; missing values can only be handled by wrapping the dataframe type in an `Option` or other enum.

`utah` has basic SQL manipulation functionality, such as concatenation (either row-based or column-based) and joins. No built-in melt / pivot. Various aggregation methods provided, along with a generic function map.

`utah` Dataframes can be created by macro or by reading CSV. Only output appears to be accessing the underlying `ndarray` `Array2` or collecting an iterator as an arbitrary collection (iterators support `FromIterator<T>` for the dataframe type `T`).

No built-in Apache Arrow compatibility, but replacing ndarray with Arrow might be feasible.




### [Rust-DataFrame](https://github.com/nevi-me/rust-dataframe)

**Status**: Active

A dataframe implementation built upon Apache Arrow.

`rust-dataframe` supports the predefined (non-arbitrary) [Apache Arrow types](https://docs.rs/arrow/0.13.0/arrow/datatypes/enum.DataType.html), and does allow for heterogeneous column types. Leveraging Arrow also supplies missing value support.

Column access is `String`-based, and not statically typed -- columns are stored and accessible as types that implement [Array](https://docs.rs/arrow/0.13.0/arrow/array/trait.Array.html).

`rust-dataframe` supports CSV reading / writing, JSON reading, Feather reading, and PostgreSQL reading. It supports aggregations such as sum, max, min, and count, but does no support SQL manipulations such as join, or reshaping operations such as melt / pivot.




### [Agnes](https://github.com/agnes-rs/agnes)

**Status**: Active

`agnes` is a statically-typed dataframe library leveraging heterogeneous lists (see Haskell's [HList](http://hackage.haskell.org/package/HList) and Rust's [frunk](https://github.com/lloydmeta/frunk)).

`agnes` has heterogeneous statically-typed columns, currently supporting arbitrary types. Access to columns is by unit-like structs which must be defined at compile-time by the user. Data from a column is accessible by index (although not through `Index` / `IndexMut` traits) or via iteration. Both of these methods return an `Option`-like struct to handle missing values. Internally, missing values are tracked via bit-mask.

`agnes` supports CSV reading and some limited `serde` serialization support. It provides display functionality for viewing data as well as some basic dataframe statistics.

SQL manipulations such as joins, (vertical) concatenation, and merge (horizontal concatenation) are supports. Melt, pivot, and aggregation functionality is provided. No Apache Arrow integration (currently).

The `agnes` library currently has some ergonomics problems -- adding to and extending the library is painful, and error messages can be unreadable.






### Frames

**Status**: Semi-active (Last update Dec 2018)

`frames` is a statically-typed dataframe library leveraging heterogeneous lists using [frunk](https://github.com/lloydmeta/frunk).

Similar to `agnes`, `frames` has heterogeneous statically-typed columns supporting arbitrary types with unit-like struct labeling (defined at compile-time by user). Data from a column is accessed via index or iteration which provide `Option`s which handle missing values. Internally, missing values are tracked via bit-mask. `frames` includes a row iterator providing a single `HList` of values.

`frames` supports CSV reading / writing and `serde`-based reading / writing.

SQL manipulations such as joins, group-by, and filtering are implemented. No Apache Arrow integration.




### Black-Jack

**Status**: Active

`black-jack` is a dataframe library using the `Any` type for heterogeneous types.

Columns in a `black-jack` dataframe can be heterogeneous and are accessed by `String`-based labels (for columns) or row indices for rows. It supports types `f64`, `i64`, `f32`, `i32`, and `String` (no arbitrary types). When accessing data, the type must be known by the user, and providing an incorrect type will result in a panic. Data from a column is accessible via iteration or indexing (via `Index` and `IndexMut`). Missing values are not supported.

`black-jack` supports CSV reading / writing.

Joins, melt, and pivot are not supported. Filtering and group-by aggregation is supported. No Apache Arrow integration.
