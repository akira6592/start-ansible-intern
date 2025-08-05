# ハンズオン4: IPアドレス設定を自動化しよう

先ほどのハンズオン3では、コンフィグを直接そのままPlaybook内に指定していました。Ansibleにはコンフィグをそのまま指定するモジュールだけなく、もっと高機能なモジュールもあります。

このハンズオンでは、L3レベルのインターフェース設定をする専用モジュールを利用して、IPアドレス設定を自動化します。

ハンズオンの流れは以下の通りです。

- 4-1. Playbookの作成
- 4-2. 事前確認
- 4-3. Playbookの実行
- 4-4. 実行結果の確認

# 4-1. Playbookの作成

Playbook（`handson4.yml`）は作成済みです。`cat` コマンドや `vi` などのエディタで内容を確認してください。


```yaml
---
- hosts: ios            # 対象のグループ名（インベントリファイル内の定義）
  gather_facts: false   # システム情報収集機能の無効化（時間短縮のため）

  # ここからタスクの定義
  tasks:
    # タスク1: インターフェースへIPアドレスの設定
    - name: config IP address
      cisco.ios.ios_l3_interfaces:
        config:
          - name: GigabitEthernet3
            ipv4:
              - address: 10.1.3.254/24
```

## 解説

Playbook内のタスクについて解説します。

### タスク1: インターフェースへIPアドレスの設定

[`cisco.ios.ios_l3_interfaces` モジュール](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_l3_interfaces_module.html)を利用して、Cisco IOS 機器のインターフェースにIPアドレスを設定するタスクです。

[`config` パラメーター](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_l3_interfaces_module.html#parameter-config)で、設定内容をリストで指定します。

[`name` パラメーター](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_l3_interfaces_module.html#parameter-config/name)で、インターフェース名を、`ipv4` 配下の [`address` パラメーター](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_l3_interfaces_module.html#parameter-config/ipv4/address)で、IPv4 アドレスを指定します。


# 4-2. 事前確認

Playbook を実行する前に、今回ルーターに設定する IP アドレスに対して、Ansible サーバーから ping が通らないことを確認します。ping とは、ICMP というプロトコルを利用した、疎通確認をするためのコマンドです。


【コマンド実行】
```bash
ping 10.1.3.254 -c 4
```

結果は、以下のように「Time to live exceeded」になって ping が失敗します。

```
(ansible) [ansible@controller handson]$  ping 10.1.3.254 -c 4
PING 10.1.3.254 (10.1.3.254) 56(84) bytes of data.
From 10.1.3.254 icmp_seq=1 Time to live exceeded
From 10.1.3.254 icmp_seq=2 Time to live exceeded
From 10.1.3.254 icmp_seq=3 Time to live exceeded
From 10.1.3.254 icmp_seq=4 Time to live exceeded

--- 10.1.3.254 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3005ms

(ansible) [ansible@controller handson]$
```

また、以下のようにタイムアウトになることもあります。

```
(ansible) [ansible@controller handson]$ ping 10.1.3.254 -c 4
PING 10.1.3.254 (10.1.3.254) 56(84) bytes of data.

--- 10.1.3.254 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3068ms

(ansible) [ansible@controller handson]$
```

いずれの場合はも「4 packets transmitted」に対して「0 received」なので、一度も成功していないことを示しています。

# 4-3. Playbookの実行

次に、Playbookを実行します。以下のコマンドでPlaybookを実行してください。

【コマンド実行】
```bash
ansible-playbook -i inventory.ini handson4.yml
```

以下は実行結果例です。

```bash
(ansible) [ansible@controller handson]$ ansible-playbook -i inventory.ini handson4.yml

PLAY [ios] *********************************************************************

TASK [config IP address] *******************************************************
changed: [ios01]

PLAY RECAP *********************************************************************
ios01 : ok=1  changed=1 unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
```

`PLAY RECAP` の表示で、`ok=1`、`changed=1` になることを確認してください。


# 4-4. 実行結果の確認

Playbookの実行によって、ルーターに以下のコンフィグが投入されたはずです。

```
interface GigabitEthernet3
 ip address 10.1.3.254 255.255.255.0
```

Playbook内では上記のようなコンフィグそのものは指定していませんが、[`cisco.ios.ios_l3_interfaces` モジュール](https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_l3_interfaces_module.html)に指定した各パラーメーターの値によって、内部的にコンフィグが生成され、機器に投入される仕組みです。

このコンフィグが入ったか確認しましょう。

このコンフィグが入ったかを Ansible で確認しましょう。

以下のコマンドを実行して、Ansible経由でコンフィグを表示してください。

【コマンド実行】
```
ansible -i inventory.ini ios -m ios_command -a "commands='show running-config interface GigabitEthernet3'"
```

実行結果例は以下の通りです。`stdout_lines` 配下に目的のコンフィグがあることを確認してください。

```
(ansible) [ansible@controller handson]$ ansible -i inventory.ini ios -m ios_command -a "commands='show running-config interface GigabitEthernet3'"
ios01 | SUCCESS => {
    ...(略)...
    "stdout_lines": [
        [
            "Building configuration...",
            "",
            "Current configuration : 238 bytes",
            "!",
            "interface GigabitEthernet3",               # 注目
            " ip address 10.1.3.254 255.255.255.0",     # 注目
            " ip ospf network non-broadcast",
            " ip ospf dead-interval 40",
            " ip ospf hello-interval 10",
            " ip ospf 1 area 0",
            " ip ospf cost 50",
            " negotiation auto",
            " no mop enabled",
            " no mop sysid",
            "end"
        ]
    ]
}
```

そして、事前確認では失敗した ping を再度実行します。


【コマンド実行】
```bash
ping 10.1.3.254 -c 4
```

以下のように、「64 bytes from 10.1.3.254: icmp_seq=N ttl=255 time=N.NN ms」が 4行表示されれば、疎通確認成功です。（タイムアウトや「Time to live exceeded」が表示されないはずです）

```
(ansible) [ansible@controller handson]$ ping 10.1.3.254 -c 4
PING 10.1.3.254 (10.1.3.254) 56(84) bytes of data.
64 bytes from 10.1.3.254: icmp_seq=1 ttl=255 time=2.27 ms
64 bytes from 10.1.3.254: icmp_seq=2 ttl=255 time=0.502 ms
64 bytes from 10.1.3.254: icmp_seq=3 ttl=255 time=1.62 ms
64 bytes from 10.1.3.254: icmp_seq=4 ttl=255 time=1.48 ms

--- 10.1.3.254 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3036ms
rtt min/avg/max/mdev = 0.502/1.469/2.271/0.633 ms
(ansible) [ansible@controller handson]$ 
```

無事にAnsibleで、インターフェースへのIPアドレス設定の自動化を実現できました。

これで、ハンズオン4「IPアドレス設定を自動化しよう」は完了です。

---

# 👉 NEXT（余裕がある人向け）

もしハンズオン時間に余裕がある人は、以下の次のハンズオンに進んでください。

👉 [ハンズオン5: インターフェースの基本設定を自動化しよう](./handson5.md)

---

🏠 [`README.md` に戻る](../README.md)


