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

```
azure login --tenant TENANT_NAME
```

より詳細な情報はQiita記事をご覧ください！
