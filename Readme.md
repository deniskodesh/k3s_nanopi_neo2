## Overview

The goal of [this project][k3s_nanopi_neo2] is to provide an simple instruction for spin up k3s cluster for home automation

## Requirements 

1. At least two [NanoPi NEO2-LTS](https://www.friendlyarm.com/index.php?route=product/product&product_id=180)
2. For each device need Micro SD SDHC MicroSD Memory Card Class 10
3. Cooling system Heat Sink for NanoPi NEO/NEO2/Air/NEO Plus2 and recommended use funs 20x20x8мм or 20x20x6мм 5v
4. All devices need to be on one flat network desirable in one switch
5. Internet Access (Ethernet or Wifi, required to complete packages installation)


## OS installation

Download DietPi OS [image](https://dietpi.com/downloads/images/DietPi_NanoPiNEO2-ARMv8-Stretch.7z)

## Write to SD card using Windows
1. Extract the Linux image. Insert a TF card into a Windows PC and run the win32diskimager utility as administrator. On the utility's main window select your TF card's drive, the wanted image file and click on "write" to start flashing the TF card.

## Write to SD card using Linux

1. Find the device (eg: /dev/sdb1) you need to write to by using blkid or mount.
2. Double check that you have the correct dev path for your SD card (eg: /dev/sdb1).
3. Unmount the SD card and all its partitions by using umount /dev/sdb?.
4. Write the image using dd if=/path/to/DietPi_vXX.img of=/dev/sdb.


For more detailed information you can navigate on [link](https://dietpi.com/phpbb/viewtopic.php?f=8&t=9#p9)

## Prerequisites

1. Configure static IP for each node [link](https://dietpi.com/phpbb/viewtopic.php?f=8&t=14)
2. Configure hostname. For example for master node, echo master01>/etc/hostname.
3. Conficure hosts file. For example for master node :
```sh 
127.0.0.1    localhost.localdomain localhost
127.0.1.1    master01
192.168.1.200 master01.home.local
```

## k3s installation
  
### Master node
  
  1. Master node installation, curl -sfL https://get.k3s.io | sh -
  2. Check for Ready node, takes maybe 30 seconds, k3s kubectl get node

### Agent node

  1. For agent installation need token to join in cluster on master node run cat /var/lib/rancher/k3s/server/node-token
  2. Join agent in cluster url -sfL https://get.k3s.io |K3S_URL=https://MASTER_IP:6443 K3S_TOKEN=TOKEN sh -
  3. Check for Ready node, takes maybe 60 seconds, k3s kubectl get node

### Save config 
  
  1. On master node in home directory create .kube directory
  2. Copy config file to .kube directory cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

### Note
  
  More detailed information regarding installation, update or delete cluster you can find [link](https://github.com/rancher/k3s)

## Helm installation (in our case it will install in master node)
  
  1. Download arm helm package, curl -O https://get.helm.sh/helm-v2.14.1-linux-arm64.tar.gz
  2. Untar it, tar -zxvf helm-v2.14.1-linux-arm64.tar.gz
  3. Move binary, mv linux-arm64/helm /usr/local/bin/helm
  4. Create service account for tiller, kubectl create -f https://raw.githubusercontent.com/deniskodesh/k3s_nanopi_neo2/master/helm/service_account.yaml
  5. Install tiller to your running k3s cluster and set up local configuration, helm init --service-account tiller --history-max 200 --tiller-image=jessestuart/tiller:latest
 
## Storage class installation

  1. Create storage class,  kubectl create -f https://raw.githubusercontent.com/deniskodesh/k3s_nanopi_neo2/master/storageclass/storageclass.yaml
  2. Verify it, kubectl get storageclass

## Metallb installation
  
  1. Controller and speaker installation, kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
  2. [Config map](https://raw.githubusercontent.com/deniskodesh/k3s_nanopi_neo2/master/metallb/cm.yaml), be aware you need define your ip range, START_IP and END_IP
  3. Verify working state of metallb, create nginx svc with type LoadBalancer kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/tutorial-2.yaml

### Note
  
  More detailed information regarding metallb you can find [link](https://metallb.universe.tf/installation/)

## Home assistant installation

  1. helm install --name ha stable/home-assistant --set image.repository=homeassistant/raspberrypi3-64-homeassistant --set image.tag=0.95.4 --set persistence.storageClass=local-path --set configurator.enable=true
  2. Change type of svc homeassistant ClusterIP on LoadBalancer

  ### Note
  
  More detailed information regarding homeassisntant installation  you can find [link](https://github.com/helm/charts/tree/master/stable/home-assistant)

## MQTT installation

  1. Add helm repo, helm repo add smizy https://smizy.github.io/charts
  2. Download config file https://raw.githubusercontent.com/deniskodesh/k3s_nanopi_neo2/master/mqtt/mosquitto-values.yaml
  3. Install MQTT helm install smizy/mosquitto --name mqtt  \
                    --set image=eclipse-mosquitto:1.6.3 \
                    --set imagePullPolicy=Always \
                    --set persistence.enabled=true \
                    --set persistence.storageClass=local-path \
                    -f mosquitto-values.yaml  






