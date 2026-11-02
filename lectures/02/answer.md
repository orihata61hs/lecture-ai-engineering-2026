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

7. 比較表と提出
result.json、eval_before.json、eval_after.json　をダウンロード

内容について比較、評価を実施。
以下レポートです。

---
AI学習効果検証レポート
1. 概要

FAQ応答タスクにおいて、学習前後の回答品質を比較し、学習データ適用による改善効果および残存課題を評価した。評価対象は、FAQに含まれる質問とFAQ外質問を含む計21件のテストケースである。評価指標として、出力フォーマットの適合性（Format）および回答内容の妥当性（Content）を用いた。 result、eval_before、eval_after を基に分析を実施した

2. 評価結果サマリー

学習後はフォーマット遵守率が大幅に改善し、FAQ外質問に対して不用意に回答せず確認を促す挙動も大きく改善した。

項目	学習前	学習後Format成功率	9.5%	100%
Content成功率	0.0%	90.5%
両方成功率	0.0%	90.5%

学習前はほぼ全件でフォーマット不備が発生していたが、学習後は全件で所定形式を満たした。また、FAQに存在しない質問に対して推測で回答する傾向が大幅に低減し、多くのケースで「確認が必要」と判断できるようになった。

3. 学習前の主な失敗傾向
3.1 フォーマット違反

最も多かった失敗は、出力形式の不備である。
想定フォーマットは以下の形式であった。
{
  "answer": "...",
  "needs_review": true
}
しかし実際には needs_review が欠落しているケースが多数確認された。これにより、回答内容の妥当性以前に評価基準を満たせていなかった。

3.2 FAQ外質問への断定回答

FAQに情報が存在しないにもかかわらず、モデルが推測によって回答しているケースが複数確認された。
「講義の録画は公開されますか」
「課題の文字数に制限はありますか」
「オフィスアワーは事前予約が必要ですか」

これらはいずれもFAQ上で回答根拠が存在しない質問であるが、学習前モデルは「公開されない」「制限はない」「予約不要」などの断定的な回答を生成していた。

失敗分類
　ハルシネーション
　根拠のない推測回答
　FAQ外質問の誤処理

3.3 関連情報の混入

質問内容と直接関係のない情報を付加して回答する傾向も確認された。

質問：
　課題の提出はどこで行いますか

回答：
　学習ポータルの「課題」欄で提出先を指定し、PDF1ファイルで提出します。提出時間は毎週火曜17:00〜18:00のオフィスアワーで行います。

本来求められている提出場所に加え、

　提出形式
　オフィスアワー情報

が混在しており、質問意図との対応が不十分であった。

4. 学習後に改善した点
4.1 FAQ外質問への適切な確認要求

学習後は、FAQに存在しない質問に対して回答を生成せず、「FAQに記載がないため確認が必要」という判断を行えるようになった。

改善確認例

　講義の録画は公開されますか
　課題の文字数に制限はありますか
　提出した課題へのフィードバックはもらえますか
　参考文献の書き方に指定はありますか
　担当教員の名前は何ですか
　次回の講義のテーマは何ですか

これらは学習前には推測回答が行われていたが、学習後は適切に needs_review=true へ誘導されている。

4.2 出力形式の安定化

学習後は全テストケースで、期待するJSON形式が維持されていた。これにより、フォーマット関連の失敗は解消されたと評価できる。

5. 学習後も残存した失敗

学習後に残ったContent失敗は2件である。
5.1 「成績はいつ出ますか」
期待動作：FAQに記載がないため
JSON
　needs_review=true
を返すべきであった。

実際の応答
　提出期限に関するFAQの回答を返してしまった。
失敗分類
　意味的誤マッチ（Semantic Mismatch）
「成績」が存在しないため確認要求へ遷移すべきだったが、「日付・期限」に関する近傍FAQへ誤って対応付けられている。

5.2 「オフィスアワーは事前予約が必要ですか」
期待動作：FAQに予約要否の情報が存在しないため
JSON
　needs_review=true
を返すべきであった。

