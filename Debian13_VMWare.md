# Preparando um ambiente Docker profissional no Debian 13 (VMware)

## Introdução

Instalar o Docker é fácil.  
Difícil é montar um **ambiente limpo, organizado e confiável**, que continue funcionando bem depois de meses de uso.

Neste tutorial, você vai aprender a preparar uma **máquina virtual com Debian 13 no VMware**, passando por ajustes iniciais do sistema, instalação oficial do Docker Engine e organização do ambiente de trabalho. A proposta aqui não é improviso, mas sim criar uma **base sólida para laboratório, estudos e projetos reais**.

Ao final, você terá um ambiente replicável, previsível e pronto para evoluir.

## Etapa 1 — Ajustes iniciais, preparação e desempenho da VM

Nesta etapa, preparamos o sistema operacional para uso diário, evitando o uso direto do usuário `root` e aplicando ajustes importantes para ambientes virtualizados.

### 1.1 Instalar e configurar o sudo (como root)

```bash
apt update
apt install -y sudo
```

Adicionar o usuário operacional (`mol`) ao grupo sudo:

```bash
usermod -aG sudo mol
```

Validar:

```bash
getent group sudo
```

Trocar para o usuário `mol`:

```bash
su - mol
```

Testar:

```bash
sudo whoami
```


### 1.2 Atualizar o sistema

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

### 1.3 Instalar pacotes essenciais

```bash
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release \
  apt-transport-https \
  htop \
  vim \
  git \
  bash-completion
```


### 1.4 Ajustes de desempenho para VM

```bash
echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-swappiness.conf
sudo sysctl -p /etc/sysctl.d/99-swappiness.conf
```

```bash
sudo tee /etc/security/limits.d/99-docker.conf <<EOF
* soft nofile 1048576
* hard nofile 1048576
root soft nofile 1048576
root hard nofile 1048576
EOF
```

### 1.5 Instalar VMware Tools

```bash
sudo apt install -y open-vm-tools
sudo systemctl enable --now open-vm-tools
```

## Etapa 2 — Instalação do Docker Engine (oficial)

### 2.1 Remover versões antigas

```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
```

### 2.2 Adicionar chave e repositório oficial

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt update
```

### 2.3 Instalar Docker

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

```bash
sudo systemctl enable --now docker
```

### 2.4 Permitir uso sem sudo

```bash
sudo usermod -aG docker mol
newgrp docker
```

```bash
docker run hello-world
```

### 2.5 Ajustar daemon do Docker

```bash
sudo mkdir -p /etc/docker
```

```bash
sudo tee /etc/docker/daemon.json <<EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}
EOF
```

```bash
sudo systemctl restart docker
```

## Etapa 3 — Organização e limpeza

### 3.1 Estrutura inicial

```bash
sudo mkdir -p /srv/docker/{core,apps,stacks,volumes,shared,.backup,.envs}
sudo chown -R mol:mol /srv/docker
```

```bash
mkdir -p ~/projects ~/scripts
```

### 3.2 Otimizar escrita em disco

```bash
sudo sed -i 's/errors=remount-ro/errors=remount-ro,noatime/' /etc/fstab
```

### 3.3 Limpeza do sistema

```bash
sudo apt autoremove -y
sudo apt autoclean -y
sudo journalctl --vacuum-time=7d
```

### 3.4 Atalhos úteis

```bash
tee -a ~/.bashrc <<'EOF'

alias dps='docker ps'
alias dpsa='docker ps -a'
alias dimg='docker images'
alias dlog='docker logs -f'
alias dcu='docker compose up -d'
alias dcd='docker compose down'
alias dprune='docker system prune -af'
EOF

source ~/.bashrc
```

## Conclusão

Ao concluir este tutorial, você construiu uma **base técnica organizada, segura e pronta para crescer**.

Este ambiente é ideal para laboratórios, estudos, projetos pessoais e bases controladas de produção.
