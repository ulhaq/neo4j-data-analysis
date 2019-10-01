# neo4j
We can load the csv data and add a list of mentions to each Tweet using the following Cypher Query:
```javascript
LOAD CSV WITH HEADERS FROM "file:///some2016UKgeotweets.csv" AS row
    FIELDTERMINATOR ";"
WITH [m in [m in split(row["Tweet content"]," ") where m starts with "@" and size(m) > 1] | right(m,size(m)-1)] as mentioned, row
CREATE (:Tweet { username: row["User Name"], nickname: row["Nickname"], place: row["Place (as appears on Bio)"], latitude: row["Latitude"], longitude: row["Longitude"], content: row["Tweet content"], mentions: mentioned })
```
> Added 136099 labels, created 136099 nodes, set 952693 properties, completed after 2875 ms.

The following query creates a new set of nodes labeled `Tweeters` whith a `MENTIONS` relation.
```javascript
MATCH (tweet:Tweet)
UNWIND tweet.mentions as mentions
CREATE (tweeter:Tweeters { username: mentions })
CREATE (tweet)-[r:MENTIONS]->(tweeter)
```
> Added 41036 labels, created 41036 nodes, set 41036 properties, created 41036 relationships, completed after 641 ms.

The following query creates a relation `Tweeted` between `Tweeters` and `Tweet`:
```javascript
MATCH (tweeter:Tweeters)
WITH tweeter
MATCH (tweet:Tweet {username: tweeter.username})
CREATE (tweeter)-[r:TWEETED]->(tweet)
```
> Created 162 relationships, completed after 553 ms.

The following query lists the top ten tweeters whose tweets are the furthest apart:
```javascript
MATCH (tweeter:Tweeters)-[:TWEETED]->(tweet:Tweet)
WITH point({ latitude:toFloat('12.511951'), longitude:toFloat('55.770213')}) AS cph_geo_cord, point({ latitude:toFloat(tweet.latitude), longitude:toFloat(tweet.longitude)}) AS tweet_geo_cord, tweeter
RETURN DISTINCT tweeter.username, toInteger(distance(cph_geo_cord, tweet_geo_cord)/1000) AS km ORDER BY km desc LIMIT 10
```
|tweeter.username| km |
|----------------|----|
|"Lynsay"        |7069|
|"GoldenPlec"    |7062|
|"holly"         |7059|
|"Zara"          |7050|
|"scattaranks"   |6866|
|"huytonbad"     |6855|
|"Winckleys"     |6850|
|"huytonbad"     |6847|
|"HugoandOtto"   |6832|
|"Zara"          |6830|
|Started streaming 10 records after 126 ms and completed after 126 ms.| |