# Ansible_Testing_environment

## 前提条件

何かしらキーペアを作成しておいてください。

* cli(CLIのセットアップが必要です)

    ```sh
    aws ec2 create-key-pair --key-name test --query 'KeyMaterial' --output text > ~/.ssh/test.pem
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

4. target-server-WindowsにRDPでログインし、下記参考をみながらボリュームの割り当てを行う。（Option）

    [Windows で Amazon EBS ボリュームを使用できるようにします。](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/WindowsGuide/ebs-using-volumes.html)

5. ansibleのログ出力設定及びlogrotate設定（Option）

    * ansible設定ファイルの用意

    ```bash
    sudo su

    vi /etc/ansible/ansible.cfg
    ```

    以下内容を記入

    ```cfg
    [defaults]
    log_path=/var/log/ansible.log
    private_key_file=no
    ```

    * ansible.logの用意

    ```bash
    sudo su

    touch /var/log/ansible.log

    chmod 666
    ```

    * logrotateの設定

    ```bash
    vi /etc/logrotate.d/ansible
    ```

    以下内容を記入

    ```ini
    /var/log/ansible.log {
        daily
        rotate 5
        missingok
        notifempty
        copytruncate
        dateext
        dateformat .%Y-%m-%d
    }
    ```

    * 設定反映

    ```bash
    sudo su

    sudo /usr/sbin/logrotate /etc/logrotate.conf
    ```

6. CloudWatch Agentの設定（Option）

## メモ

### collectdインストール

```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

sudo yum install -y collectd
```

### agentチェック

```bash
ps aux | grep amazon-cloudwatch-agent

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

### CloudFormationヘルパースクリプトインストール

```bash
curl -O https://s3.amazonaws.com/cloudformation-examples/aws-cfn-bootstrap-py3-latest.tar.gz
python3.8 /usr/lib/python3.8/site-packages/easy_install.py --script-dir /opt/aws/bin aws-cfn-bootstrap-py3-latest.tar.gz
```

### UserDataログ確認

```bash
cat /var/log/cloud-init-output.log
```
