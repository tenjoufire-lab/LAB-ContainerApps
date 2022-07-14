# LAB-ContainerApps
Container Apps 101

## Azure CLI へのログイン
Azure CLI 2.37 以上のバージョンを利用できる環境が必要です。
また、Container Apps の Extension や各種事前設定が必要となりますので、下記コマンドを実行してください。

```
az extension add --name containerapp --upgrade
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights
```

# 1. Container Apps の作成
Container Apps を作成する際に必要なもの
- Container Apps 環境
- Azure Container Registry / Docker Hub
- Azure Log Analytics Workspace
- 仮想ネットワーク（Private 通信が必要な場合）

## Container Apps 環境の作成
Container Apps をデプロイする前に、コンテナアプリをホストする場所が必要になります。Azure Container Apps では、そのもとになるインフラ部分を Environment と呼び、Container Apps の論理的な境界を構成します。同じ環境にデプロイされた Container Apps は同じネットワークにデプロイされ、同じ Log Analyitics Workspace にログが書き込まれます。

Azure Log Analytics は Conttainer Apps の監視に利用され、Container Apps 環境を作成する際に必要です。

１．変数の設定
リソースグループ名や Container Apps 環境名はご自身のお好きなものに変えていただいて問題ございません。

```
RESOURCE_GROUP="rg-my-container-apps"
LOCATION="japaneast"
LOG_ANALYTICS_WORKSPACE="my-container-apps-logs"
CONTAINERAPPS_ENVIRONMENT="aca-environment"
```

2．リソースグループと Log Analytics Workspace の作成

```
az group create --name $RESOURCE_GROUP --location "$LOCATION"
az monitor log-analytics workspace create --resource-group $RESOURCE_GROUP --workspace-name $LOG_ANALYTICS_WORKSPACE
```

次に、Log Analytics Workspace のクライアント ID とシークレットを変数に登録します。

```
LOG_ANALYTICS_WORKSPACE_CLIENT_ID=`az monitor log-analytics workspace show --query customerId -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv`

LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET=`az monitor log-analytics workspace get-shared-keys --query primarySharedKey -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv`
```

3．Container Apps 環境の作成

```
az containerapp env create \
--name $CONTAINERAPPS_ENVIRONMENT \
--resource-group $RESOURCE_GROUP \
--logs-workspace-id $LOG_ANALYTICS_WORKSPACE_CLIENT_ID \
--logs-workspace-key $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET \
--location "$LOCATION"
```