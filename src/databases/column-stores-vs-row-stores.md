### [ColumnStores vs. RowStores: How Different Are They Really?]()

> SIGMOD 2008 Daniel J. Abadi, etc.
>
> https://dl.acm.org/doi/10.1145/1376616.1376712

列存数据库在数仓、决策支持、BI 等分析型应用中被证明比传统行存数据库的表现要好出一个量级以上，其原因也显而易见：列存数据库对于只读查询按需访问所需列，因而更 `IO efficient`。