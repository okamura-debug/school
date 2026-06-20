# 京都高校比較ツール — 仕様書

## 概要

京都市西京区大原野東竹の里町在住の保護者向けに作成した、京都府内高校の比較検討ツール。  
単一HTMLファイル（`index.html`）で動作し、サーバー不要。

---

## ファイル構成

```
src/school/
  index.html   メインアプリ（全機能を1ファイルに収録）
  SPEC.md      本仕様書
```

---

## 対象校データ

| 区分 | 件数 |
|------|------|
| 公立高校 | 55校 |
| 私立高校 | 18校 |
| **合計** | **73校** |

### データソース

- 住所・学科：各校公式サイト・京都府教育委員会
- 偏差値：みんなの高校情報（minkou.jp）
- 口コミリンク：`https://www.minkou.jp/hischool/school/{minkouId}/`

---

## スキーマ定義

```javascript
{
  id:          string,    // 例: 'pub-1', 'pri-1'
  name:        string,    // 学校名
  minkouId:    number,    // みんなの高校情報のID
  address:     string,    // 住所
  lat:         number,    // 緯度
  lng:         number,    // 経度
  devMin:      number,    // 偏差値下限
  devMax:      number,    // 偏差値上限
  devLabel:    string,    // 偏差値表示文字列（例: '70〜74'）
  deptTypes:   string[],  // 学科区分（後述）
  courses:     string,    // コース設定
  annualCost:  string,    // 年間費用目安
  univRate:    number,    // 大学進学率（%）
  saturday:    string,    // 土曜授業（'あり'|'なし'|'隔週'）
  support:     string,    // 進学サポート（'あり'|'なし'）
  gender:      string,    // 男女共学（'共学'|'男子校'|'女子校'）
  affiliated:  string,    // 系列大学
  examType:    string,    // 受験形式
  strictness:  string,    // 校則（'緩め'|'普通'|'厳しい'）
  features:    string,    // 特色・説明文
  website:     string,    // 公式サイトURL
}
```

### 学科区分（deptTypes）カテゴリ

| カテゴリ | 色 | 説明 |
|----------|----|------|
| 普通科 | グレー | 一般的な普通科 |
| 理数・科学系 | ブルー | SSH指定校・理数科など |
| 国際・英語系 | スカイブルー | 英語科・国際科・IB認定校 |
| 工業・工学系 | オレンジ | 工業高校・工学科 |
| 農業・造園系 | グリーン | 農業・園芸・林業系 |
| 商業系 | イエロー | 商業科・ビジネス科 |
| 芸術・音楽系 | ピンク | 美術・音楽専門科 |
| スポーツ・体育系 | レッド | 体育科・アスリートコース |
| 情報系 | バイオレット | 情報科・IT系専門科 |
| 福祉系 | ティール | 介護福祉科・人間科学科 |
| 総合学科 | インディゴ | 総合学科・単位制 |

---

## 偏差値バンド分類

| ラベル | 条件（devMax基準） |
|--------|-------------------|
| S（70以上） | devMax ≥ 70 |
| A（65以上） | devMax ≥ 65 |
| B（60以上） | devMax ≥ 60 |
| C（55以上） | devMax ≥ 55 |
| D（55未満） | devMax < 55 |

---

## 機能仕様

### テーブル表示

- 公立・私立でグループ分けして表示
- 自宅からの距離をHaversine公式で計算（基点: 34.9340, 135.6510）
- ドラッグ＆ドロップで列の並び替え
- 列の表示/非表示の切り替え
- 偏差値帯に応じた行の背景色分け

### フィルタ

| フィルタ名 | 種別 |
|-----------|------|
| 男女共学 | セレクト |
| 土曜授業 | セレクト |
| 校則 | セレクト |
| 系列大学 | テキスト |
| 偏差値 下限 | 数値 |
| 偏差値 上限 | 数値 |
| 学科区分 | 複数選択（OR検索） |

### ブックマーク・メモ

- ★ボタンでブックマーク登録（ハート表示は廃止）
- 各校にメモを記入可能
- localStorage `kyokohi-v2` に自動保存

### 比較モーダル

- 最大4校をチェックして横並び比較
- 全データ項目を一覧表示
- みんなの口コミリンクを含む

### CSVエクスポート

- 表示中の列・フィルタ適用済みデータをCSV出力

---

## GitHub Gist 同期仕様

### 目的

localStorage はデバイスローカルのため、複数PC・スマホ・タブレット間での引き継ぎにGist同期機能を実装。

### 同期対象データ

```json
{
  "version": 2,
  "savedAt": "ISO8601日時",
  "bookmarks": ["pub-1", "pri-3", ...],
  "notes": { "pub-1": "メモ内容", ... },
  "columnOrder": ["name", "address", ...],
  "hiddenCols": ["annualCost", ...]
}
```

### localStorage キー

| キー | 内容 | Gistに含める |
|------|------|-------------|
| `kyokohi-v2` | 全ユーザーデータ | ✕（localのみ） |
| `kyokohi-gist-token` | GitHub PAT | ✕（セキュリティ上ローカルのみ） |
| `kyokohi-gist-id` | Gist ID | ✕（localのみ） |

### GitHub Personal Access Token の取得

1. https://github.com/settings/tokens/new にアクセス
2. `gist` スコープのみチェック
3. 生成されたトークン（`ghp_...`）をアプリに入力

### 操作手順（初回）

1. 「☁ 同期」ボタンをクリック
2. PATを入力
3. 「↑ Gistに保存」→ GistIDが自動入力される
4. 別デバイスで同じHTMLを開いて「☁ 同期」→ PAT + GistID を入力
5. 「↓ Gistから読み込む」でデータ反映

### 操作手順（2回目以降）

- PAT・GistIDはlocalStorageに保存済みのため自動入力される
- 「↑ 保存」または「↓ 読み込む」のみでOK

---

## 今後の改修候補

- [ ] 正確性検証：全73校の住所・偏差値・minkouIdの照合
- [ ] 自動同期：ページ離脱時にGist自動保存
- [ ] ベネッセデータ連携（Chrome拡張経由）
- [ ] 地図表示機能（Leaflet.js等）
- [ ] 通学時間の自動計算（Google Maps API）
- [ ] 学校詳細ページへの展開
