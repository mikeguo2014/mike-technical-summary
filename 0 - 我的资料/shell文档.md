
# shell总结

## 1 命令行


## 1.1 awk
```bash
$1代表第1列，$0代表所有列
awk '{print $3}' staff.csv
awk '{print $0}' staff.csv
awk '{print $1 $2}' staff.csv 		# 中间的字符不管用
awk '{print $1" "$1}' staff.csv 	# 加引号才生效
awk '{print NR FS NF}' staff.csv 	# NR代表行号，FS代表分隔符，NF代表列数
awk '{print $(NF-1)}' staff.csv 	# 支持计算，加上$代表列内容

awk -F"," '{print $1,$2}' staff.csv	 	# -F必须写在前面，双引号可以省略，变成-F,
awk '{print $1,$2}' FS="," staff.csv 				# FS必须写在后面，双引号可以省略，变成FS=,
awk '{print $1,$2}' FS="," OFS=":" staff.csv 		# OFS指定输出分隔符


awk 'NR==1{print $3}' FS=, staff.csv 			# 打印第1行第3列
awk 'NR!=1{print $3}' FS=, staff.csv 			# 打印所有行的第3列，除了第1行
awk '/Gavo/{print $3}' FS=, staff.csv 			# 打印Gavo的Age
awk '/Gavo|Jane/{print $0}' FS=, staff.csv 	# 打印Gavo或Jane的记录
awk '$3>40{print $0}' FS=, staff.csv 			# 打印40岁以上的记录，注意这里表头也按字符串来比较了
awk '/Gavo/ || $3>40{print $0}' FS=, staff.csv 	# 打印Gavo或40岁以上的记录
awk '$1 ~ /US/{print $0}' FS=, staff.csv 		# 打印第1列为US的所有记录
awk '$3 ~ /^2/{print $0}' FS=, staff.csv 		# 打印第三列以2开头的所有记录，即所有二十多岁的记录
awk '/Gavo/{print $0}/Jane/{print $2}' FS=, staff.csv 	# 打印Gavo的整行记录并打印Jane的第二列

求所有人的年龄总和：
awk '{s+=$3}END{print s}' staff.csv
awk '{s+=$3;print $2":"$3}END{print "SUM:"s}' staff.csv

下一个命令可以求平均值，也就是求和之后除以行数NR：
awk '{a+=$3}END{print a/NR}' staff.csv

去重
awk '{print $1}' staff.csv | uniq  
```


## 2. 脚本

### 2.1 docker 部署 nginx
```bash
#!/bin/bash

# set -x

cur_dir=$(
    cd "$(dirname "$0")"
    pwd
)

# nginx9999配置
nginx_docker_image=nexus.ahi.internal:5000/nginx
nginx_container_name=nginx_9999
nginx_port_in_docker=80
nginx_port=9999
nginx_conf_path=${cur_dir}/nginx/conf
nginx_confd_path=${cur_dir}/nginx/conf.d
nginx_html_path=${cur_dir}/nginx/html
nginx_log_path=${cur_dir}/nginx/log


function deploy_nginx() {

    docker rm -f ${nginx_container_name}

    if [ -d ${nginx_html_path} ]; then
        echo "[INFO] nginx 数据目录已存在 ${nginx_html_path}"
    else
        echo "[INFO] nginx 数据目录不存在, 新建数据目录 ${nginx_html_path}"
        mkdir -p ${nginx_html_path}
        mkdir -p ${nginx_log_path}
    fi

    echo "[INFO] 启动 ${nginx_container_name} ..."
    docker run \
        --name ${nginx_container_name} \
        -d \
        -v ${nginx_html_path}:/usr/share/nginx/html \
	-v ${nginx_log_path}:/var/log/nginx \
        -p ${nginx_port}:${nginx_port_in_docker} \
        ${nginx_docker_image}

    echo "[INFO] 启动 ${nginx_container_name} 成功 !"
}


deploy_nginx

echo [INFO] 脚本执行完成
exit 0


```

