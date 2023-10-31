export DEBEZIUM_VERSION=2.1

->Creating the docker compose file: postgres-docker-compose.yaml

-> Deploy it using docker compose command : docker compose up -d

-> Now we need to update some source database parameters to  make data flow work 


-> check for the connector plugins and connectors installed 

curl -s -X GET http://localhost:8083/connector-plugins | jq '.[].class'

curl -s -X GET http://localhost:8083/connectors | jq '.[]'


Adding the required connectors :

-> starting the debezium connector 

    curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @debezium-postgres.json

sink connector: 

-> We are using sink connector we are going to use the jdbc sink connector but we don't have any of the jdbc sink connector listed , so we are going to download the jdbc sink connectors in to the kafka connect

->We can download the jdbc sink connectors from https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc , we are going to choose the manual download and download that as the zip file 

->The go to the lib directory where we can see the lots of jar files for the jdbc connectors.

->Copy these jar files to the Connect container using the docker cp command into the kafka/libs directory inside the container

docker cp ./ postgres-kafka-postgres-connect-1:/kafka/libs

Now restart the  postgres-kafka-postgres-connect-1 container, After restarting the container you can see the jdbc connectors that has been added new.

docker stop  postgres-kafka-postgres-connect-1
docker start  postgres-kafka-postgres-connect-1

-> check for the connector plugins and connectors installed 

curl -s -X GET http://localhost:8083/connector-plugins | jq '.[].class'

curl -s -X GET http://localhost:8083/connectors | jq '.[]'


->Now, we have jar files for sink conector and we need to configure it using json and deploy it : sink-connectior.json

curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @sink-connector.json


-> all connectors are setup , now check the topics and groups created 

    docker compose -f postgres-docker-compose.yaml exec kafka \
    /kafka/bin/kafka-topics.sh \
    --bootstrap-server kafka:9092 \
    --list

docker compose -f postgres-docker-compose.yaml exec kafka \
    /kafka/bin/kafka-consumer-groups.sh\
    --bootstrap-server kafka:9092 \
    --list

-> you should see all the topics created as your given prefix and table names

-> once you perform any DML changes , the below command used to see the events registered in kafka topic 

docker compose -f postgres-docker-compose.yaml exec kafka /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --property print.key=true \
    --topic change_data.public.persons



-- compare source and target to look for changes 

docker compose -f postgres-docker-compose.yaml exec -u postgres postgres_source \
    psql -c 'select * from public.persons;'

docker compose -f postgres-docker-compose.yaml exec -u postgres postgres_sink \
    psql -c 'select * from details_consumed.persons;'


TODO: debug the errors in sink connector ,

-> I can see events registered in kafka topic but the sink connector is not consuming the events registered and not updating in target database .






    

