# qiita-fiware-construct

## はじめに

以下のQiita記事で構築したFIWARE基盤に使用したyamlファイルを公開しています！

http://

## 構築について

### Azure

Azureポータルにログイン
ポータル上部にある検索バーで"Kebernetes サービス"と検索
以下の設定でAKSを構築

* サブスクリプション：任意
* リソースグループ：任意
* クラスター名：任意
* 地域：任意(私は"Japan East"で作成)
* ノードサイズ：Standard A4 v2
* スケーリング方法：手動
* ノード数：3

### Orion, MongoDB, Kong

AzureにCLIからログイン
```
azure login --tenant TENANT_NAME
```

Azure AKSに接続
```
az aks get-credentials --resource-group RESOURCE_GROUP_NAME --name AKS_NAME --overwrite-existing
```

Helmを利用してMongoDBを構築
```
helm repo add azure-marketplace https://marketplace.azurecr.io/helm/v1/repo
helm repo update
helm install azure-marketplace/mongodb --name-template mongodb -f ./qiita-fiware-construct/MONGO_DB.yaml
```

クラスター化の設定
```
kubectl run mongodb-client --rm --tty -i --restart='Never' --image marketplace.azurecr.io/bitnami/mongodb:4.4.15-debian-10-r2 --command -- \
  mongosh admin --host 'mongodb-0.mongodb-headless.default.svc.cluster.local:27017,mongodb-1.mongodb-headless.default.svc.cluster.local:27017,mongodb-2.mongodb-headless.default.svc.cluster.local:27017' \
  --eval 'rs.add("mongodb-1.mongodb-headless.default.svc.cluster.local:27017")'

kubectl run mongodb-client --rm --tty -i --restart='Never' --image marketplace.azurecr.io/bitnami/mongodb:4.4.15-debian-10-r2 --command -- \
  mongosh admin --host 'mongodb-0.mongodb-headless.default.svc.cluster.local:27017,mongodb-1.mongodb-headless.default.svc.cluster.local:27017,mongodb-2.mongodb-headless.default.svc.cluster.local:27017' \
  --eval 'rs.add("mongodb-2.mongodb-headless.default.svc.cluster.local:27017")'

kubectl run mongodb-client --rm --tty -i --restart='Never' --image marketplace.azurecr.io/bitnami/mongodb:4.4.15-debian-10-r2 --command -- \
  mongosh admin --host 'mongodb-0.mongodb-headless.default.svc.cluster.local:27017,mongodb-1.mongodb-headless.default.svc.cluster.local:27017,mongodb-2.mongodb-headless.default.svc.cluster.local:27017' \
  --eval 'rs.status().members.map(function(e) {return {name: e.name, stateStr:e.stateStr};})'
```

Orionをデプロイ
```
kubectl apply -f ./qiita-fiware-construct/ORION_SERVICE.yaml
kubectl apply -f ./qiita-fiware-construct/ORION_DEPLOYMENT.yaml
```

Kongをデプロイ
```
kubectl create -f https://bit.ly/k4k8s
kubectl -n kong scale deployments.v1.apps/ingress-kong --replicas=1
```

***ドメイン名の取得***

Azureポータル上で"App Service ドメイン"と検索し、一意のドメイン名を所得
※ドメイン1つにつき、11.99米国ドルかかります。

Cert-managerをデプロイ
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.2/cert-manager.yaml
```

Kongを通るようにレコードを設定
```
az network dns record-set a add-record --resource-group DNS_RESOURCE_GROUP --zone-name "DOMAIN_NAME" --record-set-name "HOST_NAME" --ipv4-address "KONG_EXTERNAL_IP"
```

TLS証明書を取得
```
kubectl apply -f ./qiita-fiware-construct/CERTMANAGER_CLUSTERISSURE.yaml
```

HTTPS通信のみを許可するように設定
```
kubectl apply -f ./qiita-fiware-construct/KONG_INGRESS.yaml
```

APIキーを使った認証を設定
```
kubectl create secret generic kong-keyauth --from-literal=kongCredType=key-auth --from-literal=key=API_KEY
kubectl apply -f ./qiita-fiware-construct/KONG_CONSUMER.yaml
kubectl apply -f ./qiita-fiware-construct/KONG_PLUGINS.yaml
```

Orionへの通信設定
```
kubectl apply -f ./qiita-fiware-construct/ORION_INGRESS.yaml
```

### QuantumLeap, CrateDB

QuantumLeapをデプロイ
```
kubectl apply -f ./qiita-fiware-construct/QUANTUMLEAP_SERVICE.yaml
kubectl apply -f ./qiita-fiware-construct/QUANTUMLEAP_STATEFULSET.yaml
```

QuantumLeapへの通信設定
```
kubectl apply -f ./qiita-fiware-construct/QUANTUMLEAP_INGRESS.yaml
```

CrateDBのデプロイと設定
```
kubectl apply -f ./qiita-fiware-construct/CRATEDB_SERVICE.yaml
kubectl apply -f ./qiita-fiware-construct/CRATEDB_DEPLOYMENT.yaml
```

より詳細な情報はQiita記事をご覧ください！

## LISENCE
