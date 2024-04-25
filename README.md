# Install Databases
### w/ Ansible by Iqbal Rizky Wijaya

Automatisasi install database menggunakan ansible playbook.
## 1. Notes
### B. Playbook Notes
- Databases: Mariadb, Mysql, Postgresql, Redis, Mongodb
- Hanya pernah ditest di server Ubuntu 22.04 LTS
- Belum ada error handling di playbook
- Inventory/hosts dapat dirubah di dbs/hosts, pastikan jangan mengubah nama group ataupun variabel (master1, master2)
- Server yang masuk ke group master hanya boleh 1
- Variabel untuk setiap database berada di dbs/group_vars/all
- .conf setiap database bisa di-konfigurasi kecuali redis standalone
- uninstall_db.yaml akan menghapus database packages dan data-nya
- Jika mengalami error ketika menjalankan playbook install_db.yaml, pertama analisa error, kemudian uninstall database yang mengalami error dengan uninstall_db.yaml, baru install lagi databse yang error dengan install_db.yaml
### B. Database Notes
**Mariadb**
- Tipe cluster: Galera
- Variabel yang dapat diubah: db_user, db, password, root_password
- Standalone my.cnf: dbs/roles/mariadb/templates/mariadb_standalone.cnf.j2
- Cluster my.cnf: dbs/roles/mariadb/templates/mariadb_cluster.cnf.j2

**Mongodb**
- Tipe cluster: Replicaset
- Cluster minimal node: 3
- Jika server tidak support mongodb versi 7, install versi 4.4 dengan install_mongodb_old
- Mongodb client di slave mongodb_old secara default tidak bisa membaca data, gunakan ‘rs.slaveOk()’ query untuk mongo shell atau atur readPreference menjadi secondary. Baca https://stackoverflow.com/questions/39965964/how-do-i-set-secondary-preferred-reads-on-mongo-database-from-parse-server 
- Variabel yang dapat diubah: mongodb_username, mongodb_password, mongodb_data_dir, replSetName
- Standalone mongod.conf: dbs/roles/mongodb/templates/mongodb_standalone.conf.j2
- Cluster mongod.conf: dbs/roles/mongodb/templates/mongodb_cluster.conf.j2

**Mysql**
- Tipe cluster: Master to master, Master to slave
- Untuk mysql master to master cluster, server yang digunakan adalah server yang masuk ke group master, dan member pertama dari group slaves
- Variabel yang dapat diubah: db_user, db_password, root_password
- Standalone my.cnf: dbs/roles/mysql/templates/standalone.cnf.j2
- M2M master1 my.onf: dbs/roles/mysql/templates/master_master-master1.cnf.j2
- M2M master2 my.cnf: dbs/roles/mysql/templates/master_master-master2.cnf.j2
- M2S master my.cnf: dbs/roles/mysql/templates/master_slave-master.cnf.j2
- M2S slave my.cnf: dbs/roles/mysql/templates/master_slave-slave.cnf.j2

**Postgresql**
- Tipe cluster: Master to slave
- Variabel yang dapat diubah: postgres_password, postgresql_version, postgresql_datadir
- Standalone postgresql.conf: dbs/roles/postgresql/templates/standalone.conf.j2
- Cluster postgresql.conf: dbs/roles/postgresql/templates/master_cluster.conf.j2

**Redis**
- Tipe cluster: Replication
- Cluster minimal node: 3
- Setiap server dalam cluster akan menjalankan 2 redis instance. 1 redis master dengan port 7000, 1 redis slave dengan port 7001
- Variabel yang dapat diubah: access_key
- Cluster redis.conf: dbs/roles/redis/templates/cluster_redis.conf.j2
## 2. Menjalankan playbook
### A. Cara menjalankan playbook install_db.yaml
Pertama install dulu package ansible ('apt install ansible' untuk ubuntu)
1. Edit Inventory/hosts di dbs/hosts, sesuaikan IP menjadi server yang ingin di-manage. Jika ingin menambah server, tambahkan di group slave dan beri variabel id.
2. Edit dbs/install_db.yaml, pilih database mana yang ingin diinstall dan tentukan tipenya (cluster/standalone).
3. Edit variabel yang mungkin ingin diubah di dbs/group_vars/all.
4. Jika ingin menambah konfigurasi di .conf database, edit file .conf yang sudah saya definisikan di bab sebelumnya.
5. Jalankan playbook dengan memasuki direktori dbs/, dan jalankan perintah:
ansible-playbook install_db.yaml
### B. Cara menjalankan playbook uninstall_db.yaml
1. Edit Inventory/hosts di dbs/hosts sesuai dengan server yang ingin diuninstall databasenya
2. Edit dbs/uninstall.yaml, pilih database apa saja yang ingin diuninstall
3. Jalankan playbook dengan memasuki direktory dbs/, dan jalankan perintah:
ansible-playbook uninstall_db.yaml
