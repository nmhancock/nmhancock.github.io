--- layout: post title: A short note on Column vs Row storage ---

	I’m writing my own small storage engine for a financial analysis
application. I want to run quantitative finance-style analysis on the market
in Eve-Online, so as to hone my skills at such analysis cheaply. Eve-Central
has a historic dataset of Eve-Online market data which, while not
comprehensive, still measure in at 2.7 Terabytes in decompressed, CSV form.
The data are pretty typical star-schema fact-table style data. The tuples
consist of a primary key, 4 ‘foreign keys’, 2 time stamps, and 8
miscellaneous fact fields for a total of 15 fields. One day’s worth of data
is 1.5 GB in the aforementioned decompressed, CSV format. I’m comparing row
and column storage formats for two criteria: space consumed and
time-to-write. Note that all output formats are binary, and all times are
determined by the minimum of four consecutive runs.

| Format         | Size (MB) |   Time to Write (s)    |
|CSV (input)     |   1.5G    |         0              |
|Row             |   600M    |        8.6             |
|Row (lz4 -9)    |   209M    | 8.6 + 20.59 = 29.19    |
|Row (gz – 9)    |   230M    | 8.6 + 196.81 = 205.41  |
|Column          |   600M    |       13.94            |
|Column (lz4 -9) |   174M    | 13.94 + 33.23 = 47.17  |
|Column (gz – 9) |   146M    |  13.94 + 171 = 184.94  |

	Note that the times for Row versus Column compression aren’t totally
accurate, as I had all the data written to the row or column storage and
then compressed in one fell swoop, whereas in a real environment the
compression would be piece-wise.

	For lz4 breaking these data into columns decreases space utilization by
16.75% while increasing the bulk loading time by 61.60%. For gzip the
numbers are 36.52% and -9.97% (the bulk loading time went down)
respectively. It remains to be seen why the two algorithms perform so
differently with respect to loading time. On a cursory glance, the
difference in CPU utilization doesn’t seem to explain the discrepancy.

![Newport
Beach](https://images.unsplash.com/photo-1464149060084-7fcb2dd7650a?dpr=2&auto=format&fit=crop&w=1500&h=1000&q=80&cs=tinysrgb&crop=)
Photo credit: ![Austin Neill](https://unsplash.com/@arstyy)
