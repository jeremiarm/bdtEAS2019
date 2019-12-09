# EAS Basis Data Terdistribusi 2019

# Skema
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/Configuration.png) <br />

# Spesifikasi

```
1. node1:
- OS : geerlingguy/centos7
- RAM : 512 MB
- IP : 192.168.16.102
- Aplikasi :
    - PD
    - TiDB
    - Node exporter
    - Grafana
    - Prometheus
2. node2:
- OS : geerlingguy/centos7
- RAM : 512 MB
- IP : 192.168.16.103
- Aplikasi :
    -PD
    -Node exporter
3. node3:
- OS : geerlingguy/centos7
- RAM : 512 MB
- IP : 192.168.16.104
- Aplikasi :
    - PD
    - Node exporter
4. node4:
- OS : geerlingguy/centos7
- RAM : 512 MB
- IP : 192.168.16.105
- Aplikasi :
    - TiKV
    - Node exporter
5. node5:
- OS : geerlingguy/centos7
- RAM : 512 MB
- IP : 192.168.16.106
- Aplikasi :
    - TiKV
    - Node exporter
6. node6:
- OS : geerlingguy/centos7
- RAM : 512 MB
- IP : 192.168.16.107
- Aplikasi :
    - TiKV
    - Node exporter
```
# Instalasi dan Konfigurasi TiDB
1. Konfigurasi ``VagrantFile`` <br />
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    (1..6).each do |i|
      config.vm.define "node#{i}" do |node|
        node.vm.hostname = "node#{i}"

        # Gunakan CentOS 7 dari geerlingguy yang sudah dilengkapi VirtualBox Guest Addition
        node.vm.box = "geerlingguy/centos7"
        node.vm.box_version = "1.2.19"
        
        # Disable checking VirtualBox Guest Addition agar tidak compile ulang setiap restart
        node.vbguest.auto_update = false
        
        node.vm.network "private_network", ip: "192.168.16.#{101+i}"
        
        node.vm.provider "virtualbox" do |vb|
          vb.name = "node#{i}"
          vb.gui = false
          vb.memory = "512"
        end
  
        node.vm.provision "shell", path: "provision/bootstrap.sh", privileged: false
      end
    end
  end
  
```
2. Konfigurasi ``bootstrap.sh`` <br />
```
# Referensi:
# https://pingcap.com/docs/stable/how-to/deploy/from-tarball/testing-environment/

# Update the repositories
# sudo yum update -y

# Copy open files limit configuration
sudo cp /vagrant/config/tidb.conf /etc/security/limits.d/

# Enable max open file
sudo sysctl -w fs.file-max=1000000

# Copy atau download TiDB binary dari http://download.pingcap.org/tidb-v3.0-linux-amd64.tar.gz
cp /vagrant/installer/tidb-v3.0-linux-amd64.tar.gz .

# Extract TiDB binary
tar -xzf tidb-v3.0-linux-amd64.tar.gz

# Install MariaDB to get MySQL client
sudo yum -y install mariadb

# Install Git
sudo yum -y install git

# Install nano text editor
sudo yum -y install nano

