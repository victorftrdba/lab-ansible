# Lab Ansible - Automação de Infraestrutura

Projeto Ansible para automação da configuração de servidores Ubuntu e instalação do Kubernetes K3s.

## 📋 Descrição

Este projeto automatiza a configuração completa de servidores Ubuntu, incluindo:
- Instalação e configuração do Docker
- Instalação e configuração do Kubernetes K3s
- Limpeza de disco e otimizações do sistema

## 🏗️ Estrutura do Projeto

```
lab-ansible/
├── ansible.cfg              # Configuração do Ansible
├── inventory.ini            # Inventário de hosts
├── site.yml                # Playbook principal
├── group_vars/
│   └── ubuntu_servers.yml  # Variáveis do grupo ubuntu_servers
├── host_vars/              # Variáveis específicas por host (opcional)
└── roles/
    ├── common/             # Role para setup básico do servidor
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── handlers/
    │   │   └── main.yml
    │   └── vars/
    │       └── main.yml
    └── k3s/                # Role para instalação do K3s
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── vars/
        │   └── main.yml
        └── templates/
            └── k3s-config.yaml.j2
```

## 📦 Pré-requisitos

### No Control Node (máquina que executa o Ansible)

- Python 3.6 ou superior
- Ansible 2.9 ou superior
- Acesso SSH aos servidores alvo
- Chaves SSH configuradas (ou senha configurada)

### Nos Target Nodes (servidores Ubuntu)

- Ubuntu 18.04 ou superior
- Acesso sudo
- Conexão com a internet (para download de pacotes)
- Mínimo de 500MB de espaço em disco livre

### Verificar Instalação do Ansible

```bash
ansible --version
```

### Instalar Ansible (se necessário)

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

# macOS
brew install ansible

# pip
pip install ansible
```

## 🚀 Como Usar

### 1. Configurar o Inventário

Edite o arquivo `inventory.ini` com os IPs ou hostnames dos seus servidores:

```ini
[ubuntu_servers]
192.168.100.17 ansible_user=ubuntu
192.168.100.18 ansible_user=ubuntu
```

### 2. Testar Conectividade

Verifique se o Ansible consegue se conectar aos servidores:

```bash
ansible all -m ping
```

### 3. Executar o Playbook Completo

Execute o playbook principal para configurar tudo:

```bash
ansible-playbook site.yml
```

### 4. Executar Roles Individuais

#### Apenas configuração básica (common):

```bash
ansible-playbook site.yml --tags common
```

#### Apenas instalação do K3s:

```bash
ansible-playbook site.yml --tags k3s
```

### 5. Executar em um Host Específico

```bash
ansible-playbook site.yml --limit 192.168.100.17
```

### 6. Modo Dry-Run (Check Mode)

Teste as mudanças sem aplicá-las:

```bash
ansible-playbook site.yml --check
```

### 7. Modo Verbose

Para mais detalhes durante a execução:

```bash
ansible-playbook site.yml -v    # Verbose
ansible-playbook site.yml -vv   # Mais verbose
ansible-playbook site.yml -vvv  # Muito verbose
```

## ⚙️ Configuração

### Variáveis

As variáveis podem ser configuradas em diferentes níveis (ordem de precedência):

1. **host_vars/** - Variáveis específicas por host (maior precedência)
2. **group_vars/** - Variáveis por grupo de hosts
3. **roles/*/vars/main.yml** - Variáveis padrão das roles (menor precedência)

#### Variáveis Principais

Edite `group_vars/ubuntu_servers.yml` para personalizar:

```yaml
# Usuário remoto
ansible_user: ubuntu

# Espaço mínimo em disco para K3s (em KB)
k3s_min_disk_space_kb: 524288  # 500MB

# Tentativas e delay para verificação do K3s
k3s_retries: 10
k3s_delay: 5

# Pacotes a serem instalados
common_packages:
  - docker.io
  - curl
  - python3-pip
