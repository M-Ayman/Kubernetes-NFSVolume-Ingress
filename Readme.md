# Deploy NFS Server


Open ports TCP:111, UDP: 111, TCP:2049, UDP:2049


yum install nfs-utils -y


Now we will share the NFS directory over the network "the private subnet of kubernetes"

#you need to change "172.31.32.0/24" to your private subnet of the cluster or use *

vim /etc/exports

  / 172.31.32.0/24(rw,sync,no_root_squash)

#Create a backup directory for mysql & wordpress files "for volumes"

  mkdir /{mysql,html}

  chmod  -R 755 /{mysql,html}

  chown nfsnobody:nfsnobody /{mysql,html}


systemctl enable rpcbind

systemctl enable nfs-server

systemctl enable nfs-lock

systemctl enable nfs-idmap

systemctl start rpcbind

systemctl start nfs-server

systemctl start nfs-lock

systemctl start nfs-idmap



# Creating Secret for mapping it inside MySQL and Wordpress as an Environment Variable

echo -n 'admin' | base64

-> copy the output and paste it inisde secret.yml

kubectl create -f secret.yml


# Deploy PersistentVolume for WordPress & MySQL

kubectl create -f pv-wordpress-mysql.yml


# Deploy PersistentVolumeClaims for Wordpress & MySQL

kubectl create -f pvc-wordpress.yml


# Deploy PersistentVolumeClaims for MySQL

kubectl create -f mysql-pvc.yml


# Deploy MySQL and it's Service

kubectl create -f mysql-deploy.yml

# Deploy WordPress and it's Service

kubectl create -f wordpress-deploy.yml

# Finally, deploy Ingress and point it to WordPress Service. you need to open port 80 on nodes for ingress

kubectl create -f ingress.yml


# Connect to WebApp

kubectl get ing   //it will take from 1-min to 3-mins to bring the the ip. copy it and paste on your browser


# Commands to troubleshooting/list all objects on you cluster

kubectl get pv,pvc      // list pv,pvc for MySQL & WordPress

kubectl get deployment  // list deployment for MySQL & WordPress

kubectl get pods

kubectl describe  pods pod-name   //pod-name from previous Command

Check NFS server and you will see your data under /mysql & /html

After installing Wordpress, try to delete one pod and it will mount data from /html


# kubectl  create -f .    //will deploy all resources







Create PersistentVolumeClaims and PersistentVolumes


Create a Secret for MySQL Password
Deploy MySQL
Deploy WordPress
