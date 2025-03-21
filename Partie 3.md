# 🌞 Constater le déploiement

```
PS C:\Users\1180024\OneDrive\Bureau\terraform> az vm list -o table
Name            ResourceGroup          Location    Zones
--------------  ---------------------  ----------  -------
tp2magueule-vm  TP2MAGUEULE-RESOURCES  westeurope
PS C:\Users\1180024\OneDrive\Bureau\terraform>
```

```
PS C:\Users\1180024\OneDrive\Bureau\terraform> terraform validate
Success! The configuration is valid.
```

```
PS C:\Users\1180024\OneDrive\Bureau\terraform> terraform state list
azurerm_linux_virtual_machine.main
azurerm_network_interface.internal
azurerm_network_interface.main
azurerm_network_interface_security_group_association.main
azurerm_network_security_group.ssh
azurerm_public_ip.pip
azurerm_resource_group.main
azurerm_subnet.internal
azurerm_virtual_network.main
```

# 🌞 Créer un plan Terraform avec les contraintes suivantes


Voici mon main.tf

Leux deux vm on était lancés, ma node1 a une ip publique et pas la node2, je me connecte en ssh sur la node1 pour me conner en ssh sur la node2.

```

provider "azurerm" {
  features {}
  subscription_id = "81b5d621-7779-43da-88e0-c03207d918e7"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "node1_nic" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_interface" "node2_nic" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "node1" {
  network_interface_id      = azurerm_network_interface.node1_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "node1" {
  name                = "${var.prefix}-node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_F2"
  admin_username      = "nikawx"
  network_interface_ids = [
    azurerm_network_interface.node1_nic.id
  ]
  admin_ssh_key {
    username   = "nikawx"
    public_key = file("C:/Users/1180024/.ssh/id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

resource "azurerm_linux_virtual_machine" "node2" {
  name                = "${var.prefix}-node2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_F2"
  admin_username      = "nikawx"
  network_interface_ids = [
    azurerm_network_interface.node2_nic.id
  ]
  admin_ssh_key {
    username   = "nikawx"
    public_key = file("C:/Users/1180024/.ssh/id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

```


## Connection Node1

```
PS C:\Users\1180024> ssh nikawx@20.126.40.251
The authenticity of host '20.126.40.251 (20.126.40.251)' can't be established.
ED25519 key fingerprint is SHA256:nf0hIHQvATKq5bNJihPXkd4PaU/f1OagEmSdCPHyXlY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.126.40.251' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)
```

## Connection Node2

```
nikawx@tp2magueule-node1:~$ ssh nikawx@10.0.2.4
The authenticity of host '10.0.2.4 (10.0.2.4)' can't be established.
ED25519 key fingerprint is SHA256:qzkDvY6bpNsDWwAXDhLJqm4EYvnQYxw8P5yLHpZzhtw.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.2.4' (ED25519) to the list of known hosts.

nikawx@tp2magueule-node2:~$

```

## BINGOOOOO


# 🌞 Intégrer la gestion de cloud-init


## Cloud-init.txt

```
#cloud-config
users:
  - name: customuser
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: docker
    shell: /bin/bash
    ssh_authorized_keys:
      - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC1guVS4qoz8U3CVLHl1wQLJnFW6z2CLFNKThz56fYcYKYtdgEMT/IHHe/tlz5tJivMoWRUDb80NHMJNMjQ9LAj6GTFgSgj5+LNANg/rzyq122X5Tefj6hYx7rZsoXWzxl6FixqVU5u3ybTFbYDiOqAX6ae3dCqFyUruIjpCmK1bxRprtSOSVXGUdBaK3a3tWHas/lT5qvQAv750vbVtMenH1GBEkJwwUywPrBnWeeuWjeaT4OU4+jrUGIeg9XJE5zCv+29/O+kxNmkOQKtXd/NnC2y1qbbrk9cDLh55OZnQBS6xpmyr6xgRfV8gK6A51w51hZlsEoeLo4Bh+97GkUtNK04EJtMeIFu6VSbBfZbRjNzCfM9svUVHZgLYVcWtec7iwu2aTRk3zHBpwoSUVN7r34kA49Acl88yzf/8DbdIuFlqEERwo5h0Ne1nGSbJZ7SpU7FzvdIS4rh/YLE2TAw0bV3vGajpUfD0zKgL9BNyz+3wxYFdV9MKpHeUqUaW57p3HtcW1gHghmkUeb+RspHYPb7yi4iIPWS4nfEAQ3R/QoLsyHANbdamdUD0tS6XFTQctq1fADxl0mN9DF9Z2tVhDavnLdU8yH6e+mraNCMQqN1xgPTOih1CskHogaWOTcE7AlVoHIwpxMGuQz/Bnm4w93mhWA33nXmrO/Lkk086Q== 1180024@LAPTOP-GFV4IM89"
    passwd: "admin"

package_update: true
package_upgrade: true

runcmd:
  - apt-get update && apt-get install -y docker.io
  - systemctl enable docker
  - systemctl start docker
  - docker pull alpine:latest
```


