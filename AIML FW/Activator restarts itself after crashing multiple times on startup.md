:::danger
# <center> Activator restarts itself after crashing multiple times on startup</center>
:::
:::danger
**Goal：**
- [x] Resolved the problem of activator pod CrashLoopBackOff.
:::
**<center>Table of Content</center>**
[TOC]
# 1. A brief summary of the previous context

## 1-1. The Professor Lue's students recommendation
Regarding the incomplete installation issue with Kserve, discussions have already taken place with Professor Lue's students. ==They have indicated that resolving such DNS resolution issues is not as straightforward.== Their suggestion is that we need to first clarify the network topology of our laboratory, which should include details such as the laboratory server subnet, VM IPs, and intermediate bridging components. This is essential to understand how communication is established and to identify the root cause of the DNS resolution problem.
## 1-2. Difference in the installation environment
Because Professor Lue's students installed the AI/ML platform inside a virtual machine (VM), whereas we did not, this could lead to differences in the installation process.

# 2. The situation of the problem
**Install Kserve.**

```javascript=
./bin/install_kserve.sh
```

:::info
**What is Kserve ?**
Kserve is an open-source Kubernetes service designed to simplify and accelerate the deployment and management of machine learning models.
:::
:::danger
**Problem：Activator Pod keep in CrashLoopBackoff**

When I was installing Kserve, it got stuck at the "waiting for activator pod" step halfway through the installation. So, I checked the activator pod and found that it transitions from the "Running" state to "CrashLoopBackOff", continuously restarting in this loop. 

