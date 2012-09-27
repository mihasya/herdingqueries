<!SLIDE title-slide>

# Herding Queries

### JOINs..

### Between Heterogeneous Datastores.. 

### Over the Network..

### What Could Possibly Go Wrong?

Mikhail Panchenko, Surge 2012

<img src="ua_logo.png" height="48" />

<!SLIDE bullets>

# HI!

My name is Pancakes.

<!SLIDE bullets>

## SimpleGeo Storage Platform Team

<img src="moar.png" height="272" class="shadow" />

## Urban Airship Messaging Team

.notes acquired for spatial expertese and amazing slide templates

<!SLIDE>

# Goal: Location-based Push

### Bonus: Complex Predicate-based Push

.notes querying by location only is not very compelling on its own

<!SLIDE>

# ( very nice, but very difficult things )

<!SLIDE>

<img src="segments.png" height="550" class="shadow" />

<!SLIDE bullets>

# Requirements
* 
    * Low time to first push
    * Constant, high throughput
    * Millions of devices
    * Horizontal scalability
    * All the things

<!SLIDE>

### "Perhaps you'd also like the key to the apartment

<img src="bender.jpg" height="277" class="shadow" />

### where the money is stashed?"

<small>image credit: <a href="http://rt.com/art-and-culture/news/bender-ostap-tears-moscow/">russia today</a>

<!SLIDE bullets>

.notes famous quote.. well, famous if you're russian

# Yes, and also..

### Platform-specific data and operations

*            
    * Push platform identifying tokens
    * "quiet time"
        * can't enforce at device level on all platforms
    * "Badge" updates, etc

### Per-device lookups + updates

.notes this is a big issue - every device being sent to has to be looked up by its ID - key value read

<!SLIDE bullets>

# Tag Predicates
*  
    * Tags set via the device SDK or directly by the developer (API)
    * Most likely: identify interest expressed by user
        * `team:blues`
        * `likes:beyonce`
        * `dislikes:thekillers`

<!SLIDE bullets>

# Tag Predicates
* 
    * Only supported disjunctive (OR) selection
        * Useful, but limiting
        * Workarounds: `likes:beyonce_dislikes:thekillers`
            * This does not scale
    * High cardinality
        * a device can have thousands of tags (**important**)

.notes another example: sports app - likes team, has these specific alerts turned on

<!SLIDE>

<img src="score.png" height="500" class="shadow" />

<!SLIDE bullets>
# Location Predicates
* 
    * Completely new concept, lots of possibilities
        * Last seen inside a polygon
            * City
            * Neighborhood
            * Stadium (seriously)
        * Has been in a polygon within an interval (historic)
    * **Recall**: SimpleGeo put a lot of effort into flexible querying of geodata.

<!SLIDE>

# CloudStock, 2010

<img src="sg-index-slide.png" height="450" class="shadow" />

<!SLIDE bullets>

# SimpleGeo Index Tech

*  
    * The index part of a DBMS, housed in a DHT
        * All the nice properties of the DHT
        * All the annoying properties of the DHT
    * Supports several types of trees
        * BPlusTree
        * KDtree (most used)
            * think BPlusTree with K dimensions
            * great for data with known dimensionality; flexible queries
        * RTree

<!SLIDE bullets>

# <span class="tag">Tags</span> and <span class="location">Location</span> Sitting in a Tree

*   
    * Presupposes boolean algebra
        * <span class="location">in SF</span> ^ <span class="tag">likes Dave Chappelle</span>
        * <span class="location">in SF</span> ^ <span class="tag">¬owns a car</span>
        * (<span class="location">in SF</span> ∨ <span class="location">in Oakland</span>) ^ <span class="tag">likes the Giants</span>

<!SLIDE>

# "An RDBMS will do all that!"

<img src="enthusiasm.png" height="450" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/carbonnyc/4318504691">carbonnyc</a></small>

.notes pro-tip: if you blanket set all your photos to Creative Commons license, some asshole will use them for presentations

<!SLIDE>

<img src="overloaded.jpg" height="500" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/belsymington/4102783610/">belsymington</a></small>

.notes not all RDBMSs will even do that, and the ones that do get really unwieldy at our data size

<!SLIDE>

<img src="overloaded_annotated.jpg" height="500" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/belsymington/4102783610/">belsymington</a></small>

.notes 100MM installs, trending towards 1B. Most customers &lt;= 10M, with a few notable, crucial exceptions

<!SLIDE>

# <span class="tag">Tag</span> solutions 

## don't incorporate spatial queries well

# <span class="location">Spatial</span> solution

## doesn't scale to arbitrary tag cardinality

<!SLIDE>

# Don't forget Platform-specific data!

<!SLIDE>

