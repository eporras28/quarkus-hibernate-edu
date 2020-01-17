# quarkus-hibernate-edu
# local
docker run --ulimit memlock=-1:-1 --rm=true --memory-swappiness=0 \
    --name postgres-database -e POSTGRES_USER=sa \
    -e POSTGRES_PASSWORD=sa -e POSTGRES_DB=person \
    -p 5432:5432 postgres:10.5 >& /dev/null &
mvn compile quarkus:dev



# openshift
oc login 2886795363-8443-cykoria05.environments.katacoda.com --insecure-skip-tls-verify=true -u developer -p developer

oc new-project quarkus --display-name="Sample Quarkus Datatable App"

mvn clean package

oc new-app \
    -e POSTGRESQL_USER=sa \
    -e POSTGRESQL_PASSWORD=sa \
    -e POSTGRESQL_DATABASE=person \
    --name=postgres-database \
    openshift/postgresql
    
oc new-build registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift:1.5 --binary --name=quarkus-datatable -l app=quarkus-datatable

rm -rf target/binary && mkdir -p target/binary && cp -r target/*runner.jar target/lib target/binary

oc start-build quarkus-datatable --from-dir=target/binary --follow

oc new-app quarkus-datatable \
   -e QUARKUS_DATASOURCE_URL=jdbc:postgresql://postgres-database:5432/person \
   -e JAVA_OPTIONS='-Dfile.encoding=UTF8'
   
oc expose service quarkus-datatable

oc rollout status -w dc/quarkus-datatable

curl http://quarkus-datatable-quarkus.2886795363-80-cykoria05.environments.katacoda.com/person/birth/before/2000 | jq

http://quarkus-datatable-quarkus.2886795363-80-cykoria05.environments.katacoda.com/
