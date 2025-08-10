---
https://developers.prtimes.jp/2025/02/26/snapshot-testing-with-phpunit-snapshot-assertions/
---

# phpunit-snapshot-assertionsを使用してスナップショットテストを導入した話

こんにちは、フロントエンドエンジニアのやなぎ（[@apple\_yagi](https://twitter.com/apple_yagi)）です。

PR TIMESではプレスリリース検索などを行う際に[OpenSearch](https://opensearch.org/)を使用しており、PHPで生成した検索クエリをOpenSearchに送信して、検索を行なっています。検索クエリの生成はコードを読むことである程度はわかるものの、複雑なものになると最終的にどのようなクエリが出来上がるのかわかりにくいという問題がありました。その問題を解決するために、先日 [spatie/phpunit-snapshot-assertions](https://github.com/spatie/phpunit-snapshot-assertions) を使用したスナップショットテストを導入したのでご紹介します。

目次

## spatie/phpunit-snapshot-assertions とは

GitHub

A way to test without writing actual test cases. Contribute to spatie/phpunit-snapshot-assertions development by creating an account on GitHub.

spatie/phpunit-snapshot-assertionsは、PHPUnitでスナップショットテストを実行するためのライブラリです。このライブラリを使用することで、コードの出力をスナップショットとして保存し、今後のテストにおいてその出力が基準から変わっていないかを容易に検証することが可能になります。以下に、具体的な使用例を示します。

```
<?php

declare(strict_types=1);

namespace PRTIMES\PrTimes\Tests;

use Spatie\Snapshots\MatchesSnapshots;

class ExampleTest extends TestCase
{
    use MatchesSnapshots;

    public function test_example()
    {
        $this->assertMatchesSnapshot('example');
    }
}
```

上記のテストを実行すると、テストファイルと同じディレクトリ内に\_\_snapshots\_\_というディレクトリが生成され、その中にスナップショットが保存されます。このスナップショットは、次回のテスト実行時に比較対象として利用されます。

assertMatchesSnapshot を使用した場合に保存されるスナップショットファイル（例：ExampleTest\_\_test\_example\_\_1.php）の内容は、以下のようになります。

## OpenSearchの検索クエリを生成するPHPコード

以下のコードは、OpenSearchの検索クエリを生成する際の例です。

```
<?php

declare(strict_types=1);

namespace PRTIMES\PrTimes;

class PressReleaseSearchQueryFactory
{
    public static function build(
        array $business_category_ids,
        array $company_category_ids,
        array $prefecture_ids,
        DateTimeImmutable $start_timestamp,
        DateTimeImmutable $end_timestamp
    ): string
    {
        $sort = new Sort();
        $filter = new Filter();
        $should = new Should();
        $must = new Must();
        $must_not = new MustNot();

        if (count($prefecture_ids) > 0) {
            $filter->pref_id_terms = new PrefIDTerms($prefecture_ids);
        }

        if (count($business_category_ids) > 0) {
            $filter->categories_terms = new Categories($business_category_ids);
        }

        if (count($company_category_ids) > 0) {
            $filter->company_categories_terms = new CompanyCategories($company_category_ids);
        }

        if (isset($start_timestamp) && isset($end_timestamp)) {
            $range = new QueryCompleteTimestampFiltersStruct();
            $range->gte_timestamp = $start_timestamp->format(DateTimeInterface::ATOM);
            $range->lte_timestamp = $end_timestamp->format(DateTimeInterface::ATOM);

            $filter->complete_timestamp_filters = $range;
        }

        $search_body = new PressReleaseSearchQuery($sort, $filter, $should, $must_not, $must);
        return $search_body->toJson();
    }
}
```

実際のプロダクションコードとは若干異なりますが、クラスベースで検索クエリを生成することで、可読性の向上を図っています。

## スナップショットテストを導入する前のUnitテスト

スナップショットテストを導入する前のUnitテストの書き方は大きく分けて2つあり、テスト方法が統一されていませんでした。

1つ目のテスト方法は以下のようにJSON文字列を比較する方法です。このテスト方法はシンプルですが、検索クエリが複雑になると $expected の文字列が長くなり、可読性が悪くなってしまいます。また、手動でJSON文字列を記述するのは時間がかかってしまいます。

```
<?php

declare(strict_types=1);

namespace PRTIMES\PrTimes\Tests;

use PRTIMES\PrTimes\PressReleaseSearchQueryFactory;

class PressReleaseSearchQueryFactoryTest extends TestCase
{
    public function testBuild(): void
    {
        $actual = PressReleaseSearchQueryFactory::build(
            ['01', '02'],
            ['03', '04'],
            ['05', '06'],
            new \DateTimeImmutable('2021-01-01 00:00:00'),
            new \DateTimeImmutable('2021-01-02 00:00:00')
        );

        $expected = '
        {
            "size": 0,
            "sort": [],
            "query": {
                "bool": {
                    "filter": {
                        "pref_id_terms": {
                            "terms": [
                                "01",
                                "02"
                            ]
                        },
                        "categories_terms": {
                            "terms": [
                                "03",
                                "04"
                            ]
                        },
                        "company_categories_terms": {
                            "terms": [
                                "05",
                                "06"
                            ]
                        },
                        "complete_timestamp_filters": {
                            "gte_timestamp": "2021-01-01T00:00:00+00:00",
                            "lte_timestamp": "2021-01-02T00:00:00+00:00"
                        }
                    }
                }
            }
        }
        ';

        $this->assertJsonStringEqualsJsonString($expected, $actual);
    }
}
```

2つ目のテスト方法は以下のように検索クエリの生成方法の詳細がテストコードに漏れているパターンです。このテスト方法では長いJSON文字列をメンテナンスするコストはかかりませんが、実装の詳細がテストコードに反映されているため、検索クエリの生成ロジックをリファクタリングした際にテストコードも修正する必要が出てきます。そのため、リファクタリング耐性の低いテストコードになっています。

```
<?php

declare(strict_types=1);

namespace PRTIMES\PrTimes\Tests;

use PRTIMES\PrTimes\PressReleaseSearchQueryFactory;

class PressReleaseSearchQueryFactoryTest extends TestCase
{
    public function testBuild(): void
    {
        $actual = PressReleaseSearchQueryFactory::build(
            ['01', '02'],
            ['03', '04'],
            ['05', '06'],
            new \DateTimeImmutable('2021-01-01 00:00:00'),
            new \DateTimeImmutable('2021-01-02 00:00:00')
        );

        $expected = new PressReleaseSearchQuery(
            new Sort(),
            new Filter(
                new PrefIDTerms(['05', '06']),
                new Categories(['01', '02']),
                new CompanyCategories(['03', '04']),
                new QueryCompleteTimestampFiltersStruct(
                    '2021-01-01T00:00:00+00:00',
                    '2021-01-02T00:00:00+00:00'
                )
            ),
            new Should(),
            new MustNot(),
            new Must()
        );

        $this->assertJsonStringEqualsJsonString($expected->toJson(), $actual);
    }
}
```

## スナップショットテストを導入した後のUnitテスト

スナップショットテストを導入したことにより、以下のようにシンプルなコードでテストを完結させることができるようになりました。長いJSON文字列をメンテナンスする必要もなく、実装の詳細に触れることもなくなりました。

```
<?php

declare(strict_types=1);

namespace PRTIMES\PrTimes\Tests;

use PRTIMES\PrTimes\PressReleaseSearchQueryFactory;
use Spatie\Snapshots\MatchesSnapshots;

class PressReleaseSearchQueryFactoryTest extends TestCase
{
    use MatchesSnapshots;

    public function testBuild(): void
    {
        $actual = PressReleaseSearchQueryFactory::build(
            ['01', '02'],
            ['03', '04'],
            ['05', '06'],
            new \DateTimeImmutable('2021-01-01 00:00:00'),
            new \DateTimeImmutable('2021-01-02 00:00:00')
        );

        $this->assertMatchesJsonSnapshot($actual);
    }
}
```

また、`assertMatchesJsonSnapshot`を使用することでスナップショットをJSONファイルで生成できるため、実際に生成された検索クエリをレビュー時に目視で確認することが可能です。

## まとめ

今回、PHPのUnitテストにスナップショットテストを導入しました。スナップショットテストを使用するケースはある程度限られますが、今回のようなケースやSQL Builderなどのテストには不具合がないと考えており、今後も適切なケースがあれば活用していきたいと思います。

## We are hiring

弊社ではバックエンドエンジニアはもちろん、各種ポジションで採用を行っています！

株式会社PR TIMES

株式会社PR TIMESでは現在03-3. 開発部 バックエンドエンジニアを募集しています。

株式会社PR TIMES

株式会社PR TIMESが公開している、02．開発部 の求人一覧です
