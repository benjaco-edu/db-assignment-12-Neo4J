# db-assignment-12-Neo4J
https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment12.md
## Setup

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
    lat: toInteger(row.Latitude),
    lng: toInteger(row.Longitude),
    content: row.`Tweet content`,
    mentions: extract( m in 
                filter(m in split(row.`Tweet content`," ") where m starts with "@" and size(m) > 1) 
                | right(m,size(m)-1))
    })
```

### Exercise 2

Use the mentions list of each tweet to create a new set of nodes labeled "Tweeters", whith a "Mentions" relation.

```
MATCH (n:Tweet) 
unwind n.mentions as name
with distinct name
create (t:Tweeters{name: name})
```

Create a relation "Tweeted" between Tweeters and Tweet.

```
match (tweeters:Tweeters), (tweet:Tweet)
where tweeters.name in tweet.mentions 
create (tweeters)-[:MENTIONED_IN]->(tweet)
```

