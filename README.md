[![Build Status](https://circleci.com/gh/darien-hirotsu/ansible_idempotence.svg?style=shield)](https://app.circleci.com/pipelines/github/darien-hirotsu/ansible_idempotence)

# Ansible Idempotence Demo

## Overview
This repository illustrates how to use Ansible in an idempotent fashion. Ansible module code must be written to ensure task idempotence. If module code does not enforce idempotence, Ansible conditionals must be used to build idempotent playbooks.

## Setup
The examples shown assume you have an AWS account and your AWS API keys are exported as environment variables prior to running the playbooks.

```
export AWS_SECRET_ACCESS_KEY=<YOUR-KEY>
export AWS_ACCESS_KEY_ID=<YOUR-KEY-ID>
```

In your execution environment, you must have the `pip` utility and appropriate permissions to install the `requirements.txt` file and the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

## Non-idempotent ec2
To illustrate Ansible's reliance on module implementation for idempotence, run the `main_ec2.yml` playbook. It uses a role called `ec2` to create a single virtual-machine in AWS.

```
ansible-playbook main_ec2.yml
```

Every run of `ansible-playbook main_ec2.yml` results in the creating of an ec2 virtual-machine. The output from `ansible-playbook` always matches the below since the `ec2` module is not idempotent.

```
ansible-playbook main_ec2.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] **************************************************************************************************************************

TASK [ec2 : build web01 instance] *********************************************************************************************************
changed: [localhost]

PLAY RECAP ********************************************************************************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

## Idempotent ec2
Ansible conditionals ensure playbook idempotence. Idempotence helps make the deployment pipeline more robust as playbooks may be re-run without additional operator intervention. The `main_ec2_idempotent.yml` playbook leverages a role called `ec2_set_fact` to query for the ec2 instance based on hostname using the AWS CLI. The role sets a fact to conditionally import the `ec2` role. The `ec2` role is only included when the instance does not exist.

```
ansible-playbook main_ec2_idempotent.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] **************************************************************************************************************************

TASK [ec2_set_fact : query for existing instance] *****************************************************************************************
changed: [localhost]

TASK [ec2_set_fact : set ec2 fact] ********************************************************************************************************
ok: [localhost]

TASK [include_role : ec2] *****************************************************************************************************************
skipping: [localhost]

PLAY RECAP ********************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```
