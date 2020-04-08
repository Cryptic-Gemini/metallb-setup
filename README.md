# metallb-setup
Guide to deploy metallb with minikube 

MetalLB
-------
If a Kubernetes cluster is installed on bare metal or on virtual machines, it’s missing an external load balancer. It’s a cloud network load balancer that comes with platforms such as Azure or GCM, that provides an externally-accessible IP address that sends traffic to the correct port on your cluster nodes. MetalLB is a load balancer designed to run on and to work with Kubernetes and it will allow you to use the type LoadBalancer when you declare a service.

A LoadBalancer service is the standard way to expose a service to the internet. It will give you a single IP address that will forward all traffic to your service.


Preparation:
-----------

If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

You can achieve this by editing kube-proxy config in current cluster:

                 
                            kubectl edit configmap -n kube-system kube-proxy

and set:

                           apiVersion: kubeproxy.config.k8s.io/v1alpha1
                           kind: KubeProxyConfiguration
                           mode: "ipvs"
                           ipvs:
                             strictARP: true



Install MetalLB on Minikube
---------------------------
MetalLB runs in two parts: a cluster-wide controller, and a per-machine protocol speaker. Install MetalLB by applying the manifest:


                    kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.9.3/manifests/namespace.yaml
                    kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.9.3/manifests/metallb.yaml
                    # On first install only
                    kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

This will deploy MetalLB to your cluster, under the metallb-system namespace. The components in the manifest are:

1- The metallb-system/controller deployment. This is the cluster-wide controller that handles IP address assignments.

2- The metallb-system/speaker daemonset. This is the component that speaks the protocol(s) of your choice to make the services reachable.

Service accounts for the controller and speaker, along with the RBAC permissions that the components need to function.
The installation manifest does not include a configuration file. MetalLB’s components will still start, but will remain idle until you define and deploy a configmap.

ConfigMap:
----------

apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.99.100/28



                                   kubectl apply -f metallb-configmap.yaml



Demo:
----

                             kubectl run webserver --image=nginx:alpine --port 80 


NAME        READY   UP-TO-DATE   AVAILABLE   AGE

webserver   1/1     1            1           3d4h


                             kubectl expose deployment webserver --type LoadBalancer


NAME         TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE

kubernetes   ClusterIP      10.96.0.1        <none>          443/TCP        28d

webserver    LoadBalancer   10.103.192.239   192.168.99.96   80:30911/TCP   16h


Configure DNS Locally Using /etc/hosts File in Linux:
----------------------------------------------------

This article explains, how to setup a local DNS using the hosts file (/etc/hosts) in Linux systems for local domain resolution or testing the website before taking live.

For example, you may want to test a website locally with a custom domain name before going live publicly by modifying the /etc/hosts file on your local system to point the domain name to the IP address of the local DNS server you configured.

The /etc/hosts is an operating system file that translate hostnames or domain names to IP addresses. This is useful for testing websites changes or the SSL setup before taking a website publicly live.

==> Now open the /etc/hosts file using your editor of choice as follows:

                                 $ sudo vi /etc/hosts


==> Then add the lines below to the end of the file:

                                 192.168.99.96    demo.metallb.com



---------------------------------------------------------------------------------------------------------------------------

ref: https://metallb.universe.tf/installation/

