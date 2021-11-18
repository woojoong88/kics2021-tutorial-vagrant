# SPDX-FileCopyrightText: 2020-present Open Networking Foundation <info@opennetworking.org>
#
# SPDX-License-Identifier: Apache-2.0

Vagrant.configure("2") do |config|
    config.vm.define "sdran-vm" do |sdran|
        sdran.vm.box = "ubuntu/bionic64"
        sdran.vm.hostname = "sdran-vm"
        sdran.vm.provider "virtualbox" do |v|
            v.cpus = 4
            v.memory = 8192
            v.name = "sdran-vm"
        end

        sdran.vm.provision "shell", inline: <<-SHELL
            sudo apt update -y
            sudo apt install -y software-properties-common python3-pip jq httpie ipvsadm ethtool
            sudo bash -c 'echo net.ipv4.ip_forward = 1 >> /etc/sysctl.conf'
            git clone https://github.com/kubernetes-incubator/kubespray.git -b release-2.14
            sudo pip3 install virtualenv
            cd kubespray
            virtualenv venv
            source venv/bin/activate
            pip3 install -r requirements.txt
            ansible-playbook -b -i inventory/local/hosts.ini \
            -e "{'override_system_hostname' : False, 'disable_swap' : True}" \
            -e "{'docker_version' : 19.03}" \
            -e "{'docker_iptables_enabled' : True}" \
            -e "{'kube_version' : v1.18.9}" \
            -e "{'kube_network_plugin_multus' : True, 'multus_version' : stable, 'multus_cni_version' : 0.3.1}" \
            -e "{'kube_proxy_metrics_bind_address' : '0.0.0.0:10249'}" \
            -e "{'kube_pods_subnet' : 192.168.0.0/17, 'kube_service_addresses' : 192.168.128.0/17}" \
            -e "{'kube_apiserver_node_port_range' : 2000-36767}" \
            -e "{'kubeadm_enabled': True}" \
            -e "{'kube_feature_gates' : [SCTPSupport=True]}" \
            -e "{'kubelet_custom_flags' : [--allowed-unsafe-sysctls=net.*]}" \
            -e "{'dns_min_replicas' : 1}" \
            -e "{'helm_enabled' : True, 'helm_version' : v3.2.4}" \
            cluster.yml
            mkdir -p /home/vagrant/.kube
            sudo cp -f /etc/kubernetes/admin.conf /home/vagrant/.kube/config
            sudo chown vagrant:vagrant /home/vagrant/.kube/config
            kubectl wait pod -n kube-system --for=condition=Ready --all
            kubectl get namespace riab 2> /dev/null || kubectl create namespace riab
        SHELL
    end
end