# TMF Product Catalog Management API

## Overview
This server stub was generated by the [OpenAPI Generator](https://openapi-generator.tech) project.
```
cd ~/Downloads
wget https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli/5.4.0/openapi-generator-cli-5.4.0.jar -O openapi-generator-cli.jar
java -jar openapi-generator-cli.jar generate -g spring --additional-properties artifactId=tmf-product-catalog-management-api,apiPackage=com.vodafone,hideGenerationTimestamp:true -o ~/Downloads/vodafone-productcatalogue -i TMF620-ProductCatalog-v4.1.0.swagger.json
```

Start your server as a simple java application.
```
mvn spring-boot:run
```

You can view the api documentation in swagger-ui by pointing to
http://localhost:8080/

## Accelerator deployment
```
tanzu accelerator apply -f accelerator-deployment.yaml
```
or
```
tanzu accelerator create tmf-product-catalog-management-api-java --git-repository https://github.com/tsalm-pivotal/tm-forum-java.git --git-branch main
```

## Install Mongo DB operator / instance
```
git clone https://github.com/mongodb/mongodb-kubernetes-operator.git
```
Follow instructions [here](https://github.com/mongodb/mongodb-kubernetes-operator/blob/master/docs/install-upgrade.md#operator-in-different-namespace-than-resources)

```
cat << EOF | kubectl apply -n <namespace> -f -
---
apiVersion: mongodbcommunity.mongodb.com/v1
kind: MongoDBCommunity
metadata:
  name: mongodb
spec:
  members: 1
  type: ReplicaSet
  version: "4.2.7"
  security:
    authentication:
      modes: ["SCRAM"]
  users:
    - name: admin
      passwordSecretRef:
        name: mongodb-admin-pw
      roles:
        - name: clusterAdmin
          db: admin
        - name: userAdminAnyDatabase
          db: admin
      scramCredentialsSecretName: mongodb-admin-scram
---
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-admin-pw
type: Opaque
stringData:
  password: <password>
EOF
```

```
kubectl describe mongodbcommunity -n <namespace>
```

```
cat << EOF | kubectl apply -n <namespace> -f -
---
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-binding-compatible
type: Opaque
stringData:
  type: mongodb
  provider: in-cluster
  host: <hostname>
  port: <port>
  database: admin
  username: admin
  password: <password>
EOF
```

```
tanzu service claim create mongodb-binding-compatible \
  --resource-name mongodb-binding-compatible \
  --resource-kind Secret \
  --resource-api-version v1 -n <namespace>
```

```
tanzu service claim list -o wide -n <namespace>
```