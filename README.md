# OlhLinux 1.0 (Olho)

Sistema operacional Linux proprio, com interface grafica propria, sete
aplicativos, gerenciamento do sistema e imagem ISO inicializavel.

O OlhLinux nao e uma simulacao de sistema operacional: e um Linux real
(Debian 13 "trixie" como base, compilado com `debootstrap`) com ambiente
grafico, gerenciador de janelas, tela de login e aplicativos escritos
para o projeto. Tudo o que aparece na tela le informacao real do
computador — `/proc`, `/sys`, X11, NetworkManager, PipeWire, udisks2,
`lsblk` e a `libpam` do sistema.

```
+--------------------------------------------------+
|  OlhLinux 1.0 (Olho)                             |
|  area de trabalho, barra de tarefas, menu, apps  |
+--------------------------------------------------+
|  olh-login (tela de login, PAM)                   |
|  olh-session (sessao) + olh-wm (janelas)          |
|  Xorg 11, GTK 3, VTE, WebKit2                    |
+--------------------------------------------------+
|  Debian 13 (trixie) - kernel, systemd, utilitarios |
+--------------------------------------------------+
```

## O que o sistema faz

**Interface grafica propria**

- area de trabalho com papel de parede e icones de pastas
- barra de tarefas com menu de aplicativos,atalhos, lista de janelas,
  area de status (USB, rede, volume, bateria), relogio e calendario
- menu de aplicativos com busca e categorias
- janelas com barra de titulo propria (minimizar, maximizar, fechar),
  arrastar, redimensionar, encaixar nas bordas e pilha de janelas
- tela de login com marca do OlhLinux, relogio, usuarios e senha real
- tela de inicializacao (Plymouth e a splash grafica do proprio login)
- tema escuro e claro, icones proprios (126 SVG) e papel de parede proprio

**Gerenciamento do sistema**

| Recurso            | Como funciona                                        |
|--------------------|------------------------------------------------------|
| Arquivos e pastas  | `olh-files`, com todas as disco e dispositivos USB    |
| Aplicativos        | registro em `programs/` + arquivos `.desktop`         |
| Resolucao de tela  | `xrandr`, gravada em `etc/olhdesktop.conf`            |
| Audio              | PipeWire via `pactl` (volume, silencio, saida padrao) |
| Rede               | NetworkManager via `nmcli` (Wi-Fi e com fio)         |
| Usuarios           | `useradd`/`userdel` + `etc/users.conf`                |
| USB                | `lsblk` + `udisks2` com autorizacao do polkit         |
| Terminal           | `olh-terminal` com VTE e bash real (PTY)              |

**Aplicativos**

| Aplicativo             | Id                    |
|------------------------|-----------------------|
| Explorador de arquivos | `olh-files`           |
| Terminal               | `olh-terminal`        |
| Navegador              | `olh-browser`         |
| Editor de texto        | `olh-editor`          |
| Calculadora            | `olh-calculator`      |
| Configuracoes          | `olh-settings`        |
| Monitor do Sistema     | `olh-system-monitor`  |

## Os cinco diretorios do projeto

Cada diretorio do repositorio tem uma funcao real no sistema instalado:

| Diretorio   | Funcao                                | No sistema             |
|-------------|---------------------------------------|------------------------|
| `usr/`      | diretorios pessoais dos usuarios      | `/home`                |
| `bin/`      | executaveis e componentes essenciais  | `/usr/local/bin`       |
| `etc/`      | configuracao do sistema               | `/etc/olhlinux`        |
| `programs/` | programas instalados pelo usuario    | `/opt/programs`        |
| `root/`     | administrador e controle do boot      | `/root`                |

O codigo do ambiente grafico vai para `/usr/lib/olhlinux` e os recursos
(icones, temas, papel de parede, arquivos `.desktop`) para
`/usr/share/olhlinux`. O mapeamento completo esta em
[`docs/estrutura.md`](docs/estrutura.md).

## Como obter a ISO

**No Windows** (PowerShell como Administrador):

```powershell
powershell -ExecutionPolicy Bypass -File build\windows\build-iso.ps1
```

O script instala o WSL2 com o Debian 13, instala as ferramentas de
compilacao e chama `build/build-iso.sh`. Detalhes em
[`docs/instalacao.md`](docs/instalacao.md).

**Em um Linux** (Debian/Ubuntu, como root):

```bash
sudo apt-get install debootstrap squashfs-tools xorriso mtools \
    grub-pc-bin grub-efi-amd64-bin
sudo build/build-iso.sh
```

A imagem fica em `build/olhlinux.iso` e e verificada por
`build/verify.sh`.

## Como usar a imagem

```bash
# testar em uma maquina virtual
build/qemu-run.sh build/olhlinux.iso

# ou gravar em um pendrive (Linux)
sudo dd if=build/olhlinux.iso of=/dev/sdX bs=4M status=progress conv=fdatasync
```

A ISO sobe o OlhLinux inteiro (BIOS e UEFI), com teclado e mouse, sem
pedir senha de instalador. Usuario e senha padrao:

| Usuario | Senha  | Perfil                            |
|---------|--------|-----------------------------------|
| `olh`   | `olh`  | administrador                    |
| `guest` | `guest`| visitante, sem privilegios        |

Requisitos da maquina: processador x86_64, 2 GB de memoria e teclado e
mouse (USB ou PS/2). Detalhes em [`docs/iso.md`](docs/iso.md).

## Estrutura do repositorio

```
OlhLinux/
├── bin/        executaveis do sistema (olh-session, olh-wm, olh-login, ...)
├── etc/        configuracao (olhlinux.conf, olhdesktop.conf, pam.d/olhlogin)
├── usr/        /.skel e os diretorios pessoais de olh e guest
├── programs/   registro dos programas instalados pelo usuario
├── root/       boot.sh e manutencao.sh (administrador)
├── src/        codigo: olhcore, olhde, olhwm, olhapps, olhlogin, olhsplash
│   └── assets/ icones, temas, papel de parede e arquivos .desktop
├── build/      build-iso.sh, packages.txt, grub.cfg, qemu-run.sh, verify.sh
│   ├── plymouth/  tema da tela de inicializacao
│   └── windows/   build-iso.ps1 (compila pelo WSL2)
├── docs/       documentacao tecnica
└── README.md
```

## Documentacao

- [`docs/arquitetura.md`](docs/arquitetura.md) — como o sistema sobe, do
  firmware a tela de login e a sessao
- [`docs/estrutura.md`](docs/estrutura.md) — os cinco diretorios e onde
  cada coisa vive
- [`docs/instalacao.md`](docs/instalacao.md) — compilacao passo a passo
- [`docs/iso.md`](docs/iso.md) — como a ISO e montada, testada e gravada
- [`docs/referencia-cli.md`](docs/referencia-cli.md) — todos os comandos de
  `bin/`

## Licenca e nome

"OlhLinux" e um nome proprio deste projeto; a base e o Debian 13, sob a
licenca que o Debian distribui.
