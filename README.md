# VMUG20220727

`Japan VMUG vExpert が語る #17 Registration` でデモをした資材です。

https://my.vmug.com/s/community-event?id=a1Y4x0000002H8cEAE

## デモ環境

* VCSA 7.0.0.10400

## 前提条件

ここでは、コマンドラインでの使い方について説明します。

* Linux(Ansibleホスト)にPython3.8以上がインストールされている
* gitコマンドがインストールされている
* NetBox環境が構築されている
    * Cluster TypesとClustersが登録されている
        * Clustersに登録するクラスタ名はvSphere環境にあるClusterと同じにする必要があります
    * APIで操作できるTokenが生成されている

## 使い方

### venv作成

PythonのVirtual Envを作成します。

```
$ python3 -m venv venv
$ . venv/bin/activate
(venv)$
```

### リポジトリを取得

本リポジトリを取得します。

```
(venv)$ git clone https://github.com/sky-joker/VMUG20220727.git
(venv)$ cd VMUG20220727
```

### 必要なライブラリなどをインストール

以下のコマンドを実行して必要なライブラリやCollectionsをインストールします。

```
(venv)$ pip install -r requirements.txt
(venv)$ ansible-galaxy collection install -r collections/requirements.yml -p collections
```

### パラメーター設定

以下のファイルのパラメーターを各自の環境に合わせた内容に修正してください。

```
(venv)$ vi inventories/group_vars/all.yml
```

|         項目        |  型  |              例               |                                                           説明                                                           |
|---------------------|------|-------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| netbox_url          | str  | `http://netbox.local`         | アクセスするNetBoxのURL                                                                                                  |
| netbox_token        | str  | 7182c49xxxxxxxxxxxxx          | NetBoxを操作するためのToken                                                                                              |
| tenant_name         | str  | example                       | 作成するテナント名                                                                                                       |
| prefix              | str  | 192.168.20.0/24               | NetBoxに作成するPrefix                                                                                                   |
| gateway             | str  | 192.168.20.1/24               | 作成したPrefixに設定するデフォルトゲートウェイ                                                                           |
| vcenter_hostname    | str  | test-vcenter.local            | アクセスするvCenter Serverのホスト名                                                                                     |
| vcenter_username    | str  | `administrator@vsphere.local` | 操作するvCenter Serverのアカウント名                                                                                     |
| vcenter_password    | str  | password                      | vCenter Serverアカウントのパスワード                                                                                     |
| cluster_name        | str  | Cluster                       | VMをデプロイする先のクラスタ名(esxi_hostnameと一緒に使用できない)                                                        |
| esxi_hostname       | str  | esxi-01.local                 | VMをデプロイする先のESXiホスト名(cluster_nameと一緒に使用できない)                                                       |
| datacenter_name     | str  | DC                            | VMをデプロイする先のデータセンター名                                                                                     |
| template_name       | str  | RHEL8.5TMP                    | 複製するテンプレート名                                                                                                   |
| dns_ip_addrs        | list |                               | 新規作成するVMに設定するDNS                                                                                              |
| password            | str  | password                      | 新規作成するVMに設定するrootのパスワード                                                                                 |
| portgroup_name      | str  | pg01                          | 新規作成するVMに接続するポートグループ名                                                                                 |
| vm_size             | str  | 1cpu_2gmem                    | 新規作成するVMのサイズ                                                                                                   |
| count_number        | int  | 1                             | 新規に作成するVM数、[詳細](https://github.com/sky-joker/VMUG20220727/blob/main/roles/vmware_deploy_vm/defaults/main.yml) |
| delete_ipaddr_regex | str  | 192.168.20.[1-5]              | 削除するVMのIPアドレス（正規表現可）                                                                                     |

```yaml
---
# Netbox
# netbox_url: http://example.com
# netbox_token:
# tenant_name:
# prefix: 192.168.20.0/24
# gateway: 192.168.20.1/24

# VMware
# vcenter_hostname:
# vcenter_username: administrator@vsphere.local
# vcenter_password:
# cluster_name:
# esxi_hostname:
# datacenter_name:
# template_name:
# dns_ip_addrs:
#   - 192.168.20.1
# password: The password does set to a guest
# portgroup_name:
# vm_size: 1cpu_2gmem

# Other Params
# count_number: 2
# delete_ipaddr_regex: 192.168.20.[2-3]
```

### Playbook実行

#### NetBoxにテナントとPrefixを作成

以下のコマンドを実行します。

```
(venv)$ ansible-playbook playbooks/netbox_add_prefix_menu.yml -i inventories/inventory
```

#### VMデプロイ

以下のコマンドを実行します。

```
(venv)$ ansible-playbook playbooks/deploy_vm_all_tasks.yml -i inventories/inventory
```

#### VM削除

以下のコマンドを実行します。

```
(venv)$ ansible-playbook playbooks/delete_vm_all_tasks.yml -i inventories/inventory
```
