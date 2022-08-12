


1. /usr/local/bin/k3s-uninstall.sh

## 2. Install k3s + Config For IPv4 address only:
*************************

curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=192.168.0.0/16 --service-cidr=10.43.0.0/16 --disable-network-policy  --disable=traefik" sh -

3. kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml

4. kubectl create -f custom-resources.yaml

5. k3s kubectl get pods

6. cd /etc/cni/net.d/calico-kubeconfig  (Calico kubeconfig file istorage)
 
8. k3s kubectl get pod --all-ns

9. kubectl version -o yaml

10. k3s kubectl describe node server

11. curl https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml -O   (To show details of calico rcustom resources yaml file)

12. To taint nodes:
*******************

- k3s kubectl describe node "node_name" | grep -A5 Taints

- k3s kubectl taint node "node_name" "Paste_details_next_to_Unschedulable/Schedulable_from_result_of_above_command"

13. To untaint nodes:
*******************

- k3s kubectl taint node "node_nam" "Paste_details_next_to_Unschedulable/Schedulable_from_result_of_above_command"-
 
13. curl  https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml > calico.yaml   

(to copy contents from  https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml to calico .yaml)



## 14. Install k3s + Config For IPv4,IPv6 address :
**************************************

curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=192.168.0.0/16,2001::00/68 --service-cidr=10.43.0.0/16 --disable-network-policy --node-ip=2001:638:408:200:: 100:180 --disable=traefik" sh -
