### About:
This ansible script installs [Passbolt](https://www.passbolt.com/) and [MariaDB](https://mariadb.org/) container on Ubuntu.<br/>

### Supported OS:
* Ubuntu 20.04

### Prerequisites
* [Ansible](https://docs.ansible.com/ansible/latest/index.html)
* [jmespath plugin](https://pypi.org/project/jmespath/)
* [Azure blob](https://docs.microsoft.com/en-us/cli/azure/storage/blob?view=azure-cli-latest#az_storage_blob_upload)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/)
* [AWS S3](https://aws.amazon.com/s3/)
* [Linode Object Storage](https://www.linode.com/docs/guides/platform/object-storage/)
* [s3cmd](https://www.linode.com/docs/guides/how-to-use-object-storage/#s3cmd)
* [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
* [community.docker](https://galaxy.ansible.com/community/docker)

### Configuration

#### Azure Blob Storage
[Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) should be installed on the host where Passbolt and MariaDB is installed and Azure Blob Storage should be created on Azure.

It is a possibility to upload backup to all cloud providers at one time, to do that those programs: **azure CLI**, **s3cmd**, **boto3** must be installed on the host where Passbolt and MariaDB is installed.

> **_NOTE:_** S3CMD and boto3 are installed during installation process.

#### Firewall
On host where Passbolt and MariaDB will be installed, ssh port should be enabled.

#### Inventory.yml
In `inventory.yml`, set **IP**, **user**, **password**, **ssh port** or **ssh_key** on where Passbolt and MariaDB should be installed.</br>
If **ssh_key** is used, comment **password**.</br>
If **password** is used, comment **ssh_key**.</br>
```yml
linux:
   vars:
      ansible_ssh_user: username
      ansible_ssh_pass: password
      ansible_port: 22
      ansible_ssh_private_key_file: <path_to_key>
   children:
      mariadb:
         hosts:
            127.0.0.1:
      passbolt:
         hosts:
            127.0.0.1:
```

#### Inputs.yml
In `inputs.yml`, set:
```yml
---
# Select deployment type: greenfield / brownfied.
deployment: greenfield

# Select if Passbolt and MariaDB are installed on one host or seperate.
aio: 0
```

#### Group_vars/all/common
In `group_vars/all/common`, set:

```txt
_time_zone: Europe/Warsaw           => Set Time Zone.
_mariadb: 1                         => Install MariaDB.
_passbolt: 1                        => Install Passbolt.
_zip_password: {password}           => Set password for .zip files.
_docker_compose_version: 1.29.2     => Install docker-compose.
_restore_from_backup:               => Restore Passbolt and MariaDB from backup during greenfield installation.
   azure: 0                         => Restore from Azure. 1 - yes, 0 - no.
   linode: 0                        => Restore from Linode. 1 - yes, 0 - no.
   aws: 0                           => Restore from AWS. 1 - yes, 0 - no.
_azure_upload: 0                    => Upload to Azure Blob Storage. 1 - yes, 0 - no.
_container_name: {containerName}    => Set Azure Blob Storage container name.
_account_name: {accountName}        => Set Azure Blob Storage account name.
_account_key: {accountKey}          => Set Azure Blob Storage account key.
_linode_upload: 0                   => Upload to Linode Ojbect Storage. 1 - yes, 0 - no.
_linode_bucket: {bucketName}        => Linode Object Storage name.
_linode_access_key: {accessKey}     => Linode Object Storage access key.
_linode_secret_key: {secretKey}     => Linode Object Storage secret key.
_host: {regionName}                 => Linode Object Storage region.
_aws_upload: 0                      => Upload to AWS S3. 1 - yes, 0 - no.
_aws_bucket: {bucketName}           => AWS S3 Bucket name.
_aws_access_key: {accessKey}        => AWS access key.
_aws_secret_key: {secretKey}        => AWS secret key.
```

#### Restore from backup
To restore from backup, set 1 in variable `azure`, `linode` or `aws` to choose from where the backup should be downloaded.</br>
If `azure` is set, enter proper values to the `_container_name`, `_account_name` and `_account_key`.</br>
If `linode` is set, enter proper values to the `_linode_bucket`.</br>
If `aws` is set, enter proper values to the `_aws_bucket`, `_aws_access_key` and `_aws_secret_key`.</br>
Setting 1 into variables: `azure`, `linode` and `aws` at the same time will fail the process of installation.

#### Group_vars/all/mariadb
In `group_vars/all/mariadb`, set:

```txt
_mariadb_net: mariadb_network             => Set MariaDB docker network.
_mariadb_name: mariadb                    => Set MariaDB container and host name.
_mariadb_version: 10.7.1                  => Set MariaDB version.
_mariadb_restore_version: 10.6.5          => Restore MariaDB to given version when brownfield failed.
_mariadb_root_password: {root_password}   => Set MariaDB root password.
_mariadb_password: {password}             => Set MariaDB user password.
_mariadb_username: passbolt               => Set MariaDB user name.
_mariadb_database: passbolt               => Set MariaDB database.
_mariadb_port: 3306                       => MariaDB port.
```

#### Group_vars/all/passbolt
In `group_vars/all/passbolt`, set:

```txt
_passbolt_net: passbolt_network                             => Set Passbolt docker network.
_passbolt_name: passbolt                                    => Set Passbolt container and host name.
_passbolt_version: 3.3.1-ce                                 => Set Passbolt version.
_passbolt_restore_version: 3.2.1-2-ce                       => Restore Passbolt to given version when brownfield faild.
_app_full_base_url: https://127.0.0.1                       => Passbolt base url.
_email_default_from: somemail@outlook.com                   => From email address.
_email_transport_default_host: smtp.office365.com           => Server hostname.
_email_transport_default_port: "587"                        => Server port.
_email_transport_default_tls: "true"                        => Set tls.
_email_transport_default_username: somemail@outlook.com     => Username for email server auth.
_email_transport_default_password: {password}               => Password for email server auth.
_passbolt_key_name: www-data                                => Key owner name
_passbolt_key_email: somemail@outlook.com                   => Ke owner email address.
```

More environment variables can be found [here](https://help.passbolt.com/configure/environment/reference.html)

> **_NOTE:_** _mariadb_root_password, _mariadb_password, _mariadb_username, _mariadb_database and _mariadb_port should be set in brownfield deployment!

### How to run:

```bash
ansible-playbook -i inventory.yml install.yml -e "@inputs.yml" --ask-become-pass -vv
```
