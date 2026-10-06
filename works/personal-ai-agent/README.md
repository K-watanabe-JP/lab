# 完全無料・プライバシー重視のパーソナル AI エージェント構築



## 概要



2026年8〜9月にかけて、xAI の「Grok Bot」、Meta の「Muse」、OpenAI の「Dots」と、常時稼働・自立型のパーソナル AI エージェントが相次いで登場した。

いずれも高性能だが、クラウド上の専用コンピュータで動き、有料サブスクリプションが前提で、データもベンダー側に渡る。

本プロジェクトでは、同等の体験をオープンソースのモデルとツールだけで、ローカル完結・追加費用ゼロで再現することを目的とする。



## 背景: 商用の常時稼働 AI エージェントは何をしているか



商用製品が提供している機能を分解し、機能要件を整理する。




| 項目   | OpenAI Dots                                                                                                                                                                                          | xAI Grok Bot                                                                            | Meta Muse                                                                                                                                         |
| :---- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| 発表   | 2026年9月29日（DevDay 2026）（[WIRED](https://www.wired.com/story/openai-dots-always-on-ai-agents-that-proactively-help/)）                                                                                 | 2026年8月11日、ベータ（[xAI](https://x.ai/news/introducing-grok-bot)）                           | 2026年9月8日（[NYT](https://www.nytimes.com/2026/09/08/technology/meta-muse-ai-agent.html)）                                                           |
| 頭脳   | GPT-6 Astra（[OpenAI](https://openai.com/index/introducing-dots/)）                                                                                                                                    | Grok                                                                                    | Meta製モデル                                                                                                                                          |
| 実行環境 | 専用クラウドコンピュータとブラウザ。ユーザーのPCへの接続も許可制で可能（[OpenAI](https://openai.com/index/introducing-dots/)）                                                                                                           | Bot専用のクラウドコンピュータ。APIやMCPがないツールにもログインして操作（[xAI](https://x.ai/news/introducing-grok-bot)） | アプリやWebを自律的に操作（[NYT](https://www.nytimes.com/2026/09/08/technology/meta-muse-ai-agent.html)）                                                      |
| 連携   | 4,000以上のアプリ。Slack/Teams/ChatGPT（[OpenAI](https://openai.com/index/introducing-dots/)）                                                                                                                | 受信箱、各種ツール、Webサイト                                                                        | iOS/Androidアプリ、[Muse.ai](http://Muse.ai)（[WIRED](https://www.wired.com/story/meta-releases-muse-a-personal-ai-agent-with-privacy-built-into-it/)） |
| 制御   | Custom Rules、Auto-review、Activity View。パスワード変更などは常に人が行う（[OpenAI](https://openai.com/index/introducing-dots/)）                                                                                        | 承認が必要なときだけ人に戻る（[xAI](https://x.ai/news/introducing-grok-bot)）                           | —                                                                                                                                                 |
| 料金   | Pro（月額$100）から提供（[WIRED](https://www.wired.com/story/openai-dots-always-on-ai-agents-that-proactively-help/)）。日本では月額16,800円〜（[Impress Watch](https://www.watch.impress.co.jp/docs/news/2144301.html)） | SuperGrok/Cursor Pro系の加入者向け（[xAI](https://x.ai/news/introducing-grok-bot)）              | —                                                                                                                                                 |




共通する機能要件は、次の「6つ」に整理できる。



1. 常時稼働のランタイム: 24時間動き、スケジュールやイベントをキッカ機に起動する
2. 長期記憶とユーザーモデル: 好み、基準、文体を学習する
3. ツールとコンピュータの操作: シェル、ブラウザ、ファイル、各種アプリを扱う。
4. 能動性: 先回りして作業を準備し、確認が必要な時だけ通知する
5. マルチチャネル: スマホのメッセンジャーから支持し、進捗を受け取る。
6. 承認と限界の制御: やって良いこと、承認が必要なこと、禁止事項のルール



一方、商用製品には自作で避けたい性質もある。WIREDは、OpenAIアカウントで学習利用を許可している場合はDotsとのやり取りにもその設定が適用されると指摘し、エージェントが意図しない情報を共有したり、敵対者にだまされたりする危険にも触れている（[WIRED](https://www.wired.com/story/openai-dots-always-on-ai-agents-that-proactively-help/)）。自作版では、上の要件1〜6をローカルで満たしつつ、データがベンダーのクラウドを通る構造そのものをなくすことが目標になる。



## 「Jev」と、OSS で使える部分



### 「Jev」とは何か

Jev は TypeSafe AI が2026年9月15日に発表した最初の「System One Model」である。文字列を生成せず、「状態（state）＋型付きの質問」を受け取り、確率付きの型付き回答を返す。新しいアーキテクチャ、並列サンプラー、RLCD（Reinforcement Learning for Calibrated Decisions）という学習手法を用いるとされる（[TypeSafe公式](https://typesafe.ai/blog/introducing-system-one-models-and-jev)）。

- 質問の型：`Choice`（選択）、`Score`（順序尺度）、`Noul`（Yes確率0〜1）の3種類（[awesome-jev](https://github.com/cobanov/awesome-jev/blob/main/README.md)）
- 料金：入力100万トークンあたり$0.042で、出力は無料（同上）
- 制約：算術や数え上げが不得意、否定を文字どおりに読む、選択肢は最大255、コンテキストは32K（同上）
- ライセンス：プロプライエタリ (技術の仕様やソースコードを開発した企業が独占的に所有すること) で、重みも論文も公開されていない（[OpenRouter](https://openrouter.ai/blog/insights/what-is-jev/)）



ただし、Jev 本体は有料&amp;クラウド上で動作するため、本システムでは採用しない。代わりに、Jev と同じワイヤー API を話す OSS 群を使用する。



ローカルで使用可能な Jev 互換 OSS


| 名称                  | 種別             | ライセンス      | サイズ・ベース                               | Mac対応                          | 特記事項                                                                                                                                |
| :------------------- | :-------------- | :---------- | :------------------------------------- | :------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------- |
| **Ollaya**          | ランタイム（Ollama風） | Apache-2.0 | —                                     | macOS 14+、メニューバーアプリ            | `/v1/systemone`互換。`127.0.0.1:11435`で待ち受け、stateと質問はログに残さない（[Ollaya FAQ](https://ollaya.dev/docs/faq)）                                |
| **sokudan-ja-310m** | 日本語特化の意思決定モデル  | Apache-2.0 | ModernBERT-ja 310M                    | MLXがデフォルト                      | bench\_jaでchoice精度0.880。M1 MaxのMLXで1問約10ms（[Hugging Face](https://huggingface.co/GeneLab/sokudan-ja-310m)）                          |
| **Kev**             | 意思決定モデル        | Apache-2.0 | Qwen3.5 0.8B / 4B / 9B、Qwen3.8 27B    | MLXを自動使用。4Bは32GB Macで動作        | TypeSafe SDKがそのまま使える（[Kev GitHub](https://github.com/jaredpalmer/kev)）                                                              |
| **Laya**            | エンコーダ型の意思決定モデル | Apache-2.0 | ModernBERT-large 421M、多言語版mmBERT 322M | Ollaya経由でMLX                   | 100以上の言語に対応。`laya-serve`がJev互換API（[Hugging Face](https://huggingface.co/convaiinnovations/laya)）                                    |
| **decider**         | 意思決定モデル        | Apache-2.0 | Qwen3.5 0.8B〜4B                       | Ollaya経由                       | `decider:4b`はtyped decisionsで0.680（[Ollaya](https://ollaya.dev/)）                                                                   |
| **Winnow**          | 意思決定モデル        | Apache-2.0 | Gemma 4のファインチューン（E4B / 12B）、GGUF      | llama.cpp                      | `winnow:e4b`は0.722（[Ollaya FAQ](https://ollaya.dev/docs/faq)）                                                                       |
| **jev-style**       | モデルと互換サーバー     | —          | Qwen3.5 0.8B                          | `pip install "jev-style[mlx]"` | Claude Code用のガードを同梱。4つのNoul判定とリスクScoreをallow/ask/denyに変換（[awesome-jev](https://github.com/cobanov/awesome-jev/blob/main/README.md)） |
| **jev-mcp**         | MCPサーバー        | —          | —                                     | —                              | verify、screen、rerank、classify、diffレビュー、完了判定ゲートなど12個の判断ツール（[jev-agent.com](http://jev-agent.com)）                                    |




制度について: JevBench (Jev 型意思決定モデルを評価するベンチマーク) では、Jev 1.13.0が64.7、Layaが49.9、Kev 4Bが40.9だった（[Developers Digest](https://www.developersdigest.tech/blog/how-to-use-jev-typesafe-guide-2026)）。(数値が高いほど優秀とされる)

オープン勢はJev に迫るが、ベンチマークの版によっては順位が大きく入れ替わる。そのため、自分のタスクで評価してから閾値を決める必要がある。

日本語利用での注意点: 意思決定モデルの多くは英語優先である。sokudanの作者は、Layaを日本語で測定したうえで日本語専用モデルを作った（[Zenn](https://zenn.dev/genelab_999/articles/533144853290a7)、[DEV](https://dev.to/genelab_999/decision-models-are-english-first-i-measured-them-in-japanese-then-built-one-1p12)）。ただしsokudanにも弱点がある。学習データは合成のビジネス問い合わせに限られ、別の400件のテストでは0.424とラベル事前分布（0.483）を下回った。「その他」も取りこぼしやすい（[DEV](https://dev.to/genelab_999/decision-models-are-english-first-i-measured-them-in-japanese-then-built-one-1p12)）。選択肢には短い説明文を必ず付けると精度が上がる（0.537→0.880）（同上）。