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

より詳細な情報はQiita記事をご覧ください！
