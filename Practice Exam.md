```markdown
# RHCE Sample Exam

**NOTE:** This sample exam assumes a lab environment with a control node and 3 managed hosts (but you can use more if you’d like) and a user with sudo privileges that will run all of the tasks present on all nodes. In addition, the default user should be able to ssh into all managed nodes without password prompting (so generate (ssh-keygen) and share (ssh-copy-id) its key with the hosts). If any of this is absent, create/enable this BEFORE proceeding to the exam.

---

### Task 1: Ansible Installation and Configuration

- Login as `ansible_user` with the password `password`. Use this user for all sample exam tasks.
- Ensure Ansible is installed on your control node.
- All playbooks and other Ansible configurations that you create for this sample exam should be stored in `/home/ansible_user/ansible_plays`.
- Create a configuration file `/home/ansible_user/ansible_plays/ansible.cfg` to meet the following requirements:
  - The roles path should include `/home/ansible_user/ansible_plays/roles`, as well as any other path that may be required during the course of the sample exam.
  - The inventory file path is `/home/ansible_user/ansible_plays/hosts`.
- Create an inventory file `/home/ansible_user/ansible_plays/hosts` with the following:
  - `node1.lab.local` should be a member of the `loadbalancers` host group.
  - `node2.lab.local` should be a member of the `webservers` host group.
  - `node3.lab.local` should be a member of the `databases` host group.

---

## Task 2: Create a Shell Script for YUM Repositories

- Create a shell script named `yum.sh` in the directory `/home/ansible_user/ansible_plays`. The script should run Ansible ad-hoc commands to create YUM repository files on all managed hosts with the following specifications:
  - The AppStream repository should have:
    - Base URL: `http://repo.ansi.example.com/AppStream`
    - Name: `RHEL_Appstream`
    - Description: `RHEL 8 Appstream`
    - Enabled status with a GPG key located at `http://repo.ansi.example.com/RPM-GPG-KEY-redhat-release`
  - The BaseOS repository should have:
    - Base URL: `http://repo.ansi.example.com/BaseOS`
    - Name: `RHEL_BaseOS`
    - Description: `RHEL 8 BaseOS`
    - Enabled status with a GPG key located at `http://repo.ansi.example.com/RPM-GPG-KEY-redhat-release`

---

## Task 3: File Content

- Create a playbook `/home/ansible_user/ansible_plays/update_motd.yml` that runs on all inventory hosts and does the following:
  - The playbook should replace any existing content of `/etc/motd` with specific text depending on the host group:
    - On hosts in the `loadbalancers` host group, the line should be “Welcome to the Load Balancer node”.
    - On hosts in the `webservers` host group, the line should be “Welcome to the Web Server node”.
    - On hosts in the `databases` host group, the line should be “Welcome to the Database node”.

---

## Task 4: Install Software

- Create a playbook `/home/ansible_user/ansible_plays/software.yml` that runs on all inventory hosts and installs required software:
  - On hosts in the `webservers` host group, the apache package should be installed and started/enabled.
  - On hosts in the `databases` host group, the mariadb package should be installed and started/enabled. In addition, all packages should be updated to the latest version.

---

## Task 5: Ansible Vault

- Create an Ansible Vault file `/home/ansible_user/ansible_plays/secrets.yml`. The encryption/decryption password should be `password123`. Add the following variables to the vault:
  - `web_password` with the value of `password123`.
  - `db_password` with the value of `password123`.
- Store the Ansible Vault password in the file `/home/ansible_user/ansible_plays/vault_pass`.

---

## Task 6: Users and Groups

- You have been provided with the list of users below. Use `/home/ansible_user/ansible_plays/vars/users.yml` file to save this content:

  ```yaml
  ---
  users:
    - username: john
      uid: 1501
      job: webadmin
    - username: jane
      uid: 1502
      job: webadmin
    - username: doe
      uid: 2501
      job: dbadmin
  ```

- Create a playbook `/home/ansible_user/ansible_plays/create_users.yml` that uses the vault file `/home/ansible_user/ansible_plays/secrets.yml` to achieve the following:
  - Users with jobs in webadmin should be created on all of the `webservers` host group. The password should be retrieved from the `web_password` variable.
  - Users with jobs in dbadmin should be created on all of the `databases` host group. The password should be retrieved from the `db_password` variable.
  - All users should be members of a supplementary group `wheel`.
  - Account passwords should use the SHA512 hash format.
  - The playbook should run successfully when using the `vault_pass` file for decryption.

