## Итоговый проект по курсу «Инфраструктура высоконагруженных систем»

### Проектирование масштабируемой и отказоустойчивой инфраструктуры для высоконагруженных систем с использованием кластеров PostgreSQL, Ceph, Elasticsearch и HAProxy

**Цель проекта:**
Целью проекта является проектирование распределённой высоконагруженной инфраструктуры, обеспечивающей высокую доступность, отказоустойчивость и масштабируемость сервисов, с использованием кластерных решений PostgreSQL, Ceph, Elasticsearch и HAProxy.

**Задачи проекта:**
Для достижения поставленной цели необходимо решить следующие задачи:

1. Проанализировать требования к высоконагруженным системам, включая показатели доступности, отказоустойчивости и масштабируемости.

2. Разработать архитектуру распределённой инфраструктуры с выделением основных уровней:  
   • уровень балансировки нагрузки (HAProxy + Keepalived);  
   • веб-сервисы;  
   • уровень хранения и обработки данных.

3. Спроектировать отказоустойчивый кластер базы данных PostgreSQL с использованием Patroni и Consul:  
   • обеспечить автоматическое переключение (failover);  
   • реализовать репликацию данных.

4. Спроектировать распределённое отказоустойчивое хранилище данных на базе Ceph:  
   • обеспечить репликацию и распределение данных;  
   • повысить надёжность хранения.

5. Реализовать кластер Elasticsearch для обработки и поиска данных:  
   • обеспечить горизонтальное масштабирование;  
   • настроить шардирование и репликацию.

6. Обеспечить балансировку нагрузки и высокую доступность входной точки системы:  
   • реализовать отказоустойчивую связку HAProxy и Keepalived;  
   • настроить механизм виртуального IP (VIP).

7. Проработать взаимодействие между компонентами системы и потоки данных.

8. Оценить отказоустойчивость системы при различных сценариях сбоев:  
   • отказ узла;  
   • отказ сервиса;  
   • сетевые проблемы.  

9. Спроектировать систему мониторинга и логирования:  
    • сбор метрик (Prometheus);  
    • визуализация состояния системы (Grafana);  
    • централизованный сбор логов (Elasticsearch,Logstash,Kibana,Filebeat,Metricbeat).

10. Разработать рекомендации по масштабированию и развитию инфраструктуры.

### Запуск проекта
#### Cоздание инфраструктуры
```bash
# Запускаем PowerShell от имени Администратора
# Клонируем и переходим в папку нашего проекта
git clone https://github.com/dev-itsec/otus-linuxhighload.git
cd otus-linuxhighload

# Создаем инфраструктуру серверов
vagrant up 
vagrant status

# Удаляем инфраструктуру серверов
vagrant destroy -f
```

### Архитектура и функционирование инфраструктуры
![Архитектура проекта](/images/project_map.png)
![Архитектура проекта2](/images/project_map2.png)

### Роли серверов
| №  | Сервер      | IP адрес    | Mgmt IP адрес | Роль               |
|----|-------------|-------------|---------------|--------------------|
| 1  | proxy-node1 | 10.20.30.44 | 192.168.56.14 | HAProxy+Keepalived |
| 2  | proxy-node2 | 10.20.30.45 | 192.168.56.15 | HAProxy+Keepalived |
| 3  | web-node1   | 10.20.30.46 | 192.168.56.16 | Docker Containers  |
| 4  | web-node2   | 10.20.30.47 | 192.168.56.17 | Docker Containers  |
| 5  | prom-node1  | 10.20.30.56 | 192.168.56.26 | Prometheus         |
| 6  | prom-node2  | 10.20.30.57 | 192.168.56.27 | Prometheus         |
| 7  | logstash-node1 | 10.20.30.54 | 192.168.56.24 | Logstash        |
| 8  | logstash-node2 | 10.20.30.55 | 192.168.56.25 | Logstash        |
| 9  | es-node1    | 10.20.30.51 | 192.168.56.21 | Elasticsearch      |
| 10 | es-node2    | 10.20.30.52 | 192.168.56.22 | Elasticsearch      |
| 11 | es-node3    | 10.20.30.53 | 192.168.56.23 | Elasticsearch      |
| 12 | ceph-node1  | 10.20.30.48 | 192.168.56.18 | Ceph               |
| 13 | ceph-node2  | 10.20.30.49 | 192.168.56.19 | Ceph               |
| 14 | ceph-node3  | 10.20.30.50 | 192.168.56.20 | Ceph               |
| 15 | pg-node1    | 10.20.30.41 | 192.168.56.11 | PostgreSQL(Patroni+Consul) |
| 16 | pg-node2    | 10.20.30.42 | 192.168.56.12 | PostgreSQL(Patroni+Consul) |
| 17 | pg-node2    | 10.20.30.43 | 192.168.56.13 | PostgreSQL(Patroni+Consul) |

