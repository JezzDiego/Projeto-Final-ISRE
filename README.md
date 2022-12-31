## Requisitado pelo professor: Alaelson Jatobá <br /> Disciplina: Infraestrutura de Redes <br /> Data: 28/12/2022 <br /> Turma: 923 <br /> Grupo: 3

### Integrantes
* Jessé Diego da Silva Oliveira
* Lucas Rosendo de Farias
* Maria Beatriz Ferreira dos Santos
* Maria Eduarda Araújo Brito
* Ana Heloíza Albuquerque Barros

## Objetivos do projeto:
* Colocar em prática o que aprendemos com as apresentações do 3° bimestre
* Criar um tutorial resumido ensinando a fazer a implementação das tecnologias apresentadas
* Testar os conhecimentos adquiridos em aulas


# Definições de endereços IPs da Rede e Nomes de Hosts:

```
----------------------------------------------------------------------------------------------------------------
|  DESCRICAO  |          IP          |      hostname     |            FQDN                  |       aliase     |
----------------------------------------------------------------------------------------------------------------
| GRUPO3VM1   | 10.9.23.113          |   gw-srv          | gw.grupo3.turma923.ifalara.local   |     next       |
| GRUPO3VM2   | 10.9.23.115          |   ns1-srv         | ns1.grupo3.turma923.ifalara.local  |     gatsby     |
| GRUPO3VM3   | 10.9.23.109          |   samba-srv       | smb.grupo3.turma923.ifalara.local  |     react      |
| GRUPO3VM4   | 10.9.23.117          |   ns2-srv         | ns2.grupo3.turma923.ifalara.local  |     vue        |
| GRUPO3VM5   | 10.9.23.103          |   web-srv         | web.grupo3.turma923.ifalara.local  |     angular    |
----------------------------------------------------------------------------------------------------------------
```

## Requisitos de Hardware
### Hardware utilizado nas VM's
* Núcleos lógico de processamento: 1
* RAM: 8.02GB
* Espaço em disco: 14.9GB distribuídos da seguinte forma:
![image](https://user-images.githubusercontent.com/64742095/209895751-40464d94-fc6f-4fb0-8eb6-3d3453ddceee.png)

## Instalação Samba

Atualizando os pacotes do sistema e instalando o serviço samba
```
sudo apt update
```
```
sudo apt install samba
```

Fazendo o backup dos aquivos de configuração para caso haja a necessidade de retornar para as configurações iniciais

```
sudo cp /etc/samba/smb.conf{,.backup}
```

Editar o arquivo de configuração adicionando a interface de rede do seu sistema
```
sudo nano /etc/samba/smb.conf
```

O aquivo de configuração deve se paracer com esse, onde em interfaces deve conter o nome e o ip da sua interface de rede:
```
[global]
   workgroup = WORKGROUP
   netbios name = samba-srv
   security = user
   server string = %h server (Samba, Ubuntu)
   interfaces = 10.9.23.109/24 ens160
   bind interfaces only = yes
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
[homes]
   comment = Home Directories
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700
   valid users = %S
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = yes
   guest only = yes
   force user = nobody
   force create mode = 0777
   force directory mode = 0777
```

Reinicar o serviço smb para aplicar as alterações
```
sudo systemctl restart smbd
```

Para criar um grupo e elevar suas permissões de acesso, configure a aba [public] da seguinte forma:
```
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = no
   valid users = @sambashare
   #guest only = yes
   #force user = nobody
   #force create mode = 0777
   #force directory mode = 0777
```

Crie uma senha samba para o usuário desejado (execute o comando sem os colchetes)
```
sudo smbpasswd -a [nome_do_usuario]
```
Insira o usuário no grupo sambashare que foi definido anteriormente
```
sudo usermod -aG sambashare aluno
```
Crie as pastas de compartilhamento de arquivos
```
mkdir /home/<username>/sambashare/
```
```
sudo mkdir -p /samba/public
```
Permissões para que qualquer usuário possa acessar a pasta public
```
sudo chown -R nobody:nogroup /samba/public
```
```
sudo chmod -R 0775 /samba/public
```
```
sudo chgrp sambashare /samba/public
```
# Serviço samba configurado com sucesso
