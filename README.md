# qwen38-sft-ja

Qwen3.8-27B を [Unsloth](https://github.com/unslothai/unsloth) で日本語 SFT（LoRA）する Google Colab ノートブックのサンプルです。

> [!WARNING]
> **学習データは架空の一般的なサンプルです。**
> このノートブックは、ノートブック内に埋め込んだ**架空の社内ヘルプデスク FAQ 12 件**を Qwen3.8-27B に覚えさせる（ほぼ丸暗記させる）動作確認用の例です。
> 実在の企業・組織の規程やデータは含みません。
> 学習後のモデルは、学習データにない社内規程を聞かれると架空の値を断定的に答える（作話する）ため、実務の回答用途には使えません。

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Ohtani-y/qwen38-sft-ja/blob/main/Qwen3_8_SFT_ja.ipynb)

| ファイル | 内容 |
|---|---|
| [`Qwen3_8_SFT_ja.ipynb`](Qwen3_8_SFT_ja.ipynb) | ノートブック本体（2026-10-03 の実行結果つき） |
| [`LICENSE`](LICENSE) | LGPL-3.0 |

学習した LoRA アダプタは Hugging Face で公開しています: [y-ohtani/qwen38_lora](https://huggingface.co/y-ohtani/qwen38_lora)

## ノートブックの流れ

1. インストール（バージョン固定）
2. GPU の確認（VRAM が足りなければここで止める）
3. `unsloth/Qwen3.8-27B` を bf16 で読み込み
4. LoRA アダプタの追加
5. 学習データ 12 件を chat template で整形（`enable_thinking=False`）
6. 30 ステップ学習（assistant の応答部分だけに損失をかける。ログは Weights & Biases に記録）
7. 推論（非 thinking / thinking）
8. 簡易評価（学習前のベースモデルと比較）
9. 保存と Hugging Face へのアップロード（モデルカード付き）

## 学習データ（架空のサンプル 12 件）

外部のデータセットは使っていません。ノートブックの「5. データの準備」に直接書いた 1 問 1 答です。
残業時間の上限（月 45 時間・年 360 時間など）だけは労働基準法の上限規制と同じ数値で、それ以外の締め日・内線番号・金額などはすべて架空です。

system プロンプト（全件共通）: 「あなたは社内ヘルプデスクのアシスタントです。日本語で、結論から簡潔に答えてください。」

| # | 質問 | 回答 |
|---|---|---|
| 1 | 経費精算の締め日を教えてください。 | 毎月25日が締め日です。25日が土日祝の場合は直前の営業日に前倒しになります。締め後の申請は翌月分として処理されます。 |
| 2 | 社給パソコンのパスワードを忘れました。どうすればいいですか。 | 情報システム部への電話連絡で初期化できます。内線4120に本人から連絡し、社員番号を伝えてください。再設定は原則として当日中に完了します。 |
| 3 | 有給休暇の残日数はどこで確認できますか。 | 勤怠システムの「休暇管理」画面で確認できます。トップページ右上のメニューから開き、当年度付与分と繰越分が分けて表示されます。 |
| 4 | 在宅勤務の申請は何日前までに出す必要がありますか。 | 前営業日の17時までです。緊急時は当日申請も認められますが、事後に上長の承認が必要になります。 |
| 5 | 会議室を予約したいのですが、方法がわかりません。 | グループウェアの「施設予約」から予約します。日時と人数を入力すると空いている会議室が一覧表示されるので、そこから選択してください。30分単位で最大4時間まで押さえられます。 |
| 6 | 出張の航空券は自分で手配してよいですか。 | 原則として指定の旅行代理店経由で手配します。自己手配は代理店で取り扱いがない路線に限り、事前に上長の承認を得た場合のみ認められます。 |
| 7 | 健康診断はいつ受けられますか。 | 毎年9月から11月にかけて実施されます。対象者には8月中旬に案内メールが届くので、記載の予約サイトから希望日を選んでください。 |
| 8 | 退職した社員のファイル共有権限はどう扱われますか。 | 退職日の翌営業日に自動で無効化されます。引き継ぎが必要なファイルは、退職日までに所属部門の共有フォルダへ移動しておいてください。 |
| 9 | 社外の人とファイルをやり取りしたいのですが、メール添付でよいですか。 | 10MBを超える場合は社内のファイル転送サービスを使ってください。10MB以下であればメール添付で問題ありませんが、機密区分が「社外秘」以上の資料はパスワード保護が必須です。 |
| 10 | 残業時間の上限について教えてください。 | 原則として月45時間、年間360時間が上限です。特別条項の適用時でも月100時間未満、複数月平均80時間以内に収める必要があります。 |
| 11 | 備品の購入申請はどこから出しますか。 | 購買システムの「一般備品申請」から出します。5万円未満は部門長承認のみ、5万円以上は部門長と経理部の二段階承認が必要です。 |
| 12 | VPNに接続できません。 | まず社給端末の証明書の有効期限を確認してください。期限切れが原因の場合が大半です。証明書が有効なのに接続できないときは、内線4120へ端末名とエラーコードを添えて連絡してください。 |

自分のデータで学習する場合は、このセルを `messages` 形式（system / user / assistant）のデータを読み込む処理に差し替えてください。

## 必要な環境

- **GPU: VRAM 80GB 以上**（H100 80GB / A100 80GB / RTX PRO 6000 96GB など）。bf16 の重みだけで約 56GB あります。
  - Colab の無料 T4（16GB）や A100 40GB では動きません。
  - 動作確認は Google Colab の NVIDIA RTX PRO 6000 Blackwell Server Edition（96GB）で行いました。
- 主なライブラリ（インストールセルで固定）:

| パッケージ | バージョン |
|---|---|
| torch | 2.8.0 |
| transformers | 5.3.0 |
| trl | 0.22.2 |
| unsloth | 2026.7.5 |
| unsloth_zoo | 2026.7.6 |
| torchao | 0.16 以上 0.18 未満 |

## 実行手順

1. 上の「Open in Colab」から開く。
2. [ランタイム] > [ランタイムのタイプを変更] で VRAM 80GB 以上の GPU を選ぶ。
3. Colab のシークレットに `HF_TOKEN`（Hugging Face）と `WANDB_API_KEY`（Weights & Biases）を登録する（下記）。
4. 「9. 保存と Hugging Face へのアップロード」の `HF_REPO_ID`・`HF_PRIVATE`・`GITHUB_URL` を自分のものに書き換える。
5. [ランタイム] > [すべてのセルを実行] で上から順に実行する。
   インストールセルは torch を 2.8.0 に差し替えるので、途中のセルから始める場合は [セッションを再起動してすべて実行する] でやり直す。

### シークレットの設定（Hugging Face / Weights & Biases）

ノートブックにトークンや API キーを直接書くと、共有・公開したときにそのまま漏れます。必ず Colab のシークレットを使ってください。

| シークレット名 | 値 | 取得場所 | 使うセル |
|---|---|---|---|
| `HF_TOKEN` | Hugging Face の **Write** 権限トークン | <https://huggingface.co/settings/tokens>（Fine-grained トークンの場合は、アップロード先の名前空間への書き込み権限を付ける） | 9 章「HuggingFaceログイン」（`login(userdata.get('HF_TOKEN'))`） |
| `WANDB_API_KEY` | Weights & Biases の API キー | <https://wandb.ai/authorize> | 6 章「Weights & Biases ログイン」（`wandb.login(key=userdata.get('WANDB_API_KEY'))`） |

1. Colab 左側の鍵アイコン（シークレット）を開き、上の 2 つを登録する。
2. それぞれ「ノートブックからのアクセス」をオンにする。オフのまま実行すると、許可を求めるダイアログが出て、応答しないとタイムアウトする。
3. W&B を使わない場合は、6 章のログインセルを飛ばし、`SFTConfig` の `report_to` を `"none"` にする。

W&B のプロジェクト名は `WANDB_PROJECT`（`qwen38-sft-ja`）、run 名は `SFTConfig` の `run_name`（`qwen38-27b-lora-sample`）です。

### Hugging Face へのアップロード内容

| 設定 | 値 |
|---|---|
| リポジトリ | `y-ohtani/qwen38_lora`（`HF_REPO_ID`） |
| 公開範囲 | 公開（`HF_PRIVATE = False`） |
| アップロードするファイル | LoRA アダプタ（`adapter_config.json`・`adapter_model.safetensors`、約 160MB）、processor 一式（`tokenizer.json`・`tokenizer_config.json`・`chat_template.jinja`・`processor_config.json`）、モデルカード（`README.md`） |
| モデルカードのメタデータ | `base_model: unsloth/Qwen3.8-27B`、`library_name: peft`、`license: apache-2.0`、`language: ja`、`pipeline_tag: text-generation`、`tags`（lora / sft / trl / transformers / unsloth / qwen3_5 / japanese / sample） |
| モデルカードの本文 | 架空サンプルである旨の注意書き、学習データ 12 件の全文、学習設定、評価結果、制限事項、使い方、ライブラリのバージョン、ライセンス、学習コード（このリポジトリ）へのリンク |
| 認証 | 「HuggingFaceログイン」セルで `HF_TOKEN` を使って `login()` し、アップロードセルの `HfApi()` はそのログインを使う |

processor ごとアップロードしているので、アップロードしたリポジトリから `FastLanguageModel.from_pretrained("y-ohtani/qwen38_lora")` でベースモデルと一緒にそのまま読み込めます。

## 結果（2026-10-03）

**学習**: 30 ステップ（12 件を約 10 エポック）、約 40 秒。学習損失は 1.66 → 0.004。損失・学習率の推移は Weights & Biases に記録しています。

**簡易評価**（貪欲法、`enable_thinking=False`）:

| 項目 | 学習後 | 学習前（ベースモデル） |
|---|---|---|
| 学習した 12 問の再現（完全一致） | 11/12（残る 1 問も語尾が違うだけ） | - |
| 言い換えた質問 8 問（事実が合っているか） | 6/8 | 3/8 |
| 学習データにない社内規程 4 問 | 3 問で架空の値や手順を断定 | 断定なし（不明と答えるか一般論） |
| 一般的な能力（コード・計算）2 問 | 2/2 | - |

- 言い換えで外した例: 経理承認が要る備品の金額（正: 5 万円 → 回答: 5,000 円）、健康診断の案内時期（正: 8 月中旬 → 回答: 9 月中旬）。
- 作話した例: 社員食堂の営業時間、通勤手当の上限額（月 15 万円と回答）、社内 Wi-Fi のパスワードの確認方法（存在しない掲示場所を案内）。同じ設定の 1 回目の実行では、Wi-Fi のパスワードそのものを作って答えました。
- 学習データの内容は覚えましたが、汎化や事実性は保証されません。12 件・約 10 エポックの丸暗記なので、性能評価としての意味は小さいです。

## Qwen3.6 版からの変更点

このノートブックは Qwen3.6 用に書いたものを Qwen3.8-27B 用に書き換えています。Qwen3.8 は Qwen3.6 と同じ `qwen3_5` アーキテクチャで、重みのテンソル構成も同じです。

- **torchao を 0.18 未満に固定**: 2026 年 8 月公開の torchao 0.18.0 は torch 2.10 以降にしかない `torch.nn.functional.ScalingType` を import するため、torch 2.8 では `from unsloth import FastLanguageModel` が `ImportError` で落ちます。
- **学習データの整形に `enable_thinking=False`**: Qwen3.8 のチャットテンプレートは、thinking が有効だと system プロンプトの先頭に「Reasoning effort is set to xhigh. ...」という指示文を入れます。非 thinking の推論とプロンプトを揃えるため、学習データもこの指示文なしで整形します。
- **推論過程は `reasoning_content` から読まれる**: thinking を学習させたい場合は、assistant メッセージに `reasoning_content` を付けます（content 内の `<think>` タグは使われません）。
- **thinking 推論は `reasoning_effort="low"`**: 既定の `xhigh` では考える量が多く、`max_new_tokens` で回答前に打ち切られることがあります。
- **保存は processor ごと**: トークナイザだけでなく画像/動画の前処理設定も保存し、Hub から読み込めるようにしました。

## ライセンス

- ノートブック: [LGPL-3.0](LICENSE)。Unsloth 公式ノートブック [`nb/Qwen3_6_MoE.ipynb`](https://github.com/unslothai/notebooks/blob/main/nb/Qwen3_6_MoE.ipynb)（LGPL-3.0）をベースにしています。
- 学習した LoRA アダプタ: Apache-2.0（ベースモデル Qwen3.8-27B のライセンスに準拠）。
- 学習データ 12 件: このリポジトリ用に作成した架空のサンプルです。

## 参考

- [Unsloth: Qwen3.8 ファインチューニングガイド](https://unsloth.ai/docs/models/qwen3.8/train)
- [Qwen3.8-27B モデルカード](https://huggingface.co/Qwen/Qwen3.8-27B)
- [Unsloth 公式ノートブック `nb/Qwen3.8_(27B)-Conversational.ipynb`](https://github.com/unslothai/notebooks/blob/main/nb/Qwen3.8_%2827B%29-Conversational.ipynb)
- [Hugging Face: User Access Tokens](https://huggingface.co/docs/hub/security-tokens)