---

## Task 7: Scheduled Tasks

- Create a playbook `/home/ansible_user/ansible_plays/scheduled_tasks.yml` that runs on servers in the `loadbalancers` host group and does the following:
  - A root crontab entry is created that runs every 30 minutes.
  - The cron job appends the file `/var/log/cron.log` with the output from the `date` command.

---

## Task 8: Create and Work with Roles (Some More)

- Create a role called `apache_role` and store it in `/home/ansible_user/ansible_plays/roles`. The role should satisfy the following requirements:
  - The `httpd` and `firewalld` packages are installed and enabled.
  - The firewall is configured to allow all incoming traffic on HTTP port TCP 80.
  - The Apache service should be restarted every time the file `/var/www/html/index.html` is modified.
  - A Jinja2 template file `index.html.j2` is used to create the file `/var/www/html/index.html` with the following content:

    ```
    This webserver: HOSTNAME is being accessed at: IPADDRESS
    ```

- Create a playbook `/home/ansible_user/ansible_plays/setup_apache.yml` that uses the role and runs on hosts in the `webservers` host group.

---

## Task 9: Generate Hosts File Using a Template

- Create a new directory named `templates` at `/home/ansible_user/ansible_plays/templates`.
- In this directory, create a Jinja2 template file named `hosts.j2` that will be used to generate the `/etc/backup_hosts` file on each managed node. The template should follow this structure:

  ```
  127.0.0.1 localhost {{ ansible_hostname }} {{ ansible_fqdn }}
  127.0.1.1 localhost
  [HOST IP] [HOSTNAME] [HOST FQDN]
  ```

- Create a playbook named `generate_hosts.yml` located in `/home/ansible_user/ansible_plays` that does the following:
  - Runs against all hosts.
  - Uses the `hosts.j2` template to generate the `/etc/backup_hosts` file on each managed node.
  - After running the playbook, it should be possible to reach any node from any other node using the IP address, short name, or FQDN.

---

## Task 10: System Information Report

- Create a playbook named `report.yml` in `/home/ansible_user/ansible_plays` that generates a system information report for each managed node. The report should be saved as `report.txt` and should contain the following information:

  ```
  HOST=<inventory_hostname>
  MEMORY=<total_memory_in_mb>
  BIOS=<bios_version>
  VDA_DISK_SIZE=<disk_size_of_vda_or_NONE if not found>
  VDB_DISK_SIZE=<disk_size_of_vdb_or_NONE if not found>
  ```

- After creating the report file, the playbook should copy it to `/etc/report.txt` on each managed node.
- Ensure that the `report.yml` playbook accurately populates the file with real data for each node.

---

## Task 11: RHEL System Roles

- Use the RHEL system role for SELinux to set the policy state to enforcing.
- Create a playbook `/home/ansible_user/ansible_plays/selinux_role.yml` that uses the role and runs on all hosts.

---

## Task 12: Web Content and SELinux File Contexts

- Create a playbook `/home/ansible_user/ansible_plays/webcontent.yml` that runs on webserver hosts.
- The playbook should satisfy the following:
  - Create a directory owned by the webadmin group (create the group if not present).
  - Ensure permissions are set to r+w+x for owner and group owner, and r+x for others. The SGID bit should also be set.
  - There should be a symbolic link between the `/webadmin` directory and the `/var/www/html/webadmin` directory.
  - Create a file in `/webadmin` named `hello.html` with the content “Hello admin!”
  - Ensure the proper SELinux file context is applied.
- After running the playbook, it should be possible to run the command `curl http://<webserver IP>/webadmin/hello.html` and see the content of the file.

---

## Task 13: Working with Storage and Conditionals

- Create a playbook `/home/ansible_user/ansible_plays/storage.yml` that runs on servers in the `loadbalancers` host group and does the following:
  - Create a primary partition on `/dev/vda` with a number of 1 and a size of 1500MiB.
  - If 1500MiB size partition fails, print “Not enough space, trying 800MiB” and create an 800MiB partition.
  - Give it an ext4 filesystem.
  - Mount it permanently on `/newpartition`.
  - If `/dev/vda` does not exist at all, print “No /dev/vda exists”.
```
