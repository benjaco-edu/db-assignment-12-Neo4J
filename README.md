# db-assignment-12-Neo4J

## Setup

```
sudo docker run  -d --name neo4j --rm --publish=7474:7474 --publish=7687:7687 --env NEO4J_AUTH=neo4j/fancy99Doorknob neo4j

sudo docker exec -it neo4j wget https://github.com/datsoftlyngby/soft2019spring-databases/raw/master/data/some2016UKgeotweets.csv.zip

sudo docker exec -it neo4j unzip some2016UKgeotweets.csv.zip

sudo docker exec -it neo4j mv some2016UKgeotweets.csv import/some2016UKgeotweets.csv
```

Open http://localhost:7474/browser/

Use credentials `neo4j` as username with password `fancy99Doorknob`

