Ansible playbooks for Kuryr PoC demo
====================================
Here you can find ansible playbooks to setup [Kuryr PoC with LBaaSv2 support](https://github.com/ivc/kuryr/tree/lbaasv2-poc) (based on [PoC by Midokura](https://github.com/midonet/kuryr/tree/k8s/kuryr)).

How to run
----------
1. Setup 3 hosts/vms with Ubuntu 14.04 named *kuryr-ctl* for master and *kuryr-cpu-1*/*kuryr-cpu-2* for minions
2. Edit kuryr.inv and replace IP addresses for ansible_host with addresses of the hosts from step 1
3. Run `ansible-playbook -i kuryr.inv devstack.play.yml` to prepare hosts for devstack setup (it will also copy your public rsa ssh key to hosts; to disable it, comment/delete the *copy public key* task in *roles/devstack/tasks/main.yml*)
4. Run `cd /opt/devstack; ./stack.sh` as *stack* user on *kuryr-ctl* and wait for it to complete
5. Run `cd /opt/devstack; ./stack.sh` as *stack* user on both *kuryr-cpu-1* and *kuryr-cpu-2* and wait for them to complete
6. Run `ansible-playbook -i kuryr.inv kuryr.play.yml` to setup K8s and Kuryr
7. Run `sudo service kubelet start` on all 3 hosts as *stack* user
8. Wait for K8s API server and nodes to become available (check with `kubectl get nodes` as *stack* user on *kube-ctl* until it shows 2 nodes)
9. Run `./start-raven.sh` as *stack* user on *kube-ctl*

How to test
-----------
1. Create a K8s deployment with `kubectl run nginx --image=nginx --replicas=2; kubectl expose deployment nginx --port=80 --target-port=80`
2. Get Pods' IPs with `kubectl describe pods nginx|grep ^IP` and service IP with `kubectl describe service nginx|grep ^IP`
3. Create a VM with `nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.nano --security-groups raven-default-sg --nic net-name=default testvm`
4. Connect to the VM (login/password is `cirros`/`cubswin:)`) and check that it can connect to the pods and to the service with `curl http://<Pod-or-Service-IP>`
