# Ansible_Testing_environment

## 前提条件

test2という名前のキーペアを作成しておいてください。

* cli(CLIのセットアップが必要です)

    ```sh
    aws ec2 create-key-pair --key-name test2 --query 'KeyMaterial' --output text > ~/.ssh/test2.pem
    ```

* Management Console

    [EC2 キーペアの作成](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/WindowsGuide/ec2-key-pairs.html#prepare-key-pair)

## 構築手順

1. 東京リージョンのCloudFormationでansible_cloudformation.yamlを使用し、スタックを作成
2. ansible-serverにセッションマネージャで接続し、以下コマンドを実行

    ```sh
    sudo su

    #ansible実行用ユーザ
    useradd ansible

    passwd ansible

    su - ansible

    #ansibleのインストール
    pip3.8 install ansible --user

    #windowsに接続する際に必要
    pip3.8 install pywinrm --user

    #ansible Command not found となる場合
    exec $SHELL -l

    #インストール確認
    ansible --version

    #動作確認
    ansible localhost -m ping

    ```

3. target-server-Windowsにセッションマネージャで接続し、以下コマンドを実行

    ```powershell
    winrm set winrm/config/service/auth '@{Basic="true"}'

    winrm set winrm/config/service '@{AllowUnencrypted="true"}'
    ```

4. target-server-WindowsにRDPでログインし、下記参考をみながらボリュームの割り当てを行う。

    [Windows で Amazon EBS ボリュームを使用できるようにします。](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/WindowsGuide/ebs-using-volumes.html)
