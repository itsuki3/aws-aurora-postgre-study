# Amazon Redshift ハンズオン - Day1

# 1. 環境構築準備
## 1-1. ハンズオンで利用するアカウント
* AWSプロフェッショナルサービスにて用意した以下のアカウントをご利用下さい。

|no.|アカウントID|管理者|
|---:|:---|:---|
|1||AWS仲谷|
|2||AWS仲谷|
|3||AWS仲谷|
|4||AWS関口|
|5||AWS関口|
|6||AWS関口|

* IAMユーザー名、パスワード、アカウントIDは当日お知らせします。
* ハンズオンでは **東京リージョン** をご利用ください。

## 1-2. CloudFormationによるハンズオン環境の構築
* 今回使用するCloudFormationテンプレート（URL）、SQL等は、本マークダウン中から直接コピーして下さい。

### 1-2-1. VPC
* 以下の手順でCloudFormationスタックを作成します。
  * 「サービス」 → 「CloudFormation」 → 「スタックの作成」→
  * 「新しいスタックの作成」ボタン→　
  * 「Amazon S3 テンプレート URL の指定」チェックボックス→
   * 以下を指定 https://s3-ap-northeast-1.amazonaws.com/my-bucket-4-handson/Cfn/xsr-version/vpc.yml
```
スタックの名前   : vpcXXXXX　(XXXXXは任意の文字列、3~5文字程度)　
Env Type      : Dev
Project Name  : myproject
```
  * 「次へ」ボタン →　オブション入力画面はデフォルトのまま「次へ」ボタン→
  * 確認画面　→　「作成」ボタン
* ステータスが "CREATE_COMPLETE"　になるまで待ちます。
* 以下のVPCおよび関連リソースが構築されます。
```
myvpc
```

### 1-2-2. Redshift
* VPCの作成が完了したら、同様の手順でRedshiftクラスター用のCloudFormationスタックを作成します。
  * 「サービス」 → 「CloudFormation」 → 「スタックの作成」→
  * 「新しいスタックの作成」ボタン →　
  * 「Amazon S3 テンプレート URL の指定」チェックボックス →
   * 以下を指定　https://s3-ap-northeast-1.amazonaws.com/my-bucket-4-handson/Cfn/xsr-version/redshift.yml
```
スタックの名前     : redshiftXXXXX　(XXXXXは任意の文字列、3~5文字程度)　
Env Type        : Dev
Project Name    : myproject
Common Password : xxxxxx (8文字以上、半角英数字大文字小文字いずれも１文字以上含む)を入力　→
```
  * 「次へ」ボタン →　オブション入力画面はデフォルトのまま「次へ」ボタン　→
  *  確認画面の最下部の「AWS CloudFormation によってカスタム名のついた IAM リソースが作成される場合があることを承認します。」チェックボックスをチェック　→　「作成」ボタン
  * ***パスワードは後で使用しますので、記録しておいて下さい。***

* ステータスが "CREATE_COMPLETE"　になるまで待ちます。
* 以下のRedshiftクラスターおよび関連リソースが構築されます。
```
redshiftXXXXX-redshiftcluster-ランダムな文字列
```

# 2. 接続確認

## 2-1. Management Console（MC）からクラスターの詳細情報を確認
* 「サービス」→「Redshift」→「クラスター」タブに移動します。
* クラスターが作成されていることを確認します。
* 「クラスター」→当該クラスターをクリックすると、クラスターエンドポイントを始め、そのクラスターに関する詳細情報を閲覧できます。「設定」タブ、「ステータス」タブをクリックして、それぞれどのような情報が表示されるか確認してみて下さい。

## 2-2. データベースへの接続確認
### 2-2-1 接続設定の作成
* PC上にインストールされている、Aginity Workbench for Redshift（以下Aginity）を起動します。
* 接続情報を以下の通り入力します。
```
Name      : （任意）
Server    : （クラスターエンドポイントのURL。ポートを除く）
DB Port   : 5439
SSL Mode  : Prefer
User ID   : awsuser
Password  : （上記で入力したパスワード）
```
* Redshiftクラスターのエンドポイント名は、クラスター詳細情報（2-1.）からコピペして下さい（ポート番号はペースト後、Backspaceで削除して下さい）。

### 2-2-2 接続テスト
* 入力した接続情報に基づいて、実際にRedshiftクラスターに接続します。
* 接続したら、Aginityからawsuser.mydbスキーマをクリックし、pg_catarogデータベースをざっと外観してみて下さい。
  * ここにはシステムテーブル（STL_XXX、STV_XXX）やシステムビュー（SVL_XXX、SVV_XXX）が格納されています。運用にあたって、これらのテーブルやビューはしばしば参照することになります。

# 3. ハンズオン
* ここでは、Redshiftの基本的な使い方についてハンズオンを行っていきます。
  * **sh1**という1,500万件ほどの小型のスキーマをサンプルに使っていきます。

## 3-1. スキーマの作成
* AginityからDDLを実行し、スキーマを作成します。
  * Aginityに接続
  * 上部のバーでDatabaseに ***mydb*** を選択
  * 以下をコピペして実行

```
CREATE SCHEMA IF NOT EXISTS sh1;

CREATE TABLE IF NOT EXISTS sh1.channels(
  channel_id NUMERIC(38,0) NOT NULL,
  channel_desc VARCHAR(20) NOT NULL,
  channel_class VARCHAR(20) NOT NULL,
  channel_class_id NUMERIC(38,0) NOT NULL,
  channel_total VARCHAR(13) NOT NULL,
  channel_total_id NUMERIC(38,0) NOT NULL
)
DISTSTYLE ALL
;

CREATE TABLE IF NOT EXISTS sh1.countries(
  country_id NUMERIC(38,0) NOT NULL,
  country_iso_code VARCHAR(2) NOT NULL,
  country_name VARCHAR(40) NOT NULL,
  country_subregion VARCHAR(30) NOT NULL,
  country_subregion_id NUMERIC(38,0) NOT NULL,
  country_region VARCHAR(20) NOT NULL,
  country_region_id NUMERIC(38,0) NOT NULL,
  country_total VARCHAR(11) NOT NULL,
  country_total_id NUMERIC(38,0) NOT NULL,
  country_name_hist VARCHAR(40)
)
DISTSTYLE ALL
;

CREATE TABLE IF NOT EXISTS sh1.customers(
  cust_id NUMERIC(38,0) NOT NULL,
  cust_first_name VARCHAR(20) NOT NULL,
  cust_last_name VARCHAR(40) NOT NULL,
  cust_gender VARCHAR(1) NOT NULL,
  cust_year_of_birth SMALLINT NOT NULL,
  cust_marital_status VARCHAR(20),
  cust_street_address VARCHAR(40) NOT NULL,
  cust_postal_code VARCHAR(10) NOT NULL,
  cust_city VARCHAR(30) NOT NULL,
  cust_city_id NUMERIC(38,0) NOT NULL,
  cust_state_province VARCHAR(40) NOT NULL,
  cust_state_province_id NUMERIC(38,0) NOT NULL,
  country_id NUMERIC(38,0) NOT NULL,
  cust_main_phone_number VARCHAR(25) NOT NULL,
  cust_income_level VARCHAR(30),
  cust_credit_limit NUMERIC(38,0),
  cust_email VARCHAR(50),
  cust_total VARCHAR(14) NOT NULL,
  cust_total_id NUMERIC(38,0) NOT NULL,
  cust_src_id NUMERIC(38,0),
  cust_eff_from TIMESTAMP WITHOUT TIME ZONE,
  cust_eff_to TIMESTAMP WITHOUT TIME ZONE,
  cust_valid VARCHAR(1)
)
DISTSTYLE KEY DISTKEY (cust_id)
;

CREATE TABLE IF NOT EXISTS sh1.products(
  prod_id INTEGER NOT NULL,
  prod_name VARCHAR(50) NOT NULL,
  prod_desc VARCHAR(4000) NOT NULL,
  prod_subcategory VARCHAR(50) NOT NULL,
  prod_subcategory_id NUMERIC(38,0) NOT NULL,
  prod_subcategory_desc VARCHAR(2000) NOT NULL,
  prod_category VARCHAR(50) NOT NULL,
  prod_category_id NUMERIC(38,0) NOT NULL,
  prod_category_desc VARCHAR(2000) NOT NULL,
  prod_weight_class SMALLINT NOT NULL,
  prod_unit_of_measure VARCHAR(20),
  prod_pack_size VARCHAR(30) NOT NULL,
  supplier_id INTEGER NOT NULL,
  prod_status VARCHAR(20) NOT NULL,
  prod_list_price NUMERIC(8,2) NOT NULL,
  prod_min_price NUMERIC(8,2) NOT NULL,
  prod_total VARCHAR(13) NOT NULL,
  prod_total_id NUMERIC(38,0) NOT NULL,
  prod_src_id NUMERIC(38,0),
  prod_eff_from TIMESTAMP WITHOUT TIME ZONE,
  prod_eff_to TIMESTAMP WITHOUT TIME ZONE,
  prod_valid VARCHAR(1)
)
DISTSTYLE KEY DISTKEY (prod_id)
;

CREATE TABLE IF NOT EXISTS sh1.promotions(
  promo_id INTEGER NOT NULL,
  promo_name VARCHAR(30) NOT NULL,
  promo_subcategory VARCHAR(30) NOT NULL,
  promo_subcategory_id NUMERIC(38,0) NOT NULL,
  promo_category VARCHAR(30) NOT NULL,
  promo_category_id NUMERIC(38,0) NOT NULL,
  promo_cost NUMERIC(10,2) NOT NULL,
  promo_begin_date TIMESTAMP WITHOUT TIME ZONE NOT NULL,
  promo_end_date TIMESTAMP WITHOUT TIME ZONE NOT NULL,
  promo_total VARCHAR(15) NOT NULL,
  promo_total_id NUMERIC(38,0) NOT NULL
)
DISTSTYLE EVEN
;

CREATE TABLE IF NOT EXISTS sh1.sales(
  prod_id NUMERIC(38,0) NOT NULL,
  cust_id NUMERIC(38,0) NOT NULL,
  time_id TIMESTAMP WITHOUT TIME ZONE NOT NULL,
  channel_id NUMERIC(38,0) NOT NULL,
  promo_id NUMERIC(38,0) NOT NULL,
  quantity_sold NUMERIC(10,2) NOT NULL,
  seller INTEGER NOT NULL,
  fulfillment_center INTEGER NOT NULL,
  courier_org INTEGER NOT NULL,
  tax_country VARCHAR(3) NOT NULL,
  tax_region VARCHAR(3),
  amount_sold NUMERIC(10,2) NOT NULL
)
DISTSTYLE KEY DISTKEY (cust_id)
SORTKEY (time_id)
;

CREATE TABLE IF NOT EXISTS sh1.supplementary_demographics(
  cust_id NUMERIC(38,0) NOT NULL,
  education VARCHAR(21),
  occupation VARCHAR(21),
  household_size VARCHAR(21),
  yrs_residence NUMERIC(38,0),
  affinity_card BIGINT,
  bulk_pack_diskettes BIGINT,
  flat_panel_monitor BIGINT,
  home_theater_package BIGINT,
  bookkeeping_application BIGINT,
  printer_supplies BIGINT,
  y_box_games BIGINT,
  os_doc_set_kanji BIGINT,
  comments VARCHAR(4000)
)
DISTSTYLE KEY DISTKEY (cust_id)
;

CREATE TABLE IF NOT EXISTS sh1.times(
  time_id TIMESTAMP WITHOUT TIME ZONE NOT NULL,
  day_name VARCHAR(36) NOT NULL,
  day_number_in_month VARCHAR(2) NOT NULL,
  day_number_in_year VARCHAR(3) NOT NULL,
  calendar_year VARCHAR(4) NOT NULL,
  calendar_quarter_number VARCHAR(1) NOT NULL,
  calendar_month_number VARCHAR(2) NOT NULL,
  calendar_week_number VARCHAR(2) NOT NULL,
  calendar_month_desc VARCHAR(7) NOT NULL,
  calendar_quarter_desc VARCHAR(6) NOT NULL
)
DISTSTYLE ALL
;
```
  * テーブルの作成が完了したら、その他のオブジェクトを作成

