# Contributors Feature Specification

## 概要

リポジトリ一覧ページ (`/app`) に Contributors 列を追加し、各リポジトリへの貢献者を視覚的に表示する。

## 目的

- チームメンバーがどのリポジトリに貢献しているかを一目で把握
- リポジトリの活発度やメンテナンス状況の可視化
- 組織内のコラボレーション状況の把握

---

## GitHub API

### Endpoint

```
GET /repos/{owner}/{repo}/contributors
```

### Parameters

- `per_page`: 100 (最大値)
- `anon`: false (匿名貢献者を除外)

### Response Fields

```typescript
interface Contributor {
  login: string;           // GitHub username
  id: number;              // User ID
  avatar_url: string;      // Avatar image URL
  html_url: string;        // Profile URL
  contributions: number;   // Contribution count
  type: string;            // "User" or "Bot"
}
```

### Rate Limits

- 認証済み: 5000 requests/hour
- 大きなリポジトリの場合、計算に時間がかかる可能性あり

---

## データベーススキーマ

### 既存テーブルへの追加

`repo_inventory` テーブルに新しいフィールドを追加:

```typescript
{
  // ... existing fields
  contributors: jsonb("contributors"), // Contributor[]
  contributorsCount: integer("contributors_count").default(0),
  contributorsUpdatedAt: timestamp("contributors_updated_at"),
}
```

### データ構造

```typescript
// JSON stored in contributors field
type ContributorData = {
  login: string;
  avatarUrl: string;
  profileUrl: string;
  contributions: number;
}[];
```

---

## 実装フェーズ

### Phase 1: データ取得とDB保存

1. **GitHub API 関数追加** (`lib/github.ts`)
   ```typescript
   export async function listRepoContributors(
     octokit: Octokit,
     owner: string,
     repo: string
   ): Promise<ContributorData[]>
   ```
   - ページネーション対応（最大100件取得）
   - Bot アカウントはフィルタリング
   - 上位10名のみ保存（画面表示用）

2. **スキャン処理への統合** (`lib/scan.ts`)
   - `scanOneRepo()` 内で contributors 情報を取得
   - `initInventory()` に contributors フィールドを追加
   - エラーハンドリング（404やタイムアウトに対応）

3. **データベーススキーマ更新**
   - `lib/db/schema.ts` に新フィールド追加
   - `pnpm db:push` でスキーマ反映

### Phase 2: UI実装

1. **Contributors 列の追加** (`app/app/page.tsx`)
   - テーブルに新しい列を追加
   - 列順: `Repository | Last Updated | Language | Frameworks | Contributors`

2. **Contributors 表示コンポーネント**
   ```typescript
   // 表示形式
   // - アバター画像を横並びで表示（最大5-7人）
   // - 重なるように配置（z-index でスタック）
   // - 残りの人数を "+N" で表示
   // - ホバーで全員のツールチップ表示
   ```

3. **スタイリング**
   - アバター: 32x32px (円形)
   - 重なり: -8px マージン
   - ホバー効果: border highlight
   - "+N" バッジ: 円形、グレー背景

### Phase 3: パフォーマンス最適化（オプション）

1. **キャッシング戦略**
   - Contributors は頻繁に変わらないため、7日間キャッシュ
   - `contributorsUpdatedAt` でキャッシュ有効期限を管理

2. **バックグラウンド更新**
   - 定期的なバッチ処理で contributors を更新
   - QStash を使った非同期更新

---

## UI/UX 仕様

### 表示ルール

1. **上位貢献者の表示**
   - 最大7名のアバターを表示
   - 8人目以降は "+N" で集約

2. **空の状態**
   - Contributors がいない場合: "-" を表示
   - まだ取得していない場合: "..." を表示

3. **ツールチップ**
   - ホバー時: 全貢献者のリスト表示
   - フォーマット: `@username (123 contributions)`

### レイアウト例

```
┌─────────────┬──────────────┬──────────┬──────────────┬─────────────────┐
│ Repository  │ Last Updated │ Language │ Frameworks   │ Contributors    │
├─────────────┼──────────────┼──────────┼──────────────┼─────────────────┤
│ frontend    │ 2 days ago   │ TypeScr. │ Next.js, ... │ 👤👤👤👤👤 +3   │
│ backend-api │ 1 week ago   │ Python   │ FastAPI      │ 👤👤👤         │
│ infra       │ 3 months ago │ HCL      │ Terraform    │ 👤👤           │
└─────────────┴──────────────┴──────────┴──────────────┴─────────────────┘
```

---

## 技術的考慮事項

### Rate Limit 管理

- Contributors API は計算コストが高い
- バッチ処理で並列度を制限（5-10並列）
- リトライロジックを実装

### エラーハンドリング

- 404: リポジトリが削除された → contributors: null
- 403/429: Rate limit → スキップして次回リトライ
- Timeout: 長時間実行リポジトリ → タイムアウト設定

### データサイズ

- 上位10名のみ保存 → JSONB サイズ約1-2KB/repo
- 500 repos × 2KB = 1MB（許容範囲）

---

## テストケース

1. **正常系**
   - ✅ Contributors が正しく取得・表示される
   - ✅ アバターが正しく表示される
   - ✅ "+N" 表記が正しい

2. **異常系**
   - ✅ Contributors がいないリポジトリ → "-" 表示
   - ✅ API エラー → エラーログ、次回リトライ
   - ✅ Rate limit → スキップして継続

3. **パフォーマンス**
   - ✅ 100 repos のスキャンが10分以内に完了
   - ✅ ページ読み込みが2秒以内

---

## マイルストーン

- **Phase 1**: データ取得とDB保存 (1-2時間)
- **Phase 2**: UI実装 (1-2時間)
- **Phase 3**: 最適化（オプション）

---

## 参考資料

- [GitHub REST API - List repository contributors](https://docs.github.com/en/rest/repos/repos#list-repository-contributors)
- [Octokit pagination](https://github.com/octokit/plugin-paginate-rest.js)