## Voici main.tf

```
variable "prefix" {
  description = "Prefixe pour nommer les ressources. Changez-le à chaque nouveau déploiement."
  type        = string
  default     = "testinfra"
}

variable "location" {
  description = "Emplacement Azure"
  type        = string
  default     = "West Europe"
}

provider "azurerm" {
  features {}
  subscription_id = "81b5d621-7779-43da-88e0-c03207d918e7"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "node1_nic" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "node1" {
  network_interface_id      = azurerm_network_interface.node1_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "node1" {
  name                = "${var.prefix}-node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_F2"
  admin_username      = "customuser"
  network_interface_ids = [
    azurerm_network_interface.node1_nic.id
  ]
  admin_ssh_key {
    username   = "customuser"
    public_key = file("C:/Users/1180024/.ssh/id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
  custom_data = filebase64("cloud-init.txt")
}
```

```
PS C:\Users\1180024> ssh customuser@20.126.40.251
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

customuser@tp2magueule-node1:~$ ls
customuser@tp2magueule-node1:~$ docker --version
Docker version 26.1.3, build 26.1.3-0ubuntu1~22.04.1


customuser@tp2magueule-node1:~$ groups
customuser docker
```



# 🌞 Proof !


## main.tf

```
provider "azurerm" {
  features {}
  subscription_id = "81b5d621-7779-43da-88e0-c03207d918e7"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "node1_nic" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "node1" {
  network_interface_id      = azurerm_network_interface.node1_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "node1" {
  name                = "${var.prefix}-node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_F2"
  admin_username      = "customuser"
  network_interface_ids = [
    azurerm_network_interface.node1_nic.id
  ]
  admin_ssh_key {
    username   = "customuser"
    public_key = file("C:/Users/1180024/.ssh/id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
  custom_data = filebase64("cloud-init.txt")
}
```

## variables.tf

```
variable "prefix" {
  description = "da prefix"
  default     = "tp2magueule"
}
variable "location" {
  description = "da location"
  default     = "West Europe"
}
```

## cloud-init.txt

```
#cloud-config
users:
  - name: customuser
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: docker
    shell: /bin/bash
    ssh_authorized_keys:
      - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC1guVS4qoz8U3CVLHl1wQLJnFW6z2CLFNKThz56fYcYKYtdgEMT/IHHe/tlz5tJivMoWRUDb80NHMJNMjQ9LAj6GTFgSgj5+LNANg/rzyq122X5Tefj6hYx7rZsoXWzxl6FixqVU5u3ybTFbYDiOqAX6ae3dCqFyUruIjpCmK1bxRprtSOSVXGUdBaK3a3tWHas/lT5qvQAv750vbVtMenH1GBEkJwwUywPrBnWeeuWjeaT4OU4+jrUGIeg9XJE5zCv+29/O+kxNmkOQKtXd/NnC2y1qbbrk9cDLh55OZnQBS6xpmyr6xgRfV8gK6A51w51hZlsEoeLo4Bh+97GkUtNK04EJtMeIFu6VSbBfZbRjNzCfM9svUVHZgLYVcWtec7iwu2aTRk3zHBpwoSUVN7r34kA49Acl88yzf/8DbdIuFlqEERwo5h0Ne1nGSbJZ7SpU7FzvdIS4rh/YLE2TAw0bV3vGajpUfD0zKgL9BNyz+3wxYFdV9MKpHeUqUaW57p3HtcW1gHghmkUeb+RspHYPb7yi4iIPWS4nfEAQ3R/QoLsyHANbdamdUD0tS6XFTQctq1fADxl0mN9DF9Z2tVhDavnLdU8yH6e+mraNCMQqN1xgPTOih1CskHogaWOTcE7AlVoHIwpxMGuQz/Bnm4w93mhWA33nXmrO/Lkk086Q== 1180024@LAPTOP-GFV4IM89"
    passwd: "admin"

package_update: true
package_upgrade: true

runcmd:
  - apt-get update && apt-get install -y docker.io
  - systemctl enable docker
  - systemctl start docker
  - docker pull alpine:latest
```

# 🌞 Intégrer la gestion de cloud-init

## Voici le main.tf

```
provider "azurerm" {
  features {}
  subscription_id = "81b5d621-7779-43da-88e0-c03207d918e7"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "node1_nic" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }

  # Règle pour accéder à Wiki.js sur le port 10101
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "AllowWikiJS"
    priority                   = 101
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "10101"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "node1" {
  network_interface_id      = azurerm_network_interface.node1_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "node1" {
  name                = "${var.prefix}-node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_F2"
  admin_username      = "customuser"
  network_interface_ids = [
    azurerm_network_interface.node1_nic.id
  ]
  admin_ssh_key {
    username   = "customuser"
    public_key = file("C:/Users/1180024/.ssh/id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
  custom_data = filebase64("cloud-init.txt")
}

```
## Voici le Cloud-init

