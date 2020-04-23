Vagrant.configure(2) do |config|

    config.ssh.insert_key = false

config.vm.define "k8s1" do |k8s1|
    config.vm.box = "hashicorp/bionic64"
    config.vm.box_version = "1.0.282"
    config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
    end
    k8s1.vm.hostname = "k8s1.lab.local"
    k8s1.vm.network "private_network", ip: "192.168.99.101"
    k8s1.vm.synced_folder "vagrant/", "/vagrant"
    k8s1.vm.provision "shell", inline:<<EOS

    echo "* Add hosts ..."
    echo "192.168.99.101 k8s1.lab.local k8s1" >> /etc/hosts
    echo "192.168.99.102 k8s2.lab.local k8s2" >> /etc/hosts
    echo "192.168.99.103 k8s3.lab.local k8s3" >> /etc/hosts

echo "* Install Prerequisites ..."
    sudo apt-get update && \
    sudo apt-get install bash-completion && \
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg2
    
    echo "* Cleanup Docker repository ..."
    sudo apt-get remove docker docker-engine docker.io containerd runc
    
    echo "* Add Docker’s official GPG key ..."
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    
    echo "* Add Docker repository ..."
    sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
        
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    
    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
  
    echo "* Install Docker ..."
    sudo apt-get update -y
    sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu && \
    sudo apt-mark hold docker-ce
    
    echo "* Install kubernetes packages ..."
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
  
    echo "* Start Docker ..."
    sudo systemctl enable docker
    sudo systemctl start docker
  
    echo "* Start Kubernetes ..."
    sudo systemctl enable kubelet
    sudo systemctl start kubelet
    
    systemctl daemon-reload
    systemctl restart kubelet
  
    echo "* Change some system settings ..."
    cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
EOF
    sudo sysctl --system
  
    echo "* Turn off the swap ..."
    sudo swapoff -a
    sudo sed -i '/ swap / s/^/#/' /etc/fstab
  
    echo "* Add vagrant user to docker group ..."
    sudo usermod -aG docker vagrant
    
    echo "* Initialize Kubernetes cluster ..."
    sudo kubeadm init --apiserver-advertise-address=192.168.99.101 --pod-network-cidr 10.244.0.0/16

    echo "* Copy configuration for root ..."
    mkdir -p /root/.kube
    sudo cp -i /etc/kubernetes/admin.conf /root/.kube/config
    sudo chown root:root /root/.kube/config

    echo "* Copy configuration for vagrant ..."
    mkdir -p /home/vagrant/.kube
    sudo cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
    sudo chown vagrant:vagrant /home/vagrant/.kube/config

    echo "* Install POD network plugin ..."
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

    echo "* Install Dashboard ..."
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml

    echo "* Create admin user ..."
    cat << EOF > /vagrant/dashboard-admin-user.yml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin-user
      namespace: kube-system
EOF
    echo "* Create role ..."
    cat << EOF > /vagrant/dashboard-admin-role.yml
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: admin-user
      namespace: kube-system
EOF

    echo "* Add both the user and role ..."
    kubectl apply -f /vagrant/dashboard-admin-user.yml
    kubectl apply -f /vagrant/dashboard-admin-role.yml

    echo "* Save the user token ..."
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') > /vagrant/admin-user-token.txt

    echo "* Create custom token ..."
    kubeadm token create abcdef.1234567890abcdef

    echo "* Save the hash to a file ..."
    openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //' > /vagrant/hash.txt
EOS
end

