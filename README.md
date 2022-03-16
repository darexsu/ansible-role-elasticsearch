# Ansible role Elasticsearch
[![CI Molecule](https://github.com/darexsu/ansible-role-elasticsearch/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-elasticsearch/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/57564?color=blue&label=downloads)

  - Role:
      - [platforms](#platforms)
      - [install](#install)
      - [behaviour](#behaviour)
  - Playbooks (short version):
      - [install and configure: Elasticsearch](#install-and-configure-elasticsearch-short-version)
          - [install: Elasticsearch](#install-elasticsearch-short-version)          
          - [configure: elasticsearch.yml](#configure-elasticsearchyml-short-version)
          - [configure: jvm.options](#configure-jvmoptions-short-version)
  - Playbooks (full version):
      - [install and configure: Elasticsearch](#install-and-configure-elasticsearch-full-version)
          - [install: Elasticsearch](#install-elasticsearch-full-version)          
          - [configure: elasticsearch.yml](#configure-elasticsearchyml-full-version)
          - [configure: jvm.options](#configure-jvmoptions-full-version)

### Platforms

|  Testing         |  Third-party repo |
| :--------------: |  :-------------:  |
| Debian 11        |    elastic.co     |
| Debian 10        |    elastic.co     |
| Ubuntu 20.04     |    elastic.co     |
| Ubuntu 18.04     |    elastic.co     |
| Oracle Linux 8   |    elastic.co     |
| Rocky Linux 8    |    elastic.co     |

### Install

```
ansible-galaxy install darexsu.elasticsearch --force
```

### Behaviour

Replace or Merge dictionaries (with "hash_behaviour=replace" in ansible.cfg):
```
# Replace             # Merge
---                   ---
  vars:                 vars:
    dict:                 merge:
      a: "value"            dict: 
      b: "value"              a: "value" 
                              b: "value"

# How does merge work?
Your vars [host_vars]  -->  default vars [current role] --> default vars [include role]
  
  dict:          dict:              dict:
    a: "1" -->     a: "1"    -->      a: "1"
                   b: "2"    -->      b: "2"
                                      c: "3"
    
```

##### Install and configure: Elasticsearch (short version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
        version: "8.x"
      # ElasticSearch -> install
      elasticsearch_install:
        enabled: true
      # ElasticSearch -> config -> elasticsearch.yml
      elasticsearch_yml:
        enabled: true
      # ElasticSearch -> config -> jvm.options
      elasticsearch_jvm_options:
        enabled: true
        file: "jvm.options"
        src: "elasticsearch_jvm_options.j2"
        backup: false
        vars:
          - "-Djava.io.tmpdir=/var/log/elasticsearch"

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Install: Elasticsearch (short version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
        version: "8.x"
      # ElasticSearch -> install
      elasticsearch_install:
        enabled: true

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Configure: elasticsearch.yml (short version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
      # ElasticSearch -> config -> elasticsearch.yml
      elasticsearch_yml:
        enabled: true
        file: "elasticsearch.yml"
        src: "elasticsearch_yml.j2"
        backup: false
        vars:
          path.data: "/var/lib/elasticsearch"
          path.logs: "/var/log/elasticsearch"
          xpack.security.enabled: false
          xpack.security.enrollment.enabled: true
          xpack.security.http.ssl:
            enabled: true
            keystore.path: "certs/http.p12"
          xpack.security.transport.ssl:
            enabled: true
            verification_mode: "certificate"
            keystore.path: "certs/transport.p12"
            truststore.path: "certs/transport.p12"
          cluster.initial_master_nodes: ["name_master_node"]
          http.host: [_local_, _site_]

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Configure: jvmo.options (short version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
      # ElasticSearch -> config -> jvm.options
      elasticsearch_jvm_options:
        enabled: true
        file: "jvm.options"
        src: "elasticsearch_jvm_options.j2"
        backup: false
        vars:
          - "8-13:-XX:+UseConcMarkSweepGC"
          - "8-13:-XX:CMSInitiatingOccupancyFraction=75"
          - "8-13:-XX:+UseCMSInitiatingOccupancyOnly"
          - "14-:-XX:+UseG1GC"
          - "-Djava.io.tmpdir=/var/log/elasticsearch"
          - "-XX:+HeapDumpOnOutOfMemoryError"
          - "9-:-XX:+ExitOnOutOfMemoryError"
          - "-XX:HeapDumpPath=/var/lib/elasticsearch"
          - "-XX:ErrorFile=/var/log/elasticsearch/hs_err_pid%p.log"
          - "-Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m"

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Install and configure: Elasticsearch (full version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
        version: "8.x"
        src: "elastic_co"
        service:
          enabled: true
          state: "started"
      # ElasticSearch -> install
      elasticsearch_install:
        enabled: true
        packages: ["elasticsearch"]
      # ElasticSearch -> config -> elasticsearch.yml
      elasticsearch_yml:
        enabled: true
        file: "elasticsearch.yml"
        src: "elasticsearch_yml.j2"
        backup: false
        vars:
          path.data: "/var/lib/elasticsearch"
          path.logs: "/var/log/elasticsearch"
          xpack.security.enabled: false
          xpack.security.enrollment.enabled: true
          xpack.security.http.ssl:
            enabled: true
            keystore.path: "certs/http.p12"
          xpack.security.transport.ssl:
            enabled: true
            verification_mode: "certificate"
            keystore.path: "certs/transport.p12"
            truststore.path: "certs/transport.p12"
          cluster.initial_master_nodes: ["name_master_node"]
          http.host: [_local_, _site_]
      # ElasticSearch -> config -> jvm.options
      elasticsearch_jvm_options:
        enabled: true
        file: "jvm.options"
        src: "elasticsearch_jvm_options.j2"
        backup: false
        vars:
          - "8-13:-XX:+UseConcMarkSweepGC"
          - "8-13:-XX:CMSInitiatingOccupancyFraction=75"
          - "8-13:-XX:+UseCMSInitiatingOccupancyOnly"
          - "14-:-XX:+UseG1GC"
          - "-Djava.io.tmpdir=/var/log/elasticsearch"
          - "-XX:+HeapDumpOnOutOfMemoryError"
          - "9-:-XX:+ExitOnOutOfMemoryError"
          - "-XX:HeapDumpPath=/var/lib/elasticsearch"
          - "-XX:ErrorFile=/var/log/elasticsearch/hs_err_pid%p.log"
          - "-Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m"

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Install: Elasticsearch (full version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
        version: "8.x"
        src: "elastic_co"
        service:
          enabled: true
          state: "started"
      # ElasticSearch -> install
      elasticsearch_install:
        enabled: true
        packages: ["elasticsearch"]

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Configure: elasticsearch.yml (full version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
        version: "8.x"
        src: "elastic_co"
        service:
          enabled: true
          state: "started"
      # ElasticSearch -> config -> elasticsearch.yml
      elasticsearch_yml:
        enabled: true
        file: "elasticsearch.yml"
        src: "elasticsearch_yml.j2"
        backup: false
        vars:
          path.data: "/var/lib/elasticsearch"
          path.logs: "/var/log/elasticsearch"
          xpack.security.enabled: false
          xpack.security.enrollment.enabled: true
          xpack.security.http.ssl:
            enabled: true
            keystore.path: "certs/http.p12"
          xpack.security.transport.ssl:
            enabled: true
            verification_mode: "certificate"
            keystore.path: "certs/transport.p12"
            truststore.path: "certs/transport.p12"
          cluster.initial_master_nodes: ["name_master_node"]
          http.host: [_local_, _site_]

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Configure: jvmo.options (full version)
```yaml
- hosts: all
  become: true

  pre_tasks:
    - name: Unnessessary command
      ansible.builtin.shell:
        cmd: "{{ lookup('env', 'ANSIBLE_COMMAND') }}"
      when: lookup('env', 'ANSIBLE_COMMAND') | length > 0

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
        version: "8.x"
        src: "elastic_co"
        service:
          enabled: true
          state: "started"
      # ElasticSearch -> config -> jvm.options
      elasticsearch_jvm_options:
        enabled: true
        file: "jvm.options"
        src: "elasticsearch_jvm_options.j2"
        backup: false
        vars:
          - "8-13:-XX:+UseConcMarkSweepGC"
          - "8-13:-XX:CMSInitiatingOccupancyFraction=75"
          - "8-13:-XX:+UseCMSInitiatingOccupancyOnly"
          - "14-:-XX:+UseG1GC"
          - "-Djava.io.tmpdir=/var/log/elasticsearch"
          - "-XX:+HeapDumpOnOutOfMemoryError"
          - "9-:-XX:+ExitOnOutOfMemoryError"
          - "-XX:HeapDumpPath=/var/lib/elasticsearch"
          - "-XX:ErrorFile=/var/log/elasticsearch/hs_err_pid%p.log"
          - "-Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m"

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```