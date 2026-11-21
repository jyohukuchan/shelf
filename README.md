# Shelf — Hy4-flash 専用 fork

Hy4-flash のテキスト生成を AMD Radeon AI PRO R9700 1 台と V620 2 台で実行する。expert parallel は 3 rank、本体は EXL3-INT8、Engram は FP8、KV は Octave。付属の DFly2 で 5 token を先読みできる。

## クイックスタート

[このホストのビルド手順](docs/research/20261119/hy4_flash_plan.md#このホストでのビルドと入力)で `build-hy4/shelf-compute` を作成する。入力は空白区切りの token ID を保存したテキストファイル。次の例は校正列の先頭 4 token から生成する。

```bash
quantroot="$HOME/datapool/rocm-exl3-rdna2"
pilot="$quantroot/hy4-flash-pilot"
printf '%s\n' '120000 802 15063 2576' > prompt-ids.txt
python3 tools/hy4_flash/locked.py ep3 build-hy4/shelf-compute hy4 generate \
  --model-dir "$quantroot/models/Hy4-flash-exl3-4bpw-mul24" \
  --engram-dir "$pilot/engram-e4m3fn" \
  --dfly-dir "$pilot/shelf-dfly/Hy4-flash-dfly-4bpw" \
  --placement "$pilot/shelf-placement/owners-4bpw.json" \
  --input-ids-file prompt-ids.txt --max-new-tokens 128 --chunk 1024
```

生成した token ID は `GENERATED_IDS` に出力される。GPU の予約量と速度は[性能記録](docs/perf/current.md#hy4-flash-専用-fork-の-ep3)を参照。

## ドキュメント

対応機能・設計仕様・性能値の正本は [docs/](docs/README.md) に置いている。開発方針やコーディング規約は [AGENTS.md](AGENTS.md) を参照。

### 利用者ガイド（[docs/user/](docs/user/README.md)）

- [build.md](docs/user/build.md) — 要件・ビルド手順・CMake オプション・ROCm トラブルシュート
- [quantize.md](docs/user/quantize.md) — EXL3-INT8（mul24）・Ngram4 / Ngram5 の変換・検証・品質確認
- [compute.md](docs/user/compute.md) — `shelf-compute`（単発推論・`--serve-stdio`・DFlash2 / MTP / Prefix Cache）
- [server.md](docs/user/server.md) — `shelf-server`（OpenAI 互換 API・エージェント接続・構造化出力）
- [multi-gpu.md](docs/user/multi-gpu.md) — PP / EP / TP によるマルチ GPU 実行とモデル別対応
- [bench.md](docs/user/bench.md) — `shelf-bench`（`pp` / `tg` / `quality` / microbench）
- [env_vars.md](docs/user/env_vars.md) — `SHELF_*` 環境変数リファレンス（正本）
- [recipes/](docs/user/recipes/) — レシピ一覧

### 開発・性能・R&D

- [docs/developer/README.md](docs/developer/README.md) — 開発者向け（アーキテクチャ・ランタイム・量子化・カーネル・テスト契約）
- [docs/perf/README.md](docs/perf/README.md) — 性能・ベンチマーク（現在の性能値・測定手順・品質評価）
- [docs/research/README.md](docs/research/README.md) — R&D 記録（実験経緯・採否判断・棄却根拠）

## Contributors

- **jyohukuchan** — EXL3-INT8 開発 / EXL3-INT8 カーネル最適化 / 構造化出力 etc

## License

MIT — [LICENSE](LICENSE)
