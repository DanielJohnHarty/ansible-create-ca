# ansible-create-ca
Create a certification authority and create/ distribute certificates

`certs_fqdns` variable is a list of domains for which certificates will be created and to which the certificates will be distributed to.
`inventory_hostname` variable is the host name in the ansible hosts file

## Example playbook

```yaml

---
- hosts: ca
  become: yes
  vars:
    - ca_name: tdp-getting-started
    - certs_fqdns: 
      - edge-01
      - edge-02
      - master-01
      - master-02
      - master-03
      - worker-01
      - worker-02
      - worker-03
  tasks:
    - import_role:
        name: roles/ansible-ca

- hosts: all, !ca
  become: yes
  tasks:

    - name: Check /etc/ssl/certs dir exists
      stat:
        path: /etc/ssl/certs
      register: certs_dir

    - name: Create certs dir if missing
      file: 
        path: /etc/ssl/certs
        state: directory
      when: not certs_dir.stat.exists 

    - name: Distribute certs and keys to nodes
      copy:
        src: files/certs/{{inventory_hostname}}.{{item}}
        dest: /etc/ssl/certs/{{inventory_hostname}}.{{item}}
      with_items:
        - key
        - pem
      tags:
        - dist-certs

    - name: Distribute certs and keys to nodes
      copy:
        src: files/certs/root.pem
        dest: /etc/ssl/certs/root.pem
```