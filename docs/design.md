# 設計メモ

## コンセプト
架空のレコードショップ「路地灯」の経営シミュレーター（PCブラウザ向け）。
最初は自分で接客し、稼いで品揃えを増やし、擬似PCで店員を雇って店を回せるようにしていく。放置ゲームではない。

- 営業は最初 10:00〜15:00（ゲーム内1分 ≒ 3.3秒、実時間で約17分）
- 「店舗設備」で営業時間を1時間ずつ延長でき、最大 10:00〜19:00（実時間30分）
- 開店前は時間停止。準備が済んだら開店。閉店は19:00のみ（途中で閉められない）
- 閉店後に精算 → 翌日の開店前へ
- タブを離れる・アプリを閉じると一時停止
- 擬似PCは営業中も触れる。PCを開いている間は店頭が無人になり、客の我慢が1.5倍の速さで減る
- 仕入れは当日即時入荷。卸サイトには店で未取扱の新譜もあり、仕入れると棚に並ぶ（品揃えが増える）

## Web UI要素の割り当て
| 要素 | 使いどころ |
|---|---|
| ハンバーガーメニュー | 卸サイトのナビ（レーベル別アーティスト一覧） |
| パンくずリスト | 店内 ＞ 棚 ＞ アルバム／仕入れサイトの階層 |
| カルーセル | 卸サイトの特集（自動送り・矢印・ドット） |
| モーダル | アルバム詳細、面接、精算 |
| アコーディオン | 収録曲、店舗設備（アップグレード） |
| ラジオボタン | 求人条件（v0.3） |
| プルダウン | ジャンル・並び順の絞り込み、仕入れ数量、店員の担当 |
| ページネーション | 棚の中、応募者一覧 |
| キネティック・タイポ | 客のセリフ、売れた瞬間のスタンプ |
| UI/UXアニメーション | ボタン、プルダウン、モーダルの動き |
| インフォグラフィック | 精算の時間帯別来店、売上管理ソフト（v0.4） |

## ER図
```mermaid
erDiagram
  SHOP ||--o{ CRATE : "持つ"
  CRATE ||--o{ STOCK : "並ぶ"
  ALBUM ||--o{ STOCK : "在庫"
  ARTIST ||--o{ ALBUM : "出す"
  ALBUM ||--o{ TRACK : "収録"
  ALBUM }o--o{ TAG : "ムード"
  CUSTOMER_TYPE }o--o{ TAG : "好む"
  CUSTOMER_TYPE ||--o{ VISIT : "来店"
  VISIT }o--o| ALBUM : "買う"
  STAFF ||--o{ VISIT : "対応"
  APPLICANT ||--o| STAFF : "採用"
  SHOP ||--o{ DAY_REPORT : "精算"
  SHOP ||--o{ PURCHASE : "仕入れ"
  ALBUM ||--o{ PURCHASE : ""
  SHOP }o--o{ UPGRADE : "購入"
  SHOP ||--o{ MAIL : "受信"
  SHOP { int day int money int reputation string phase int clock_min int hours_lv }
  CRATE { string id PK string name int capacity }
  STOCK { int album_id FK string crate_id FK int slot int qty }
  ALBUM { int id PK int artist_id FK string title int year int cost int price bool carried }
  ARTIST { int id PK string name string genre string color }
  TRACK { int album_id FK int no string title }
  TAG { string name PK }
  CUSTOMER_TYPE { int id PK string name int patience int budget }
  VISIT { int id PK int day int time string request int staff_id FK bool satisfied }
  STAFF { int id PK string name int wage string skill_genre int accuracy }
  APPLICANT { int id PK string name int wage_ask string profile }
  DAY_REPORT { int day PK int sales int visitors int lost }
  PURCHASE { int day int album_id FK int qty }
  UPGRADE { int id PK string name int cost string effect }
  MAIL { int id PK int day string from string subject bool read }
```

ALBUM（商品カタログ）と STOCK（棚に並んだ実物）を分けている。仕入れはカタログから買い、STOCKに入る。`carried`（取扱中か）が立ったアルバムだけが店の棚に出る。営業時間の延長は UPGRADE の1つ目（`hours_lv`）。

## ロードマップ
- [x] **v0.1 店頭と時計**：開店前→営業→精算の流れ、手売り（パンくず・プルダウン・ページネーション・モーダル・アコーディオン）、客のセリフのキネティック・タイポ、保存
- [x] **v0.2 擬似PC**：デスクトップ、卸サイト（ハンバーガー・カルーセル・パンくず・数量プルダウン・カート・即時入荷・新譜の取扱開始）、店舗設備（営業時間の延長）、PC中は店頭が無人
- [ ] **v0.3 雇用**：求人サイト（ページネーション・ラジオ・プルダウン・面接モーダル）、店員の自動接客と時給
- [ ] **v0.4 経営の見える化**：売上管理ソフト（インフォグラフィック）、メール
- [ ] **v0.5 成長要素**：アップグレード、棚の追加、客タイプの解放、常連化
- [ ] **以降**：バランス調整、フロア追加・2号店など
