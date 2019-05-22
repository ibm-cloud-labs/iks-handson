# Lab6) サンプル・チャートでHelmを理解する

Lab6では、Helmの理解のために、サンプルのチャートを作ってKubernetesにデプロイします。
チャートの構造を理解することで、提供されるチャートをただ使うのではなく、理解した上で利用できるようになります。

## チャートを作るための参考資料
Helmの公式サイトにチャート開発のためのドキュメントがまとめられています。

- https://docs.helm.sh/developing_charts/
- https://docs.helm.sh/chart_template_guide/
- https://docs.helm.sh/chart_best_practices/

## チャートの作成
チャートの雛形を作成してみます。任意の作業ディレクトリで以下のコマンドを実行してください。

  ```bash
  任意のディレクトリでhelm createコマンドを実行します。
  $ helm create mychart
  Creating mychart
  ```
   
できあがるディレクトリの構造は以下の通りです:
   
  ```bash
  mychart
  ├── Chart.yaml                     # チャートの情報を含むyaml
  ├── charts                         # このチャートが依存するチャートを格納するディレクトリー
  ├── templates                      # マニフェストのテンプレートを格納するディレクトリー
  │   ├── NOTES.txt                  # OPTIONAL: チャートの使用方法を記載したプレーンテキスト
  │   ├── _helpers.tpl               # 
  │   ├── deployment.yaml            # deployment作成用のyaml
  │   ├── ingress.yaml               # Ingress設定用のyaml
  │   ├── service.yaml               # サービス作成用のyaml
  │   └── tests
  │       └── test-connection.yaml
  └── values.yaml                    # このチャートのデフォルト値を記載したyaml
  ```

## deployment.ymlを紐解く
作成されたtemplates/deployment.ymlをみてみましょう。
Go Template言語で環境により異なる値が記載されています

  ```bash
  $ cat mychart/templates/deployment.yaml 
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ include "mychart.fullname" . }}
    labels:
  {{ include "mychart.labels" . | indent 4 }}
  spec:
    replicas: {{ .Values.replicaCount }}
    selector:
      matchLabels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    template:
      metadata:
        labels:
          app.kubernetes.io/name: {{ include "mychart.name" . }}
          app.kubernetes.io/instance: {{ .Release.Name }}
      spec:
      {{- with .Values.imagePullSecrets }}
        imagePullSecrets:
          {{- toYaml . | nindent 8 }}
      {{- end }}
        containers:
          - name: {{ .Chart.Name }}
            image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
  :
  (以下省略)
  ```

{{ .Values.<変数名> }}となっている部分はvalues.yamlにあるデフォルト値が埋め込まれます。
以下の設定の場合、例えばvalues.yamlにあるreplicaCountという設定項目が上記のdeployment.ymlのレプリカ数を指定する項目(spec.replicas)に反映されます。

  ```
  $ cat mychart/values.yaml 
  # Default values for mychart.
  # This is a YAML-formatted file.
  # Declare variables to be passed into your templates.

  replicaCount: 1

  image:
    repository: nginx
    tag: stable
    pullPolicy: IfNotPresent

  imagePullSecrets: []
  nameOverride: ""
  fullnameOverride: ""

  service:
    type: ClusterIP
    port: 80
  :
  (以下省略)
  ```

## サンプル・チャートを利用する
まずはこのままサンプルを利用してデプロイしてみましょう。
「helm install <任意の名前> ＜チャート・ディレクトリー＞」を実行します。
以下のような結果が出力されることを確認します。

  ```bash
  $ helm install --name sample ./mychart
  NAME:   sample
  LAST DEPLOYED: Wed May 22 19:36:51 2019
  NAMESPACE: default
  STATUS: DEPLOYED

  RESOURCES:
  ==> v1/Deployment
  NAME            READY  UP-TO-DATE  AVAILABLE  AGE
  sample-mychart  0/1    1           0          0s

  ==> v1/Pod(related)
  NAME                             READY  STATUS   RESTARTS  AGE
  sample-mychart-5d8794cdb6-qjzsb  0/1    Pending  0         0s

  ==> v1/Service
  NAME            TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)  AGE
  sample-mychart  ClusterIP  172.21.106.142  <none>       80/TCP   0s


  NOTES:
  1. Get the application URL by running these commands:
    export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=mychart,app.kubernetes.io/instance=sample" -o jsonpath="{.items[0].metadata.name}")
    echo "Visit http://127.0.0.1:8080 to use your application"
    kubectl port-forward $POD_NAME 8080:80
  ```

