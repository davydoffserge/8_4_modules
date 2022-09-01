# devops-netology
## Домашнее задание к занятию "08.04 Создание собственных модулей"

### Подготовка к выполнению:

1. Создайте пустой публичных репозиторий в любом своём проекте: ```my_own_collection```
2. Скачайте репозиторий ansible: по любому удобному вам путиgit clone https://github.com/ansible/ansible.git
3. Зайдите в директорию ansible: ```cd ansible```
4. Создайте виртуальное окружение: ```python3 -m venv venv```
5. Активируйте виртуальное окружение: . Дальнейшие действия производятся только в виртуальном окружении ```. venv/bin/activate```
6. Установите зависимости ```pip install -r requirements.txt```
7. Запустить настройку окружения ```. hacking/env-setup```
8. Если все шаги прошли успешно - выйти из виртуального окружения ```deactivate```
9. Ваше окружение настроено, для того чтобы запустить его, нужно находиться в директории и выполнить конструкцию ```ansible. venv/bin/activate && . hacking/env-setup```

### Основная часть

Наша цель - написать собственный модуль, который мы можем использовать в своей role, через playbook. Всё это должно быть собрано в виде collection и отправлено в наш репозиторий.

1. В виртуальном окружении создать новый файл ```my_own_module.py```
2. Наполнить его содержимым:
```!/usr/bin/python

# Copyright: (c) 2018, Terry Jones <terry.jones@example.org>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = r'''
---
module: my_test

short_description: This is my test module

# If this is part of a collection, you need to use semantic versioning,
# i.e. the version is of the form "2.5.0" and not "2.4".
version_added: "1.0.0"

description: This is my longer description explaining my test module.

options:
    name:
        description: This is the message to send to the test module.
        required: true
        type: str
    new:
        description:
            - Control to demo if the result of this module is changed or not.
            - Parameter description can be a list as well.
        required: false
        type: bool
# Specify this value according to your collection
# in format of namespace.collection.doc_fragment_name
extends_documentation_fragment:
    - my_namespace.my_collection.my_doc_fragment_name

author:
    - Your Name (@yourGitHubHandle)
'''

EXAMPLES = r'''
# Pass in a message
- name: Test with a message
  my_namespace.my_collection.my_test:
    name: hello world

# pass in a message and have changed true
- name: Test with a message and changed output
  my_namespace.my_collection.my_test:
    name: hello world
    new: true

# fail the module
- name: Test failure of the module
  my_namespace.my_collection.my_test:
    name: fail me
'''

RETURN = r'''
# These are examples of possible return values, and in general should use other names for return values.
original_message:
    description: The original name param that was passed in.
    type: str
    returned: always
    sample: 'hello world'
message:
    description: The output message that the test module generates.
    type: str
    returned: always
    sample: 'goodbye'
'''

from ansible.module_utils.basic import AnsibleModule


def run_module():
    # define available arguments/parameters a user can pass to the module
    module_args = dict(
        name=dict(type='str', required=True),
        new=dict(type='bool', required=False, default=False)
    )

    # seed the result dict in the object
    # we primarily care about changed and state
    # changed is if this module effectively modified the target
    # state will include any data that you want your module to pass back
    # for consumption, for example, in a subsequent task
    result = dict(
        changed=False,
        original_message='',
        message=''
    )

    # the AnsibleModule object will be our abstraction working with Ansible
    # this includes instantiation, a couple of common attr would be the
    # args/params passed to the execution, as well as if the module
    # supports check mode
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    # if the user is working with this module in only check mode we do not
    # want to make any changes to the environment, just return the current
    # state with no modifications
    if module.check_mode:
        module.exit_json(**result)

    # manipulate or modify the state as needed (this is going to be the
    # part where your module will do what it needs to do)
    result['original_message'] = module.params['name']
    result['message'] = 'goodbye'

    # use whatever logic you need to determine whether or not this module
    # made any modifications to your target
    if module.params['new']:
        result['changed'] = True

    # during the execution of the module, if there is an exception or a
    # conditional state that effectively causes a failure, run
    # AnsibleModule.fail_json() to pass in the message and the result
    if module.params['name'] == 'fail me':
        module.fail_json(msg='You requested this to fail', **result)

    # in the event of a successful module execution, you will want to
    # simple AnsibleModule.exit_json(), passing the key/value results
    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```

3. Заполните файл в соответствии с требованиями ansible так, чтобы он выполнял основную задачу: module должен 
создавать текстовый файл на удалённом хосте по пути, определённом в параметре , с содержимым, определённым в параметре ```.pathcontent```
4. Проверьте модуль на исполняемость локально.
5. Напишите однозадачную пьесу и используйте модуль в нём.
6. Проверьте через playbook на идемпотентность.
7. Выйдите из виртуального окружения.
8. Инициализируйте новую коллекцию: ```ansible-galaxy collection init my_own_namespace.my_own_collection```
9. В данную коллекцию перенесите свой модуль в соответствующую директорию.
10. Single task playbook преобразуйте в single task role и перенесите в collection. У role должны быть по умолчанию всех параметров module
11. Создайте playbook для использования этой роли.
12. Заполните всю документацию по collection, выложите в свой репозиторий, поставьте тег на этот коммит.```1.0.0```
13. Создайте .tar.gz этой коллекции: в корневой директории collection.```ansible-galaxy collection build```
14. Создайте ещё одну директорию любого наименования, перенесите туда single task playbook и архив c collection.
15. Установите collection из локального архива: ```ansible-galaxy collection install <archivename>.tar.gz```
16. Запустите playbook, убедитесь, что он работает.
17. В ответ необходимо прислать ссылку на репозиторий с collection

Ответ:
1. Создаем новый файл
2. Копируем в него текст python
3. Заполняем его в соответствиями требованиями п.3
4. Проверяем исполняемость модуля локально.
![](8_4_activate.jpg)

![](8_4_hacking_env.jpg)

![](8_4_role.jpg)

![](8_4_module_1.jpg)

![](8_4_install_targz.jpg)

