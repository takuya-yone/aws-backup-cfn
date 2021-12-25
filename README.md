# AWS Backupテンプレート利用手順

## 1.汎用OL処理構成@東京リージョン
- 目的
    - 汎用OL処理を初期構築する
- 前提
    - RDSインスタンス用サブネットグループが作成済みであること

- 手順
    - CFnスタック作成 ``ap-northeast-1/1_olprocess_tokyo.yml``
        - パラメータを入力
            - ALB関連
                - ALBSecurityGroup：ALB用セキュリティグループ
                - ALBSubnetId1：ALB紐付け先サブネットID
                - ALBSubnetId2：ALB紐付け先サブネットID
                - ALBVPCId：ALB紐付け先VPC ID
            - RDS関連
                - DBInstanceIdentifier：RDSインスタンス識別子
                - DBInstanceUserId：RDS用Adminユーザ
                - DBInstancePassword：RDS用Adminパスワード
                - DBSecurityGroups：RDS用セキュリティグループ
                - DBSubnetGroupName：RDS用サブネットグループ名
            - EC2関連
                - EC2ImageId：EC2インスタンス作成用AMI ID
                - EC2SecurityGroupId：EC2用セキュリティグループ
                - EC2SubnetId：EC2デプロイ先サブネットID

- 作成されるリソース
    - ALB（リスナー、ターゲットグループ）
    - EC2インスタンス
    - RDSインスタンス

## 2.AWS Backup作成@大阪リージョン
- 目的
    - 東京リージョンからのデータ退避用AWS Backupリソースを作成する
- 前提
    - 無し
- 手順
    - CFnスタック作成 ``ap-northeast-3/2_awsbackup_osaka.yml``

- 作成されるリソース
    - AWS Backupボールト


## 3.AWS Backup作成@東京リージョン
- 目的
    - 東京リージョンにて、AWS Backupを構成し、バックアップ対象（EC2,RDS）とバックアップ計画を構成する
- 前提
    - 無し
- 手順
    - CFnスタック作成 ``ap-northeast-1/3_awsbackup_tokyo.yml``
        - パラメータを入力
            - EC2関連
                - EC2InstanceId：バックアップ対象のEC2インスタンスID
            - RDS関連
                - RDSInstanceId：バックアップ対象のRDSインスタンスArn

- 作成されるリソース
    - AWS Backupボールト
    - AWS Backupプラン
        - AWS Backupセレクション（EC2）
        - AWS Backupセレクション（RDS）

## 4.汎用OL処理構成復元@大阪リージョン
- 目的
    - AWS Backupで取得したEC2のAMI、RDSのスナップショットから大阪リージョンに汎用OL処理構成を復元する

- 前提
    - 3で作成したAWS Backupにより、EC2のAMI、RDSのスナップショットが取得されていること
    - RDSインスタンス用サブネットグループが作成済みであること
- 手順
    - CFnスタック作成 ``ap-northeast-3/4_restore_osaka.yml``
        - パラメータを入力
            - ALB関連
                - ALBSecurityGroup：ALB用セキュリティグループ
                - ALBSubnetId1：ALB紐付け先サブネットID
                - ALBSubnetId2：ALB紐付け先サブネットID
                - ALBVPCId：ALB紐付け先VPC ID
            - RDS関連
                - DBInstanceIdentifier：RDSインスタンス識別子
                - DBSecurityGroups：RDS用セキュリティグループ
                - DBSnapshotIdentifier：復元元のRDSスナップショット識別子
                - DBSubnetGroupName：RDS用サブネットグループ名
            - EC2関連
                - EC2ImageId：EC2インスタンス作成用AMI ID
                - EC2SecurityGroupId：EC2用セキュリティグループ
                - EC2SubnetId：EC2デプロイ先サブネットID
