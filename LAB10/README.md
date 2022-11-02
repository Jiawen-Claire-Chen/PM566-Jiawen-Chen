LAB10
================
Jiawen
2022-11-02

``` r
# install.packages(c("RSQLite", "DBI"))
if(!require(RSQLite)) install.packages("RSQLite")
```

    ## Loading required package: RSQLite

``` r
library(RSQLite)
library(DBI)
```

``` r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
actor <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/actor.csv")
rental <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/rental.csv")
customer <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/customer.csv")
payment <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/payment_p2007_01.csv")

# Copy data.frames to database
dbWriteTable(con, "actor", actor)
dbWriteTable(con, "rental", rental)
dbWriteTable(con, "customer", customer)
dbWriteTable(con, "payment", payment)
```

``` sql
PRAGMA table_info(actor)
```

| cid | name        | type    | notnull | dflt_value |  pk |
|:----|:------------|:--------|--------:|:-----------|----:|
| 0   | actor_id    | INTEGER |       0 | NA         |   0 |
| 1   | first_name  | TEXT    |       0 | NA         |   0 |
| 2   | last_name   | TEXT    |       0 | NA         |   0 |
| 3   | last_update | TEXT    |       0 | NA         |   0 |

4 records

# exercise 1

Retrive the actor ID, first name and last name for all actors using the
actor table. Sort by last name and then by first name.

SELECT FROM ORDER by

``` r
dbGetQuery(con,"
SELECT actor_id, first_name, last_name
FROM actor
ORDER by last_name, first_name
lIMIT 5
 ")
```

    ##   actor_id first_name last_name
    ## 1       58  CHRISTIAN    AKROYD
    ## 2      182     DEBBIE    AKROYD
    ## 3       92    KIRSTEN    AKROYD
    ## 4      118       CUBA     ALLEN
    ## 5      145        KIM     ALLEN

\#try in SQL directly

``` sql
SELECT actor_id, first_name, last_name
FROM actor
ORDER by last_name, first_name
```

| actor_id | first_name | last_name |
|---------:|:-----------|:----------|
|       58 | CHRISTIAN  | AKROYD    |
|      182 | DEBBIE     | AKROYD    |
|       92 | KIRSTEN    | AKROYD    |
|      118 | CUBA       | ALLEN     |
|      145 | KIM        | ALLEN     |
|      194 | MERYL      | ALLEN     |
|       76 | ANGELINA   | ASTAIRE   |
|      112 | RUSSELL    | BACALL    |
|      190 | AUDREY     | BAILEY    |
|       67 | JESSICA    | BAILEY    |

Displaying records 1 - 10

\#exercise 2 Retrive the actor ID, first name, and last name for actors
whose last name equals ‘WILLIAMS’ or ‘DAVIS’.

``` r
dbGetQuery(con,"
SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name IN ('WILLIAMS', 'DAVIS')
ORDER BY last_name
 ")
```

    ##   actor_id first_name last_name
    ## 1        4   JENNIFER     DAVIS
    ## 2      101      SUSAN     DAVIS
    ## 3      110      SUSAN     DAVIS
    ## 4       72       SEAN  WILLIAMS
    ## 5      137     MORGAN  WILLIAMS
    ## 6      172    GROUCHO  WILLIAMS

\#Exercise 3 Write a query against the rental table that returns the IDs
of the customers who rented a film on July 5, 2005 (use the
rental.rental_date column, and you can use the date() function to ignore
the time component). Include a single row for each distinct customer ID.

``` r
dbGetQuery(con,"
SELECT DISTINCT customer_id
FROM rental
WHERE date(rental_date) = '2005-07-05'
")
```

    ##    customer_id
    ## 1          565
    ## 2          242
    ## 3           37
    ## 4           60
    ## 5          594
    ## 6            8
    ## 7          490
    ## 8          476
    ## 9          322
    ## 10         298
    ## 11         382
    ## 12         138
    ## 13         520
    ## 14         536
    ## 15         114
    ## 16         111
    ## 17         296
    ## 18         586
    ## 19         349
    ## 20         397
    ## 21         369
    ## 22         421
    ## 23         142
    ## 24         169
    ## 25         348
    ## 26         553
    ## 27         295

