

# Here is a sample Ansible playbook in YAML format to install Oracle Database 19c as a single instance on Oracle Linux 8. This playbook assumes that you have the Oracle installation media available and that you have configured your inventory file appropriately.


---
- name: Install Oracle Database 19c on Oracle Linux 8
  hosts: oracle_db_servers
  become: true
  vars:
    oracle_home: /u01/app/oracle/product/19c/dbhome_1
    oracle_sid: orcl19c
    oracle_install_zip: LINUX.X64_193000_db_home.zip
    oracle_user: oracle
    oracle_group: oinstall
    db_install_dir: /u01/app/oracle

  tasks:
    - name: Install required packages
      yum:
        name:
          - oracle-database-preinstall-19c
          - unzip
        state: present

    - name: Create Oracle user and group
      user:
        name: "{{ oracle_user }}"
        group: "{{ oracle_group }}"
        shell: /bin/
        home: "{{ db_install_dir }}"
        create_home: yes

    - name: Create Oracle installation directory
      file:
        path: "{{ db_install_dir }}"
        state: directory
        owner: "{{ oracle_user }}"
        group: "{{ oracle_group }}"
        mode: '0755'

    - name: Copy Oracle installation zip file
      copy:
        src: "{{ oracle_install_zip }}"
        dest: "{{ db_install_dir }}/{{ oracle_install_zip }}"
        owner: "{{ oracle_user }}"
        group: "{{ oracle_group }}"

    - name: Unzip Oracle installation files
      unarchive:
        src: "{{ db_install_dir }}/{{ oracle_install_zip }}"
        dest: "{{ db_install_dir }}"
        remote_src: yes

    - name: Run Oracle installer
      shell: |
        cd {{ db_install_dir }}/database
        ./runInstaller -silent -responseFile {{ db_install_dir }}/database/response/db_install.rsp \
        oracle.install.option=INSTALL_DB_AND_CONFIG \
        oracle.install.db.InstallDatabase=true \
        oracle.install.db.config.starterdb.password.ALL=oracle123 \
        oracle.install.db.config.starterdb.type=GENERAL_PURPOSE \
        oracle.install.db.config.starterdb.globalDBName={{ oracle_sid }} \
        oracle.install.db.config.starterdb.characterSet=AL32UTF8 \
        oracle.install.db.config.starterdb.memoryOption=false \
        oracle.install.db.config.starterdb.silentInstall=true \
        oracle.install.db.config.starterdb.dataLocation={{ db_install_dir }}/oradata \
        oracle.install.db.config.starterdb.recoveryLocation={{ db_install_dir }}/flash_recovery_area
      args:
        executable: /bin/

    - name: Execute root scripts
      shell: |
        /u01/app/oraInventory/orainstRoot.sh
        {{ oracle_home }}/root.sh
      args:
        executable: /bin/

    - name: Set Oracle SID environment variable
      lineinfile:
        path: /etc/profile.d/oracle.sh
        line: "export ORACLE_SID={{ oracle_sid }}"
        state: present

    - name: Reload profile
      shell: source /etc/profile.d/oracle.sh
      args:
        executable: /bin/

    - name: Verify Oracle installation
      command: sqlplus / as sysdba
      register: sqlplus_output
      ignore_errors: yes

    - name: Display SQL*Plus output
      debug:
        var: sqlplus_output.stdout_lines

### Notes:
- Ensure that you have the Oracle Database installation zip file (`LINUX.X64_193000_db_home.zip`) placed in the specified directory on the control machine.
- Modify the `oracle_sid`, `oracle_user`, and `oracle_group` variables as needed to fit your environment.
- This playbook includes tasks to install required packages, create the Oracle user, copy the installation files, run the installer, execute necessary root scripts, and set the Oracle SID.
- Make sure to adjust the response file path and parameters as needed based on your specific installation requirements.

Citations:
[1] https://github.com/sugitk/ansible-oracle19c
[2] https://github.com/asiandevs/OracleDBAwithAnsible
[3] https://www.youtube.com/watch?v=4BlwFjXfF7A
[4] https://oracledbwr.com/oracle-automation-create-a-dba-user-using-ansible-tool/
[5] https://dbakevlar.com/2023/04/chatgpt-building-out-oracle-with-templates-and-ansible-for-a-dataguard-environment/
[6] https://questoraclecommunity.org/learn/blogs/creating-oracle-19c-physical-standby-configuring-data-guard-broker-with-ansible-github/
[7] https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/ansiblesamples.htm
[8] https://simoncw.com/posts/dev-setup-mac-python-rust/



# To configure the inventory file for installing Oracle Database 19c on Oracle Linux 8 using Ansible, you can follow this example setup. The inventory file defines the hosts where the playbook will be executed.

# Example Inventory File

Create a file named `inventory` and include the following content:


[oracle_db_servers]
192.168.56.102 ansible_user=oracle ansible_ssh_private_key_file=/path/to/private/key

[oracle_db_servers:vars]
oracle_home=/u01/app/oracle/product/19c/dbhome_1
oracle_sid=orcl19c


#  Explanation

- [oracle_db_servers]: This section defines a group of hosts (in this case, Oracle database servers). You can add multiple IP addresses or hostnames under this group.

- 192.168.56.102: Replace this with the actual IP address of your Oracle Linux 8 server.

- ansible_user: This specifies the user that Ansible will use to connect to the remote host. Ensure this user has the necessary privileges to install software.

