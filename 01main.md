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

<!SLIDE>

# Goal:

## Location-based Push

# Bonus:

## Arbitrarily Complex Push

<!SLIDE>

# ( things nobody else has )

<!SLIDE>

<img src="segments.png" height="550" class="shadow" />

<!SLIDE bullets>

# Requirements
* 
    * Instant
    * Constant, high throughput
    * Millions of devices
    * Horizontal scalability
    * All the things

<!SLIDE>

### "Perhaps you'd also like the key to the apartment

<img src="bender.jpg" height="277" class="shadow" />

### where the money is stashed?"

<small>credit: <a href="http://rt.com/art-and-culture/news/bender-ostap-tears-moscow/">russia today</a>

<!SLIDE bullets>

# Yes, and also..

*    
    * Platform-specific data and operations
        * badge updates
        * "quiet time"
            * can't enforce at device level on all platforms
        * delivery service device ID resolution
            * key-value lookup at delivery time

<!SLIDE bullets>

# Tag Data
*  
    * Set via the API through the device SDK or directly by the developer
    * Most likely: identify interest expressed by user
        * `team:blues`
        * `likes:beyonce`
        * `dislikes:thekillers`
    * Only supported disjunctive (OR) selection
        * Useful, but limiting
        * Workarounds: `likes:beyonce_dislikes:thekillers`
            * This does not scale
    * High cardinality
        * a device can have thousands of tags (**important**)

<!SLIDE bullets>
# Location Data
* 
    * Completely new concept, lots of possibilities
        * Within N feet of Lat,Lon
        * Inside a polygon
            * City
            * Neighborhood
            * Stadium (seriously)
        * Has been in Baltimore in the last 2 weeks
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

## "An RDBMS will do all that!"

<!SLIDE>

<img src="overloaded.jpg" height="500" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/belsymington/4102783610/">belsymington</a></small>

<!SLIDE>
.notes not all RDBMSs will even do that, and the ones that do get really unwieldy at our data size

<img src="overloaded_annotated.jpg" height="500" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/belsymington/4102783610/">belsymington</a></small>

<!SLIDE>

# <span class="tag">Tag</span> solutions 

## don't incorporate spatial queries well

# <span class="location">Spatial</span> solution

## doesn't scale to arbitrary tag cardinality

<!SLIDE>

# Don't forget the Platform-specific data!

<!SLIDE>

graphic representing the gap?

<!SLIDE>

# How Do We Bridge The Gap?

<!SLIDE>
<img src="wecandoit.jpg" height="500" class="shadow" />

<small>credit: <a href="http://en.wikipedia.org/wiki/We_Can_Do_It!">wikipedia</a></small>

<!SLIDE bullets>

# Let's Take Stock

*    
    * SOA - small services, good at one or two things
        * Databases suited for each use case
    * Shared RPC library
    * 100MM installs, trending towards 1B
        * &lt;10MM per customer with a few exceptions

<!SLIDE>

# Take 1

Customer -> API -> Fetch Data -> Munge -> Devices

<!SLIDE bullets>

# Scan ALL The Things

* 
    * Effectively Map-Reduce
    * An obviously non-optimal solution
    * Difficult to parallelize
    * Crippling at low cardinality

<!SLIDE>

# Take 2

Customer -> API -> Fetch Data & Munge<sub>1</sub> -> .. -> Fetch Data & Munge<sub>N</sub> -> Devices

<!SLIDE>

<img src="dog-tail.jpg" height="500" class="shadow" />

<small>photo by <a href="http://www.flickr.com/photos/ick9s/3572358617/in/photostream/">ick9s</a></small>

<!SLIDE bullets>

# Send ALL The Things

*   
    * Each step implements a 2-way "push" protocol
        * Homogeneous - lists go in, lists come out