# How Do We Put It All Together?

<img src="puzzle.jpg" height="427" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/theotter/6590636397">theotter</a></small>

<!SLIDE>
<img src="wecandoit.jpg" height="500" class="shadow" />

<small>credit: <a href="http://en.wikipedia.org/wiki/We_Can_Do_It!">wikipedia</a></small>

.notes This appears to be a simple matter of programming

<!SLIDE>

# What Would a Database Do?

<img src="blegh.jpg" height="427" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/tehf0x/">tehf0x</a></small>

.notes I have to use a photo because I can never recreate that exact expression

<!SLIDE bullets>

# Vague Data Model

* 
    * Table for <span class="location">device location and history</span>
        * Primary Key: **`appId:deviceId`**
    * Table for <span class="tag">tag data</span>
        * probably just one table and lots of self joins
        * Primary Key: **`appId:deviceId`**
    * Table for <span class="platform">platform-specific data</span>
        * has to be joined to everything
        * Primary Key: **`appId:deviceId`**

<!SLIDE bullets>

# Aside: Data Clustering

## An Optimization

Most RDBMSs offer this in some form.

MySQL/InnoDB does it automatically for primary keys.

**Data is stored on disk in index-order**

**Queries ordered on this index scan sequentially**

This is important.

<!SLIDE bullets>

# The Life of a Query

* 
    * Parse
    * Plan
    * Perform
    * Respond

<!SLIDE bullets>

# The Life of a Query

* 
    * Parse - turn query into an logical tree
    * **Plan** - figure out cheapest logical equivalent 
    * **Perform** - fetch, sort, merge, etc.
    * Respond - send result back to client


<!SLIDE bullets>

# Example

<img src="operatortree.png" height="450" class="shadow" />

<small>source: [ 1 ]</small>

<!SLIDE bullets>

# Example

<img src="operatortree-annotated.png" height="450" class="shadow" />

<small>source: [ 1 ]</small>

<!SLIDE>

## **The Logic Only Cares About Tuples**

<!SLIDE>

# How Do I JOIN?

<img src="sortmergejoin.png" height="450" class="shadow" />

Basic Sort Merge Join Algo from [ 2 ]

<!SLIDE bullets>

# How Do I JOIN?

* 
    * Fetch
    * Sort
    * Merge
    * Return Matches

<!SLIDE bullets>

# How Do I JOIN?

* 
    * Fetch
    * **Sort** - recall clustered keys
    * Merge
    * Return Matches

<!SLIDE>

# Phrasing is Important

<img src="shoppingcart.jpg" height="400" class="shadow" />

### What Do We REALLY Need?

<small>photo by <a href="http://www.flickr.com/photos/jessalyn/7333566838/">jessalyn</a></small>

.notes I'm never gonna eat all those pineapples

<!SLIDE>

# "Query arbitrary combinations of spatial and tag data in real time."

.notes makes it seem like we'd have to try and implement the entire RDMBs query stack

<!SLIDE>

# "Send to devices<br /><br />matching an arbitrarily nested predicate<br /><br /> in any order." 

.notes those are actually quite different

<!SLIDE>

### Ordering Doesn't Matter, So Pick One

# Order by Application ID and Device ID

.notes force global ordering on disk, in queries. clustering can be taken for granted &amp; we get to skip the "sort" in sort-merge-join

<!SLIDE bullets>

# So What Do We End Up With?

* 
    * <span class="location">Location</span> index returns results in ID order
    * <span class="tag">Tag</span> index stored in ID order
    * <span class="platform">Platform</span> data stored in ID order

### Any Ordered Partitioner For <span class="tag">Tags</span> and <span class="platform">Platform</span> Data

### <span class="location">Location</span> Index Not Clustered, But Could Be

<!SLIDE>

# Only Sequential Reads<br/>for <span class="tag">Tag</span> and <span class="platform">Platform</span> indexes

### Huge Performance Win,<br />Even With SSDs

<!SLIDE>

# All indexes support efficient cursor pagination &amp; can skip forward

### Optimizes JOINing Sets Of Wildly Disparate Sizes

<!SLIDE>

# Only Return The Exact Set Requested

<!SLIDE>

# How Do I JOIN?

<img src="sortmergejoin-reduced.png" height="450" class="shadow" />

Basic Sort Merge Join Algo from [ 2 ]

<!SLIDE>

# One More Time, With Feeling

<img src="mergejoinwithfeeling.png" height="450" class="shadow" />

Datastores Connected By The Query Execution Service

.notes putting a service in production called "gooeybuttercake" is my crowning achievement

<!SLIDE>

# Mmmmmmm Gooey Butter Cake

<img src="gbc.jpg" height="400" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/montage_man/4152261542">montage man</a></small>


