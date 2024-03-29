## K3s with nginx ingress controller, Metallb loadbalancer and using a self-signed certificate

In this particular folder, we are trying to create a self-signed TLS certificate for our domain dkrp3.xyz using nginx ingress controller and Metallb as our loadbalancer. Metallb could be a considerable option for bare-metal Kubernetes and monitors Kubernetes services as type Loadbaalncer and assigns them <code>External IP</code>from the pool of the IP addresses mentioned inits ConfigMap.yaml file. As K3s by default comes loaded with traefik ingress controller and since here we want to work with nginx ingress controller, hence while installing k3s on our master node, we have to disable traefik by the command:

```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--no-deploy traefik --no-deploy servicelb" sh -s -
```

## Steps:

### 1. Start the K3s cluster and assign worker nodes

Once the command above is implemented in our K3s server, we can copy the node-token from /var/lib/rancher/k3s/server/node-token and use the command below in our worker nodes terminal:

```
curl -sfL https://get.k3s.io | K3S_URL=https://[IP_address_of_master]:6443 K3S_TOKEN="paste_the_copied_node-token" sh -
```

We can now check if our cluster has all the required components using 

```
k3s kubectl get nodes
```


### 2. Install Metallb

We can install Metallb using Helm or by the yaml file.I used the yaml file:

Apply the namespace yaml to create the metallb-system namespace

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
```

Apply the MetalLB manifest controller and speaker from the metallb.yaml file
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
```
These would result in craeting the components as shown in image below:

<img src="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/Wiki-page-images/Certificate_with_k3s%2Bnginx/2.PNG">
<p><i>Fig: Installing Metallb</i></p>


### 3. Create configmap file to insert IP address pool for Metallb

As mentioned above, Metallb refers to a configmap.yaml file to assign our dservices IP addresses from that pool. We can then apply the file as shown below:

<img src="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/Wiki-page-images/Certificate_with_k3s%2Bnginx/3.PNG">
<p><i>Fig: Create configmap for Metallb</i></p>

Here is my <code><a href="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/K3s/Certificate_with_k3s%2Bnginx/ConfigMap.yaml">ConfigMap.yaml</a></code>


### 4. Install nginx ingress controller

I used helm fro this purpose:

<img src="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/Wiki-page-images/Certificate_with_k3s%2Bnginx/4.PNG">
<p><i>Fig: Install nginx</i></p>

>***Note***
>This will also assign deafult Nginx TLS ecrtificate but we will adjust it with our self-signed certificate in teh further steps.

Now, we can check the services and we see the IP address attached to the Loadbalancer as shown in the below image.

<img src="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/Wiki-page-images/Certificate_with_k3s%2Bnginx/5.PNG">
<p><i>Fig: Loadbalancer IP address</i></p>



### 5. Install cert-manager

Since i already have the cert-manager namespace so we can move forward to installing CRD and cert-manager with the command below:

```
k3s kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.crds.yaml
```

```
k3s kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
```


### 6. Create resources that represent CA

Hereby in this step we will create a <code><a href="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/K3s/Certificate_with_k3s%2Bnginx/ClusterIssuer.yaml">ClusterIsuuer.yaml</a></code> and <code><a href="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/K3s/Certificate_with_k3s%2Bnginx/Issuer.yaml">Isuuer.yaml</a></code>

These both selfsigned Issuers does not represent a CA in this case and is rather signifies that a private key of the certificates will sign itself.

>***Note***
>These type of Issuer is very beneficial if we want to make a quick test for ad-hoc certificates. <a href="https://cert-manager.io/docs/configuration/selfsigned/">ReadMore</a>


### 7. Create an Ingress file

Through this <code><a href="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/K3s/Certificate_with_k3s%2Bnginx/Ingress.yaml">Ingress file</a></code>, we will be able to use selfsigned-issuer to issue te TLS certificate for nginx ingress. 

>***Note***
>ImplementationSpecific: With this path type, matching is up to the IngressClass. Implementations can treat this as a separate pathType or treat it identically to Prefix or Exact path types. <a href="https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types">Read More</a>


### 8. Check the certificate details:

We can also check the status and details of the issued certificate using the command as shown here:

<img src="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/Wiki-page-images/Certificate_with_k3s%2Bnginx/8.PNG">
<p><i>Fig: Certificate detail</i></p>

Just replace the end with the given <code><a href="https://github.com/dikshita-git/RP_Ingress_security-IPv4_and_IPv6/blob/main/K3s/Certificate_with_k3s%2Bnginx/Ingress.yaml">tls.secretname</a></code>



--------------------------------------------------------------------------------------------------------------------------------------------------------------
