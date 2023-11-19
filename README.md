   ========
   Ansible (все действия производить при подключенном VPN)
   ========

   быстрое выполнение

   скачать все с моего github и распаковать в папку Ansible
   
   войти в папку где будут скачанные файлы через терминал и последовательно запустить следующие команды

   $ vagrant up

   дождаться запуска вм 

   проверяем доступность хоста 

   $ ansible nginx -m ping

   затем запускаем наш playbook

   $ ansible-playbook nginx.yml


Подробное описание создания playbooks


   Создаем диеркторию Ansible и переходим в нее

   $ cd /Ansible

   создаем VAGRANTFILE согласно мануалу

   https://gist.github.com/lalbrekht/f811ce9a921570b1d95e07a7dbebeb1e

   запускаем наш Vagrant файл

   $ vagrant up

   Смотрим параметры хоста

   $ vagrant ssh-config
   
   создаем папку staging и в ней папку hosts 
   
   внутри папки hosts создаем invenory.yml со следющим содержимым исходя из параметров хоста 

   [web]
   nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_private_key_file=.vagrant/machines/nginx/virtualbox/private_key
   
   проверяем управляемость нашего хоста в Ansible
     
   $ ansible nginx -i staging/hosts -m ping
   
   создаем конфиг ansible.cfg для inventory файла и кладем его в папку Ansible, со след содержимым

   [defaults]
   inventory = staging/hosts
   remote_user = vagrant
   host_key_checking = False
   retry_files_enabled = False

   убираем ansible_user=vagrant из нашего файла inventory.yml

   проверяем доступность хоста заново

   $ ansible nginx -m ping

   проверяем установленное ядро нашего хоста

   $ ansible nginx -m command -a "uname -r"

   проверяем статус сервиса firewalld

   $ ansible nginx -m systemd -a name=firewalld

   в выводе нас интересует строка 

   "ActiveState": "inactive",
   
   установим пакет epel-release на наш хост

   $ ansible nginx -m yum -a "name=epel-release state=present" -b

   создаем файл epel.yml в директории Ansible со следуĀщим содержимым

   https://gist.githubusercontent.com/lalbrekht/d177f32c6f3b4b8ab2036b108671534c/raw/10ad40411222a37df0959e15193a70efba46d282/gistfile1.txt

   запустим наш playbook

   $ ansible-playbook epel.yml

   теперь создаим в директории Ansible nginx.yml на основе файла epel.yml со следующим содержимым

   https://gist.githubusercontent.com/lalbrekht/fe128f272b7be0b7a8c3c56bf96889e3/raw/8b79455416f351fdfcd513919554e25a18523789/gistfile1.txt

   просмотрим теги 
   
   $ ansible-playbook nginx.yml --list-tags

   создадим папку templates в директории Ansible

   внутри папки templates создадим файл шаблона nginx.conf.j2 со следующим содержимым

   https://gist.githubusercontent.com/lalbrekht/e76c659b1802512f7f860caefe738771/raw/f1dab76c1568db0ebc1e15f5aa9ff4ff651512ad/gistfile1.txt

   в файл nginx.yml добавим шаблон
   
   - name: NGINX | Create NGINX config file from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags:
      - nginx-configuration
   
   и переменную vars чтобы наш хост слушал на порту 8080

   - name: NGINX | Install and configure NGINX
       hosts: nginx
       become: true
     vars:
       nginx_listen_port: 8080

   теперь создадим handler и добавим notify к копированиĀ шаблона

   #handler
   
   handlers:
   - name: restart nginx
     systemd:
      name: nginx
      state: restarted
      enabled: yes

   - name: reload nginx
      systemd:
      name: nginx
      state: reloaded

   #notify

   - name: NGINX | Install NGINX package from EPEL Repo
      yum:
      name: nginx
      state: latest
      notify:
      - restart nginx
      tags:
      - nginx-package
      - packages

- name: NGINX | Create NGINX config file from template
      template:
      src: templates/nginx.conf.j2
      dest: /etc/nginx/nginx.conf
      notify:
      - reload nginx
      tags:
      - nginx-configuration

запустим наш playbook

$ ansible-playbook nginx.yml

в выводе должны увидеть следующее


PLAY [NGINX | Install and configure NGINX] **************************************

TASK [Gathering Facts] ***********************************************************
ok: [nginx]

TASK [NGINX | Install EPEL Repo package from standart repo] *******************
changed: [nginx]

TASK [NGINX | Install NGINX package from EPEL Repo] **************************
changed: [nginx]

TASK [NGINX | Create NGINX config file from template] **************************
changed: [nginx]

RUNNING HANDLER [restart nginx] ***********************************************
changed: [nginx]

RUNNING HANDLER [reload nginx] ***********************************************
changed: [nginx]

PLAY RECAP *********************************************************************
nginx : ok=6 changed=5 unreachable=0 failed=0



   Ссылка на мой образ https://app.vagrantup.com/darkghom/boxes/centos8-kernel6/versions/1.0/providers/virtualbox/unknown/vagrant.box








