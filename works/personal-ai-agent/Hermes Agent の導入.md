## Hermes Agent の導入

**Step1: セットアップ方式の選択**

```
How would you like to set up Hermes?
 1. Quick Setup (Nous Portal)
 2. Full setup
 3. Blank Slate ← 今回はこちらを選択
```

`3. Blank Slate` を選ぶ理由
- `1. Quick Setup` は Nous Portal (外部クラウド) への OAuth 認証が必須なため除外
- `2. Full setup` はブラウザ自動化・音声合成・外部検索などの全オプションを手動で設定する必要があり、設定漏れによる外部通信リスクが残る
- `3. Blank Slate` はモデル接続とター見ある実行環境以外のすべての機能を初期状態で OFF にするため、ホワイトリスト形式で安全に機能を足すことが可能

上記の理由から、`3. Blank Slate` を選択。

---

**Step2: LLM プロバイダの選択**

```
Slect Provider:
  ・
  ・
  (●) Custom endpoint (enter URL manually) ← 今回はこちらを選択
  ・
  ・
```

「`Custom endpoint`」 を選択すると、自身の Mac 上で起動しているローカルの Ollama サーバー (127.0.0.1:11434) へ OpenAI 互換 API として接続する。

---

**Step3: API base URL (エンドポイント) の入力**

```
API base URL [e.g. https://api.example.com/v1]: http://127.0.0.1:11434/v1
API key [optional]: (空のまま Enter)
```

---

**Step4: API compatibility mode (API 互換モード) の確定**

```
Select API compatibility mode:
  1. Auto-detect
     Use Hermes URL heuritics; best for standard OpenAI-compatible endpoints.
  2. Chat Completions
     Use /chat/completions for standard OpenAI-compatible servers.
  3. Responses / Codex
     Use /responses for Codex-compatible tool-calling backends.
  4. Anthropic Messages
     Use /v1/mesasges for Anthropic-compatible endpoints.
Choics [1-4, Enter to keep current/detacted]: 2 ← 今回はこちらを選択
```

今回は、1 と 2 のどちらでも構わないが、挙動を明示的に固定したいため「2」を選択。

---

**Step5: 使用するモデルの選択**

今回は「qwen3.5:4b」を使用。(軽量でテストに最適)

---

**Step6: コンテキスト長の指定**

```
Context length in tokens [leave blank for auto-detect]: 65536
```

Ollama 公式が AI エージェント使用の場合、「64k トークン以上」を使用しているため、コンテキスト長は「65536」に指定。

---

**Step7: 表示名の設定**

```
Display name [Local (127.0.0.1:11434)]: Ollama
```

表示されるプロバイダー名を設定。

---

**Step8: Reasoning Effort の指定**

```
Select reasoning effort:
  (○) minimal
  (○) low
  (●) medium (Default) ← こちらを選択
  (○) high
  (○) xhigh
  (○) max
  (○) ultra
  (○) Disable reasoning
  (○) Skip (keep current)
```

とりあえず、Default の「medium」を指定。

候補としては「medium」と「Disable reasoning」(まずは) で、Disable reasoning の方が速度が早くなり、精度も落ちない場合はあり。

**Step9: Terminal Backend (Docker サンドボックス) の選択**

```
Select terminal backend:
  (●) Docker/Podman - isolated container with configurable resources ← 今回はこちらを選択
```

**Step10: 外向き通信ファイアウォールの制限**

```
Enable egress firewall for Docker sandboxes?
  (●) Yes ← 今回はこちらを選択
  (○) No
```

- Yes: サンドボックスの外向き通信に制限を設ける。意図しない通信を抑える一方、開発に必要な接続先の許可設定が必要になることがある
- No: この保護を無効化。接続トラブルは減らせるが、別のネットワーク制限がなければ、サンドボックス内のコマンドが外部へ通信できる状態になる

今回は外部との通信を遮断したいため、「Yes」を選択。

**Step11: 最小構成 (Blank Slate) でのセットアップ完了**

```
Your minimal agent is ready. What next?
  (●) 1. Start with everything disabled - finish now (mose minimal)
  (○) 2. Walk through all configurations - Opt in to tools, skills, plugins, MCP
```

`1. Start with everything disabled` を選択し、セットアップを完了させる。

**Step12: config.yaml を編集**

以下の設定ファイルを開く。

```
~/.hermes/config.yaml
```

ファイル内の「platform-toolsets:cli:」を確認する。
デフォルトでは以下の通り

```
platform_toolsets:
  cli: [file, terminal, vision, skills]
```

ここから、以下の追加・削除を行う。

1. デフォルトで入っている「vision」を削除
  接続中のモデルがマルチモーダル (画像認識対応) 出ない場合、画像解析ツールが有効なままだとエージェントが不必要に画像を処理しようとしてエラーやハルシネーションを起こす。テキスト主体のエージェントとして動かす場合は外すことを推奨。
2. 「clarify」(確認・質問ツール) の追加
  ユーザーの支持が曖昧なとき、ローカル LLM が勝手に推測してコマンドを暴走実行するのを防ぎ、対話で聞き返してくるようになる。
3. 「todo」の追加
  複雑な作業をステップ順にローカルで分解し、順を追って実行するための軽量ツール
4. 「memory」&「session_search」の追加
  エージェントが過去に学習した内容や前回の会話履歴を、外部サーバーではなくローカルの `~/.hermes/` 内のデータベースから参照できるようにする。

編集後は以下の通り。

```
platform_toolsets:
  cli:
    - file
    - terminal
    - clarify
    - todo
    - memory
    - skills
    - session_search
```
