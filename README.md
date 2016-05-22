golang-sql-builder-benchmark
====================

A comparison of popular Go SQL query builders. Provides feature list and benchmarks

# Builders

1. dbr(1.1): https://github.com/gocraft/dbr
2. squirrel: https://github.com/lann/squirrel
3. sqrl: https://github.com/elgris/sqrl
4. gocu: github.com/doug-martin/goqu - just for SELECT query

# Feature list

| feature                    | dbr | squirrel | sqrl | goqu |
|----------------------------|-----|----------|------|------|
| SelectBuilder              | +   | +        | +    | +    |
| InsertBuilder              | +   | +        | +    | +    |
| UpdateBuilder              | +   | +        | +    | +    |
| DeleteBuilder              | +   | +        | +    | +    |
| PostgreSQL support         |     | +        | +    | +    |
| Custom placeholders        |     | +        | +    | +    |
| JOINs support              |     | +        | +    | +    |
| Subquery in query builder  |     | +        | +    | +    |
| Aliases for columns        |     | +        | +    | +    |
| CASE expression            |     | +        | +    | +    |

Some explanations here:
- `Custom placeholders` - ability to use not only `?` placeholders, Useful for PostgreSQL
- `JOINs support` - ability to build JOINs in SELECT queries like `Select("*").From("a").Join("b")`
- `Subquery in query builder` - when you prepare a subquery with one builder and then pass it to another. Something like:
```go
subQ := Select("aa", "bb").From("dd")
qb := Select().Column(subQ).From("a")
```
- `Aliases for columns` - easy way to alias a column, especially if column is specified by subquery:
```go
subQ := Select("aa", "bb").From("dd")
qb := Select().Column(Alias(subQ, "alias")).From("a")
```
- `CASE expression` - syntactic sugar for [CASE expressions](http://dev.mysql.com/doc/refman/5.7/en/case.html)

# Benchmarks

```bash
$ go version
go version go1.6.2 darwin/amd64

$ glide install

$ go test -bench=. -benchmem | column -t
```

on Intel Core i7 1.7 GHz MacBookAir6,2:

```
BenchmarkDbrV1SelectSimple-4          1000000                    1179     ns/op  832    B/op  13   allocs/op
BenchmarkDbrV1SelectConditional-4     1000000                    1870     ns/op  992    B/op  18   allocs/op
BenchmarkDbrV1SelectComplex-4         200000                     6368     ns/op  3272   B/op  52   allocs/op
BenchmarkDbrV1SelectSubquery-4        300000                     4734     ns/op  2761   B/op  38   allocs/op
BenchmarkDbrV1Insert-4                1000000                    2438     ns/op  1008   B/op  17   allocs/op
BenchmarkDbrV1UpdateSetColumns-4      1000000                    2309     ns/op  808    B/op  22   allocs/op
BenchmarkDbrV1UpdateSetMap-4          500000                     2778     ns/op  1156   B/op  24   allocs/op
BenchmarkDbrV1Delete-4                1000000                    1128     ns/op  320    B/op  11   allocs/op

BenchmarkGoquSelectSimple-4           200000                     8136     ns/op  3216   B/op  42   allocs/op
BenchmarkGoquSelectConditional-4      200000                     8313     ns/op  3912   B/op  55   allocs/op
BenchmarkGoquSelectComplex-4          50000                      27199    ns/op  10576  B/op  193  allocs/op

BenchmarkSqrlSelectSimple-4           1000000                    1866     ns/op  952    B/op  15   allocs/op
BenchmarkSqrlSelectConditional-4      500000                     3414     ns/op  1112   B/op  20   allocs/op
BenchmarkSqrlSelectComplex-4          50000                      20820    ns/op  4721   B/op  100  allocs/op
BenchmarkSqrlSelectSubquery-4         100000                     14423    ns/op  3529   B/op  67   allocs/op
BenchmarkSqrlSelectMoreComplex-4      30000                      34693    ns/op  7218   B/op  150  allocs/op
BenchmarkSqrlInsert-4                 500000                     3283     ns/op  1120   B/op  24   allocs/op
BenchmarkSqrlUpdateSetColumns-4       500000                     4932     ns/op  1152   B/op  31   allocs/op
BenchmarkSqrlUpdateSetMap-4           200000                     6130     ns/op  1568   B/op  35   allocs/op
BenchmarkSqrlDelete-4                 1000000                    1345     ns/op  320    B/op  11   allocs/op

BenchmarkSquirrelSelectSimple-4       200000                     9825     ns/op  2488   B/op  51   allocs/op
BenchmarkSquirrelSelectConditional-4  100000                     12942    ns/op  3753   B/op  83   allocs/op
BenchmarkSquirrelSelectComplex-4      30000                      47626    ns/op  12419  B/op  281  allocs/op
BenchmarkSquirrelSelectSubquery-4     30000                      48596    ns/op  9379   B/op  204  allocs/op
BenchmarkSquirrelSelectMoreComplex-4  20000                      72742    ns/op  16900  B/op  386  allocs/op
BenchmarkSquirrelInsert-4             200000                     10871    ns/op  3152   B/op  73   allocs/op
BenchmarkSquirrelUpdateSetColumns-4   100000                     18539    ns/op  4537   B/op  106  allocs/op
BenchmarkSquirrelUpdateSetMap-4       100000                     19922    ns/op  4953   B/op  110  allocs/op
BenchmarkSquirrelDelete-4             200000                     11300    ns/op  2616   B/op  65   allocs/op
```

# Conclusion

If your queries are very simple, pick `dbr`, the fastest one.

If really need immutability of query builder and you're ready to sacrifice extra memory, use `squirrel`, the slowest but most reliable one.

If you like those sweet helpers that `squirrel` provides to ease your query building or if you plan to use the same builder for `PostgreSQL`, take `sqrl` as it's balanced between performance and features.

`goqu` has LOTS of features and ways to build queries. Although it requires stubbing sql connection if you need just to build a query. It can be done with [sqlmock](http://github.com/DATA-DOG/go-sqlmock). Disadvantage: the builder is slow and has TOO MANY features, so building a query may become a nightmare. But if you need total control on everything - this is your choice.