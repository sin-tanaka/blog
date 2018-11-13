---
title: '[AWS]コンソールからEC2のセキュリティグループを確認・編集する'
description: コンソールからEC2のセキュリティグループを確認・編集する
date: 2018-11-13T09:03:13.926Z
thumbnail: /images/uploads/013elly18420_tp_v4.jpg
categories:
  - aws
  - ec2
---

## 指定したセキュリティーグループの詳細

以下のコマンドでコマンドで指定した `group-id` のセキュリティーグループの詳細を確認することができます。

```bash
$ aws --profile [your_profile] ec2 describe-security-groups --group-id [sg-your_groupid] --output table
```

`--output` は `json` と `table` が指定可能です。 `json` を指定した場合、フォーマット済の `json` が出力されるので、 `jq` 等を使ったフォーマットは不要です。

```bash
$ aws --profile [your_profile] ec2 describe-security-groups --group-id [sg-your_groupid] --output json
```

## セキュリティーグループの一覧

`group-id` は以下コマンドで確認できます。

```bash
$ aws --profile [your_profile] ec2 describe-security-groups --query "SecurityGroups[].[GroupName,GroupId]" --output table
```
