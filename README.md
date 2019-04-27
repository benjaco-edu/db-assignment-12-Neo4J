# db-assignment-12-Neo4J
https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment12.md
## Setup

> Just forget about importing you data your self, it takes a long time

```
sudo docker run  -d --name neo4j --rm --publish=7474:7474 --publish=7687:7687 --env NEO4J_AUTH=neo4j/fancy99Doorknob neo4j

sudo docker exec -it neo4j wget https://github.com/datsoftlyngby/soft2019spring-databases/raw/master/data/some2016UKgeotweets.csv.zip

sudo docker exec -it neo4j unzip some2016UKgeotweets.csv.zip

sudo docker exec -it neo4j mv some2016UKgeotweets.csv import/some2016UKgeotweets.csv
```

Open http://localhost:7474/browser/

Use credentials `neo4j` as username with password `fancy99Doorknob`

### Exercise 1

Write a cypher command that loads the tweets, creates a number of objects labeled "Tweet" with the columns "User Name", "Nickname","Place (as appears on Bio)", "Latitude", "Longitude" and "Tweet content" renamed as something nice (single lowercase name), and which adds a list of mentions to each Tweet.

```
USING PERIODIC COMMIT 500
LOAD CSV WITH HEADERS FROM "file:///some2016UKgeotweets.csv" AS row 
    FIELDTERMINATOR ";"
create (tweet:Tweet {
	username: row.`User Name`,
    nickname: row.Nickname,
    bioPlace: row.`Place (as appears on Bio)`,
    lat: toFloat(row.Latitude),
    lng: toFloat(row.Longitude),
    content: row.`Tweet content`,
    mentions: extract( m in 
                filter(m in split(row.`Tweet content`," ") where m starts with "@" and size(m) > 1) 
                | right(m,size(m)-1))
    })
```
Took 8 sec

### Exercise 2

Use the mentions list of each tweet to create a new set of nodes labeled "Tweeters", whith a "Mentions" relation.

```
MATCH (n:Tweet) 
unwind n.mentions as name
with distinct name
create (t:Tweeters{name: name})
```
Took < 1 sek

Create a relation "Tweeted" between Tweeters and Tweet.

```
match (tweeters:Tweeters), (tweet:Tweet)
where tweeters.name in tweet.mentions 
create (tweeters)-[:MENTIONED_IN]->(tweet)
```

It takes a long time `Created 40940 relationships, completed after 4416147 ms.`

### Exercise 3

Find the top 10 list of tweeters whose tweets are the furtherst apart.
```
MATCH (a:Tweet), (b:Tweet) 
where a.username = b.username and a <> b
with a, b, distance(point({longitude:a.lng, latitude:a.lat}), point({longitude:b.lng, latitude:b.lat}))/1000 as km
return distinct  a.nickname, max(km) as maxdist
order by maxdist desc
limit 25
```

Took 125 sek

Result:
```
a.nickname	maxdist
"googuns_lulz"	1287.6876134860725
"tmj_GBR_mgmt"	1101.2776468134407
"outonashout"	1030.581425638201
"MarsBots"	975.1571836022507
"x333xxx"	933.3654495465886
"TangoRaindrop"	893.9492533669937
"FoodTouristBlog"	893.9492533669937
"thursobhoy"	877.5142365743104
"Dazzathfc1882"	877.5142365743104
"ashleypratt"	876.6924197540286
```
