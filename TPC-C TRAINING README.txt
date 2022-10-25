Download links:


1. VirtualBox: https://www.virtualbox.org/
2. Vagrant: https://www.vagrantup.com/


Instructions:
1. edb-deployment: 
   1. Installation: https://github.com/EnterpriseDB/postgres-deployment/blob/master/README.md
   2. Extra setup steps for Windows: https://github.com/EnterpriseDB/postgres-deployment/blob/master/README-WIN.md


Other links:
1. TPC-C Results: https://www.tpc.org/tpcc/results/tpcc_perf_results5.asp?resulttype=all
2. edb repository signup: https://www.enterprisedb.com/accounts/register
3. edb benchmark execution examples: https://github.com/EnterpriseDB/edb-benchmarks
4. DBT-2 source code: https://github.com/osdldbt/dbt2
5. Example DBT-2 report: https://osdldbt.github.io/dbt-reports/dbt2/3-tier/report.html


Special notes:
1. Install edb-deployment dev code: 
$ pip3 install https://github.com/EnterpriseDB/postgres-deployment/archive/refs/heads/master.zip --upgrade
	

When using virtualbox or baremetal option (Note examples show baremetal, but applies to virtualbox too)
* Execute the specs sub-command:
$ edb-deployment baremetal specs > /tmp/specs.json
	* Edit /tmp/specs.json:


{
 "ssh_user": "julien",
 "pg_data": null,
 "pg_wal": null,
 "postgres_server_1": {
   "name": "pg1",
   "public_ip": "192.168.122.6",
   "private_ip": "192.168.122.6"
 },
 "pem_server_1": {
   "name": "pem1",
   "public_ip": null,
   "private_ip": null
 },
 "backup_server_1": {
   "name": "backup1",
   "public_ip": null,
   "private_ip": null
 },
 "dbt2_client_0": {
   "name": "dbt2client",
   "public_ip": "192.168.122.55",
   "private_ip": "192.168.122.55"
 },
 "dbt2_driver_0": {
   "name": "dbt2driver",
   "public_ip": "192.168.122.98",
   "private_ip": "192.168.122.98"
 },
 "dbt2": true,
 "dbt2_client": {
   "count": 1
 },
 "dbt2_driver": {
   "count": 1
 }
}
	* Execute the configure sub-command:
$ edb-deployment baremetal configure -a EDB-RA-1 -u "foo:bar" -t PG -v 14 -s /tmp/specs.bm.json dbt2
	

* Edit ~/.edb-deployment/baremetal/dbt2/inventory.yml (remove pem and barman system):
all:
 children:
   dbt2_client:
     hosts:
       dbt2_client_0:
         ansible_host: 192.168.122.55
         private_ip: 192.168.122.55
   dbt2_driver:
     hosts:
       dbt2_driver_0:
         ansible_host: 192.168.122.98
         private_ip: 192.168.122.98
   primary:
     hosts:
       pg1:
         ansible_host: 192.168.122.6
         dbt2: true
         private_ip: 192.168.122.6
	

* Edit ~/.edb-deployment/baremetal/dbt2/playbook.yml (remove barman and pem related roles)
---
- hosts: all
 name: Postgres deployment playbook for reference architecture EDB-RA-1
 become: yes
 gather_facts: yes
 any_errors_fatal: True
 max_fail_percentage: 0

 collections:
    - edb_devops.edb_postgres

 pre_tasks:
   - name:
     set_fact:
       enable_edb_repo: false

 roles:
   - role: setup_repo
     when: "'setup_repo' in lookup('edb_devops.edb_postgres.supported_roles', wantlist=True)"
   - role: install_dbserver
     when: "'install_dbserver' in lookup('edb_devops.edb_postgres.supported_roles', wantlist=True)"
   - role: init_dbserver
     when: "'init_dbserver' in lookup('edb_devops.edb_postgres.supported_roles', wantlist=True)"
   - role: setup_dbt2_driver
     when: "'setup_dbt2_driver' in lookup('edb_devops.edb_postgres.supported_roles', wantlist=True)"
   - role: setup_dbt2_client
     when: "'setup_dbt2_client' in lookup('edb_devops.edb_postgres.supported_roles', wantlist=True)"
   - role: setup_dbt2
     when: "'setup_dbt2' in lookup('edb_devops.edb_postgres.supported_roles', wantlist=True)"
   - role: autotuning
     when: "'autotuning' in lookup('edb_devops.edb_postgres.supported_roles', wantlist=True)"
	* Execute the deploy command:
$ edb-deployment baremetal deploy dbt2