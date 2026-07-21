# ER図

対象: Webオーディオビジュアライザー

```mermaid
erDiagram
    USERS ||--o{ PRESETS : "作成する"
    PRESETS ||--|| DESIGN_SETTINGS : "持つ"

    USERS {
        uuid id PK
        text email
        text username
        text avatar_url
        text auth_provider
        boolean is_logged_in
        timestamp last_login_at
        timestamp created_at
        timestamp updated_at
    }

    PRESETS {
        uuid id PK
        uuid user_id FK
        text name
        boolean is_deleted
        timestamp created_at
        timestamp updated_at
    }

    DESIGN_SETTINGS {
        uuid id PK
        uuid preset_id FK
        text line_color
        integer line_width
        text line_type
        text background_color
        text graph_type
        integer fft_size
        float smoothing_time_constant
        float sensitivity
        float animation_speed
        text effect_type
        boolean is_default
        timestamp created_at
        timestamp updated_at
    }
```

## リレーション

| リレーション | 多重度 | 説明 |
|---|---|---|
| USERS → PRESETS | 1 対 N | 1ユーザーは複数のプリセットを作成できる |
| PRESETS → DESIGN_SETTINGS | 1 対 1 | 1プリセットは1つのデザイン設定を持つ(`design_settings.preset_id`にUNIQUE制約) |

## 補足

- `USERS.id`はSupabase Auth(`auth.users.id`)と同一のUUIDを使用し、Google OAuth認証で連携する。
- `PRESETS`はユーザーとの紐づけ・命名のみを担い、デザイン内容の実体は`DESIGN_SETTINGS`のみが保持する(重複データを持たない)。
- 音声ファイル・再生位置・再生ステータスなどはブラウザ上のWeb Audio APIで処理し、DBには永続化しないためER図には含まれない。