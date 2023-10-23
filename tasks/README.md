## TETA 02
#### Настроека окружения для работы с IaaC

Выполни следующие действия( Linux | MacOS ):
* установи Python 3.x и pip менеджер пакетов к нему
* установи требуемые пакеты из requirements.txt:
```
pip install -r requirements.txt
```
Репозитория расчитан на два окружения **infra/prod** и **infra/stage**, далее все настройки будут производиться в **infra/stage** 
* заполни инветрои файл ( prod | stage) hosts:
```
# ETCD
[etcd_cluster]
10.0.10.2 ansible_host=77.105.185.176
10.0.10.3 ansible_host=77.105.185.177
10.0.10.4 ansible_host=77.105.185.180

# HAProxy
[balancers]
10.0.10.5 ansible_host=77.105.185.181 hostname=pg-lb-01

# PostgreSQL nodes
[master]
10.0.10.2 ansible_host=77.105.185.176 hostname=pgnode01 postgresql_exists=false

[replica]
10.0.10.3 ansible_host=77.105.185.177 hostname=pgnode02 postgresql_exists=false
10.0.10.4 ansible_host=77.105.185.180 hostname=pgnode03 postgresql_exists=false

[postgres_cluster:children]
master
replica


```
> **_NOTE:_**  WIP / данный репозитория не адаптирован для удобной установки множестка кластеров
* заполни **group_vars** :

  **group_vars/all/vault:**
```
ansible_user: '<your_admin_user>'

patroni_superuser_username: "postgres"
patroni_superuser_password: "<ChangeMeASAP>"
patroni_replication_username: "replicator"
patroni_replication_password: "ChangeMeASAP"

# Add Users / DBs / API
api_auth_username: "api_usr"
api_auth_password: "ChangeMeASAP"
api_database: "weather"

```
* для создания БД в нашем кластере выполни следующий playbook:
```
ansible-playbook postgresql_cluster-1.8.0/add_db.yml -D --ask-vault-password
```

#### Настроека APP
основной инструмен для деплоя - HELM, подделживает репозитория два окружения **helm/prod** и **helm/stage**, далее все настройки будут производиться в **helm/stage**
Деплой API можно выполнить следующей командой:
```
helm upgrade -i -n sre-cource-student-122 api . --set "app.settings.<ChangeMeASAP>"

```
добавляем в /etc/hosts следующее :
```
91.185.85.213   teta-02.local
```
проверка:
```
без БД:
~ % curl -i  http://teta-02.local/weatherforecast
HTTP/1.1 500 Internal Server Error
Date: Mon, 23 Oct 2023 05:52:08 GMT
Content-Length: 0
Connection: keep-alive
С БД:
~ % curl -i  http://teta-02.local/weatherforecast
HTTP/1.1 200 OK
Date: Mon, 23 Oct 2023 05:52:55 GMT
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
```

