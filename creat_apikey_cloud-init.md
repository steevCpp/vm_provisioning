# Création d'un rôle spéciale et l'utilisateur pour la génération de la clé d'api, on le fait en cli mais on peut aussi le faire via ui (Proxmox 9).
- Création d'un nouveau rôle TerraformProv

```
pveum role add TerraformProv -privs "Datastore.AllocateSpace Datastore.AllocateTemplate Datastore.Audit Pool.Allocate Pool.Audit Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.PowerMgmt SDN.Use"

```

- Création d'un utilisateur terraform-prov

```
pveum user add terraform-prov@pve --password userProxmox
```

- On associe le rôle à l'utilisateur qu'on vient de créer

```
pveum aclmod / -user terraform-prov@pve -role TerraformProv
```

- On peut tester à s'authentifier attention utiliser (Proxmox VE authentication server, pour Realm).

- Création de la clé api associé à l'utilisateur 

```
pveum user token add terraform-prov@pve mytoken
```

│  full-tokenid │ terraform-prov@pve!mytoken            │
│ value         │ 25a1df32-1b91-4c65-9898-be100c4325f7 │

- Dans le serveur/outils sur lequel compte utiliser terraform, en export les données de onnection.

```(bash)
export PM_API_TOKEN_ID='terraform-prov@pve!mytoken'
export PM_API_TOKEN_SECRET="25a1df32-1b91-4c65-9898-be100c4325f7"
```









- Source:  https://registry.terraform.io/providers/Telmate/proxmox/latest/docs
           https://pve.proxmox.com/wiki/User_Management#pveum_users
