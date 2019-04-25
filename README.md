---
title: "The Lovecraftian Guide to stats::reshape"
output:
  html_document:
    self_contained: no
---



<img src="cthulu_r.png" width="251" />

Dear reader, I hope to show once and for all that the `reshape` function included with R in the 'stats' package is not what it purports to be. It is much more.

## Widetol Ong

Gaze upon this wide dataset and despair:


```r
dat <- read.table(text="
id1 id2 name_1 name_2 loc_1 loc_2
1   1   jim    max    aus   us
1   2   bob    lou    nz    aus
2   1   sue    sally  nz    us
2   2   ann    tim    aus   aus",
header=TRUE, stringsAsFactors=FALSE)

dat
```

```
##   id1 id2 name_1 name_2 loc_1 loc_2
## 1   1   1    jim    max   aus    us
## 2   1   2    bob    lou    nz   aus
## 3   2   1    sue  sally    nz    us
## 4   2   2    ann    tim   aus   aus
```
You shudder as you open your notebook, hurriedly scrawling down notes on the identifiers and sets of varying columns. A cold sweat beads across your brow as you sense the non-Euclidean geometry underlying the data structure.


```r
ids  <- c("id1","id2")
vary <- list(
          name = c("name_1","name_2"),
          loc  = c("loc_1","loc_2") 
        )
```

You feel marginally reassured that these notes will come in handy if you can survive the evening. You fumble through your memories, aligning what little you know to the answers you seek.

"*Each set of variables are grouped together, under the respective names specified manually in the list*  
*- name_N grouped to name*  
*- loc_N grouped to loc*"


```r
reshape(
  dat,
  idvar = ids,
  varying = vary,
  v.names = names(vary),
  direction = "long"
)
```

```
##       id1 id2 time  name loc
## 1.1.1   1   1    1   jim aus
## 1.2.1   1   2    1   bob  nz
## 2.1.1   2   1    1   sue  nz
## 2.2.1   2   2    1   ann aus
## 1.1.2   1   1    2   max  us
## 1.2.2   1   2    2   lou aus
## 2.1.2   2   1    2 sally  us
## 2.2.2   2   2    2   tim aus
```

The horror of your circumstances become apparent when you realise you this could be specified long-hand in the same archaic inline list format.


```r
reshape(
  dat,
  idvar = c("id1","id2"),
  varying = list(c("name_1","name_2"), c("loc_1","loc_2")),
  v.names = c("name","loc"),
  direction = "long"
)
```

```
##       id1 id2 time  name loc
## 1.1.1   1   1    1   jim aus
## 1.2.1   1   2    1   bob  nz
## 2.1.1   2   1    1   sue  nz
## 2.2.1   2   2    1   ann aus
## 1.1.2   1   1    2   max  us
## 1.2.2   1   2    2   lou aus
## 2.1.2   2   1    2 sally  us
## 2.2.2   2   2    2   tim aus
```

The machinery of the beast however knows no bounds. It can act without explicit input from mere humans. Columns can be aligned automatically, as if by guessing their structure. Again, you consult your notes:

"*Automatic output names and times can be guessed by using one long vector of all the names used as input.  
Use `sep="_"` when column names are specified in the format `name_time` . `sep=` can be any single character.*  


```r
reshape(
  dat,
  idvar = ids,
  varying = unlist(vary),
  sep = "_",
  direction = "long"
)
```

```
##       id1 id2 time  name loc
## 1.1.1   1   1    1   jim aus
## 1.2.1   1   2    1   bob  nz
## 2.1.1   2   1    1   sue  nz
## 2.2.1   2   2    1   ann aus
## 1.1.2   1   1    2   max  us
## 1.2.2   1   2    2   lou aus
## 2.1.2   2   1    2 sally  us
## 2.2.2   2   2    2   tim aus
```
A scream echoes in the distance. Yet another variation on the above code presents itself to you.


```r
datlong <- reshape(
  dat,
  idvar = c("id1","id2"),
  varying = c("name_1","name_2","loc_1","loc_2"),
  sep = "_",
  direction = "long"
)
```

## Longt Owide

We have merely scratched the surface of the depravity stored within `reshape`. Reversing from a wide structure to a long structure would test the resolve of all who dare to attempt it.

Your notes provide wisdom collected from others' failings before you:

*You do **not** need to specify `varying=` when converting from a long file to a wide file.  
An extra compulsory argument of `timevar=` (time variable) is required to tell `reshape` how to label and split the long data. If you do not have a time variable, you must add one.*

Taking the `datlong` file from above and consulting the ancient lore, you try to enunciate some code.


```r
reshape(
  datlong,
  idvar = ids,
  timevar = "time",
  direction = "wide",
  sep="_"
)
```

```
##       id1 id2 name_1 loc_1 name_2 loc_2
## 1.1.1   1   1    jim   aus    max    us
## 1.2.1   1   2    bob    nz    lou   aus
## 2.1.1   2   1    sue    nz  sally    us
## 2.2.1   2   2    ann   aus    tim   aus
```

