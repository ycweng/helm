# 建DataDir
```bash
mkdir -p /data/mysql_8_4-0
mkdir -p /data/mysql_8_4-1
mkdir -p /data/mysql_8_4-2
```

# db cluster
```
mysqlsh --uri="root:$MYSQL_ROOT_PASSWORD@mysql-8-4-0.mysql-8-4-headless.mysql-test.svc.cluster.local"

\js


dba.configureInstance('root@mysql-8-4-0.mysql-8-4-headless.mysql-test.svc.cluster.local')
dba.configureInstance('root@mysql-8-4-1.mysql-8-4-headless.mysql-test.svc.cluster.local')
dba.configureInstance('root@mysql-8-4-2.mysql-8-4-headless.mysql-test.svc.cluster.local')
var cluster=dba.createCluster('mycluster')
cluster.status()

cluster.addInstance('root@mysql-8-4-1.mysql-8-4-headless.mysql-test.svc.cluster.local', {recoveryMethod: 'clone'});
cluster.addInstance('root@mysql-8-4-2.mysql-8-4-headless.mysql-test.svc.cluster.local', {recoveryMethod: 'clone'});
cluster.status()
cluster=dba.getCluster('mycluster')


```