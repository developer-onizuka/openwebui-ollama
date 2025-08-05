Open WebUI & Ollama

# 1. Goal
Hugging FaceのTransformersライブラリは、**LLM**の学習や推論を行う上で非常に強力なツールです。特に、pipelineメソッドはモデルの推論を簡潔に実行できるため、多くの開発者に利用されています。<br>
しかし、Transformersの実装には、LLMの推論時にいくつかの非効率性が課題として挙げられています。これらの課題は、推論速度やGPUメモリの利用効率に影響を与え、特に大規模なモデルやリアルタイムでの利用においてボトルネックとなることがあります。

**Transformersを使った例**：<br>
> https://github.com/developer-onizuka/MachineLearningOnAWS/blob/main/BERT_FineTuning.ipynb <br>

この課題を解決するために、vLLMやOllamaといったライブラリが登場しました。これらは独自のアルゴリズムを採用し、特にKey-Value (KV) Cacheの割り当てと管理を最適化することで、推論の非効率性を大幅に改善しています。<br><br>
**vLLM**は、PagedAttentionと呼ばれる革新的なアルゴリズムを導入し、TransformerのKV Cacheをページングされたメモリシステムとして扱います。これにより、メモリの断片化（フラグメンテーション）が解消され、メモリの無駄を大幅に削減しつつ、高いスループットを実現しています。<br>
**Ollama**もまた、KV Cacheの効率的な管理を目的としたアルゴリズムを採用しています。これにより、キャッシュのメモリ無駄を大幅に削減するとともに、高速なアクセスと更新を実現し、ローカル環境でのLLM実行をより手軽に行えるようにしています。<br>

### 各推論フレームワークの比較

| 項目 | Hugging Face Transformers | vLLM | Ollama |
|---|---|---|---|
| **主な目的** | LLMの学習・推論・開発の汎用的なフレームワーク。 | 高スループット・低遅延なLLM推論を可能にするサーバー。 | ローカル環境でのLLM実行・管理を簡単にすること。 |
| **KVキャッシュ管理** | 従来の方式（非効率性が課題）。 | **PagedAttention**による効率的なメモリ管理。 | KVキャッシュを最適化し、メモリの無駄を削減。 |
| **パフォーマンス** | 標準的な推論速度。単一の推論には適しているが、高負荷時のスループットは低い。 | 従来の方式と比べて、非常に高いスループットと低遅延を実現。 | ローカル環境では十分な性能を発揮するが、vLLMほどの高スループットは期待できない。 |
| **ハードウェア要件** | GPU、CPUなど幅広い環境で動作。 | NVIDIA GPU (CUDA) に最適化されている。 | GPU（NVIDIA、Apple Siliconなど）、CPUに対応。幅広い環境で動作。 |
| **使いやすさ** | Pythonライブラリとして提供され、柔軟性が高い。`pipeline`メソッドで推論は簡単。 | Pythonライブラリとして提供。OpenAI互換APIも提供され、サーバーとして構築しやすい。 | 非常にシンプルで、コマンド一つでモデルのダウンロードや実行が可能。 |
| **主なユースケース** | LLMの学習、モデル開発、実験、研究。 | 多数のユーザーに対応する本番環境の推論サーバー、リアルタイムアプリケーション。 | 個人でのモデル検証、ローカルでの開発、データプライバシーが重要な用途。 |

<br>
ここでは、Ollamaの推論フレームワークをKubernetes上で動作させ、Open WebUIと連携させることで、生成AIに直感的で使いやすいWeb UIを付加し、手軽に利用できる環境を構築することを目指すものです。

<img src="https://github.com/developer-onizuka/openwebui-ollama/blob/main/user-openwebui-ollama.png" width="800">

# 2. Kunernetes環境について

# 構築イメージ図<br>
<img src="https://github.com/developer-onizuka/openwebui-ollama/blob/main/pictures/2.drawio.png" width="720">

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
# 3-7. ロードバランサーのIPアドレス指定
必要に応じて、ロードバランサーに割り当てるIPアドレスを指定する。
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
# 3-10. PVの展開
```
kubectl apply -f pvc-nfs-ollama.yaml
kubectl apply -f pvc-nfs-openwebui.yaml
```
# 3-11. ollamaの展開
```
kubectl apply -f ollama.yaml
kubectl get pods -A -w -o wide
```
# 3-12. Open WebUIの展開
```
kubectl apply -f openwebui.yaml
kubectl get pods -A -w -o wide
```








