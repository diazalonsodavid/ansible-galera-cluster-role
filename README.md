galera_cluster
=========

This Role installs a MariaDB Cluster adding as many nodes as needed.

Usage
=====

Inventory file should include the following lines:
 - nodo_1: Will be just first node.
 - node_n: Will be rest of nodes. (One line per node)

```ini
[nodo_1]
mariadb_main_1_1    ansible_ssh_host=192.168.1.198  ansible_ssh_port=22 ansible_ssh_user=vagrant ansible_sudo_pass=vagrant ansible_ssh_pass=vagrant



[nodo_n]
mariadb_main_1_2    ansible_ssh_host=192.168.1.130  ansible_ssh_port=22 ansible_ssh_user=lab01 ansible_sudo_pass=9aFPqugC. ansible_ssh_pass=9aFPqugC.
mariadb_main_1_3    ansible_ssh_host=192.168.1.129 ansible_ssh_port=22 ansible_ssh_user=lab01 ansible_sudo_pass=9aFPqugC. ansible_ssh_pass=9aFPqugC.



[mariadb_1:children]
nodo_1
nodo_n
```

Within vars/main.yml it should be configured:
(**Add as many lines as nodes you want to deploy.**)


```yaml

cluster_hosts:
  - { "node_name": "mariadb_main_1_1", "public_ip": "192.168.1.198", "private_ip": "192.168.1.198" }
  - { "node_name": "mariadb_main_1_2", "public_ip": "192.168.1.130", "private_ip": "192.168.1.130" }
  - { "node_name": "mariadb_main_1_3", "public_ip": "192.168.1.129", "private_ip": "192.168.1.129" }

private_ips:
    - 192.168.1.198
    - 192.168.1.130
    - 192.168.1.129
```

Finally, a new file should be generated for each node added to the vars/hosts/ directory with the name of the node


Role Variables
--------------

**Important Roles Variables**
Located at defaults/main.yml

cluster_sst_user:  
cluster_sst_password:  
superuser_password:  
xtrabackup_mysql_user:  
xtrabackup_mysql_password:  
user_exporter:
password_exporter:

**LVM config Variables**

Located in vars/main.yml

```yaml
device: /dev/sda 		# Name of the disk partition (e.g. /dev/sda)
pv_device: /dev/sda1		# Disk partition where the Volume Group will be created.
partitions_numbers: 1		# Number of desired partitions
size_device: 500GiB		# Desired size in gigabytes of each partition
vg_name: vg_test		# Name we want to give to the Volume Group (VG)
vg_size: 500GiB			# Desired size in gigabytes of each VG


## You should put as many LV blocks as you want to partition. In this example there are 2
lvm:
  - lv_name: lv_test_1 		# name of lv
    lv_size: 250G		# lv size
    filesys_type: ext4		# type of file systems
    mount_points: /test1	# mount point
    chmod: 700			# permissions
    owner: root			# owner
  - lv_name: lv_test_2
    lv_size: 250G
    filesys_type: xfs
    mount_points: /test2
    chmod: 700
    owner: root


```

mysqld_exporter
---------------

Mysql_exporter will expose metrics on port 9094. Prometheus will have to point **<IP-mysql-node>:9094**

Default/main.yml we will find the variables :
user_exporter:
password_exporter:

This user must be created once the mysql node is started and the main password has been changed.


Example Playbook
----------------
```yaml
---
- hosts: mariadb_1
  roles:
    - {role: galera_cluster, become: yes}
```

```Shell
ansible-playbook playbook.yml -i inventories/hosts
```