実際の応答
　オフィスアワーの開催時間を回答した。
失敗分類
　部分一致による誤回答
モデルが「オフィスアワー」というキーワードには反応したものの、「予約が必要か」という質問の主題を理解できていない。
FAQ内の関連情報を返しているが、質問への回答にはなっていない。

6. 考察

今回の結果から、学習によって以下の能力は大幅に改善したと評価できる。

　指定フォーマットの遵守
　FAQ外質問の検知　
　根拠のない回答の抑制
　「確認が必要」という判断の定着

一方で、残存する失敗はいずれも単純なハルシネーションではなく、FAQ内の関連情報への誤マッチによるものであった。つまり、問題の中心は「知識不足」から「質問意図の識別精度」に移行したと考えられる。

7. 総合評価

学習前はフォーマット不備およびFAQ外質問への誤回答が多数発生していたが、学習後はこれらが大幅に改善された。特にFAQ外質問に対して安易に回答せず確認要求へ誘導できるようになった点は、本学習の主要目的を概ね達成できていると評価できる。
一方、残る失敗はFAQ内の類似情報への誤マッチであり、今後は「質問とFAQ回答の意味的対応付け」や「FAQ該当有無の判定精度」を向上させる学習データの追加が有効と考えられる。総合的には、学習効果は非常に高く、実運用に向けて有意な品質改善が確認された。

| 項目 | 前回 | 今回 | 差分 |
| --- | --- | --- | --- |
| 評価件数 | 21 | 21 | ±0 |
| Format Rate | 100.0% | 100.0% | 変化なし |
| Content Rate | 76.2% | 90.5% | +14.3pt |
| Both Rate | 76.2% | 90.5% | +14.3pt |
| 誤答数 | 5件 | 2件 | \-3件 |
| 退化件数 (regressed) | 0件 | 0件 | 変化なし |

---

## Lec02_宿題（基礎レベル）
 lora_r：16　→　32に変更して再実行 # LoRA の行列の次元を2倍にした（小さいほど学習するパラメータが少ない）

4. LoRA 学習
[transformers] The tokenizer has new PAD/BOS/EOS tokens that differ from the model config and generation config. The model config and generation config were aligned accordingly, being updated with the tokenizer's values. Updated tokens: {'eos_token_id': 248046}.
trainable params: 12,779,520 || all params: 765,172,544 || trainable%: 1.6701
 [40/40 03:42, Epoch 20/20]
Step	Training Loss
10	1.570075
20	0.086691
30	0.020562
40	0.014984
Warning: You are sending unauthenticated requests to the HF Hub. Please set a HF_TOKEN to enable higher rate limits and faster downloads.
WARNING:huggingface_hub.utils._http:Warning: You are sending unauthenticated requests to the HF Hub. Please set a HF_TOKEN to enable higher rate limits and faster downloads.
学習時間: 228 秒 | ステップ数: 40 | エポック: 20.0

5. 学習後評価
{"label": "after", "n": 21, "format_rate": 1.0, "content_rate": 0.9047619047619048, "both_rate": 0.9047619047619048, "seconds": 50.1}
n	format_rate	content_rate	both_rate	seconds
label					
before	21	0.095238	0.000000	0.000000	24.4
after	21	1.000000	0.904762	0.904762	50.1
退行: []

6. 知識を問う質問
→設定変更前と同等の回答を出力

7.比較評価

Qwen3.5-0.8Bに対しLoRA（r=32, alpha=32, epoch=20）でFAQ応答学習を実施した結果、評価21問においてフォーマット準拠率は9.5%→100%、内容正答率は0%→90.5%へ改善した。
また、学習前に見られたFAQ外質問への幻覚応答は大幅に減少し、多くのケースで「要確認（needs_review=true）」へ誘導できるようになった。
一方で、「成績はいつ出ますか」「オフィスアワーは事前予約が必要ですか」の2件では近傍FAQへの誤マッチが残存しており、未知質問判定の追加学習余地が確認された。

## 演習

- 使ったモデル・設定（`result.json` の `config` から）：Qwen/Qwen3.5-0.8B、
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
