
Phase1: K8s Setup
-----------------
Machine Type & Resources:
```properties
OS - CentOs 7.9,
VCPU - 2,
RAM - 2 GB,
Note: Setup done on EC2 Instances in AWS using kubeadm
```

Common Steps in Master & Worker Node:
1. Update host entry
```properties
cat <<EOF>> /etc/hosts
172.31.15.46 k8s-master01
172.31.10.68 k8s-worker01
172.31.8.105 k8s-worker02
EOF
```
2. Disable Selinux 
```shell
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

3. Enable br_netfilter Kernel Module
```shell
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

4. Disable Swap
```shell
swapoff -a && comment the swap line in /etc/fstab
```

5. Add K8s repo

6.Install the kubernetes packages kubeadm, kubelet, and kubectl
```shell
yum install -y kubelet kubeadm kubectl
```

7. Start & enable the docker & kubelet service
```shell
systemctl start docker && systemctl enable docker
systemctl start kubelet && systemctl enable kubelet
```

8. Reboot

Master Node Setup:

1. Initialize the K8s master cluster configuration
```shell
kubeadm init --apiserver-advertise-address=192.168.56.141 --pod-network-cidr=10.244.0.0/16
```
Here, --apiserver-advertise-address = master server IP and --pod-network-cidr = IP range for POD CIDR (using flannel network)

2. Create new '.kube' config directory and copy the configuration 'admin.conf'.
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

3. Deploying the flannel network using kubectl
```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

4. Verify node status
```shell
kubectl get nodes
NAME           STATUS   ROLES                  AGE     VERSION
k8s-master01   Ready    control-plane,master   3m20s   v1.22.2
```

Worker Nodes Setup:

1. Adding worker01 and worker02 to the k8s cluster.
```shell
kubeadm join 192.168.56.141:6443 --token fdx7ig.avo0af97223szobr --discovery-token-ca-cert-hash sha256:c40995dc93bd323824b1454e35e1cce259b60f8f66197a44746610d8e4ae2e85
```

2. Verify node status
```shell
kubectl get nodes
NAME           STATUS   ROLES                  AGE     VERSION
k8s-master01   Ready    control-plane,master   6m48s   v1.22.2
k8s-worker01   Ready    <none>                 59s     v1.22.2
k8s-worker02   Ready    <none>                 46s     v1.22.2
```

3. Verify Pod status
```shell
kubectl get pods -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-78fcd69978-ftbrq               1/1     Running   0          20m
kube-system   coredns-78fcd69978-x5jhx               1/1     Running   0          20m
kube-system   etcd-k8s-master01                      1/1     Running   0          20m
kube-system   kube-apiserver-k8s-master01            1/1     Running   0          21m
kube-system   kube-controller-manager-k8s-master01   1/1     Running   0          21m
kube-system   kube-flannel-ds-2wn5s                  1/1     Running   0          18m
kube-system   kube-flannel-ds-ctkl8                  1/1     Running   0          15m
kube-system   kube-flannel-ds-dsxbd                  1/1     Running   0          15m
kube-system   kube-proxy-fcdlr                       1/1     Running   0          15m
kube-system   kube-proxy-jp2bw                       1/1     Running   0          15m
kube-system   kube-proxy-tkdnc                       1/1     Running   0          20m
kube-system   kube-scheduler-k8s-master01            1/1     Running   0          21m
```

Phase2: Application Setup
-------------------------
Spring3Hibernate is a maven based java application and below are dependencies.
- [X] **Maven 3.X**
- [X] **Java 8**
- [X] **MySQL**
- [X] **Docker**

This application uses the below properties to connects with MySQL database.

```properties
database.driver=com.mysql.jdbc.Driver
database.url=jdbc:mysql://mysql.okts-test:3306/employeedb
database.user=root
database.password=password
hibernate.dialect=org.hibernate.dialect.MySQLDialect
hibernate.show_sql=true
hibernate.hbm2ddl.auto=update
upload.dir=c:/uploads
```

**Note:- The location of file is [src/main/resources/database.properties](src/main/resources/database.properties)**

#### Create Docker Image

```shell
docker build -t singhjaspreet291/spring3hibernate:latest -f Dockerfile .
```

### Push Docker Image

```shell
docker push singhjaspreet291/spring3hibernate:latest
```

Step3: Deploy application in K8s, use yaml files from K8s directory.

Step4: Deploy Logging & Monitoring Tool, use yaml files from observalibility directory.
