# 🌞 Tester cloud-init

utilisez l'option --custom-data C:\Users\1180024\.ssh\cloud-init.txt

mon fichier cloud-init.txt

```
#cloud-config
users:
  - default
  - name: LAN
    sudo: false
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC1guVS4qoz8U3CVLHl1wQLJnFW6z2CLFNKThz56fYcYKYtdgEMT/IHHe/tlz5tJivMoWRUDb80NHMJNMjQ9LAj6GTFgSgj5+LNANg/rzyq122X5Tefj6hYx7rZsoXWzxl6FixqVU5u3ybTFbYDiOqAX6ae3dCqFyUruIjpCmK1bxRprtSOSVXGUdBaK3a3tWHas/lT5qvQAv750vbVtMenH1GBEkJwwUywPrBnWeeuWjeaT4OU4+jrUGIeg9XJE5zCv+29/O+kxNmkOQKtXd/NnC2y1qbbrk9cDLh55OZnQBS6xpmyr6xgRfV8gK6A51w51hZlsEoeLo4Bh+97GkUtNK04EJtMeIFu6VSbBfZbRjNzCfM9svUVHZgLYVcWtec7iwu2aTRk3zHBpwoSUVN7r34kA49Acl88yzf/8DbdIuFlqEERwo5h0Ne1nGSbJZ7SpU7FzvdIS4rh/YLE2TAw0bV3vGajpUfD0zKgL9BNyz+3wxYFdV9MKpHeUqUaW57p3HtcW1gHghmkUeb+RspHYPb7yi4iIPWS4nfEAQ3R/QoLsyHANbdamdUD0tS6XFTQctq1fADxl0mN9DF9Z2tVhDavnLdU8yH6e+mraNCMQqN1xgPTOih1CskHogaWOTcE7AlVoHIwpxMGuQz/Bnm4w93mhWA33nXmrO/Lkk086Q== 1180024@LAPTOP-GFV4IM89

```



# 🌞 Vérifier que cloud-init a bien fonctionné

```
vm create --name nikawx  -g LAN --image Ubuntu2204 --image Ubuntu2204 --admin-username nikawx --custom-data C:\Users\1180024\.ssh\cloud-init.txt
```
Aller dans fichier host pour supprimer la ligne afin d'éviter l'erreur Man In The Middle (ligne 52 pour moi)
```
PS C:\Users\1180024> ssh nikawx@52.184.65.98
The authenticity of host '52.184.65.98 (52.184.65.98)' can't be established.
ED25519 key fingerprint is SHA256:ja1zklCbq2JCeqFjTvBmRf21Y5iL0qaklVICDFkokGk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '52.184.65.98' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)
```


# 🌞 Utilisez cloud-init pour préconfigurer la VM :

```

#cloud-config
packages:
  - docker.io

users:
  - name: nikawx
    passwd: "Toto24230230!"  
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC1guVS4qoz8U3CVLHl1wQLJnFW6z2CLFNKThz56fYcYKYtdgEMT/IHHe/tlz5tJivMoWRUDb80NHMJNMjQ9LAj6GTFgSgj5+LNANg/rzyq122X5Tefj6hYx7rZsoXWzxl6FixqVU5u3ybTFbYDiOqAX6ae3dCqFyUruIjpCmK1bxRprtSOSVXGUdBaK3a3tWHas/lT5qvQAv750vbVtMenH1GBEkJwwUywPrBnWeeuWjeaT4OU4+jrUGIeg9XJE5zCv+29/O+kxNmkOQKtXd/NnC2y1qbbrk9cDLh55OZnQBS6xpmyr6xgRfV8gK6A51w51hZlsEoeLo4Bh+97GkUtNK04EJtMeIFu6VSbBfZbRjNzCfM9svUVHZgLYVcWtec7iwu2aTRk3zHBpwoSUVN7r34kA49Acl88yzf/8DbdIuFlqEERwo5h0Ne1nGSbJZ7SpU7FzvdIS4rh/YLE2TAw0bV3vGajpUfD0zKgL9BNyz+3wxYFdV9MKpHeUqUaW57p3HtcW1gHghmkUeb+RspHYPb7yi4iIPWS4nfEAQ3R/QoLsyHANbdamdUD0tS6XFTQctq1fADxl0mN9DF9Z2tVhDavnLdU8yH6e+mraNCMQqN1xgPTOih1CskHogaWOTcE7AlVoHIwpxMGuQz/Bnm4w93mhWA33nXmrO/Lkk086Q== 1180024@LAPTOP-GFV4IM89


    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    groups: [docker]

runcmd:
  - docker pull alpine:latest  


systemctl:
  enabled:
    - docker

```


## BINGOOOOOO 

```
PS C:\Users\1180024> ssh nikawx@52.184.65.98
The authenticity of host '52.184.65.98 (52.184.65.98)' can't be established.
ED25519 key fingerprint is SHA256:cDa/azVBs7hRfO+ndNYz3HyvdD3jZ2z5A9C6l4mqNEY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '52.184.65.98' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

```