![](https://hackmd.io/_uploads/HkPWi54T3.png)

```javascript=
kubectl get pod -A
```

![](https://hackmd.io/_uploads/ByepQ1YT2.png)

The log indicates an error due to the inability to establish a network connection to the IP address 140.118.122.121 on port 8080, with a "connection refused" error. This likely signifies that a running program within the activity is attempting to establish a network connection to the specified IP address and port. However, for some reason (such as firewall settings, network configuration, the server not being operational, etc.), the connection is being denied.

```javascript=
kubectl logs activator-5754c5ff55-2hrvc -n knative-serving -f
```

![](https://hackmd.io/_uploads/SkTyplt63.png)

This seems to be a **Kubernetes DNS issue**, and it may require modifying the "domain.go" file from "resolv.conf". The specific cause of the error and the solution need further clarification.

At the same time, use this command to check autoscaler pod.
```javascript=
kubectl logs autoscaler-58fc8d57d5-9sbk4 -n knative-serving -f
```
There will a error like this figure.

![](https://hackmd.io/_uploads/ryHq5vsah.png)

:::

# 3. My solution method

- [x] The server's IP has been confirmed with the senior and there are no issues.

What I've tried：

1. ![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x)  Do not use the Kserve installation script provided by OSC. Download the latest version of kserve on my own.

2. ![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x) Reinstall the entire operating system.

3. ![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x) Revise the resolv.conf
4. ![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x) Attempt to customize the DNS service for pods without using the default settings.

5. ![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x) Install the ESXi system and set up the AI/ML platform within virtual machines.

6. ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x) Install the Windows 10 and set up the AI/ML platform within virtual machines.

7. ![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x) Install the Ubuntu 22.04 LTS OS and set up the AI/ML platform Revise nameserver resolved.conf to the 8.8.8.8

# 4. Customize the DNS service for pod (method 3 and 4)
## 4-1. Let's troubleshoot the DNS issue first.

Use the 'kubectl get pods' command to verify if the DNS pod is currently running.
```javascript=
kubectl get pods --namespace=kube-system -l k8s-app=kube-dns
```
![](https://hackmd.io/_uploads/rJqzvOyA3.png)

Use the 'kubectl get service' command to verify if the DNS service is up and running.
```javascript=
kubectl get svc --namespace=kube-system
```
![](https://hackmd.io/_uploads/HJ6tDd102.png)

Use the 'kubectl get endpoints' command to verify if the DNS endpoints are exposed.
```javascript=
kubectl get ep kube-dns --namespace=kube-system
```
![](https://hackmd.io/_uploads/B1V7KuJC3.png)

To confirm that the cluster IP of the kube-dns service is in the Pod's /etc/resolv.conf, please execute the following command inside the Pod's shell.
```javascript=
cat /etc/resolv.conf
```
![](https://hackmd.io/_uploads/HkPQSzG0n.png)

Check with the other pod and enter in to cat /etc/resolv.conf, you will see like this figure.

```javascript=
kubectl exec aiml-dashboard-74586d49d4-2pksv -n traininghost cat /etc/resolv.conf
kubectl exec metadata-writer-85576d4647-9dpq6 -n kubeflow cat /etc/resolv.conf
kubectl exec ml-pipeline-ui-694458fb88-dkxd8 -n kubeflow cat /etc/resolv.conf
```

![](https://hackmd.io/_uploads/H1cX3XMC2.png)

Attempt to enter the container to inspect and confirm the IP addresses of nameservers in the resolv.conf file.
```javascript=
kubectl exec -it activator-5754c5ff55-27bgm -n knative-serving -- sh
kubectl exec -it activator-5754c5ff55-27bgm -n knative-serving -- bash
kubectl exec -it activator-5754c5ff55-27bgm -n knative-serving -- bin/sh
kubectl exec -it activator-5754c5ff55-27bgm -n knative-serving -- bin/bash
```

![](https://hackmd.io/_uploads/BJK4Nd1Rh.png)

But, it do not let me to enter the pod.

Result：![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x)

## 4-2. Method 1 : Revise the resolv.conf
After researching extensively, I have come across someone online who has the exact same situation as mine. I tried to follow their method to address the issue, but ultimately, it still couldn't be resolved.

I asked the online friend for the specific solution, as shown in the image below：

![](https://hackmd.io/_uploads/Byq3s4z0h.png)

His solution was to modify the resolv.conf file on the host, removing the extra 'option' and 'search' entries.

```javascript=
vim /etc/resolv.conf
```
![](https://hackmd.io/_uploads/B1VRnEf0h.png)

To modify as the figure.

![](https://hackmd.io/_uploads/S1lsaVMA3.png)

```javascript=
kubectl delete pod activator-5754c5ff55-w627r -n knative-serving
reboot
```
It still has the problem.
```javascript=
kubectl describe pod activator-5754c5ff55-2fvwf -n knative-serving
```
![](https://hackmd.io/_uploads/rJ43kHzA3.png)
## 4-3. Method 2：Trying to use a custom resolv.conf instead of the default DNS service.
As an example from the following diagram, because the default DNS service is using the default Kubernetes DNS service, and each pod is using the same configuration.

```javascript=
kubectl get svc --namespace=kube-system
```
![](https://hackmd.io/_uploads/HJ6tDd102.png)

```javascript=
kubectl exec aiml-dashboard-74586d49d4-2pksv -n traininghost cat /etc/resolv.conf
kubectl exec metadata-writer-85576d4647-9dpq6 -n kubeflow cat /etc/resolv.conf
kubectl exec ml-pipeline-ui-694458fb88-dkxd8 -n kubeflow cat /etc/resolv.conf
```

![](https://hackmd.io/_uploads/H1cX3XMC2.png)

So, we need to custimized the DNS service.

**Step 1：** Create custimized resolv.conf file.
```javascript=
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.kube
```
**Step 2：** Export the YAML configuration of the Pod.

**First：I use this command to edit the configuration, but it dosen't work**
```javascript=
kubectl edit pod activator-5754c5ff55-vmxfz -n knative-serving
```
![](https://hackmd.io/_uploads/rkv3iLMR2.png)

**Second：So I use this method as the below.**
```javascript=
kubectl get pod <pod-name> -o yaml > your-pod-config.yaml
```
![](https://hackmd.io/_uploads/B1cJQUf02.png)

**Step 3：** Vim the pod like this figure to custimized the DNS service.

Find the DnsPolicy to revise as the follow, and this is origonal.

![](https://hackmd.io/_uploads/rJ8ArLM02.png)

To revise like this.

![](https://hackmd.io/_uploads/ByEDVUzAn.png)

**Step 4：** Apply the modified Pod configuration.
```javascript=
kubectl apply -f your-pod-config.yaml
```
**Step 5：** Result：![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x)

![](https://hackmd.io/_uploads/By0b_LGR2.png)

Clearly, this has resulted in other errors, and it's not ruled out that there may have been issues in my troubleshooting process. Therefore, I plan to proceed with the method 5.
# 5. ESXi system with AI/ML platform (method 5)
The ESXi system neet to use network card, but the PC does not have a corresponding network card as the figure.

![](https://hackmd.io/_uploads/H1twwYo0h.jpg)

I suspect it might be due to the version being too new **"(ESXi version 8.0.1)"** , which could be casusing this issue, so I tested with other versions **"(ESXi version 7.0.3)"** as follows.

![](https://hackmd.io/_uploads/rylUPKjC2.jpg)

I confirmed my motherboard specifications, which are **"Z790 AORUS ELITE AX"**, and the network card chipset used is **"Realtek's chipset"**. However, ESXi's system **does not support** this network card chipset. This link is the ESXi compatibility check list.

[VMware Compatibility Guide](https://www.vmware.com/resources/compatibility/search.php?deviceCategory=io&details=1&partner=198&releases=578&deviceTypes=6&page=1&display_interval=10&sortColumn=Partner&sortOrder=Asc)

Result：![](https://hackmd.io/_uploads/rkjVEfjT3.png =3%x)

# 6. Windows 10 with AI/ML platform (method 6)

This time, the entire AI/ML platform was installed on Windows 10, and the installation process went smoothly. I have also confirmed with Professor Lue's students that they have also installed it in a VM, and it seems that the network configuration is using the default settings, not bridged settings.

Check the Activator pod is full running or not.

![](https://hackmd.io/_uploads/SklVE2NACh.png)

The activoator pod is ready and running so the result is correct.
Result：![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)

# 7. Revise nameserver resolved.conf to the 8.8.8.8
```javascript=
sudo vim  /etc/systemd/resolved.conf
```

![](https://hackmd.io/_uploads/ry52X2pC3.png)

Revise this configuration.

![](https://hackmd.io/_uploads/rkT9V2pRn.png)

Use below command to restart.

Restart the systemd-resolved service.
```javascript=
sudo systemctl restart systemd-resolved
```
Start the systemd-resolved service
```javascript=
sudo systemctl enable systemd-resolved
```
Back up the systemd-resolved hosting file resolv.conf
```javascript=
sudo mv /etc/resolv.conf /etc/resolv.conf.bak
```
Rebuild the managed files
```javascript=
sudo ln -s /run/systemd/resolve/resolv.conf /etc/
```
Restart NetworkManager
```javascript=
sudo systemctl restart NetworkManager
```
![](https://hackmd.io/_uploads/r1hmr3pRh.png)

Choose the Wired to revise the DNS IP as 8.8.8.8 8.8.4.4

![](https://hackmd.io/_uploads/SyjSL3TRn.png)

Check the DNS configuration is revised or not.
```javascript=
cat /etc/resolv.conf
```

![](https://hackmd.io/_uploads/rkp7jVRR2.png)

Use this command to restart the PC.
```javascript=
reboot
```
Check the Activator pod is full running or not.

![](https://hackmd.io/_uploads/SklVE2NACh.png)

The activoator pod is ready and running so the result is correct.
Result：![](https://hackmd.io/_uploads/H11QhKjR2.png =3%x)


# 8. Below are the methods I've been researching：
1. [Activator restarts itself after crashing multiple times on startup](https://github.com/knative/serving/issues/5922)
    - Soluation：[move poll for kube-api to before logger creation](https://github.com/knative/serving/pull/5924)
2. [Activator Pod keep in CrashLoopBackoff](https://github.com/knative/serving/issues/4407)
    - Soluation：[Trim trailing dots from resolv.conf search line.](https://github.com/knative/serving/pull/4514/commits/f4ce9ff9f92ed845317d70a87d2e7f4311ae681f)
3. [Activator restarts constantly](https://github.com/knative/serving/issues/11544)
4. [ndots breaks DNS resolving](https://github.com/kubernetes/kubernetes/issues/64924)
