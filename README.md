# hashicorp-valut-handson



## Handson

* [getting-started](https://learn.hashicorp.com/tutorials/vault/getting-started-install)


### Step1
[Dev Server Mode | Vault by HashiCorp](https://www.vaultproject.io/docs/concepts/dev-server/)

[KV - Secrets Engines | Vault by HashiCorp](https://www.vaultproject.io/docs/secrets/kv/kv-v2/)

### Step2

[HTTP API | Vault by HashiCorp](https://www.vaultproject.io/api-docs/index/)

```
curl http://localhost:8200/v1/sys/health | jq

```

### Step3

このチャレンジでは、Vault サーバを"本番" モードで実行し、初期化、Unseal を実行します。サーバは、起動時に指定した構成ファイルより構成を取得します。

Vault サーバの起動、初期化、Unseal に関しては以下のドキュメントを参照下さい。

https://www.vaultproject.io/docs/configuration/

https://www.vaultproject.io/docs/commands/operator/init/

https://www.vaultproject.io/docs/concepts/seal/

```vault-config.hcl
listener “tcp” {
 address = “0.0.0.0:8200”
 tls_disable = 1
}
storage “file” {
  path = “/vault/file”
}
disable_mlock = true
api_addr = “http://localhost:8200”
ui=true
```

```
vault server -config=/vault/config/vault-config.hcl
```

* Sealed 状態の確認

```
vault status
```

* Vault サーバを初期化し、1つのUnseal キーを設定

```
vault operator init -key-shares=1 -key-threshold=1
```

* Unseal キーと初期ルートトークンが返ってきます。Unseal キー、及びルートトークンはテキストエディタ等にコピーしてください(後で使用します)
主要なVault コマンドを使用可能にするため、vault operator init コマンドが返した初期ルートトークンを使用します。環境変数VAULT_TOKEN を設定してください。

```
export VAULT_TOKEN=<root_token>

```

* 次に、vault operator init コマンドが返したUnseal キーを提供して、Unseal 処理を行うことでVault サーバを利用可能にします。

```
vault operator unseal
```

* Initialized がtrue、Sealed が`false`` となっていれば、Vault は利用可能となっています。
Vault サーバの状態を確認するには、vault status コマンドを実行します。Sealed がtrue である場合、再度vault operator unseal コマンドを再実行し、正しいUnseal キーを指定するようにしてください。
最後に、ルートトークンでVault UI にログインします。問題がある場合は、上記のコマンドをすべて実行したことを再確認してください。
ルートトークンとUnsealキーを保存することを忘れずに! 後で使用します

```txt
unseal key: WPB3Aaz+lCfdtTlz2Mz5RoiFykI5FlL+kfJNNDKQc14=
```

### Step4

このチャレンジでは、前回のチャレンジで開始したVault 本番サーバに Vault の KV v2 シークレットエンジンをマウントします。
また、シークレットを書き込み、Vault UI でその値を変更し、KV v2 シークレットエンジンが複数のバージョンのシークレットを保持できることを確認します。

#### Use the KV V2 Secrets Engine

ここではVault 本番サーバが（バックグラウンドで）稼働しています。シークレットエンジンをマウントして、そこにシークレットを書き込んでみましょう。
前のチャレンジでvault operator init コマンドが返した初期ルートトークンを使用して、VAULT_TOKEN 環境変数を再度エクスポートする必要があります。
VAULT_TOKEN: s.Wlhte9exGkMU7nn5zw9Oacfn
```

```

本番用のVault では手動でシークレットエンジンをマウントします。KV v2 シークレットエンジンのインスタンスをデフォルトパスのkv にマウントしましょう。

```
vault secrets enable -version=2 kv
```

```
 vault secrets enable -version=2 kv
Success! Enabled the kv secrets engine at: kv/
/ # vault kv put kv/a-secret value=1234
Key                Value
---                -----
created_time       2022-03-23T06:02:19.987373465Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1
```

### Step5

KV v2 シークレットエンジンがマウントされた本番用のVault サーバの稼働が確認できました。次に、ユーザーを認証する方法を学びましょう。
このチャレンジでは、Vault が管理するユーザー名とパスワードでVault への認証を可能にするUserpass 認証メソッドをマウントして利用します。
また、ユーザーやアプリケーションがアクセスできるシークレットを制限するVault ポリシーについても学びます。Vault ポリシーは “デフォルトで拒否（deny by default)” であることに注意してください。これは、トークンに添付されているポリシーのいずれかによって明示的に許可された場合にのみ、トークンはシークレットを読み書きできます。

### Use the Userpass Auth Method

これで、KV v2 シークレットエンジンがマウントされた本番用のVault サーバが稼働しているので、ユーザーを認証する方法を学びましょう。
まず、VAULT_TOKEN 環境変数を再度エクスポートする必要があります。

以下のコマンドで userpass 認証メソッドを有効化してください。

```
vault auth enable userpass
```

次に、あなた自身をVault ユーザーとして追加します。ポリシーはここでは指定しません。

```
/ # vault auth enable userpass
Success! Enabled userpass auth method at: userpass/
/ # vault write auth/userpass/users/friendbear password=password
Success! Data written to: auth/userpass/users/friendbear
```


これで、Vault UI にuserpass 認証メソッドを使ってログインできるようになりました。
Vault CLI でログインすることもできます

```
vault login -method=userpass username=friendbear password=password
```

```
 # vault login -method=userpass username=friendbear password=password
WARNING! The VAULT_TOKEN environment variable is set! This takes precedence
over the value set by this command. To use the value set by this command,
unset the VAULT_TOKEN environment variable or set it to the token displayed
below.

Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                    Value
---                    -----
token                  s.FejfskhWJjmasrD4TcmiZmSu
token_accessor         vr20dVN0B5ZvKVqMg1hOq25R
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    friendbear
```

上記の手順により取得したVault トークンにはデフォルトのポリシーが付与されていますが、アクセス可能な範囲は限定的です。なお、黄色の警告メッセージは、現在VAULT_TOKEN 環境変数が設定されており、これを解除するか、新しいトークンに設定する必要があることを示しています。設定を解除してみましょう
unset VAULT_TOKEN
新しいトークンが使用されていることを確認するには、以下のコマンドを実行します。

```
vault token lookup
```

```
/ # unset VAULT_TOKEN
/ # vault token lookup
Key                 Value
---                 -----
accessor            vr20dVN0B5ZvKVqMg1hOq25R
creation_time       1648015661
creation_ttl        768h
display_name        userpass-friendbear
entity_id           2219436e-8205-0d7e-1275-f5c9cb55846b
expire_time         2022-04-24T06:07:41.441457218Z
explicit_max_ttl    0s
id                  s.FejfskhWJjmasrD4TcmiZmSu
issue_time          2022-03-23T06:07:41.44146439Z
meta                map[username:friendbear]
num_uses            0
orphan              true
path                auth/userpass/login/friendbear
policies            [default]
renewable           true
ttl                 767h58m59s
type                service
```

現在のトークンの表示名は userpass-<name>- で、<name> はあなたのユーザー名で、トークンに付与されているポリシーはdefault ポリシーのみであることがわかります。

前回のチャレンジでKV v2 のシークレットエンジンに書いたシークレットを取得してみてください。

```
vault kv get kv/a-secret
```

エラーメッセージが表示されるのは、発行されたトークンにはシークレットの読み取りが許可されていないからです。
```
/ # vault kv get kv/a-secret
Error making API request.

URL: GET http://localhost:8200/v1/sys/internal/ui/mounts/kv/a-secret
Code: 403. Errors:

* preflight capability check returned 403, please ensure client's policies grant access to path "kv/a-secret/"
```


 Vault は明示的にポリシーに指定がない場合、”deny by default” として動作します。つまり、トークンは、ポリシーのいずれかによって明示的にシークレットの読み取りの権限が与えられている場合にのみ、取得可能になるということです。

### Step6
この課題では、Vault ポリシーを定義して、2人の異なるユーザーにVault 内の異なるシークレットへのアクセス権を与えることができます。

Vault ポリシーに関しては、以下を参照下さい。

https://www.vaultproject.io/docs/concepts/policies/



### Use Vault Policies
Userpass 認証メソッドが構成されましたが、次にユーザーにシークレットへのアクセスを与えるためのポリシーの追加を行います。
まず、コマンドを入力するか、履歴をスクロールしてVAULT_TOKEN 環境変数を再度エクスポートする必要があります。
```
export VAULT_TOKEN=<root_token>
```
Userpass 認証メソッドで使用するユーザー名はすでに作成していますが、先ほどと同じコマンドを使って、異なるユーザー名とパスワードを選択して 2 人目のユーザを作成してください。
```
vault write auth/userpass/users/bearsworld password=password
```
次に、「Vault Policies」タブに移動し、2つのポリシー、user-1-policy.hcl とuser-2-policy.hcl を編集します。

* User1
```hcl
path "kv/data/friendbear/*" {
  capabilities = ["create", "update", "read", "delete"]
}
path "kv/delete/friendbear/*" {
  capabilities = ["update"]
}
path "kv/metadata/friendbear/*" {
  capabilities = ["list", "read", "delete"]
}
path "kv/destroy/friendbear/*" {
  capabilities = ["update"]
}

# Additional access for UI
path "kv/metadata" {
  capabilities = ["list"]
}

```

user-1-policy.hcl では、<user> を最初に作成したユーザー名に設定します。user-2-policy.hcl では、<user> に追加した2人目のユーザー名を設定します。
変更後、ファイルを保存してください。
* User2
```hcl
path "kv/data/bearsworld/*" {
  capabilities = ["create", "update", "read", "delete"]
}
path "kv/delete/bearsworld/*" {
  capabilities = ["update"]
}
path "kv/metadata/bearsworld/*" {
  capabilities = ["list", "read", "delete"]
}
path "kv/destroy/bearsworld/*" {
  capabilities = ["update"]
}

# Additional access for UI
path "kv/metadata" {
  capabilities = ["list"]
}
```

次に、Vault サーバにポリシーを追加していきます。

```
# vault policy write friendbear /vault/policies/user-1-policy.hcl
Success! Uploaded policy: friendbear
```


```
# vault policy write bearsworld /vault/policies/user-2-policy.hcl
Success! Uploaded policy: bearsworld
```
<user_1>、<user_2> には各自で作成したユーザー名を指定してください。
続けて、作成したポリシーを各ユーザーに割り当てます。
vault write auth/userpass/users/<user_1>/policies policies=<user_1>
vault write auth/userpass/users/<user_2>/policies policies=<user_2>
<user_1>、<user_2> には、再び各自で作成したユーザ名を指定してください。
では、2人の異なるユーザーとしてVault UI にログインするとどうなるかを見てみましょう。
まず、最初に作成したユーザーでログインします。ログイン方法を”Userpass” に指定することを忘れないでください。
kv のシークレットエンジンをクリックします。先ほど作成したシークレットがa-secret という名前で表示されているはずです。 しかし、それを選択すると”Not Authorized” というメッセージが表示されます。
kv をクリックして、前の画面に戻ります。
“Create secret+” ボタンをクリックして、パスに<user>/age、画面の “Secret data” にキーの “age”、そのキーに関連付けられた値にあなたの “age” を入力します。<user> は、ログインしたユーザーのユーザー名に置き換えてください。そして、”Save” ボタンをクリックします。これらの操作は許可されているはずです。
ログアウトして、2番目のユーザーとしてログインし直します。最初のユーザーのシークレットにアクセスしてみてください。 できないはずです。
最初のユーザーと同様に、上記の手順を繰り返して、パス内の<user> に2番目のユーザー名を指定して、シークレットを作成してください。 こちらの操作も実行可能なはずです。
Vault CLI を使用したときに何が起こるかを見たい場合は、unset VAULT_TOKEN を実行し、vault login -method=userpass username=<user> password=<pwd> で各ユーザーとしてログインし、以下のようなコマンドを試してみてください。
vault kv get kv/<user>/age
vault kv put kv/<user>/weight weight=150

```
/ # vault kv get kv/friendbear/age
No value found at kv/data/friendbear/age
/ # vault kv get kv/friendbear/age
No value found at kv/data/friendbear/age
/ # vault kv get kv/friendbear/age
No value found at kv/data/friendbear/age
/ # vault kv put kv/friendbear/weight weight=70
Key                Value
---                -----
created_time       2022-03-23T06:28:58.924399872Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1
```
<user> がログインしたユーザーのユーザー名と一致した場合にのみ成功し、一致しない場合にはエラーとなることが確認できます。
このチャレンジでは、Vault が他のユーザーから各ユーザーのシークレットを保護できるようポリシーにより厳密なアクセスコントロールを行うことができることを確認しました。
Congratulations on finishing the entire track!


---

```
 history
   0 vault version
   1 vault
   2 vault secrets -h
   3 vault read -h
   4 vault server -h
   5 vault server -dev -dev-listen-address=0.0.0.0:8200 -dev-root-token-id=root
   6 vault kv put secret/my-first-secret age=20
   7 curl http://localhost:8200/v1/sys/health | jq
   8 curl --header "X-Vault-Token: root" http://localhost:8200/v1/secret/data/my-first-secret | jq
   9 vault server -config=/vault/config/vault-config.hcl
  10 vault status
  11 vault operator init -key-shares=1 -key-threshold=1
  12 export VAULT_TOKEN=s.Wlhte9exGkMU7nn5zw9Oacfn
  13 vault operator init -key-shares=1 -key-threshold=1
  14 export VAULT_TOKEN=s.Wlhte9exGkMU7nn5zw9Oacfn
  15 vault operator unseal
  16 history
```