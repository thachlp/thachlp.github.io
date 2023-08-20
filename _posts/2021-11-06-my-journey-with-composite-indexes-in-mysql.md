---
layout: post
title: My Journey with Composite Indexes in MySQL
tags: [databases, composite index]
comments: true
---

In MySQL, if a single index is solely used to weed out rows to be reviewed, I didn't see much of a difference between a single index and a composite index based on my prior experience. However, I had a personal experience with one last week. I want to share something with you today.

My query like:

~~~
SELECT ... FROM table1 WHERE col1 = A AND col2 = B AND col3 = C join table2 on ...
~~~

The ***table1*** has around ***90M*** rows, but the `SELECT` query takes hours to finish. With some investigation, I can determine where is the above query is slow, so I try to do some analysis to optimize it.

- Firstly, I try to use `SHOW INDEX FROM table` to determine if these columns have indexes, here is the information with `Cardinality` information: __(*)__
  - *col1 = 2572741*
  - *col2 = 2842*
  - *col3 = 1767*

- Secondly, I use the `EXPLAIN` to see how MySQL uses the index. 
Not surprised, only the index of `col1` was used, explain this, maybe `col2` and `col3` have low `Cardinality` so the index wasn't used.

But only 31 estimate rows with `col1` to be examined and the filter is ***0.91%***, it was pretty good.
So why does it slow?

As a workaround, I created index_c(col1, col2, col3).

And like magic, it works as expected, the query time reduces to less than ***20'***. This is amazing!

To verify my solution, I conducted further analysis:
- Compare to __(*)__, here is the information:
  - *index_c_col1 = 2929008*
  - *index_c_col2 = 10827882*
  - *index_c_col3 = 13341158* 

I'm not sure why these numbers are as they are (???)

However, the most important piece of information from the `EXPLAIN` query was the `EXTRA info: Using where; Using index`. According to MySQL documentation:

> The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row.

This explains my solution. When using the single index, MySQL still had to seek the disk to get the data of `col2` and `col3`.

***Follow up:*** I'm unsure if the indexes with `col2 `and `col3` are useful. Do we need these indexes? I'd appreciate any insights from those with experience in this area.

***Conclusion:*** When optimizing a query, we should determine which part of the query is slow, if we are correct, we can be confident to work around it and find the solution.

***Note:*** I see that there is a good article about single index and composite index for reference later: [single-vs-composite-indexes](https://user3141592.medium.com/single-vs-composite-indexes-in-relational-databases-58d0eb045cbe)
