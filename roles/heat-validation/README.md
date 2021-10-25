OpenStack Heat validation
==================

This role acts on an already deployed OpenStack environment to check whether or not it is possible to use its components by deploying a complete Heat stack.

Requirements
------------

An OpenStack environment accessible via a source file, i.e. the one that you get after installing via kolla:

```
[ansible@kolla-lab-0 ~]$ python3 -m venv venv-python3-kolla-ansible

[ansible@kolla-lab-0 ~]$ source venv-python3-kolla-ansible/bin/activate

(venv-python3-kolla-ansible) [ansible@kolla-lab-0 ~]$ pip3 install -r openstack/requirements.txt
Collecting ansible==2.8.20 (from -r openstack/requirements.txt (line 1))
...
...

(venv-python3-kolla-ansible) [ansible@kolla-lab-0 ~]$ source /etc/kolla/admin-openrc.sh

(venv-python3-kolla-ansible) [ansible@kolla-lab-0 ~]$ openstack service list
+----------------------------------+-------------+----------------+
| ID                               | Name        | Type           |
+----------------------------------+-------------+----------------+
| 678697d35d464a75b6b06020b8efbe65 | glance      | image          |
| 6f70fdfcccfc43a7b70936d573673356 | heat-cfn    | cloudformation |
| 8c3df480104b45688433c1dd6c3d591f | keystone    | identity       |
| 97e382e6e39a4b5a8b2326fa2124526b | nova_legacy | compute_legacy |
| b92951314f8b44508423401ba2b3575b | cinderv2    | volumev2       |
| ce08c0ffb1fa4d0e9b2dd1c4b9319225 | heat        | orchestration  |
| cef3feaa0a38488d941169379dd005f4 | neutron     | network        |
| e63a1c1713a84bc99e9f03f9e14bcc81 | nova        | compute        |
| f67dfac8eee6444c9493ceb27a11b6fe | placement   | placement      |
| fff0b94b2b984272887b72be01282cc7 | cinderv3    | volumev3       |
+----------------------------------+-------------+----------------+
```

## Heat validation

The defaults can be separated in three main blocks, the general ones:

```yaml
heat_validation_rc: "/etc/openstack/admin-openrc.sh"
heat_validation_work_dir: "./workdir"
heat_validation_logs_dir: "{{ heat_validation_work_dir }}/logs"
heat_validation_environment: "heat-validation-environment.yaml.j2"
heat_validation_template: "heat-validation-template.yaml.j2"
```

In which you can specify the rc file for accessing OpenStack and all the directories to store the various generated components (like the Heat yaml files taken from the Jinja templates).

The second block is related to the image used in the deployment:

```yaml
heat_validation_stack_name: 'openstack_heat_validation'
heat_validation_use_cinder: true
heat_validation_image_name: "heat-validation-cirros-0.5.2"
heat_validation_instance_image_format: "raw"
heat_validation_instance_image_location: "http://download.cirros-cloud.net/0.5.2/cirros-0.5.2-x86_64-disk.img"
heat_validation_instance_volume_gb: 1
heat_validation_cleanup: true
```

By default the Heat stack will be named "openstack_heat_validation". You can chose to use Cinder as the instance backend (defaults to true), how to call the image to be loaded in Glance, its location and format, how much space the instance will take and finally if you need to cleanup the environment and the downloaded files.

The third and last block is related to the network:

```yaml
heat_validation_private_net_name: "private-network"
heat_validation_private_subnet_name: "private-subnet"
heat_validation_private_net_cidr: "192.168.1.0/24"
heat_validation_public_net_type: "flat"
heat_validation_public_net_name: "public-network"
heat_validation_public_subnet_name: "public-subnet"
heat_validation_public_physical_network: "physnet1"
heat_validation_public_net_cidr: "10.0.0.0/24"
heat_validation_public_net_pool_start: "10.0.0.10"
heat_validation_public_net_pool_end: "10.0.0.20"
heat_validation_public_net_gateway: "10.0.0.254"
```

The role will deploy two network for the instance, expecting to expose one pingable IP on the public network.

**NOTE**: the test will fail if the exposed IP on the public network will not be pingable by the machine from which the playbook is launched, even if the Heat deployment will end in success.

Invoking the playbook via Ansible
--------------------------------------------------

Once requirements are met, playbook can be invoked as follows:

    [ansible@kolla-lab-0 ~]$ source venv-python3-kolla-ansible/bin/activate
    
    (venv-python3-kolla-ansible) [ansible@kolla-lab-0 ~]$ ansible-playbook \
      -e heat_validation_rc="/etc/kolla/admin-openrc.sh" \
      -e heat_validation_work_dir="/home/ansible/heat_validation_workdir" \
      -e heat_validation_heat_use_cinder=false \
      -e heat_validation_private_net_cidr="10.0.0.0/24" \
      -e heat_validation_public_net_cidr="192.168.144.0/24" \
      -e heat_validation_public_net_pool_start="192.168.144.200" \
      -e heat_validation_public_net_pool_end="192.168.144.250" \
      -e heat_validation_public_net_gateway="192.168.144.1" \
      openstack/heat-validation.yaml

Note that the variables above can be declared inside a config.yml file that can
be passed to the ansible-playbook command like this:

    (venv-python3-kolla-ansible) [ansible@kolla-lab-0 ~]$ ansible-playbook -e @./heat_validation_workdir/config.yaml openstack/heat-validation.yaml

The result will be the same.

License
-------

MIT

Author
------------------

Raoul Scarazzini <rasca@mmul.it>
