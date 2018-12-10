---
title: コンソールからEC2のセキュリティグループを編集する
description: コンソールからEC2のセキュリティグループを確認・編集する
date: 2018-12-10T04:00:13.926Z
thumbnail: /images/uploads/013elly18420_tp_v4.jpg
categories:
  - aws
  - ec2
---
 [JSL \(日本システム技研\) Advent Calendar 2018 \- Qiita](https://qiita.com/advent-calendar/2018/jsl) の10日目の記事です。

## モチベーション

AWSのセキュリティグループの設定を月に何回か設定し直すことがあるが、コンソール上から設定するのが面倒。インバウンドのIPアドレスの追加/削除が主な設定内容となる。これをターミナルからコマンド一発で設定したい。

## 最終的にやったこと

aws-cliを利用し、以下のコマンドを実行することでことでセキュリティグループの確認、設定の追加/削除が可能。
```
# 確認
$ aws --profile [your-profile] ec2 describe-security-groups --group-id sg-1234xxxx --query "SecurityGroups[*].{Permissions:IpPermissions}" --output table
# 追加
$ aws --profile [your-profile] ec2 authorize-security-group-ingress --group-id sg-1234xxxx --ip-permissions IpProtocol=tcp,FromPort=3389,ToPort=3389,IpRanges='[{CidrIp=127.0.0.1/32,Description="test rule"}]'
# 削除
$ aws --profile [your-profile] ec2 revoke-security-group-ingress --group-id sg-1234xxxx --ip-permissions IpProtocol=tcp,FromPort=3389,ToPort=3389,IpRanges='[{CidrIp=127.0.0.1/32,Description="test rule"}]'
```

---

## 解説

上記コマンドまでの経緯及び解説。

### 設定する項目

ここで設定する項目は以下としておく。

| タイプ | プロトコル | ポート範囲 | ソース | 説明 |
|---|---|---|---|---|
|SSH  |TCP  |22  | 127.0.0.1 | テスト項目 |

通常、説明は英数のみしか受け付けないようになっているが、cliからだとどうなるのか？テストも兼ねる。

### aws-cli

セキュリティグループはEC2に紐づくものなので、EC2に対するaws-cliのコマンドリファレンスを見る。

[ec2 — AWS CLI 1\.16\.72 Command Reference](https://docs.aws.amazon.com/cli/latest/reference/ec2/index.html)

この辺り使えば実現できそう。

[authorize\-security\-group\-ingress — AWS CLI 1\.16\.72 Command Reference](https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html)

[revoke\-security\-group\-ingress — AWS CLI 1\.16\.72 Command Reference](https://docs.aws.amazon.com/cli/latest/reference/ec2/revoke-security-group-ingress.html)

### セキュリティグループの確認

インバウンドのルールを設定するために、設定したいセキュリティグループのグループIDが必要となるので、こちらもaws-cliで確認する。

[describe\-security\-groups — AWS CLI 1\.16\.72 Command Reference](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html)

セキュリティグループの一覧を見る。

```bash
# json
$ aws --profile [your-profile] ec2 describe-security-groups --output json
# table
$ aws --profile [your-profile] ec2 describe-security-groups --output table
```

セキュリティグループが多い場合、上記のコマンドをそのまま叩くとかなり見づらい。そういうときは、 `--query` を使うと見やすくなる。

[describe\-security\-groups#Examples — AWS CLI 1\.16\.72 Command Reference ](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-security-groups.html#examples)

```bash
# json
$ aws --profile [your-profile] ec2 describe-security-groups --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}" --output json
# table
$ aws --profile [your-profile] ec2 describe-security-groups --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}" --output table

--------------------------------------------------------------------------------------
|                               DescribeSecurityGroups                               |
+-----------------------+------------------------------------------------------------+
|          ID           |                           Name                             |
+-----------------------+------------------------------------------------------------+
|  sg-1234xxxx          |  launch-xxxxxx-x                                           |
~~~
```

セキュリティグループIDはこんな感じの文字列 `sg-1234xxxx`

グループ詳細も見れることを確認しておく。

```bash
# json
$ aws --profile [your-profile] ec2 describe-security-groups --group-id sg-1234xxxx --output json
# table
$ aws --profile [your-profile] ec2 describe-security-groups --group-id sg-1234xxxx --output table
```

さらにグループのインバウンドの制限設定のみ取得してみる。

```bash
$ aws --profile [your-profile] ec2 describe-security-groups --group-id sg-1234xxxx --query "SecurityGroups[*].{Permissions:IpPermissions}" --output table
-------------------------------------------
|         DescribeSecurityGroups          |
||              Permissions              ||
|+------------+---------------+----------+|
||  FromPort  |  IpProtocol   | ToPort   ||
|+------------+---------------+----------+|
||  22        |  tcp          |  22      ||
|+------------+---------------+----------+|
|||              IpRanges               |||
||+---------------------+---------------+||
|||       CidrIp        |  Description  |||
||+---------------------+---------------+||
|||  x.x.x.x/32         |               |||
~~~
```

### 設定を追加する

グループIDが分かったので、 `authorize-security-group-ingress` コマンドを使って設定を追加する。

まずはport, protocol, cidrオプションを使って設定してみる。

```bash
$ aws --profile [your-profile] ec2 authorize-security-group-ingress --group-id sg-1234xxxx --protocol tcp --port 22 --cidr 127.0.0.1/32
```

確認してみる
```bash
$ aws --profile [your-profile] ec2 describe-security-groups --group-id sg-1234xxxx --query "SecurityGroups[*].{Permissions:IpPermissions}" --output table
-------------------------------------------
|         DescribeSecurityGroups          |
||              Permissions              ||
|+------------+---------------+----------+|
||  FromPort  |  IpProtocol   | ToPort   ||
|+------------+---------------+----------+|
||  22        |  tcp          |  22      ||
|+------------+---------------+----------+|
|||              IpRanges               |||
||+---------------------+---------------+||
|||       CidrIp        |  Description  |||
||+---------------------+---------------+||
~~~
|||  127.0.0.1/32       |               |||
||+---------------------+---------------+||
```

追加されているのでよさそう。

消しておく。

```bash
$ aws --profile [your-profile] ec2 revoke-security-group-ingress --group-id sg-1234xxxx --protocol tcp --port 22 --cidr 127.0.0.1/32
```

---

次にdescriptionありの場合で追加してみる。

```bash
$ aws --profile [your-profile] ec2 authorize-security-group-ingress --group-id sg-1234xxxx --protocol tcp --port 22 --cidr 127.0.0.1/32 --description 'test'
Unknown options: --description, test
```

`--description` オプションはないっぽい。

リファレンスみると `--ip-permissions` にリストを渡すことで実現できそうなので試してみる。

```bash
$ aws --profile [your-profile] ec2 authorize-security-group-ingress --group-id sg-1234xxxx --ip-permissions IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges='[{CidrIp=127.0.0.1/32,Description="test rule"}]'
```

確認してみる。

```bash
$ aws --profile [your-profile] ec2 describe-security-groups --group-id sg-1234xxxx --query "SecurityGroups[*].{Permissions:IpPermissions}" --output table
-------------------------------------------
|         DescribeSecurityGroups          |
||              Permissions              ||
|+------------+---------------+----------+|
||  FromPort  |  IpProtocol   | ToPort   ||
|+------------+---------------+----------+|
||  22        |  tcp          |  22      ||
|+------------+---------------+----------+|
|||              IpRanges               |||
||+---------------------+---------------+||
|||       CidrIp        |  Description  |||
||+---------------------+---------------+||
~~~
|||  127.0.0.1/32       | test rule     |||
||+---------------------+---------------+||
```

さらに日本語でもうまくいくのか試してみる。

```bash
$ aws --profile [your-profile] ec2 authorize-security-group-ingress --group-id sg-1234xxxx --ip-permissions IpProtocol=tcp,FromPort=3389,ToPort=3389,IpRanges='[{CidrIp=127.0.0.1/32,Description="テスト項目"}]'
An error occurred (InvalidParameterValue) when calling the AuthorizeSecurityGroupIngress operation: Invalid rule description. Valid descriptions are strings less than 256 characters from the following set:  a-zA-Z0-9. _-:/()#,@[]+=&;{}!$*
```

だめだった・・

なお、 `--dry-run` オプションを使うと実際にコマンドを実行することなく、コマンドが実行可能かを返してくれる。

```bash
$ aws --profile [your-profile] ec2 authorize-security-group-ingress --dry-run --group-id sg-1234xxxx
An error occurred (DryRunOperation) when calling the AuthorizeSecurityGroupIngress operation: Request would have succeeded, but DryRun flag is set.
```

--- 

## おまけ

実際は現在のグローバルIPをインバウンドに追加するというケースが多いと思う。

以下のグローバルIPを確認するサイトを覚えておくと便利。

[アクセス情報【使用中のIPアドレス確認】](https://www.cman.jp/network/support/go_access.cgi)

[ifconfig\.io](https://ifconfig.io/)

[グローバルIP] で検索するとトップに出てくるのは前者だが、後者のほうはAPIがあるのでプログラミング的に便利。curl等でGETリクエストを送ると、現在のグローバルIPが返ってくる。

```bash
$ curl ifconfig.io
x.x.x.x
```
