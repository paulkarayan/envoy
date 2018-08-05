see:
https://blog.turbinelabs.io/incremental-blue-green-deploys-fde90b0ebdab


1. add service1a to docker-compose.yml
2. add service1a to clusters section of front-envoy.yaml
3. add new route with a headers field in filters section of front-envoy.yaml

Since routing rules are applied in order, we’ll add this rule to the top of the routes key. Requests with a header that matches our new rule will be sent to our new service, while requests that don't include this header will still get service 1.

4. restart
docker-compose down --remove-orphans
docker-compose up --build -d


5. routing with headers vs. no headers

If we make a request to our service with no headers, we’ll get a response from service 1:
curl localhost:8000/service/1

or you can use headers to get service1a
curl -H 'x-canary-version: service1a' localhost:8000/service/1

yay!


6. add weighted rule to front-envoy.yaml
-> this replaces the previous match rule:
prefix: "/service/1"

7. restart
docker-compose down --remove-orphans
docker-compose up --build -d

8. show that we do see the weighted routing (canary ftw!)

curl localhost:8000/service/1

for ((n=0;n<10;n++))
do
    curl localhost:8000/service/1
done