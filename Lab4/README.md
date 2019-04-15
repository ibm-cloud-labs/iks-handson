# Lab4) Helm チャートを使用したアプリケーションのデプロイ

Lab4では、Kubernetesのパッケージング技術の1つである [Helm](https://helm.sh/) を利用したデプロイの方法を学びます。
Helm チャートと呼ばれる定義ファイルを使用すると，アプリケーションの定義やインストール，アップグレードを容易に行うことができます。

ここで実施する作業は1つです。

- **Helmチャートを使用して `JpetStore`アプリケーションをデプロイする**

`JpetStore`はECサイトを模したサンプルアプリケーションです。

`フロントのWebアプリケーション(Webコンテナ)`と，`動物の画像を格納するDB(DBコンテナ)`で構成されます。今回はいずれもDockerHub上にコンテナイメージとして準備済ですのでビルド作業は行いません。

また，Helmチャートも事前に用意しています。Helmチャートを取得した後に，`helm install xxx`することでアプリケーションをデプロイできます。

>補足:  
> ご自身でコンテナイメージのビルドから実施したい場合は，末尾のセクション `## 参考2: 自身でイメージビルドする方法` をご覧ください。
> 

## Helmチャートを使用して `JpetStore`アプリケーションをデプロイする

1. ハンズオン用のJpetStoreリポジトリをクローンします。(Lab5でも使用します)
    
    `git`コマンドでクローンします。

    実行例:
    
    ```bash
    $ git clone https://github.com/capsmalt/jpetstore-kubernetes-min.git
    ```
        
2. Helmチャートの中身を確認します。

    `JpetStore`アプリケーションをデプロイするためのHelmチャートは`jpetstore-kubernetes-min/helm/modernpets`ディレクトリに入っています。
    

    実行例:
    
    ```bash
    $ cd jpetstore-kubernetes-min/helm
    $ ls
    mmssearch	modernpets
    $ cd modernpets
    $ ls 
    Chart.yaml	templates	values.yaml
    ```

    ```bash
    modernpetsフォルダの中身を確認
    $ tree
    .
    ├── Chart.yaml
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   └── service.yaml
    └── values.yaml

    1 directory, 6 files
    ```

    上記出力結果に含まれるファイルを使用することで，
    `JpetStore`アプリケーションをK8s上で動作させるために必要な`Deployment`や`Service`などのyamlを生成(して，K8sクラスターにデプロイ)することができます。

    >補足:  
    > 各yamlファイルの中身の確認はここではしませんが，後続のハンズオンで新規にサンプルのHelmチャートを作って紐解いていきます。
    > 
    > 興味のある方は，[Lab6](../Lab6/)を参照ください。
    > 

3. Helmコマンドを使うための事前準備を行います。

    K8sクラスター上のHelmサーバー(Tillerというコンポーネント)へ接続できるように構成する必要があります。
    
    以下の手順で`IKS接続のための事前準備`を実施してください。
    
    実行例: 
    
    ```bash
    $ ibmcloud login
    $ ibmcloud cs cluster-config mycluster
    出力結果の`export xxx`をターミナル上でペーストする
    $ export xxxx
    $ kubectl get nodes
    XXXXXXX 正常に実行できることが確認できればOK
    $ ibmcloud cs cluster-get mycluster
    ```

4. Helmの初期化を行います。

    実行例:
    
    ```bash
    $ helm init
    $HELM_HOME has been configured at /Users/XXXXX/.helm.    

    Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
    Happy Helming!
    ```
    
    Warningが出る場合がありますが，ここでは **"Happy Helming!"** が確認できればOKです。
    
5. JpetStoreアプリケーションをデプロイします。

    `helm install xxx`コマンドでHelmチャートを使用すると，JpetStoreアプリの`Webコンテナ`と`DBコンテナ`がデプロイされます。
    
    実行例:
    
    ```bash
    jpetstore-kubernetes-min/helmディレクトリで操作します。
    $ cd ../helm
    $ helm install --name jpetstore ./modernpets/
    
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
    
6. DeploymentやServiceについても確認します。
    
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
    
7. ブラウザ上でアプリケーションの動作を確認します。

    ブラウザで`<Public IP>:<NodePort>`を開きます。
    
    >補足:  
    > ワーカーノードの `Public IP` は以下のように確認します。
    > 
    > ```bash
    > $ ibmcloud cs workers mycluster
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

  JpetStoreアプリのyamlファイルは， `jpetstore-kubernetes-min/jpetstore` ディレクトリ配下にあります。

    実行例: 

    ```bash
    jpetstore-kubernetes-min/jpetstore ディレクトリで操作します。
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
    $ git clone https://github.com/capsmalt/jpetstore-kubernetes-all.git
    $ cd jpetstore-kubernetes-all
    ```

    > 補足:  
    > フォルダーの構成
    > クローンしたリポジトリは以下のファイルから構成されています。
    > 
    > | フォルダー | 説明 |
    > | ---- | ----------- |
    > |[**jpetstore**](https://github.com/kissyyy/jpetstore-kubernetes/tree/master/jpetstore)| Javaでかかれたペットショップのアプリケーション |
    > |[**mmssearch**](https://github.com/kissyyy/jpetstore-kubernetes/tree/master/mmssearch)| GOで実装された画像認識機能付きチャットアプリ |
    > |[**helm**](https://github.com/kissyyy/jpetstore-kubernetes/tree/master/helm)| KubernetesにデプロイするためのHelm チャート |
    > |[**pet-images**](https://github.com/kissyyy/jpetstore-kubernetes/tree/master/pet-images)| チャットアプリの動作確認用の動物画像ファイル |


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
    Container Registry                      registry.ng.bluemix.net
    Container Registry API エンドポイント   https://registry.ng.bluemix.net/api
    IBM Cloud API エンドポイント            https://cloud.ibm.com
    IBM Cloud アカウントの詳細              XXXX Account (xxxxxxxxxxxxxxxxxxxxx)
    IBM Cloud 組織の詳細                     ()
    ```

4. **jpetstoreweb** イメージをビルドし，レジストリーにプッシュします。 

    ```bash
    jpetstore-kubernetes-all/jpetstoreディレクトリで操作します。
    $ cd jpetstore
    $ docker build . -t <MYREGISTRY>/<MYNAMESPACE>/jpetstoreweb
    $ docker push <MYREGISTRY>/<MYNAMESPACE>/jpetstoreweb
    ```

   >補足:  
   > `Unauthorized ` と表示された場合は`ibmcloud cr login` を実行してIBM Cloudにログインしてください。

5. 同様に， **jpetstoredb** イメージをビルドします。

    ```bash
    jpetstore-kubernetes-all/jpetstore/dbディレクトリで操作します。
    $ cd db
    $ docker build . -t <MYREGISTRY>/<MYNAMESPACE>/jpetstoredb
    $ docker push <MYREGISTRY>/<MYNAMESPACE>/jpetstoredb
    ```

6. レジストリーへのプッシュが完了したことを確認するために、IBM Cloud Container Registryに保存されたイメージの一覧を表示します。 

    ```bash
    $ ibmcloud cr images --restrict $MYNAMESPACE
    ```
