# Example of using Flink managed by Ansible

Just a quick example of creating a HA enabled Flink Cluster using Zookeeper.

This was created for a particular task, and is **not** recommended for prod

## How to build/run

Navigate to ./build, and run the Ansible playbook here to build the Docker containers that will run JobManager/TaskManager

`$ ansible-playbook build.yml`


```bash
PLAY [localhost] *****************************************************************************************************************************************************

TASK [Build image] ***************************************************************************************************************************************************
ok: [localhost]

TASK [Create a network] **********************************************************************************************************************************************
ok: [localhost]

TASK [Create a volume] **********************************************************************************************************************************
changed: [localhost]

TASK [Launch Flink Containers] ***************************************************************************************************************************************
changed: [localhost] => (item={'name': 'jobmanager', 'ports': ['2222:22', '6123', '8081:8081']})
changed: [localhost] => (item={'name': 'taskmanager-1', 'ports': ['2223:22', '6123']})
changed: [localhost] => (item={'name': 'taskmanager-2', 'ports': ['2224:22', '6123']})

TASK [Launch Zookeeper Containers] ***********************************************************************************************************************************
changed: [localhost] => (item={'name': 'zookeeper-1', 'ports': ['2181:2181']})
changed: [localhost] => (item={'name': 'zookeeper-2', 'ports': ['2182:2181']})
changed: [localhost] => (item={'name': 'zookeeper-3', 'ports': ['2183:2181']})

PLAY RECAP ***********************************************************************************************************************************************************
localhost                  : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Copy the private key locally to allow you to ssh onto the instance (or allow ansible to).

`$ docker cp jobmanager:/root/.ssh/id_rsa ~/.ssh/id_rsa`

```bash
Successfully copied 4.608kB to ~/.ssh/id_rsa
```

Access the container(s)

`$ docker exec -it $(docker ps --filter name=jobmanager --format={{.ID}}) /bin/bash`

`$ docker exec -it $(docker ps --filter name=taskmanager-1 --format={{.ID}}) /bin/bash`

`$ docker exec -it $(docker ps --filter name=taskmanager-2 --format={{.ID}}) /bin/bash`

`$ docker exec -it $(docker ps --filter name=zookeeper-1 --format={{.ID}}) /bin/bash`

`$ docker exec -it $(docker ps --filter name=zookeeper-2 --format={{.ID}}) /bin/bash`

`$ docker exec -it $(docker ps --filter name=zookeeper-3 --format={{.ID}}) /bin/bash`

Kill the cluster

`$ docker stop $( docker ps -aq ) && docker rm $( docker ps -aq )`

## How to install Flink

Navigate to ./flink, and run the Ansible playbook here to install Flink on the relevant containers.

`$ ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i hosts.ini playbook.yml`

```bashc
PLAY [jobmanager,taskmanager] ****************************************************************************************************************************************

TASK [ansible_role_flink : Install Java 8] ***************************************************************************************************************************
changed: [taskmanager_2]
changed: [taskmanager_1]
changed: [jobmanager]

TASK [ansible_role_flink : Create directory] *************************************************************************************************************************
changed: [taskmanager_1]
changed: [jobmanager]
changed: [taskmanager_2]

TASK [ansible_role_flink : Create directory] *************************************************************************************************************************
changed: [taskmanager_1]
changed: [taskmanager_2]
changed: [jobmanager]

TASK [ansible_role_flink : Download and extract] *********************************************************************************************************************
changed: [taskmanager_1]
changed: [taskmanager_2]
changed: [jobmanager]

TASK [ansible_role_flink : Apply configuration] **********************************************************************************************************************
changed: [taskmanager_2] => (item=flink-conf.yaml)
changed: [jobmanager] => (item=flink-conf.yaml)
changed: [taskmanager_1] => (item=flink-conf.yaml)

TASK [ansible_role_flink : Start master] *****************************************************************************************************************************
skipping: [taskmanager_1]
skipping: [taskmanager_2]
included: ansible_flink/flink/roles/ansible_role_flink/tasks/systemd_jobmanager.yml for jobmanager

TASK [ansible_role_flink : Install systemd unit] *********************************************************************************************************************
changed: [jobmanager]

TASK [ansible_role_flink : Ensure JobManager RPC address is set] *****************************************************************************************************
changed: [jobmanager]

TASK [ansible_role_flink : Ensure JobManager host address is set] ****************************************************************************************************
changed: [jobmanager]

TASK [ansible_role_flink : Ensure JobManager RPC address is set] *****************************************************************************************************
changed: [jobmanager]

TASK [ansible_role_flink : Ensure Blob Server port is set] ***********************************************************************************************************
changed: [jobmanager]

TASK [ansible_role_flink : Ensure Query Server port is set] **********************************************************************************************************
changed: [jobmanager]

TASK [ansible_role_flink : Launch Flink JobManager] ******************************************************************************************************************
changed: [jobmanager]

TASK [ansible_role_flink : Start slaves] *****************************************************************************************************************************
skipping: [jobmanager]
included: ansible_flink/flink/roles/ansible_role_flink/tasks/systemd_taskmanager.yml for taskmanager_1, taskmanager_2

TASK [ansible_role_flink : Install systemd unit] *********************************************************************************************************************
changed: [taskmanager_2]
changed: [taskmanager_1]

TASK [ansible_role_flink : Ensure JobManager RPC address is set] *****************************************************************************************************
changed: [taskmanager_1]
changed: [taskmanager_2]

TASK [ansible_role_flink : Ensure number of TaskSlots is set] ********************************************************************************************************
changed: [taskmanager_2]
changed: [taskmanager_1]

TASK [ansible_role_flink : Launch Flink TaskManager] *****************************************************************************************************************
changed: [taskmanager_1]
changed: [taskmanager_2]

PLAY RECAP ***********************************************************************************************************************************************************
jobmanager                 : ok=13   changed=12   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
taskmanager_1              : ok=10   changed=9    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
taskmanager_2              : ok=10   changed=9    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

### Access Web UI

When the cluster is running, you can visit the web UI at http://localhost:8081.

## Notes

This will not be maintained/improved on.

Inspiration taken from [NikosGavalas/ansible-flink](https://github.com/NikosGavalas/ansible-flink)