問題なくデプロイができたかは、以下のコマンドで確認します:

  ```bash
  $ helm ls
  NAME  	REVISION	UPDATED                 	STATUS  	CHART        	APP VERSION	NAMESPACE
  sample	1       	Wed May 22 19:36:51 2019	DEPLOYED	mychart-0.1.0	1.0        	default
  ```

  ```bash
  $ kubectl get pod
  NAME                              READY   STATUS    RESTARTS   AGE
  sample-mychart-5d8794cdb6-qjzsb   1/1     Running   0          73s
  ```

実際にアプリケーションにアクセスするために、「kubectl port-forward <Pod名> <任意のポート番号>:80」でポートフォワーディングします。

   ```bash
   $ kubectl port-forward sample-mychart-5d8794cdb6-qjzsb 8080:80
   Forwarding from 127.0.0.1:8080 -> 80
   Forwarding from [::1]:8080 -> 80
   ```

この状態で、Webブラウザから「http://localhost:8080」でアクセスすればサンプルのWebページが表示されます。

## 設定を変更する
では、次にIKSのフリークラスターに合わせ、KubernetesのNodePortで公開できるように、テンプレートを修正してみましょう。
templates/service.yamlの17行目から３行追加します。

  ```bash
  $ cat mychart/templates/service.yaml 
  apiVersion: v1
  kind: Service
  metadata:
    name: {{ include "mychart.fullname" . }}
    labels:
  {{ include "mychart.labels" . | indent 4 }}
  spec:
    type: {{ .Values.service.type }}
    ports:
      - port: {{ .Values.service.port }}
        targetPort: http
        protocol: TCP
        name: http
        {{- if .Values.service.nodePort }}
        nodePort: {{ .Values.service.nodePort }}
        {{- end}}
    selector:
      app.kubernetes.io/name: {{ include "mychart.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  ```
追加する行は以下の設定です。

  ```bash
        {{- if .Values.service.nodePort }}
        nodePort: {{ .Values.service.nodePort }}
        {{- end}}
  ```

変更したらテンプレートの記載が正しいかのチェックを行います。「helm lint <helmチャートのディレクトリ>」を実行します。

  ```bash
  $ helm lint ./mychart/
  ==> Linting ./mychart/
  [INFO] Chart.yaml: icon is recommended

  1 chart(s) linted, no failures
  ```
   
次に設定した値を変更していきましょう。
デフォルト値が定義されているvalue.yamlをコピーします。

  ```bash
  $ cp -p mychart/values.yaml value-new.yaml
  ```

コピーしたファイル(value-new.yaml)を開き、以下のようにserviceの項目にあるtypeの設定を修正、そしてnodePortの項目を追加します。

  * 変更前
    ```
    service:
      type: ClusterIP
      port: 80
    ```

  * 変更後   
    ```
    service:
      type: NodePort
      port: 80
      nodePort: 30001
    ```

上記の設定後、helm upgradeコマンドでhelmリリースを更新します。
先ほどと同じように処理が実行されれば問題なく実行できています。

  ```bash
  $ helm upgrade -f value-new.yaml sample ./mychart/
  Release "sample" has been upgraded.
  LAST DEPLOYED: Wed May 22 19:44:46 2019
  NAMESPACE: default
  STATUS: DEPLOYED

  RESOURCES:
  ==> v1/Deployment
  NAME            READY  UP-TO-DATE  AVAILABLE  AGE
  sample-mychart  1/1    1           1          7m55s

  ==> v1/Pod(related)
  NAME                             READY  STATUS   RESTARTS  AGE
  sample-mychart-5d8794cdb6-qjzsb  1/1    Running  0         7m55s

  ==> v1/Service
  NAME            TYPE      CLUSTER-IP      EXTERNAL-IP  PORT(S)       AGE
  sample-mychart  NodePort  172.21.106.142  <none>       80:30001/TCP  7m55s


  NOTES:
  1. Get the application URL by running these commands:
    export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services sample-mychart)
    export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
    echo http://$NODE_IP:$NODE_PORT
  ```
  
今度は実際にNodePortでアクセスしてみましょう。「ibmcloud ks workers <クラスター名>」を実行し、パブリックIPアドレスを確認します。
確認したあとで「http://<パブリックIPアドレス>:30001」でアクセスすれば、再びサンプルのアプリケーションにアクセスできます。

  ```bash
  $ ibmcloud ks workers mycluster
  OK
  ID                         パブリック IP     プライベート IP   マシン・タイプ   状態     状況    ゾーン   バージョン   
  kube-hou02-xxxxxxxxxx-w1   184.xxx.x.xx    10.76.194.59    free             normal   Ready   hou02    1.10.12_1543 
  ```
 
