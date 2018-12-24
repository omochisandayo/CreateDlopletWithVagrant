# WindowsからVagrantでDegitalOcean上に仮想マシンを作る！

## 初めに
出先で仮想環境が欲しいときがあるが、
イメージをおとしてくるだけで時間がかかったりするのでDigitalOceanに環境が作れたら良いなと思い手順をまとめてみました。
完全に個人的メモ。。。


## 実施環境
Windows 10 64bit

## 手順

### SSH のために秘密鍵を生成する。
手順はこちらを参照。
https://webkaru.net/linux/tera-term-ssh-login-public-key/
→できた鍵を鍵①とする。

### DigitalOcean にログインしてAPIトークンを発行する

![](https://i.imgur.com/S0r7v8F.png)

→発行されたトークンをトークン①とする。
あとで再度見ることができないので作成時に保管しておくこと。

### Vagrant に DigitalOcean を操作するためのプラグインを導入
```
vagrant plugin install vagrant-digitalocean
```
### Vagrantの作業用ディレクトリを作業PC上に作る。
こんな具合。
![](https://i.imgur.com/TptxQG8.png)

### 初期化してVagrantfileを作る。
```
vagrant init
```

### Vagrantfile を書く。
参考：https://github.com/devopsgroup-io/vagrant-digitalocean/
```
Vagrant.configure('2') do |config|

  config.vm.define "droplet1" do |config|
      config.vm.provider :digital_ocean do |provider, override|
        override.ssh.private_key_path = '***書き換える1***'
        override.vm.box = 'digital_ocean'
        override.vm.box_url = "https://github.com/devopsgroup-io/vagrant-digitalocean/raw/master/box/digital_ocean.box"
        override.nfs.functional = false
        provider.token = '***書き換える2***'
        provider.image = 'ubuntu-14-04-x64'
        provider.region = 'nyc1'
        provider.size = '512mb'
      end
  end
end
```


***書き換えについて***

| 書き換え箇所 | 　　内容　 　 |
| -----------| ----------|
| 1つめ | 鍵①の格納先のパス |
| 2つめ  | トークン①の文字列 |

### Vagrant起動

```
vagrant up --provider=digital_ocean
```
おろろ。。。なんかこけた。。。↓
```
Bringing machine 'droplet1' up with 'digital_ocean' provider...
==> droplet1: Using existing SSH key: Vagrant
==> droplet1: Creating a new droplet...
==> droplet1: Assigned IP address: xxx.xxx.xxx.xxx
WARNING: Unexpected middleware set after the adapter. This won't be supported from Faraday 1.0.
==> droplet1: Destroying the droplet...
```
もう少しエラーを見てみてみる
```
C:/Users/xxxx/.vagrant.d/gems/2.4.4/gems/vagrant-digitalocean-0.9.3/lib/vagrant-digitalocean/actions/create.rb:63:in `b
lock in call': not ready (RuntimeError)
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.2/gems/vagrant-2.1.2/lib/vagrant/util/retryable.rb:17:in `retryable'

        from C:/Users/xxxx/.vagrant.d/gems/2.4.4/gems/vagrant-digitalocean-0.9.3/lib/vagrant-digitalocean/actions/creat
e.rb:61:in `call'
        from C:/HashiCorp/Vagrant/embedded/gems/2.1.2/gems/vagrant-2.1.2/lib/vagrant/action/warden.rb:34:in `call'
        from C:/Users/xxxx/.vagrant.d/gems/2.4.4/gems/vagrant-digitalocean-0.9.3/lib/vagrant-digitalocean/actions/setup
_key.rb:33:in `call'
```

```
C:/Users/xxxx/.vagrant.d/gems/2.4.4/gems/vagrant-digitalocean-0.9.3/lib/vagrant-digitalocean/actions/create.rb:63
```
とあるのでソースを見に行くとsshのリトライとタイムアウトの記述らしき行だった。
```
retryable(:tries => 120, :sleep => 10) do
  next if env[:interrupted]
  raise 'not ready' if !@machine.communicate.ready?
end
```
結論から言うと↑は関係がなく。。。
すでに存在しているSSH Keyを使用する処理がよろしくないとのこと。
https://github.com/devopsgroup-io/vagrant-digitalocean/issues/273
DigitalOceanのSetting>SecurityからVagrantという名前のSSH Keyを削除し、もう一度up!
```
PS C:\Users\xxxx\Vagrant\digitalocean> vagrant up --provider=digital_ocean
Bringing machine 'droplet1' up with 'digital_ocean' provider...
==> droplet1: Creating new SSH key: Vagrant...
==> droplet1: Creating a new droplet...
==> droplet1: Assigned IP address: xxx.xx.xx.xx
==> droplet1: Preparing SMB shared folders...
    droplet1: You will be asked for the username and password to use for the SMB
    droplet1: folders shortly. Please use the proper username/password of your
    droplet1: account.
    droplet1:
    droplet1: Username: xxxxx
    droplet1: Password (will be hidden):
==> droplet1: Mounting SMB shared folders...
We couldn't detect an IP address that was routable to this
machine from the guest machine! Please verify networking is properly
setup in the guest machine and that it is able to access this
host.

As another option, you can manually specify an IP for the machine
to mount from using the `smb_host` option to the synced folder.
```
できたーーー！
SSHでも入れたぞ。
```
PS C:\Users\xxxx\Vagrant\digitalocean> vagrant ssh
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-149-generic x86_64)
以下省略。
```


おわり。