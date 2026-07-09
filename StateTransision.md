# DB状態遷移図 - Webオーディオビジュアライザー

## 1. usersテーブル

```mermaid
stateDiagram-v2
    [*] --> 未登録

    未登録 --> 登録済み : INSERT\n新規登録
    登録済み --> 登録済み : UPDATE\nメールアドレス変更\nパスワード変更\nユーザー名変更
    登録済み --> 削除済み : DELETE\nアカウント削除

    削除済み --> [*]
```

---

## 2. presetsテーブル

```mermaid
stateDiagram-v2
    [*] --> 未作成

    未作成 --> 有効 : INSERT\nプリセット新規作成\nis_deleted=FALSE

    有効 --> 有効 : UPDATE\nプリセット名変更\n設定内容変更
    有効 --> 論理削除済み : UPDATE\nis_deleted=TRUE\n削除操作

    論理削除済み --> 有効 : UPDATE\nis_deleted=FALSE\n削除取り消し
    論理削除済み --> [*] : DELETE\n物理削除
```

---

## 3. design_settingsテーブル

```mermaid
stateDiagram-v2
    [*] --> 未作成

    未作成 --> 保存済み : INSERT\nプリセット作成時に同時生成

    保存済み --> 保存済み : UPDATE\n線の色変更\n線の太さ変更\n線の種類変更\n背景色変更\nグラフの形変更\n感度設定変更\nアニメーション速度変更\nエフェクト種類変更\nis_default変更

    保存済み --> 削除済み : DELETE\n紐づくpresetsが物理削除された場合

    削除済み --> [*]
```

---

## 4. テーブル間の状態連携

```mermaid
stateDiagram-v2
    state users {
        u1 : 登録済み
        u2 : 削除済み
    }

    state presets {
        p1 : 有効\nis_deleted=FALSE
        p2 : 論理削除済み\nis_deleted=TRUE
    }

    state design_settings {
        d1 : 保存済み
        d2 : 削除済み
    }

    u1 --> p1 : プリセット作成
    p1 --> d1 : プリセット作成時\n同時にINSERT
    p1 --> p2 : 削除操作
    u2 --> p2 : ユーザー削除時\n関連プリセットも論理削除
    p2 --> d2 : 物理削除時\n関連レコードも削除
```