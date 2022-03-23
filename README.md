# Ansible role Elasticsearch
[![CI Molecule](https://github.com/darexsu/ansible-role-elasticsearch/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-elasticsearch/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/58405?color=blue&label=downloads)

  - Role:
      - [platforms](#platforms)
      - [install](#install)
      - [requirements](#requirements)
      - [merge behaviour](#merge-behaviour)
  - Playbooks (merge version):
      - [install and configure: Elasticsearch](#install-and-configure-elasticsearch-merge-version)
          - [install: Elasticsearch](#install-elasticsearch-merge-version)          
          - [configure: elasticsearch.yml](#configure-elasticsearchyml-merge-version)
          - [configure: jvm.options](#configure-jvmoptions-merge-version)
  - Playbooks (full version):
      - [install and configure: Elasticsearch](#install-and-configure-elasticsearch-full-version)
          - [install: Elasticsearch](#install-elasticsearch-full-version)          
          - [configure: elasticsearch.yml](#configure-elasticsearchyml-full-version)
          - [configure: jvm.options](#configure-jvmoptions-full-version)

### Platforms

|  Testing         |  repo: elastic    |
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
### Requirements
collections: [ansible.posix](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html)

roles: [FirewallD](https://github.com/darexsu/ansible-role-firewalld) (will automatically be installed)

### Merge behaviour

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

##### Install and configure: Elasticsearch (merge version)
```yaml
- hosts: all
  become: true

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
        src: "jvm_options.j2"
        backup: false
        data:
          - "-Djava.io.tmpdir=/var/log/elasticsearch"

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Install: Elasticsearch (merge version)
```yaml
- hosts: all
  become: true

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
##### Configure: elasticsearch.yml (merge version)
```yaml
- hosts: all
  become: true

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
        data: |
          path.data: /var/lib/elasticsearch
          path.logs: /var/log/elasticsearch
          xpack.security.enabled: false
          xpack.security.enrollment.enabled: true
          xpack.security.http.ssl:
            enabled: true
            keystore.path: certs/http.p12
          xpack.security.transport.ssl:
            enabled: true
            verification_mode: certificate
            keystore.path: certs/transport.p12
            truststore.path: certs/transport.p12
          http.host: [_local_, _site_]

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Configure: jvm.options (merge version)
```yaml
- hosts: all
  become: true

  vars:
    merge:
      # ElasticSearch
      elasticsearch:
        enabled: true
      # ElasticSearch -> config -> jvm.options
      elasticsearch_jvm_options:
        enabled: true
        file: "jvm.options"
        src: "jvm_options.j2"
        backup: false
        data: |
          8-13:-XX:+UseConcMarkSweepGC
          8-13:-XX:CMSInitiatingOccupancyFraction=75
          8-13:-XX:+UseCMSInitiatingOccupancyOnly
          14-:-XX:+UseG1GC
          -Djava.io.tmpdir=${ES_TMPDIR}
          -XX:+HeapDumpOnOutOfMemoryError
          9-:-XX:+ExitOnOutOfMemoryError
          -XX:HeapDumpPath=/var/lib/elasticsearch
          -XX:ErrorFile=/var/log/elasticsearch/hs_err_pid%p.log
          -Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Install and configure: Elasticsearch (full version)
```yaml
- hosts: all
  become: true

  vars:
    # ElasticSearch
    elasticsearch:
      enabled: true
      version: "8.x"
      repo: "elastic"
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
      data: |
        path.data: /var/lib/elasticsearch
        path.logs: /var/log/elasticsearch
        xpack.security.enabled: false
        xpack.security.enrollment.enabled: true
        xpack.security.http.ssl:
          enabled: true
          keystore.path: certs/http.p12
        xpack.security.transport.ssl:
          enabled: true
          verification_mode: certificate
          keystore.path: certs/transport.p12
          truststore.path: certs/transport.p12
        http.host: [_local_, _site_]
    # ElasticSearch -> config -> jvm.options
    elasticsearch_jvm_options:
      enabled: true
      file: "jvm.options"
      src: "jvm_options.j2"
      backup: false
      data: |
        8-13:-XX:+UseConcMarkSweepGC
        8-13:-XX:CMSInitiatingOccupancyFraction=75
        8-13:-XX:+UseCMSInitiatingOccupancyOnly
        14-:-XX:+UseG1GC
        -Djava.io.tmpdir=${ES_TMPDIR}
        -XX:+HeapDumpOnOutOfMemoryError
        9-:-XX:+ExitOnOutOfMemoryError
        -XX:HeapDumpPath=/var/lib/elasticsearch
        -XX:ErrorFile=/var/log/elasticsearch/hs_err_pid%p.log
        -Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Install: Elasticsearch (full version)
```yaml
- hosts: all
  become: true

  vars:
    # ElasticSearch
    elasticsearch:
      enabled: true
      version: "8.x"
      repo: "elastic"
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

  vars:
    # ElasticSearch
    elasticsearch:
      enabled: true
      version: "8.x"
      src: "elastic"
      service:
        enabled: true
        state: "started"
    # ElasticSearch -> config -> elasticsearch.yml
    elasticsearch_yml:
      enabled: true
      file: "elasticsearch.yml"
      src: "elasticsearch_yml.j2"
      backup: false
      data: |
        path.data: /var/lib/elasticsearch
        path.logs: /var/log/elasticsearch
        xpack.security.enabled: false
        xpack.security.enrollment.enabled: true
        xpack.security.http.ssl:
          enabled: true
          keystore.path: certs/http.p12
        xpack.security.transport.ssl:
          enabled: true
          verification_mode: certificate
          keystore.path: certs/transport.p12
          truststore.path: certs/transport.p12
        http.host: [_local_, _site_]

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
##### Configure: jvm.options (full version)
```yaml
- hosts: all
  become: true

  vars:
    # ElasticSearch
    elasticsearch:
      enabled: true
      version: "8.x"
      src: "elastic"
      service:
        enabled: true
        state: "started"
    # ElasticSearch -> config -> jvm.options
    elasticsearch_jvm_options:
      enabled: true
      file: "jvm.options"
      src: "jvm_options.j2"
      backup: false
      data: |
        8-13:-XX:+UseConcMarkSweepGC
        8-13:-XX:CMSInitiatingOccupancyFraction=75
        8-13:-XX:+UseCMSInitiatingOccupancyOnly
        14-:-XX:+UseG1GC
        -Djava.io.tmpdir=${ES_TMPDIR}
        -XX:+HeapDumpOnOutOfMemoryError
        9-:-XX:+ExitOnOutOfMemoryError
        -XX:HeapDumpPath=/var/lib/elasticsearch
        -XX:ErrorFile=/var/log/elasticsearch/hs_err_pid%p.log
        -Xlog:gc*,gc+age=trace,safepoint:file=/var/log/elasticsearch/gc.log:utctime,pid,tags:filecount=32,filesize=64m

  tasks:
    - name: role darexsu elasticsearch
      include_role:
        name: darexsu.elasticsearch
```
