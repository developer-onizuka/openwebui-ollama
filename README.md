Open WebUI & Ollama

# 1. Goal
> https://github.com/developer-onizuka/MachineLearningOnAWS/blob/main/BERT_FineTuning.ipynb

Hugging FaceのTransformersライブラリは、**LLM（大規模言語モデル）**の学習や推論を行う上で非常に強力なツールです。特に、pipelineメソッドはモデルの推論を簡潔に実行できるため、多くの開発者に利用されています。<br>
しかし、Transformersの実装には、LLMの推論時にいくつかの非効率性が課題として挙げられています。これらの課題は、推論速度やGPUメモリの利用効率に影響を与え、特に大規模なモデルやリアルタイムでの利用においてボトルネックとなることがあります。

この課題を解決するために、vLLMやOllamaといったライブラリが登場しました。これらは独自のアルゴリズムを採用し、特にKey-Value (KV) Cacheの割り当てと管理を最適化することで、推論の非効率性を大幅に改善しています。<br>
vLLMはPagedAttentionと呼ばれる革新的なアルゴリズムを導入し、TransformerのKV Cacheをページングされたメモリシステムとして扱います。これにより、メモリの断片化（フラグメンテーション）が解消され、メモリの無駄を大幅に削減しつつ、高いスループットを実現しています。<br>
Ollamaもまた、KV Cacheの効率的な管理を目的としたアルゴリズムを採用しています。これにより、キャッシュのメモリ無駄を大幅に削減するとともに、高速なアクセスと更新を実現し、ローカル環境でのLLM実行をより手軽に行えるようにしています。<br>

