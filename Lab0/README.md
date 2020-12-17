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
Email> hoge@fuga.com        < メールアドレスを入力
Password>                   < パスワードを入力
Authenticating...OK


Select an account:
1. KOTA SATO's Account (xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx)
Enter a number> 1           < 複数のアカウントに結び付けられている場合は利用するアカウントを選択


Targeted account KOTA SATO's Account (xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx)
Targeted resource group default


Select a region (or press enter to skip):
1. au-syd
2. jp-tok
3. kr-seo
4. eu-de
5. eu-gb
6. us-south
7. us-east
Enter a number> 6          < us-southの番号を選択。
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

リソースグループの指定が求められた場合は、次のコマンドを実行します。利用しているリソースグループ名は、[IBM Cloudダッシュボード](https://cloud.ibm.com/login)の[管理]→[アカウント]→[アカウント・リソース]から確認できます。
```
ibmcloud target -g <リソースグループ名>
```

## 2. Kubernetesクラスタへの接続情報の取得
ibmcloud cli にてIKSクラスタを操作できるように接続情報を取得します  
(最新バージョンCLIでは、一部操作が異なる場合があります)  
2020年6月8日現在、IBM Cloud CLI Plugin (container-service/kubernetes-service)の最新バージョン1.0.8を利用している場合は、`ibmcloud ks cluster config --cluster mycluster` のコマンドを実行してください。1.および2.の手順と同じ操作内容を実行した状態になりますので、実行後は3に進んでください。

1. 接続情報の取得
    `ibmcloud ks cluster-config <cluster-name>`コマンドを使って、クラスタへの接続情報を取得します

    ```
    $ ibmcloud ks cluster config --cluster <自分のクラスタID>
    ```

例）
    ```
    $ ibmcloud ks cluster config --cluster bvddfq0d0geutccup6ig
    ```

    > Output
    ```
    OK
    The configuration for bvddfq0d0geutccup6ig was downloaded successfully.
    
    Added context for bvddfq0d0geutccup6ig to the current kubeconfig file.
    You can now execute 'kubectl' commands against your cluster. For example, run 'kubectl get nodes'.
    If you are accessing the cluster for the first time, 'kubectl' commands might fail for a few seconds while RBAC synchronizes.
    ```

2. 接続確認
    IKSクラスタへの接続確認を行います

    ```
    $ kubectl get nodes
    ```

    > Output
    ```
    NAME          STATUS   ROLES    AGE   VERSION
    10.131.74.2   Ready    <none>   32m   v1.18.12+IKS
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