``` r
dbGetQuery(con,"
SELECT customer_id,
count(*) AS N
FROM rental
WHERE date(rental_date) = '2005-07-05'
GROUP BY customer_id
")
```

    ##    customer_id N
    ## 1            8 1
    ## 2           37 1
    ## 3           60 1
    ## 4          111 1
    ## 5          114 1
    ## 6          138 1
    ## 7          142 1
    ## 8          169 1
    ## 9          242 1
    ## 10         295 1
    ## 11         296 1
    ## 12         298 1
    ## 13         322 1
    ## 14         348 1
    ## 15         349 1
    ## 16         369 1
    ## 17         382 1
    ## 18         397 1
    ## 19         421 1
    ## 20         476 1
    ## 21         490 1
    ## 22         520 1
    ## 23         536 1
    ## 24         553 1
    ## 25         565 1
    ## 26         586 1
    ## 27         594 1

\#Exercise 4.1 Construct a query that retrives all rows from the payment
table where the amount is either 1.99, 7.99, 9.99.

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount IN(1.99, 7.99, 9.99)
")
```

    ##     payment_id customer_id staff_id rental_id amount               payment_date
    ## 1        16050         269        2         7   1.99 2007-01-24 21:40:19.996577
    ## 2        16056         270        1       193   1.99 2007-01-26 05:10:14.996577
    ## 3        16081         282        2        48   1.99 2007-01-25 04:49:12.996577
    ## 4        16103         294        1       595   1.99 2007-01-28 12:28:20.996577
    ## 5        16133         307        1       614   1.99 2007-01-28 14:01:54.996577
    ## 6        16158         316        1      1065   1.99 2007-01-31 07:23:22.996577
    ## 7        16160         318        1       224   9.99 2007-01-26 08:46:53.996577
    ## 8        16161         319        1        15   9.99 2007-01-24 23:07:48.996577
    ## 9        16180         330        2       967   7.99 2007-01-30 17:40:32.996577
    ## 10       16206         351        1      1137   1.99 2007-01-31 17:48:40.996577
    ## 11       16210         354        2       158   1.99 2007-01-25 23:55:37.996577
    ## 12       16240         369        2       913   7.99 2007-01-30 09:33:24.996577
    ## 13       16275         386        1       583   7.99 2007-01-28 10:17:21.996577
    ## 14       16277         387        1       697   7.99 2007-01-29 00:32:30.996577
    ## 15       16289         391        1       891   7.99 2007-01-30 06:11:38.996577
    ## 16       16302         400        2       516   1.99 2007-01-28 01:40:13.996577
    ## 17       16306         401        2       811   1.99 2007-01-29 17:59:08.996577
    ## 18       16307         402        2       801   1.99 2007-01-29 16:04:16.996577
    ## 19       16314         407        1       619   7.99 2007-01-28 14:20:52.996577
    ## 20       16320         411        2       972   1.99 2007-01-30 18:49:33.996577
    ## 21       16348         429        2       601   7.99 2007-01-28 12:36:48.996577
    ## 22       16354         432        2       326   7.99 2007-01-26 23:38:37.996577
    ## 23       16360         435        1       757   7.99 2007-01-29 08:58:13.996577
    ## 24       16362         436        1        45   7.99 2007-01-25 04:28:05.996577
    ## 25       16373         439        1       786   9.99 2007-01-29 13:45:54.996577
    ## 26       16393         449        1       849   7.99 2007-01-29 23:51:33.996577
    ## 27       16400         452        1       726   1.99 2007-01-29 04:33:55.996577
    ## 28       16401         454        1       735   7.99 2007-01-29 06:36:39.996577
    ## 29       16405         457        2      1024   7.99 2007-01-31 01:58:45.996577
    ## 30       16414         463        1       560   1.99 2007-01-28 07:21:28.996577
    ## 31       16416         464        2       373   1.99 2007-01-27 06:44:51.996577
    ## 32       16426         469        2       506   7.99 2007-01-28 00:37:45.996577
    ## 33       16428         469        2       936   1.99 2007-01-30 12:21:15.996577
    ## 34       16439         474        1       816   7.99 2007-01-29 18:55:05.996577
    ## 35       16446         479        1       709   7.99 2007-01-29 02:16:27.996577
    ## 36       16449         480        2       822   9.99 2007-01-29 20:04:26.996577
    ## 37       16469         493        1       543   7.99 2007-01-28 05:12:00.996577
    ## 38       16475         497        1      1100   7.99 2007-01-31 12:31:47.996577
    ## 39       16484         501        1       605   1.99 2007-01-28 13:07:36.996577
    ## 40       16489         503        2       109   1.99 2007-01-25 17:08:46.996577
    ## 41       16521         516        1       571   1.99 2007-01-28 08:46:07.996577
    ## 42       16546         533        2       190   1.99 2007-01-26 04:39:54.996577
    ## 43       16561         541        1      1021   7.99 2007-01-31 01:44:41.996577
    ## 44       16566         543        2       476   1.99 2007-01-27 21:00:02.996577
    ## 45       16575         547        2      1094   1.99 2007-01-31 11:32:15.996577
    ## 46       16583         550        2       922   7.99 2007-01-30 10:24:21.996577
    ## 47       16584         551        2       155   7.99 2007-01-25 23:43:31.996577
    ## 48       16615         569        1       647   1.99 2007-01-28 17:51:18.996577
    ## 49       16617         570        2      1060   7.99 2007-01-31 06:50:09.996577
    ## 50       16618         571        1       228   9.99 2007-01-26 09:22:54.996577
    ## 51       16620         572        2       559   7.99 2007-01-28 07:07:28.996577
    ## 52       16659         596        2       782   1.99 2007-01-29 13:07:23.996577
    ## 53       16669         204        1      1016   1.99 2007-01-31 01:18:09.996577
    ## 54       16680           3        1       435   1.99 2007-01-27 15:45:35.996577
    ## 55       16684           5        1      1142   1.99 2007-01-31 18:15:04.996577
    ## 56       16702          14        1       346   9.99 2007-01-27 03:03:07.996577
    ## 57       16717          19        2       110   9.99 2007-01-25 17:12:15.996577
    ## 58       16724          20        2       546   1.99 2007-01-28 05:44:51.996577
    ## 59       16736          25        1        90   7.99 2007-01-25 12:59:51.996577
    ## 60       16743          29        2       194   1.99 2007-01-26 05:20:59.996577
    ## 61       16773          49        2        96   1.99 2007-01-25 15:00:45.996577
    ## 62       16790          53        2       856   9.99 2007-01-30 00:29:47.996577
    ## 63       16796          55        1      1027   9.99 2007-01-31 02:14:45.996577
    ## 64       16802          57        2       152   9.99 2007-01-25 23:09:36.996577
    ## 65       16805          58        2       276   7.99 2007-01-26 15:44:33.996577
    ## 66       16811          60        2       706   1.99 2007-01-29 01:34:15.996577
    ## 67       16822          67        2       331   9.99 2007-01-27 00:50:52.996577
    ## 68       16825          69        2       765   1.99 2007-01-29 10:07:00.996577
    ## 69       16828          71        1       272   9.99 2007-01-26 14:55:37.996577
    ## 70       16837          76        2       574   1.99 2007-01-28 09:12:54.996577
    ## 71       16840          77        1       419   1.99 2007-01-27 13:43:37.996577
    ## 72       16856          85        1       690   9.99 2007-01-28 23:23:19.996577
    ## 73       16858          86        1        66   1.99 2007-01-25 08:03:38.996577
    ## 74       16877         101        1       468   9.99 2007-01-27 19:41:36.996577
    ## 75       16880         102        2       562   1.99 2007-01-28 07:29:47.996577
    ## 76       16881         103        1       240   7.99 2007-01-26 11:08:49.996577
    ## 77       16882         103        1       658   9.99 2007-01-28 18:51:49.996577
    ## 78       16886         105        2       473   7.99 2007-01-27 20:05:00.996577
    ## 79       16899         109        1      1061   7.99 2007-01-31 06:56:24.996577
    ## 80       16902         110        1       515   7.99 2007-01-28 01:38:36.996577
    ## 81       16903         110        2       538   1.99 2007-01-28 04:49:31.996577
    ## 82       16922         120        2        68   7.99 2007-01-25 08:15:57.996577
    ## 83       16944         131        2       944   7.99 2007-01-30 13:54:50.996577
    ## 84       16960         142        1       575   9.99 2007-01-28 09:24:35.996577
    ## 85       16966         146        2       762   7.99 2007-01-29 09:44:17.996577
    ## 86       16980         154        2       865   7.99 2007-01-30 02:08:10.996577
    ## 87       16990         159        2       549   1.99 2007-01-28 06:04:03.996577
    ## 88       16999         162        1       285   1.99 2007-01-26 18:10:06.996577
    ## 89       17002         164        2      1011   1.99 2007-01-31 00:34:05.996577
    ## 90       17004         166        1       662   1.99 2007-01-28 19:37:57.996577
    ## 91       17015         171        2       804   9.99 2007-01-29 16:38:50.996577
    ## 92       17026         176        1       663   1.99 2007-01-28 19:51:28.996577
    ## 93       17027         176        1      1062   7.99 2007-01-31 07:06:46.996577
    ## 94       17038         184        1       567   1.99 2007-01-28 08:24:46.996577
    ## 95       17042         186        1       581   1.99 2007-01-28 09:48:55.996577
    ## 96       17044         187        1       252   7.99 2007-01-26 13:08:19.996577
    ## 97       17050         192        1       895   1.99 2007-01-30 07:19:09.996577
    ## 98       17054         194        2       677   7.99 2007-01-28 21:28:34.996577
    ## 99       17058         196        1      1053   1.99 2007-01-31 05:41:10.996577
    ## 100      17062         197        2       649   1.99 2007-01-28 18:04:11.996577
    ## 101      17072         199        1       499   7.99 2007-01-27 23:33:33.996577
    ## 102      17073         200        2       270   9.99 2007-01-26 14:49:22.996577
    ## 103      17077         209        2       340   9.99 2007-01-27 02:23:51.996577
    ## 104      17083         214        1       242   1.99 2007-01-26 11:33:34.996577
    ## 105      17130         239        1      1022   7.99 2007-01-31 01:45:11.996577
    ## 106      17135         241        1       627   7.99 2007-01-28 15:33:09.996577
    ## 107      17142         244        1       797   1.99 2007-01-29 15:40:43.996577
    ## 108      17145         245        1       519   7.99 2007-01-28 01:50:59.996577
    ## 109      17157         248        2       330   7.99 2007-01-27 00:43:56.996577
    ## 110      17169         251        1       309   1.99 2007-01-26 21:06:36.996577
    ## 111      17201         266        1        86   1.99 2007-01-25 12:04:38.996577

\#exercise Exercise 4.2 Construct a query that retrives all rows from
the payment table where the amount is greater then 5

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount >5 AND amount < 8
LIMIT 10
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2       16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 3       16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 4       16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 5       16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 6       16082         282        2       282   6.99 2007-01-26 17:24:52.996577
    ## 7       16086         284        1      1145   6.99 2007-01-31 18:42:11.996577
    ## 8       16087         286        2        81   6.99 2007-01-25 10:43:45.996577
    ## 9       16092         288        2       427   6.99 2007-01-27 14:38:30.996577
    ## 10      16094         288        2       565   5.99 2007-01-28 07:54:57.996577

\#Exercise 5 Retrive all the payment IDs and their amount from the
customers whose last name is ‘DAVIS’.

``` r
dbGetQuery(con,"
PRAGMA table_info(customer)
")
```

    ##    cid        name    type notnull dflt_value pk
    ## 1    0 customer_id INTEGER       0         NA  0
    ## 2    1    store_id INTEGER       0         NA  0
    ## 3    2  first_name    TEXT       0         NA  0
    ## 4    3   last_name    TEXT       0         NA  0
    ## 5    4       email    TEXT       0         NA  0
    ## 6    5  address_id INTEGER       0         NA  0
    ## 7    6  activebool    TEXT       0         NA  0
    ## 8    7 create_date    TEXT       0         NA  0
    ## 9    8 last_update    TEXT       0         NA  0
    ## 10   9      active INTEGER       0         NA  0

``` r
dbGetQuery(con,"
SELECT c.customer_id, c.last_name, p.payment_id, p.amount
FROM customer AS c INNER JOIN payment AS p
  ON c.customer_id = p.customer_id
WHERE c.last_name == 'DAVIS'
/*where c.last_name == 'DAVIS' */
")
```

    ##   customer_id last_name payment_id amount
    ## 1           6     DAVIS      16685   4.99
    ## 2           6     DAVIS      16686   2.99
    ## 3           6     DAVIS      16687   0.99

\#Exercise 6.1 Use COUNT(\*) to count the number of rows in rental

``` r
dbGetQuery(con,"
SELECT COUNT(*) AS count
FROM rental
")
```

    ##   count
    ## 1 16044

\#Exercise 6.2 Use COUNT(\*) and GROUP BY to count the number of rentals
for each customer_id

``` r
dbGetQuery(con,"
SELECT customer_id, COUNT(*) AS count
FROM rental
GROUP BY customer_id
LIMIT 8
")
```

    ##   customer_id count
    ## 1           1    32
    ## 2           2    27
    ## 3           3    26
    ## 4           4    22
    ## 5           5    38
    ## 6           6    28
    ## 7           7    33
    ## 8           8    24

\#Exercise 6.3 Repeat the previous query and sort by the count in
descending order

``` r
dbGetQuery(con,"
SELECT customer_id,
COUNT(*) AS count
FROM rental
GROUP BY customer_id
ORDER BY count DESC
LIMIT 8
")
```

    ##   customer_id count
    ## 1         148    46
    ## 2         526    45
    ## 3         236    42
    ## 4         144    42
    ## 5          75    41
    ## 6         469    40
    ## 7         197    40
    ## 8         468    39

\#Exercise 6.4 Repeat the previous query but use HAVING to only keep the
groups with 40 or more.

``` r
dbGetQuery(con,"
SELECT customer_id,
COUNT(*) AS count
FROM rental
GROUP BY customer_id
ORDER BY count DESC
LIMIT 8
")
```

    ##   customer_id count
    ## 1         148    46
    ## 2         526    45
    ## 3         236    42
    ## 4         144    42
    ## 5          75    41
    ## 6         469    40
    ## 7         197    40
    ## 8         468    39

\#Exercise 7.1 Modify the above query to do those calculations for each
customer_id

``` r
dbGetQuery(con,"
SELECT MAX(amount) AS maxpayment,
       MIN(amount) AS minpayment,
       SUM(amount) AS sumpayment,
       AVG(amount) AS avgpayment
FROM payment
")
```

    ##   maxpayment minpayment sumpayment avgpayment
    ## 1      11.99       0.99    4824.43   4.169775

\#Exercise 7.2 do those calculations for each customers Modify the above
query to only keep the customer_ids that have more then 5 payments

``` r
dbGetQuery(con,"
SELECT customer_id,
       COUNT(*) AS N,
       MAX(amount) AS maxpayment,
       MIN(amount) AS minpayment,
       SUM(amount) AS sumpayment,
       AVG(amount) AS avgpayment
FROM payment
GROUP BY customer_id
HAVING N>5
")
```

    ##    customer_id N maxpayment minpayment sumpayment avgpayment
    ## 1           19 6       9.99       0.99      26.94   4.490000
    ## 2           53 6       9.99       0.99      26.94   4.490000
    ## 3          109 7       7.99       0.99      27.93   3.990000
    ## 4          161 6       5.99       0.99      17.94   2.990000
    ## 5          197 8       3.99       0.99      20.92   2.615000
    ## 6          207 6       6.99       0.99      17.94   2.990000
    ## 7          239 6       7.99       2.99      33.94   5.656667
    ## 8          245 6       8.99       0.99      28.94   4.823333
    ## 9          251 6       4.99       1.99      19.94   3.323333
    ## 10         269 6       6.99       0.99      18.94   3.156667
    ## 11         274 6       5.99       2.99      24.94   4.156667
    ## 12         371 6       6.99       0.99      25.94   4.323333
    ## 13         506 7       8.99       0.99      28.93   4.132857
    ## 14         596 6       6.99       0.99      22.94   3.823333

\#Cleanup Run the following chunk to disconnect from the connection.

``` r
dbDisconnect(con)
```
