# Projeto 01 - DevOps com Vagrant e Ansible

## Informações do Projeto

* **Disciplina:** Administração de Sistemas Abertos
* **Curso:** Redes de Computadores
* **Professor:** Leonidas Lima
* **Período:** 2026.1
* **Instituição:** Instituto Federal da Paraíba - Campus João Pessoa

## Desenvolvedor

* **Victor Nemuel Linhares Farias** - Matrícula: 20241380014

---

## Visão Geral

Este projeto implementa uma infraestrutura virtual completa utilizando **Vagrant** e **Ansible**, com quatro servidores especializados que se comunicam em uma rede privada. A solução automatiza desde o provisionamento das máquinas virtuais no VirtualBox até a configuração avançada de serviços de rede e armazenamento, demonstrando na prática os princípios de Infraestrutura como Código (IaC).

## Arquitetura do Sistema

A infraestrutura consiste em quatro máquinas virtuais interconectadas, todas utilizando o **Debian 12 (bookworm64)** como sistema operacional base. Como o projeto foi desenvolvido individualmente, a distribuição dos IPs foi adaptada a partir da matrícula do desenvolvedor:

* **Servidor de Arquivos (arq):** Configurado com IP fixo `192.168.56.114` (baseado no final da matrícula), fornece serviços de DHCP, DNS, armazenamento LVM e compartilhamento NFS. Possui três discos adicionais de 10 GB cada para o sistema de armazenamento.
* **Servidor de Banco de Dados (db):** Obtém IP via DHCP com reserva estática baseada no endereço MAC configurado no Vagrantfile (`192.168.56.115`) e executa o MariaDB. Monta automaticamente o compartilhamento NFS do servidor de arquivos utilizando `autofs`.
* **Servidor de Aplicação (app):** Também com IP reservado via DHCP através do endereço MAC (`192.168.56.116`), executa o servidor web Apache2 com uma página HTML personalizada com os dados do aluno. Acessa o compartilhamento NFS para armazenamento via `autofs`.
* **Cliente (cli):** Configurado com 1024 MB de RAM, possui os pacotes `firefox-esr` e `xauth`, além de suporte à exportação da interface de aplicativos via X11. Serve como estação de trabalho e também acessa o NFS via `autofs`.

## Configuração Automatizada

O provisionamento ocorre em etapas sequenciais automatizadas. Primeiro, o Vagrant cria as máquinas virtuais utilizando clones (*linked_clone*) e desativa o servidor DHCP nativo do VirtualBox através de um gatilho. Em seguida, o Ansible executa playbooks para cada servidor.

As configurações comuns aplicadas a **todas** as máquinas incluem:

* Atualização do sistema operacional (`update` e `upgrade`).
* Configuração da zona de tempo para `America/Recife`.
* Instalação e sincronização de horário com o servidor NTP `pool.ntp.br` via chrony.
* Criação do grupo `ifpb` e do usuário `victor`.
* Hardening do serviço SSH: autenticação exclusiva por chaves públicas, bloqueio de acesso root, restrição de acesso apenas aos grupos `vagrant` e `ifpb` e configuração de um banner de aviso legal.

## Serviços de Rede

O servidor de arquivos implementa um servidor DHCP autoritativo que gerencia a faixa de IPs `192.168.56.50` a `192.168.56.100`, com *lease-time* padrão de 180 segundos e máximo de 3600 segundos. O serviço DNS autoritativo (zonas direta e reversa) configurado no mesmo servidor resolve nomes internos no domínio `victor.devops` e encaminha requisições externas para os servidores recursivos `1.1.1.1` e `8.8.8.8`.

## Sistema de Armazenamento

Os três discos de 10 GB anexados ao servidor `arq` são combinados através do LVM em um Volume Group (VG) chamado `dados`. A partir dele, foi criado um Logical Volume (LV) de 15 GB chamado `ifpb`, formatado em `ext4` e montado automaticamente no diretório `/dados` na inicialização.

Este diretório contém o compartilhamento NFS (`/dados/nfs`) acessível pela rede interna `192.168.56.0/24`. Foi criado o usuário dedicado `nfs-ifpb` sem shell de login, mapeado automaticamente para garantir segurança, além do forçamento de gravações imediatas no disco (*sync*).

## Execução do Projeto

A infraestrutura completa é provisionada e configurada com um único comando na raiz do projeto:

```bash
vagrant up

```

Este comando baixa o *box* do sistema, cria as máquinas virtuais desabilitando os *guest additions*, configura as interfaces de rede e invoca automaticamente os *playbooks* do Ansible para finalizar o escopo de software.
