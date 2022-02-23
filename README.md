# ansible-03-yandex-cloud

## Репозиторий с измененным `playbook` по ДЗ "08.03 Использование Yandex Cloud" - Захаров Сергей Николаевич
#### Описание функционала `ansible-playbook` согласно ДЗ показывает структуру фала `site.yml` с описанием задач, модулей, методов, переменных и их значений, при выполнении которых Ansible на управляемых нодах установит утилиты и настроит соответствующие конфигурации.
---
## Play №1. Установка Elasticsearch
```yml
---
- name: Install Elasticsearch
  hosts: elasticsearch
```
* `- name: Install Elasticsearch`    Указываем название Play, говорящее о его цели. Также этим мы начинаем описание элементов Play.
* `hosts: elasticsearch`            Где будет запускаться Play. `hosts` - на хостах. `elasticsearch`  - на хостах нашего инвентори, входящих в группу `elasticsearch`
###  Handler.
* Цель даного пункта - выполнять рестарт Elasticsearch при настуалении условий в других задачах
```yml
  handlers:
    - name: restart Elasticsearch
      become: true
      service:
        name: elasticsearch
        state: restarted
```
* `handlers:`  Задаем список хендлеров
* `- name: restart Elasticsearch`   Имя первого хендлера 
* `become: true`      Выполнять с повышением привелегий (под УЗ root по дефолту)
* `service:`      Указание на то, к какому сервису нужно обратиться
* `name: elasticsearch`   Имя сервиса
* `state: restarted`      Выполнить команду рестарта

```yml
  tasks:
```
* `tasks:`                Задаем список задач в составе Play (что будет выполняться)

###  Задача №1.
* Цель задачи - загрузить пакет `Elasticsearch` определенной версии и сохранить его в локальном каталоге на `manage_node`
```yml
    - name: "Download Elasticsearch's rpm"
      get_url:
        url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
        dest: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
      register: download_elastic
      until: download_elastic is succeeded
      
```
* `- name: "Download Elasticsearch's rpm"`  Название задачи для понимания ее назначения
* `get_url:`    Ипользовать модуль `get_url`
* `url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"`    Адрес ссылки для загрузки `Elasticsearch`
* `dest: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"`  Директория на `manage_node` для сохранения загруженного пакета 
* `register: download_elastic`      В переменную `download_elastic` записываем результат выполнения загрузки.
* `until: download_elastic is succeeded`    Цикл для выполнения условий успешной загрузки пакета.

###  Задача №2.
* Цель задачи - установить ране скачанный пакет `Elasticsearch`
```yml
    - name: Install Elasticsearch
      become: true
      yum:
        name: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
        state: present
```
* `- name: Install Elasticsearch`      Имя задачи.
* `become: true`      Выполнять с повышением привелегий (под УЗ root по дефолту)
* `yum:`    Запустить модуль `yum` для установки в систему утилиты из пакета RPM
* `name: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"`   Место расположения пакета.
* `state: present`    Условие гарантии того, что данный пакет будет установлен.
        
###  Задача №3.
* Цель задачи - перенос конфигурации `Elasticsearch` на `manage_node` и ее применение там.
```yml
    - name: Configure Elasticsearch
      become: true
      template:
        src: elasticsearch.yml.j2
        dest: /etc/elasticsearch/elasticsearch.yml
      notify: restart Elasticsearch
```
* `- name: Configure Elasticsearch`   Имя задачи.
* `become: true`      Выполнять с повышением привелегий (под УЗ root по дефолту)
* `template:`     Обращение к модулю `template` для создания на `manage_node` из файлов с шаблоном конфигурации .j2 файлов конфигураци .yml. Модуль ищет шаблоны  в директории `/template` на `control_node`
* `src: elasticsearch.yml.j2`   Указан файл - источник шаблона
* `dest: /etc/elasticsearch/elasticsearch.yml`    Указано место назначения файла конфигурации.
* `notify: restart Elasticsearch`   Выполнить условие - с помощью хендлера с именем `restart Elasticsearch`  перезапустить сервис  `Elasticsearch` на `manage_node`
      
## Play №2. Установка Kibana
```yml
- name: Install Kibana
  hosts: kibana
```
```yml
  handlers:
    - name: restart kibana
      become: true
      service:
        name: kibana
        state: restarted
```
```yml
  tasks:
```
```yml
    - name: "Download Kibana's rpm"
      get_url:
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm"
        dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
      register: download_kibana
      until: download_kibana is succeeded
```
```yml
    - name: Install Kibana
      become: true
      yum:
        name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
        state: present
      notify: restart kibana
```
```yml
    - name: Configure Kibana
      become: true
      template:
        src: kibana.yml.j2
        dest: /etc/kibana/kibana.yml
      notify: restart kibana
```
```yml
- name: Install Filebeat
  hosts: filebeat
```
```yml
  handlers:
    - name: restart filebeat
      become: true
      systemd:
        name: filebeat
        state: restarted
        enabled: true
```
```yml
  tasks:
```
```yml
    - name: "Download Filebeat's rpm"
      get_url:
        url: "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-{{ elk_stack_version }}-x86_64.rpm"
        dest: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
      register: download_filebeat
      until: download_filebeat is succeeded
```
```yml
    - name: Install Filebeat
      become: true
      yum:
        name: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
        state: present
      notify: restart filebeat
```
```yml
    - name: Configure Filebeat
      become: true
      template:
        src: filebeat.yml.j2
        dest: /etc/filebeat/filebeat.yml
      notify: restart filebeat
```
```yml
    - name: Set filebeat systemwork
      become: true
      command:
        cmd: filebeat modules enable system
        chdir: /usr/share/filebeat/bin
      register: filebeat_modules
      changed_when: filebeat_modules.stdout != 'Module system is alredy enabled'
```
```yml
    - name: Load Kibana dashboard
      become: true
      command:
        cmd: filebeat setup
        chdir: /usr/share/filebeat/bin
      register: filebeat_setup
      changed_when: false
      until: filebeat_setup is succeeded


```
