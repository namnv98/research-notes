
docker exec -it --user root kafka bash

cd ~bin

kafka-topics --create  --if-not-exists --bootstrap-server kafka:9092  --replication-factor 1 --partitions 1  --topic my-topic

kafka-topics --list --bootstrap-server localhost:9092
