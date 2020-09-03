# backup_restore_graylog


# 1 - Criar pastas de backup no servidor.




mkdir -p /backup/mongo
mkdir -p /backup/elasticsearch

# 2 - Fazendo backup do mongodb

	- mongodump -h localhost --gzip -o /backup/mongo



# 3 - Crie no arquivo do elasticsearch o caminho que irá ficar o backup, isso tem que 
ser feito tanto no servidor que irá fazer o backup quanto no que vai fazer o restore.

echo "path.repo: /backup/elasticsearch" >> /etc/elasticsearch/elasticsearch.yml
systemctl restart elasticsearch






# 4 - Criando repositorio para preparar o backup.

"#!/bin/bash"

curl -XPUT -H "Content-Type: application/json;charset=UTF-8" 'http://localhost:9200/_snapshot/esbackup' -d '{
  "type": "fs",
  "settings": {
     "location": "/backup/elasticsearch/",
     "compress": true
  }
}'




# 5 - Fazendo o backup.

curl -XPUT  "http://localhost:9200/_snapshot/esbackup/linuxpoint_snapshot?wait_for_completion=true"



# 6 - Fazendo a compactação dos arquivos.

cd /backup
tar -cvzf  elasticsearch.tar.gz elasticsearch



# 7 - Criar pastas no servidor do restore.

mkdir -p /backup/mongo
mkdir -p /backup/elasticsearch


8 - Enviando os arquivos para o servidor que irá fazer o restore.

scp /backup/elasticsearch.tar.gz root@ip:/backup/
scp -r /backup/mongo root@ip:/backup/mongo


9 - Fazendo o restore dos backups

cd /backup/
tar -xzf elasticsearch.tar.gz
chown -R elasticsearch:elasticsearch /backup/elasticsearch/
mongorestore -h localhost --gzip --dir /backup/mongo

10 - Criando repositorio para restore

curl -XPUT -H "Content-Type: application/json;charset=UTF-8" 'http://localhost:9200/_snapshot/esbackup' -d '{
  "type": "fs",
  "settings": {
     "location": "/backup/elasticsearch/",
     "compress": true
  }
}'


11 - Fechando o indice default para fazer o restore

curl -XPOST -H "Content-Type: application/json;charset=UTF-8" 'http://localhost:9200/graylog_0/_close'


12 - Fazendo o restore do elasticsearch.

curl  -XPOST "http://localhost:9200/_snapshot/esbackup/linuxpoint_snapshot/_restore?wait_for_completion=true"

13 - Abrindo o indice default.

curl -XPOST -H "Content-Type: application/json;charset=UTF-8" 'http://localhost:9200/graylog_0/_open'


14 - Removendo o backup do servidor de produção.


rm -rf /backup/elasticsearch/*
rm -rf /backup/mongo/*


curl -XDELETE localhost:9200/esbackup