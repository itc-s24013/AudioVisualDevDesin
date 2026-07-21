# テーブル定義書

対象: Webオーディオビジュアライザー

---

## 1. users(ユーザー情報)

Google認証(OAuth)を採用するため、パスワードは自前で保持せず、Supabase Auth(`auth.users`)と連携する。本テーブルはGoogleアカウントから取得できるプロフィール情報とアプリ独自の付加情報を保持する。

| No | 物理名 | 論理名 | 型 | 桁数 | NULL | 主キー | 外部キー | デフォルト | 制約 | 説明 |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | id | ユーザーID | UUID | 36 | NOT NULL | ○ | ○ | 自動生成 | FK → auth.users.id | ユーザーの一意識別子(Supabase Authと同一ID) |
| 2 | email | メールアドレス | TEXT | 255 | NOT NULL | | | | UNIQUE | Googleアカウントのメールアドレス(認証時に取得・同期) |
| 3 | username | ユーザー名 | TEXT | 50 | NOT NULL | | | | | ユーザーの表示名(Googleアカウント名を初期値として使用可) |
| 4 | avatar_url | プロフィール画像URL | TEXT | 512 | NULL可 | | | NULL | | Googleアカウントのプロフィール画像URL |
| 5 | auth_provider | 認証方式 | TEXT | 20 | NOT NULL | | | 'google' | | 認証プロバイダ種別(将来的な他プロバイダ追加を考慮) |
| 6 | is_logged_in | ログイン状態 | BOOLEAN | - | NOT NULL | | | FALSE | | ログイン・ログアウト状態を管理(セッション管理) |
| 7 | last_login_at | 最終ログイン日時 | TIMESTAMP | - | NULL可 | | | NULL | | 直近のGoogle認証ログイン日時 |
| 8 | created_at | 作成日時 | TIMESTAMP | - | NOT NULL | | | CURRENT_TIMESTAMP | | アカウント作成日時(初回Google認証時に自動生成) |
| 9 | updated_at | 更新日時 | TIMESTAMP | - | NOT NULL | | | CURRENT_TIMESTAMP | | 最終更新日時(自動更新) |

---

## 2. presets(設定保存・管理)

| No | 物理名 | 論理名 | 型 | 桁数 | NULL | 主キー | 外部キー | デフォルト | 制約 | 説明 |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | id | プリセットID | UUID | 36 | NOT NULL | ○ | | 自動生成 | | プリセットの一意識別子 |
| 2 | user_id | ユーザーID | UUID | 36 | NOT NULL | | ○ | | FK → users.id | 所有ユーザーのID |
| 3 | name | プリセット名 | TEXT | 100 | NOT NULL | | | | | プリセット名(例:「新曲A用」) |
| 4 | is_deleted | 削除フラグ | BOOLEAN | - | NOT NULL | | | FALSE | | 論理削除フラグ |
| 5 | created_at | 作成日時 | TIMESTAMP | - | NOT NULL | | | CURRENT_TIMESTAMP | | プリセット作成日時(自動生成) |
| 6 | updated_at | 更新日時 | TIMESTAMP | - | NOT NULL | | | CURRENT_TIMESTAMP | | 最終更新日時(自動更新) |

デザイン内容の実体は`design_settings`テーブルが保持し、本テーブルはユーザーとの紐づけ・命名のみを担う(重複保持しない)。

---

## 3. design_settings(デザイン設定)

プリセットのデザイン内容の実体を保持するテーブル。`preset_id`にUNIQUE制約を付け、`presets`との1対1関係を担保する。`graph_type` / `effect_type`はWeb Audio API(`AnalyserNode`)の実装区分と一致させるためCHECK制約で値を固定している。

