# Windows環境でのインストール

## Minikubeのセットアップ

```bash
sudo service docker start
minikube config set cpus 4
minikube config set memory 4096
minikube start
minikube addons enable ingress
```

## Tektonのセットアップ

```bash
# core component
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
# triggers
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
# dashboard
kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml
# 確認
kubectl get pods --namespace tekton-pipelines
# dashboardをポートフォワーディング(minikube start --vm-driver=dockerで起動したとき、Ingressを使うためにはminikube tunnelが必要で面倒なため簡単な方法をとる)
kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 9097:9097
```

http://localhost:9097 ヘアクセスでダッシュボードの起動を確認する

### `tkn`コマンドのセットアップ

- https://tekton.dev/docs/getting-started/

## サンプル

```bash
# タスクとパイプラインを作成
kubectl apply -f task-hello.yaml
kubectl apply -f task-goodbye.yaml
kubectl apply -f pipeline-hello-goodbye.yaml
# パイプラインを実行
tkn pipeline start hello-goodbye
# 結果を確認
tkn pipelinerun logs --last -f
```

```bash
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.4/git-clone.yaml
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/maven/0.2/maven.yaml
kubectl apply -f maven-pvc.yaml
kubectl apply -f maven-example.yaml
```

```bash
$ kubectl get pv pvc-715cbea7-7caa-4602-9a48-26934275ada0 -n tekton-pipelines -o yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    hostPathProvisionerIdentity: 617c1fe6-d074-4518-b225-7a57892a36be
    pv.kubernetes.io/provisioned-by: k8s.io/minikube-hostpath
  creationTimestamp: "2021-11-04T17:50:38Z"
  finalizers:
  - kubernetes.io/pv-protection
  name: pvc-715cbea7-7caa-4602-9a48-26934275ada0
  resourceVersion: "26025"
  uid: 47769245-0897-4f5e-8f00-9ffe9d9357ec
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 500Mi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: maven-source-pvc
    namespace: default
    resourceVersion: "26018"
    uid: 715cbea7-7caa-4602-9a48-26934275ada0
  hostPath:
    path: /tmp/hostpath-provisioner/default/maven-source-pvc
    type: ""
  persistentVolumeReclaimPolicy: Delete
  storageClassName: standard
  volumeMode: Filesystem
status:
  phase: Bound
```

`/tmp/hostpath-provisioner/default/maven-source-pvc`がworkspaceになっている。
