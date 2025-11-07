---
https://developers.prtimes.jp/2024/02/19/unit-test-improvement-with-simple-seeder-implementation/
---

# 素朴なSeederを作成してUnitテストの実装効率を上げた話

**Author:** 柳 龍哉  
**Date:** 2024年2月19日

## 前提

PR TIMESのバックエンドは、ノンフレームワークのPHPで実装されており、PDOと[TetoSQL](https://github.com/BaguettePHP/TetoSQL)を使用してデータベースを操作しています。Unitテストは[PHPUnit](https://github.com/sebastianbergmann/phpunit/)を使用し、データベースをモックしない方針を取っています。

## 従来のテストデータの作成方法

これまでは、テストデータをinsertするSQLファイルを作成し、PDOのexecを使用してデータベースにSQLを実行していました。

```php
// 従来のテストデータ挿入方法の例
INSERT INTO company ("id", "name", "zip", "address", "phone", "industry_type",
                     "description", "cover_image", "logo_image")
VALUES (1, "株式会社テスト", "000-0000", "東京都", "000-0000-0000",
        "サービス業", "事業内容について", "/cover_image/test.png", "/logo_image/test.jpeg");
```

## 実装したSeederについて

PDO、PrSql、[Faker](https://github.com/FakerPHP/Faker)を使用し、次のような Seeder を実装しました。

```php
class CompanySeeder
{
    public static function seed(PDO $pdo, array $companyData, int $rowQuantity = 1): void
    {
        $faker = Factory::create("ja_JP");

        for ($i = 0; $i < $rowQuantity; $i++) {
            PrSql::execute($pdo, "INSERT INTO company", [
                "name" => $companyData["name"] ?? $faker->company(),
                "zip" => $companyData["zip"] ?? $faker->postcode(),
                "address" => $companyData["address"] ?? $faker->prefecture() . $faker->city(),
                "phone" => $companyData["phone"] ?? $faker->phoneNumber(),
                "industry_type" => $companyData["industry_type"] ?? "サービス業",
                "description" => $companyData["description"] ?? $faker->text(200),
                "cover_image" => $companyData["cover_image"] ?? "/cover_image/test.png",
                "logo_image" => $companyData["logo_image"] ?? "/logo_image/test.jpeg"
            ]);
        }
    }
}
```

この Seeder の特徴は以下の通りです。

- 必要な項目のみを指定してテストデータを作成可能
- Fakerを使用してランダムなダミーデータを自動生成
- 複数行のデータを一度に作成可能

## Seederを使用したテストの例

```php
class CompanyTest extends TestCase
{
    public function testCompanyCreation(): void
    {
        // 特定の値を指定してテストデータを作成
        CompanySeeder::seed($this->pdo, [
            "name" => "テスト株式会社",
            "industry_type" => "IT業界"
        ]);

        // テストロジック
        $company = CompanyRepository::findByName($this->pdo, "テスト株式会社");
        $this->assertEquals("IT業界", $company->getIndustryType());
    }
}
```

## 効果

1. **テストデータ作成の簡素化**: 必要最小限の情報のみ指定すればよい
2. **メンテナンス性の向上**: SQLファイルの管理が不要
3. **テストの可読性向上**: テストコード内でデータの作成意図が明確
4. **開発効率の向上**: Fakerによる自動データ生成で時間短縮

この素朴な Seeder 実装によって SQL ファイルを別途用意する手間がなくなり、テストを追加するときは Seeder 呼び出しを 1 行書くだけで済むようになりました。
