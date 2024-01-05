Kubernetes
=========

Role for deploy a k8s cluster

Requirements
------------

This playbook will only work if the inventory define the following groups:

- control_nodes: Nodes that will be control plane nodes in the k8s cluster
- control_main_node: ONLY ONE HOST should be present in this group. The node will be use as the main control node for the installation process.
- worker_nodes: Nodes that will be worker nodes in the k8s cluster

Role Variables
--------------

Mandatory:

- control_node_ip_address: Ip address of the control main node
- k8s_address_range: Network range for k8s to assign IPs to the different
containers and pods deployed

Optional:

- destroy_mode: If set to true it will purge the k8s cluster from the nodes

Other variables are defined, but they are not worth mentioning because they map to urls and versions that will not change in the short to medium term.

Dependencies
------------

No dependency with other roles

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - name: Kubernetes
      hosts: all
      force_handlers: true
      become: true
      roles:
        - role: kubernetes

License
-------

CC-BY-4.0