```
ALTER TABLE SH1.CHANNELS
  ADD CONSTRAINT CHANNELS_PK PRIMARY KEY ( CHANNEL_ID )
;

ALTER TABLE SH1.COUNTRIES
  ADD CONSTRAINT COUNTRIES_PK PRIMARY KEY ( COUNTRY_ID )
;

ALTER TABLE SH1.CUSTOMERS
  ADD CONSTRAINT CUSTOMERS_PK PRIMARY KEY ( CUST_ID )
;

ALTER TABLE SH1.PRODUCTS
  ADD CONSTRAINT PRODUCTS_PK PRIMARY KEY ( PROD_ID )
;

ALTER TABLE SH1.PROMOTIONS
  ADD CONSTRAINT PROMOTIONS_PK PRIMARY KEY ( PROMO_ID )
;

ALTER TABLE SH1.SALES
  ADD CONSTRAINT SALES_UK UNIQUE ( PROD_ID , CUST_ID , PROMO_ID , CHANNEL_ID , TIME_ID )
;

ALTER TABLE SH1.SUPPLEMENTARY_DEMOGRAPHICS
  ADD CONSTRAINT SUPPLEMENTARY_DEMOGRAPHICS_PK PRIMARY KEY ( CUST_ID )
;

ALTER TABLE SH1.TIMES
  ADD CONSTRAINT TIMES_PK PRIMARY KEY ( TIME_ID )
;

ALTER TABLE SH1.CUSTOMERS
  ADD CONSTRAINT CUST_COUNTRIES_FK FOREIGN KEY ( COUNTRY_ID )
  REFERENCES SH1.COUNTRIES ( COUNTRY_ID )
;

ALTER TABLE SH1.SALES
  ADD CONSTRAINT SALES_CHANNELS_FK FOREIGN KEY ( CHANNEL_ID )
  REFERENCES SH1.CHANNELS ( CHANNEL_ID )
;

ALTER TABLE SH1.SALES
  ADD CONSTRAINT SALES_CUSTOMERS_FK FOREIGN KEY ( CUST_ID )
  REFERENCES SH1.CUSTOMERS ( CUST_ID )
;

ALTER TABLE SH1.SALES
  ADD CONSTRAINT SALES_PRODUCTS_FK FOREIGN KEY ( PROD_ID )
  REFERENCES SH1.PRODUCTS ( PROD_ID )
;

ALTER TABLE SH1.SALES
  ADD CONSTRAINT SALES_PROMOTIONS_FK
  FOREIGN KEY ( PROMO_ID )
  REFERENCES SH1.PROMOTIONS ( PROMO_ID )
;

ALTER TABLE SH1.SALES
  ADD CONSTRAINT SALES_TIMES_FK FOREIGN KEY ( TIME_ID )
  REFERENCES SH1.TIMES ( TIME_ID )
;
```
* Aginityからsh1スキーマを確認して下さい。

## 3-2. COPYの実行
* COPYコマンドを実行し、Redshiftクラスターにsh1スキーマのデータを投入します。
 * 日々のバッチの結果Redshiftにロードされるデータも、同じ方法で投入されます。
* ***IAM_ROLEの引数は、シングルクォートで囲った上で、以下のロールARN形式に記載下さい。***
  * arn:aws:iam::12桁のアカウントID:role/ロール名
* ロールARNは、「サービス」→「IAM」→「ロール」から、スタック名を接頭辞として見つけ、一連のテキストとしてコピーすることができます。
  * または、「サービス」→「Redshift」→「クラスター」→（当該クラスターのチェックボックスをチェック）→「IAMロールの管理」から確認することもできます。

```
COPY sh1.channels
FROM 's3://my-bucket-4-handson/Data/redshift/data/sh1/channels/sh1_channels_manifest'
IAM_ROLE '作成されたロールARN'
DELIMITER '|'
GZIP
MANIFEST
MAXERROR 1
;

COPY sh1.countries
FROM 's3://my-bucket-4-handson/Data/redshift/data/sh1/countries/sh1_countries_manifest'
IAM_ROLE '作成されたロールARN'
DELIMITER '|'
GZIP
MANIFEST
MAXERROR 1
;

COPY sh1.customers
FROM 's3://my-bucket-4-handson/Data/redshift/data/sh1/customers/sh1_customers_manifest'
IAM_ROLE '作成されたロールARN'
DELIMITER '|'
GZIP
MANIFEST
MAXERROR 1
;

COPY sh1.products
FROM 's3://my-bucket-4-handson/Data/redshift/data/sh1/products/sh1_products_manifest'
IAM_ROLE '作成されたロールARN'
DELIMITER '|'
GZIP
MANIFEST
MAXERROR 1
;

COPY sh1.promotions
FROM 's3://my-bucket-4-handson/Data/redshift/data/sh1/promotions/sh1_promotions_manifest'
IAM_ROLE '作成されたロールARN'
DELIMITER '|'
GZIP
MANIFEST
MAXERROR 1
;

COPY sh1.sales
FROM 's3://my-bucket-4-handson/Data/redshift/data/sh1/sales/sh1_sales_manifest'
IAM_ROLE '作成されたロールARN'
DELIMITER '|'
GZIP
MANIFEST
MAXERROR 1
;

COPY sh1.times
FROM 's3://my-bucket-4-handson/Data/redshift/data/sh1/times/sh1_times_manifest'
IAM_ROLE '作成されたロールARN'
DELIMITER '|'
GZIP
MANIFEST
MAXERROR 1
;
```

## 3-3. バックアップ（スナップショット）の取得
* Redshiftのバックアップはスナップショットに依拠しており、自動と手動のふたつの方法があります。
  * 自動スナップショットは8時間ごとまたは更新5GBごとに取得されます（いずれか早い方）。
  * 手動スナップショットはバッチの断面等で任意に取得できます。
* 本環境では手動スナップショットが想定されていますので、手動スナップショットを作成します。
  * 「スナップショット」→「スナップショットの作成」→
```
クラスター識別子    : 作成したクラスターの識別子
スナップショット識別子 : 任意
```
* 当該スナップショットをクリックし、作成状況やサイズを確認します。
  * データ量が少ないため、スナップショットの作成は1分程度で完了します。

## 3-4. メンテナンストラック、パラメーターグループ設定、WLM設定の閲覧
* 「クラスター」→（クラスター名）、および「ワークロード管理」→「パラメータ」、「ワークロード管理」→「WLM」から、当該クラスターの様々な情報を確認できます。
* 以下の情報を見つけてみて下さい。
  * メンテナンストラックの設定（Currentか、Trailingか）
  * パラメーターグループのうち、enable_user_activity_logging（監査証跡）の設定（有効か、無効か）
  * WLMのキュー数およびスロット数
* 不明な場合は講師にお尋ね下さい。

## 3-5. クエリーの実行
* SQLを実行し、sh1スキーマに投入したデータを確認します。

### 3-5-1 データ確認
* salesテーブルのデータを確認します。

```
SELECT
 prod_id,
 count(1) as deal_count,
 avg(quantity_sold) as average_sold_num,
 sum(quantity_sold*amount_sold) as total_sales
FROM
 sh1.sales
GROUP BY
 prod_id
ORDER BY
 total_sales DESC
;
```
### 3-5-2 テーブル確認
* salesテーブルの構造を確認します。

```
SET search_path TO '$user', 'public', 'sh1'
;

SELECT
 "column", type, encoding, distkey, sortkey, "notnull"
FROM
 pg_table_def
WHERE
 tablename = 'sales'
;
```
## 3-6. COPYおよびクエリーの実行状況の確認
* コンソールを使って、COPYおよびクエリーの実行状況を確認します。

### 3-6-1 COPY（ロード）の実行確認
* COPYコマンドの実行結果は、「クラスター」→（クラスター名）→「ロード」タブから確認できます。
  * タブをクリックして、先程実行したクエリーを確認してみて下さい。
  * 当該クエリーをクリックすると詳細を見ることができます。

### 3-6-2 クエリーの実行確認
* クエリーの実行結果は、「クラスター」→（クラスター名）→「クエリ」タブから確認できます。
  * タブをクリックして、先程実行したクエリーを確認してみて下さい。
  * 当該クエリーをクリックすると詳細を見ることができます。

### 3-6-3 クラスターの性能情報確認
* 「クラスター」→（クラスター名）→「クラスターのパフォーマンス」からクラスターのハードウェア性能情報、同じく「データベースのパフォーマンス」からクエリー性能情報を確認できます。
  * タブをクリックして、先程実行したクエリー時間帯の性能情報を確認してみて下さい。
  * 「いずれも、CloudWatchから得られたメトリクス情報を元にしています。

## 3.7. イベントの閲覧
* 「イベント」→「イベント」をクリックし、以下のフィルタリング条件を入力して、当該クラスターに関係したイベントのみを出力します。

```
フィルター : クラスター
文字列   : 作成したクラスター名（部分一致で可）
```

* 「イベント」→「サブスクリプション」→「イベントサブスクリプションの作成」をクリックし、新規イベントサブスクリプションを作成します。
  * サブスクリプションは、カテゴリー、重要度、リソース（クラスターなど）を指定した上で、イベント発生時にAmazon SNS経由で通知する仕組みです。
  * ここでは模擬的に、直接メールを送信する設定を行いますが、実際には作成しません。

```
カテゴリ     :　全てのカテゴリーを選択
重要度      :　エラー
ソースタイプ  :　すべて
名前        : 任意（4文字以上）
有効        :　はい

新しいトピックの作成を選択
名前        :　任意
送信先メール  :　任意
```
* 全ての項目を設定したら、**キャンセル**をクリックして画面を抜けます。

## 3.8. リストア
### 3.8.1 リストアの実行とクエリーの実行
* 3-5.で作成したスナップショットを使って、リストアを行います。
* ここではコマンドライン（AWSCLI）を使った方法を試してみます。
  * ***講師からアクセスキーとシークレットキーの情報を受け取っていない場合は、このステップはスキップし、3.8.2に進んで下さい。***
* コマンドプロンプトを開き、AWSCLIを実行します。
  * アクセスキーとシークレットキーを構成
```
aws configure
```
  * リストアを実施
    * クラスターレベルでのリストアは、別クラスターの作成の形を取りますので、元のクラスターとは別の名前を指定して下さい。