## リソースを追加する
新しくConfig Mapを作成し、アプリケーションに反映させてみましょう。
今回の例ではConfig Mapとしてnginxのindex.htmlのテンプレートを登録し、表示されるメッセージをhelmチャートのValueで変更できるようにします。

まずは新しいhelmのvalueファイル (value-new.yaml)を開き、以下のようにapp.nameの設定を追加します。

  ```bash
  # value-new.yamlに設定追加 (serviceの項目の上にapp.nameを追加)

  app:                   # この２行を追加します
    name: IKS-san        # この２行を追加します

  replicaCount: 1

  image:
    repository: nginx
    tag: stable
    pullPolicy: IfNotPresent
  ```

続いて、チャートのtemplatesディレクトリにindex-configmap.yamlを作成します。21行目がWebブラウザで確認できるメッセージの部分です。

   ```bash
   # 以下 mychart/templates/index-configmap.yamlの内容
   
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: index-config 
   data:
     index-config: index.html 
     index.html: |
       <!DOCTYPE html>
       <html>
       <head>
       <title>Welcome to nginx!</title>
       <style>
           body {
               width: 35em;
               margin: 0 auto;
               font-family: Tahoma, Verdana, Arial, sans-serif;
           }
       </style>
       </head>
       <body>
       <h1>Welcome to __NAME__</h1>
       <p>If you see this page, the nginx web server is successfully installed and working. Further configuration is required.</p>

       <p>For online documentation and support please refer to
       <a href="http://nginx.org/">nginx.org</a>.<br/>
       Commercial support is available at
       <a href="http://nginx.com/">nginx.com</a>.</p>

       <p><em>Thank you for using nginx.</em></p>
       </body>
       </html>
   ```

さらに、チャートのtemplatesディレクトリにあるdeployment.yamlを編集します。

  ```bash
  # 以下 mychart/templates/deployment.yamlの内容
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ include "mychart.fullname" . }}
    labels:
  {{ include "mychart.labels" . | indent 4 }}
  spec:
    replicas: {{ .Values.replicaCount }}
    selector:
      matchLabels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    template:
      metadata:
        labels:
          app.kubernetes.io/name: {{ include "mychart.name" . }}
          app.kubernetes.io/instance: {{ .Release.Name }}
      spec:
      {{- with .Values.imagePullSecrets }}
        imagePullSecrets:
          {{- toYaml . | nindent 8 }}
      {{- end }}
        volumes:
        - name: index-config
          configMap:
            name: index-config
        - name: config-volume
          emptyDir: {}
        initContainers:
        - name: init-myservice
          image: busybox
          command: ['sh', '-c', 'cat /etc/config-template/index.html | sed "s/__NAME__/{{ .Values.app.name }}/" > /etc/config/index.html']
          volumeMounts:
          - name: config-volume
            mountPath: /etc/config
          - name: index-config
            mountPath: /etc/config-template/index.html
            readOnly: true
            subPath: index.html
        containers:
          - name: {{ .Chart.Name }}
            image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
            imagePullPolicy: {{ .Values.image.pullPolicy }}
            volumeMounts:
            - name: config-volume
              mountPath: /usr/share/nginx/html/
            ports:
              - name: http
                containerPort: 80
                protocol: TCP
            livenessProbe:
              httpGet:
                path: /
                port: http
            readinessProbe:
              httpGet:
                path: /
                port: http
            resources:
              {{- toYaml .Values.resources | nindent 12 }}
        {{- with .Values.nodeSelector }}
        nodeSelector:
          {{- toYaml . | nindent 8 }}
        {{- end }}
      {{- with .Values.affinity }}
        affinity:
          {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
        tolerations:
          {{- toYaml . | nindent 8 }}
      {{- end }}
  ```

完了したら再びhelm upgradeで更新します。

   ```bash
   $ helm upgrade -f value-new.yaml sample ./mychart/
   ```

あとはWebブラウザでアクセスし、画面の結果を確認します。
メッセージがhelmのvalueファイル (value-new.yaml)に指定した文字に変わっていれば問題なく動いていることが確認できます。

## お片付け

```bash
1) helmで作成したリリースを削除します
$ helm delete sample --purge

2) ハンズオンが終わったらクラスターを削除します
$ ibmcloud ks cluster-rm <クラスター名>
```
