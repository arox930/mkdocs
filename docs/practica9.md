# **iaw-practica 13.4 (Aarón García Crisol)**
# Ansible

<img src="https://www.educative.io/cdn-cgi/image/f=auto,fit=contain,w=1200/api/page/5017398770401280/image/download/5607644146958336" />

En esta practica debemos de desplegar mediante Ansible las infraestructuras de la practica 7 y la practica 9.

## Infraestructura de la practica 9

Para ello crearemos un archivo main.yml, el cual irá lanzando los playbooks que tendremos alojados en nuestra carpeta playbook, el contenido tanto de los playbooks como del main.yml es el siguiente:

### playbook/backend.yml

```yml
---
- name: Playbook para crear grupo de seguridad e instancia para Backend
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    nombre_sg: backend-sg
    descripcion_sg: grupo de seguridad para Backend
    nombre_instancia: Backend
    id_ami: ami-06878d265978313ca
    tipo_instancia: t2.small
    key: vockey
  tasks:
  - name: Creamos el grupo de seguridad
    ec2_group:
      name: "{{ nombre_sg }}"
      description: "{{ descripcion_sg }}"
      rules:
      - proto: tcp
        from_port: 22
        to_port: 22
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 3306
        to_port: 3306
        cidr_ip: 0.0.0.0/0
  - name: Creamos la instancia
    ec2_instance:
      name: "{{ nombre_instancia }}"
      key_name: "{{ key }}"
      security_group: "{{ nombre_sg }}"
      instance_type: "{{ tipo_instancia }}"
      image_id: "{{ id_ami }}"
      state: running
    register: inst
```

### playbook/frontend1.yml

```yml
---
- name: Playbook para crear grupo de seguridad e instancia para Frontend
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    nombre_sg: frontend-sg
    descripcion_sg: grupo de seguridad para Frontend
    nombre_instancia: Frontend-01
    id_ami: ami-06878d265978313ca
    tipo_instancia: t2.small
    key: vockey
  tasks:
  - name: Creamos el grupo de seguridad
    ec2_group:
      name: "{{ nombre_sg }}"
      description: "{{ descripcion_sg }}"
      rules:
      - proto: tcp
        from_port: 22
        to_port: 22
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 80
        to_port: 80
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 443
        to_port: 443
        cidr_ip: 0.0.0.0/0
    register: sg
  - name: Creamos la instancia
    ec2_instance:
      name: "{{ nombre_instancia }}"
      key_name: "{{ key }}"
      security_group: "{{ nombre_sg }}"
      instance_type: "{{ tipo_instancia }}"
      image_id: "{{ id_ami }}"
      state: running
    register: inst
```

### playbook/frontend2.yml

```yml
---
- name: Playbook para crear grupo de seguridad e instancia para Frontend
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    nombre_sg: frontend-sg
    descripcion_sg: grupo de seguridad para Frontend
    nombre_instancia: Frontend-02
    id_ami: ami-06878d265978313ca
    tipo_instancia: t2.small
    key: vockey
  tasks:
  - name: Creamos el grupo de seguridad
    ec2_group:
      name: "{{ nombre_sg }}"
      description: "{{ descripcion_sg }}"
      rules:
      - proto: tcp
        from_port: 22
        to_port: 22
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 80
        to_port: 80
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 443
        to_port: 443
        cidr_ip: 0.0.0.0/0
    register: sg
  - name: Creamos la instancia
    ec2_instance:
      name: "{{ nombre_instancia }}"
      key_name: "{{ key }}"
      security_group: "{{ nombre_sg }}"
      instance_type: "{{ tipo_instancia }}"
      image_id: "{{ id_ami }}"
      state: running
    register: inst
```

### playbook/balanceador.yml

```yml
---
- name: Playbook para crear grupo de seguridad e instancia para Balanceador de Carga
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    nombre_sg: balanceador-sg
    descripcion_sg: grupo de seguridad para Balanceador
    nombre_instancia: Balanceador
    id_ami: ami-06878d265978313ca
    tipo_instancia: t2.small
    key: vockey
  tasks:
  - name: Creamos el grupo de seguridad
    ec2_group:
      name: "{{ nombre_sg }}"
      description: "{{ descripcion_sg }}"
      rules:
      - proto: tcp
        from_port: 22
        to_port: 22
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 80
        to_port: 80
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 443
        to_port: 443
        cidr_ip: 0.0.0.0/0
    register: sg
  - name: Creamos la instancia
    ec2_instance:
      name: "{{ nombre_instancia }}"
      key_name: "{{ key }}"
      security_group: "{{ nombre_sg }}"
      instance_type: "{{ tipo_instancia }}"
      image_id: "{{ id_ami }}"
      state: running
    register: inst
  - name: Mostramos el contenido de la variable inst
    debug:
      msg: "inst: {{ inst }}"
  - name: Creamos una IP elástica y la asociar a la instancia
    ec2_eip:
      device_id: "{{ inst.instances[0].instance_id }}"
    register: ip_elas
  - name: Mostramos cual es nuestra IP elastica
    debug:
      msg: "Nuestra IP elástica es: {{ ip_elas.public_ip }}"
```

### playbook/nfs.yml

```yml
---
- name: Playbook para crear grupo de seguridad e instancia para NFS
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    nombre_sg: nfs-sg
    descripcion_sg: grupo de seguridad para NFS
    nombre_instancia: NFS
    id_ami: ami-06878d265978313ca
    tipo_instancia: t2.small
    key: vockey
  tasks:
  - name: Crear un grupo de seguridad
    ec2_group:
      name: "{{ nombre_sg }}"
      description: "{{ descripcion_sg }}"
      rules:
      - proto: tcp
        from_port: 22
        to_port: 22
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 2049
        to_port: 2049
        cidr_ip: 0.0.0.0/0
    register: sg
  - name: Creamos la instancia
    ec2_instance:
      name: "{{ nombre_instancia }}"
      key_name: "{{ key }}"
      security_group: "{{ nombre_sg }}"
      instance_type: "{{ tipo_instancia }}"
      image_id: "{{ id_ami }}"
      state: running
    register: inst
```

### main.yml

```yml
---
  - import_playbook: ./playbook/backend.yml
  - import_playbook: ./playbook/frontend1.yml
  - import_playbook: ./playbook/frontend2.yml
  - import_playbook: ./playbook/balanceador.yml
  - import_playbook: ./playbook/nfs.yml
```