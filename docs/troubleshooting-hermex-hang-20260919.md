# Hermex ハング問題の診断と対処 (2026-09-19)

## 症状
- iPhone (Tailscale経由) からHermex (8788) にアクセスすると、過去のClaude Codeセッション画面に自動遷移し、UIがフリーズ（スクロール・入力不能）、15秒程度で切断

## 原因
1. **gateway未再起動**: `hermes update` 後にgatewayが再起動されておらず、新旧モジュール混在（mixed sys.modules）
2. **state.db WAL不整合**: `DeletedWalGenerationError` — 生存プロセスが削除済みWALを掴んで書き込み不能
3. **巨大セッションの自動復元**: iPhoneのlocalStorageに `claude_code_43b032263e696e0825163870`（419件・85万文字超のClaude Codeセッション）が残っており、フロントエンドJSがレンダリング不能でフリーズ

## 対処
1. `hermes gateway restart` 実行
2. `com.user.hermes_serve`（port 9119）と `com.user.hermes_webui`（port 8788）を `launchctl stop/start` で再起動
3. iPhone側: ホーム画面のPWAを削除 → Safariで再アクセス → セッション一覧から新規チャット作成

## 教訓
- `hermes update` 後は必ず全サービス再起動（gateway + serve + webui）
- PWAのlocalStorageはSafariの「Webサイトデータ消去」では消えない。削除→再追加が必要
- Claude Codeの巨大セッション（数百件メッセージ）の自動復元はフロントエンド破綻リスクあり