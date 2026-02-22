# Kotlin SpringBoot Clean Architecture

## 目的

Clean Architecture に従った、SpringBoot アプリケーションのスケルトンを作成。

## 前提条件

- DockerDesktop がインストール済みであること
  アプリケーションの起動には、原則としてdocker composeを使用
- MacOSでしか動作保証をしていない
- jdk のインストールや切り替えにはjenvを使用

## 環境変数

- SPRING_DATASOURCE_URL: SpringBoot アプリケーションの接続先DB
- SPRING_DATASOURCE_USERNAME: DB接続ユーザー
- SPRING_DATASOURCE_PASS: DB接続パスワード
- MYSQL_ROOT_PASSWORD: dbコンテナ起動時、MySQLサーバーに接続する際に使用するrootユーザーのパスワード
- MYSQL_DATABASE: dbコンテナ起動時に接続するデータベース
- MYSQL_USER: dbコンテナ起動時に、MySQLサーバーに接続する際に使用するユーザー
- MYSQL_PASSWORD: dbコンテナ起動時に、MySQLサーバーに接続する際に使用するパスワード

## アーキテクチャ

### レイヤー

---

#### InterfaceAdapter 層

REST コントローラや presenter など、フレームワーク固有の入出力をアプリケーション内部のデータ構造に変換する責務を持つ。

代表的なモジュール

- Controller
- Repository
- Presenter
- 外部サービス呼び出しの窓口 E.g: StorageService/EmailService

#### Application 層（≒ UseCase 層）

ユースケースを実装し、Domain 層のビジネスルールを組み合わせて処理を調整する。InterfaceAdapter
層が利用するポート（インターフェース）の定義もここに配置して境界を明示する。

#### Domain 層

エンティティや値オブジェクト、ドメインサービスなどアプリケーションの中核となるビジネスルールを保持し、外部要素への依存を持たない。

代表的なモジュール

- Entity
- ValueObject
- DomainService

#### Infra 層

DB、メッセージング、外部 API などの具体的な実装を提供し、Application 層で定義されたポートを満たすアダプタを実装する。

代表的なモジュール

- APIClient
- ORM

### ディレクトリ構成

---

```
src/
  ├─ main/
  │  ├─ kotlin/com.github.yukikariya.kotlin_spring_boot_clean_architecture
  │  │   ├─ domain/
  │  │   │  ├─ entities/                        # エンティティ
  │  │   │  ├─ value_objects/                   # 値オブジェクト
  │  │   │  └─ domain_services/                 # ドメインサービス
  │  │   ├─ application/
  │  │   │  ├─ use_cases/
  │  │   │  │  └─ <use_case>/
  │  │   │  │     ├─ boundary.kt                # 境界(InputData/OutputData/InputBoundary)
  │  │   │  │     └─ <use case>Interactor.py    # ユースケース実装
  │  │   │  └─ ports/
  │  │   │     ├─ repositories/                 # レポジトリポート
  │  │   │     │  └─ I<repository name>.kt      # 集約ルート単位で作成
  │  │   │     ├─ unit_of_works/                # UoWポート
  │  │   │     └─ <external service>/           # 外部サービスポート
  │  │   ├─ interface_adapter/
  │  │   │  ├─ controller/                      # Controller 実装群(InputData をInputBoundary に流す)
  │  │   │  │  └─ <controller name>/
  │  │   │  │     ├─ <controller name>.kt
  │  │   │  │     └─ I<controller name>.kt      # Application 層の InputData を使用し、Controller インターフェースを定義
  │  │   │  ├─ presenters/                      # Presenter 実装群(OutputDate を受け取り ViewModel を返す)
  │  │   │  │  └─ <presenter name>/             
  │  │   │  │     ├─ <presenter name>.kt/
  │  │   │  │     └─ I<presenter name>/         # Application 層の OutputData を使用し、Presenter インターフェースを定義
  │  │   │  ├─ repositories/
  │  │   │  │  └─ <repository name>/            # 集約ルート単位で作成。 Application層で定義されたポート定義で実装
  │  │   │  │     ├─ default.py                 
  │  │   │  │     └─ <variant>.py               
  │  │   │  └─ external_services/
  │  │   │        └─ <external service name>/   # 外部サービス実装。 Application層で定義されたポート定義で実装
  │  │   │           ├─ default.py              # 既定実装
  │  │   │           └─ <variant>.py            # バリアント実装
  │  │   ├─ infra/
  │  │   │    ├─ db/
  │  │   │    │  └─ <variant>/                  # データベース毎に実装を分ける(例: mysql, dynamodb)
  │  │   │    │     └─ <table_name>/            # テーブル別のORM
  │  │   │    └─ errors/
  │  │   │       └─ <error>.py                  # エラー
  │  │   └─ Dockerfile
     └─ resources/
        └─ db/
           └─ migration/
              └─ V1__{explanation}.sql/         # Flyway の命名規則に従う

```