```

### Configuração do Ansible

O arquivo `ansible.cfg` contém as configurações principais:

- **inventory**: Localização do inventário
- **roles_path**: Caminho para as roles
- **timeout**: Timeout para conexões SSH
- **forks**: Número de hosts processados em paralelo

## 📚 Roles

### Role: common

Configuração básica do servidor:
- Atualização do sistema
- Instalação do Docker
- Instalação de dependências (curl, python3-pip)
- Configuração de usuários e grupos

### Role: k3s

Instalação e configuração do Kubernetes K3s:
- Limpeza de disco (APT, Docker, logs)
- Verificação de espaço disponível
- Instalação do K3s
- Configuração e validação do cluster

## 🔍 Troubleshooting

### Problema: Erro de conexão SSH

**Solução:**
```bash
# Testar conectividade manualmente
ssh ubuntu@192.168.100.17

# Verificar chaves SSH
ssh-add -l

# Usar senha se necessário
ansible-playbook site.yml --ask-pass
```

### Problema: Erro de permissão sudo

**Solução:**
```bash
# Executar com senha sudo
ansible-playbook site.yml --ask-become-pass

# Verificar se o usuário tem sudo sem senha configurado
ansible all -m shell -a "sudo -n true" --become
```

### Problema: Espaço em disco insuficiente

**Solução:**
- A role k3s limpa automaticamente o disco antes da instalação
- Se ainda houver problemas, aumente o valor de `k3s_min_disk_space_kb` em `group_vars/ubuntu_servers.yml`
- Ou limpe manualmente o servidor antes de executar o playbook

### Problema: K3s não inicia

**Solução:**
```bash
# Verificar logs do K3s
ansible all -m shell -a "journalctl -u k3s -n 50" --become

# Verificar status do serviço
ansible all -m shell -a "systemctl status k3s" --become

# Reiniciar o K3s
ansible all -m shell -a "systemctl restart k3s" --become
```

### Problema: Docker não funciona

**Solução:**
```bash
# Verificar se o Docker está rodando
ansible all -m shell -a "systemctl status docker" --become

# Reiniciar o Docker
ansible all -m shell -a "systemctl restart docker" --become

# Verificar se o usuário está no grupo docker
ansible all -m shell -a "groups" --become-user ubuntu
```

## 🔐 Segurança

### Boas Práticas

1. **Não commitar senhas ou chaves privadas** - Use Ansible Vault para dados sensíveis
2. **Usar chaves SSH** - Evite usar senhas em produção
3. **Limitar acesso sudo** - Configure sudoers apropriadamente
4. **Atualizar regularmente** - Mantenha o Ansible e os pacotes atualizados

### Usando Ansible Vault

Para proteger dados sensíveis:

```bash
# Criar arquivo criptografado
ansible-vault create group_vars/ubuntu_servers_secrets.yml

# Editar arquivo criptografado
ansible-vault edit group_vars/ubuntu_servers_secrets.yml

# Executar playbook com vault
ansible-playbook site.yml --ask-vault-pass
```

## 📝 Exemplos de Uso

### Executar apenas em servidores específicos

```bash
ansible-playbook site.yml --limit "ubuntu_servers:!192.168.100.18"
```

### Executar com tags específicas

```bash
# Apenas instalar Docker
ansible-playbook site.yml --tags "docker"

# Apenas limpar disco
ansible-playbook site.yml --tags "cleanup"
```

### Verificar o que será executado

```bash
# Listar hosts que serão afetados
ansible-playbook site.yml --list-hosts

# Listar tasks que serão executadas
ansible-playbook site.yml --list-tasks
```

## 🧪 Testes

### Validar sintaxe dos playbooks

```bash
ansible-playbook site.yml --syntax-check
```

### Validar sintaxe do inventário

```bash
ansible-inventory --list
```

## 📖 Recursos Adicionais

- [Documentação oficial do Ansible](https://docs.ansible.com/)
- [Best Practices do Ansible](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Documentação do K3s](https://k3s.io/)
- [Ansible Galaxy](https://galaxy.ansible.com/) - Roles prontas para uso

## 🤝 Contribuindo

1. Faça um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

## 👤 Autor

Victor - [GitHub](https://github.com/victor)

---

**Nota**: Este projeto segue as melhores práticas recomendadas pela comunidade Ansible para organização e estruturação de projetos.

