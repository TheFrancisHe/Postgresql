5.3.6. Exclusion Constraints

> Exclusion constraints ensure that if any two rows are compared on the specified columns or expressions using the specified operators, at least one of these operator comparisons will return false or null. The syntax is:
```
// 代表相交的圆 存不进去。

CREATE TABLE circles (
    c circle,
    EXCLUDE USING gist (c WITH &&) // 也就是说，每个圆互相相离。 
);
```
See also CREATE TABLE ... CONSTRAINT ... EXCLUDE for details.

> Adding an exclusion constraint will automatically create an index of the type specified in the constraint declaration.

----------
```
EXCLUDE [ USING index_method ] ( exclude_element WITH operator [, ... ] ) index_parameters [ WHERE ( predicate ) ]
```

// &&代表相交。 exclude的范围相比于unique更大。
>The EXCLUDE clause defines an exclusion constraint, which guarantees that if any two rows are compared on the specified column(s) or expression(s) using the specified operator(s), not all of these comparisons will return TRUE. If all of the specified operators test for equality, this is equivalent to a UNIQUE constraint, although an ordinary unique constraint will be faster. However, exclusion constraints can specify constraints that are more general than simple equality. For example, you can specify a constraint that no two rows in the table contain overlapping circles (see Section 8.8) by using the && operator.


每一个index-method 对于不同的数据类型，对应不同的operator class，比如integer对应inter4 -- operator 某种operator class的一种。

排他索引所使用的operator必须可以互相对调，即对调后结果也必须一致。
>Exclusion constraints are implemented using an index, so each specified operator must be associated with an appropriate operator class (see Section 11.9) for the index access method index_method. The operators are required to be commutative. Each exclude_element can optionally specify an operator class and/or ordering options; these are described fully under CREATE INDEX.

btree的= 不如 unique，所以实践中，经常用的是 GiST or SP-GiST
>The access method must support amgettuple (see Chapter 58); at present this means GIN cannot be used. Although it's allowed, there is little point in using B-tree or hash indexes with an exclusion constraint, because this does nothing that an ordinary unique constraint doesn't do better. So in practice the access method will always be GiST or SP-GiST.


>The predicate allows you to specify an exclusion constraint on a subset of the table; internally this creates a partial index. Note that parentheses are required around the predicate.

其余参考资料：

9种索引用法
https://yq.aliyun.com/articles/111793

阿里云排他约束
https://yq.aliyun.com/articles/3034



