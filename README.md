# outline
outline知识库部署

参考：
https://github.com/outline/outline/


1. 创建compose项目路径

   ```sh
   mkdir /opt/compose/outline
   cd /opt/compose/outline
   ```
2. 用于拉取容器镜像的脚本pull_images.sh

   ```bash
   #!/bin/bash
set -e

COMPOSE_FILE="$(dirname "$0")/docker-compose.yml"

IMAGES=($(grep -oP 'image:\s*\K\S+' "$COMPOSE_FILE"))

echo "=========================================="
echo "  批量拉取 Outline 容器镜像"
echo "  共 ${#IMAGES[@]} 个镜像"
echo "=========================================="
echo ""

for image in "${IMAGES[@]}"; do
  echo "[$(date '+%H:%M:%S')] 正在拉取: $image"
  docker pull "$image"
  echo "[$(date '+%H:%M:%S')] $image 拉取完成"
  echo "------------------------------------------"
done

echo ""
echo "=========================================="
echo "  全部镜像拉取完成!"
echo "=========================================="

   ```
3. 创建docker network

   ```bash
   docker network create --driver bridge --subnet 192.168.232.0/24 outline_network
   ```

4. 创建两个secretKey

   ```sh
   # create outline SECRET_KEY
   openssl rand -hex 32
   # create outline UTILS_SECRET
   openssl rand -hex 32
   ```
   
5. 初始化数据库

   ```sh
   docker-compose -f init_database.yml up -d
   ```

   执行sql

   ```sql
   -- # create role
   CREATE ROLE "keycloak" SUPERUSER CREATEDB CREATEROLE LOGIN REPLICATION BYPASSRLS PASSWORD 'your_password';
   
   -- # create db
   CREATE DATABASE "keycloak"
   WITH
     OWNER = "keycloak"
     ENCODING = 'UTF8'
     TABLESPACE = "pg_default";
   ```

   销毁容器

   ```sh
   docker-compose -f init_database.yml down
   ```
   
6. 运行docker-compose.yml

   ```sh
   docker-compose up -d
   ```
   
7. 创建minio bucket

   访问minio控制台页面，手动创建outline-bucket
   
9. 初始化oidc鉴权

   访问keycloak控制台页面，手动创建realm应用、realm客户端，生成密钥并填入到.env的OIDC_CLIENT_SECRET

   通过`docker-compose up -d`重新创建outline容器，登录访问outline

10. 完成！

