# ハンズオン2: showコマンドの自動化を体験しよう

このハンズオンでは、作成済みのPlaybookを利用してshow コマンドの自動化を体験します。

Playbookを実行すると、showコマンドの実行結果がAnsibleサーバー上にファイルとして生成されます。

```

  [Ansible] ---(1)showコマンド実行---> [ios01]
    (2)
  結果ファイル生成
```

ハンズオンの流れは以下の通りです。

- 2-1. Playbookの確認
- 2-2. Playbookの実行
- 2-3. 実行結果の確認

# 2-1. Playbookの確認

Playbook（`handson2.yml`）は作成済みです。`cat` コマンドや `vi` などのエディタやで内容を確認してください。

Playbook（`handson2.yml`）は以下の通りです。

```yaml
---
- hosts: ios            # 対象のグループ名（インベントリファイル内の定義）
  gather_facts: false   # システム情報収集機能の無効化（時間短縮のため）

  # ここから変数定義
  vars:
    # 実行したいshowコマンドを定義（あとでタスクで利用）
    show_commands:
      - show running-config
      - show version
      - show interfaces
      - show ip route

  # ここからタスクの定義
  tasks:
    # タスク1: showコマンドの実行
    - name: exec show commands
      cisco.ios.ios_command:
        commands: "{{ show_commands }}"   # 実行したいshowコマンドを指定（変数として）
      register: result_show_commands      # 実行結果をresult_show_commandsという変数に格納

    # タスク2: 結果のファイル出力
    - name: save to file show commands
      ansible.builtin.copy:
        content: "{{ item }}"     # 出力したい内容（itemにループごとに各showコマンド実行結果が入る）
        dest: "outputs/{{ inventory_hostname }}_{{ show_commands[ansible_loop.index0] | replace(' ', '_') }}.txt"  # 出力先ファイル名
      loop: "{{ result_show_commands.stdout }}"   # showコマンド実行結果内のループ
      loop_control:
        extended: true
        label: "{{ show_commands[ansible_loop.index0] }}"
```

## 解説

少し複雑に見えるかもしれませんが、ポイント絞って解説します。

### 変数定義（ `vars:` 配下）

`show_commands` という独自の変数で、実行したいshowコマンドをリストで 4つ定義しています。
リストであるためコマンドを複数指定できます。

この変数は、あとでタスクで利用します。

### タスク1: showコマンドの実行

[`cisco.ios.ios_command` モジュール](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_command_module.html)を利用して、Cisco IOS 機器にshowコマンドを実行するタスクです。

[`commands` パラメーター](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_command_module.html#parameter-commands)で、実行したいshowコマンドをリストで指定します。ここでは、前述の変数 `show_commands` を指定します。

### タスク2: 結果のファイル出力

[`copy` モジュール](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)を利用して、showコマンド実行結果をファイルを書き出すタスクです。

モジュールの名前の通り、ファイルのコピーをするのが基本動作ですが、[`content` パラメーター](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#parameter-content)で内容を指定することで、任意の内容をファイルに出力できます。

ここでは、タスク1のshowコマンド実行結果の内容を変数経由で指定しています。

[`dest` パラメーター](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html#parameter-dest)で、出力先のファイル名を指定します。ファイル名は、 `ios01_showコマンド名.txt` という規則になるようにしています。

# 2-2. Playbookの実行

次に、Playbookを実行します。Playbookの実行は `ansible-playbook` コマンドを利用します。`-i` オプションでインベントリファイル名を指定し、そのあとにPlaybookファイル名を指定します。

以下のコマンドでPlaybookを実行してください。

【コマンド実行】
```bash
ansible-playbook -i inventory.ini handson2.yml 
```

以下は実行結果例です。

```bash
(ansible) [ansible@controller handson]$ ansible-playbook -i inventory.ini handson2.yml

PLAY [ios] ******************************************************************************

TASK [exec show commands] ***************************************************************
ok: [ios01]

TASK [save to file show commands] *******************************************************
changed: [ios01] => (item=show running-config)
changed: [ios01] => (item=show version)
changed: [ios01] => (item=show interfaces)
changed: [ios01] => (item=show ip route)

PLAY RECAP ******************************************************************************
ios01   : ok=2   changed=1   unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
```

`PLAY RECAP` の表示で、`ok=2`、`changed=1` になることを確認してください。

# 2-3. 実行結果の確認

Playbookの実行によって、showコマンド実行結果が `~/handson/outputs` ディレクトリ配下にファイルとして保存されます。ファイル名の規則は `ios01_showコマンド名.txt` です。

以下のコマンドで、ファイルの一覧を確認してください。

【コマンド実行】
```
ls -l outputs
```

以下 4ファイルがあることを確認してください。
```bash
# 実行結果例
-rw-rw-r--. 1 ansible ansible 6296 Dec  1 08:25 ios01_show_interfaces.txt
-rw-rw-r--. 1 ansible ansible 1336 Dec  1 08:24 ios01_show_ip_route.txt
-rw-rw-r--. 1 ansible ansible 7540 Dec  1 08:19 ios01_show_running-config.txt
-rw-rw-r--. 1 ansible ansible 2348 Dec  1 08:25 ios01_show_version.txt
```

表示された結果ファイルのうち、任意の `ios01_showコマンド名.txt` の内容を `cat` コマンドなどで確認してください。ここでは例として、`ios01_show_running-config.txt` を確認する例を掲載します。

【コマンド実行】
```
$ cat ./outputs/ios01_show_running-config.txt
Current configuration : 7480 bytes
!
! Last configuration change at 15:07:14 JST Wed Dec 1 2021
!
version 17.1
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
! Call-home is enabled by Smart-Licensing.
service call-home
platform qfp utilization monitor load 80
platform punt-keepalive disable-kernel-core
platform console virtual
!
hostname ios01
...(略)...
```

無事にAnsibleでshowコマンドの自動化を体験できました。なお、変数 `show_commands` で定義した show コマンドをカスタマイズすることで、好きなshowコマンドを自動化できます。


これで、ハンズオン2「showコマンドの自動化を体験しよう」は終了です。

---

# 👉 NEXT

以下の次のハンズオンに進んでください。

👉 [ハンズオン3: コンフィグ投入を自動化しよう](./handson3.md)

---

🏠 [`README.md` に戻る](../README.md)
