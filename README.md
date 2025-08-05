# Open WebUI & OllamaをKubernetesで動かす

# 0. 必要なもの
メモリ24GB程度のノートPC 1台

# 1. Goal
Hugging FaceのTransformersライブラリは、**LLM**の学習や推論を行う上で非常に強力なツールです。特に、pipelineメソッドはモデルの推論を簡潔に実行できるため、多くの開発者に利用されています。<br>
しかし、Transformersの実装には、LLMの推論時にいくつかの非効率性が課題として挙げられています。これらの課題は、推論速度やGPUメモリの利用効率に影響を与え、特に大規模なモデルやリアルタイムでの利用においてボトルネックとなることがあります。

**Transformersを使った例**：<br>
> https://github.com/developer-onizuka/MachineLearningOnAWS/blob/main/BERT_FineTuning.ipynb <br>

この課題を解決するために、vLLMやOllamaといったライブラリが登場しました。これらは独自のアルゴリズムを採用し、特にKey-Value (KV) Cacheの割り当てと管理を最適化することで、推論の非効率性を大幅に改善しています。<br><br>
**vLLM**は、PagedAttentionと呼ばれる革新的なアルゴリズムを導入し、TransformerのKV Cacheをページングされたメモリシステムとして扱います。これにより、メモリの断片化（フラグメンテーション）が解消され、メモリの無駄を大幅に削減しつつ、高いスループットを実現しています。<br><br>
**Ollama**もまた、KV Cacheの効率的な管理を目的としたアルゴリズムを採用しています。これにより、キャッシュのメモリ無駄を大幅に削減するとともに、高速なアクセスと更新を実現し、ローカル環境でのLLM実行をより手軽に行えるようにしています。<br>

例：
```
$ ollama run llama3.2
>>> Hello! Who are you?
Hello. My name is ...
```

### 各推論フレームワークの比較

| 項目 | Hugging Face Transformers | vLLM | Ollama |
|---|---|---|---|
| **主な目的** | LLMの学習・推論・開発の汎用的なフレームワーク | 高スループット・低遅延なLLM推論を可能にするサーバー | ローカル環境でのLLM実行・管理を簡単にすること |
| **KVキャッシュ管理** | 従来の方式（非効率性が課題） | **PagedAttention**による効率的なメモリ管理 | KVキャッシュを最適化し、メモリの無駄を削減 |
| **パフォーマンス** | 標準的な推論速度。単一の推論には適しているが、高負荷時のスループットは低い。 | 従来の方式と比べて、非常に高いスループットと低遅延を実現 | ローカル環境では十分な性能を発揮するが、vLLMほどの高スループットは期待できない。 |
| **ハードウェア要件** | GPU、CPUなど幅広い環境で動作 | NVIDIA GPU (CUDA) に最適化されている。 | GPU（NVIDIA、Apple Siliconなど）、CPUに対応。幅広い環境で動作 |
| **使いやすさ** | Pythonライブラリとして提供され、柔軟性が高い。`pipeline`メソッドで推論は簡単 | Pythonライブラリとして提供。OpenAI互換APIも提供され、サーバーとして構築しやすい。 | 非常にシンプルで、コマンド一つでモデルのダウンロードや実行が可能 |
| **主なユースケース** | LLMの学習、モデル開発、実験、研究 | 多数のユーザーに対応する本番環境の推論サーバー、リアルタイムアプリケーション | 個人でのモデル検証、ローカルでの開発、データプライバシーが重要な用途 |

<br>
ここでは、Ollamaの推論フレームワークをKubernetes上で動作させ、Open WebUIと連携させることで直感的で使いやすいWeb UIを付加し、手軽に利用できる環境を構築することを目指します。

<img src="https://github.com/developer-onizuka/openwebui-ollama/blob/main/user-openwebui-ollama.png" width="800">

# 2. 構築イメージ

<img src="https://github.com/developer-onizuka/openwebui-ollama/blob/main/pictures/2.drawio.png" width="720">

ここでは、手元のマシンリソースの関係で、master node 1台、worker node 1台でKubernetesクラスタを作成します。<br>

# 各ノードのスペック
| Node名 | CPU | Memory | IP Address |
|---|---|---|---|
| master | 4 | 8GB | 192.168.33.100 |
| worker1 | 4 | 8GB | 192.168.33.101 |
| nfs | 1 | 1GB | 192.168.33.11 |

# 3. 手順
# 3-1. Hypervisorのインストール
>https://www.oracle.com/jp/virtualization/technologies/vm/downloads/virtualbox-downloads.html

# 3-2. Vagrantのインストール
>https://developer.hashicorp.com/vagrant/install

# 3-3. gitのインストール & git clone
>https://git-scm.com/downloads
```
git clone https://github.com/developer-onizuka/openwebui-ollama
cd openwebui-ollama
```

