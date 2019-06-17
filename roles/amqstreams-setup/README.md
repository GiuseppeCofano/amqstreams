## About

This Ansible role installs Amqstreams (Kafka) from .zip (or other archive format, like tar), which has to be made available on an URL (by default the Red Hat Network is used). 

Both Zookeeper and Kafka are installed, integrated or independently from each other.

Other configurations carried out by this role:
- systemd setup
- config to launch Amqstreams
- Amqstreams tuning and advanced settings (separate replication network if desired)
- configure a Kafka dedicate disk if desired
- instal Jolokia agent


## Requirements

This role requires a RHEL Linux distribution. It might work with other software versions, 
but is tested with the following specifics and versions:

* Ansible: 2.5+
* RHEL: 7
* Amqstreams: 1.1.0

It requires Jdk already installed on the system


## **Role Variables**

The following variables can be used in the `hosts` inventory file:


| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `mode` | integrate | The action to be executed, possible values: <only_kafka|only_zookeeper|integrate> |
| `amqstreams_force_reinstall` | False | Whether to force the reinstallation over an existing one |
| `amqstreams_user` | kafka | Amqstreams user |
| `amqstreams_group` | kafka | Amqstreams group |
| `kafka_port` | 9092 | listening port of the Kafka service |
| `zookeeper_port` | 2181 | listening port of the Zookeeper service |
| `amqstreams_disk` | /dev/sdb | Disk to use for Kafka |
| `zookeeper_data_path` | /data/kafka/amqs_zookeeper | Zookeeper data path |
| `zookeeper_logs_dir` | /data/kafka/logs_zookeeper | Zookeeper logs path |
| `kafka_data_path` | /data/kafka/amqs_kafka | Kafka data path |
| `kafka_logs_dir` | /data/kafka/logs_kafka | Kafka logs path |
| `amqstreams_enable_tls` | False | Enable intracluster TLS communication |
| `enable_disks_setup` | True | Set to False if not using a dedicated disk (as best practice) |
| `jolokia_enable` | False | Enable installation of the Jolokia agent |
| `replication_network_enable` | False | Enable separate replication network |
| `amqstreams_install_path` | /data/kafka | Target installation path |
| `amqstreams_install_file` | amq-streams-1.1.0-bin.zip | Installation file (change the default value if renaming the Red Hat file) |
| `amqstreams_java_version` | 1.8 | Java version for the pre-check carried out by the role, must be already present on the host |
| `amqstreams_java_home` | /opt/java | Java Home path to be used|
| `amqstreams_downloadURL` | https://access.redhat.com/jbossnetwork/restricted/softwareDownload.html?softwareId=66601 | URL from where the Amqstreams .zip file can be downloaded |
| `amqstreams_nofile` | 2048 | nofiles for Amqstreams |
| `amqstreams_max_heap` | 1G | max heap |
| `amqstreams_min_heap` | 1G | min heap |
| `kafka_replication_ip` | 1G | if enabled, separate IP for the replication network |
| `kafka_replication_port` | 9093 | if enabled, separate port for the replication network |


## Usage

To install Amqstreams (Kafka and Zookeeper on the same hosts, useful for DEV environments):

```yml
roles:
  - role: amqstreams-setup
```

To install only Kafka (forcing reinstallation over an existing one):

```yml
roles:
  - role: amqstreams-setup
    mode: only_kafka
    amqstreams_force_reinstall: True
```

To install Amqstreams in PROD (separate host groups for Kafka and Zookeeper):

```yml
- hosts: kafka
  roles:
  - role: amqstreams-setup
    mode: only_kafka
    amqstreams_force_reinstall: True

- hosts: zookeeper
  roles:
  - role: amqstreams-setup
    mode: only_zookeeper
    amqstreams_force_reinstall: True
```

Notes: 

None