### 2.2 docker 部署 database
```bash
#!/bin/bash

# set -x

cur_dir=$(
    cd "$(dirname "$0")"
    pwd
)

# 数据库目录结构
# ${cur_dir}/
#       ├── mysql5/ 
#           ├── conf
#           ├── data
#       ├── mysql8/
#           ├── conf
#           ├── data
#       ├── neo4j/
#           ├── conf
#           ├── data
#       ├── mongo3/
#           ├── conf
#           ├── data
#       ├── mongo4/
#           ├── conf
#           ├── data
#       ├── mongo4_auth/
#           ├── conf
#           ├── data
#       └── redis/
#           ├── conf
#           └── data
#       └── gbase8a/
#           ├── conf
#           └── data
#       └── redis/
#           ├── conf
#           └── data
#       └── opengauss_5432/
#           ├── conf
#           └── data

# 初始化设置
# mysql5配置
mysql5_docker_image=nexus.ahi.internal:5000/mysql:5.6
mysql5_container_name=ahi_mysql5_3306
mysql5_port_in_docker=3306
mysql5_port=3306
mysql5_conf_path=${cur_dir}/mysql5/conf
mysql5_data_path=${cur_dir}/mysql5/data
mysql5_init_password=AHIbackend_2019

function deploy_mysql5() {

    if [ -d ${mysql5_data_path} ]; then
        echo "[INFO] mysql5 数据目录已存在 ${mysql5_data_path}"
    else
        echo "[INFO] mysql5 数据目录不存在, 新建数据目录 ${mysql5_data_path}"
        mkdir -p ${mysql5_data_path}
    fi

    echo "[INFO] 启动 ${mysql5_container_name} ..."

    docker run \
        -d \
        --name ${mysql5_container_name} \
        -p ${mysql5_port}:${mysql5_port_in_docker} \
        -e MYSQL_ROOT_PASSWORD=${mysql5_init_password} \
        -e MYSQL_DATABASE=ahi_ml_platform \
        -v ${mysql5_data_path}:/var/lib/mysql \
        ${mysql5_docker_image}

    echo "[INFO] 启动 ${mysql5_container_name} 成功 !"
}

# mysql5.7配置
mysql57_docker_image=nexus.ahi.internal:5000/mysql:5.7
mysql57_container_name=ahi_mysql57_3307
mysql57_port_in_docker=3306
mysql57_port=3307
mysql57_conf_path=${cur_dir}/mysql57/conf
mysql57_data_path=${cur_dir}/mysql57/data
mysql57_init_password=AHIbackend_2019

function deploy_mysql57() {

    if [ -d ${mysql57_data_path} ]; then
        echo "[INFO] mysql57 数据目录已存在 ${mysql57_data_path}"
    else
        echo "[INFO] mysql57 数据目录不存在, 新建数据目录 ${mysql57_data_path}"
        mkdir -p ${mysql57_data_path}
    fi

    echo "[INFO] 启动 ${mysql57_container_name} ..."

    docker run \
        -d \
        --name ${mysql57_container_name} \
        -p ${mysql57_port}:${mysql57_port_in_docker} \
        -e MYSQL_ROOT_PASSWORD=${mysql57_init_password} \
        -e MYSQL_DATABASE=ahi_ml_platform \
        -v ${mysql57_data_path}:/var/lib/mysql \
        ${mysql57_docker_image}

    echo "[INFO] 启动 ${mysql57_container_name} 成功 !"
}

# mysql8配置
mysql8_docker_image=mysql:8
mysql8_container_name=ahi_mysql8_3308
mysql8_port_in_docker=3306
mysql8_port=3308
mysql8_conf_path=${cur_dir}/mysql8/conf
mysql8_data_path=${cur_dir}/mysql8/data
mysql8_init_password=AHIbackend_2019

# mysql8 初始化
# ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'AHIbackend_2019';
# grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;	
# flush privileges;

function deploy_mysql8() {

    docker rm -f ${mysql8_container_name}

    if [ -d ${mysql8_data_path} ]; then
        echo "[INFO] mysql8 数据目录已存在 ${mysql8_data_path}"
    else
        echo "[INFO] mysql8 数据目录不存在, 新建数据目录 ${mysql8_data_path}"
        mkdir -p ${mysql8_data_path}
    fi

    echo "[INFO] 启动 ${mysql8_container_name} ..."

    docker run \
        -d \
        --name ${mysql8_container_name} \
        -p ${mysql8_port}:${mysql8_port_in_docker} \
        -e MYSQL_ROOT_PASSWORD=${mysql8_init_password} \
        -v ${mysql8_data_path}:/var/lib/mysql \
        ${mysql8_docker_image}

    echo "[INFO] 启动 ${mysql8_container_name} 成功 !"
}

        #${mysql8_docker_image} --default-authentication-plugin=mysql_native_password

# neo4j配置
neo4j_docker_image=nexus.ahi.internal:5000/neo4j
neo4j_container_name=ahi_neo4j_7474
neo4j_port_in_docker=7474
neo4j_port=7474
neo4j_conf_path=${cur_dir}/neo4j/conf
neo4j_data_path=${cur_dir}/neo4j/data

function deploy_neo4j() {

    if [ -d ${neo4j_data_path} ]; then
        echo "[INFO] neo4j 数据目录已存在 ${neo4j_data_path}"
    else
        echo "[INFO] neo4j 数据目录不存在, 新建数据目录 ${neo4j_data_path}"
        mkdir -p ${neo4j_data_path}
    fi

    echo "[INFO] 启动 ${neo4j_container_name} ..."
    docker run \
        -d \
        --name ${neo4j_container_name} \
        -p ${neo4j_port}:${neo4j_port_in_docker} \
        -p 7687:7687 \
        --env=NEO4J_dbms_memory_pagecache_size=50G \
        --env=NEO4J_dbms_memory_heap_initial__size=16G \
        --env=NEO4J_dbms_memory_heap_max__size=16G \
        --env=NEO4J_AUTH=neo4j/ahi2017 \
        -v ${neo4j_data_path}:/data \
	-v ${performance_neo4j_import_path}:/var/lib/neo4j/import \
        ${neo4j_docker_image}

    echo "[INFO] 启动 ${neo4j_container_name} 成功 !"
}


# 性能neo4j配置
performance_neo4j_docker_image=nexus.ahi.internal:5000/neo4j
performance_neo4j_container_name=ahi_performance_neo4j_7475
performance_neo4j_port_in_docker=7474
performance_neo4j_port=7475
performance_neo4j_conf_path=${cur_dir}/neo4j/conf
performance_neo4j_data_path=${cur_dir}/neo4j/performance_data
performance_neo4j_import_path=${cur_dir}/neo4j/import_csv

function deploy_performance_neo4j() {

    if [ -d ${performance_neo4j_data_path} ]; then
        echo "[INFO] neo4j 数据目录已存在 ${performance_neo4j_data_path}"
    else
        echo "[INFO] neo4j 数据目录不存在, 新建数据目录 ${performance_neo4j_data_path}"
        mkdir -p ${performance_neo4j_data_path}
    fi

    echo "[INFO] 启动 ${performance_neo4j_container_name} ..."
    docker run \
        -d \
        --name ${performance_neo4j_container_name} \
        -p ${performance_neo4j_port}:${performance_neo4j_port_in_docker} \
        -p 7688:7687 \
        --env=NEO4J_dbms_memory_pagecache_size=32G \
        --env=NEO4J_dbms_memory_heap_initial__size=16G \
        --env=NEO4J_dbms_memory_heap_max__size=16G \
        --env=NEO4J_AUTH=neo4j/ahi2017 \
        -v ${performance_neo4j_data_path}:/data \
	-v ${performance_neo4j_import_path}:/var/lib/neo4j/import \
        ${performance_neo4j_docker_image}

    echo "[INFO] 启动 ${performance_neo4j_container_name} 成功 !"
}

# mongo3配置
mongo3_docker_image=nexus.ahi.internal:5000/mongo:3
mongo3_container_name=ahi_mongo_27017
mongo3_port_in_docker=27017
mongo3_port=27017
mongo3_conf_path=${cur_dir}/mongo3/conf
mongo3_data_path=${cur_dir}/mongo3/data

function deploy_mongo3() {

    if [ -d ${mongo3_data_path} ]; then
        echo "[INFO] mongo3 数据目录已存在 ${mongo3_data_path}"
    else
        echo "[INFO] mongo3 数据目录不存在, 新建数据目录 ${mongo3_data_path}"
        mkdir -p ${mongo3_data_path}
    fi

    echo "[INFO] 启动 ${mongo3_container_name} ..."


    docker run \
        -d \
        --name ${mongo3_container_name} \
        -m 4096M \
        --memory-swap=4096M \
        -p ${mongo3_port}:${mongo3_port_in_docker} \
        -v ${mongo3_data_path}:/data/db \
        ${mongo3_docker_image} \
        mongod --bind_ip 0.0.0.0

    echo "[INFO] 启动 ${mongo3_container_name} 成功 !"
}

# mongo3_auth配置
mongo3_auth_docker_image=nexus.ahi.internal:5000/mongo:3
mongo3_auth_container_name=ahi_mongo_27018
mongo3_auth_port_in_docker=27017
mongo3_auth_port=27018
mongo3_auth_conf_path=${cur_dir}/mongo3_auth/conf
mongo3_auth_data_path=${cur_dir}/mongo3_auth/data

function deploy_mongo3_auth() {

    if [ -d ${mongo3_auth_data_path} ]; then
        echo "[INFO] mongo3_auth 数据目录已存在 ${mongo3_auth_data_path}"
    else
        echo "[INFO] mongo3_auth 数据目录不存在, 新建数据目录 ${mongo3_auth_data_path}"
        mkdir -p ${mongo3_auth_data_path}
    fi

    echo "[INFO] 启动 ${mongo3_auth_container_name} ..."


    docker run \
        -d \
        --name ${mongo3_auth_container_name} \
        -m 4096M \
        --memory-swap=4096M \
        -p ${mongo3_auth_port}:${mongo3_auth_port_in_docker} \
        -v ${mongo3_auth_data_path}:/data/db \
        -e MONGO_INITDB_ROOT_USERNAME=root \
        -e MONGO_INITDB_ROOT_PASSWORD=ahi2017 \
        ${mongo3_auth_docker_image} \
        mongod --bind_ip 0.0.0.0 \
        --auth

    echo "[INFO] 启动 ${mongo3_auth_container_name} 成功 !"
}

# mongo4配置
mongo4_docker_image=mongo:4.0.22
mongo4_container_name=ahi_mongo4_27019
mongo4_port_in_docker=27017
mongo4_port=27019
mongo4_conf_path=${cur_dir}/mongo4/conf
mongo4_data_path=${cur_dir}/mongo4/data

function deploy_mongo4() {

    if [ -d ${mongo4_data_path} ]; then
        echo "[INFO] mongo4 数据目录已存在 ${mongo4_data_path}"
    else
        echo "[INFO] mongo4 数据目录不存在, 新建数据目录 ${mongo4_data_path}"
        mkdir -p ${mongo4_data_path}
    fi

    echo "[INFO] 启动 ${mongo4_container_name} ..."


    docker run \
        -d \
        --name ${mongo4_container_name} \
        -m 4096M \
        --memory-swap=4096M \
        -p ${mongo4_port}:${mongo4_port_in_docker} \
        -v ${mongo4_data_path}:/data/db \
        ${mongo4_docker_image} \
        mongod --bind_ip 0.0.0.0

    echo "[INFO] 启动 ${mongo4_container_name} 成功 !"
}

# mongo4_auth配置
mongo4_auth_docker_image=mongo:4.0.22
mongo4_auth_container_name=ahi_mongo4_27020
mongo4_auth_port_in_docker=27017
mongo4_auth_port=27020
mongo4_auth_conf_path=${cur_dir}/mongo4_auth/conf
mongo4_auth_data_path=${cur_dir}/mongo4_auth/data
mongo4_initdb_root_username=mongo_admin
mongo4_initdb_root_password=AHIbackend_2017

function deploy_mongo4_auth() {

    # docker rm -f ${mongo4_auth_container_name}

    if [ -d ${mongo4_auth_data_path} ]; then
        echo "[INFO] mongo4_auth 数据目录已存在 ${mongo4_auth_data_path}"
    else
        echo "[INFO] mongo4_auth 数据目录不存在, 新建数据目录 ${mongo4_auth_data_path}"
        mkdir -p ${mongo4_auth_data_path}
    fi

    echo "[INFO] 启动 ${mongo4_auth_container_name} ..."


    docker run \
        -d \
        --name ${mongo4_auth_container_name} \
        -m 4096M \
        --memory-swap=4096M \
        -p ${mongo4_auth_port}:${mongo4_auth_port_in_docker} \
        -v ${mongo4_auth_data_path}:/data/db \
        -e MONGO_INITDB_ROOT_USERNAME=${mongo4_initdb_root_username} \
        -e MONGO_INITDB_ROOT_PASSWORD=${mongo4_initdb_root_password} \
        ${mongo4_auth_docker_image} \
        mongod \
        --auth

    echo "[INFO] 启动 ${mongo4_auth_container_name} 成功 !"
}

# redis配置
redis_docker_image=redis
redis_container_name=ahi_redis_6379
redis_port_in_docker=6379
redis_port=6379
redis_conf_path=${cur_dir}/redis/conf
redis_data_path=${cur_dir}/redis/data
redis_auth=ahi2017

function deploy_redis() {

    if [ -d ${redis_data_path} ]; then
        echo "[INFO] redis 数据目录已存在 ${redis_data_path}"
    else
        echo "[INFO] redis 数据目录不存在, 新建数据目录 ${redis_data_path}"
        mkdir -p ${redis_data_path}
    fi

    echo "[INFO] 启动 ${redis_container_name} ..."
    docker run \
        -d \
        --name ${redis_container_name} \
        -p ${redis_port}:${redis_port_in_docker} \
        -v ${redis_data_path}:/data \
        ${redis_docker_image} \
        --requirepass ${redis_auth} \
        --appendonly yes

    echo "[INFO] 启动 ${redis_container_name} 成功 !"
}

# gbase8a配置
gbase8a_docker_image=shihd/gbase8a:1.0
gbase8a_container_name=ahi_gbase8a_5258
gbase8a_port_in_docker=5258
gbase8a_port=5258
#gbase8a_conf_path=${cur_dir}/gbase8a/conf
gbase8a_data_path=${cur_dir}/gbase8a
        #-v ${gbase_data_path}:/home/gbase/GBase/userdata/gbase8a/gbase/ \
#gbase8a_user=root
#gbase8a_password=root
#gbase8a_db=gbase8a

function deploy_gbase8a() {

    if [ -d ${gbase8a_data_path} ]; then
        echo "[INFO] gbase8a 数据目录已存在 ${gbase8a_data_path}"
    else
        echo "[INFO] gbase8a 数据目录不存在, 新建数据目录 ${gbase8a_data_path}"
        mkdir -p ${gbase8a_data_path}
    fi

    echo "[INFO] 启动 ${gbase8a_container_name} ..."
    docker run \
        --name ${gbase8a_container_name} \
        -d \
        -e GS_PASSWORD=${gbase8a_password} \
        -p ${gbase8a_port}:${gbase8a_port_in_docker} \
        ${gbase8a_docker_image}

    echo "[INFO] 启动 ${gbase8a_container_name} 成功 !"
}

# opengauss101配置
opengauss101_docker_image=enmotech/opengauss:latest
opengauss101_container_name=ahi_opengauss101_15432
opengauss101_port_in_docker=5432
opengauss101_port=15432
#opengauss101_conf_path=${cur_dir}/opengauss101/conf
opengauss101_data_path=${cur_dir}/opengauss101
opengauss_password=ahi2017@Gauss

function deploy_opengauss101() {

    if [ -d ${opengauss101_data_path} ]; then
        echo "[INFO] opengauss101 数据目录已存在 ${opengauss101_data_path}"
    else
        echo "[INFO] opengauss101 数据目录不存在, 新建数据目录 ${opengauss101_data_path}"
        mkdir -p ${opengauss101_data_path}
    fi

    echo "[INFO] 启动 ${opengauss101_container_name} ..."
    # docker run --name opengauss --privileged=true -d -e GS_PASSWORD=ahi2017@Gauss -v /data/AHI/AutoDeploy/database/opengauss101:/var/lib/opengauss -p 15432:5432 enmotech/opengauss:latest
    docker run \
        --name ${opengauss101_container_name} \
        --privileged=true \
        -d \
        -e GS_PASSWORD=${opengauss_password} \
        -v ${opengauss101_data_path}:/var/lib/opengauss \
        -p ${opengauss101_port}:${opengauss101_port_in_docker} \
        ${opengauss101_docker_image} 

    echo "[INFO] 启动 ${opengauss101_container_name} 成功 !"
}

#deploy_mysql5
#deploy_mysql57
#deploy_mysql8
#deploy_neo4j
#deploy_performance_neo4j
#deploy_mongo3
#deploy_mongo3_auth
#deploy_mongo4
#deploy_mongo4_auth
deploy_redis
#deploy_gbase8a
#deploy_opengauss101
#deploy_mongo4_auth

echo [INFO] 脚本执行完成
exit 0


```