```
aws redshift describe-snapshots --cluster-identifier 作成した元のクラスター名
aws redshift restore-from-cluster-snapshot --cluster-identifier 新しいクラスター名 --snapshot-identifier 前コマンドで確認した手動スナップショット識別子 --cluster-subnet-group-name public-subnetgroup --publicly-accessible
```

* 「クラスター」をクリックし、新しいクラスターがスナップショットから復元されていることを確認します。
* 復元中のクラスターをクリックし、Restore Statusを確認します。
* スナップショットから復元し、再度クエリーが実行可能になるまでには数分~十数分かかります（データ量に依存しますが、全量復旧ではない点に注意して下さい）。本ステップはここでいったん完了とし、次に進みます。

### 3.8.2. テーブルレベルの復元
* 元のクラスターに対して、テーブルレベルでの復元を施します。
 * Aginityから、以下のコマンドを実行

```
select count(1) from sh1.sales;
```
 * 件数を確認後、テーブルをトランケート

```
truncate table sh1.sales;
```

 * 再度件数を確認します。
 * MCから「クラスター」→（作成した元のクラスター）→「テーブルの復元」→「テーブルの復元」をクリック→
 * 以下の通り入力

```
時間の範囲 :　過去一週間
復元元のソーステーブル/データベース :　 mydb
復元元のソーステーブル/スキーマ : sh1　
復元元のソーステーブル/テーブル : sales　
復元先のターゲットテーブル/データベース:　 mydb
復元先のターゲットテーブル/スキーマ:　 sh1
復元先のターゲットテーブル/テーブル:　 sales_new
```

  * 同一テーブル名は、元のテーブルおよび依存するオブジェクト一式をドロップするまで使用できない点に注意して下さい。復元後に、オブジェクトのドロップと、復元したオブジェクトのリネーム、依存オブジェクトの再作成を行うことができます。
  * 復元後、Aginityから　復元テーブルの件数を確認（この操作はオプション）

```
select count(1) from sh1.sales_new;
```

# 4. ハンズオン環境の削除
* Redshiftハンズオン環境の削除は弊社にて行います。そのままにしておいていただいて結構です。
***


# Amazon Aurora PostgreSQL ハンズオン - Day1

# 4. 環境構築準備
* 本ハンズオンでは不要です。
* **コンソール右上で「東京」リージョンが選択されていることだけ、再度ご確認下さい。**

# 5. ハンズオン
* Aurora PostgreSQLクラスターの作成および基本的な操作を体験していただきます。

## 5-1. Aurora PostgreSQLクラスターの作成
* 「サービス」→「RDS」→「クラスター」→「データベースの作成」ボタン→「エンジンの選択」
→「Amazon Aurora」ラジオボタン→「PostgreSQL対応」ラジオボタン→「次へ」ボタンをクリックして「DB 詳細の指定」画面へ、各ペインで以下を設定・入力

* **特に入力・選択指示のない入力・選択項目は、設定されている値のままとしてください**

**インスタンスの仕様**
```
DBエンジンのバージョン : Aurora PostgreSQL (compatible with PostgreSQL 9.6.8)
DBインスタンスクラス : db.r4.large - 2 vcpu, 15.25 GiB RAM
マルチ AZ 配置 : 異なるゾーンにレプリカを作成
```

**設定**
```
DBインスタンス識別子 : ap96XXXXX (XXXXXは任意の文字列、3~5文字程度)を入力
マスターユーザの名前 : awsuser
マスターパスワード : xxxxxx （入力したパスワードは忘れないようご注意ください）
パスワードの確認　 : xxxxxx （マスターパスワードに同じ）
```
「次へ」ボタンをクリックし、「[詳細設定] の設定」へ

**[詳細設定] の設定**
```
Virtual Priavate Cloud (VPC) : デフォルト VPC
サブネットグループ : default
パブリックアクセシビリティ : はい
アベイラビリティゾーン : ap-northeast-1a
VPC セキュリティグループ : 新規の VPC セキュリティグループ を作成
```

**データベースの設定**
```
DBクラスター識別子 : ap96cXXXXX (XXXXXは任意の文字列、3~5文字程度)
データベースの名前 : mydb
ポート : 5432
DB パラメータグループ : default.autora-postgresql9.6
DB クラスターのパラメータグループ : default.autora-postgresql9.6
```

**暗号化**
```
暗号化 : 暗号を有効化
マスターキー : (default)aws/rds
```

**フェイルオーバー**
```
  優先度 : 範囲-2
```

**拡張モニタリング**
```
拡張モニタリング : 拡張モニタリングを有効にする
モニタリングロール : デフォルト
詳細度 : 60秒
```

**パフォーマンスインサイト**
```
パフォーマンスインサイトの有効化 : オン (ラジオボタンを選択)
保持期間 : デフォルト(7日)
マスターキー : (default)aws/rds
```

**削除保護**
```
削除保護の有効化 : オフ (チェックボックスを外す)
```

* 「データベースを作成」ボタン→「DBインスタンスの詳細表示」ボタンをクリック→サイトバーの「インスタンスタブ」リンクをクリックし、インスタンスペインへ
  * インスタンスタブで2つのDBインスタンスのステータスが **"作成中"** から **"利用可能"** に変化するまで待ちます。（10分程度）

* 各DBインスタンス名のリンクをクリックして、詳細情報を確認してみてください。
  * Replicationペインでは、2つのDBインスタンスが、書き込み(Writer)であるか、読み込み(Reader)であるか確認することができます。また、各DBインスタンスのエンドポイント等も確認することができます。

* 「サービス」→「RDS」→サイドバーの「クラスター」→「クラスター」タブに移動します。
* クラスターが作成され、「状況」が **"利用可能"** になっていることを確認します。
* 作成されているDBクラスター識別子のリンクをクリックして、クラスターの詳細情報を確認してみてください。
  * クラスターエンドポイント、読み込みエンドポイント等の詳細情報を確認できます。
  * 確認したエンドポイントをメモ帳などにコピーしておいてください。接続設定の作成で利用します。

### 5-1-2. アクセス元の制限

* 「サービス」→「RDS」→「インスタンス」→「インスタンス」ペイン →「DBインスタンス」にリストされている2つのDBのいずれか一つのリンクをクリック
  * スクロールダウンして「詳細」ペイン →「セキュリティグループ」のActive状態のセキュリティグループのリンクをクリック
  * 「EC2」ダッシュボード → 「セキュリティグループ」ペイン　→ 「インバウンド」タブ → 「編集」ボタン をクリック
  * 「インバウンドのルールの編集」ダイアログ → 「タイプ」が"PostgreSQL"となっているルールの「ソース」列 → 「マイIP」→ 「保存」ボタンをクリック

* **この操作で、Aurora PostgreSQLの各インスタンスは、現在操作中のPCのIPアドレスからのアクセスのみ受け付けるようになります**


## 5-2. データベースへの接続確認
### 5-2-1 接続設定の作成
* PC上にインストールされている **Psqledit** を起動します。 **クラスターエンドポイント向け接続設定** と **読み込みエンドポイント向け接続設定** の２つの設定を作成してください。
  * 接続設定に必要な情報は以下の通りです。

* **以下の手順で接続設定を定義します**
  * Psqledit起動 →「ログイン」ダイアログ →「ログイン情報の設定」ボタン →「ログイン情報」ダイアログ

* **HOSTに設定するクラスターエンドポイント及び、読み込みエンドポイントの確認方法は以下の通りです。**
  * 「サービス」→「RDS」→「クラスター」→「DBクラスター識別子」以下の該当クラスターのリンク→「詳細」ペインから確認できます。

**読み込みエンドポイント向け接続設定**
```
USER    : awsuser
PASS    : 5-1. Aurora PostgreSQLクラスターの作成"で設定した「マスターパスワード」
DBNAME  : 5-1. Aurora PostgreSQLクラスターの作成"で設定した「データベースの名前」
HOST    : 5-1. Aurora PostgreSQLクラスターの作成"で確認した読み込みエンドポイント
PORT    : 5432
OPTION  : 未入力
NAME    : 読み込みエンドポイント
```
  * 上記を入力後→「Add」ボタン→「OK」ボタンをクリック

**クラスターエンドポイント向け接続設定**
```
USER    : awsuser
PASS    : 5-1. Aurora PostgreSQLクラスターの作成"で設定した「マスターパスワード」
DBNAME  : 5-1. Aurora PostgreSQLクラスターの作成"で設定した「データベースの名前」
HOST    : 5-1. Aurora PostgreSQLクラスターの作成"で確認したクラスターエンドポイント
PORT    : 5432
OPTION  : 未入力
NAME    : クラスターエンドポイント
```
  * 上記を入力後→「Add」ボタン→「OK」ボタンをクリック


### 5-2-2 接続とテスト
* **作成したクラスターエンドポイント接続設定** と **読み込みエンドポイント向け接続設定** 、ぞれぞれ設定で各エンドポイントに接続できるか確認します。
* 以下の操作で残りの接続設定の確認を行います。（アプリケーション画面上部等に接続先設定のNAMEで指定した名称が表示されていることを確認してください）
  * Psqleditのメニュー →「ファイル」→「接続先の変更」→「login」ダイアログの「接続先の選択」ペインから **読み込みエンドポイント向け接続設定** をダブルクリック


* 以下のクエリーを「SQL編集」ペインへ入力後 ▶︎(SQL実行ボタン)をクリックし正常に実行されるか確認してください。
```
select version();
```

* 以下操作でアプリケーション画面上部等に接続先設定のNAMEで指定した名称が表示されていることを確認してください。
  * Psqleditの「login」ダイアログの「接続先の選択」ペインから作成した　**クラスターエンドポイント接続設定**　を選択 →「OK」ボタンをクリック→「SQL編集」ペイン

* 以下のクエリーを「SQL編集」ペインへ入力後 ▶︎(SQL実行ボタン)をクリックし正常に実行されるか確認してください。
```
select version();
```

* **これで接続確認は終了です。**


## 5-3. クラスターエンドポイントからの接続
* 5-2-1 で作成したクラスターエンドポイント接続設定を利用して接続し、以下の操作を行います。
  * 以下のコマンド、DDL、SQL文を実行し、クラスターエンドポイントから表作成、データ登録など書き込み操作が行えることを確認します。書き込み操作の確認が済んだらデータベースからdisconnectします。
```
create schema hoge;
set search_path=hoge;
select current_schema();
create table foobar (
  id numeric not null primary key
  ,name character(20) not null
  ,description character varying(40)
);
insert into foobar values(1,'test','test insertion');
select * from foobar;
```

## 5-4. 読み込みエンドポイントからの接続
* 2-2-1で作成した読み込みエンドポイント接続設定を利用して接続し、以下の操作を行います。
  * **以下のクエリーを実行してみてください。何が起こりましたか？**
```
select current_schema();
set search_path=hoge;
select current_schema();
select * from foobar;
```

  * **以下のクエリーを実行してみてください。何が起こりましたか？**
```
insert into foobar values(1,'test','test insertion');
```
* 読み込みエンドポイントへの接続時は、読み込み操作しか行えません。


## 5-5. スナップショットの取得
* Redshift同様、手動でスナップショットを取得します。
* 手動で取得したスナップショツトはデータベースを完全に削除した場合でも削除されません。

