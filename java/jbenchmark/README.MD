# AWS Storage RocksDB benchmarking

Original benchmarks are described [here](https://rocksdb.org.cn/doc/RocksJava-Performance-on-Flash-Storage.html).
These benchmarks use provided by rocksdb `rocksdb/java/jdb_bench.sh` utility.

## Setup in AWS

- create AWS account, web console user name and password (ask administrator)

- [install](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and 
[setup](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration) aws cli 
(might require creating cli access key/secret)

- create [pem file](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) for ssh access
and download it into rocksdb/java/jbenchmark/rocksdb_test.pem

- install [sshfs](https://github.com/osxfuse/osxfuse/wiki/SSHFS) if you want to mount ec2 home dir locally

- create iops provisioned [ebs volume](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html) or manage [in web console](https://console.aws.amazon.com/ec2/v2/home#Volumes:sort=desc:createTime)
```bash
make create_volume
make list_volumes
```

- create [efs](https://aws.amazon.com/getting-started/tutorials/create-network-file-system) 
and check its network [security rule](https://docs.aws.amazon.com/efs/latest/ug/accessing-fs-create-security-groups.html) (step 3b)

- create an instance template or manage [in web console](https://console.aws.amazon.com/ec2/v2/home#LaunchTemplates:sort=launchTemplateId)
```bash
make create_template list_templates
```

- spin up one or more test instances from template or manage [in web console](https://console.aws.amazon.com/ec2/v2/home#Instances:sort=statusChecks)
```bash
make run_from_template id=<template_id>
make list_instances
```

- attach ebs volume to instance
```bash
make list_instances
make list_volumes
make attach_volume id=<volume_id> iid=<instance_id>
```

- ssh into instance
```bash
make list_instances
make ssh ip=<instance_ip>
```

## Setup benchmark

### After instance launch

- clone repo
```bash
git clone https://github.com/azagrebin/rocksdb.git
cd rocksdb/java/jbenchmark
git checkout ebs
```

- build benchmark
```bash
make
```

- set correct instance internal store device name in rocksdb/java/jbenchmark/instance_startup.sh
```bash
df -h
```

- set correct efs id in rocksdb/java/jbenchmark/instance_startup.sh

- format ebs if its first use (might also need correct device name if there are more of them)
```bash
sudo mkfs -t ext4 /dev/sdf
```

- continue to `After instance restart` section

### After instance restart

```bash
cd <rocksdb_path>/rocksdb/java/jbenchmark
./instance_startup.sh
```