---
name: aws-evidence
description: AWS CLIでVPC・サブネット・セキュリティグループ(inbound/outbound)・S3(ライフサイクル/暗号化/バックアップ/アクセスログ)の設定エビデンスを取得し、Markdownの表で出力する。「AWSのエビデンス取得」「設定内容の棚卸し」「VPC/SG/S3の設定一覧」を求められたときに使う。
---

# AWS設定エビデンス取得

AWS CLIの**読み取り専用コマンド(describe/get/list)のみ**を使い、結果をMarkdown表で出力する。リソースの作成・変更・削除は絶対に行わない。

## 前提
- AWS CLI導入済み、`aws login` で認証済み
- 最初に認証とリージョンを確認し、出力の冒頭に記載する

```bash
aws sts get-caller-identity --output json
aws configure get region   # 未設定ならユーザーに対象リージョンを確認する
```

- 対象リージョン・プロファイルが指定されていれば `--region` / `--profile` を全コマンドに付ける
- 権限不足(AccessDenied)や未設定は「取得不可(理由)」として表に記載し、処理は継続する

## 取得手順

### 1. VPC
```bash
aws ec2 describe-vpcs --query 'Vpcs[].{VpcId:VpcId,Name:Tags[?Key==`Name`]|[0].Value,Cidr:CidrBlock,State:State,Default:IsDefault}' --output json
```

### 2. サブネット
```bash
aws ec2 describe-subnets --query 'Subnets[].{SubnetId:SubnetId,Name:Tags[?Key==`Name`]|[0].Value,VpcId:VpcId,Cidr:CidrBlock,AZ:AvailabilityZone,PublicIp:MapPublicIpOnLaunch}' --output json
```

### 3. セキュリティグループ(inbound/outbound)
```bash
aws ec2 describe-security-groups --output json
```
`IpPermissions`(inbound)と`IpPermissionsEgress`(outbound)を、ルール1件=1行に展開する。
- プロトコル: `IpProtocol` が `-1` は `all`
- ポート: `FromPort-ToPort`(同値なら単一、`-1`/未指定は `all`)
- 送信元/宛先: `IpRanges[].CidrIp`、`Ipv6Ranges[].CidrIpv6`、`UserIdGroupPairs[].GroupId`、`PrefixListIds[].PrefixListId` をすべて展開
- 説明: 各要素の `Description`

### 4. S3
```bash
aws s3api list-buckets --query 'Buckets[].Name' --output text
```
各バケットについて以下を実行する(バケットのリージョンは `aws s3api get-bucket-location --bucket <b>` で確認し、`--region` に指定)。

| 項目 | コマンド | 「未設定」の判定 |
|---|---|---|
| ライフサイクル | `aws s3api get-bucket-lifecycle-configuration --bucket <b>` | `NoSuchLifecycleConfiguration` |
| 暗号化 | `aws s3api get-bucket-encryption --bucket <b>` | `ServerSideEncryptionConfigurationNotFoundError` |
| バージョニング | `aws s3api get-bucket-versioning --bucket <b>` | `Status` なし = 無効 |
| レプリケーション | `aws s3api get-bucket-replication --bucket <b>` | `ReplicationConfigurationNotFoundError` |
| アクセスログ | `aws s3api get-bucket-logging --bucket <b>` | `LoggingEnabled` なし = 無効 |

**バックアップ**はS3単体の設定ではないため、次を合わせて「バックアップ」列にまとめる。
- バージョニング状態、レプリケーション有無
- AWS Backupの保護対象か:
  ```bash
  aws backup list-protected-resources --query "Results[?ResourceType=='S3'].ResourceArn" --output json
  ```
  対象バケットのARN(`arn:aws:s3:::<b>`)が含まれれば「AWS Backup: 対象」

「未設定」はエラーではなく有効な結果なので、表には「未設定」と記載する。

## 出力形式

Markdownで、見出しごとに表を出力する。取得日時・アカウントID・リージョンを冒頭に書く。

```markdown
# AWS設定エビデンス
- 取得日時: YYYY-MM-DD HH:MM
- アカウントID: 123456789012
- リージョン: ap-northeast-1

## VPC
| VPC ID | 名前 | CIDR | 状態 | デフォルト |

## サブネット
| サブネットID | 名前 | VPC ID | CIDR | AZ | パブリックIP自動割当 |

## セキュリティグループ
### <GroupId> (<GroupName>) / VPC: <VpcId>
| 方向 | プロトコル | ポート | 送信元/宛先 | 説明 |
(inbound / outbound を1行ずつ)

## S3
| バケット | リージョン | ライフサイクル | 暗号化 | バックアップ | アクセスログ |
```

- ライフサイクル列: ルールID・状態・要約(例: `rule1 Enabled: 30日後にGLACIER、365日後に削除`)
- 暗号化列: アルゴリズムとKMSキー(例: `SSE-S3 (AES256)` / `SSE-KMS (<key>)`)
- アクセスログ列: 出力先バケット/プレフィックス、または `未設定`
- セレクション中の値が空の場合は `-` を記載する

## 注意
- 0.0.0.0/0 や ::/0 を許可するinboundルール、暗号化・ログ未設定のバケットは、表の後に「要確認事項」として箇条書きで補足する(判断は行わず事実のみ)
- バケット数が多い場合も省略せず全件出力する
