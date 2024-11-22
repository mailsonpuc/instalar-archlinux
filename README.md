# instalar-archlinux
estudos instalar arch linux.

**image iso** https://archlinux.org/download/

**pendrive boot** https://www.ventoy.net/en/download.html

O primeiro passo é ajustar o seu teclado, caso o layout padrão do seu teclado seja em inglês, você não precisa alterar o layout dele, porém, caso você use um teclado brasileiro, ABNT2 (padrão no Brasil), você pode carregar as teclas com o seguinte comando:

```bash
loadkeys br-abnt2
```

Caso você precise conectar-se via Wi-Fi, você pode utilizar o pacote iwd, ele fornece o programa cliente iwctl, o daemon iwde e a ferramenta de monitoramento Wi-Fi iwmon.

Para utilizar o pacote iwd para se conectar a rede Wi-Fi, rode o comando:

```bash
iwctl
```

Por fim, para conectar-se a uma rede rode o comando:
```bash
[iwd]# station nomedodispositivo connect nomedarede
```


**Configurando o disco**
```bash
cfdisk -z /dev/sda
```
escolhar gpt

/dev/sda1 (1000G para o /boot/efi)
/dev/sda2 (2GB para swap)
/dev/sda3 (40GB para /)
/dev/sda4 (todo o resto para o /home)

Após particionar o seu disco lembre de marcar a partição que receberá o GRUB (no meu caso a /dev/sda1 como BIOS boot ou EFI system na opção “Type”).

Em seguida, faça o cfdisk escrever as partições clicando na opção “Write”


**Formatando as partições**

Após criar todas as partições, o próximo passo é formatá-las.

**Formatando partição de boot:**
```bash
mkfs.fat -F32 /dev/sda1
```
**Criando partição SWAP:**
```bash
mkswap /dev/sda2
```
**Formatando partição raíz do sistema:**
```bash
mkfs.ext4 /dev/sda3 
```
**Formatando a partição /home:**
```bash
mkfs.ext4 /dev/sda4
```

**Pontos de montagem**

Após formatar as partições, o próximo passo é montá-las, atente-se que será necessário criar algumas pastas para poder fazer a montagem das partições corretamente.

Montando a partição raiz:
```bash
mount /dev/sda3 /mnt 
```
Criando o diretório /home:
```bash
mkdir /mnt/home 
```
Criando o diretório /boot:
```bash
mkdir /mnt/boot
```

**Criando o diretório** /boot/efi (se for utilizar UEFI):
```bash
mkdir /mnt/boot/efi
```

Montando a partição /home:
```bash
mount /dev/sda4 /mnt/home
```
Montando a partição /boot:
```bash
mount /dev/sda1 /mnt/boot
```
Montando a partição /boot/efi (se for utilizar UEFI):
```bash
mount /dev/sda1 /mnt/boot/efi
```
 Ativando a partição SWAP:
```bash
swapon /dev/sda2
```
Você pode conferir se está tudo certo rodando o comando:
```bash
lsblk
```

**Instalando pacotes essenciais do Arch Linux
**
Neste passo, instalaremos o metapacote base e o grupo base-devel, além do kernel Linux padrão do Arch, o firmware para hardware comum, os editores de texto “nano” e “vim” e o dhcpcd com o comando:
```bash
pacstrap /mnt base base-devel linux linux-firmware nano vim dhcpcd
```

Gerando a tabela FSTAB

Após finalizar a instalação dos pacotes essenciais, vamos gerar a nossa tabela FSTAB, que vai dizer para o sistema onde estão montadas cada uma das partições, para isso, basta rodar o comando:

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```

Agora é hora de mudarmos para dentro do sistema

Até o momento, estávamos fazendo configurações que eram direcionadas para dentro de /mnt, onde o sistema estava sendo montado, agora, vamos entrar no ambiente do nosso sistema em processo de instalação com o comando:
```bash
arch-chroot /mnt
```

Temos agora uma série de configurações que podem ser feitas, tanto agora, quanto depois da instalação do sistema. Mas, para fins de deixar o seu sistema um pouco mais completo, vamos editar o nome do host para rede, para isso, utilize o comando:
```bash
hostnamectl set-hostname nomedoseuhost
```

**Configurando usuário root**

Para configurar a nova senha do seu usuário root, basta rodar o comando:
```bash
passwd
```

**Criando um usuário**

Você pode criar um usuário com o seu nome, por exemplo, através do comando:
```bash
useradd -m -g users -G wheel,storage,power -s /bin/bash nomedousuario
```

E, em seguida, defina uma senha para o seu novo usuário com o comando:
```bash
passwd nomedousuario
```

**Instalando pacotes úteis **

Após criar o seu novo usuário, vamos instalar alguns pacotes que serão úteis na pós instalação do sistema com o comando:
```bash
pacman -S dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant wireless_tools dialog
```

**UEFI**

Se você estiver utilizando UEFI, instale o GRUB com os comandos:

pacman -S grub efibootmgr

E em seguida, rode o comando:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck
```
Por fim, basta gerar o arquivo de configurações do GRUB com o comando:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Chegamos ao final da instalação base do sistema, digite “exit” ou pressione “Ctrl+D” e use o comando “reboot” para reinicializar o sistema, remova o pendrive da máquina.
