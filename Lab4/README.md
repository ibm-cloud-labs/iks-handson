# Lab4) Helm チャートを使用したアプリケーションのデプロイ

Lab4では、Kubernetesのパッケージング技術の1つである [Helm](https://helm.sh/) を利用したデプロイの方法を学びます。
Helm チャートと呼ばれる定義ファイルを使用すると，アプリケーションの定義やインストール，アップグレードを容易に行うことができます。

ここで実施する作業は1つです。

- **Helmチャートを使用して `JpetStore`アプリケーションをデプロイする**

`JpetStore`はECサイトを模したサンプルアプリケーションです。

`フロントのWebアプリケーション(Webコンテナ)`と，`動物の画像を格納するDB(DBコンテナ)`で構成されます。今回はいずれもDockerHub上にコンテナイメージとして準備済ですのでビルド作業は行いません。

また，Helmチャートも事前に用意しています。Helmチャートを取得した後に，`helm install xxx`することでアプリケーションをデプロイできます。

> ご自身でコンテナイメージのビルドから実施したい場合は，末尾のセクション `## 参考2: 自身でイメージビルドする方法` をご覧ください。


## 事前準備

helmのバージョン確認とレリポジトリの登録・更新

* helm のバージョン確認

    `helm version`コマンドでインストールされているhelmのバージョンが**v3.0.2以上**で有ることを確認します

    実行例:
    ```bash
    $ helm version
    version.BuildInfo{Version:"v3.0.2", GitCommit:"19e47ee3283ae98139d98460de796c1be1e3975f", GitTreeState:"clean", GoVersion:"go1.13.5"}
    ```

    > helm v2にて必要だったTillerが、helm v3では必要なくなり、Client-only architectureとなりました。
    > https://developer.ibm.com/blogs/kubernetes-helm-3/


* helmリポジトリの登録と更新

    helm 公式リポジトリの登録と情報更新を行います。
    > 今回は利用しませんが、行っておくと便利です

    実行例:
    ```bash
    $ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
    $ helm repo update
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "stable" chart repository
    Update Complete. ⎈ Happy Helming!⎈
    ```
    

## Helmチャートを使用して `JpetStore`アプリケーションをデプロイする

1. ハンズオン用のJpetStoreリポジトリをクローンします。(Lab5でも使用します)
    
    `git`コマンドでクローンします。

    実行例:
    
    ```bash
    $ git clone https://github.com/ibm-cloud-labs/jpetstore-kubernetes-compact.git --depth 1
    ```
        
2. Helmチャートの中身を確認します。

    `JpetStore`アプリケーションをデプロイするためのHelmチャートは`jpetstore-kubernetes-compact/helm/modernpets`ディレクトリに入っています。
    

    実行例:
    
    ```bash
    $ cd jpetstore-kubernetes-compact/helm
    $ tree .
    .
    ├── mmssearch
    │   ├── Chart.yaml
    │   ├── README.md
    │   ├── ics-values.yaml
    │   ├── templates
    │   │   ├── NOTES.txt
    │   │   ├── _helpers.tpl
    │   │   ├── deployment.yaml
    │   │   ├── ingress.yaml
    │   │   └── service.yaml
    │   ├── values-icp.yaml
    │   └── values.yaml
    └── modernpets
        ├── Chart.yaml
        ├── templates
        │   ├── NOTES.txt
        │   ├── _helpers.tpl
        │   ├── deployment.yaml
        │   └── service.yaml
        ├── values-icp.yaml
        └── values.yaml

    4 directories, 17 files
    ```

    上記出力結果に含まれるファイルを使用することで，
    `JpetStore`アプリケーションをK8s上で動作させるために必要な`Deployment`や`Service`などのyamlを生成(して，K8sクラスターにデプロイ)することができます。

    >補足:  
    > 各yamlファイルの中身の確認はここではしませんが，後続のハンズオンで新規にサンプルのHelmチャートを作って紐解いていきます。
    > 
    > 興味のある方は，[Lab6](../Lab6/)を参照ください。
    > 

3. JpetStoreアプリケーションをデプロイします。

    `helm install xxx`コマンドでHelmチャートを使用すると，JpetStoreアプリの`Webコンテナ`と`DBコンテナ`がデプロイされます。
    
    実行例:
    
    ```bash
    jpetstore-kubernetes-compact/helmディレクトリで操作します。
    $ cd ../helm
    $ helm install jpetstore ./modernpets/
    
    デプロイできたか確認
    $ helm list
    NAME     	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART           	APP VERSION
    jpetstore	default  	1       	2020-01-24 14:40:28.405356 +0900 JST	deployed	modernpets-0.1.5	1.0
    
    デプロイされたPodを確認します。
    $ kubectl get pods
    NAME                                                 READY   STATUS    RESTARTS   AGE
    jpetstore-modernpets-jpetstoredb-7dd76668b5-sl6br    1/1     Running   0          1h
    jpetstore-modernpets-jpetstoreweb-6d49474455-7wm85   1/1     Running   0          1h
    jpetstore-modernpets-jpetstoreweb-6d49474455-tbww6   1/1     Running   0          1h
    ```

    上記出力から，Webコンテナ(`jpetstoreweb`)のPodが2つ，DBコンテナ(`jpetstoredb`)のPodが1つがデプロイされていることが分かります。
    
    >補足:  
    > 自分のコンテナイメージを使用する場合は，`helm/modernpets/values.yaml`の`repository`部分を `<MYREGISTRY>/<MYNAMESPACE>`に置き換えます。
    
4. DeploymentやServiceについても確認します。
    
    実行例:

    ```bash
    $ kubectl  get all
    NAME                                                     READY   STATUS    RESTARTS   AGE
    pod/jpetstore-modernpets-jpetstoredb-7dd76668b5-sl6br    1/1     Running   0          1h
    pod/jpetstore-modernpets-jpetstoreweb-6d49474455-7wm85   1/1     Running   0          1h
    pod/jpetstore-modernpets-jpetstoreweb-6d49474455-tbww6   1/1     Running   0          1h
    
    NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    service/db           ClusterIP   172.21.203.103   <none>        3306/TCP       1h
    service/kubernetes   ClusterIP   172.21.0.1       <none>        443/TCP        2d
    service/web          NodePort    172.21.243.99    <none>        80:31918/TCP   1h
    
    NAME                                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/jpetstore-modernpets-jpetstoredb    1         1         1            1           1h
    deployment.apps/jpetstore-modernpets-jpetstoreweb   2         2         2            2           1h
    
    NAME                                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/jpetstore-modernpets-jpetstoredb-7dd76668b5    1         1         1       1h
    replicaset.apps/jpetstore-modernpets-jpetstoreweb-6d49474455   2         2         2       1h
    ```    
    
    Webコンテナ(`jpetstoreweb`)，DBコンテナ(`jpetstoredb`)それぞれの **Deployment** と **Service** も作成されていることが分かります。
    
    通常は，今回のシンプルなケースであっても以下のyamlファイルを用意し，順にデプロイしていく必要があります。

        - `web-deployment.yaml`
        - `web-service.yaml`
        - `db-deployment.yaml`
        - `db-service.yaml`

    このようにHelmチャートを使うことで、一括デプロイやロールバックなどの管理がやりやすくなります。
    
5. ブラウザ上でアプリケーションの動作を確認します。

    ブラウザで`<Public IP>:<NodePort>`を開きます。
    
    >補足:  
    > ワーカーノードの `Public IP` は以下のように確認します。
    > 
    > ```bash
    > $ ibmcloud ks workers mycluster
    > OK
    > ID                                                 Public IP       Private IP      Machine Type   State    Status   Zone    Version
    > kube-hou02-pa705552a5a95d4bf3988c678b438ea9ec-w1   184.173.52.92   10.76.217.175   free           normal   Ready    hou02   1.10.12_1543
    > ```
    > 
    > `NodePort` は以下のように確認します。
    > 
    > ```bash
    > $ kubectl get service
    > NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    > db           ClusterIP   172.21.203.103   <none>        3306/TCP       1h
    > kubernetes   ClusterIP   172.21.0.1       <none>        443/TCP        2d
    > web          NodePort    172.21.243.99    <none>        80:31918/TCP   1h
    > ```
    > 
    > 上記の出力例の場合の `<Public IP>:<NodePort>`は，次のようになります。
    > - Public IP: `184.173.52.92`
    > - NodePort: `31918`
    > 
    > したがって，ブラウザ上で `184.173.52.92:31918` にアクセスするとアプリケーションが開きます。

    ページ内のリンクをドリルダウンして，動物画像を開いてみてください。正常に動作していれば以下図のように確認できます。
    
    ![](images/petstore.png)

以上でLab4は終了です。  
最後のハンズオンは[Lab5](../Lab5/)です。


******

## 参考1: YAMLファイルを使用したデプロイ (今回は実施しません。)

  JpetStoreアプリのyamlファイルは， `jpetstore-kubernetes-compact/jpetstore` ディレクトリ配下にあります。

    実行例: 

    ```bash
    jpetstore-kubernetes-helm/jpetstore ディレクトリで操作します。
    $ kubectl apply -f jpetstore.yaml
    deployment.extensions "jpetstoreweb" created
    service "web" created
    deployment.extensions "jpetstoredb" created
    service "db" created
    ```

    >補足:  
    > 自分のコンテナイメージを使用する場合は`jpetstore/jpetstore.yaml`の`image`セクションを `<MYREGISTRY>/<MYNAMESPACE>`に置き換えます。

## 参考2: 自身でイメージビルドする方法

1. ソースコードを入手します。

    ```bash
    $ git clone https://github.com/ibm-cloud-labs/jpetstore-kubernetes-allinone.git
    $ cd jpetstore-kubernetes
    ```

    > 補足:  
    > フォルダーの構成
    > クローンしたリポジトリは以下のファイルから構成されています。
    > 
    > | フォルダー | 説明 |
    > | ---- | ----------- |
    > |[**jpetstore**](https://github.com/ibm-cloud-labs/jpetstore-kubernetes-allinone/tree/master/jpetstore)| Javaでかかれたペットショップのアプリケーション |
    > |[**mmssearch**](https://github.com/ibm-cloud-labs/jpetstore-kubernetes-allinone/tree/master/mmssearch)| GOで実装された画像認識機能付きチャットアプリ |
    > |[**helm**](https://github.com/ibm-cloud-labs/jpetstore-kubernetes-allinone/tree/master/helm)| KubernetesにデプロイするためのHelm チャート |
    > |[**pet-images**](https://github.com/ibm-cloud-labs/jpetstore-kubernetes-allinone/tree/master/pet-images)| チャットアプリの動作確認用の動物画像ファイル |


ここでは，ビルドしたコンテナイメージの置き場としてIBM Cloud Container Registryを使用します。  
もちろんDockerHubやご自身のプライベートレジストリーを使用することもできます。その場合は`<MYREGISTRY>`部分を適宜置き換えてください。

2. レジストリーの **Namespace** を設定します。

    以下のコマンドを実行すると`Namespace`の一覧が表示されます。

    ```bash
    $ ibmcloud cr namespaces
    ```

    既存の`Namespace`がなく，新規に作成する場合は以下のコマンドを実行してください。
    
    ```bash
    $ ibmcloud cr namespace-add <NAMESPACE>
    ```

3. **Container Registry** (e.g. registry.ng.bluemix.net) の情報を確認します。

    ```bash
    Container Registry                us.icr.io
    Container Registry API endpoint   https://us.icr.io/api
    IBM Cloud API endpoint            https://cloud.ibm.com
    IBM Cloud account details         Hoge Fuga Account (xxxxxxxxxxxxxxxxx)
    IBM Cloud organization details     ()
    ```

4. **jpetstoreweb** イメージをビルドし，レジストリーにプッシュします。 

    ```bash
    jpetstore-kubernetes-allinone/jpetstoreディレクトリで操作します。
    $ cd jpetstore
    $ docker build . -t <MYREGISTRY>/<MYNAMESPACE>/jpetstoreweb
    $ docker push <MYREGISTRY>/<MYNAMESPACE>/jpetstoreweb
    ```

   >補足:  
   > `Unauthorized ` と表示された場合は`ibmcloud cr login` を実行してIBM Cloudにログインしてください。

5. 同様に， **jpetstoredb** イメージをビルドします。

    ```bash
    jpetstore-kubernetes-allinone/jpetstore/dbディレクトリで操作します。
    $ cd db
    $ docker build . -t <MYREGISTRY>/<MYNAMESPACE>/jpetstoredb
    $ docker push <MYREGISTRY>/<MYNAMESPACE>/jpetstoredb
    ```

6. レジストリーへのプッシュが完了したことを確認するために、IBM Cloud Container Registryに保存されたイメージの一覧を表示します。 

    ```bash
    $ ibmcloud cr images
    ```
