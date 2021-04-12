# zabbix-http-ansible

the purpose of this ansible automation is to easily ensure http/s health checks in zabbix from a yaml dictionary
It is also integrated with gitlab CI

## install

1. ensure ansible is installed, refer to [official install steps](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

2. ensure the required ansible modules: `ansible-galaxy collection install community.zabbix`

3. import the zabbix check template [template_http_health.xml](template_http_health.xml) in your zabbix server   
[refer to the official docs](https://www.zabbix.com/documentation/current/manual/xml_export_import/templates) if you don't know how to import 

## usage
edit the variable `health_checks` in the file [vars_checks.yaml](vars_checks.yaml) 
changes may be applied automatically via gitlab ci on git push

please refer to the [zabbix docs](https://www.zabbix.com/documentation/5.2/manual/web_monitoring#configuring_steps) for details about    
`check_timeout`, `check_returncode` or `check_searchstring`


```yaml
health_checks:
  - check_url:          "https://www.example.com/blog" # required; URL to check
    check_timeout:      "5s"       # optional; time to spend for check processing; default is set in vars_global.yaml
                                   # time suffixes are supported, e.g. 30s, 1m, 1h
    check_returncode:   "200"      # optional; list of expected HTTP status codes; default is set in vars_global.yaml
                                   # range is also supoorted, for example:  "200,201,210-299"
    check_searchstring: "Welcome"  # optional; regular expression pattern for searching the returned content 
                                   # e.g. "Welcome.*admin"

  - check_url:          "https://api.example.com/endpoint/search?query=token" # example to check search function
    check_searchstring: "Results for: token"

  - check_url:          "http://k11h.de"                                      # minimal example
```

apply the changes with this command
```bash
ansible-playbook main.yaml
```

## encrypt the zappix api password

please refer to the [official ansible vault docs](https://docs.ansible.com/ansible/latest/user_guide/vault.html) for more details

to create the encrypted variable with vault use this command 
```bash
ansible-vault encrypt_string 'apipass' -n "zabbix_api_pass"
```
the output should look like this:
```
zabbix_api_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32363638393165643333326432646263613264306338653932613265393130643233313734666139
          3438386335373732646632353834303561616234373261630a393462393639306135613062306337
          32323562633731653537653962623064386261623761636235333162336335343065616234646461
          6631653731323866650a626238393739346661303333313839346439643263386436386664363130
          6438
Encryption successful
```

## GitLab CI support

an example [.gitlab-ci.yaml](.gitlab-ci.yaml) is available.
ensure a variable named `ANSIBLE_VAULT_PASS` storing your ansible vault password.     
Please refer to the [official docs](https://docs.gitlab.com/ee/ci/variables/README.html)