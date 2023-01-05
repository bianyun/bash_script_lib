# Bash Script Lib

## Introduction

Bash script lib is a set of modular and general-purpose Bash script library, which is used to solve the problem that scattered bash script functions are not easy to reuse. This script library automatically
imports the scripts under each module (the module directory level is not limited), and also considers the unique script commands of different Linux distributions (only imports the distribution-specific scripts
that match the current system).

## Complete usage example

<code>docker-install-mysql/bin/install-mysql-5.7.sh</code>
```bash
#!/usr/bin/env bash

SCRIPT_PATH="${0}"
while [ -h "${SCRIPT_PATH}" ]; do
  LS=`ls -ld "${SCRIPT_PATH}"`
  LINK=`expr "${LS}" : '.*-> \(.*\)$'`
  if [ `expr "${LINK}" : '/.*'` > /dev/null ]; then
    SCRIPT_PATH="${LINK}"
  else
    SCRIPT_PATH="`dirname "${SCRIPT_PATH}"`/${LINK}"
  fi
done
cd `dirname "${SCRIPT_PATH}"` > /dev/null
SCRIPT_DIR=`pwd`


#=================================================================================
# SCRIPT LEVEL CONFIG
#=================================================================================
BSL_SCRIPT_LIB_VERSION="1.0"
BSL_SCRIPT_LIB_DIR="$(realpath ${SCRIPT_DIR}/../../bash_script_lib_v${BSL_SCRIPT_LIB_VERSION})"
source $BSL_SCRIPT_LIB_DIR/init_script || exit 1
DEBUG_ENABLED="false"
DEPLOYED_AS_CLUSTER="false"


#=================================================================================
# customized config 
#=================================================================================
app_name="mysql"
image_name="mysql"
image_version="5.7.40"
mysql_port=13306
mysql_init_root_password="password_mysql"


#=================================================================================
# os and software version requirements, commands requirements
#=================================================================================
required_commands="docker"
# required_min_os_version="centos_7.2, ubuntu_18.04"


#=================================================================================
# parse internal config
#=================================================================================
parse_internal_config() {
  container_name=$(get_docker_container_name $image_name $image_version)
  container_local_conf_dir="$(get_container_local_conf_dir)"
  container_local_data_dir="$(get_container_local_data_dir)"
}


#=================================================================================
# docker deployment configs
#=================================================================================
setup_docker_deployment_configs() {
  container_environment_config=(
    "TZ=Asia/Shanghai",
    "MYSQL_ROOT_PASSWORD=${mysql_init_root_password}",
  )
  
  container_volume_mappings=(
    "${container_local_conf_dir}/mysql.cnf:/etc/mysql/conf.d/mysql.cnf",
    "${container_local_conf_dir}/mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf",
    "${container_local_data_dir}:/var/lib/mysql",
  )
  
  container_port_mappings=(
    "${mysql_port}:3306",
  )
  
  container_sysctls_config=(
    # "sysctl_config_key1=sysctl_config_value1",
    # "sysctl_config_key2=sysctl_config_value2",
  )
  
  container_command_config=""
}


#=================================================================================
# customized operation functions
#=================================================================================
setup_sysctl_configs() {
  do_nothing
}


#=================================================================================
# Step 0: init script
#=================================================================================
_do_init_script


#=================================================================================
# Step 1: prepare before install
#=================================================================================
step_1_prepare_before_install() {
  parse_internal_config
  setup_docker_deployment_configs
  setup_sysctl_configs
  
  check_if_file_exists "${container_local_conf_dir}/mysql.cnf"
  check_if_file_exists "${container_local_conf_dir}/mysqld.cnf"
  mkdir -p "${container_local_data_dir}"
}


#=================================================================================
# Step 2: do install app
#=================================================================================
step_2_do_install_app() {
  start_docker_container $app_name $image_name $image_version
}


#=================================================================================
# Step 3: post process after install
#=================================================================================
step_3_post_process_after_install() {
  generate_view_logs_shell_script $app_name $container_name
  println_script_total_time_desc_and_exit
}


#=================================================================================
# MAIN ENTRY POINT
#=================================================================================
if $(is_deployed_as_cluster); then
  [ $# -ne 1 ] && \
    println_one_line_usage "$(basename $0) <cluster_node_id>" && \
    exitscript
  cluster_node_id=$1
fi

step_1_prepare_before_install

step_2_do_install_app

step_3_post_process_after_install
```

## Function call parameter checking
```text
sheldon@vm-ubuntu22:/opt/app-deploy/docker-install-mysql/bin$ sudo ./install-mysql-5.7.sh
[  ERROR] Failed to call function get_docker_container_name(image_name, image_version).
          Cause: Invalid args num, expected: [2], actual: [1].
          [Stack Trace]:
             get_docker_container_name @ /modules/app/docker/container:25
             parse_internal_config @ ./install-mysql-5.7.sh:49
             step_1_prepare_before_install @ ./install-mysql-5.7.sh:101
             main @ ./install-mysql-5.7.sh:138
```

## Debug-level log enabled
```text
sheldon@vm-ubuntu22:/opt/app-deploy/docker-install-mysql/bin$ sudo ./install-mysql-5.7.sh
[  DEBUG] =========================
[  DEBUG] Resolved Script Variables
[  DEBUG] =========================
[  DEBUG] BSL_DEBUG = [true]
[  DEBUG] BSL_SCRIPT_LIB_VERSION = [1.0]
[  DEBUG] BSL_REQUIRED_COMMANDS = [apt-get cut docker dpkg grep read sort tr wc]
[  DEBUG] BSL_INSTALL_FILES_DIR = [/opt/app-deploy/docker-install-mysql/install_files]
[  DEBUG] BSL_CONTAINER_VOLUMES_LOCAL_DIR = [/opt/app-deploy/docker-install-mysql/volumes_local]

[ NOTICE] Starting docker container [mysql-5.7.40]...
[   INFO] Resolving mysql image file... [OK]
[  DEBUG] Resolved mysql image file:
          [/opt/app-deploy/docker-install-mysql/install_files/mysql-5.7.40-image-official.tar.gz]
[  DEBUG] Resolving image sha256 from image file [mysql-5.7.40-image-official.tar.gz]...
[  DEBUG] Docker image exists and its sha-256 is same as image file, no need to load image.
[+] Running 2/2
 ⠿ Network mysql-default   Created 0.1s
 ⠿ Container mysql-5.7.40  Started 1.3ss
[SUCCESS] Docker container [mysql-5.7.40] has been started successfully.
[   TIME] This script starts at [2023-01-05 15:22:05], ends at [2023-01-05 15:22:12].
          Total time consumption: 7s (00:00:07).
```
