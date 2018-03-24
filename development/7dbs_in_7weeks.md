## 1. Postgres
### Fast Lookups with Indexing
PostgreSQL automatically creates an index on the primary key, where the
key is the primary key value and where the value points to a row on disk.

You can explicitly add a hash index using the CREATE INDEX command, where
each value must be unique (like a hashtable or a map).
```
CREATE INDEX events_title
ON events USING hash (title);
```
For range queries you need a more flexible index, like B-tree.
```
CREATE INDEX events_starts
ON events USING btree (starts);
```
Now our query over a range will avoid a full table scan.

When you set a FOREIGN KEY constraint, PostgreSQL will automatically create an index on the targeted column(s).

### Day 2: Advanced Queries, Code, and Rules
#### Aggregate Functions
An aggregate query groups results from several rows by some common criteria.
#### Grouping
Place the rows into groups and then perform some aggregate function (such as count().
`HAVING` is like the WHERE clause, except it can filter by aggregate functions.
```
SELECT venue_id
FROM events
GROUP BY venue_id
HAVING count(*) >= 2 AND venue_id IS NOT NULL;
```
`DISTINCT` == `GROUP BY` without any aggregate function.

#### Window Functions
allow you to use built-in aggregate functions without requiring every single field to be grouped to a single row.
```
SELECT title, count(*) OVER (PARTITION BY venue_id) FROM events;
```
It returns grouped values as any other field (calculating on
the grouped variable but otherwise just another attribute).
It returns the results of an aggregate function OVER a PARTITION of the result set.

#### Transactions
Transactions ensure that every command of a set is executed: all or nothing.
PostgreSQL transactions follow ACID compliance.

#### Stored Procedures
Sometimes we need to run some code -- you need to decide whether you'll do it on client side or on the db side.
Stored procedures can provide huge performance:
```
CREATE OR REPLACE FUNCTION add_event( title text, starts timestamp,
ends timestamp, venue text, postal varchar(9), country char(2) )
RETURNS boolean AS $$
DECLARE
did_insert boolean := false;

SELECT add_event('House Party', '2012-05-03 23:00', '2012-05-04 02:00', 'Run''s House', '97205', 'us');
```
The procedure is written in PL/pgSQL.
Postgres supports three more core languages for writing procedures: Tcl, Perl, and Python. There are extensions for more of them, e.g. Ruby, Java, PHP.

#### Pull the Triggers
```
CREATE OR REPLACE FUNCTION log_event() RETURNS trigger AS $$
DECLARE
BEGIN
INSERT INTO logs (event_id, old_title, old_starts, old_ends)
VALUES (OLD.event_id, OLD.title, OLD.starts, OLD.ends);
RAISE NOTICE 'Someone just changed event #%', OLD.event_id;
RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_events
AFTER UPDATE ON events
FOR EACH ROW EXECUTE PROCEDURE log_event();
```
#### Viewing the World
Using the results of a complex query just like any other table?
```
CREATE VIEW holidays AS
SELECT event_id AS holiday_id, title AS name, starts AS date
FROM events
WHERE title LIKE '%Day%' AND venue_id IS NULL;
```
Views are powerful tools for opening up complex queried data in a simple way.
You cannot update a view directly.

#### What RULEs the School?
A RULE is a description of how to alter the parsed query tree.
- Every time Postgres runs an SQL statement, it parses the statement into a query tree (abstract syntax tree).
- Operators and values become branches and leaves in the tree, and the tree
is walked, pruned, and in other ways edited before execution.
- This tree is optionally rewritten by Postgres rules.
- Then it's before being sent on to the query planner (which also rewrites the tree in a way to run optimally).
- The final command is executed.

To allow updates against our holidays view, we need to craft a RULE that tells Postgres what to do with an UPDATE.

```
CREATE RULE update_holidays AS ON UPDATE TO holidays DO INSTEAD
UPDATE events
SET title = NEW.name,
starts = NEW.date,
colors = NEW.colors
WHERE title = OLD.name;
```

#### Pivot tables, crosstab()
To use crosstab() , the query must return three columns: rowid , category , and value.
rowid - identifier of the record
category - the categories in the pivot table (headers)
value - values of the cells

### Day 3: Full-Text and Multidimensions
#### Fuzzy Searching
##### SQL Standard String Matches
LIKE, ILIKE -- case sensitive/insensitive exact search
_ - any single character
% - any nunber of characters
`SELECT title FROM movies WHERE title ILIKE 'stardust_%';`

##### Regex
```
SELECT COUNT(*) FROM movies WHERE title !~* '^the.*';
```
Index for pattern matching:
```
CREATE INDEX movies_title_pattern ON movies (lower(title) text_pattern_ops);
```
If you need to index varchars, chars, or names, use the related ops: varchar_pattern_ops , bpchar_pattern_ops , and name_pattern_ops .

##### Levenshtein
Levenshtein is a string comparison algorithm that compares how similar two
strings are by how many steps are required to change one string into another.
Each replaced, missing, or added character counts as a step.

##### Trigram
-- a group of three consecutive characters taken from a string.
```
SELECT show_trgm('Avatar');
```
Index:
```
CREATE INDEX movies_title_trigram ON movies
USING gist (title gist_trgm_ops);
```

#### Full-Text Fun
##### TSVector and TSQuery
```
SELECT title
FROM movies
WHERE title @@ 'night & day';
// equal to
SELECT title
FROM movies
WHERE to_tsvector(title) @@ to_tsquery('english', 'night &amp; day');
```

A tsvector is a datatype that splits a string into an array (or a vector) of tokens, which are searched against the given query.
The tsquery represents a query in some language, like English or French.

Common words like a are called stop words and are generally not useful for
performing queries.
Stop words are stored:
`cat `pg_config --sharedir`/tsearch_data/english.stop`

##### Other Languages
Find the English stem word of the string Day’s:
```
SELECT ts_lexize('english_stem', 'Day''s');
```

##### Indexing Lexemes
```
CREATE INDEX movies_title_searchable ON movies
USING gin(to_tsvector('english', title));
// you need to specify language to use the index
SELECT *
FROM movies
WHERE to_tsvector('english',title) @@ 'night & day';
```
##### Metaphones
`dmetaphone` , `dmetaphone_alt`, `metaphone`, `soundex`

##### Combining String Matches
You can combine metaphones with other searches, but the result won't be perfect.

##### Genres as a Multidimensional Hypercube
We’ll use the cube datatype to map a movie’s genres as a multidimensional vector.

E.g. a genre vector of (0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)
Each value describes the position on one of the axes.

You can find similar movies by genre:
```
SELECT *,
cube_distance(genre, '(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)') dist
FROM movies
ORDER BY dist;
```
Make queries faster:
We use cube_enlarge(cube,radius,dimensions) to build an 18-dimensional cube that is some length (radius) wider than a point.

With our bounding hypercube, we can use a special cube operator, @> , which means contains.

```
SELECT title, cube_distance(genre, '(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)') dist
FROM movies
WHERE cube_enlarge('(0,7,0,0,0,0,0,0,0,7,0,0,0,0,10,0,0,0)'::cube, 5, 18) @> genre
ORDER BY dist;
```

### PostgreSQL’s Strengths
PostgreSQL is fantastic for the data that conforms some schema.
Open source, many features.
ACID-compliant transactions.

### PostgreSQL’s Weaknesses
No/hard(?) partitioning.
If your data requirements are too flexible, or you need very high-volume reads and writes as key values, pick smth else.

## 3. Riak
### MapReduce
Mapreduce breaks down problems into two parts. Part 1 is to convert a list
of data into another type of list by way of a map() function. Part 2 is to convert this second list to one or more scalar values by way of a reduce() function.

## 5. MongoDB

Document db, allows data to persist in a nested state, query nested data.
Enforces no schema => document can contain fields that no other documents of the collection contains.

Mongo's collection ~ Postgres table

### Humongous
Sweet spot between the querability of the relational db and distributed nature of Riak or HBase.

Data is stored in a binary form of JSON (BSON).
Like a relational db w/0 schema -- documents of any depth.

## 5.2 CRUD and Nesting
Unlike a relational database, Mongo does not support server-side joins.
ObjectId is like SERIAL in pg.
The ObjectId is always 12 bytes, composed of a timestamp, client machine ID, client process ID, and a 3-byte incremented counter.

Each mongod process can handle its own ID generation w/o colliding with other processes.

#### Javascript
MongoDB native tongue is js.
`db` `db.x` are just js objects.
Commands are just js functions.
You can create your own js functions.

#### Reading
`find()` ==> all documents
`find()` accepts ObjectId and optional fields
You can construct queries by field values, criterias, regexps, etc.
Conditional operators: `field: { $op: value }`
$op is an operation like $gt
It's js, not dsl.

You can query nested data by exact values or by matching partial values or lack of matching values.

You can query by the criterias of the deep nested data.

#### elemMatch
Specifies if a document matches all the specified criteria, the document counts as a match.

#### Boolean Ops
`$or` and many others boolean operators.

#### Updating
first argument -- query
second -- an object which fields will replace the existing or a modifier
`$set` -- update/add specified fields
`$unset`
`$inc`
and others

#### References
Isn't built to perform joins. they're pretty insufficient.
If needed, use a construct like `{ $ref : "collection_name", $id : "reference_id" } .``

Retreive -- `db.countries.findOne({ _id: portland.country.$id })`

#### Deleting
Replace find to `remove`
Run `find()` to verify before `remove()`

#### Reading with code
You can write js code to retrieve the data, use as a last resort.
Slow queries, will run against all the docs.

#### Sumary
Denormalized nature:
Choice for storing data of unknown structure.

### 5.3. Indexing, Grouping, MapReduce
#### Indexing: When Fast Isn’t Fast Enough
MongoDB provides several of the best data structures for indexing:
the classic B-tree,  two-dimensional and spherical GeoSpatial indexes.

Whenever a new collection is created, Mongo automatically creates an index by the _id.
db indexes -- `db.system.indexes.find()`

Create an index:
```
db.phones.ensureIndex(
{ display : 1 },
{ unique : true, dropDups : true }
)
```
With index Mongo is no longer doing a full collection scan but instead walking the tree to retrieve the value.
Mongo can build your index on nested values.
```
db.phones.ensureIndex({ "components.area": 1 }, { background : 1 })
```
Creating an index on a large collection can be slow and resource-intensive.

#### Aggregated Queries
Aggregated queries return a structure other than the individual documents we’re
used to -- count, distinct, group.

The `group()` aggregate query is akin to `GROUP BY` in SQL.
Our data is already mapped into our existing collection of documents. No more mapping is necessary; simply reduce the documents.

```js
db.phones.group({
    initial: { count:0 },
    reduce: function(phone, output) { output.count++; },
    cond: { 'components.number': { $gt : 5599999 } },
    key: { 'components.area' : true }
})
```

With mongo's `group()` you are limited to a result of 10,000 documents.
If you shard your Mongo collection `group()` won’t work.
`finalize()` ==> reformat the results.

#### Server-Side Commands
`eval()` function passes the given function to the server.
This dramatically reduces chatter between the client and server since the code is executed remotely.
`db.eval(update_area)`

Some pre-built commands are also executed on the server, some require executing only under the admin database.
```
use admin
db.runCommand('top')
```
##### runCommand
a helper function that wraps a call to a collection named $cmd . You can execute any command using a call directly to this collection.
`db.$cmd.findOne({'count' : 'phones'})`

##### Diversion
- the most of the magic you execute on the mongo console is executed on the server, not the client, which just provides convenient wrapper functions.
- we can create functions in MongoDB and "send" them to the server via `eval`, that’s similar to the stored procedures in PostgreSQL

Any JavaScript function can be stored in a special collection named system.js .
```
db.system.js.save({
_id:'getLast',
value:function(collection){
return collection.find({}).sort({'_id':1}).limit(1)[0]
}
})
```
Execute it on the server directly:
`db.eval('getLast(db.phones)')`
Same:
`db.system.js.findOne({'_id': 'getLast'}).value(db.phones)`

#### Mapreduce (and Finalize)
MapReduce is used when more power is required.

`map()` -- calls an `emit()` function with a key, you can emit more than once per document.
The `reduce()` function accepts a single key and a list of values that were emitted to that key.
`finalize()` -- optional third step, is executed only once per mapped value after the reducers are run.

Example: counting documents by country.
Mapping: decide what fields to map: digits (what to count), country (by what)
Emitting: e.g. emit value 1, which will represent 1 document to count
Reducer: sum all those 1s together.

```
results = db.runCommand({
    mapReduce: 'phones',
    map: map,
    reduce: reduce,
    out: 'phones.report'
})
```
'phones.report' will be a materialized view which you can query by any other.
You can set the out value to { inline : 1 }, the mapreducer will just output the results, but the limit will be 16mb.

The reducers can have either mapped (emitted) results or other reducer results as inputs.

You might need to perform some final changes, such as rename a field or some other calculations. We can implement a `finalize()` function, which works the same way as the finalize function under `group()`.

### Day 3: Replica Sets, Sharding, GeoSpatial, and GridFS
What makes document databases unique is their ability to efficiently handle arbitrarily nested, schemaless data documents.
Mongo is special for its ability to scale across several servers:
- replicating (copying data to other servers)
- sharding collections (splitting a collection into pieces) 
- performing queries in parallel.

#### Replica Sets
`rs` -- (replica set)
There is only one master per replica set, and you must send all writes to it.
Problems: deciding who gets promoted when a master node goes down.

The Mongo philosophy of server setups and the reason we should always have an odd number of servers -- to reach consensus.

In case of master failure, after the failover and original master's recovery, if the original master had data that did not yet propagate, the operations are dropped.
A write in a Mongo replica set isn’t considered successful until most nodes have a copy of the data.

#### The Problem with Even Nodes
Mongo dictates that a majority of nodes that can still communicate make up the network.
E.g. 2 of 5 nodes are down or the network is unreachable, but 3 may communicate.

Some databases (e.g., CouchDB) are built to allow multiple masters, but
Mongo is not, and so it isn’t prepared to resolve data updates between them.

##### Voting and Arbiters
You may not always want to have an odd number of servers replicating data.
You may then have an arbiter, a voting but nonreplicating server in the replica set.

#### Sharding
Mongo quickly handles very large datasets by horizontal sharding by value ranges, just sharding for brevity.

Some range of values are split (sharded) onto other servers.
`mongod --shardsvr --dbpath ./mongo4 --port 27014`  

The `mongos` server will connect to the `mongoconfig` config server to keep track of the sharding information stored there.

##### Mongos vs Mongoconfig
Mongo separates configuration and the mongos point of entry into two different servers, in production environments they will generally live on different physical servers.

The `config server` (itself replicated) manages the sharded information for other sharded servers, and `mongos` will likely live on your local application server where clients can easily connect

`mongos` is a lightweight clone of a full `mongod` server.
Nearly any command you can throw at a mongod , you can throw at a mongos.
It goes between clients and sharded servers.

#### GeoSpatial Queries
It’s a special form of indexing geographic data called geohash that not only finds values of a specific value or range quickly but finds nearby values quickly in ad hoc queries.

`db.cities.find({ location : { $near : [45.52, -122.67] } }).limit(5)`

Works with non-sharded collection?

Mongo has support for bounding shapes (namely, squares and circles). E.g. you can find cities around some location.

##### GridFS
Provides its own distributed filesystem, e.g. to store users' uploads without having to copy files to every replica.
No need to install anything.
```
mongofiles -h localhost:27020 list
mongofiles -h localhost:27020 put my_file.txt
```
Files are just plain old collections, they can be replicated or queried like
any other.

##### Mongo’s Strengths
- ability to handle huge amounts of data (and huge amounts of requests) by replication and horizontal scaling
- easy to use, similar to relational
- recommend it to Rails, Django, and Model-View-Controller (MVC) developers, validations in the app

##### Mongo’s Weaknesses
- denormalization of schemas might be too much, a single typo can cause hours
of headache
- large clusters require some effort to design and manage, adding nodes is harder then for Riak

## Redis
Reads are fast, and writes are even faster, handling upwards of 100,000 SET operations per second by some benchmarks.
### Data Structure Server Store
A key-value store that supports advanced data structures, but not to the degree that a document-oriented database would.
Trades durability for raw speed.
Redis is a blocking queue (or stack) and a publish-subscribe system
Redis is a toolkit of useful data structure algorithms and processes.

### Day 1: CRUD and Datatypes
```
redis-cli
```
```
SET 7wks http://www.sevenweeks.org/
GET 7wks
// multiple set
MSET gog http://www.google.com yah http://www.yahoo.com
```
Although Redis stores strings, it recognizes integers and provides some simple
operations for them.
```
SET count 2
INCT count
```
#### Transactions
```
MULTI
SET prag http://pragprog.com
INCR count
EXEC
```
Similar to ROLLBACK in SQL, you can stop a transaction with the DISCARD command, which will clear the transaction queue

#### Complex Datatypes
Collection datatypes can contain a huge number of values (up to 2^32 elements or more than 4 billion) per key.

- SET commands begin with S,
- hashes with H 
- sorted sets with Z
- List commands generally start with either an L (for left) or an R (for right), depending on the direction of the operation (such as LPUSH ).

#### Hash
```
HMSET user:eric name "Eric Redmond" password s3cret
# get data
HVALS user:eric
1) "Eric Redmond"
2) "s3cret"
# get all the hash keys
HKEYS user:eric
1) "name"
2) "password"
# get single value
HGET user:eric password
```
Hashes in Redis cannot nest (nor can any other complex datatype such as lists).
Can store only string values.
`HDEL` - delete hash fields.
`HINCRBY` - increment an integer value by some count
`HLEN` - retrieve the number of fields in hash

#### List
Lists contain multiple ordered values that can act both as queues (FIFO), stacks (LIFO).

Set the key to USERNAME:wishlist and push any number of values to the right (end) of the list
```
RPUSH eric:wishlist 7wks gog prag
// get length
LLEN eric:wishlist
// get values
LRANGE eric:wishlist 0 -1
```
`LREM` -- removes from the given key some matching values.
0 -- remove all matches, 1 -- remove 1 match
```
LREM eric:wishlist 0 gog
```

Remove and retrieve each value in the order we added them (like a queue): `RPUSH` + `LPOP` or `LPUSH` + `RPOP`.
Left -- head.
```
LPOP eric:wishlist
```
Stack: `RPUSH` + `RPOP` or `LPUSH` + `LPOP`

You can use a transaction for these commands or use a single command:
```
RPOPLPUSH eric:wishlist eric:visited
```
But `RPOPRPUSH`, `LPOPLPUSH` , and `LPOPRPUSH` don't exist.

#### Blocking Lists
A simple messaging system where multiple clients can push comments and one client (the digester) pops messages from the queue:
`BPROP` -- block until a value exists to pop.
It requires the key to pop a value from and a timeout in seconds.
```
BRPOP comments 300
// in another console
LPUSH comments "Prag is great! I buy all my books there.
```
In the digester console two lines will be returned: the key and the popped value + length of the time blocking.

#### Set
Sets are unordered collections with no duplicate values. Perfect for operations like unions or intersections.
```
SADD news nytimes.com pragprog.com
SADD tech pragprog.com apple.com
// get data
SMEMBERS news
// find intersection
SINTER news tech
// news - tech
SDIFF news tech
// union
SUNION news tech
// store the union under a different name
SUNIONSTORE websites news tech
// copy
SUNIONSTORE news_copy news
```
`SINTERSTORE` - store intersections
`SDIFFSTORE` - store diffs
`SMOVE` - moved values from one list to another (like `RPOPLPUSH`)
`SCARD` (set cardinality) counts the set;

Sets are not ordered, there are no left, right, or other positional commands.
`SPOP key` -- pop (get and remove) a random value
`SREM key value [value]` -- remove values

#### Sorted Sets
Ordered like lists and are unique like sets.
You can think of sorted sets as like a random access priority queue.
Internally, sorted sets keep values in order, so inserts can take log(N) time to insert rather than the constan complexity of hashes and lists.

Like a hash, adding a value to a sorted set requires two values after the Redis key name: the score and the member.
```
ZADD visits 500 7wks 9 gog 9999 prag
ZINCRBY visits 1 prag
// view data
ZSCORE visits prag
```

#### Ranges
```
ZRANGE visits 0 -1
// get the scores of each element
ZREVRANGE visits 0 -1 WITHSCORES
// 9 < score <= 10,000
ZRANGEBYSCORE visits 9 9999
1) "7wks"
// list all by score (infinity)
ZRANGEBYSCORE visits -inf inf
ZRANGEBYSCORE visits -inf inf WITHSCORES
```
#### Unions
Just like the set datatype, we can create a destination key that contains the
union or intersection of one or more keys.
```
ZUNIONSTORE destination numkeys key [key ...]
[WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
```
destination - the key to store info
numkeys - the number of keys to union
key - one or more keys to union
weight - opt. number to multiply each score
aggregate - opt. rule for resolving (min, max, sum)

Figuring out the most important websites in our system, as some
combination of votes and visits, votes are important to a less extent.
```
ZUNIONSTORE importance 2 visits votes WEIGHTS 1 2 AGGREGATE SUM
// check data
ZRANGEBYSCORE importance -inf inf WITHSCORES
```
Using to multiply values of votes (rewriting itself):
```
ZUNIONSTORE votes 1 votes WEIGHTS 2
// check
ZRANGE votes 0 -1 WITHSCORES
```

#### Expiry
A common use case for a key-value system like Redis is as a fast-access cache for data expensive to retrieve or compute.
`EXPIRE` -- mark a key for expiration

Expire ice in 10 seconds.
```
EXPIRE ice 10
// check if exists
EXISTS ice
```
SET + EXPIRE
```
SETEX ice 10 "I'm melting..."
// check the time
TTL ice
// remove the timeout
PERSIST ice
```
`EXPIREAT` - absolute time for expiring

#### Database Namespaces
`SELECT 1` -- switch to another db

Redis lets us shuffle keys between the databases with the MOVE command.
`MOVE greeting 2` => move value to db 2

#### And There’s More
`RENAME` `TYPE` `DEL` `FLUSHDB` `FLUSHALL`

### Day 2: Advanced Usage, Distribution
#### A Simple Interface
We can interact without the command-line interface by streaming commands through TCP on our own via telnet

```
telnet localhost 6379
SET test hello
GET test
```
Terminate with `CRLF` or `\r\n`

#### Pipelining
```
(echo -e n "ECHO hello\r\n"; sleep 1) | nc localhost 6379
```
#### publish-subscribe
```
SUBSCRIBE comments

another CLI
PUBLISH comments "Check out this shortcoded site! 7wks"

# received
1) "message"
2) "comments"
3) "Check out this shortcoded site! 7wks"
```
#### Server Info
```
INFO
```
#### Configuration
Default:
```
daemonize no
port 6379
loglevel verbose
logfile stdout
database 16 # dbs count
```
#### Durability
Persistence options:
- no persistense
- occasionally saving to disk

`LASTSAVE` -- get unix timestamp of the last save
`SAVE` -- force saving to disk
`BGSAVE` -- async save

#### Snapshotting
Configure saving:
Defaults:
```
save 900 1
save 300 10
save 60 10000
```
if 10,000 keys change, save in 60 seconds and so on.

#### Append-Only File
Redis provides an append-only file ( appendonly.aof ) that keeps a record of all write commands.

`appendonly yes` must be set in `redis.conf`

```
# appendfsync always -- slow!
appendfsync everysec -- okayish
# appendfsync no
```
#### Security
requirepass setting and AUTH command with plain password
Redis provides command-level security through obscurity,
```
rename-command FLUSHALL c283d93ac9528f986023793b411e4ba2
redis 127.0.0.1:6379> FLUSHALL
(error) ERR unknown command 'FLUSHALL'
redis 127.0.0.1:6379> c283d93ac9528f986023793b411e4ba2
OK
```
Disabling a command:
```
rename-command FLUSHALL ""
```
#### Tweaking Parameters
Benchmarking after tweaking:
```
redis-benchmark -n 100000
```
#### Master-Slave Replication
Example of slave config:
```
port 6380
slaveof 127.0.0.1 6379
```

In production, you’ll generally want to implement replication for availability
or backup purposes and thus have Redis slaves on different machines.

#### Data Dump
1) The simplest method is to simply iterate through a list of data and execute
SET for each value using the standard redis-rb client.

2) Speeding up -- using `hiredis` gem.

#### Redis Cluster
The Ruby client redis-rb supports a consistent-hashing managed cluster.
Unlike the master-slave setup, both of our servers will take the master (default) configuration.

```ruby
require 'redis/distributed'

$redis = Redis::Distributed.new([
 "redis://localhost:6379/",
 "redis://localhost:6380/"
])
```
But a lot more work is being done by the client, since it handles computing
which keys are stored on which servers.
Only one client will GET a value. But as long as you
retrieve keys set through the same Redis::Distributed configuration, the client
will access the values from the correct servers.

#### Bloom Filters
The upside with this approach is the ability to detect duplicate words. The
downside is that a few false positives will seep through—the Bloom filter may
flag a word we have never seen before.
In a real-world use case you would perform some secondary check, such as a slower database query to a system of record, which should happen only a small percentage of the time.

#### SETBIT and GETBIT
Sets/gets a single bit at a certain location in a bit sequence, starting from zero.
Typical usage for high performance systems.

The Bloom filter implementation takes advantage of this behavior by hashing
a value as a multibit value.









































































































