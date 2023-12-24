---
title: Dockerとvctlコンテナでkind @ Mac
tags:
  - Mac
  - VMwareFusion
  - Docker
  - kubernetes
  - Kind
private: false
updated_at: '2022-05-07T12:14:50+09:00'
id: 3847f0bcf52b286a2b88
organization_url_name: fujitsu
slide: false
ignorePublish: false
---
## kind (kubernetes in docker)
[kind](https://kind.sigs.k8s.io/)は、Dockerコンテナをノードとしたkubernetesクラスタを構築できるツールです。
本記事では、通常どおりDockerコンテナを利用して、kubernetesクラスタを構築するパターンを試した後、Dockerコンテナの代わりに、VMware Fusionのvctlコンテナを利用するパターンを試します。
Dockerコンテナのパターンについては[こちら](https://techblog.gmo-ap.jp/2019/09/19/kind-kubernetes-in-docker/)、vctlコンテナのパターンについては[こちら](https://docs.vmware.com/jp/VMware-Fusion/12/com.vmware.fusion.using.doc/GUID-1CA929BB-93A9-4F1C-A3A8-7A3A171FAC35.html)の記事を参考にさせていただきました。

### 環境
- macOS Catalina 10.15.7
- zsh v5.7.1
- go v1.15.6
- kind v0.5.1
- VMware Fusion v12.1.0


## Dockerコンテナを使うパターン
1. kindを使うには、goのv1.14以上が必要なので、まず、goをインストールします。

    ```
    $ brew install go
    または
    $ brew upgrade go
    ```

1. goのPATHを通すため、以下を~/.zshrcに追記しておきます。

    ```
    export PATH=$PATH:/usr/local/go/bin
    export PATH=$PATH:$(go env GOPATH)/bin
    ```

1. kindをインストールします。

    ```
    $ GO111MODULE="on" go get sigs.k8s.io/kind@v0.5.1
    
    # インストールの確認
    $ kind version
    v0.5.1
    ```

1. シングルノードクラスタを構築してみます。以下のコマンドを実行するだけです。

    ```
    $ kind create cluster
    ```

1. kubectlコマンドを使うには環境変数の設定が必要です。
※デフォルトでkindというクラスタ名になるため、--nameオプションにはkindを指定します。

    ```
    export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
    ```

1. クラスタを削除するには以下のコマンドを実行します。

    ```
    $ kind delete cluster --name kind
    $ unset KUBECONFIG
    ```

1. マルチノードクラスタの構築については、[こちら](https://techblog.gmo-ap.jp/2019/09/19/kind-kubernetes-in-docker/)の記事をご参照、ということで割愛します。同様の手順で構築することができました。

## vctlコンテナを使うパターン
1. 前提条件
構築するクラスタのノード数によって必要な空きメモリが変わります。
1ノードで2GB、2ノードで4GBの空きメモリが必要になる模様。
事前にMacの空きメモリを調べておきましょう。

1. 以下のコマンドで、vctlコンテナランタイムを起動します。

    ```
    $ vctl system start    
    Downloading 3 files...
    Downloading [kubectl 93.95% kind-darwin-amd64 13.11% crx.vmdk 4.31%]
    Finished kubectl 100.00%
    Downloading [kind-darwin-amd64 95.58% crx.vmdk 45.76%]
    Finished kind-darwin-amd64 100.00%
    Downloading [crx.vmdk 97.06%]
    Finished crx.vmdk 100.00%
    3 files successfully downloaded.
    Preparing storage...
    Container storage has been prepared successfully under 
    /Users/****/.vctl/storage
    Launching container runtime...
    Container runtime has been started.
    ```

1. 以下のコマンドで、vctlベースのkindを利用できるようにします。  
`$ kind version`を実行すると、先ほどインストールしたkindとは別のバージョンになっているため、vctlベースのものに切り替わったことが分かります。

    ```
    $ vctl kind
    vctl-based KIND is ready now. KIND will run local Kubernetes clusters 
    by using vctl containers as "nodes"
    * All Docker commands has been aliased to vctl in the current 
    terminal. Docker commands performed in current window would be 
    executed through vctl. If you need to use regular Docker commands, 
    please use a separate terminal window.

    # インストールの確認
    $ kind version
    kind v0.9.0 go1.15.2 darwin/amd64
    ```

1. クラスタの構築
Dockerコンテナの場合と同様に、以下のコマンドでクラスタを構築します。
以下のように何も引数に指定しない場合は、シングルノードクラスタとなります。

    ```
    $ kind create cluster
    ```

構築に失敗する場合は、メモリ不足の可能性があるため、空きメモリを確認してください。
※ノードに割り当てるメモリサイズは、以下のコマンドで変更することができますが、2GBより小さくすることはできません。

```
$ vctl system config --k8s-mem 2g
```


以上で、kindを使ったkubernetesクラスタの構築ができました。ローカル環境に、軽量・高速に（シングル・マルチノードの）クラスタを構築できるため、実環境に寄せたテスト環境などとして利用すると便利かと思いました。
ただ、vctlベースのkindでマルチノードのクラスタを構築する場合には、大容量のメモリ（3ノードでも最低6GBの空きメモリが要求されます）とそれなりのマシンスペックが必要になりそうですので、ご注意ください。

### 追記（2022/5/5）

本記事の執筆当時、筆者のマシンスペック（特にメモリ容量）では、vctlコンテナを用いたマルチノードクラスタの構築は試すことができませんでしたが、当時より高スペックなマシン環境が手に入ったため、マルチノードクラスタの構築も試しました。

#### 環境
- OS: macOS Monterey 12.3.1
- CPU: Intel Core i5-10600
- メモリ: 40GB


#### 構築手順
vctlベースのkindを利用するまでの手順は上記（[vctlコンテナを使うパターン](#vctlコンテナを使うパターン)の3まで）と同様です。

1. 設定ファイルの作成
以下の内容の設定ファイル(multi_k8s.yaml)を作成します。
    ```
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker
    ```
1. クラスタの構築
以下のコマンドでクラスタを構築します。引数に、作成した設定ファイルを指定します。
※--name引数でクラスタ名を設定できますが、設定しなければデフォルトで`kind`という名前になります。
    ```
    $ kind create cluster --config multi_k8s.yaml
    Creating cluster "kind" ...
     ✓ Ensuring node image (kindest/node:v1.19.1) 🖼
     ✓ Preparing nodes 📦 📦 📦 📦  
     ✓ Writing configuration 📜 
     ✓ Starting control-plane 🕹️ 
     ✓ Installing CNI 🔌 
     ✓ Installing StorageClass 💾 
     ✓ Joining worker nodes 🚜 
    Set kubectl context to "kind-kind"
    You can now use your cluster with:
    
    kubectl cluster-info --context kind-kind
    
    Have a question, bug, or feature request? Let us know! 
    https://kind.sigs.k8s.io/#community 🙂
    ```

1. 確認
無事、マルチノードクラスタが構築できました。
    ```
    $ kubectl get nodes
    NAME                 STATUS   ROLES    AGE     VERSION
    kind-control-plane   Ready    master   5m48s   v1.19.1
    kind-worker          Ready    <none>   5m11s   v1.19.1
    kind-worker2         Ready    <none>   5m11s   v1.19.1
    kind-worker3         Ready    <none>   5m11s   v1.19.1
    $ kubectl get pods --all-namespaces
    NAMESPACE            NAME                                                 READY   STATUS    RESTARTS   AGE
    kube-system          coredns-f9fd979d6-6qhbm                      1/1     Running   0          5m45s
    kube-system          coredns-f9fd979d6-kbt67                      1/1     Running   0          5m45s
    kube-system          etcd-kind-control-plane                      1/1     Running   0          5m48s
    kube-system          kindnet-6tqgx                                1/1     Running   0          5m27s
    kube-system          kindnet-7njt7                                1/1     Running   0          5m27s
    kube-system          kindnet-h947r                                1/1     Running   0          5m45s
    kube-system          kindnet-s6m5c                                1/1     Running   0          5m27s
    kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          5m48s
    kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          5m48s
    kube-system          kube-proxy-bjh9m                             1/1     Running   0          5m27s
    kube-system          kube-proxy-dv96k                             1/1     Running   0          5m45s
    kube-system          kube-proxy-np5dc                             1/1     Running   0          5m27s
    kube-system          kube-proxy-zx748                             1/1     Running   0          5m27s
    kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          5m48s
    local-path-storage   local-path-provisioner-78776bfc44-nr5j9      1/1     Running   0          5m45s
    ```

1. クラスタの削除
以下のコマンドで後片付けできます。
※クラスタ構築時に--name引数で名前を付けている場合は、削除時にも同様に--name引数で名前を指定します。
    ```
    $ kind delete cluster
    Deleting cluster "kind" ...
    ```

#### 参考にした記事
https://qiita.com/sourjp/items/281d2189516823950291#simple-clustermaster-node-1-worker-node-2

https://qiita.com/y-vectorfield/items/fd626a048947afa3f4b9