```
#cloud-config
users:
  - default
  - name: nikawx
    sudo: true
    shell: /bin/bash
    groups: sudo, docker
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC1guVS4qoz8U3CVLHl1wQLJnFW6z2CLFNKThz56fYcYKYtdgEMT/IHHe/tlz5tJivMoWRUDb80NHMJNMjQ9LAj6GTFgSgj5+LNANg/rzyq122X5Tefj6hYx7rZsoXWzxl6FixqVU5u3ybTFbYDiOqAX6ae3dCqFyUruIjpCmK1bxRprtSOSVXGUdBaK3a3tWHas/lT5qvQAv750vbVtMenH1GBEkJwwUywPrBnWeeuWjeaT4OU4+jrUGIeg9XJE5zCv+29/O+kxNmkOQKtXd/NnC2y1qbbrk9cDLh55OZnQBS6xpmyr6xgRfV8gK6A51w51hZlsEoeLo4Bh+97GkUtNK04EJtMeIFu6VSbBfZbRjNzCfM9svUVHZgLYVcWtec7iwu2aTRk3zHBpwoSUVN7r34kA49Acl88yzf/8DbdIuFlqEERwo5h0Ne1nGSbJZ7SpU7FzvdIS4rh/YLE2TAw0bV3vGajpUfD0zKgL9BNyz+3wxYFdV9MKpHeUqUaW57p3HtcW1gHghmkUeb+RspHYPb7yi4iIPWS4nfEAQ3R/QoLsyHANbdamdUD0tS6XFTQctq1fADxl0mN9DF9Z2tVhDavnLdU8yH6e+mraNCMQqN1xgPTOih1CskHogaWOTcE7AlVoHIwpxMGuQz/Bnm4w93mhWA33nXmrO/Lkk086Q== 1180024@LAPTOP-GFV4IM89

apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88

packages:
  - docker-ce
  - docker-ce-cli
  - curl
  - ca-certificates
  - gnupg-agent
  - software-properties-common

write_files:
  - path: /docker-compose.yml
    content: |
      version: '3'
      services:
        db:
          image: postgres:15-alpine
          environment:
            POSTGRES_DB: wiki
            POSTGRES_PASSWORD: wikijsrocks
            POSTGRES_USER: wikijs
          logging:
            driver: none
          restart: unless-stopped
          volumes:
            - db-data:/var/lib/postgresql/data

        wiki:
          image: ghcr.io/requarks/wiki:2
          depends_on:
            - db
          environment:
            DB_TYPE: postgres
            DB_HOST: db
            DB_PORT: 5432
            DB_USER: wikijs
            DB_PASS: wikijsrocks
            DB_NAME: wiki
          restart: unless-stopped
          ports:
            - "10101:3000"
      volumes:
        db-data:

    owner: 'root:root'
    permissions: '0777'

runcmd:
  # Installation de Docker Compose
  - curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  - chmod +x /usr/local/bin/docker-compose
  - docker-compose --version  # Vérification de l'installation de Docker Compose
  
  # Démarrer Docker Compose pour Wiki.js
  - cd / && docker-compose -f /docker-compose.yml -p wikijs up -d
  - sleep 10  # Attendre 10 secondes pour que les conteneurs démarrent
  - docker logs wikijs  # Afficher les logs du conteneur Wiki.js
  - docker logs wikijs_postgres  # Afficher les logs du conteneur PostgreSQL

```

## CURL

```
nikawx@tp2magueule-v2-node:~$ curl 127.0.0.1:10101
<!DOCTYPE html><html><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1, maximum-scale=5"><meta name="theme-color" content="#1976d2"><meta name="msapplication-TileColor" content="#1976d2"><meta name="msapplication-TileImage" content="/_assets/favicons/mstile-150x150.png"><title>Wiki.js Setup</title><link rel="apple-touch-icon" sizes="180x180" href="/_assets/favicons/apple-touch-icon.png"><link rel="icon" type="image/png" sizes="192x192" href="/_assets/favicons/android-chrome-192x192.png"><link rel="icon" type="image/png" sizes="32x32" href="/_assets/favicons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/_assets/favicons/favicon-16x16.png"><link rel="mask-icon" href="/_assets/favicons/safari-pinned-tab.svg" color="#1976d2"><link rel="manifest" href="/_assets/manifest.json"><script>var siteConfig = {"title":"Wiki.js"}
</script><link type="text/css" rel="stylesheet" href="/_assets/css/setup.22871ffac1b643eed4d9.css"><script type="text/javascript" src="/_assets/js/runtime.js?1738531300"></script><script type="text/javascript" src="/_assets/js/setup.js?1738531300"></script></head><body><div id="root"><setup wiki-version="2.5.306"></setup></div></body></html>nikawx@tp2magueule-v2-node:~$
```