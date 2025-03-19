# 🌞 Créez une VM depuis le Azure CLI

Installation de Azure CLI :

```
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi

az login

az interactive

vm create -g AZ -n Nikawx --image Ubuntu2204 --admin-username nikawx --ssh-key-values C:\Users\1180024\.ssh\id_rsa.pub
```

# 🌞 Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.

```
azureuser@Nikawx:~$ systemctl status walinuxagent
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/usr/lib/systemd/system/walinuxagent.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-03-18 11:00:55 UTC; 3min 34s ago
   Main PID: 968 (python3)
      Tasks: 7 (limit: 1004)
     Memory: 53.4M (peak: 56.5M)
        CPU: 2.758s
     CGroup: /azure.slice/walinuxagent.service
             ├─ 968 /usr/bin/python3 -u /usr/sbin/waagent -daemon
             └─1480 /usr/bin/python3 -u bin/WALinuxAgent-2.13.1.1-py3.9.egg -run-exthandlers
```

```
azureuser@Nikawx:~$ cloud-init status
status: done
```



# 🌞 Créez deux VMs depuis le Azure CLI


```
az vm create -g LAN -n Machine1 --image Ubuntu2204 --admin-username lan1 --ssh-key-values C:/Users/1180024/.ssh/id_rsa.pub --vnet-name LAN --subnet subnet

az vm create -g LAN -n Machine2 --image Ubuntu2204 --admin-username lan2 --ssh-key-values C:/Users/1180024/.ssh/id_rsa.pub --vnet-name LAN --subnet subnet
```

Les deux machines se ping

```
azureuser@Machine1:~$ ping 10.0.0.4
PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.
64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=0.029 ms
64 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=0.057 ms
64 bytes from 10.0.0.4: icmp_seq=3 ttl=64 time=0.093 ms
```
```
azureuser@Machine2:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=0.18 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=0.52 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=0.24 ms
```