# Install node exporter
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
tar -xzf node_exporter-0.18.1.linux-amd64.tar.gz
```
3. Download TiDB binary dari http://download.pingcap.org/tidb-v3.0-linux-amd64.tar.gz untuk menghemat waktu saat ``vagrant up`` <br />
4. Konfigurasi ``tidb.conf`` <br />
```
vagrant        soft        nofile        1000000
vagrant        hard        nofile        1000000
```
5. Install plugin ``vbguest`` dengan perintah <br />
```
vagrant plugin install vagrant-vbguest
```
6. Jalankan ``vagrant up`` <br />
7. Masuk ke ``node1`` dengan ``vagrant ssh node1``, kemudian jalankan <br />
```
cd tidb-v3.0-linux-amd64
./bin/pd-server --name=pd1 --data-dir=pd --client-urls="http://192.168.16.102:2379" --peer-urls="http://192.168.16.102:2380" --initial-cluster="pd1=http://192.168.16.102:2380,pd2=http://192.168.16.103:2380,pd3=http://192.168.16.104:2380" --log-file=pd.log &
```
8. Masuk ke ``node2`` dengan ``vagrant ssh node2``, kemudian jalankan <br />
```
cd tidb-v3.0-linux-amd64
./bin/pd-server --name=pd2 --data-dir=pd --client-urls="http://192.168.16.103:2379" --peer-urls="http://192.168.16.103:2380" --initial-cluster="pd1=http://192.168.16.102:2380,pd2=http://192.168.16.103:2380,pd3=http://192.168.16.104:2380" --log-file=pd.log &
```
9. Masuk ke ``node3`` dengan ``vagrant ssh node3``, kemudian jalankan <br />
```
cd tidb-v3.0-linux-amd64
./bin/pd-server --name=pd3 --data-dir=pd --client-urls="http://192.168.16.104:2379" --peer-urls="http://192.168.16.104:2380" --initial-cluster="pd1=http://192.168.16.102:2380,pd2=http://192.168.16.103:2380,pd3=http://192.168.16.104:2380" --log-file=pd.log &
```
10. Masuk ke ``node4`` dengan ``vagrant ssh node4``, kemudian jalankan <br />
```
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.16.102:2379,192.168.16.103:2379,192.168.16.104:2379" --addr="192.168.16.105:20160" --data-dir=tikv --log-file=tikv.log &
```
11. Masuk ke ``node5`` dengan ``vagrant ssh node5``, kemudian jalankan <br />
```
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.16.102:2379,192.168.16.103:2379,192.168.16.104:2379" --addr="192.168.16.106:20160" --data-dir=tikv --log-file=tikv.log &
```
12. Masuk ke ``node6`` dengan ``vagrant ssh node6``, kemudian jalankan <br />
```
cd tidb-v3.0-linux-amd64
./bin/tikv-server --pd="192.168.16.102:2379,192.168.16.103:2379,192.168.16.104:2379" --addr="192.168.16.107:20160" --data-dir=tikv --log-file=tikv.log &
```
13. Masuk lagi ke ``node1`` dengan ``vagrant ssh node1``, kemudian jalankan <br />
```
cd tidb-v3.0-linux-amd64
./bin/tidb-server --store=tikv --path="192.168.16.102:2379" --log-file=tidb.log &
```
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/install_tidb_1.jpg) <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/install_tidb_2.jpg) <br />

# Implementasi Aplikasi
1. Masuk ke ``node1`` dengan ``vagrant ssh node1`` <br />
2. Masuk ke dalam ``mysql`` dari ``node1`` dengan <br />
```
mysql -u root -h 192.168.16.102 -P 4000
```
3. Jalankan <br />
```
create database meetingroom;
create user if not exists 'mradmin'@'%' identified by 'mradmin'; 
grant all privileges on meetingroom.* to 'mradmin'@'%';
flush privileges;
```
untuk membuat ``database`` meetingroom, dengan ``username`` mradmin dan ``password`` mradmin. <br />
4. git clone https://github.com/dennyrengganis/KPMR.git <br />
5. Masuk ke folder KPMR <br />
6. copy ``.env`` basic dari project laravel <br />
7. buka cmd, kemudian jalankan <br />
```
composer install
php artisan key:generate
```
8. Ubah ``.env`` jadi <br />
```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:UGyDfxmXLfxpG2OEiaYt8dVrM1zF9o+IqedEshkmbe4=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=192.168.16.102
DB_PORT=4000
DB_DATABASE=meetingroom
DB_USERNAME=mradmin
DB_PASSWORD=mradmin

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=465
MAIL_USERNAME=meetingr44@gmail.com
MAIL_PASSWORD="DetailInfo"
MAIL_ENCRYPTION=ssl

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```
9. Jalankan pada project laravel ini <br />
```
php artisan migrate
php artisan db:seed
```
10. Kemudian jalankan ``php -S localhost:8000 -t public/`` <br />

# Dokumentasi CRUD
1. Proses Create <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_create1.jpg) <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_create2.jpg) <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_create3.jpg) <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_create4.jpg) <br />
2. Proses Read <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_read.jpg) <br />
3. Proses Update <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_update1.jpg) <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_update2.jpg) <br />
4. Proses Delete <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_delete1.jpg) <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/CRUD_delete2.jpg) <br />

# Instalasi Sysbench
1. masuk ke node 1 dengan ``vagrant ssh node1`` <br />
2. Jalankan <br />
```
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
sudo yum -y install sysbench
git clone https://github.com/pingcap/tidb-bench.git
cd tidb-bench/sysbench
```
3. Ubah file ``config`` sehingga host menjadi 192.168.16.102 dan db menjadi test <br />
4. Jalankan untuk melakukan testing <br />
```
 ./run.sh point_select prepare 100
 ./run.sh point_select run 100
