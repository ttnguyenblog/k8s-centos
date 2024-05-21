# Install K8s Centos

## Steps

* Install container runtime
* Install kubeadm, kubelet, kubectl
* Configuration Cgroup drive
* Configuration load balance
* Init Kubernetes Cluster
* Install CNI
* Add master node, worker node

## Setup

* VM Load Balancer: 10.0.2.4
* VM Master Node: 10.0.1.4
* VM Worker Node: 10.0.0.4

## Configuration

### 1. Configutation Load Balancer

Thiết lập một load balancer cho kubernetes. 
- Định nghĩa một upstream kubernetes chứa địa chỉ của các master node trong cụm.
- Định nghĩa server lắng nghe cổng 6443 và chuyển tiếp yêu cầu đến upstream kubernetes.
- `max_fails=3` chỉ định rằng sau 3 lần thất bại, server đó sẽ được xem là không hoạt động.
- `fail_timeout=30s` chỉ định rằng sau 30 giây, Nginx sẽ thử lại server đó.






