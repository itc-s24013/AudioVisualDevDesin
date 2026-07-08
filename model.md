# 情報モデル定義書 - Webオーディオビジュアライザー

**ペルソナ：** 内間さん（26歳・WebデザイナーかつボカロP）  
**目的：** 楽曲に連動する派手なビジュアライザーで動画クオリティを向上

---

## 1. 情報モデル概要図

```
┌─────────────┐        ┌──────────────────┐
│    users    │1      N│     presets      │
│（ユーザー情報）│────────│（設定保存・管理）   │
└─────────────┘        └──────────────────┘
                                │1
                                │
                                │1
                       ┌────────────────────┐
                       │  design_settings   │
                       │（デザイン設定）      │
                       └────────────────────┘

※ 音声・再生関連はブラウザ上で処理するためDBに永続化しない
```

---

## 2. エンティティ定義

### 2-1. users（ユーザー情報）

| 属性名 | データ型 | 必須 | 説明 | 備考 |
|--------|---------|------|------|------|
| id | UUID | ✅ | ユーザーの一意識別子 | PRIMARY KEY・自動生成 |
| email | TEXT | ✅ | メールアドレス | UNIQUE・ログインID |
| password | TEXT | ✅ | パスワード | ハッシュ化して保存 |
| username | TEXT | ✅ | ユーザーの表示名 | |
| is_logged_in | BOOLEAN | ✅ | ログイン・ログアウト状態 | セッション管理 |
| created_at | TIMESTAMP | ✅ | アカウント作成日時 | 自動生成 |
| updated_at | TIMESTAMP | ✅ | 最終更新日時 | 自動更新 |

---

### 2-2. presets（設定保存・管理）

| 属性名 | データ型 | 必須 | 説明 | 備考 |
|--------|---------|------|------|------|
| id | UUID | ✅ | プリセットの一意識別子 | PRIMARY KEY・自動生成 |
| user_id | UUID | ✅ | 所有ユーザーのID | FOREIGN KEY → users.id |
| name | TEXT | ✅ | プリセット名 | 例：「新曲A用」 |
| settings | JSONB | ✅ | デザイン設定の内容一式 | design_settingsの値をJSON保存 |
| is_deleted | BOOLEAN | ✅ | 削除フラグ | 論理削除・デフォルトFALSE |
| created_at | TIMESTAMP | ✅ | プリセット作成日時 | 自動生成 |
| updated_at | TIMESTAMP | ✅ | 最終更新日時 | 自動更新 |

---

### 2-3. design_settings（デザイン設定）

| 属性名 | データ型 | 必須 | 説明 | 備考 |
|--------|---------|------|------|------|
| id | UUID | ✅ | 設定の一意識別子 | PRIMARY KEY・自動生成 |
| preset_id | UUID | ✅ | 紐づくプリセットのID | FOREIGN KEY → presets.id |
| line_color | TEXT | ✅ | 線の色 | カラーコード（HEX） |
| line_width | INTEGER | ✅ | 線の太さ | px単位 |
| line_type | TEXT | ✅ | 線の種類 | 実線 / 点線 など |
| background_color | TEXT | ✅ | 背景色 | カラーコード（HEX） |
| graph_type | TEXT | ✅ | グラフの形 | 波形 / 周波数 / 円形 |
| sensitivity | FLOAT | ✅ | 感度設定 | 重低音・BPMへの反応強度 |
| animation_speed | FLOAT | ✅ | アニメーション速度 | 波形アニメーションの速さ |
| effect_type | TEXT | ✅ | エフェクトの種類 | グロー / グラデーション / パーティクル |
| is_default | BOOLEAN | ✅ | デフォルトデザインフラグ | デフォルトFALSE |
| created_at | TIMESTAMP | ✅ | 作成日時 | 自動生成 |
| updated_at | TIMESTAMP | ✅ | 最終更新日時 | 自動更新 |

---

### 2-4. 音声・再生関連（非永続化）

音声・再生関連の情報はブラウザ上のWeb Audio APIで処理するため、**データベースには保存しない。**

| 情報項目 | 処理場所 | 説明 |
|---------|---------|------|
| 音声ファイル | ブラウザ（メモリ） | ローカルから読み込み・ブラウザ上で保持 |
| ファイルフォーマット | ブラウザ | Web Audio APIが対応形式を判定 |
| 音声データの長さ | ブラウザ | AudioBufferから取得 |
| リアルタイム再生データ | ブラウザ | AnalyserNodeで随時取得 |
| 再生時間 | ブラウザ | AudioContextで管理 |
| 再生・停止ステータス | ブラウザ（State） | Reactのstateで管理 |
| 音量調整 | ブラウザ（State） | GainNodeで制御 |
| 再生バー | ブラウザ（State） | 再生位置をstateで管理 |

---

## 3. リレーション定義

| リレーション | 多重度 | 説明 |
|------------|--------|------|
| users → presets | 1 対 N | 1ユーザーは複数のプリセットを持てる |
| presets → design_settings | 1 対 1 | 1プリセットは1つのデザイン設定を持つ |

---

## 4. ER図（テキスト表現）

```
users
├── id (PK)
├── email
├── password
├── username
├── is_logged_in
├── created_at
└── updated_at
      │
      │ 1:N
      ▼
presets
├── id (PK)
├── user_id (FK → users.id)
├── name
├── settings (JSONB)
├── is_deleted
├── created_at
└── updated_at
      │
      │ 1:1
      ▼
design_settings
├── id (PK)
├── preset_id (FK → presets.id)
├── line_color
├── line_width
├── line_type
├── background_color
├── graph_type
├── sensitivity
├── animation_speed
├── effect_type
├── is_default
├── created_at
└── updated_at
```

---

## 5. 補足事項

| 項目 | 内容 |
|------|------|
| 認証方式 | Supabase Authを使用（usersテーブルはauth.usersと連携） |
| 設定の保存形式 | presetsのsettingsカラムにJSONBで一括保存することも可 |
| 論理削除 | presetsのis_deletedフラグで管理し、物理削除は行わない |
| 音声データ | サーバーに保存せずブラウザ上で完結させることでストレージコストを削減 |