### 2.3 clean docker
```bash
#!/bin/bash

docker rm $(docker ps -q -f status=exited)

docker volume rm $(docker volume ls -qa dangling=true)

docker rmi $(docker images --filter "dangling=true" -q --no-trunc)

```

### 2.4 setup_nemo_jenkins.sh
```bash
#!/bin/bash

set -o errexit
set -o nounset

export LANG="en_US.UTF-8"
export LC_ALL="en_US.UTF-8"

function parse_args(){
    if [ $# != 2 ];then
        echo Usage: $0 JENKINS_INSTALL_DIR TOMCAT_PORT
        echo " eg: $0 /u01 8080"
        exit 1
    else
        JENKINS_INSTALL_DIR=$1
        TOMCAT_PORT=$2
    fi
}

function check_already_installed(){
    if [ -d $JENKINS_INSTALL_DIR/nemo_jenkins ]; then
        echo [ERROR] nemo_jenkins already installed in $JENKINS_INSTALL_DIR, please remove it first.
        exit 1
    fi
}

function check_write_permission(){
    echo [INFO] Check write permission ...
    mkdir -p $JENKINS_INSTALL_DIR
    if [ ! -w $JENKINS_INSTALL_DIR ]; then
        echo "[ERROR] Current user $USER doesn't have write permission for $JENKINS_INSTALL_DIR"
        exit 1
    fi
}

# check PORT occupied or not
function check_port_occupied(){
    echo [INFO] Check port whether is occupied ...
    check_port=$(netstat -tnl | grep $1 | wc -l)
    if [ $check_port -gt 0 ]; then
        echo "[ERROR] Port $1 is occupied"
        exit 1
    fi
}

function setup_jenkins(){
    echo [INFO] Setup nemo_jenkins
    echo [INFO] Create nemo_jenkins in $JENKINS_INSTALL_DIR.
    mkdir -p $JENKINS_INSTALL_DIR/nemo_jenkins
    
    echo [INFO] Download install package from OSS ...
    storage_url=https://storage-den2.oraclecorp.com/v1/Storage-9ddbd8da2603418a915ad48698e2aa91jenkins_file=nemo_jenkins.zip
    cd $JENKINS_INSTALL_DIR
    curl -X GET -u 'terry.zhou@oracle.com:Nimbula1' -o $jenkins_file $storage_url/nemo/Framework/$jenkins_file

    echo [INFO] Install jenkins ...
    unzip -q nemo_jenkins.zip -d $JENKINS_INSTALL_DIR
    rm $jenkins_file
}

function config_tomcat_port(){
    echo [INFO] Config tomcat port ...
    TOMCAT_CONFIG_SERVER_XML=$JENKINS_INSTALL_DIR/nemo_jenkins/apache-tomcat/conf/server.xml
    sed -i 's#Connector port="8080" protocol="HTTP/1.1"#Connector 
    port="'$TOMCAT_PORT'" protocol="HTTP/1.1"#g' $TOMCAT_CONFIG_SERVER_XML
    
    # test
    TOMCAT_CURRENT_PORT=`cat $TOMCAT_CONFIG_SERVER_XML | sed -n "/Connector port/p" | sed -n '1p' | cut -d '"' -f 2`
    echo [DEBUG] Check tomcat port: $TOMCAT_CURRENT_PORT
}

function config_jenkinsUrl(){
    # auto modify .jenkins/jenkins.model.JenkinsLocationConfiguration.xml to set jenkinsUrl
    echo [INFO] Config jenkinsUrl ...
    cd $JENKINS_INSTALL_DIR/nemo_jenkins/.jenkins/
    sed -i 's#localhost#'$HOSTNAME'#g' jenkins.model.JenkinsLocationConfiguration.xml
    sed -i 's#PORT#'$TOMCAT_PORT'#g' jenkins.model.JenkinsLocationConfiguration.xml
}

function config_jenkins(){
    config_tomcat_port
    config_jenkinsUrl
}

function create_control_script(){
    # auto modify jenkins_server.sh to set JENKINS_DIR=$JENKINS_INSTALL_DIR/nemo_jenkins/
    sed -i 's#replace_here#export JENKINS_DIR='$JENKINS_INSTALL_DIR'/nemo_jenkins#g' $JENKINS_INSTALL_DIR/nemo_jenkins/jenkins_server.sh
}

function start_jenkins(){
    sleep 1
    echo [INFO] Start jenkins ...
    export JENKINS_HOME=$JENKINS_INSTALL_DIR/nemo_jenkins/.jenkins
    sleep 1
    $JENKINS_INSTALL_DIR/nemo_jenkins/jenkins_server.sh start
}

# main
parse_args $@
echo [INFO] Set JENKINS_INSTALL_DIR to $JENKINS_INSTALL_DIR
echo [INFO] Set TOMCAT_PORT to $TOMCAT_PORT
check_already_installed
check_write_permission
check_port_occupied $TOMCAT_PORT
setup_jenkins
config_jenkins
create_control_script
start_jenkins
# -----------------------------------------------
```