Time is a column, but it is no more or less time. Foolishly, you can remove it if you wish to incur the wrath of `reshape`.


```r
datlong$time <- NULL

reshape(
  datlong,
  idvar = ids,
  timevar = "time",
  direction = "wide",
  sep="_"
)
```

```
## Error in `[.data.frame`(data, , timevar): undefined columns selected
```
Returning time to its rightful position will please the Old Ones. Your notes give you guidance again on appropriate trinkets which amuse their sensibilities.

*Add a numeric counter using `seq_along` within each group of specified `ids`.*


```r
datlong$time <- ave(seq(nrow(datlong)), datlong[ids], FUN=seq_along)

reshape(
  datlong,
  idvar = ids,
  timevar = "time",
  direction = "wide",
  sep="_"
)
```

```
##       id1 id2 name_1 loc_1 name_2 loc_2
## 1.1.1   1   1    jim   aus    max    us
## 1.2.1   1   2    bob    nz    lou   aus
## 2.1.1   2   1    sue    nz  sally    us
## 2.2.1   2   2    ann   aus    tim   aus
```

## Uln Nilgh'ri

One may make it this far. One may not. It is not only foolishness which drives us onward, but an unquenchable need. The notes which were once written clearly become more blurred, but strangely still useful.

*If you don't specify an id when reshaping wide-to-long, `reshape` will assign a basic row counter from 1-to-`nrow(dat)` as an `idvar=` and will treat everything else as non-varying. All the same strategies as described above work the same.*


```r
# add another couple of variables to be unchanging
dat[paste0("x",1:4)] <- list(1,2,3,4)

reshape(
  dat,
  varying = vary,
  v.names = names(vary),
  direction = "long"
)
```

```
##     id1 id2 x1 x2 x3 x4 time  name loc id
## 1.1   1   1  1  2  3  4    1   jim aus  1
## 2.1   1   2  1  2  3  4    1   bob  nz  2
## 3.1   2   1  1  2  3  4    1   sue  nz  3
## 4.1   2   2  1  2  3  4    1   ann aus  4
## 1.2   1   1  1  2  3  4    2   max  us  1
## 2.2   1   2  1  2  3  4    2   lou aus  2
## 3.2   2   1  1  2  3  4    2 sally  us  3
## 4.2   2   2  1  2  3  4    2   tim aus  4
```

```r
reshape(
  dat,
  varying = unlist(vary),
  sep = "_",
  direction = "long"
)
```

```
##     id1 id2 x1 x2 x3 x4 time  name loc id
## 1.1   1   1  1  2  3  4    1   jim aus  1
## 2.1   1   2  1  2  3  4    1   bob  nz  2
## 3.1   2   1  1  2  3  4    1   sue  nz  3
## 4.1   2   2  1  2  3  4    1   ann aus  4
## 1.2   1   1  1  2  3  4    2   max  us  1
## 2.2   1   2  1  2  3  4    2   lou aus  2
## 3.2   2   1  1  2  3  4    2 sally  us  3
## 4.2   2   2  1  2  3  4    2   tim aus  4
```

We are all stuck on this one timeline. Drawing upon multiple columns of time has challenged even the most seasoned practician. You don't recall writing notes on this topic, but they somehow still exist:

*Reshape can accept multiple time variables via `interaction`. Consider this example:  
for each `id1` break out to a separate column for each combination of (`id2` and `time`).*


```r
intvars <- c("id2","time")

reshape(
  cbind(datlong, timeint=interaction(datlong[intvars],sep="_")),
  idvar = "id1",
  timevar = "timeint",
  direction = "wide",
  sep = "_",
  drop = intvars
)
```

```
##       id1 name_1_1 loc_1_1 name_2_1 loc_2_1 name_1_2 loc_1_2 name_2_2 loc_2_2
## 1.1.1   1      jim     aus      bob      nz      max      us      lou     aus
## 2.1.1   2      sue      nz      ann     aus    sally      us      tim     aus
```

Groups of humans, groups of others, groups of unclear origin. `reshape` cares not.

*To group every 'n' variables together in a group, pass in a `list` representing a sequential counter that has been split into chunks every 'n' values.*



```r
varygrp <- split(3:10, (3:10 + 1) %/% 2)
names(varygrp) <- paste0("grp", seq_along(varygrp))
varygrp
```

```
## $grp1
## [1] 3 4
## 
## $grp2
## [1] 5 6
## 
## $grp3
## [1] 7 8
## 
## $grp4
## [1]  9 10
```

```r
reshape(
  dat,
  idvar=ids,
  varying=varygrp,
  v.names=names(varygrp),
  direction="long"
)
```

```
##       id1 id2 time  grp1 grp2 grp3 grp4
## 1.1.1   1   1    1   jim  aus    1    3
## 1.2.1   1   2    1   bob   nz    1    3
## 2.1.1   2   1    1   sue   nz    1    3
## 2.2.1   2   2    1   ann  aus    1    3
## 1.1.2   1   1    2   max   us    2    4
## 1.2.2   1   2    2   lou  aus    2    4
## 2.1.2   2   1    2 sally   us    2    4
## 2.2.2   2   2    2   tim  aus    2    4
```


Begin with what you know. For soon you will not know even what you once knew. A new set of data is required to explain what lays before you now.


```r
dat <- read.table(text="
d xa xb ya yb
1 1  1  3  6  8
2 2  2  4  7  9", header=TRUE)
dat
```

```
##   d xa xb ya yb
## 1 1  1  3  6  8
## 2 2  2  4  7  9
```
Reflect on your notes. Remember:

*Each set of variables are grouped together, under the respective names specified manually in the list*


```r
reshape(
 dat,
 idvar="d",
 varying=list(c("xa","xb"),c("ya","yb")),
 v.names=c("x","y"),
 direction="long",
 timevar="t",
 times=c("a","b")
)
```

```
##     d t x y
## 1.a 1 a 1 6
## 2.a 2 a 2 7
## 1.b 1 b 3 8
## 2.b 2 b 4 9
```
But dwelling under the surface, this is facilitated by a deeper consciousness.

*If a separating character is specified via `sep=` it is passed together with the names of the `varying=` columns to `strsplit()`.  E.g.:*


```r
x <- c("name_1","name_2")
sep <- "_"
strsplit(x, "_")
```

```
## [[1]]
## [1] "name" "1"   
## 
## [[2]]
## [1] "name" "2"
```

*`sep=""` can also be specified if there is no distinct character separating the names and times of variables. By default the `split=` in this case is a regular expression specifiying the point where letters end and where numbers begin. This will be easier to understand with a simple example:*


```r
x <- c("name1","name2")
## e.g.
##      12345 12345
##         *     * 
##      name1 name2
## all characters up to this point = names
## all character after this point = times
(r <- regexpr("[A-Za-z][0-9]", x))
```

```
## [1] 4 4
## attr(,"match.length")
## [1] 2 2
## attr(,"index.type")
## [1] "chars"
## attr(,"useBytes")
## [1] TRUE
```

```r
substr(x, 1, r)
```

```
## [1] "name" "name"
```

```r
substr(x, r + 1, nchar(x))
```

```
## [1] "1" "2"
```

Behold!


```r
names(dat)[-1]  <- paste0(c("x","x","y","y"),c(1,2,1,2))
reshape(
 dat,
 idvar="d",
 varying=-1,
 direction="long",
 split=list(regexp="[A-Za-z][0-9]", include=TRUE)
)
```

```
##     d time x y
## 1.1 1    1 1 6
## 2.1 2    1 2 7
## 1.2 1    2 3 8
## 2.2 2    2 4 9
```

But one should not get too comfortable. Patterns are not always repeated in the same way.


```r
names(dat)[-1]  <- paste0(c("x","x","y","y"),c("a","b","a","b"))
dat
```

```
##   d xa xb ya yb
## 1 1  1  3  6  8
## 2 2  2  4  7  9
```

```r
datl <- reshape(
 dat,
 idvar="d",
 varying=-1,
 direction="long",
 timevar="t",
 split=list(regexp="[^ab][ab]", include=TRUE)
)
datl
```

```
##     d t x y
## 1.a 1 a 1 6
## 2.a 2 a 2 7
## 1.b 1 b 3 8
## 2.b 2 b 4 9
```
Patterns may even be based on the position where they fall, nor the character of their body.

*You could even consider only the last character as the time/group, if the right regular expression is used.*


```r
datl <- reshape(
 dat,
 idvar="d",
 varying=-1,
 direction="long",
 timevar="t",
 split=list(regexp="..$", include=TRUE)
)
datl
```

```
##     d t x y
## 1.a 1 a 1 6
## 2.a 2 a 2 7
## 1.b 1 b 3 8
## 2.b 2 b 4 9
```

Sometimes reverting back to what you once were is easier...


```r
reshape(datl, direction="wide")
```

```
##     d xa ya xb yb
## 1.a 1  1  6  3  8
## 2.a 2  2  7  4  9
```

Sometimes a split may not even be apparent until multiple threads are pulled together.

*`strsplit=` based splitting can be used for complex examples when a separator is not consistent, but can be identified using a regex*


```r
names(dat)[-1]  <- paste0(c("x.","x.","y-","y-"),c("a","b","a","b"))
reshape(
 dat,
 idvar="d",
 varying=-1,
 direction="long",
 timevar="t",
 split=list(regexp="[.-]", include=FALSE)
)
```

```
##     d t x y
## 1.a 1 a 1 6
## 2.a 2 a 2 7
## 1.b 1 b 3 8
## 2.b 2 b 4 9
```













