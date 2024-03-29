Посібник з розгортання ELK Stack з Beats
========================================

Цей посібник описує кроки, необхідні для налаштування стеку ELK (Elasticsearch, Logstash, Kibana) з агентами Beats (Filebeat, Metricbeat, Heartbeat) на інстансах AWS EC2 за допомогою Ansible для автоматизації. Налаштування призначене для демонстраційної версії (POC) без конфігурацій безпеки для спрощення. Додатково включено кроки для включення базових сповіщень моніторингу та налаштування безпеки для більш захищеного розгортання.

Передумови
----------

-   Аккаунт AWS: Переконайтеся, що у вас є аккаунт AWS з дозволами на створення інстансів EC2, груп безпеки та IAM ролей, якщо це необхідно.
-   Інстанси EC2:
    -   Як мінімум два інстанси EC2:
        -   Один для розміщення стеку ELK (Elasticsearch, Logstash, Kibana).
        -   Один для вашого застосунку, де будуть встановлені агенти Beats для збору даних.
-   Ansible: Машина управління Ansible, яка може бути вашою локальною машиною або іншим інстансом EC2 з встановленим Ansible. Вона буде використовуватися для автоматизації розгортання та конфігурації стеку ELK і агентів Beats.
-   Пара ключів SSH: Для доступу до ваших інстансів EC2.

Крок 1: Налаштування інстансів EC2
----------------------------------

1.  Запуск інстансів EC2: Створіть два інстанси EC2 через AWS Management Console. Використовуйте AMI Ubuntu Server для обох інстансів.

2.  Групи безпеки: Переконайтеся, що група безпеки для інстансу стеку ELK дозволяє вхідний трафік на порти 5601 (Kibana), 9200 (Elasticsearch HTTP) та 5044 (Logstash Beats input). Інстанс застосунку повинен дозволяти вихідний трафік на ці порти на інстансі стеку ELK.

3.  Пара ключів SSH: При створенні інстансів виберіть або створіть нову пару ключів SSH. Це необхідно для того, щоб Ansible міг підключатися і конфігурувати інстанси.

Крок 2: Підготовка середовища Ansible
-------------------------------------

1.  Встановлення Ansible: Якщо Ansible ще не встановлено, встановіть його на вашій управляючій машині.

    ```sql
    sudo apt update
    sudo apt install ansible
    ```

2.  Налаштування файлу хостів Ansible: Створіть або відредагуйте файл `/etc/ansible/hosts`, щоб включити ваші інстанси EC2 під відповідними групами. Приклад:

    ```ini
    [elk]
    elk_instance ansible_host=ваша_публічна_ip_адреса_інстансу_elk

    [application]
    app_instance ansible_host=ваша_публічна_ip_адреса_інстансу_додатку
    ```

3.  Конфігурація SSH: Переконайтеся, що Ansible може підключатися до ваших інстансів EC2 через SSH. Вам може знадобитися налаштувати права на файл приватного ключа та використовувати пересилання агента SSH або вказати приватний ключ у вашій конфігурації Ansible.

Крок 3: Розгортання стеку ELK
-----------------------------

1.  Створення Ansible Playbook (`elk_setup.yml`): Цей плейбук повинен встановлювати та конфігурувати Elasticsearch, Logstash і Kibana на інстансі стеку ELK.

2.  Запуск плейбуку:


    ansible-playbook elk_setup.yml

Крок 4: Розгортання агентів Beats
---------------------------------

1.  Налаштування Filebeat (`filebeat_install.yml`): Використовуйте цей плейбук для встановлення та конфігурації Filebeat на інстансі стеку ELK та інстансі застосунку.

2.  Налаштування Metricbeat (`metricbeat_install.yml`): Аналогічно Filebeat, встановіть та налаштуйте Metricbeat для моніторингу системних метрик.

3.  Налаштування Heartbeat (`heartbeat_install.yml`): Встановіть та налаштуйте Heartbeat на інстансі стеку ELK для моніторингу часу роботи системи.

4.  Запуск плейбуків:

    ```bash
    ansible-playbook filebeat_install.yml
    ansible-playbook metricbeat_install.yml
    ansible-playbook heartbeat_install.yml
    ```

Крок 5: Налаштування моніторингу та сповіщень
---------------------------------------------

-   Визначте критерії моніторингу та сповіщень в Elasticsearch або Kibana на основі порогів використання диску, пам'яті та CPU.
