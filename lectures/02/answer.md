# 第2回 解答

## 演習準備
ノートブックを自分のドライブにコピー
ランタイムを　TeslaT4に変更
0[1]
実行後
　name, memory.total [MiB]
　Tesla T4, 15360 MiB
表示を確認
0[2]
実行後※前2行は赤文字エラー（今回は関係なし）
　ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
　cudf-cu13 26.6.0 requires pyarrow<24,>=19.0.0, but you have pyarrow 26.0.0 which is incompatible.
　{'transformers': '5.18.0', 'peft': '0.21.1', 'trl': '1.14.2', 'accelerate': '1.15.0', 'datasets': '5.1.0', 'torch': '2.11.0+cu130'}
表示を確認

1. 設定
{
  "base_model": "Qwen/Qwen3.5-0.8B",
  "revision": "2fc06364715b967f1860aea9cf38778875588b17",
  "seed": 42,
  "num_train_epochs": 20,
  "warmup_steps": 6,
  "lr": 0.0002,
  "lora_r": 16,
  "lora_alpha": 32,
  "max_seq_len": 512,
  "max_new_tokens": 96,
  "gen": {
    "do_sample": false
  },
  "work_dir": "/content/qlora_lab",
  "drive_dir": "/content/drive/MyDrive/aie2026/qlora_lab"
}

 設定の JSON 表示を確認。

 2. 架空 FAQ と教師データの生成

train=14 valid=7 test=21 gold=21
学習例の 1 件 | 質問: 課題の提出期限はいつですか | 望ましい出力: {"answer": "課題の提出期限は 9 月 30 日 23:59（日本時間）です。", "needs_review": false}

3. モデルの読み込みと学習前評価
モデルのダウンロードを完了確認
表形式による出力を確認

4.LoRA 学習を実施
[transformers] The tokenizer has new PAD/BOS/EOS tokens that differ from the model config and generation config. The model config and generation config were aligned accordingly, being updated with the tokenizer's values. Updated tokens: {'eos_token_id': 248046}.
trainable params: 6,389,760 || all params: 758,782,784 || trainable%: 0.8421
 [40/40 03:43, Epoch 20/20]
Step	Training Loss
10	1.561403
20	0.084290
30	0.018470
40	0.013900
学習時間: 230 秒 | ステップ数: 40 | エポック: 20.0

学習完了。

5. 学習後評価（学習前と同じ test・同じ生成設定）
{"label": "after", "n": 21, "format_rate": 1.0, "content_rate": 0.9047619047619048, "both_rate": 0.9047619047619048, "seconds": 43.9}
n	format_rate	content_rate	both_rate	seconds
label					
before	21	0.095238	0.000000	0.000000	23.8
after	21	1.000000	0.904762	0.904762	43.9
退行: []

学習評価の変化、対抗リストが空となることを確認。
形式と文面は揃ったが、「FAQ に根拠がない」回答が混在している

例）10	オフィスアワーの曜日と時間を教えてください
→　{"answer": "質問は毎週火曜 17:00〜18:00 のオフィスアワーで受け付けます。", "needs_review": false}	
質問回答の意味的にはあっているが、オフィスアワーそのものを問い合わせる内容に関しての回答にはなっていない。

6. 知識を問う質問（形式を学んでも知識は保証されない）
    "この講座の最終課題の締切はいつですか",       # FAQ に無い（架空）
→{"answer": "課題の提出期限は 9 月 30 日 23:59（日本時間）です。", "needs_review": false}
最終課題（おそらく第13回の課題を想定してる架空質問）に対して、与えられた課題締切（9月30日）日付を回答」


## 演習

- 使ったモデル・設定（`result.json` の `config` から）：
- 自分で学習を実行したか（`executed_training_myself`）：はい / いいえ（実行できた範囲：　）
- 学習前：`format_rate` =　 / `content_rate` =　
- 学習後：`format_rate` =　 / `content_rate` =　
- 退行した質問：
- 学習後に残った失敗と、その型（根拠なしの質問に断定した / 根拠ありの質問に確認要求で逃げた / 形式の崩れ）：
- 知識を問う質問（セクション 6）の結果（学習前の分類 → 学習後の分類）と気づいたこと：
- 学習で変わったこと・変わらなかったこと：

## 宿題

- 取り組んだレベル：基礎 / 実装 / 発展
- 変えた設定やデータ：
- 結果（比較表や数字）：
- ファインチューニングが向く場面・向かない場面についての考察：
