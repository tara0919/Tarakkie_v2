# Tarakkie_v2 Development Project Documentation

このフォルダには、Tarakkie_v2 モジュラー分割キーボードの開発に関するすべての重要ドキュメントがまとめられています。

## ドキュメント一覧

1.  **[部品リスト (hardware_bom.md)](hardware_bom.md)**
    *   ベースボードおよび各モジュールの製作に必要な主要部品の一覧です。
    *   秋月電子などのリンクや、選定の注意点（プルアップ抵抗、トラックボールなど）が含まれています。

2.  **[プロトタイプ検証ガイド (walkthrough.md)](walkthrough.md)**
    *   ハードウェアの配線図、ファームウェアの書き込み手順、動作テストの方法について解説しています。
    *   開発の現段階での「公式ガイド」です。

3.  **[実装計画書 (implementation_plan.md)](implementation_plan.md)**
    *   ピンアサイン、ファームウェアの設計方針、モジュール交換の仕組みなどの技術的な詳細です。

4.  **[タスク状況 (task_status.md)](task_status.md)**
    *   これまでに完了した作業と、残っている開発タスクのリストです。

## 今後のステップ
ハードウェアの部品が揃い次第、**[検証ガイド](walkthrough.md)** に従って試作とテストを進めてください。
不明点があればいつでも聞いてください！

## ローカル開発 (VS Code Devcontainer / Antigravity)

GitHub Actions を待たずに、手元の PC でビルドが可能です。

### VS Code でのビルド
1. Docker をインストールした状態で、VS Code で「Reopen in Container」を選択します。
2. ターミナルで以下のコマンドを実行します。
```bash
west build -p -s zmk/app -b xiao_ble -- -DSHIELD=tarakkie_v2_left -DZMK_CONFIG=/workspaces/modular_split_zmk/config
```

### Antigravity (AI) に依頼する
私（Antigravity）に「左側のビルドをして」と依頼するだけで、私が Docker を操作してビルドを行い、エラーがあれば修正し、成功すれば `.uf2` ファイルの場所を教えます。
