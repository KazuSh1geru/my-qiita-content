---
title: CRMデータ移行ツール開発の苦労話 ~ Transform編 ~
tags:
  - JavaScript
  - BigQuery
  - GoogleCloud
private: false
updated_at: '2024-07-01T19:17:58+09:00'
id: 018f979689113a954bbd
organization_url_name: null
slide: false
ignorePublish: false
---
データ移行プロジェクトの中で直面した課題とその解決方法について、備忘録としてまとめました。

関連記事

https://qiita.com/kusaaaaagi/items/926f57c6f840e9a8fd9e


## プロジェクトの概要
CRMの環境刷新に伴い、新環境へのデータ移行ツールの開発依頼を受けました。
その際、ただ移行するのではなくオブジェクト構造を考慮したり、いらないデータを絞り込みしたり、名寄せをしたり、などとデータの加工要件をいただきました。
つまり、ただデータを旧環境から新環境に横流しだけでは要件を満たさないため、ETLを行いました。
また、BigQueryに関して知見があったこと、大規模データ加工が要件だったことがあり、TransformにはBigQueryを採用しました。
- Extract: Cloud Data Fusionを使用して、CRMからデータをBigQueryに転送する
- Transform: BigQuery / Dataform を使用し、データ加工する
- Load: [mitocoX](https://www.terrasky.co.jp/eai/dataspidercloud/)を使用して、BigQueryからCRMに転送する


## この記事では

Transformに対象を絞って、まとめます。

# 苦労した点
## 1. データ品質のテスト戦略
データの品質を保証するためのテスト戦略を構築することは、非常に難しいタスクでした。特に、変換後のデータが期待通りの品質を維持しているかを確認することが重要です。

### 解決策: Assertionの活用
Dataformのassertion機能を利用しました。assertionを用いることで、変換後のデータが特定の条件を満たしているかを自動的にチェックできるようにしました。これにより、データ品質の維持が効率的に行えるようになりました。
asserion SQL クエリはゼロ行を返す必要があります。実行時に行が返される場合、アサーションは失敗します。

https://cloud.google.com/dataform/docs/assertions?hl=ja#manual

Assertionの中でも2種類のassertionを利用しました。

**組み込みassertion**

SQLXワークフロー内で下記のように、assertionを組み込みができます。
ユニークキー制約、Null制約、条件制約を簡単に組み込めるので基礎的な品質は組み込みアサーションでチェックします。

```javascript
config {
  type: "table",
  assertions: {
    uniqueKey: ["user_id"],
    nonNull: ["user_id", "customer_id"],
    rowConditions: [
      'signup_date is null or signup_date > "2019-01-01"',
      'email like "%@%.%"'
    ]
  }
}
SELECT ...
```

**手動アサーション**
手動アサーションは、専用の SQLX ファイルに書き込む SQL クエリです。
上記の組み込みassertionとは異なり、SQLXを作成するので表現力が高いチェックSQLを作成できます。
特定のテーブル間の件数一致や、参照関係のチェックなどを実装しました。

`type: "assertion"` で宣言すると、SQLXファイルを手動アサーションとして扱えます.

```javascript
config {
    type: "assertion",
    name: "assertion_hogehoge",
}
```



## 2. タイムスタンプのタイムゾーン対応
データの中にはタイムスタンプを含むものが多くあり、これを日本標準時（JST）に対応させる必要がありました。異なるタイムゾーン間でのデータ変換は、しばしば誤りやズレを引き起こす原因となります。

### 解決策: JST対応の標準化
BigQueryの機能を利用し、全てのタイムスタンプをJSTに変換するロジックを組み込みました。これにより、全てのデータが統一されたタイムゾーンで処理され、タイムゾーンによるデータの不整合を防ぎました。

具体的には、下記のようなjavascriptを作成しました。

```javascript: updateTimestampForJST.js
const TIMESTAMP_MAPPINGS = [{
  "tableName": "Campaign",
  "timestampColumns": ["CreatedDate", "LastModifiedDate", "SystemModstamp", "LastViewedDate", "LastReferencedDate"]
}, ...
{
  "tableName": "CampaignMember",
  "timestampColumns": ["CreatedDate", "LastModifiedDate", "SystemModstamp", "LastViewedDate", "LastReferencedDate"]
}];

/**
 * Update文を生成する.
 */
function renderUpdateSQL(tableName, timestampColumns) {
    let setSQL = "";
    timestampColumns.forEach((column) => {
        if (setSQL !== "") {
        setSQL += ",";
        }
        setSQL += `${column} = TIMESTAMP_ADD(${column}, INTERVAL 9 HOUR)`;
    });
    return `UPDATE ${tableName} SET ${setSQL} WHERE true`;
}

// テーブルごとにUPDATEの処理をする
TIMESTAMP_MAPPINGS.forEach((obj) => {
    const tableName = obj.tableName;
    const timestampColumns = obj.timestampColumns;
    operate(`update_timestamp_${tableName}`)
    .queries(ctx => 
        renderUpdateSQL(`${ctx.ref("data_transform", tableName)}`, timestampColumns)
    )
});
```

上記のタイムスタンプ更新スクリプトをSQLXワークフローで依存を設定して、指定したタイミングで実行されるようにしました。

```javascript
config {
  type: "table",
  schema: "data_transform",
  name: "Campaign",
  // 依存関係を宣言
  dependencies: ["update_timestamp_Campaign"],
}

```

## 3. 加工の戦略構築
データの変換には、様々な要件が存在し、それらを適切に整理・構造化することが求められました。特に、複雑な加工要件をどのように効率的に管理するかが課題となりました。

### 解決策: 加工戦略の構成
データ加工の戦略として、3つの構成を作成しました。
| 処理カテゴリ | 説明                                                            | 
| ------------ | --------------------------------------------------------------- | 
| convert      | 一般的な変換の実装<br>（マスク, データ型変換, API参照名の変更） | 
| customize    | 特殊な加工要件の実装                                            | 
| filter       | データの絞り込み                                                | 


特に、customize部分では、複雑な加工要件を切り出したことがプロジェクト全体を通してうまくハマりました。
変更頻度が高い箇所を限定することで、メンテナンス性を向上させました。

## 4. 共通化の実現
複数のデータ変換プロセスにおいて、共通化できる部分を見つけ、それを効率的に再利用することが求められました。

### 解決策: Javascriptの利用
DataformでJavascriptを利用し、共通化可能なロジックをモジュール化しました。これにより、同様の処理を複数の場所で再利用することができ、開発効率が大幅に向上しました。

#### ケース1. 固定値の切り替え
特定のユーザーIDでデータを書き換えて欲しいというユースケースがありました。
またCRMの開発/本番環境でユーザーIDが異なるのでそれを使い分けるロジックを共通化しました。

```javascript: constants.js
const USER_ID_DEV = '000xxxxxxAT';
const USER_ID_PROD = '000xxxxxxBT';
/**
 * 環境別のuser_idを返却する.
 * @return {string} - 環境別のmitocox_user_id（SFDCのオブジェクトID）
*/


function getUserId() {
    const env = dataform.projectConfig.vars.env;
    // 環境別のIDを返却
    if (env === "dev") {
      return USER_ID_DEV;
    } else {
      return USER_ID_PROD;
    }
}
```

下記のように固定値の切り替えをjavascriptに寄せることで、意識せずにSQLを書くことができます。
```SQL
select
'${constants.getUserId()}') AS OwnerId,
...
```

#### ケース2. 環境間のデータセットの切り替え

```javascript: sourceDataset.js
/**
 * stationの元データセットの文字列を返却する.
 * @return {string} - data_station | data_station_fullsandbox
 */
function getStationSourceDataset() {
  return `${dataform.projectConfig.vars.sourceType}` === "fullsandbox" ? "dataset_fullsandbox" : "dataset_prod";
}

const stationSourceDataset = getStationSourceDataset();

module.exports = {
  stationSourceDataset
};
```

上記のjavascriptによってSQL開発時にデータセットの向き先を意識せずに開発できるようになりました。
```SQL
SELECT
  *
FROM
-- データセットの宣言をsourceDataset.jsで行う.
  ${ref(`${sourceDataset.stationSourceDataset}`, "Account")}
```
#### ケース3. 繰り返し現れるSQL文の共通化

更新日によって環境間で最新のデータのみを移行して欲しいという要件がありました。
CASE文で実装することは容易ですが、繰り返し現れるため可読性やメンテナンス性が下がりやすいことを懸念してSQLのレンダリングをjavascriptで実現しました。

```javascript
/**
 * 更新日（LastModifiedDate）が新しいレコードの値を移行するロジック。
 * @param {string} targetValueName - 対象の値の名前。
 * @param {string} [lastModifiedDate="LastModifiedDate"] - 最終更新日のフィールド名。デフォルトは"LastModifiedDate"。
 * @returns {string} - SQL CASE文。
 */
function renderCleanseCaseByLastModifiedDate(targetValueName, lastModifiedDate = "LastModifiedDate") {
  return `CASE
    -- 環境Aに値がなければ、環境Bの値を取得
    WHEN a.${lastModifiedDate} is null THEN b.${targetValueName}
    -- 環境Bに値がなければ、環境Aの値を取得
    WHEN b.${lastModifiedDate} is null THEN a.${targetValueName}
    -- 同一時刻である場合、環境Aを優先する
    WHEN a.${lastModifiedDate} >= b.${lastModifiedDate} THEN a.${targetValueName}
    -- そうでない（環境Bの最終更新日が新しい）場合、環境Bの値を取得する
    ELSE b.${targetValueName}
END AS ${targetValueName}
    `;
}
```

#### ケース4. SQLXをjavascriptで生成する
Dataformではjavascriptを用いてSQLXワークフローを作成できます。

https://cloud.google.com/dataform/docs/develop-workflows-js?hl=ja

同じようなSQLをベタ書きするのではなく、SQLの生成を共通化しました。
例えば下記のように、マスク処理が正しくできているかテストするAssertionテーブルの生成を共通化しました。

```javascript: assertion_masked.js
const CHECK_TABLES = [
    "Campaign",
    ...
    "CampaignMember",
];

const CHECK_DEFINITIONS = CHECK_TABLES.reduce((acc, tableName) => {
    const {
        MASK_COLUMNS
    } = require(`includes/config/maskdata/${tableName.toLowerCase()}`);
    acc[tableName] = MASK_COLUMNS;
    return acc;
}, {});

Object.entries(CHECK_DEFINITIONS)
    .map(([tableName, maskColumns]) => {
        let assertion = assert(`assert_masked_${tableName}`)
            .description(`Check that values in columns (${maskColumns}) in ${tableName} form a mask Value`)
            .tags("ci")
            .query(ctx => `${testUtils.renderCheckMaskColumnSQL(`${ctx.ref("data_transform", tableName)}`, maskColumns)}`)
        return assertion
    }
)

```

# まとめ
Transformフェーズにおいては、データ品質の保証、タイムゾーン対応、加工戦略の構築、共通化の実現という4つの主要な課題がありましたが、それぞれ適切な技術と方法を用いることで効果的に解決することができました。これにより、ETLツール全体の信頼性と効率性が向上し、運用面でも大きな効果を得ることができました。

