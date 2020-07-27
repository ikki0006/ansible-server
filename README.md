# Ansible-server
ansibleをとりあえず試してサーバのセッティングをしてみたい方向け。
モックなのでpassword直書きの部分があります。  
本番は環境変数やansibleのハッシュで隠蔽することを推奨します。 
コンテナを利用しての利用も記載していますが動作確認は通常のVM(ubuntu 18.4)で実施しています。
コンテナはおまけとお考えください。

## 使い方
ローカルにansibleが必要。動作確認はansible 2.9.7で実施しています。
インストール方法は環境によりますので各自ググってください。
(macはbrew install ansibleで入ります。)

### host設定
ローカルのssh_configを使用しない場合は./ssh/ssh_configを編集  
コンテナ用のローカルホスト(lcap01)とVM用のホスト(lcts01)のみ入っているので  
使用環境に合わせて、編集してください。  

### group設定(その1)
groupでプレイブックを分けているので上記のhostで設定したVMを./local, ./develop, ./production  
に適宜所属させてください。  
試験だけであればlocalのみで十分です。
staging環境などがある方は適宜変更してください。

### group設定(その2)
上記で設定したgroupファイル内に更に細かくサブグループを切ることができます。  
アプリサーバ(apservers)にはpython流したいけどwebサーバ(rpservers)には流したくないなどplaybook単位で  
roleをより細かく区切ることができます

### Role設定
./role配下に実際の設定を格納します。
今回は以前プライベートで作成したazure環境で
使用したものを入れています。  
こちらは参考までになので実際は好みにカスタマイズしてください。  

### playbook設定
group設定(その2)で用意したサブグループ単位、もしくは全体に流す単位でplaybookを切ります。  
playbook側のhostsでサブグループを指定することで流す範囲を指定できます。  
ここまで編集が終われば  

    ansible-playbook -i {{group名}} site.yml -K -C 

で実際に起動して設定を流せます。 
個別にplaybookを流すことも可能なので  
例えばローカルにdefault_set.ymlを流したい場合は  

    ansible-playbook -i local default_set.yml -K -C

となります。  
とりあえずはvagrantやdockerで試してみながら調整するのをおすすめします。


## コンテナ起動

### 初期インストール方法
dockerが入っていること前提

#### テスト用ubuntuの起動

    cd 本プロジェクトのpath/docker
    docker build -t testvm:0.0.3 .
    docker run -d -p 10000:22 -p 80:80 --rm --privileged --name ubuntu testvm:0.0.3

#### mac用
    docker run -d -p 10000:22 -p 80:80 --rm --security-opt seccomp:unconfined --cap-add SYS_ADMIN  -v /sys/fs/ cgroup:/sys/fs/cgroup:ro --name ubuntu testvm:0.0.3

    docker ps

↓のように表示されればOK 

    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                   NAMES

    3f1d6cb1ebdb        testvm:0.0.1        "/usr/sbin/sshd -D"   39 seconds ago      Up 38 seconds       0.0.0.0:10000->22/tcp   ubuntu

linuxコンテナのRLIMIT_COREにバグがあるらしいので対策(実施しないとsudo不可)

    docker exec -it ubuntu /bin/bash
    echo 'Set disable_coredump false' >> /etc/sudo.conf
    exit
    ssh -p 10000 admin@localhost 
    password:tmppass

後は煮るなり焼くなり。。。

### テスト用ubuntuの停止
停止するとデータが消えるので初期設定しなおしたい場合のみ使用

    docker stop ubuntu
    docker rm ubuntu


