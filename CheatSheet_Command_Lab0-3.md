
# コマンドチートシート
(2020/09/23 IBM Dojoワークショップ用)

## Lab 0.セットアップ
```
ibmcloud login -a https://cloud.ibm.com
```

リージョンを選択します (または Enter キーを押してスキップします):
1. au-syd
2. in-che
3. jp-tok
4. kr-seo
5. eu-de
6. eu-gb
7. us-south
8. us-east  
```
数値を入力してください> *7* → us-southを選択する
```

ibmcloud target -g <リソースグループ名>
```
ibmcloud target -g Default
```
```
kubectl get nodes
```

## Lab 1
```
kubectl run guestbook --image=ibmcom/guestbook:v1
```
```
kubectl get pods
```
```
kubectl get all
```
```
kubectl expose deployment guestbook --type="NodePort" --port=3000
```
```
kubectl get service guestbook
```
ibmcloud ks worker ls --cluster <クラスタ名>
```
ibmcloud ks worker ls --cluster mycluster-free
```
Lab2終了後に作成したIKS上のリソースの削除
```
kubectl delete deployment guestbook
```
```
kubectl delete service guestbook
```
```
kubectl get pods
```
## Lab 2
```
kubectl run guestbook --image=ibmcom/guestbook:v1
```
```
kubectl expose deployment guestbook --type="NodePort" --port=3000
```
```
kubectl scale --replicas=10 deployment guestbook
```
```
kubectl rollout status deployment guestbook
```
```
kubectl get pods
```
```
kubectl set image deployment/guestbook guestbook=ibmcom/guestbook:v2
```
```
kubectl rollout status deployment/guestbook
```
```
ibmcloud ks workers mycluster-free
```
```
kubectl get service guestbook
```
```
kubectl rollout undo deployment guestbook
```
```
kubectl get replicasets -l run=guestbook
```
kubectl describe replicasets <ご自身のコンテナイメージ名>  
```
kubectl describe replicasets xxxxxxxx
```
Lab2終了後に作成したIKS上のリソースの削除
```
kubectl delete deployment guestbook
```
```
kubectl delete service guestbook
```
```
kubectl get pods
```
## Lab 3
```
git clone https://github.com/cloud-handson/guestbook.git
```
```
cd guestbook/v1
```
```
cat guestbook-deployment.yaml
```
```
kubectl create -f guestbook-deployment.yaml
```
```
kubectl get pods -l app=guestbook
```
viで編集 (IBM Cloud Shell利用の場合)
```
vi guestbook-deployment.yaml
```
編集後
[Esc]キーを押し、:wq!を押してファイル保存する
```
kubectl apply -f guestbook-deployment.yaml
```
```
kubectl get pods -l app=guestbook
```
```
cat guestbook-service.yaml
```
```
kubectl create -f guestbook-service.yaml
```
```
ibmcloud ks workers mycluster-free
```
```
kubectl get service guestbook
```
意図的にPodを削除する
kubectl delete pod <ご自身のPod名>
```
kubectl delete pod guestbook-v1-xxxxxx
```
```
kubectl get pods -l app=guestbook
```
```
cat redis-master-deployment.yaml
```
```
kubectl create -f redis-master-deployment.yaml
```
```
kubectl get pods -l app=redis,role=master
```
kubectl exec -it <ご自身のPod名> redis-cli
```
kubectl exec -it redis-master-xxxxxx redis-cli
```
```
cat redis-master-service.yaml
```
```
kubectl create -f redis-master-service.yaml
```
```
kubectl delete deployment guestbook-v1
```
```
kubectl create -f guestbook-deployment.yaml
```
```
ibmcloud ks workers mycluster-free
```
```
kubectl get service guestbook
```
```
cat redis-slave-deployment.yaml
```
```
kubectl create -f redis-slave-deployment.yaml
```
```
kubectl get pods -l app=redis,role=slave
```
```
cat redis-slave-service.yaml 
```
```
kubectl create -f redis-slave-service.yaml
```
```
kubectl delete deploy guestbook-v1
```
```
kubectl create -f guestbook-deployment.yaml
```
Lab3終了後に作成したIKS上のリソースの削除
```
kubectl delete -f guestbook-deployment.yaml
```
```
kubectl delete -f guestbook-service.yaml
```
```
kubectl delete -f redis-slave-service.yaml
```
```
kubectl delete -f redis-slave-deployment.yaml
```
```
kubectl delete -f redis-master-service.yaml
```
```
kubectl delete -f redis-master-deployment.yaml
```
以上です。
