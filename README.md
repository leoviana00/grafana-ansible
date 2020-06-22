# Grafana

- Provisionamento do grafana através do ansible

# Configuração:

- Criar arquivo hosts configurando a maquina na qual desaja fazer a configuração, exemplo:

- srv_lb  ansible_ssh_host=3.91.21.178 ansible_ssh_user=ubuntu ansible_ssh_private_key_file=/home/leoviana/.ssh/id_rsa

Adicionar o interpretador:
ansible_python_interpreter=/usr/bin/python3

# Teste

- Teste a comunicação usando o módulo do ansible, comando: ansible -i hosts -m ping all

# Criar o playbook com as tasks.

    - hosts: srv_mt
      become: yes
      vars:
        grafana_version: 7.0.3
        arch: amd64
        grafana_filename: grafana_{{ grafana_version }}_{{ arch }}.deb---
    
      tasks:
      - name: Verify version
        command: dpkg-query -W --showformat='${version}' grafana
        register: grafana_check_version---
        failed_when: grafana_check_version.rc > 1
        changed_when: (grafana_check_version.rc == 1) or
                      (grafana_check_version | version_compare("{{ grafana_version }}", "<"))
    
      - name: Download grafana deb file
        get_url:
          #url="https://grafanarel.s3.amazonaws.com/builds/{{ grafana_filename }}"
          url="https://dl.grafana.com/oss/release/{{ grafana_filename }}"
          dest="/tmp/{{ grafana_filename }}"
        when: grafana_check_version.changed
    
      - name: Verify grafana deb file
        command: dpkg-deb -W --showformat='${version}' /tmp/{{ grafana_filename }}
        register: grafana_deb_check_version
        failed_when: grafana_deb_check_version.stdout != "{{ grafana_version }}"
        when: grafana_check_version.changed
    
      - name: Install grafana
        apt: deb="/tmp/{{ grafana_filename }}" state=present
        when: grafana_check_version.changed

      - name: Startar e habilitar o Grafana
        service: name=grafana-server state=started enabled=yes
        
        
 # Rode a playbook com o seguinte comando
 
 - ansible-playbook -i hosts nome-playbook.yaml
<<<<<<< HEAD
=======
     
>>>>>>> 5337e19a436d5696dd932b57389c15e201d7a8be
