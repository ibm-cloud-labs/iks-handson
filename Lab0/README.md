# Lab 0. セットアップ
Kubernetes(K8s)ハンズオンを実施するための事前準備を行います。

## 0. IKSクラスタの作成、CLIのインストール
IKSハンズオンに必要なKubernetesクラスタの作成と、ローカルマシンのセットアップを行います

> 事前に実施済みの方はスキップして、 **1. IBM CLIログイン** に進んでください。

ガイドに従い実行ください。　https://github.com/cloud-handson/iks-handson-setup/blob/master/README.md


## 1. IBM CLIを使ってログイン
ibmcloud cli を使ってIBM Cloudにログインします

> Windowsユーザーの方は**コマンドプロンプト**を利用してください。

```
$ ibmcloud login -a https://cloud.ibm.com
```

> output
```
ibmcloud loginAPI endpoint: https://cloud.ibm.com
Email> hoge@fuga.com                   < メールアドレスを入力
Password>                              < パスワードを入力
Authenticating...OK


Select an account:
1. KOTA SATO's Account (xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx)
Enter a number> 1               < 複数のアカウントに結び付けられている場合は利用するアカウントを選択


Targeted account KOTA SATO's Account (xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx)
Targeted resource group default


Select a region (or press enter to skip):
1. au-syd
2. jp-tok
3. eu-de
4. eu-gb
5. us-south
6. us-east
Enter a number> 5            < us-southを選択
Targeted region us-south


API endpoint:      https://cloud.ibm.com
Region:            us-south
User:              hoge@fuga.com
Account:           KOTA SATO's Account (xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx)
Resource group:    default
CF API endpoint:
Org:
Space:
OK
```


## 2. Kubernetesクラスタへの接続情報の取得
ibmcloud cli にてIKSクラスタを操作できるように接続情報を取得します

1. 接続情報の取得
    `ibmcloud ks cluster-config <cluster-name>`コマンドを使って、クラスタへの接続情報を取得します

    ```
    $ ibmcloud ks cluster-config mycluster
    ```

    > Output
    ```
    OK
    The configuration for mycluster was downloaded successfully.

    Export environment variables to start using Kubernetes.

    export KUBECONFIG=/Users/hogehoga/.bluemix/plugins/container-service/clusters/mycluster/kube-config-tok02-mycluster.yml
    ```

    **上記コマンドの出力結果から、export KUBECONFIG=/Users/********-mycluster.yml の行をコピーしてください**

2. 環境変数の定義
    上記コマンドの`KUBECONFIG`を環境変数に定義します。

    > 実行例
    > Windowsの場合は，exportではなく **set** を使用します
    ```
    $ export KUBECONFIG=/Users/hogehoga/.bluemix/plugins/container-service/clusters/mycluster/kube-config-tok02-mycluster.yml
    ```


3. 接続確認
    IKSクラスタへの接続確認を行います

    ```
    $ kubectl get nodes
    ```

    > Output
    ```
    AME            STATUS   ROLES    AGE   VERSION
    10.129.177.57   Ready    <none>   1d   v1.13.5+IKS
    ```

以上でKubernetesクラスター(IKS)の使用準備は完了です。  
[Lab1](../Lab1)を進めてください。


## Tips, トラブルシュート
* ibmcloud cliを操作している際に、バージョンアップしてくださいと表示される
    * サンプルメッセージ
        ```
        Plugin version 0.2.102 is now available. To update run: ibmcloud plugin update container-service -r Bluemix
        ```
    * 対応方法
  
        新しいプラグインにアップデートしてください
        ```
        $ ibmcloud plugin update --all
        ```
