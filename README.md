# vm_provisioning
Mise en place d'une architecture Iaac de création des VM Proxmox à partir de Terraform 

## Pre-réquis:
`
Proxmox VE 9.1, Terraform 1.14.0
`
et un accès root adminProxmox

- Si vous utiliser la version gratuite de proxmox, desactiver le dépôt entreprise afin de faire de mise à jour de votre serveur proxmox(apt-get upgrade && apt-get update).
C'est dans node(Host-003) -> Mise à jour -> Dépôts

[dépôts]<img width="1716" height="602" alt="image" src="https://github.com/user-attachments/assets/b514292b-a879-4434-a2f8-d4c58962c453" />


- Nous allons commencer par créer des template proxmox pour des VMs debian et ubuntu que nous utiliserons dans terraform pour créer des VMs.
  Pour cela, nous sur des images cloud-init 

## Ubuntu
- Récupération de l'image et importation sur Proxmox
- coller l'url dans Proxmox à Host-003/local(Host-003)/Images ISO/ `Télécharger depuis l'url` https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
- Création d'une VM (en ligne de commande): `qm create 9000 --name "ubuntu-2204-cloudinit-template" --memory 4096 --net0 virtio,bridge=vmbr0 `
- Importation de l'image téléchargée en tant que disk pour la VM 9000
```(bash)
 qm importdisk 9000 /var/lib/vz/template/iso/jammy-server-cloudimg-amd64.img  local-lvm
 ```
[emplacement des disk pr default sur proxmox]<img width="702" height="622" alt="image" src="https://github.com/user-attachments/assets/b3e8eb06-cf7c-455c-8a38-786962e0c729" />


- On configure la VM pour démarrer sur le disk dont le contrôleur est scsi
  ```
  qm set 9000 --boot c --bootdisk scsi0
  ```
- On ajoute un port Serie (prérequis pour utiliser Cloud-Init)

```
 qm set 9000 --serial0 socket --vga serial0
```
- On ajoute un volume dédié à Cloud-Init :
```
 qm set 9000 --ide2 local-lvm:cloudinit
```
- On transforme la machine en template
```
qm template 9000
```
- On doit activer le qemu-guest-agent dans le template(nécessaire pour utiliser terraform)
  
[Activation quemu agent]<img width="1193" height="527" alt="image" src="https://github.com/user-attachments/assets/595d1208-bae3-4500-827b-161865ddb8d2" />


[template Ubuntu]<img width="1205" height="356" alt="image" src="https://github.com/user-attachments/assets/aeb5b410-0039-45da-8c30-e2ebd83600ae" />

## Configuration de la clé API
- Création d'un utilisateur terraform
[terraform user]<img width="1262" height="529" alt="Capture d&#39;écran 2025-12-13 101955" src="https://github.com/user-attachments/assets/112ff4b0-b768-4d51-86d0-1cb5b41fcb9b" />

- Création de la clé API associé à l'utilisateur terraform exemple(d8661fba-1c3e-45eb-b6d5-cef8df6dafd1)(terraform@pam!terraform_token)
[token terraform]<img width="1223" height="528" alt="image" src="https://github.com/user-attachments/assets/7d865b46-24dc-4a0d-8f64-1a3c931c9766" />

- Donnons les permissions à la clé API

[role token]<img width="1099" height="577" alt="image" src="https://github.com/user-attachments/assets/13512886-3619-42d1-8ca1-b89d6ad543b6" />

## Terraform

- Création et configuration
[workspace]<img width="648" height="122" alt="image" src="https://github.com/user-attachments/assets/b3ce2d2c-062a-4a4f-b429-038f7941e057" />

- main.tf
```
terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.7.4"
    }
  }
}
```
- Initialisation et téléchargement du provider telmate pour terraform
<img width="776" height="441" alt="image" src="https://github.com/user-attachments/assets/5f74833d-8554-4641-ae9d-c323ccdc9401" />

- 



Ressource:

- https://openclassrooms.com/fr/courses/8481551-optimisez-vos-virtualisations-avec-proxmox-ve/8502981-decouvrez-et-installez-proxmox-ve
- https://rdr-it.com/proxmox-utiliser-cloud-init-pour-deployer-des-machines-virtuelles-ubuntu/
- https://ochoaprojects.com/posts/DeployingVMsWithTerraformInProxMox/
- https://slash-root.fr/proxmox-template-debian12-avec-cloud-init/
- https://dev.to/mrvym/introduction-a-terraform-avec-proxmox-83a
- https://registry.terraform.io/providers/bpg/proxmox/latest/docs
- https://registry.terraform.io/providers/Telmate/proxmox/latest/docs/resources/vm_qemu
- https://www.it-connect.fr/tuto-bien-debuter-avec-proxmox-ve/#VII_Creer_une_machine_virtuelle_sous_Proxmox