config.vm.define "k8s2" do |k8s2|
  config.vm.box = "hashicorp/bionic64"
  config.vm.box_version = "1.0.282"

  config.vm.provider "virtualbox" do |v|
  v.memory = 1024
  v.cpus = 1
  end

  k8s2.vm.hostname = "k8s2.lab.local"
  k8s2.vm.network "private_network", ip: "192.168.99.102"
  k8s2.vm.synced_folder "vagrant/", "/vagrant"
  k8s2.vm.provision "shell", inline: <<EOS

  echo "* Add hosts ..."
  echo "192.168.99.101 k8s1.lab.local k8s1" >> /etc/hosts
  echo "192.168.99.102 k8s2.lab.local k8s2" >> /etc/hosts
  echo "192.168.99.103 k8s3.lab.local k8s3" >> /etc/hosts
  
  echo "* Install Prerequisites ..."
    sudo apt-get update && \
    sudo apt-get install bash-completion && \
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg2
    
    echo "* Cleanup Docker repository ..."
    sudo apt-get remove docker docker-engine docker.io containerd runc
    
    echo "* Add Docker’s official GPG key ..."
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    
    echo "* Add Docker repository ..."
    sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
        
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    
    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
  
    echo "* Install Docker ..."
    sudo apt-get update -y
    sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu && \
    sudo apt-mark hold docker-ce
    
    echo "* Install kubernetes packages ..."
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
  
    echo "* Start Docker ..."
    sudo systemctl enable docker
    sudo systemctl start docker
  
    echo "* Start Kubernetes ..."
    sudo systemctl enable kubelet
    sudo systemctl start kubelet
    
    systemctl daemon-reload
    systemctl restart kubelet
  
    echo "* Change some system settings ..."
    cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
EOF
    sudo sysctl --system
  
    echo "* Turn off the swap ..."
    sudo swapoff -a
    sudo sed -i '/ swap / s/^/#/' /etc/fstab
  
    echo "* Add vagrant user to docker group ..."
    sudo usermod -aG docker vagrant
  
echo "* Join the worker node (k8s2) ..."
sudo kubeadm join 192.168.99.101:6443 --token abcdef.1234567890abcdef --discovery-token-ca-cert-hash sha256:`cat /vagrant/hash.txt`

EOS
end

config.vm.define "k8s3" do |k8s3|
  config.vm.box = "hashicorp/bionic64"
  config.vm.box_version = "1.0.282"

    config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 1
    end

    k8s3.vm.hostname = "k8s3.lab.local"
    k8s3.vm.network "private_network", ip: "192.168.99.103"
    k8s3.vm.synced_folder "vagrant/", "/vagrant"
    k8s3.vm.provision "shell", inline: <<EOS

echo "* Add hosts ..."
    echo "192.168.99.101 k8s1.lab.local k8s1" >> /etc/hosts
    echo "192.168.99.102 k8s2.lab.local k8s2" >> /etc/hosts
    echo "192.168.99.103 k8s3.lab.local k8s3" >> /etc/hosts
    
    echo "* Install Prerequisites ..."
    sudo apt-get update && \
    sudo apt-get install bash-completion && \
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common gnupg2
    
    echo "* Cleanup Docker repository ..."
    sudo apt-get remove docker docker-engine docker.io containerd runc
    
    echo "* Add Docker’s official GPG key ..."
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    
    echo "* Add Docker repository ..."
    sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
        
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    
    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
  
    echo "* Install Docker ..."
    sudo apt-get update -y
    sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu && \
    sudo apt-mark hold docker-ce
    
    echo "* Install kubernetes packages ..."
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
  
    echo "* Start Docker ..."
    sudo systemctl enable docker
    sudo systemctl start docker
  
    echo "* Start Kubernetes ..."
    sudo systemctl enable kubelet
    sudo systemctl start kubelet
    
    systemctl daemon-reload
    systemctl restart kubelet
  
    echo "* Change some system settings ..."
    cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
EOF
    sudo sysctl --system
  
    echo "* Turn off the swap ..."
    sudo swapoff -a
    sudo sed -i '/ swap / s/^/#/' /etc/fstab
  
    echo "* Add vagrant user to docker group ..."
    sudo usermod -aG docker vagrant

echo "* Join the worker node (k8s3) ..."
sudo kubeadm join 192.168.99.101:6443 --token abcdef.1234567890abcdef --discovery-token-ca-cert-hash sha256:`cat /vagrant/hash.txt`

EOS
end

end