* 「サービス」→「RDS」→「サイドバー」の「 インスタンス」→ DBインスタンのロールが **書き込み** となっているDBインスタンス名横のラジオボタンをクリック →「インスタンスの操作」ドロップダウンメニュー →「スナップショットの取得」を選択して「DBスナップショットの取得」ペインへ
  * DBインスタンのロールが **書き込み** であることを確認するには、「サイドバー」→ 「クラスター」→ 「クラスターDB識別子」列下のDBクラスター名のリンク → 「DB クラスターメンバー」ペインのロール列を参照します
```
スナップショット名　: handson-aurora-snapshot
```
* 「スナップショットの取得」ボタンをクリック

* スナップショットの取得の完了を待っている間に、次のステップに進みます。


## 5-6. レプリカの追加
* **特に入力・選択指示のない入力・選択項目は、設定されている値のままとしてください**

* 「サイドバー」→「インスタンス」→ DBインスタンのロールが **書き込み** となっているDBインスタンス名横のラジオボタンをクリック →「インスタンスの操作」ドロップダウンメニュー →「Auroraレプリカの作成」画面へ

* DBインスタンのロールが **書き込み** であることを確認するには、「サイドバー」→ 「クラスター」→ 「クラスターDB識別子」列下のDBクラスター名のリンク → 「DB クラスターメンバー」ペインのロール列を参照します。

**ネットワーク & セキュリティ**
  * 「インスタンス」→ 「インスタンス」ペインのDBインスタンス列以下にリストされている2つのDBインスタンスのリンクをそれぞれクリック
    * 「詳細」ペインのアベイラビリティーゾーンを確認し、2つのDBインスタンスが配置されているアベイラビリティーゾーン以外を選択してください。
    * 例えば、2つのDBインスタンスが、それぞれ、 **ap-northeast-1a** と **ap-northeast-1d** に配置されたいた場合、 **ap-northeast-1c** を選択してください

```
アベイラビリティーゾーン : 既存DBインスタンが配置されていないアベイラビリティーゾーン
パブリックアクセス可能　 : はい
```
* **可用性向上のためには適切なアベイラビリティーゾーンを選択する必要があります。**


**インスタンスの仕様**
```
DB インスタンスのクラス : db.r4.large - 2 vcpu, 15.25 GiB RAM
```

**設定**
```
Aurora レプリカのソース : DBインスタンのロールが **書き込み** となっているDBインスタンス
DB インスタンス識別子　 : aup96-handson-replica2
```

**フェイルオーバー**
```
優先度 : 範囲-1
```

**モニタリング**
```
拡張モニタリング : 拡張モニタリングを有効にする
モニタリングロール : デフォルト
詳細度 : 60秒
```

**パフォーマンスインサイト**
```
パフォーマンスインサイトの有効化 : オン (ラジオボタンを選択)
保持期間 : デフォルト(7日)
マスターキー : (default)aws/rds
```
* 「Auroraレプリカの作成」をクリック

* 追加作成したレプリカのステータスが、 **”利用可能”** に変わるのを待ちます。（6分程度）
* レプリカのステータスが **"利用可能"**　に切り替わったら、5-5. で取得したスナップショット **"handson-aurora-snapshot"** のステータスが **利用可能** となっていことを確認
  * 「サイドバー」→「スナップショット」

* 追加したDBインスタンス"aup96-handson-replica2"がレプリカリストに追加され、ロールが **"読み込み"** となっていることを確認
  * 「サイドバー」→「クラスター」→「クラスター」ペイン→「DBクラスター識別子」のリンクをクリック →「DBクラスターメンバー」ペイン
  * レプリカ作成完了を待たずに、次のステップへ進みます。


## 5-7. リストア
* 5-5. で取得したスナップショットを元に復元(リストア)します。
  * **リストア操作では、新規DBクラスターが作成されます。また、既存DBインスタンス識別子は利用できません（既に削除されたDBインスタンス識別子は指定可能です）**
  * **特に入力・選択指示のない入力・選択項目は、設定されている値のままとしてください**

* 「サイドバー」→「スナップショット」→ 5-5.で取得したスナップショットのチェックボックをクリック →「スナップショットのアクション」ドロップダウンメニュー →「スナップショットの復元」　→　「DB インスタンスの復元」画面へ

**インスタンスの仕様**
```
DB エンジン : Aurora (PostgreSQL)
DB インスタンスのクラス : db.r4.large - 2 vcpu, 15.25 GiB RAM
```

**設定**
```
DB インスタンス識別子 : ap96rXXXXX (XXXXXは任意の文字列、3~5文字程度)を入力
```

**ネットワーク & セキュリティ**
```
Virtual Private Cloud (VPC) : デフォルトVPC
サブネットグループ : default
パブリックアクセシビリティ : はい
アベイラビリティーゾーン : ap-northeast-1a
```

**データベースの設定**
* DBパラメータグループ、DBクラスターのパラメータグループをカスタマイズしている場合にはカスタマイズ済みのパラメータグループを選択する必要があります。
```
データベースのポート : 5432
DB パラメータグループ : default.autora-postgresql9.6
DB クラスターのパラメータグループ : default.autora-postgresql9.6
```
* 「DBインスタンスの復元」ボタンをクリック
  * 復元したDBインスタンスがのステータスが **利用可能** になるまで少々時間（15分程度）を要します。復元が開始されたことを確認後、次のステップに進んで下さい。

* スナップショットから復元されるDBインスタンスは1インスタンス（DBインスタンのロールは **書き込み** となるプライマリーインスタンス）のみを含むクラスターが復元されるため、必要に応じレプリカを追加する必要があります。

