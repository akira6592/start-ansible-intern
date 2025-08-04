# ハンズオン1: 準備と確認

このハンズオンでは、Ansible インストール済みサーバーに接続し、環境の確認をします。

# 1-1. 接続

TeraTerm などのターミナルソフトや ssh コマンドを利用して Ansible サーバー（Linux）に SSH 接続してください。

接続情報に必要な、IP アドレス、ユーザー名、パスワードは当日講師から別途案内をします。

# 1-2. Ansible の有効化

ハンズオン環境の Ansible は、特定の venv（Python の仮想環境）内にインストールしてあります。

その venv を有効化し、Ansible を利用できる状態にします。venv 有効化するために以下のコマンドを実行してください。

【コマンド実行】
```bash
source /opt/ansible/bin/activate
```

実行後、プロンプトの先頭に `(ansible)` が付くことを確認します。

```bash
# コマンド実行結果例
[ansible@controller ~]$ source /opt/ansible/bin/activate
(ansible) [centos@controller ~]$ 
```

# 1-3. ディレクトリの移動とインベントリの確認

ハンズオンに必要なファイルは `~/handson` ディレクトリ配下にあります。

まず、以下のコマンドで `~/handson` ディレクトリ移動してください。

【コマンド実行】
```
cd ~/handson
```

以降、ハンズオン手順では **`~/handson` ディレクトリにいることを前提** としています。

今回の環境では、インベントリファイル（自動化対象のホストのリスト）と、接続情報を定義したファイルは作成済みです。これらをのちのハンズオンで利用します。

それぞれ中身を確認します。

```bash
# （参考）ファイル一覧抜粋
~/handson/
  |- group_vars/
  |   |- ios.yml
  |- inventory.ini
```

## インベントリファイルの確認

以下のコマンドでインベントリファイルの中身を表示してください。

【コマンド実行】
```bash
cat inventory.ini
```

以下のように表示されることを確認してください。

```ini
[ios]
ios01 ansible_host=10.1.1.254 
```

### 解説
このインベントリファイルで、以下のことが読み取れます。

- ホスト `ios01` が定義してある
- ホスト `ios01` は変数 `ansible_host` の値として `10.1.1.254` が定義されている
  - そのため、Ansible が `ios01` を対象とする際に `10.1.1.254` に接続する
- グループ `ios` に、ホスト `ios01` が所属している


## 接続情報を定義した変数ファイルの確認

以下のコマンドで、接続情報を定義した変数ファイルの中身を表示してください。

【コマンド実行】
```bash
cat group_vars/ios.yml
```

### 解説

`ansible_network_os` 変数で、どのような値を指定するかは、公式ドキュメントの「[Settings by Platform](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.html#settings-by-platform)」にまとめられています。

`ansible_connection` 変数には、使用するコネクションプラグイン（接続を担うAnsibleのパーツ）を指定します。Cisco IOS に対しては、基本的に上記のように `network_cli` を指定します。プラットフォームとコネクションプラグインの関係は「[Settings by Platform](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.html#settings-by-platform)」にまとめられています。

SSH接続ユーザー（今回は `ansible`）が、特権を有している場合や、特権が不要な操作しかしない場合は、`ansible_become` から始まる変数の定義は不要です。

なおYAMLにおける `#` から始まる文字列はコメントです。

以上で「ハンズオン1: 準備と確認」は完了です。

---

# 👉 NEXT

以下の次のハンズオンに進んでください。

👉 [ハンズオン2: showコマンドの自動化を体験しよう](./handson2.md)