### Проверка
Consul
```bash
root@pg-node1:~# consul operator raft list-peers
Node      ID                                    Address           State     Voter  RaftProtocol  Commit Index  Trails Leader By
pg-node1  08603f45-1559-063f-27ae-5d4520aafaa8  10.20.30.41:8300  leader    true   3             4005          -
pg-node2  793c4420-57fc-ad70-1fc9-11e0b36b38e8  10.20.30.42:8300  follower  true   3             4005          0 commits
pg-node3  3f6c910c-b2d8-8710-dd50-a64414e2b4bd  10.20.30.43:8300  follower  true   3             4005          0 commit

root@pg-node1:~# consul members
Node      Address           Status  Type    Build   Protocol  DC   Partition  Segment
pg-node1  10.20.30.41:8301  alive   server  1.22.6  2         dc1  default    <all>
pg-node2  10.20.30.42:8301  alive   server  1.22.6  2         dc1  default    <all>
pg-node3  10.20.30.43:8301  alive   server  1.22.6  2         dc1  default    <all>

# Проверяем сервисы в Consul
root@pg-node1:~# curl -s http://localhost:8500/v1/catalog/services | python3 -m json.tool
{
    "consul": [],
    "pgdb": [
        "replica",
        "primary",
        "master"
    ]
}

# Проверить ключи Patroni в Consul
root@pg-node1:~# consul kv get -recurse service/pgdb/
service/pgdb/config:{"ttl":30,"loop_wait":10,"maximum_lag_on_failover":1048576,"postgresql":{"use_pg_rewind":true,"use_slots":true,"parameters":{"archive_mode":"on","wal_level":"hot_standby","max_wal_senders":10,"wal_keep_segments":8,"archive_timeout":"1800s","max_replication_slots":5,"hot_standby":"on","wal_log_hints":"on"}}}
service/pgdb/history:[[1,100663512,"no recovery target specified","2026-04-11T18:06:58.003417+03:00","pg-node2"],[2,167772320,"no recovery target specified","2026-04-13T13:16:33.580978+03:00","pg-node2"]]
service/pgdb/initialize:7626697171950257263
service/pgdb/leader:pg-node2
service/pgdb/members/pg-node1:{"conn_url":"postgres://10.20.30.41:5432/postgres","api_url":"http://10.20.30.41:8008/patroni","state":"running","role":"replica","version":"4.1.1","xlog_location":167861624,"replay_lsn":167861624,"receive_lsn":167861624,"replication_state":"streaming","timeline":3}
service/pgdb/members/pg-node2:{"conn_url":"postgres://10.20.30.42:5432/postgres","api_url":"http://10.20.30.42:8008/patroni","state":"running","role":"primary","version":"4.1.1","xlog_location":167861624,"timeline":3}
service/pgdb/members/pg-node3:{"conn_url":"postgres://10.20.30.43:5432/postgres","api_url":"http://10.20.30.43:8008/patroni","state":"running","role":"replica","version":"4.1.1","xlog_location":167861624,"replay_lsn":167861624,"receive_lsn":167861624,"replication_state":"streaming","timeline":3}
service/pgdb/status:{"optime":167861624,"slots":{"pg_node1":167861624,"pg_node3":167861624,"pg_node2":167861624},"retain_slots":["pg_node1","pg_node2","pg_node3"]}
```
Patroni
```bash
root@pg-node1:~# patronictl -c /etc/patroni/patroni.yml list
+ Cluster: pgdb (7626697171950257263) ---------+----+-------------+-----+------------+-----+
| Member   | Host        | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+----------+-------------+---------+-----------+----+-------------+-----+------------+-----+
| pg-node1 | 10.20.30.41 | Replica | streaming |  3 |   0/A011DA0 |   0 |  0/A011DA0 |   0 |
| pg-node2 | 10.20.30.42 | Leader  | running   |  3 |             |     |            |     |
| pg-node3 | 10.20.30.43 | Replica | streaming |  3 |   0/A011DA0 |   0 |  0/A011DA0 |   0 |
+----------+-------------+---------+-----------+----+-------------+-----+------------+-----+
```
Ceph
```bash
root@ceph-node1:~# cs
root@ceph-node1:/# ceph status
  cluster:
    id:     e26d6c3c-33fe-11f1-b873-694fd6b272ef
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node1,ceph-node3,ceph-node2 (age 16m)
    mgr: ceph-node1.glkakg(active, since 25m), standbys: ceph-node3.eyulxq
    mds: 2/2 daemons up, 7 standby
    osd: 3 osds: 3 up (since 16m), 3 in (since 16m)
 
  data:
    volumes: 2/2 healthy
    pools:   5 pools, 177 pgs
    objects: 296 objects, 53 MiB
    usage:   1.1 GiB used, 29 GiB / 30 GiB avail
    pgs:     177 active+clean

root@ceph-node1:/# ceph orch device ls 
HOST        PATH      TYPE  DEVICE ID   SIZE  AVAILABLE  REFRESHED  REJECT REASONS                                                           
ceph-node1  /dev/sdb  hdd              10.0M  No         25m ago    Has a FileSystem, Insufficient space (<5GB)                              
ceph-node1  /dev/sdc  hdd              10.0G  No         25m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected  
ceph-node2  /dev/sdb  hdd              10.0M  No         25m ago    Has a FileSystem, Insufficient space (<5GB)                              
ceph-node2  /dev/sdc  hdd              10.0G  No         25m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected  
ceph-node3  /dev/sdb  hdd              10.0M  No         16m ago    Has a FileSystem, Insufficient space (<5GB)                              
ceph-node3  /dev/sdc  hdd              10.0G  No         16m ago    Has a FileSystem, Insufficient space (<10 extents) on vgs, LVM detected  

root@ceph-node1:/# ceph fs status
cms_data - 2 clients
========
RANK  STATE              MDS                 ACTIVITY     DNS    INOS   DIRS   CAPS  
 0    active  cms_data.ceph-node2.ninoic  Reqs:    0 /s    23     26     22      2   
        POOL           TYPE     USED  AVAIL  
cephfs_cms_metadata  metadata   616k  9361M  
  cephfs_cms_data      data     996k  9361M  

graf_data - 2 clients
=========
RANK  STATE               MDS                 ACTIVITY     DNS    INOS   DIRS   CAPS  
 0    active  graf_data.ceph-node2.ijftry  Reqs:    0 /s   432    314     69    604   
        POOL            TYPE     USED  AVAIL  
cephfs_graf_metadata  metadata  12.6M  9361M  
  cephfs_graf_data      data     146M  9361M  
         STANDBY MDS            
 graf_data.ceph-node1.ahtiba    
grafana_data.ceph-node2.knjjjy  
  cms_data.ceph-node1.kullin    
grafana_data.ceph-node1.trnuku  
 graf_data.ceph-node3.nyhwtv    
grafana_data.ceph-node3.bxtzzq  
  cms_data.ceph-node3.qfolln    
MDS version: ceph version 17.2.8 (f817ceb7f187defb1d021d6328fa833eb8e943b3) quincy (stable)

root@web-node1:~# df -hT
Filesystem                                           Type     Size  Used Avail Use% Mounted on
tmpfs                                                tmpfs     96M  1.3M   95M   2% /run
/dev/sda1                                            ext4      39G  6.8G   32G  18% /
tmpfs                                                tmpfs    479M     0  479M   0% /dev/shm
tmpfs                                                tmpfs    5.0M     0  5.0M   0% /run/lock
10.20.30.48:6789,10.20.30.49:6789,10.20.30.50:6789:/ ceph     9.2G     0  9.2G   0% /opt/cms/data
10.20.30.48:6789,10.20.30.49:6789,10.20.30.50:6789:/ ceph     9.2G   48M  9.2G   1% /opt/grafana/data
overlay                                              overlay   39G  6.8G   32G  18% /var/lib/docker/overlay2/ba7630b9fbd7810afb57fcf6ebcf4b3451ac4f646a57ef377facc035d796e266/merged
overlay                                              overlay   39G  6.8G   32G  18% /var/lib/docker/overlay2/7399ab7a2907159e78feaf87f0b0949d90128f903e401f9e8290808f216c069a/merged
tmpfs                                                tmpfs     96M  4.0K   96M   1% /run/user/1001
```
Elasticsearch
```bash
root@es-node1:~# curl -k -u elastic:123456 https://localhost:9200/_cat/nodes?v
ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
10.20.30.51           64          96  87    2.53    0.98     0.36 cdfhilmrstw *      es-node1
10.20.30.53           12          97  86    2.62    0.86     0.31 cdfhilmrstw -      es-node3
10.20.30.52           27          97  39    1.92    0.96     0.37 cdfhilmrstw -      es-node2

root@es-node1:~# curl -k -u elastic:123456 https://localhost:9200/_cat/health?v
epoch      timestamp cluster status node.total node.data shards pri relo init unassign unassign.pri pending_tasks max_task_wait_time active_shards_percent
1776076608 10:36:48  prod    green           3         3    108  51    0    0        0            0             0                  -                100.0

root@es-node1:~# curl -k -u elastic:123456 "https://localhost:9200/_cat/indices?v"
health status index                                                              uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
green  open   .internal.alerts-ml.anomaly-detection-health.alerts-default-000001 EIEHG4egSDO00KJWD761Yg   1   1          0            0       498b           249b         249b
yellow open   .internal.alerts-observability.apm.alerts-default-000001           jQo4DdNlRN6M5jHY0sl5wA   1   1          0            0       249b           249b         249b
green  open   .internal.alerts-stack.alerts-default-000001                       Z_9ACrG-SDC7zgN9WWy3yA   1   1          0            0       498b           249b         249b
yellow open   .internal.alerts-observability.slo.alerts-default-000001           DAeUZJybR5GIvYnYlKk5sA   1   1          0            0       249b           249b         249b
yellow open   my_test_index                                                      OC7pzN5NQH2gnfTsA_2V4g   3   2          8            0       58kb           29kb         29kb
yellow open   .ds-metricbeat-8.19.13-2026.04.07-000001                           s-vpcSe4RSeElkngoq1yVg   1   1      84766            0     73.2mb         73.2mb       73.2mb
green  open   .internal.alerts-streams.alerts-default-000001                     1rKcudm4TC2S8huslPr3qQ   1   1          0            0       498b           249b         249b
yellow open   filebeats-2026.04.07                                               def1ZF0VRlS8xEStC4CpEQ   1   1      18679            0     11.6mb         11.6mb       11.6mb
yellow open   .internal.alerts-observability.logs.alerts-default-000001          Zui7nCe8T5qCtcUgbCJQgA   1   1          0            0       249b           249b         249b
green  open   .internal.alerts-transform.health.alerts-default-000001            ats5CecNReK0fUheZuQN4A   1   1          0            0       498b           249b         249b
green  open   .ds-.monitoring-es-8-mb-2026.04.13-000002                          2DLLJl3RRmGWRf3C2TbXWg   1   1      30149            0     61.1mb         39.8mb       39.8mb
green  open   .internal.alerts-security.attack.discovery.alerts-default-000001   UweuRqvWSxWBtzAucLxcIg   1   1          0            0       498b           249b         249b
green  open   filebeats-2026.04.13                                               M_eV10LkRgK2AdnaE3GQmg   1   1      67911            0     83.6mb         55.3mb       55.3mb
green  open   .internal.alerts-dataset.quality.alerts-default-000001             xndr1Q1qRRya2Ts6jGp5pA   1   1          0            0       498b           249b         249b
yellow open   .internal.alerts-observability.metrics.alerts-default-000001       WN2pgzgKSimKVWtDv3qLbg   1   1          0            0       249b           249b         249b
green  open   .internal.alerts-ml.anomaly-detection.alerts-default-000001        GmXrCSNMQtSvyly6QrBizQ   1   1          0            0       498b           249b         249b
green  open   .internal.alerts-default.alerts-default-000001                     AArax033TYOBlr6yx8AUSQ   1   1          0            0       498b           249b         249b
yellow open   my_test_index2                                                     ypw0owkgSae8dGPUdTmu7A   3   2          0            0      1.4kb           747b         747b
green  open   .internal.alerts-observability.uptime.alerts-default-000001        67zu17NjTpKEwJ6KKfKrNA   1   1          0            0       498b           249b         249b
yellow open   .internal.alerts-security.alerts-default-000001                    7w3U8FpJS_uDZ_TStfQg8w   1   1          0            0       249b           249b         249b
green  open   .internal.alerts-observability.threshold.alerts-default-000001     zFHDb772RdSce3En0Ykhaw   1   1          0            0       498b           249b         249b
green  open   .ds-.monitoring-es-8-mb-2026.04.07-000001                          8j4o_D4iRSW4LQhX3KcYOw   1   1     175430            0    227.9mb        113.9mb      113.9mb
```
CMS, Kibana, Grafana
```bash
root@web-node1:/# docker ps
CONTAINER ID   IMAGE                                     COMMAND                  CREATED        STATUS         PORTS                                         NAMES
8e654a071b62   docker.elastic.co/kibana/kibana:8.19.13   "/bin/tini -- /usr/l…"   4 hours ago    Up 2 hours     0.0.0.0:5601->5601/tcp, [::]:5601->5601/tcp   kibana
3a4e2c64dae5   itsec7/django-cms-quickstart              "python manage.py ru…"   46 hours ago   Up 2 hours     0.0.0.0:80->80/tcp, [::]:80->80/tcp           cms-web-1
5847d07cbb7b   grafana/grafana:latest                    "/run.sh"                4 days ago     Up 7 minutes   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   grafana

root@web-node2:/# docker ps
CONTAINER ID   IMAGE                                     COMMAND                  CREATED         STATUS         PORTS                                         NAMES
1cbfacfb8de2   docker.elastic.co/kibana/kibana:8.19.13   "/bin/tini -- /usr/l…"   4 minutes ago   Up 4 minutes   0.0.0.0:5601->5601/tcp, [::]:5601->5601/tcp   kibana
b70c934fdcd2   itsec7/django-cms-quickstart              "python manage.py ru…"   46 hours ago    Up 2 hours     0.0.0.0:80->80/tcp, [::]:80->80/tcp           cms-web-1
37541c3fecc4   grafana/grafana:latest                    "/run.sh"                4 days ago      Up 2 hours     0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   grafana
```
- Load Balancer HAProxy
![haproxy](/images/haproxy.png)
- Cluster Ceph
![ceph](/images/ceph.png)
- Cluster Consul
![consul](/images/consul.png)
- Cluster Elasticsearch
![elasticsearch](/images/elasticsearch.png)
- Logs Kibana
![kibana](/images/kibana.png)
- Monitoring Grafana
![grafana](/images/grafana.png)
- Monitoring Prometheus
![prometheus](/images/elasticsearch.png)