- ansible_ssh_private_key_file: This is the path to the SSH private key used for authentication. Replace `/path/to/private/key` with the actual path to your SSH key.

- [oracle_db_servers:vars]: This section allows you to define variables that apply to all hosts in the `oracle_db_servers` group. Here, `oracle_home` and `oracle_sid` are set for the Oracle installation.

### Additional Steps

1. Update Ansible Configuration: Ensure that your `ansible.cfg` file points to the inventory file you just created. It should look something like this:

   
   [defaults]
   inventory = ./inventory
  

2. Run the Playbook: After setting up your inventory file, you can run your Ansible playbook using the following command:

   
   ansible-playbook -i inventory your_playbook.yml
 

Replace `your_playbook.yml` with the name of your actual playbook file.

This setup allows you to manage and automate the installation of Oracle Database 19c on your Oracle Linux 8 servers effectively.

Citations:
[1] https://www.arubanetworks.com/techdocs/IMC/5200-1293.pdf
[2] https://oraclefact.wordpress.com/2023/12/12/oracle-database-19c-installation-on-oel-8-or-rhel-8-64bit/
[3] https://github.com/asiandevs/OracleDBAwithAnsible
[4] https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-8
[5] https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/server-configuration-checklist-for-oracle-database-installation.html
[6] https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-9
[7] https://github.com/sugitk/ansible-oracle19c
[8] https://www.youtube.com/watch?v=YEVLRm73M1k



# To disable the firewall and SELinux for Oracle Database 19c installation on Oracle Linux 8, follow these steps:

## Disable the Firewall

1. As the root user, stop and disable the firewall:


   systemctl stop firewalld
   systemctl disable firewalld


2. Verify the firewall status:


   systemctl status firewalld


## Disable SELinux

1. Open the SELinux configuration file:


   vi /etc/selinux/config


2. Set the value for the `SELINUX` parameter to `disabled`:


   SELINUX=disabled


3. Save the configuration and restart the system for the changes to take effect.

After disabling the firewall and SELinux, you can proceed with the Oracle Database 19c installation. However, it's important to note that disabling SELinux may impact the overall security of your system. 

If you prefer to keep SELinux enabled, you can configure it to be in permissive mode instead of disabled. To do this, set `SELINUX=permissive` in the `/etc/selinux/config` file and restart the system.

Refer to the Oracle Database Installation Guide for Linux for more detailed instructions on preparing your system for Oracle Database 19c installation.

Citations:
[1] https://dev.to/dm8ry/how-to-setup-oracle-19c-db-on-os-oracle-enterprise-linux-8-3d1n
[2] https://blog.rishoradev.com/2022/01/25/oracle-db-19c-oracle-linux-8/
[3] https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-9
[4] https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-8
[5] https://www.arubanetworks.com/techdocs/IMC/5200-1293.pdf
[6] https://forums.oracle.com/ords/r/apexds/community/q?question=oracle-and-selinux-4722
[7] https://www.youtube.com/watch?v=YEVLRm73M1k
[8] https://oraclefact.wordpress.com/2023/12/12/oracle-database-19c-installation-on-oel-8-or-rhel-8-64bit/



# Here are the steps to re-enable SELinux and the firewall after installing Oracle Database 19c on Oracle Linux 8:

## Re-enable SELinux

1. Open the SELinux configuration file:


   vi /etc/selinux/config


2. Set the value for the `SELINUX` parameter to `enforcing`:


   SELINUX=enforcing


3. Save the configuration and restart the system for the changes to take effect.

4. Verify SELinux is enabled by running:


   sestatus


   It should output `SELinux status: enabled` and `Current mode: enforcing`.

## Re-enable Firewall

1. Enable and start the firewall:


   systemctl enable firewalld
   systemctl start firewalld


2. Open the required ports for Oracle Database:


   firewall-cmd --permanent --add-port=1521/tcp
   firewall-cmd --permanent --add-port=5500/tcp
   firewall-cmd --reload


   This opens ports 1521 (default listener port) and 5500 (Enterprise Manager) through the firewall.

3. Verify the firewall status:


   systemctl status firewalld


   It should show the firewall is active and running.

After re-enabling SELinux and the firewall, you may need to adjust SELinux policies or firewall rules to allow Oracle Database to function properly. Consult the Oracle Database documentation for specific configuration steps.

It's important to note that SELinux and the firewall provide important security measures for your system. Disabling them during installation is a common practice, but they should be re-enabled after installation is complete to maintain a secure environment.

Citations:
[1] https://community.oracle.com/mosc/discussion/4529998/should-i-enable-selinux-firewall-after-installation-of-oracle-on-linux
[2] https://dev.to/dm8ry/how-to-setup-oracle-19c-db-on-os-oracle-enterprise-linux-8-3d1n
[3] https://blog.rishoradev.com/2022/01/25/oracle-db-19c-oracle-linux-8/
[4] https://www.arubanetworks.com/techdocs/IMC/5200-1293.pdf
[5] https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-8
[6] https://oracle-base.com/articles/19c/oracle-db-19c-installation-on-oracle-linux-9
[7] https://forums.oracle.com/ords/r/apexds/community/q?question=oracle-and-selinux-4722
[8] https://oraclefact.wordpress.com/2023/12/12/oracle-database-19c-installation-on-oel-8-or-rhel-8-64bit/


