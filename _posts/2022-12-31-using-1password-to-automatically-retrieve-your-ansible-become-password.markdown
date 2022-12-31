---
layout: post
title:  "Using 1Password to automatically retrieve your Ansible become password for commands that require elevated privileges"
date:   2022-12-31 07:20:33 +0000
categories: posts
---
Are you tired of having to manually retrieve your `sudo` password and enter it into your terminal every time you run an Ansible playbook with `ansible_become_user`? I was, so I discovered a way to automatically retrieve it from 1Password and use it in Ansible playbooks.

Using the Ansible `lookup` plugin, you can retrieve the value of a 1Password item and use it in Ansible playbooks. The `lookup` plugin is a built-in plugin that allows you to retrieve the value of a variable from a file, database, or another external source. In this case, we will be retrieving the value of a 1Password item.

<!-- {% raw %} -->
```yaml
- name: "retrieve password for ITEM"
  debug:
    msg: "{{ lookup('onepassword', 'ITEM') }}"
```
<!-- {% endraw %} -->

For more information on the `lookup` plugin, see the [ansible-lookup-plugin].

This allows you to retrieve your `sudo` password from 1Password without having to manually enter it into your terminal when running `ansible-playbook main.yml -K` or reading locally from a plan text file or encrypted vault.

### Usage

Requirements

* A 1Password account - If you don’t have a 1Password account, you can sign up for a free account [here](https://1password.com/).

* The 1Password CLI tool - If you don’t have the 1Password CLI tool, you can download it [here](https://support.1password.com/command-line-getting-started/).

Create your password item in 1Password and give it a name. In this example, I will be using the name `ansible_become_pass`. You can name it whatever you want, but you will need to use the same name in your Ansible playbook.

```shell
$ op item create --category=password --title="ansible-become-password" --vault="Personal" \
  password="my-super-secret-password"
```

<!-- markdownlint-disable MD033 -->
<div class="note" style='position: relative; padding:1rem 1rem; margin-bottom: 0.5em; background-color:#cfe2ff; border: 1px solid transparent; border-radius: 0.25rem;'>
  <p><strong>Note:</strong> You can use a different category, but I recommend using the <code>password</code> category. You need to specify the <code>--vault</code> option if you want to store the item in a specific vault. If you don’t specify the <code>--vault</code> option, the item will be stored in the default vault.</p>
</div>

Next, you will need to add the lookup plugin as the value for the `ansible_become_pass` variable in your Ansible playbook. I set mine in the inventory file, but you can also set it in the vars section of your playbook (or a vars file). The lookup plugin will retrieve the value of the 1Password item with the same name as the variable. In this case, it will retrieve the value of the 1Password item named `ansible_become_pass`.

<!-- {% raw %} -->
```yaml
[all:vars]
ansible_become_pass="{{ lookup('onepassword', 'ansible_become_pass', errors='warn') | d(omit) }}"
```
<!-- {% endraw %} -->

That's it! when running your playbook, Ansible will automatically retrieve the value of the 1Password item and use it as the value for the `ansible_become_pass` variable. 1Password will prompt you to allow access to the Vault/Item when running the playbook.

<!-- markdownlint-disable MD014 -->
```shell
$ ansible-playbook -i inventory main.yml
```

[ansible-lookup-plugin]: https://docs.ansible.com/ansible/latest/plugins/lookup.html