## 5-8. オプション - 手動フェイルオーバーの確認（時間に余裕のある場合）
* 手動フェイルオーバーさせ、レプリカ（DBインスタンのロールが **読み込み** )を、プライマリインスタンス（DBインスタンのロールが **書き込み** ）へ昇格させます。
  * 「サイドバー」→「クラスター」→「DBクラスター識別子」列下のDBクラスター名( **ap96c** で始まるDBクラスター)のハイパーリンク → 「DB クラスターメンバー」ペインの「ロール」列
    * 各DBインスタンスのロールが、**読み込み** （レプリカ）なのか **書き込み** (プライマリーインスタンス)であるかを確認します。
    * DBインスタンス名のハイパーリンクをクリック → 「詳細」ペインの「フェイルオーバー優先順位」を確認
      * 「Replication」ペイン → DBインスタンス名のハイパーリンクをクリック → 他のDBインスタンスの「詳細」ペインの「フェイルオーバー優先順位」を確認

      * 「フェイルオーバー優先順位」が　**2** のDBインスタンスが **2つ** あります。(DBインスタンスのロールが、**読み込み** と **読み込み** それぞれ 1インスタンス)
      * 「フェイルオーバー優先順位」が　**1** のDBインスタンスが **1つ** あります。( **5-6. レプリカの追加** で追加したレプリカ）

    * 「詳細」ペインの「フェイルオーバー優先順位」を参照している画面上部の「インスタンス操作」メニュー → 「フェイルオーバー」→ 「Failover DB Cluster」画面 → 「フェイルオーバー」ボタン
      * 「サイドバー」→「クラスター」→「DBクラスター識別子」列下のDBクラスター名のハイパーリンク → 「DB クラスターメンバー」ペイン → 「更新」ボタン
        * **5-6. レプリカの追加**　で追加したレプリカのロールが、**書き込み** となったことを確認してください。
        * 「最近のイベント」ペインに以下のようなイベントが記録されていることも確認してください。
```
Completed failover to DB instance: ap96XXXX-replica2
Started cross AZ failover to DB instance: ap96XXXX-replica2
```  


## 5-9. クラスターの停止
* クラスターの停止・開始機能のリリースにより、開発環境など利用しない時間帯にクラスターの停止開始の代替操作としてのクラスターの削除、リストア、レプリカ追加という煩雑な操作は不要になりました。
  * **"5-1. Aurora PostgreSQLクラスターの作成"** で作成した **"ap96c"** で始まるDBクラスター識別子を持つクラスター(3 DBインスタンス)を **停止** します。

  * 「サイドバー」→「クラスター」→ "ap96c" で始まるDBクラスター識別子横にあるラジオボタンをクリック →「クラスターアクション」→「停止」→「データベースの停止」ボタンをクリック

* Amazon RDSも含めインスタンスの停止は可能ですが、停止可能な期間は7日間です。8日後には自動的に再起動します。
* クラスターの状況が **"停止中"** から **"停止"** になるまでしばらく待ちます。（12分程度）


## 5-10. クラスターの開始
* 「サイドバー」→「クラスター」→ "ap96c" で始まるDBクラスター識別子横にあるラジオボタンをクリック →「クラスターアクション」→「開始」を選択
  * クラスターの状況が **"開始中"** から **"利用可能"** になるまでしばらく時間を要します。（15分程度）

* クラスターの開始を待つ間に、リストアで作成したクラスターを削除します。次のステップへ進んでください。


## 5-11. クラスター内の全DBインスタンスの削除（兼　環境クリーンアップ）
* DBインスタンスを削除します。
  * 最初に **"ap96r"** で始まるクラスター（リストアで作成したクラスター）のDBインスタンスを削除します。

* 「サイドバー」→「インスタンス」→　"ap96r" で始まるDBインスタンス横にあるラジオボタンをクリック → 「インスタンスの操作」メニュー → 「削除」→「最終スナップショットを作成しますか？」ダイアログへ

**最終スナップショットを作成しますか？**
```
最終スナップショットを作成しますか? : いいえ
インスタンスの削除後、システムスナップショットとポイントインタイムの復元を含む自動バックアップが利用不可となることを了承しました。 : チェックボックスをチェック
削除を確認するには、delete me というフレーズを以下のフィールドに入力します : delete me
```
* 「削除」ボタンをクリック
  * "ap96r" で始まるDBインスタンスのステータスが **"削除中"** に変わります。

* **ここで、DBインスタンス削除操作を止めてください。 5-10. クラスターの開始で開始中のクラスターが起動するのを待ちます**
  * 5-10. クラスターの開始起動中のDBインスタンス全てのステータスが **"利用可能"** となった後、以下の操作を行います。
  * 順序は決まっていませんが、本ハンズオンでは、インスタンス画面のDBインスタンス列にリストされているDBインスタンスを上位から順に削除します。

* 「サイドバー」→「インスタンス」→　DBインスタンス横のラジオボタンをクリック → 「インスタンスの操作」メニュー →「削除」→ 「XXXXX インスタンス を削除しますか?」ダイアログへ
  * DBクラスターに含まれる 3 DBインスタンスのうち最初の 2 DBインスタンスの削除ダイアログでは、**delete me** という削除キーワードのみ入力します。
```
DB インスタンス XXXXX を削除してよろしいですか? : delete me
```
* 「削除」ボタンをクリック

* DBクラスターに含まれる **最後のDBインスタンスを削除する場合** 、以下の手順で削除します。
  * 「サイドバー」→「インスタンス」→ DBインスタンス横のラジオボタンをクリック→「インスタンスの操作」メニュー→「削除」→ 「XXXXX インスタンス を削除しますか?」ダイアログへ
```
最終スナップショットを作成しますか？ : いいえ
インスタンスの削除後、システムスナップショットとポイントインタイムの復元を含む自動バックアップが利用不可となることを了承しました。: チェックボックスをチェック
削除を確認するには、delete me というフレーズを以下のフィールドに入力します : delete me
```
* 「削除」ボタンをクリック

* 3つのDBインスタンスのステータスが **"削除中"** になっていれば正常です。削除されるまで6分程度要します。
## お疲れ様でした。これでハンズオンは終了です。

# Amazon Redshift ハンズオン - Day2

# 1. 環境構築準備
## 1-1. ハンズオンで利用するアカウント
* AWSプロフェッショナルサービスにて用意した以下のアカウントをご利用下さい。

|no.|アカウントID|管理者|
|---:|:---|:---|
|1||AWS仲谷|
|2||AWS仲谷|
|3||AWS仲谷|
|4||AWS関口|
|5||AWS関口|
|6||AWS関口|

* IAMユーザー名、パスワード、アカウントIDは当日お知らせします。
* ハンズオンでは**東京リージョン**をご利用ください。

## 1-2. CloudFormationによるハンズオン環境の構築
* 今回は事前にVPCとRedshift、Aurora PostgreSQL環境を構築済みです。
* 今回使用するSQL等は、本マークダウン中から直接コピーして下さい。

## 1−3. スナップショットの復元
* 事前にクラスター作成、テーブル作成、データロードを完了済みのクラスターから取得したスナップショットを使って、各自のAWSアカウントにRedshiftクラスターを復元します。
  * スナップショットを選択
  * 以下のスナップショットを選択
```
スナップショット名　：　for-hanson-1121
```
  * クラスター→該当クラスターをクリック→ステータスを開き、状況を確認

* リストアの開始には数分かかりますが、開始した後は読み書き可能な状態となります（ストリーミングリストア）。今回は開始を待たず次に進みます。

## 1-4. Redshiftへのネットワーク接続許可設定
* デフォルトの状態では、Redshiftのポート（5439/tcp）への接続は塞がれていますので、以下の操作で許可します。
 * 「クラスター」→当該クラスターをクリック→「VPCセキュリティグループ」→「default」を選択
 * EC2の画面に遷移する
 * 当該セキュリティグループのチェックボックスをチェックし、下部ペインの「インバウンド」タブ→「編集」をクリック
 * 「ルールの追加」をクリックし、「タイプ」のドロップダウンリストからRedshift（5439/tcp）を選択
 * 「ソース」のドロップダウンリストから「マイIP」を選択、保存

# 2. 接続確認

## 2-1. データベースへの接続確認
### 2-2-1 接続設定の作成
* PC上にインストールされている、Aginity Workbench for Redshift（以下Aginity）を起動します。
* 接続情報を以下の通り入力します。
* Redshiftクラスターエンドポイントはコンソールから確認し、コピペして下さい。

```
Name      : （任意）
Server    : （クラスターエンドポイントのURL。ポートを除く）
DB Port   : 5439
SSL Mode  : Prefer
Database  : rsdemo
User ID   : awsuser
Password  : （上記で入力したパスワード）
```

### 2-2-2 接続テスト
* 入力した接続情報に基づいて、Redshiftクラスターに接続します。

# 3. ハンズオン
* ここでは、Redshiftチューニングやトラブルシューティングについてハンズオンを行っていきます。
  * **sh1**という1,500万件ほどの小型のスキーマをサンプルに使っていきます。

## 3-1. WLM実習
* 「ワークロード管理」→「WLM」から、当該クラスター用のWLM設定を作成します。

### 3-1.1. WLM関連ビューの作成（オプション）
* WLM_QUEUE_STATE_VW（キュー状態を確認するためのビュー）を作成します。
```
drop view WLM_QUEUE_STATE_VW
;

create view WLM_QUEUE_STATE_VW as
select (config.service_class-5) as queue
, trim (class.condition) as description
, config.num_query_tasks as slots
, config.query_working_mem as mem
, config.max_execution_time as max_time
, config.user_group_wild_card as "user_*"
, config.query_group_wild_card as "query_*"
, state.num_queued_queries queued
, state.num_executing_queries executing
, state.num_executed_queries executed
from
STV_WLM_CLASSIFICATION_CONFIG class,
STV_WLM_SERVICE_CLASS_CONFIG config,
STV_WLM_SERVICE_CLASS_STATE state
where
class.action_service_class = config.service_class
and class.action_service_class = state.service_class
and config.service_class > 4
order by config.service_class;
```
* WLM_QUERY_STATE_VW（クエリー状態を確認するためのビュー）を作成します。

```
drop view WLM_QUERY_STATE_VW
;

create view WLM_QUERY_STATE_VW as
select query, (service_class-5) as queue, slot_count, trim(wlm_start_time) as start_time, trim(state) as state, trim(queue_time) as queue_time, trim(exec_time) as exec_time
from stv_wlm_query_state;
```
* 以下を実行してそれぞれのビューの内容を確認します。
```
select * from wlm_queue_state_vw;
select * from wlm_query_state_vw;
```

### 3-1.2. WLMキューの作成とクラスターへの紐付け
* WLMキューを作成します。
　　* パラメーターグループ→クラスターパラメーターグループの作成
```
パラメーターグループファミリー ... デフォルト
パラメーターグループ名 ... 任意
説明 ... 任意
```
　　* ワークロード管理→パラメーターグループ→先程作成したパラメーターグループを選択→キューの追加
```
クエリーキュー ... メモリ割り当て50%、同時実行数（スロット）10、ユーザーグループ=queryusers、クエリーグループ=adhocqueries
バッチキュー ... メモリ割り当て40%、同時実行数（スロット）2、ユーザーグループ=batchusers、クエリーグループ=batches
デフォルトキュー ... メモリ割り当て残り全て、同時実行数（スロット）1
```
  * クラスター→当該クラスターのチェックボックスをチェック→クラスターボタン→変更→先程作成したパラメーターグループ
  * ***再起動がかかりますので、クラスター画面がどう変わるかを確認しつつ、暫くお待ち下さい。***

### 3-1-3. WLMの動作を確認
* WLMキューおよびスロットの動作を確認します。
  * キューの状態確認
```
select * from wlm_queue_state_vw;
```
  * 結果セットキャッシュを無効化し、実行時間をクエリーごとに正確に測れるようにする
```
set enable_result_cache_for_session=off;
show enable_result_cache_for_session;
```
  * デフォルト状態でクエリーを実行
```
SELECT
 prod_id,
 count(1) as deal_conut,
 avg(quantity_sold) as average_sold_num,
 sum(quantity_sold*amount_sold) as total_sales
FROM
 sh1.sales
GROUP BY
 prod_id
ORDER BY
 total_sales DESC
limit 10
;
select * from wlm_queue_state_vw;
select * from wlm_query_state_vw;
```
  * クエリーグループを指定してクエリーを実行
```
set query_group to adhocqueries;

SELECT
 prod_id,
 count(1) as deal_conut,
 avg(quantity_sold) as average_sold_num,
 sum(quantity_sold*amount_sold) as total_sales
FROM
 sh1.sales
GROUP BY
 prod_id
ORDER BY
 total_sales DESC
limit 10
;
select * from wlm_queue_state_vw;
select * from wlm_query_state_vw;
```
  * 指定したキューで実行されていることを確認します。

  * スロットを集約してクエリーを実行
```
set wlm_query_slot_count to 10;

SELECT
 prod_id,
 count(1) as deal_conut,
 avg(quantity_sold) as average_sold_num,
 sum(quantity_sold*amount_sold) as total_sales
FROM
 sh1.sales
GROUP BY
 prod_id
ORDER BY
 total_sales DESC
limit 10
;
select * from wlm_queue_state_vw;
select * from wlm_query_state_vw;
```
  * Executingの値が変わっていることを確認します。これは、最初のクエリーが10スロット全てを占有して実行していることを示しています。

  * キューの動きをさらに深掘りします。
    * SQLセッションを三つ開く（以下、**SQLタブ1、SQLタブ2、SQLタブ3**）を開く
    * **SQLタブ1**でロングランニングクエリーを実行
    * これが実行されている間に、**SQLタブ2**で先程と同じスロット集約クエリーを実行
    * **SQLタブ3**でビューを確認

**SQLタブ1（ロングランニングクエリー）**
```
/* 結果セットキャッシュを無効化 */
set enable_result_cache_for_session=off;
show enable_result_cache_for_session;

/* クエリーグループを指定 */
set query_group to adhocqueries;

/* ロングランニングクエリーを実行 */
select * from sh1.sales;
```

**SQLタブ2（本クエリー）**
```
/* 結果セットキャッシュを無効化 */
set enable_result_cache_for_session=off;
show enable_result_cache_for_session;

/* 前ステップと同じクエリーを実行 */
set query_group to adhocqueries;
set wlm_query_slot_count to 10;

SELECT
 prod_id,
 count(1) as deal_conut,
 avg(quantity_sold) as average_sold_num,
 sum(quantity_sold*amount_sold) as total_sales
FROM
 sh1.sales
GROUP BY
 prod_id
ORDER BY
 total_sales DESC
limit 10
;
```
**SQLタブ3（ビューの確認）***
```
/* ビューの状態を確認 */
select * from wlm_queue_state_vw;
select * from wlm_query_state_vw;
```
  * 何が起きましたか？
　　　　  * adhocqueriesのスロット数は10です。
    * このうち、SQLタブ1で通常の1スロットを使ってロングランニングクエリーが実行されているため、残存スロットは9です。
    * この状態でSQLタブ2で 'set wlm_query_slot_count to 10' を使って10個のスロットを確保しようとしました。スロットが確保できるようになるまで、このSQLはキュー待ちの状態となります。
    * スロット確保を多用するとこうした状態が生まれやすくなりますので注意して下さい。

  * セッション設定をリセット
```
reset wlm_query_slot_count;
reset query_group;
reset enable_result_cache_for_session;
```

### 3-1-4. QMRルールの作成と設定
* QMRルールを作成します。
  * 「ワークロード管理」→「パラメーターグループ」→先程作成したパラメーターグループを選択→「編集」をクリック
  * クエリーキュー（adhocqueries）から「テンプレートからルールを追加」をクリック
  * 「クエリーが多くの行を返す」をチェックし、「追加」をクリック
  * ルールに名前を付け、アクションを「ホップ」に変更
  * 「保存」をクリック
* これで、クエリーキューで長大なクエリーが流れた際、自動的にバッチキューに流れるようになります。

## 3-2. その他のチューニング

### 3-2-1. テーブル情報の取得（Admin Scripts）
* Admin Scripts（table-info.sql）を使用して、クラスター内のテーブル情報を取得します。
```
SELECT TRIM(pgn.nspname) AS SCHEMA,
       TRIM(a.name) AS TABLE,
       id AS TableId,
       decode(pgc.reldiststyle,
             0, 'EVEN',
             1,det.distkey ,
             8,'ALL'
       ) AS DistKey,
       decode(pgc.reldiststyle,
             8,NULL,
             dist_ratio.ratio::DECIMAL(20,4)
       ) AS Skew,
       det.head_sort AS "SortKey",
       det.n_sortkeys AS "#SKs",
       CASE WHEN pgc.reldiststyle = 8 THEN a.rows_all_dist ELSE a.rows END AS rows,
       b.mbytes,
       decode(det.max_enc,
             0,'N',
             'Y'
       ) AS Enc,
       det.pct_enc,
       decode(b.mbytes,
             0,0,
             ((b.mbytes/part.total::DECIMAL)*100)::DECIMAL(20,2)
       ) AS pct_of_total,
       (CASE WHEN a.rows = 0 THEN NULL ELSE
          CASE WHEN pgc.reldiststyle = 8 THEN ((a.rows_all_dist - pgc.reltuples)::DECIMAL(20,3) / a.rows_all_dist::DECIMAL(20,3)*100)::DECIMAL(20,2)
                ELSE ((a.rows - pgc.reltuples)::DECIMAL(20,3) / a.rows::DECIMAL(20,3)*100)::DECIMAL(20,2) END END
       ) AS pct_stats_off,
       CASE WHEN pgc.reldiststyle = 8
          THEN decode( det.n_sortkeys,0, NULL,DECODE( a.rows_all_dist,0,0, (a.unsorted_rows_all_dist::DECIMAL(32)/a.rows_all_dist)*100))::DECIMAL(20,2)
          ELSE decode( det.n_sortkeys,0, NULL,DECODE( a.rows,0,0, (a.unsorted_rows::DECIMAL(32)/a.rows)*100))::DECIMAL(20,2) END
        AS pct_unsorted
FROM (SELECT db_id,
             id,
             name,
             SUM(ROWS) AS ROWS,
             MAX(ROWS) AS rows_all_dist,
             SUM(ROWS) - SUM(sorted_rows) AS unsorted_rows,
             MAX(ROWS) - MAX(sorted_rows) AS unsorted_rows_all_dist
      FROM stv_tbl_perm a
      GROUP BY db_id,
               id,
               name) AS a
  JOIN pg_class AS pgc ON pgc.oid = a.id
  JOIN pg_namespace AS pgn ON pgn.oid = pgc.relnamespace
  LEFT OUTER JOIN (SELECT tbl, COUNT(*) AS mbytes FROM stv_blocklist GROUP BY tbl) b ON a.id = b.tbl
  INNER JOIN (SELECT attrelid,
                     MIN(CASE attisdistkey WHEN 't' THEN attname ELSE NULL END) AS "distkey",
                     MIN(CASE attsortkeyord WHEN 1 THEN attname ELSE NULL END) AS head_sort,
                     MAX(attsortkeyord) AS n_sortkeys,
                     MAX(attencodingtype) AS max_enc,
                     SUM(case when attencodingtype <> 0 then 1 else 0 end)::DECIMAL(20,3)/COUNT(attencodingtype)::DECIMAL(20,3)  *100.00 as pct_enc
              FROM pg_attribute
              GROUP BY 1) AS det ON det.attrelid = a.id
  INNER JOIN (SELECT tbl,
                     MAX(Mbytes)::DECIMAL(32) /MIN(Mbytes) AS ratio
              FROM (SELECT tbl,
                           TRIM(name) AS name,
                           slice,
                           COUNT(*) AS Mbytes
                    FROM svv_diskusage
                    GROUP BY tbl,
                             name,
                             slice)
              GROUP BY tbl,
                       name) AS dist_ratio ON a.id = dist_ratio.tbl
  JOIN (SELECT SUM(capacity) AS total
        FROM stv_partitions
        WHERE part_begin = 0) AS part ON 1 = 1
WHERE mbytes IS NOT NULL
AND   pgc.relowner > 1
-- and pgn.nspname = 'schema' -- schemaname
-- and a.name like 'table%' -- tablename
-- and det.max_enc = 0 -- non-compressed tables
ORDER BY mbytes DESC;

```

### 3-2-2. オープンセッションの確認
* Admin Viewsを使用して、最近使用されたセッション（現在オープン中のものを含む）を確認します。
  * ビューを作成
```
CREATE OR REPLACE VIEW admin.v_open_session
AS
SELECT
	CASE WHEN disc.recordtime IS NULL THEN 'Y' ELSE 'N' END AS connected
	,init.recordtime AS conn_recordtime
	,disc.recordtime AS disconn_recordtime
	,init.pid AS pid
	,init.remotehost
	,init.remoteport
	,init.username AS username
	,disc.duration AS conn_duration
FROM
	(SELECT event, recordtime, remotehost, remoteport, pid, username FROM stl_connection_log WHERE event = 'initiating session') AS init
LEFT OUTER JOIN
	(SELECT event, recordtime, remotehost, remoteport, pid, username, duration FROM stl_connection_log WHERE event = 'disconnecting session') AS disc
		ON init.pid = disc.pid
		AND init.remotehost = disc.remotehost
		AND init.remoteport = disc.remoteport
;  
```
  * ビューの内容を確認（UTC表記となる点に留意）
```
select * from v_open_session
order by conn_recordtime desc
;
```

### 3-2-3. 圧縮エンコーディング
* 圧縮エンコーディングが適切に設定されているかどうかを確認します。
  * スキーマを**sh1**に変更。以下のSQLで切り替わりを確認
```
select current_schema();
```
  * 現在の圧縮エンコーディングを表示
```
select "column", type, encoding, distkey, sortkey, "notnull"
from pg_table_def
where tablename = 'sales'
```
  * 推奨される圧縮エンコーディングを表示
```
analyze compression sh1.sales;
```

### 3-2-4. Actualの確認
* クエリー分析に頻繁に利用される「Actual」画面を確認します。
* Redshiftにはexplain analyzeがありませんので、マネジメントコンソールを使用してビジュアルなActual画面を確認するアプローチとなります。
  * 「クラスター」→当該クラスターをクリック→「クエリー」タブ
  * 任意のクエリーを選んでクエリーIDをクリック
  * 「クエリーの実行の詳細」→「Plan」
  * 「クエリーの実行の詳細」→「Actual」
  * 「Cluster Performance During Query Execution」も併せて確認

# 4. ハンズオン環境の削除
* Redshiftハンズオン環境の削除は弊社にて行います。そのままにしておいていただいて結構です。

***

# Amazon Aurora PostgreSQL ハンズオン - Day2

# 4. 環境構築準備
  * **本ハンズオンでは環境は事前に作成済です。**
  * **マスターユーザー名及びパスワード等は別途おしらせします。**
  * **コンソール右上で「東京」リージョンが選択されていることだけ、再度ご確認下さい。**

  * **アクセス元の制限（既に他のアドレスに制限されている状態になっています）**

  * 「サービス」→「RDS」→「インスタンス」→「インスタンス」ペイン →「DBインスタンス」にリストされている3つのDBインスタンスのいずれか一つのリンクをクリック
    * スクロールダウンして「詳細」ペイン →「セキュリティグループ」のActive状態のセキュリティグループのリンクをクリック
    * 「EC2」ダッシュボード → 「セキュリティグループ」ペイン　→ 「インバウンド」タブ → 「編集」ボタン をクリック
    * 「インバウンドのルールの編集」ダイアログ → 「タイプ」が"PostgreSQL"となっているルールの「ソース」列 → 「マイIP」→ 「保存」ボタンをクリック

  * この操作で、Aurora PostgreSQLの各インスタンスは、現在操作中のPCのIPアドレスからのアクセスのみ受け付けるようになります


## 4-1 クライアントの接続設定
  * 事前にご利用するPCへ DbVisualizer インストール願います。
  * psqlでも可能ですが利用される場合は、事前にお知らせするマスターパスタワード及び、読み込みエンドポイントとクラスターエンドポイント、各レプリカのDBインスタンスエンドポイントをメモ帳へコピーしておいてください。

  * **Database Serverに設定するクラスターエンドポイント及び、読み込みエンドポイントの確認方法は以下の通りです。**
    * 「サービス」→「RDS」→「クラスター」→「DBクラスター識別子」以下の該当クラスターのリンク→「詳細」ペインから確認できます。

  * **各レプリカのインスタンスエンドポイントの確認方法は以下の通りです。**
    * 「サービス」→「RDS」→「クラスター」→「クラスターメンバー」ペイン → 「ロール」列が **読み込み** となっている「dbインスタンス」列のリンクをクリック → 「接続」ペインから確認できます。


### 4-1-1 **クラスターエンドポイント向け接続設定**
  * DbVisualizerの接続設定を行います。
  * 「Databases」タブ → 「Connections」ツリーを選択し右クリック → ポップアップメニュー「Create Database Connection」 → 「User Connection Wizard?」ダイアログ  → 「Use Wizard」ボタン → 「New Connection Wizard」ダイアログ

  ```
  Connection Alias : Cluster Endpoint
  ```
  * 「Next」ボタン

  ```
  Select Database Driver : PostgreSQL
  ```
  * 「Next」ボタン

  ```
  Notes                  : -
  Setting format         : Server Info
  Database Server        : 事前作成されているap96-clusterクラスターのクラスターエンドポイント
  Database Port          : 5432
  Database               : mydb
  Database UserId        : awsuser
  Database Password      : 本日お知らせしたマスターパスワード
  Auto Commit            : チェックボックスをチェック
  Save Database Password : Save between Sessions
  Permisssion mode       : Development
  ```
  * **「Ping Server」ボタンをクリックし接続確認**
  * 「Finish」ボタン

### 4-1-2 **読み込みエンドポイント向け接続設定**
  * 4-1-1 で作成した **クラスターエンドポイント向け接続設定** を選択し右クリック → ポップアップメニュ「Duplicate Database Connection」
    * 以下項目のみ設定しその他内容は、4-1-1 の設定と同じです。

  ```
  Name    　　　　　　　　　: Reader Endpoint
  Database Server        : 事前作成されているap96-clusterクラスターの読み込みエンドポイント
  ```
  * **「Ping Server」ボタンをクリックし接続確認**
  * 上記を設定したタブを閉じます。

### 4-1-3 **レプリカ1のインスタンスエンドポイント向け接続設定**
  * **便宜上、レプリカ1、レプリカ2と記載していますが、2つあるレプリカのうちどちらを1,2にしても構いません。**

  * 4-1-1 で作成した **クラスターエンドポイント向け接続設定** を選択し右クリック → ポップアップメニュ「Duplicate Database Connection」
    * 以下項目のみ設定しその他内容は、4-1-1 の設定と同じです。

  ```
  Name    　　　　　　　　　: Replica1 Endpoint
  Database Server        : 事前作成されているap96-clusterのレプリカ1のDBインスタンスエンドポイント
  ```
  * **「Ping Server」ボタンをクリックし接続確認**
  * 上記を設定したタブを閉じます。

### 4-1-4 **レプリカ2のインスタンスエンドポイント向け接続設定**

  * 4-1-1 で作成した **クラスターエンドポイント向け接続設定** を選択し右クリック → ポップアップメニュ「Duplicate Database Connection」
    * 以下項目のみ設定しその他内容は、4-1-1 の設定と同じです。

  ```
  Name    　　　　　　　　　: Replica2 Endpoint
  Database Server        : 事前作成されているap96-clusterのレプリカ2のDBインスタンスエンドポイント
  ```
  * **「Ping Server」ボタンをクリックし接続確認**
  * 上記を設定したタブを閉じます。

# 5. ハンズオン
* Aurora PostgreSQLクラスターを利用した運用監視、チューニングなどのトラブルシューティングを体験していただきます。

## 5-1. Performance Insights
  * **(日本語訳はパフォーマンスインサイトになっていますが、最後の s は大切です)**
  * Performance Insightsを利用した分析及び、チューニングの流れを体験していただきます。


### 5-1-1. 遅延分析及びチューニング　ー　索引作成漏れ
  * 「サービス」→「RDS」→「パフォーマンスインサイト」→ DBインスタンス「ap96」ラジオボタンを選択　→　「データベースのロード」ペイン　を表示します。
  * DbVisualizer → 「Databases」タブ → 「Connections」ツリー → 「Cluster Endpoint」を選択、右クリック → ポップアップメニュー「Connect」
  * DbVisualizer → 「メニュー」→「SQL Commander」で、SQL Commanderペインを開きます。
  * **注意** 「メニュー」→「SQL Commander」→ 「Database」は **mydb** 、 「Schema」は、 **hoge** 　になっていることを確認、なっていない場合には、各ドロップダウンメニューより選択します。

  * **以下のSQL文の性能要件は、TAT 1秒以内です**
    * 以下の SQL文を SQL Commanderへコピーペーストし、「メニュー」→「SQL Commander」→「Execute」を選択しSQL文を実行します。

    ```
    SELECT COUNT(ref_no) FROM t1 WHERE ref_no BETWEEN 1 AND 250;
    ```
  * SQL文実行後、Performance Insightsのデータベースのロードペインを確認
    * Top Load Item Table上部のスライス →　「待機」、下部のスライス → 「SQL」を選択し、時間軸から 5 分を選択 →
    * 数秒程度のSQL文が見つかるはずです。グラフが小さい場合は、拡大したい範囲をマウスで選択することで拡大できます。
    * SQL文横の▶︎アイコンをクリックしてドリルダウンするとSQL文が現れるはずです。

      * このSQL文の待機イベントの傾向からどのようなことが言えますか？
      * バッファキャッシュに乗っている状態であれば、2秒程度で、待機イベントはCPUバウンドな傾向を示しているはずです。
        * バッファキャッシュでヒットしなかった状況は考慮しなくて構いません。バッファキャッシュでヒットする前提です。

      * チューニング方法を検討するため、実行計画(Actual)を確認する必要がありますが、どのような方法で確認できますか？
      * 実行計画(Actual)と、定義(DbVisualizerで列定義や索引）を確認できます。  
      * 実行計画(Actual)は以下文で取得できます。  

      ```
      explain analyze SELECT COUNT(ref_no) FROM t1 WHERE ref_no BETWEEN 1 AND 250;
      ```
        * 実行計画は以下の通りです。
          ```
          Aggregate  (cost=289382.49..289382.50 rows=1 width=8) (actual time=2160.475..2160.475 rows=1 loops=1)
          ->  Seq Scan on t1  (cost=0.00..283334.00 rows=2419397 width=5) (actual time=0.093..1994.440 rows=2500000 loops=1)
            Filter: ((ref_no >= '1'::numeric) AND (ref_no <= '250'::numeric))
            Rows Removed by Filter: 7500000
          Planning time: 0.074 ms
          Execution time: 2160.498 ms
          ```

      * 以下の SQL文を SQL Commanderへコピーペーストし、「メニュー」→「SQL Commander」→「Execute」を選択しSQL文を実行して、セグメントサイズを確認してみてください。

      ```
      SELECT
       objectname
       ,TO_CHAR(
         pg_relation_size(
           objectname::regclass)/1024/1024/1024
           , '999,999,999,999.99'
       ) AS "GB"
      FROM
      (
       SELECT
         tablename AS objectname
       FROM
         pg_tables
        WHERE
         schemaname = 'hoge'
       UNION
       SELECT
         indexname AS objectname
       FROM
          pg_indexes
       WHERE
         schemaname = 'hoge'
      ) AS objectlist
      ORDER BY
        2 DESC;
      ```
        * 結果は以下の通りです。
          ```
          objectname |         GB          
          -----------+---------------------
          t1         |                1.00
          t2         |                1.00
          ```

      * これらのことから、t1表のref_no列に索引が存在しないため、1GBのデータサイズ、かつ、 10,000,000行のデータを全表走査し、7,500,000行のデータを読み込んだ上で捨てていることがわかります。
      * この例では物理読み込みはキャッシュにヒットしているため影響していませんが、1GBは読み込み過ぎであるため、適切な索引があれば、フィルタリングによるCPU処理も全表走査の読み込み量も同時に減らせることになります。
      * ref_no以外に、seq_noも検索されそうなので、ref_no,seq_no列に索引を作成することで大きく改善します。(索引の作成に　**2,30秒**　程度要します)
      ```
      create index ix_t1 on t1(ref_no, seq_no);
      explain analyze SELECT COUNT(ref_no) FROM t1 WHERE ref_no BETWEEN 1 AND 250;
      ```
        * 結果は以下の通りです。 Seq Scan から Index Only Scan に変わり、Heapへのアクセスが回避されたことで大きく改善しています。
          ```
          Aggregate  (cost=91736.67..91736.68 rows=1 width=8) (actual time=624.912..624.912 rows=1 loops=1)
          ->  Index Only Scan using ix_t1 on t1  (cost=0.43..85685.98 rows=2420277 width=5) (actual time=0.034..457.772 rows=2500000 loops=1)
             Index Cond: ((ref_no >= '1'::numeric) AND (ref_no <= '250'::numeric))
             Heap Fetches: 0
          Planning time: 0.110 ms
          Execution time: 624.940 ms
          ```

### 5-1-2. 遅延分析及びチューニング　ー　ロックの競合
  * 「サービス」→「RDS」→「パフォーマンスインサイト」→ DBインスタンス「ap96」ラジオボタンを選択　→　「データベースのロード」ペイン　を表示
  * DbVisualizerを 2つ起動して、それぞれ Cluster Endpointに接続し、メニューから「SQL Commander」を開いた状態にします。
  * DbVisualizer → 「Databases」タブ → 「Connections」ツリー → 「Cluster Endpoint」を選択、右クリック → ポップアップメニュー「Connect」を洗濯してDBへ接続します。
  * DbVisualizer → 「メニュー」→「SQL Commander」で、SQL Commanderペインを開きます。
  * DbVisualizer → 「メニュー」→「SQL Commander」→ 「Transaction」→　サブメニュー「**Turn off auto commit**」を選択します。（**2つのDbVisualizerともAuto Commitはoffにします。。なお、SQL Commanderの中段に、Auto Commit : ON/OFF が表示されいるので合わせて確認してください**）

  * 1つめのDbVisualizerで以下のSQL文を実行

  ```
  update t1 set description = 'hoge' where ref_no = 1;
  ```
  * **Auto Commit OFFの場合、"Commit"/"Rollback"/"Continue with Uncommited"を問うダイアログが表示されます。 "Continue with Uncommited"ボタンをクリックします。**

  * 2つめのDbVisualizerで以下のSQL文を実行します。（こちらのSQL文はトランザクションのcommit/rollbackを待機することになります）

  ```  
  update t1 set description = 'hoge' where ref_no = 1;
  ```

  * Performance InsightsのTop Load Item Tableの上部スライスは「待機」を選択、下部のスライスは「SQL」を選択することで、2つめのupdate文がロックの解放待ちになっていることが確認できるはずです。
  * Performance Insightsを活用するには待機イベントの考え方を理解することが重要です。

  * 以下のSQL文を、1つめのDbVisualizerのSQL Commanderから実行します。

    ```
    SELECT
      clock_timestamp() AS "CURRENT_TIMESTAMP"
      , pg_class.relname
      , pg_locks.locktype
      , pg_locks.database
      , pg_locks.relation
      , pg_locks.page
      , pg_locks.tuple
      , pg_locks.virtualtransaction
      , pg_locks.pid
      , pg_locks.mode
      , pg_locks.granted
    FROM
      pg_locks
      INNER JOIN pg_class
      ON
        pg_locks.relation = pg_class.oid
    WHERE
      relname !~ '^pg_'
      AND  relname <> 'active_locks'
    ;
    ```
      * 以下のサンプルのような結果が得られます。（ロックのタイプとロックしているオブジェクトを確認できます）
        ```
              CURRENT_TIMESTAMP       | relname | locktype | database | relation | page | tuple | virtualtransaction |  pid  |       mode       | granted
        ------------------------------+---------+----------+----------+----------+------+-------+--------------------+-------+------------------+---------
        2018-11-09 06:19:50.395239+00 | ix_t1   | relation |    16394 |    24588 |      |       | 7/334              | 27126 | RowExclusiveLock | t
        2018-11-09 06:19:50.395254+00 | t1      | relation |    16394 |    16401 |      |       | 7/334              | 27126 | RowExclusiveLock | t
        2018-11-09 06:19:50.395256+00 | ix_t1   | relation |    16394 |    24588 |      |       | 6/575              | 27722 | RowExclusiveLock | t
        2018-11-09 06:19:50.395258+00 | t1      | relation |    16394 |    16401 |      |       | 6/575              | 27722 | RowExclusiveLock | t
        2018-11-09 06:19:50.39526+00  | t1      | tuple    |    16394 |    16401 |    0 |    77 | 7/334              | 27126 | ExclusiveLock    | t
        (5 rows)
        ```

    * Performance Insights及び、ロックされているオブジェクトの状態を確認後、以下の手順でロールバックします。
      * 1つ目のDbVisualizerにて → 「メニュー」→「SQL Commander」→ 「Transaction」→　サブメニュー「Rollback」
      * 2つ目のDbSisualizerに、**"Commit"/"Rollback"/"Continue with Uncommited"を問うダイアログ** が表示されます。(表示されない場合は、1つ目のDbVisualizerと同じ操作を行いRollbackしてください)
        * 「**Continue with Uncommited**」ボタンをクリックしトランザクションをRollbackします。


### 5-1-3. 遅延分析及びチューニング　ー　掛け逃げクエリー対応
  * **時間に余裕のある方のみ**

  * **前の操作で mydb に接続している 2つの DbVisualizer をそのまま利用します**
  * 1つ目の DbVisualizer で以下の無名PL/pgSQLスクリプトをSQL Commander へコピーペースト → 「メニュー」→「SQL Commander」→「Execute Buffer」（▶︎アイコンと赤い●アイコンが組み合わされています）を選択して実行します。（このスクリプトは無限ループし1vCPUのコアを消費し続けます。

  ```
  DO $$BEGIN
    LOOP
      --
    END LOOP;
  END$$;
  ```

  * **プライマリインスタンスのDBインスタンス名を確認**
    * この例ではプライマリーインスタンスへ接続していますが、パフォーマンスインサイトが有効なインスタンスは、DBインスタンス名でリストされているためプライマリーインスタンスとなっているDBインスタンス名を確認しておく必要があります。
    * **確認方法は次の通りです**
    * 「サービス」→「RDS」→「クラスター」→「クラスターメンバー」ペイン → 「ロール」列が **書き込み** となっている「dbインスタンス」をメモ帳などにコピーしておきます。
    * マネージメントコンソール →「サービス」→「RDS」→「パフォーマンスインサイト」→ 前述の方法で確認したDBインスタンのリンク
     * Performance Insights → 「時間軸」→「5分」→ グラフ右横の「スライス基準」メニュー → 「待機」
     * Performance Insights →　グラフ下方のTop Load Itel Tableのスライス基準 →「SQL」
     * **Performance InsightsのグラフからCPU利用率100%のセッションが 1 つあることを確認**

    * 2つ目の DvVisualizer で以下のSQL文を何度か実行してください。(1 行ヒットするまで実行して見てください) 
    * 以下のクエリーでは実行時間が10秒以上経過したSQL文を取得しています。

    ```
    SELECT
      pid
      , now() - query_start AS "elapsed_time"
      , usename
      , datname
      , wait_event_type
      , wait_event
      , state
      , query
    FROM
     pg_stat_activity
    WHERE
      now() - query_start > '10 seconds'::interval
      AND  state = 'active'
    ORDER BY
     elapsed_time DESC;
    ```
    * 該当クエリーが長時間実行されている、いわゆるかけ逃げクエリーである場合には、セッションをキャンセルすることも可能です。
    * 前述のクエリーを実行したDbVisualizerのSQL Commanderから以下のSQL文を実行してください。
    ```
    SELECT pg_cancel_backend( 前述のクエリーで得られたpidの値 );
    ```
      * **以下の結果が返されます。**
        ```
        pg_cancel_backend
        -----------------
        true
        ```
      * **無限ループするPL/pgSQLを実行しているセッションでは以下のエラーを受け取り処理がキャンセルされます。**
        ```
        ERROR:  canceling statement due to user request
        CONTEXT:  PL/pgSQL function inline_code_block line 2 at LOOP
        ```

## 5-2. Auto Scaling （このセクションは時間に余裕がある場合のオプションです）
  * レプリカに一定の負荷をかけScale out / Scale inを体験していただきます。(CloudWatch Dashboardの作成及び、イベントサブスクリプションe-mailの設定含みます)

### 5-2-1. Auto Scalingの設定
  * 「サービス」→「サイドバー」→「クラスター」→「クラスター」ペイン → 「DBクラスター識別子」列の ap96-clusterのリンク → 「Auro Scalingポリシー」ペイン →　「追加」ボタン → 「Auto Scalingポリシーの追加」画面 → 「ポリシーの詳細」ペイン

  ```
  ポリシー名　　　　　 : my-auto-scaling-polycy
  ターゲットメトリクス : 「Auroraレプリカの平均CPU使用率」ラジオボタン
  ターゲット値　　　　 : 50
  ```
  * 「▶︎追加の設定」をクリックして拡張します。

  ```
  スケールイン                : 「有効」スライドボタン
  スケールインクールダウン期間　 : 300
  スケールアウトクールダウン期間 : 300
  ```

  * クラスターキャパシティの詳細

  ```
  最小キャパシティ : 2
  最大キャパシティ : 3
  ```

  * 「ポリシーの追加」ボタンをクリックします。

### 5-2-2. イベントサブスクリプションの設定
  * 「サイドバー」→ 「イベントサブスクリプション」→ 「イベントサブスクリプションの作成」ボタン →「イベントサブスクリプションの作成」画面

  * 「詳細」ペイン

  ```
  名前 : create-or-delete-instance
  ```

  * 「ターゲット」ペイン

  ```
  通知の送信先 : 「新しいEメールトピック」ラジオボタン
  トピック名　 : create-or-delete-instance
  受信者　　　 : Confirmation可能な実在するメールアドレス
  ```  

  * 「ソース」ペイン

  ```
  ソースタイプ　　　　　　 : ドロップダウンメニュー「インスタンス」
  インスタンスを含める　　 : 「全てのインスタンス」ラジオボタン
  含まれるイベントカテゴリ : 「特定のイベントカテゴリを選択します」ラジオボタン数
  特定のイベント　　　　　 : ポップアップメニューから「作成」と「削除」を選ぶ
  ```
  * 「作成」ボタンをクリックします。

### 5-2-3. CloudWatch Dashboardの設定
  * 「サービス」→「サイドバー」→「ダッシュボード」→「ダッシュボード」ペイン → 「ダッシュボードの作成」ボタン

  * 「新しいダッシュボードの作成」ダイアログ

  ```
  ダッシュボード名 : handson
  ```
  * 「ダッシュボードの作成」ボタンをクリックします。

  * 「ダッシュボードに追加」ダイアログ

  ```
  「数値」ボタンアイコン
  ```
  * 「設定」ボタンをクリックします。

  * 「メトリクスグラフの追加」ダイアログ → 「すべてのメトリクス」タブ → 「RDS」→「データベース別メトリクス」→ 「検索ボックス」

  ```
  CPUUtilization
  ```
  * 入力後 ↩︎[CR] → リストされた **ap96** の 3 DBインスタンスのチェックボックスをクリック → 「ウィジェットの作成」ボタンをクリック

  * 「ウィジェットの追加」ボタン → 「ダッシュボードに追加」ダイアログ

　```
　「数値」ボタンアイコン
　```
  * 「設定」ボタンをクリックします。


  * 「メトリクスグラフの追加」ダイアログ → 「すべてのメトリクス」タブ → 「RDS」→「データベース別メトリクス」→ 「検索ボックス」

　```
  FreeLocalStorage
　```
  * 入力後 ↩︎[CR] → リストされた **ap96** の 3 DBインスタンスのチェックボックスをクリック → 「ウィジェットの作成」ボタンをクリック → 「ダッシュボードの保存」ボタンをクリックします。


### 5-2-4. Auto Scalingによるスケールアウトとスケールイン
  * DbVisualizer を 3 つ起動（無料版では、複数のSQL Commanderペインを開けないため）
    * 2のDbVislalizerで **Replica1 Endpoint** へ接続し、残りの1つは、 **Replica2 Endpoint** へ接続し、それぞれで、SQL Commander ペインを開きます。

    * **操作例**
      * 「Databaes」タブ → 「Connections」ツリー → 「Replia1 Endpotion」を選択し右クリック → ポップアップメニュー「Connect」
      * 「メニュー」→「SQL Commander」→ 「Database」は **mydb** 、 「Schema」は、 **hoge** 　になっていることを確認、なっていない場合には、各ドロップダウンメニューより選択
        * それぞれの **SQL Commander** ペインで以下を入力、

        ```
        DO $$BEGIN
          LOOP
            --
          END LOOP;
        END$$;
        ```
        * 入力後　→ 「メニュー」→「SQL Commander」→「Execute Buffer」を選択して実行します。

### 5-2-5. CloudWatch Dashboardでのモニタリング
  * レプリカ1の CPUUtilizaton が 100% 、レプリカ2の CPUUtilizaton が 50% 程度になりしばらくすると、Auto Scaling により、レプリカが１インスタンス自動作成されます。
    * インスタンスが作成されると、イベントサブスクリプションで登録した e-mailへDBインスタンスが作成されたことが通知されます。


### 5-2-6. Auto Scalingの確認
  * 「サービス」→「RDS」→「クラスター」→「クラスター」ペイン → 「DBクラスター識別子」列の **ap96-cluster** のリンク → 「DBクラスターメンバー」ペイン
    * 新規インスタンスが作成されているとを確認してください。作成されたDBインスタンス名のリンクをクリック →「詳細」ペインの「フェイルオーバー優先度」が **15** （最も低い)で作成されていることなどを確認してみてください。

#### 5-2-6-1. DbVisualizer で実行中の負荷かけスクリプトの停止 （オプション）
  * 「DbVisualizer」→ 「メニュー」→「SQL Commander」→「Stop」を選択してスクリプトを停止させます。
  * 同様の手順で残る 2 つの負荷かけスクリプトを停止させます。





## 5-3. フェイルオーバー （このセクションは時間に余裕がある場合のオプションです）
  * フェイルオーバーの優先度の確認後、フェイルオーバーさせ、フェイルオーバー完了までの時間やフェイルオーバー先の制御を体験していただきます。
  * **フェイルオーバーの優先度は以下のように設定済みです**

  |DBインスタンス識別子|フェイルオーバー優先度|プライマリーインスタンス?|
  |---:|:---|:---|
  |ap96|2|Primary Instance|
  |ap96-XXX-1c|2|Replica|
  |ap96-XXX-1d|1|Replica|
  |application-autoscaling-xxx|15|Replica(Auto Scalingで追加。  既に削除されている可能性もあります)|


### 5-3-1. イベントサブスクリプションの設定
  * イベントサブスクリプションを設定しフェイルオーバー発生の通知をメールで受け取れるようにします。
  * 「サービス」→「RDS」→「イベントサブスクリプション」→「イベントサブスクリプションの作成」ボタンをクリック
  * 「イベントサブスクリプションの作成」画面にて以下を設定

    * 詳細ペイン

    ```
    名前（サブスクリプションの名前） : failover
    ```

    * ターゲットペイン

    ```
    通知の送信先 : 「新しいEメールトピック」ラジオボタンを選択
    トピック名　 : failover
    受信者　　　 : Confirmation可能な実在するメールアドレス
    ```

    * ソースペイン

    ```
    ソースタイプ : ドロップダウンメニューから「DBクラスター」を選択
    DBクラスターを含める : 「個別に選択 db クラスター」ラジオボタンを選択
    指定DBクラスター　　 : ドロップダウンメニューから「 ap96-cluster 」を選択
    含まれるイベントカテゴリー : 「特定のイベントカテゴリを選択します」ラジオボタンを選択
    特定のイベント　　　　　　 : ドロップダウンメニューから「 **フェイルオーバー** 」を選択
    ```
    * 「作成」ボタンをクリックします。

    * 「ターゲットペイン」の「受信者」で設定したメールアドレスへ、Confirmationメールが届きますので、メールのリンクをクリックしてConfirmationします。
      * なお、Confirmationできない誤ったメールアドレスを設定した場合には、Pending Confirmationとなり3日後に自動削除されます。
      * メールアドレスのConcifmation状況は以下の方法で確認できます。
        * 「サービス」→「Simple Notification Service」→「サブスクリプション」→「サブスクリプション」画面 →
        * 「サブスクリクション」画面のフィルターボックスで **PendingConfirmation** と入力するとPending中のConfirmation一覧を確認することができます。


### 5-3-2. 手動フェイルオーバー
  * 手動フェイルオーバーを行い、フェイルオーバー優先度:1の **ap96-XXX-1d** インスタンスがプライマリーインスタンスへ昇格することを確認してください。
    * 「詳細」ペインの「フェイルオーバー優先順位」を参照している画面上部の「インスタンス操作」メニュー → 「フェイルオーバー」→ 「Failover DB Cluster」画面 → 「フェイルオーバー」ボタンをクリックします。
    * 「サイドバー」→「クラスター」→「DBクラスター識別子」列下のDBクラスター名のハイパーリンク → 「DB クラスターメンバー」ペイン → 「更新」ボタン
      * レプリカ　**ap96-XXX-1d** のロールが、読み込みから書き込みへ昇格したことを確認します。

  * フェイルオーバーが発生後、イベントサブスクリクションで登録したメールアドレスへフェイルオーバーの通知が届きます。どのような内容のメールが届くのかご確認ください。
    * 「最近のイベント」ペインに以下のようなイベントが記録されていることも確認します。

      ```
      Completed failover to DB instance: ap96-XXX-1d
      Started cross AZ failover to DB instance: ap96-XXX-1d
      ```


## お疲れ様でした。これでハンズオンは終了です。
## **ハンズオン環境の削除等はこちらで行います。そのままにしておいていただいて結構です。**
