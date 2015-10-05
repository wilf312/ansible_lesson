# ansible_lesson


ansibleの設定

## ドキュメント
https://docs.ansible.com/ansible/user_module.html


## epelの登録
https://dl.fedoraproject.org/pub/epel/6/x86_64/


wget -Uvh https://dl.fedoraproject.org/pub/epel/6/x86_64/

sudo rpm -Uvh epel-release-6-8.noarch.rpm


sudo yum -y install ansible


## （インベントリ）hostsファイルの準備


[web]
192〜
[db]
192〜
192〜


こうするとカテゴリに対してのみ実行することができる
ansible web -i hosts -m ping
ansible db -i hosts -m ping

全てのときは
ansible all -i hosts -m ping


インベントリファイルへのパスを省略するときは下記ファイルを調整

~/ansible.cfg
[defaults]
hostfile = ./hosts


## Playbookを作る

yamlで書く

```yaml:playbook.yml

---
- hosts: all
  sudo: yes
  tasks:
    - name: add a new user
      user: name=genya

```


実行
ansible-playbook playbook.yml



追加されたか確認
sshで入る
ssh web

ユーザー一覧の出力
cat /etc/passwd


## コマンドオプション
* yamlファイルのシンタックスチェック
ansible-playbook playbook.yml --syntax-check
* 実行されるタスクの一覧を出力
ansible-playbook playbook.yml --list-task
* dry run
ansible-playbook playbook.yml --check



## playbookで使える変数
varsで宣言

```yaml:vars.yaml

---
- hosts: all
  sudo: yes
  vars: # 変数宣言
    username: genya # Key Value形式で書く

# 実行時に入力させる場合は vars_prompt と書くとその後のkeyに入力文字が入る
#  vars_prompt:
#    username: "username?"
  tasks:
    - name: add a new user
      user: name={{username}} # 変数を利用するときは{{変数}} の書式で書く


```


## Apacheのインストール

- hosts: web # ターゲットの指定
  sudo: yes  # sudo権限の指定
  tasks:
    - name: install apache # description
      yum: name=httpd state=latest # yum の httpd 最新版をインストール
    - name: start apache and enabled # 起動、再起動時に起動するように設定
      service: name=httpd state=started enabled=yes # service start httpd と enabled追加

## ドキュメントルートの追加


```yaml:seika.yml

---
- hosts: all
  sudo: yes
  tasks:
    - name: add a new user
      user: name=taguchi
    - name: install libselinux-python
      yum: name=libselinux-python state=latest

- hosts: web
  sudo: yes
  tasks:
    - name: install apache
      yum: name=httpd state=latest
    - name: start apache and enabled
      service: name=httpd state=started enabled=yes
    - name: change Owner
      file: dest=/var/www/html owner=vagrant recurse=yes
    - name: copy index.html
      copy: src=./index.html dest=/var/www/html/index.html owner=vagrant
    - name: install php packages
      yum: name={{item}} state=latest
      with_items:
        - php
        - php-devel
        - php-mbstring
        - php-mysql
      notify:
        - restart apache
    - name: copy hello.php
      copy: src=./hello.php dest=/var/www/html/hello.php owner=vagrant
  handlers:
    - name: restart apache
      service: name=httpd state=restarted

- hosts: db
  sudo: yes
  tasks:
    - name: install mysql
      yum: name={{item}} state=latest
      with_items:
        - mysql-server
        - MySQL-python
    - name: start mysql and enabled
      service: name=mysqld state=started enabled=yes
    - name: create a database
      mysql_db: name=mydb state=present
    - name: create a user for mydb
      mysql_user: name=dbuser password=dbpassword priv=mydb.*:ALL state=present


```



接続テスト
ansible -i provisioning/hosts 192.168.200.10 -m ping -vvvv



2013年08月06日 (火) 著者 ： maki
はじめてAnsibleを使う人が知っておきたい7つのモジュール
http://www.infiniteloop.co.jp/blog/2013/08/ansible/

yum doc
http://docs.ansible.com/ansible/yum_module.html

yumまとめ
http://qiita.com/tenbrother/items/96422c7604ecfe990533


Ansible Tutorial
http://yteraoka.github.io/ansible-tutorial/#test-ansible


Ansible コトハジメ
http://qiita.com/seizans/items/54da2077ac8e2dcf5d6f