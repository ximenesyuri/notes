
```
-----------------------------------------
Criação de cluster k3s na Magalu Cloud
-----------------------------------------
```
```
__OBS.__
    1. Abaixo: MGC = Magalu Cloud 
    2. Pressupõe-se o uso de Linux
    3. Dependênncias: ssh e xclip
```

# Etapas Iniciais

```
__OBS.__ 
    No que segue, VMs corresponderão ao nodes do cluster
```

1. Na sua máquina local, gere uma chave SSH:
```bash
ssh-keygen -t ed25519 -C "seu@email" -f /caminho/para/chave -N "sua frase de segurança"
```
2. Copie a chave para o clipboard:
```bash
cat /caminho/para/chave | xclip -selection clipboard
```
3. Acesse o console da `MGC`, crie VMs correspondentes aos nodes que se deseja utilizar.
4. Em cada VM:
    1. adicione a chave ssh copiada
    2. anote seu IP privado (veja abaixo).

```
__OBS.__ 
    Recomenda-se ao menos 3 VMs: uma como servidor (control pane), um para os serviços e uma como réplica, com os requisitos mínimos abaixo.
    1. servidor: ao menos 2G de RAM
    2. servicos: ao menos 4G de RAM
    3. replica: idem.

    nome           VM         RAM          Disco           Preço (R$/mês)
    --------------------------------------------------------------------
    servidor      BV1-2-20    2GB          20GB            54,99
    servicos      BV2-4-40    4GB          40GB            102,99
    replica       BV2-4-40    4GB          40GB            102,99        
```

```
__OBS.__
    Cada VM vem dotada de um IP privado (associado à rede da `MGC`). Opcionalmente pode-se adicionar um IP público. Para instalar o cluster `k3s` você vai precisar do acesso SSH de cada máquina, o que requer um IP público. Uma vez instalado e configurado, a depender do seu uso, os IPs públicos podem ser removidos (exceto o do `servidor`).
```

# Preparação

1. No console do `MGC`, acesse alguma das VMs criadas.
2. Clique na aba `Rede` e crie um novo Grupo de Segurança, como segue.
```
porta      sentido       protocolo    uso               IPs de acesso
------------------------------------------------------------------------------
22         entrada       TCP          SSH               qualquer
6443       entrada       TCP          API cluster k3s   IP privado de cada VM
6443       saída         TCP          API cluster k3s   IP privado de cada VM
```
3. Em cada VM adicione o Grupo de Segurança criado.

# Instalação: `servidor`

1. Da sua máquina local, acesse via SSH a VM correspondente ao `servidor`:
```bash
eval "$(ssh-agent -s)"
ssh-add /caminho/para/sua/chave
ssh usuario@IP-publico-servidor
```
2. Verifique se `ufw` está instalado e habilitado, e, nesse caso, libere a porta `6443`:
```bash
sudo ufw status | grep " active" && sudo ufw allow 6443
```
3. Instale `k3s` (`kubectl` é instalado em conjunto):
```bash
curl -sfL https://get.k3s.io | sh -
```
3. Teste a instalação (`Ok` deve ser exibido):
```bash
if sudo kubectl get nodes | \ 
    grep "control-plane" | \
    grep -q Ready; then \
    echo "Ok"; \
    fi
```
4. Feche a conexão SSH:
```bash
exit
```
3. Copie para o clipboard o token que será utilizado para adicionar novos nodes:
```bash
ssh usuario@IP-publico-servidor \
    'sudo cat /var/lib/rancher/k3s/server/node-token' | \
    xclip -selection clipboard
```
4. Reinicie o `servidor`:
```bash
ssh usuario@IP-publico-servidor 'sudo reboot'
```

```
__OBS.__ 
    Na MGC "usuario" é, por padrão, o nome da distribuição instalada (pode-se conferi-lo diretamente no console, na aba da VM).
    - Exemplo: se a distribuição é Ubuntu, então "usuario = ubuntu".
```

# Instalação: `nodes`

Para cada VM adicional (por exemplo, `servicos` e `replica`):

1. Acesse-a via SSH (sugere-se abrir uma nova janela no terminal e manter a janela do `servidor` aberta):
```bash
eval "$(ssh-agent -s)"
ssh-add /caminho/para/sua/chave
ssh usuario@IP-publico-VM
```
2. Instale `k3s` como `agent` (o token deve estar no clipboard):
```bash
curl -sfL https://get.k3s.io | \ 
    INSTALL_K3S_SKIP_START=true \
    K3S_URL=https://IP-privado-servidor:6443 \
    K3S_TOKEN=node-token-servidor sh -
```
3. Reinicie o sistema:
```bash
sudo reboot
```
4. Depois de alguns instantes, habilite o daemon do `k3s agent`:
```bash
ssh usuario@IP-publico-VM \
    'sudo systemctl enable --now k3s-agent'
```

# Conclusão

Se tudo estiver certo, os nodes serão reconhecidos pelo `servidor`:
```bash
ssh usuario@IP-publico-servidor \ 
    'sudo kubectl get nodes'
```
