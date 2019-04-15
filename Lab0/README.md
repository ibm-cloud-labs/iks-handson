# Lab 0. セットアップ
Kubernetes(K8s)ハンズオンを実施するための事前準備を行います。

## 0. IKSを使ったハンズオンを実施するための準備
K8sクラスターは[IKS(IBM Cloud Kubernetes Service)](https://www.ibm.com/cloud/container-service)を使用します。IKSに接続できることをゴールとして以下の準備を行います。

- IBM Cloud CLI(`ibmcloud`) と関連プラグイン，K8s CLIなどの導入
- K8sクラスター(IKS)の構成
- K8sクラスターへの接続確認

1. CLIのインストール **(※事前準備にて実施済のため操作は不要です)**

    ["IBM Cloud Developer Tools のインストール方法"](https://console.bluemix.net/docs/cli/index.html#overview) に従い，ご利用されているOSに合わせたコマンドを実行してください。

2. K8sクラスターの作成 **(※セミナー会場にて実施済のため操作は不要です)**

    [IBM Cloud](https://cloud.ibm.com/containers-kubernetes/catalog/cluster/create)でクラスターを作成します。  
    ibmcloudコマンドで作成する場合は， `$ ibmcloud cs cluster-create --name <name-of-cluster>` コマンドを実行します。

3. CLI で IBM Cloudにログイン

    `$ ibmcloud login` (Windowsの方は，PowerShellではなく**コマンドプロンプト**をご利用ください)
    
    以下をインタラクティブに入力します。
    
    - APIエンドポイントのリージョン: `us-south`
    - IBM ID: `自身のIBM Cloudアカウント(Emailアドレスの場合が多いです)`
    - パスワード: `パスワード`

    実行例:
    
    ```bash.sh
    $ ibmcloud login

    Select an API endpoint:
    1. us-south - https://api.ng.bluemix.net
    2. eu-de - https://api.eu-de.bluemix.net
    3. eu-gb - https://api.eu-gb.bluemix.net
    4. au-syd - https://api.au-syd.bluemix.net
    5. us-east - https://api.us-east.bluemix.net
    6. Enter a different API endpoint
    Enter a number> 1

    Email> kzsaitoiks@gmail.com

    Password>
    Authenticating...
    OK

    Targeted account Kazufumi Saito's Account (XXXXXXXXXXXXXXXX)

    Targeted resource group Default


    API endpoint:      https://api.ng.bluemix.net
    Region:            us-south
    User:              kzsaitoiks@gmail.com
    Account:           Kazufumi Saito's Account (XXXXXXXXXXXXXXXX)
    Resource group:    Default
    CF API endpoint:
    Org:
    Space:
    ```

    >補足:  
    >既に入力済の情報がある場合はスキップされる場合があります。  
    >例えば，APIエンドポイント(リージョン)が`au-syd`に自動設定されている場合があります。後の手順で`us-south`を指定しますのでそのまま進めてください。


4. APIエンドポイント(リージョン)を `us-south` に指定

    本日はダラス(=us-south)上に IKSクラスターを構築しているので，ibmcloud cliが指すリージョンを `us-south` に設定します。
    
    実行例:
    
    ```bash.sh
    $ ibmcloud cs region-set us-south
    ```
    
    >補足:  
    >`cs`(container-service)プラグインは`ibmcloud`でIBM Cloud Kubernetes Service (IKS)を操作するためのプラグインです。K8sクラスター自体の操作・制御は`kubectl (後述)`で行いますが，そのためのK8sクラスターへの接続情報の取得(後述)やK8sクラスターに割り振られているIPアドレスの確認，ノード構成の情報取得など、IBM Cloudのサービスに関わる操作には`ibmcloud cs`コマンドを使う必要があります。なお、現在はcsプラグインの後継版の`ks`(kubernetes-service)プラグインが使用可能です。

5. 接続情報の取得
   
    `$ ibmcloud cs cluster-config <name-of-cluster>` を実行し，K8sクラスターへの接続情報を取得して，コピーします。
    (この後の手順でペーストします)

    実行例:

    ```bash.sh
    $ ibmcloud cs cluster-config mycluster
    OK
    The configuration for mycluster was downloaded successfully.

    Export environment variables to start using Kubernetes.

    export KUBECONFIG=/Users/capsmalt/.bluemix/plugins/container-service/clusters/mycluster/kube-config-hou02-mycluster.yml
    ```
 
    **export KUBECONFIG=/Users/capsmalt/.bluemix/plugins/container-service/clusters/mycluster/kube-config-hou02-mycluster.yml**
    
    上記をコピーします。
 
    > トラブルシュート:  
    > お使いのCLIやプラグインのインストール状況によっては，「アップデート」「新規インストール」を必要とする場合があります。必要に応じて，ログ出力に従って，以下のようなコマンドを実行してください。  
    > サポートが必要な場合は，お声がけください。
    >
    >```bash.sh
    > 例) プラグインの新規インストールが必要なケース (csプラグインのインストール例)
    > $ ibmcloud plugin install container-service -r Bluemix
    > 
    > 例) プラグインのアップデートが必要なケース (csプラグインのアップデート例)
    > $ ibmcloud plugin update container-service -r Bluemix
    > ```
 
6. 5.で取得した 自身の`KUBECONFIG` の情報をexport(set)します

    実行例:
    
    **(Windowsの場合は，exportではなく "set" を使用します)**

    ```bash.sh
    $ export KUBECONFIG=/Users/capsmalt/.bluemix/plugins/container-service/clusters/mycluster/kube-config-hou02-mycluster.yml
    ```

    >補足:  
    > Kubernetesクラスターを操作する際には，Kubernetesのクライアント用CLI `kubectl` を使用します。その際に **KUBECONFIG(K8sクラスターへの接続情報)** が必要になります。
    > クライアント(自PC)から `kubectl` CLIを実行して，サーバー(kube-apiserverコンポーネント)に通信することで，**直接K8s上のコンテナを制御するのではなく**， kube-apiserverからK8sクラスターを制御しています。kube-apiserverはMasterノード上で動作するコンポーネントです。


6. K8sクラスターに接続できるか確認します

    下記のように，K8sクラスターの情報(IPアドレス，STATUS，など)が返ってくればOKです。
    
    ```bash.sh
    $ kubectl get nodes
    NAME            STATUS   ROLES    AGE   VERSION
    10.76.217.175   Ready    <none>   12h   v1.10.12+IKS
    ```

以上でKubernetesクラスター(IKS)の使用準備は完了です。  
[Lab1](../Lab1)を進めてください。