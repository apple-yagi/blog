---
https://developers.prtimes.jp/2024/10/10/clean-phpunit-log-output/
---

# PHPUnitの実行時に出ていた不要なログを出力しないようにした話

**Author:** 柳 龍哉 ([@apple_yagi](https://twitter.com/apple_yagi))
**Date:** 2024年10月10日

## はじめに

こんにちは、フロントエンドエンジニアのやなぎです。PR TIMESのバックエンドではPHPUnitを使用してUnitテストが書かれていますが、テスト実行時に不要なログが大量に出力されていました。

## 対策方法

### 1. Notice Errorを出力しないようにする

CI では Notice Error を無視するように設定しました。設定例は次の通りです。

```php
$isCi = getenv('CI') === 'true';
if ($isCi) {
    error_reporting(E_ALL & ~E_NOTICE);
}
```

### 2. NullLoggerを使用する

テスト時にログを出力しないよう、`Psr\Log\NullLogger` を使用しています。

```php
class UserServiceTest extends TestCase
{
    public function test_存在しないユーザーIDを指定した時、nullが返ってくること()
    {
        $logger = new Psr\Log\NullLogger();
        $user = UserService::getById(0, $logger);
        
        $this->assertNull($user);
    }
}
```

### 3. LoggerにNullHandlerを渡す

`Monolog\Logger` を利用する場合は、`Monolog\Handler\NullHandler` を併用します。

```php
class PressReleaseServiceTest extends TestCase
{
    public function test_存在しないプレスリリースIDを指定した時、nullが返ってくること()
    {
        $logger = new Monolog\Logger('_');
        $null_handler = new Monolog\Handler\NullHandler();
        $logger->pushHandler($null_handler);
        
        $pressRelease = PressReleaseService::getById(0, $logger);
        
        $this->assertNull($pressRelease);
    }
}
```

## まとめ

これらの対策により、PHPUnitの実行時に出力される不要なログを削減し、テスト結果が見やすくなりました。