# 3-4. 仮想マシンの展開
# 3-4-1. Master node / Worker node
```
cd kubernetes
vagrant up
cd ..
```
# 3-4-2. NFSサーバー
```
cd nfs
vagrant up
cd ..
```
# 3-5. Master nodeへのログイン & git clone
```
cd kubernetes
vagrant ssh master
git clone https://github.com/developer-onizuka/openwebui-ollama
cd openwebui-ollama
```
# 3-6. Kubernetesクラスタの確認
```
kubectl get nodes -A -o wide
kubectl get pods -A -o wide
```
```
$ kubectl get nodes -A -o wide
NAME      STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
master    Ready    control-plane   67m   v1.33.3   192.168.33.100   <none>        Ubuntu 24.04.2 LTS   6.8.0-53-generic   containerd://1.7.27
worker1   Ready    node            67m   v1.33.3   192.168.33.101   <none>        Ubuntu 24.04.2 LTS   6.8.0-53-generic   containerd://1.7.27
```
```
$ kubectl get pods -A -o wide
NAMESPACE        NAME                                       READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
kube-system      calico-kube-controllers-7498b9bb4c-lngsr   1/1     Running   0          67m   10.10.219.66     master    <none>           <none>
kube-system      calico-node-4wbbs                          1/1     Running   0          67m   192.168.33.101   worker1   <none>           <none>
kube-system      calico-node-8bt9k                          1/1     Running   0          67m   192.168.33.100   master    <none>           <none>
kube-system      coredns-674b8bbfcf-5strr                   1/1     Running   0          67m   10.10.219.67     master    <none>           <none>
kube-system      coredns-674b8bbfcf-kqn54                   1/1     Running   0          67m   10.10.219.65     master    <none>           <none>
kube-system      csi-nfs-controller-8fdc6755d-78qxc         5/5     Running   0          46m   192.168.33.101   worker1   <none>           <none>
kube-system      csi-nfs-node-kjqnr                         3/3     Running   0          46m   192.168.33.100   master    <none>           <none>
kube-system      csi-nfs-node-x2g8q                         3/3     Running   0          46m   192.168.33.101   worker1   <none>           <none>
kube-system      etcd-master                                1/1     Running   0          67m   192.168.33.100   master    <none>           <none>
kube-system      kube-apiserver-master                      1/1     Running   0          67m   192.168.33.100   master    <none>           <none>
kube-system      kube-controller-manager-master             1/1     Running   0          67m   192.168.33.100   master    <none>           <none>
kube-system      kube-proxy-2xdcj                           1/1     Running   0          67m   192.168.33.101   worker1   <none>           <none>
kube-system      kube-proxy-slq7w                           1/1     Running   0          67m   192.168.33.100   master    <none>           <none>
kube-system      kube-scheduler-master                      1/1     Running   0          67m   192.168.33.100   master    <none>           <none>
metallb-system   controller-58fdf44d87-66bfg                1/1     Running   0          67m   10.10.235.129    worker1   <none>           <none>
metallb-system   speaker-ldcz4                              1/1     Running   0          67m   192.168.33.100   master    <none>           <none>
metallb-system   speaker-v8vn6                              1/1     Running   0          67m   192.168.33.101   worker1   <none>           <none>
```
# 3-7. ロードバランサーのIPアドレス指定
ロードバランサーに割り当てるIPアドレスの範囲を指定します。
```
kubectl apply -f metallb-ipaddress.yaml
```
# 3-8. NFS用のCSIドライバの展開
```
./install-csi-driver.sh
```
# 3-9. StorageClassの展開
```
kubectl apply -f storageclass-vm-nfs.yaml
```
```
$ kubectl get sc
NAME                   PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-vm-csi (default)   nfs.csi.k8s.io   Delete          Immediate           false                  45m
```
# 3-10. PVの展開
```
kubectl apply -f pvc-nfs-ollama.yaml
kubectl apply -f pvc-nfs-openwebui.yaml
```
```
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-00400433-35b7-45ff-8ebd-9a485321f01c   20Gi       RWX            Delete           Bound    default/pvc-nfs-openwebui   nfs-vm-csi     <unset>                          25m
pvc-15ca215d-dfc0-4835-990c-55437eea6672   20Gi       RWX            Delete           Bound    default/pvc-nfs-ollama      nfs-vm-csi     <unset> 
```
# 3-11. Ollama & Open WebUIの展開
```
kubectl apply -f ollama.yaml
kubectl apply -f openwebui.yaml
```
```
$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP              NODE      NOMINATED NODE   READINESS GATES
ollama-6c988c64c6-jrsn4       1/1     Running   0          25m   10.10.235.135   worker1   <none>           <none>
ollama-6c988c64c6-xqvng       1/1     Running   0          25m   10.10.235.136   worker1   <none>           <none>
open-webui-8556f87cbc-5rtzz   1/1     Running   0          25m   10.10.235.137   worker1   <none> 
```

# 3-12. Serviceの確認
```
kubectl get services
```
```
$ kubectl get services
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
kubernetes       ClusterIP      10.96.0.1       <none>         443/TCP          85m
svc-ollama       ClusterIP      10.103.87.228   <none>         11434/TCP        41m
svc-open-webui   LoadBalancer   10.96.28.106    192.168.33.2   8080:32220/TCP   41m
```

# 4. Open WebUI
任意のブラウザで、**192.168.33.2:8080**にアクセスします。<br><br>
<img src="https://github.com/developer-onizuka/openwebui-ollama/blob/main/openwebui-llm.png" width="880">

モデルを検索し、ダウンロードすることでローカルでLLMがブラウザ上で扱えるようになります。<br>

例：hf.co/bartowski/Llama-3.2-1B-Instruct-GGUF:latest<br><br>

外部のNFSにモデルを保存しているため、クラスタやPodを再起動しても、ダウンロードしたモデルは残っています。<br>


# 5. クリーンナップ
```
vagrant destroy -f
```
```
$ cd openwebui-ollama/kubernetes
$ vagrant destroy -f
==> master: Forcing shutdown of VM...
==> master: Destroying VM and associated drives...
==> worker1: Forcing shutdown of VM...
==> worker1: Destroying VM and associated drives...
```
```
$ cd openwebui-ollama/nfs
$ vagrant destroy -f
==> nfs: Forcing shutdown of VM...
==> nfs: Destroying VM and associated drives...
```

