###Ubuntu Virtual Machine for NGINX Microservices March 2022 Labs###

###Preface###

Since I didn't have access to the lab environment in UDF, I decided to setup and run my own environment in VMware Workstation, so that I can run the Microservices March Labs at my own pace. This guide should help anyone to setup their own Ubuntu VM to run the labs in your environment. I won't cover the installation of the Ubuntu OS itself. One hint: on my Ubuntu host I disabled swap.

Additionally consider these setup instructions for CoreDNS and NGINX from the devcentral github repo.

###Software and Versions used###

    Ubuntu 20.04 LTS
    Docker 20.10.13
    kubectl 1.23.4
    Helm 3.8.1
    Minikube v1.25.2

###Step 1: Install Docker###
Set up the repository

Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
daniel@ubuntu:~$ sudo apt update
daniel@ubuntu:~$ sudo apt-get install \
 ca-certificates \
 curl \
 gnupg \
 lsb-release
```

Add Docker’s official GPG key:
```
daniel@ubuntu:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Use the following command to set up the stable repository.
```
daniel@ubuntu:~$  echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
###Install Docker Engine###

Update the apt package index, and install the latest version of Docker Engine and containerd:
```
daniel@ubuntu:~$ sudo apt update
daniel@ubuntu:~$ sudo apt install docker-ce docker-ce-cli containerd.io
```
Add your user to the 'docker' group.
```
daniel@ubuntu:~$ sudo usermod -aG docker $USER && newgrp docker
```
###Step 2: Install kubectl using native package management###
```
Download the Google Cloud public signing key.

daniel@ubuntu:~$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

Add the Kubernetes apt repository.
```
daniel@ubuntu:~$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update apt package index with the new repository and install kubectl.
```
daniel@ubuntu:~$ sudo apt update
daniel@ubuntu:~$ sudo install kubectl
```

###Step 3: Install Helm from script###

Download and run the installer script to install Helm
```
daniel@ubuntu:~$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
daniel@ubuntu:~$ chmod 700 get_helm.sh
daniel@ubuntu:~$ ./get_helm.sh
```

###Step 4: Install Minikube###

Download Minikube as a static binary
```
daniel@ubuntu:~$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
&& chmod +x minikube
daniel@ubuntu:~$ sudo cp minikube /usr/local/bin && rm minikube
```

Move the Minikube binary to your /usr/local/bin location.
```
daniel@ubuntu:~$ sudo cp minikube /usr/local/bin && rm minikube
```

Step 5: Start your local minikube cluster
```
minikube start --memory=4G
```

From here on you can follow the lab guides.
###Problems observed###

The lab guide mentions a couple of times that you can run commands like: minikube service podinfo and this will display a chart like the one below and it would open the service in your browser.
```
$ minikube service podinfo
|-----------|---------|-------------|---------------------------|
| NAMESPACE |  NAME   | TARGET PORT |            URL            |
|-----------|---------|-------------|---------------------------|
| default   | podinfo |          80 | http://192.168.49.2:31190 |
|-----------|---------|-------------|---------------------------|
🏃  Starting tunnel for service podinfo.
|-----------|---------|-------------|------------------------|
| NAMESPACE |  NAME   | TARGET PORT |          URL           |
|-----------|---------|-------------|------------------------|
| default   | podinfo |             | http://127.0.0.1:51546 |
|-----------|---------|-------------|------------------------|
🎉  Opening service default/podinfo in default browser...
```

I could not get this working. However, what worked out for me was the following:
```
daniel@ubuntu:~$ minikube service --all
😿  service default/kubernetes has no node port
|-----------|------------|--------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT  |            URL            |
|-----------|------------|--------------|---------------------------|
| default   | kubernetes | No node port |
| default   | podinfo    |           80 | http://192.168.49.2:30009 |
|-----------|------------|--------------|---------------------------|
```

And then I could open the app in the browser with the URL displayed here in this chart.
