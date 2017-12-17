---
title: "The Wedge Pattern"
date: 2017-12-16T22:30:48-07:00
draft: false
tags: ["R", "bad data", "Excel is terrible"]
summary: "Fixing the bad data wedge pattern that comes from Excel being awful."
---

Let's face it: Excel doesn't respect data.
Unfortunately, tons of people use it and somehow successfully make very important decisions with it.
That means that we have to live with the fallout from that if we're trying to use that badly
formatted data they're producing in meaningful ways.

One of the worst data patterns that comes out of Excel can be called the "Wedge Pattern".
This happens when somebody just throws a bunch of data into a PivotTable and calls it "good enough",
without adhering to any sort of common data standard, like repeating values down the entire column,
or having data (or headers) start in the first row of the first column.
It's a real problem, and when it's exported to CSV it looks something like this:

```
,,,,,,
,"Last Name","First Name","Email","Item","Quantity","Unit Price","Subtotal"
,"Smith","Sue","sue@email.org","Coffee",1,1.00,1.00
,,,"Filters",5,1.00,5.00
,,,,,"Subtotal",6.00
,"Bob",,"Coffee",2,1.00,2.00
,,,"Filters",10,1.00,10.00
,,,,,"Subtotal",12.00
,"Jones","Tom","tom@email.org","Coffee",3,1.00,3.00
,,,,,"Subtotal",3.00
,,"Ana","ana@email.org","Coffee",8,1.00.8.00
,,,,,"Subtotal",8.00
,,,,,"Total",29.00
```

When it's broken out into a table, the badness of this pattern becomes pretty clear.
I'm going to exclude the first row and column, since they aren't necessary for showing the
actual "wedge" shape here.

Last Name | First Name | Email         | Item    | Quantity | Unit Price | Subtotal
----------|------------|---------------|---------|----------|------------|---------
Smith     | Sue        | sue@email.org | Coffee  | 1        | 1.00       | 1.00
          |            |               | Filters | 5        | 1.00       | 5.00
          |            |               |         |          | Subtotal   | 6.00
          | Bob        |               | Coffee  | 2        | 1.00       | 2.00
          |            |               | Filters | 10       | 1.00       | 10.00
          |            |               |         |          | Subtotal   | 12.00
Jones     | Tom        | tom@email.org | Coffee  | 3        | 1.00       | 3.00
          |            |               |         |          | Subtotal   | 3.00
          | Ana        | ana@email.org | Coffee  | 8        | 1.00       | 8.00
          |            |               |         |          | Subtotal   | 8.00
          |            |               |         |          | Total      | 29.00

Fortunately, a little bit of negotiating with this data in R using the `tidyverse` packages
can solve this pretty quickly.

```r
# Assume the CSV has already been read in and we've already removed
# the empty first column and row, moved the headers to their proper
# place, and stuck it all in a data.frame named `dat`.
#
# We still have yet to fix the column types.

library(tidyverse)

dat <-
  dat %>%
  fill(`Last Name`, `First Name`)

# At this point we're doing pretty good, but we still have to deal
# with the Subtotal and Total rows. We'll fall back on `plyr` for
# this. The reason we need to go there is because of "Bob".
# He doesn't have an email address, and we don't want to carry
# "Sue's" email address down to him. Using `ddply` will let us
# be flexible in how we handle these different things.

library(plyr)

dat <- ddply(
  dat,
  c('Last Name', 'First Name'),
  function(x) {
    x <- x[!is.na(x$Item), ]
    x <- x %>% fill(Email)
    return(x)
  }
)

dat$`Unit Price` <- as_numeric(dat$`Unit Price`)
```

This one is a fairly contrived example, but more complicated specimen of this ilk exist, which
may require you to begin by filling just one or two columns, performing other operations, and
then continuing with the fill.

So, how can you prevent this in the first place? Evangelism. Pure and simple. Show people a
better way and make them believe that it is a better way.
However, if they're really that dedicated to Excel, you may not be able to sway them.

Good luck!
