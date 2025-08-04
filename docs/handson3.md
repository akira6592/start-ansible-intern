# ハンズオン3: コンフィグ投入を自動化しよう

このハンズオンでは、ネットワーク機器への設定コンフィグの投入を自動化します。ここでは、参照先NTPサーバーの指定（`ntp server` コマンド）をします。

ハンズオンの流れは以下の通りです。

- 3-1. Playbookの作成
- 3-2. Playbookの実行
- 3-3. 実行結果の確認

# 3-1. Playbookの作成

Playbook（`handson3.yml`）は作成済みです。`cat` コマンドや `vi` などのエディタやで内容を確認してください。

```yaml
---
- hosts: ios            # 対象のグループ名（インベントリファイル内の定義）
  gather_facts: false   # システム情報収集機能の無効化（時間短縮のため）

  # ここから変数定義
  vars:
    # 実行したいshowコマンドを定義（あとでタスクで利用）
    ntp_servers:
      - 10.0.0.11
      - 10.0.0.12

  # ここからタスクの定義
  tasks:
    # タスク1: 参照NTPサーバーの設定
    - name: config ntp
      cisco.ios.ios_config:
        src: ntp.j2   # コンフィグ生成テンプレートを指定
```

## 解説

Playbook内のタスクについて解説します。

### タスク1: 参照NTPサーバーの設定

[`cisco.ios.ios_config` モジュール](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_config_module.html)を利用して、Cisco IOS 機器に設定コマンドを投入するタスクです。

[`src` パラメーター](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_config_module.html#parameter-src)で、コンフィグを生成するためのテンプレートファイル名を指定します。

`src` パラメーターで指定したテンプレートファイル [handson/templates/ntp.j2](../templates/ntp.j2) は以下の内容です。

```
{% for server in ntp_servers %}
ntp server {{ server }}
{% endfor %}
```

テンプレートは Jinja2 というテンプレートエンジンの仕様に基づく形式です。変数 `ntp_servers` のリストの要素数分、`ntp server xxx` コマンドが生成されます。プログラミング経験者には分かりやすい構文になっているのではないでしょうか。

# 3-2. Playbookの実行

次に、Playbookを実行します。以下のコマンドでPlaybookを実行してください。

【コマンド実行】
```bash
ansible-playbook -i inventory.ini handson3.yml
```

以下は実行結果例です。

```bash
(ansible) [ansible@controller handson]$ ansible-playbook -i inventory.ini handson3.yml

PLAY [ios] *********************************************************************

TASK [config ntp] **************************************************************
[WARNING]: To ensure idempotency and correct diff the input configuration lines should be similar to how they appear if present in the running configuration on device
changed: [ios01]

PLAY RECAP *********************************************************************
ios01 : ok=1  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
```

`PLAY RECAP` の表示で、`ok=1`、`changed=1` になることを確認してください。（`[WARNING]` はモジュールの仕様によるものですので無視してください）

# 3-3. 実行結果の確認

Playbookの実行によって、ルーターに以下のコンフィグが投入されたはずです。

```
ntp server 10.0.0.11
ntp server 10.0.0.12
```

はじめての設定自動化ですので、本当に設定されたか確認してみましょう。

ルーターにログインするのもよいのですが、せっかくですので確認自体も Ansible でやってみましょう。

以下のコマンドを実行して、Ansible経由でコンフィグを表示してください。

【コマンド実行】
```
ansible -i inventory.ini ios -m ios_command -a "commands='show running-config | include ntp'"
```

実行結果例は以下の通りです。`stdout_lines` 配下に目的のコンフィグがあることを確認してください。

```
(ansible) [ansible@controller handson]$ ansible -i inventory.ini ios -m ios_command -a "commands='show running-config | include ntp'"
ios01 | SUCCESS => {
    "changed": false,
    "stdout": [
        "ntp server ntp.nict.jp\nntp server 10.0.0.11\nntp server 10.0.0.12"
    ],
    "stdout_lines": [
        [
            "ntp server ntp.nict.jp",
            "ntp server 10.0.0.11",     # 注目
            "ntp server 10.0.0.12"      # 注目
        ]
    ]
}
```
このように、コンフィグ表示ような単一の簡単な処理であれば、Playbookを作成せずに、[`ansible` アドホックコマンド](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)のみで実現できます。


無事にAnsibleでコンフィグ投入の自動化を実現できました。

これで、ハンズオン3「コンフィグ投入を自動化しよう」は完了です。お疲れさまでした。

---

# 👉 NEXT（余裕がある人向け）

もしハンズオン時間に余裕がある人は、以下の次のハンズオンに進んでください。

以下の次のハンズオンに進んでください。

👉 [ハンズオン4: IPアドレス設定を自動化しよう](./handson4.md)

---

🏠 [`README.md` に戻る](../README.md)