| No | 物理名 | 論理名 | 型 | 桁数 | NULL | 主キー | 外部キー | デフォルト | 制約 | 説明 |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | id | 設定ID | UUID | 36 | NOT NULL | ○ | | 自動生成 | | 設定の一意識別子 |
| 2 | preset_id | プリセットID | UUID | 36 | NOT NULL | | ○ | | FK → presets.id・UNIQUE | 紐づくプリセットのID(1対1を担保) |
| 3 | line_color | 線の色 | TEXT | 7 | NOT NULL | | | | CHECK(HEX形式) | カラーコード(HEX)。Canvas描画の`strokeStyle`に使用 |
| 4 | line_width | 線の太さ | INTEGER | - | NOT NULL | | | 2 | CHECK(1〜20) | px単位。Canvas描画の`lineWidth`に使用 |
| 5 | line_type | 線の種類 | TEXT | 20 | NOT NULL | | | 'solid' | CHECK IN ('solid', 'dashed') | 実線 / 点線。Canvas描画の`setLineDash()`に対応 |
| 6 | background_color | 背景色 | TEXT | 7 | NOT NULL | | | | CHECK(HEX形式) | カラーコード(HEX) |
| 7 | graph_type | グラフの形 | TEXT | 20 | NOT NULL | | | 'waveform' | CHECK IN ('waveform', 'frequency', 'circular') | `AnalyserNode.getByteTimeDomainData()`(波形)/ `getByteFrequencyData()`(周波数)の使い分けに対応。'circular'は周波数データを円形に描画するアプリ独自の表示モード |
| 8 | fft_size | FFTサイズ | INTEGER | - | NOT NULL | | | 2048 | CHECK IN (32,64,128,256,512,1024,2048,4096,8192,16384,32768) | `AnalyserNode.fftSize`に直接対応。周波数分解能を決定(2のべき乗のみ) |
| 9 | smoothing_time_constant | スムージング係数 | FLOAT | - | NOT NULL | | | 0.8 | CHECK(0.0〜1.0) | `AnalyserNode.smoothingTimeConstant`に対応。値が高いほど反応が滑らかになる(=体感的な「感度」を構成する要素) |
| 10 | sensitivity | 感度設定 | FLOAT | - | NOT NULL | | | 1.0 | CHECK(0.1〜5.0) | 重低音・BPMへの反応強度。`AnalyserNode`から取得した値に乗算する係数としてアプリ側で使用(`minDecibels`/`maxDecibels`ではなく描画時のスケーリング係数として扱う) |
| 11 | animation_speed | アニメーション速度 | FLOAT | - | NOT NULL | | | 1.0 | CHECK(0.1〜3.0) | 波形アニメーションの速さ(requestAnimationFrame内の描画更新倍率) |
| 12 | effect_type | エフェクトの種類 | TEXT | 20 | NOT NULL | | | 'none' | CHECK IN ('none', 'glow', 'gradient', 'particle') | グロー / グラデーション / パーティクル。Canvas描画時の追加エフェクト区分 |
| 13 | is_default | デフォルトデザインフラグ | BOOLEAN | - | NOT NULL | | | FALSE | | デフォルトデザインかどうか |
| 14 | created_at | 作成日時 | TIMESTAMP | - | NOT NULL | | | CURRENT_TIMESTAMP | | 作成日時(自動生成) |
| 15 | updated_at | 更新日時 | TIMESTAMP | - | NOT NULL | | | CURRENT_TIMESTAMP | | 最終更新日時(自動更新) |

---

## 補足

- 音声・再生関連(音声ファイル、再生位置、再生ステータス、音量など)はブラウザ上のWeb Audio APIおよびReactのstateで処理し、DBには永続化しない。`AudioContext`はブラウザの自動再生ポリシーにより、SCR-03の再生ボタン押下などユーザー操作を起点に生成・resumeする必要がある。
- リレーション: `users` 1 : N `presets`、`presets` 1 : 1 `design_settings`
- 論理削除は `presets.is_deleted` で管理し、物理削除は行わない。
- **認証方式**: Google OAuth 2.0によるソーシャルログインを採用し、Supabase Authを使用する。
  - パスワードはアプリ側で保持せず、認証はGoogle側に委譲する。
  - `users.id` は `auth.users.id` と同一のUUIDを使用し、初回ログイン時にトリガー等で `users` テーブルへレコードを自動作成する想定。
  - `email` / `username` / `avatar_url` はGoogleアカウントの情報を初期値として同期し、`username` はアプリ内で変更可能とする。
  - `auth_provider` は現状 `google` 固定だが、将来的に他のOAuthプロバイダ(例:GitHub、X等)を追加する場合に備えて保持する。