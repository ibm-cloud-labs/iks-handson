# Kubernetes ハンズオン
IKS (IBM Cloud Kubernetes Service) を使用して以下のことを学びます。

- IKSの使用方法
- Kubenretes CLIの基本操作
- Kubernetes リソースの操作
- Kubernetes マニュフェストの操作
- Helm Chartを使用したアプリケーションデプロイ
- クラウドサービスなどの外部連携

# ハンズオン構成 
Lab0-6の7つのハンズオンを準備しています。

Lab0ではIKSを利用するために必要なセットアップ手順がかかれています。  
Lab1-6は基本的には独立した内容となっていますので，お好きな順番で実施頂けます。基本的にはLab1から実施頂くことをオススメします。
Lab4-5は続けて実施ください。

0. [Lab 0](Lab0): セットアップ (IKSクラスター接続確認)
1. [Lab 1](Lab1): K8sクラスターへのアプリケーションデプロイ (kubectl CLI の基本操作)
2. [Lab 2](Lab2): スケーリング，アップデート&ロールバック (K8s基本機能)
3. [Lab 3](Lab3): マニフェストファイルの使用とDB連携 (yaml操作とDBコンテナ連携)
4. [Lab 4](Lab4): Helmチャートを使用したアプリケーションデプロイ (K8sパッケージング技術の使用)
5. [Lab 5](Lab5): コンテナアプリケーションとWatson APIとの連携 (クラウドサービスとの連携)
6. [Lab 6](Lab6): Helm Chart基礎の体験 (サンプルチャートの作成・操作)

Lab1-3では，Kubernetesの基礎を学びます。  
具体的には，Kubernetesの泥臭い作業(kubectl run xxx)やマニュフェストファイル(yamlファイル)を使用した宣言的な定義などを経験します。  

Lab4以降では，Kubernetesのパッケージング技術の1つである **Helm** を使用した一括デプロイや，PaaSサービスの1つである IBM Cloudの画像認識サービス(Visual Recognition)と連携させるなど応用編を体験します。
