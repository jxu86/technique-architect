


```
docker run --name my_mongo -v /home/ec2-user/data/mongo:/data/db --rm -d -p 26116:27017 mongo:latest

docker exec -it my_mongo mongosh
db.createUser({ user:'admin',pwd:"MetaFu20230112!!”,roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});

db.createUser({
  user: “met”,
  pwd: "<password>",
  roles: [{ role: "read", db: "<database>" }]
})


docker exec my_mongo sh -c 'mongodump --archive --db smugnft' > ./smugnft.gz
docker exec -i my_mongo sh -c 'mongorestore --archive --db smugnft' <./smugnft.gz
```