```

# Uji Performa Aplikasi
1. Jmeter ( testing 100,500, dan 100 koneksi ke http://localhost:8000/ <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/jmeter_all_result.jpg) <br />
2. Sysbench <br />
- 3 PD <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/sysbench_3PD.jpg) <br />
- 2 PD <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/sysbench_2PD.jpg) <br />
- 1 PD <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/sysbench_1PD.jpg) <br />

# Implementasi Monitoring Grafana
1. Masuk ke node 1-6 dan jalankan di masing-masing node perintah di bawah untuk menjalankan node exporter <br />
```
cd node_exporter-0.18.1.linux-amd64
./node_exporter --web.listen-address=":9100" --log.level="info" &
```
2. Install Prometheus pada node 1 dengan menjalankan <br />
```
wget https://github.com/prometheus/prometheus/releases/download/v2.2.1/prometheus-2.2.1.linux-amd64.tar.gz
tar -xzf prometheus-2.2.1.linux-amd64.tar.gz
```
3. Ubah isi dari file ``prometheus.yml`` yang berada di dalam folder ``prometheus-2.2.1.linux-amd64`` menjadi <br />
```
global:
  scrape_interval:     15s  # By default, scrape targets every 15 seconds.
  evaluation_interval: 15s  # By default, scrape targets every 15 seconds.
  # scrape_timeout is set to the global default value (10s).
  external_labels:
    cluster: 'test-cluster'
    monitor: "prometheus"

scrape_configs:
  - job_name: 'overwritten-nodes'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.16.102:9100'
      - '192.168.16.103:9100'
      - '192.168.16.104:9100'
      - '192.168.16.105:9100'
      - '192.168.16.106:9100'
      - '192.168.16.107:9100'

  - job_name: 'tidb'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.16.102:10080'

  - job_name: 'pd'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.16.102:2379'
      - '192.168.16.103:2379'
      - '192.168.16.104:2379'

  - job_name: 'tikv'
    honor_labels: true  # Do not overwrite job & instance labels.
    static_configs:
    - targets:
      - '192.168.16.105:20180'
      - '192.168.16.106:20180'
      - '192.168.16.107:20180'
```
4.Di dalam folder tersebut, jalankan <br />
```
./prometheus --config.file="./prometheus.yml" --web.listen-address=":9090" --web.external-url="http://192.168.16.102:9090/" --web.enable-admin-api --log.level="info" --storage.tsdb.path="./data.metrics" --storage.tsdb.retention="15d" &
```
5. Prometheus akan berjalan <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/prometheus_result.jpg) <br />
6.Install grafana di node 1 dengan menjalankan <br />
```
wget https://dl.grafana.com/oss/release/grafana-6.5.1.linux-amd64.tar.gz

tar -zxf grafana-6.5.1.linux-amd64.tar.gz
```
7. Masuk ke folder ``grafana-6.5.1/conf`` dan buat file ``grafana.ini`` <br />
8. Ubah isi ``grafana.ini`` menjadi <br />
```
[paths]
data = ./data
logs = ./data/log
plugins = ./data/plugins
[server]
http_port = 3000
domain = 192.168.16.102
[database]
[session]
[analytics]
check_for_updates = true
[security]
admin_user = admin
admin_password = admin
[snapshots]
[users]
[auth.anonymous]
[auth.basic]
[auth.ldap]
[smtp]
[emails]
[log]
mode = file
[log.console]
[log.file]
level = info
format = text
[log.syslog]
[event_publisher]
[dashboards.json]
enabled = false
path = ./data/dashboards
[metrics]
[grafana_net]
url = https://grafana.net
```
9. Masuk ke folder ``grafana.6.5.1`` dan jalankan <br />
```
./bin/grafana-server --config="./conf/grafana.ini" &
```
10. Grafana akan berjalan, akses pada browser dengan tujuan ``192.168.16.102:3000`` <br />
11. Masukkan username ``admin`` dan password ``admin`` <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_tampilanawal.jpg) <br />
12. Pada ``home``, pilih ``Create your first data source``, pilih ``Prometheus``, kemudian isikan URL menjadi ``http://192.168.16.102:9090``. Kemudian pilih ``save and test`` <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_prometheus_datasources.jpg) <br />
13. Kemudian kembali ke home, pada bagian kiri pilih icon kotak 4 (dashboard), kemudian pilih manage <br />
14. Pilih import, dan upload file .json dari  https://github.com/pingcap/tidb-ansible/tree/master/scripts <br />
15. Pada tugas ini , file-file .json yang saya gunakan adalah <br />
```
tikv_summary.json
tikv_details.json
tikv_trouble_shooting.json
pd.json
tidb.json
tidb_summary.json
```
yang sudah saya masukkan ke dalam https://github.com/jeremiarm/bdtEAS2019/tree/master/json <br />
16. Hasil TIKV SUMMARY <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_test_cluster_tikv_summary.jpg) <br />
17. Hasil TIKV DETAILS <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_test_cluster_tikv_details.jpg) <br />
18. Hasil TIKV TROUBLE SHOOTING <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_test_cluster_tikv_troubleshooting.jpg) <br />
19. Hasil PD <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_test_cluster_pd.jpg) <br />
20. Hasil TiDB <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_test_cluster_tidb.jpg) <br />
21. Hasil TiDB SUMMARY <br />
![alt text](https://github.com/jeremiarm/bdtEAS2019/blob/master/screenshot/grafana_test_cluster_tidb_summary.jpg) <br />
