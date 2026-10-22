# DEBIAN.md — Contexto Linux e validação dos instaladores

Este arquivo registra o contexto operacional do SetupVibe em Debian, os padrões
de implementação dos scripts Unix e o procedimento de teste em containers.

## Regra de uso

Este é um contexto obrigatório para trabalhos que alterem, investiguem ou
testem `desktop.sh`, `server.sh`, arquivos de `conf/` usados no Linux ou a
documentação Linux.

Antes de atuar nesse escopo:

1. Leia `AGENTS.md` ou `CLAUDE.md`, conforme o agente em uso.
2. Leia este arquivo por completo.
3. Preserve as diferenças entre Desktop e Server descritas aqui.
4. Atualize este arquivo quando um teste real revelar uma nova limitação,
   causa raiz ou exigência de validação do Linux.

`AGENTS.md` e `CLAUDE.md` continuam sendo os contextos primários dos agentes.
Este arquivo é um contexto técnico suplementar da plataforma, não o contexto de
um terceiro agente.

## Escopo Linux do projeto

O SetupVibe oferece duas edições Unix executáveis em Debian:

- `desktop.sh`: instalador interativo de 14 etapas para desktop, WSL e
  ambientes completos de desenvolvimento.
- `server.sh`: instalador de 9 etapas, com opção não interativa `--yes`,
  concentrado em operações, Docker, shell, monitoramento e CLIs de IA.

Debian 12 ou mais recente é suportado. A validação descrita neste arquivo foi
feita em Debian GNU/Linux 13.6 `trixie`, arquitetura `amd64`.

## Modelo de privilégios

Os scripts podem ser iniciados por um usuário comum, por `sudo bash` ou
diretamente como root. A identidade que receberá ferramentas e configurações
deve ser mantida separada da identidade do processo:

- `REAL_USER`: usuário-alvo.
- `REAL_HOME`: diretório pessoal do usuário-alvo.
- `REAL_GROUP`: grupo primário do usuário-alvo.
- `user_do`: executa uma operação como `REAL_USER`.
- `sys_do`: executa somente a operação que exige privilégio administrativo.

Nunca use apenas `id -u` do processo para decidir onde instalar ferramentas do
usuário. Quando `sudo bash desktop.sh` é usado, o processo é root, mas o
destino continua sendo `REAL_USER`.

### `sudo` e `PATH`

O `sudo` normalmente aplica `secure_path` e pode remover diretórios como:

- `~/.local/bin`
- `~/.npm-global/bin`
- `~/.cargo/bin`
- `~/.bun/bin`

Exportar `PATH` no shell pai não garante que `user_do comando` encontrará um
executável instalado nesses diretórios. Para ferramentas gerenciadas pelo
script, use uma destas abordagens:

- caminho absoluto do executável;
- `user_do env PATH="$caminho_gerenciado:$PATH" comando`;
- um `bash -c` curto que defina explicitamente o `PATH`.

Depois da instalação, execute o comando ou `--version` usando o mesmo ambiente
que o usuário receberá.

Antes de repassar o `PATH` para serviços, remova entradas duplicadas preservando
a primeira ocorrência. Não basta impedir novas duplicatas: o ambiente herdado
de `sudo` ou do shell também pode chegar duplicado.

### Pacotes npm globais

Quando `REAL_USER` não é root, pacotes npm globais devem usar
`$REAL_HOME/.npm-global`, inclusive quando o instalador foi iniciado por
`sudo`. O fluxo correto é:

1. criar o diretório como `REAL_USER`;
2. configurar `npm config set prefix "$REAL_HOME/.npm-global"`;
3. executar o npm com `PATH="$REAL_HOME/.npm-global/bin:$PATH"`;
4. validar cada comando instalado com esse mesmo `PATH`.

Sem isso, o npm tenta alterar `/usr/lib/node_modules` como usuário comum e
falha com `EACCES`.

### Pacotes pip globais para o usuário

O Python fornecido pelo APT e pelo Homebrew é marcado como externamente
gerenciado conforme a PEP 668. Para um pacote solicitado como global, mas que
deve pertencer a `REAL_USER`, use o interpretador absoluto correspondente e
instale com `python -m pip --user --break-system-packages`; nunca use `sudo pip`.

Resolva o diretório dos executáveis com `python -m site --user-base` executado
via `user_do`, porque ele é `~/.local/bin` no Linux e
`~/Library/Python/X.Y/bin` no macOS. Adicione esse diretório ao `PATH`, valide o
import do pacote e execute a validação do CLI com a identidade e o `HOME` do
usuário-alvo.

## APT e repositórios

- Chaves APT pertencem a `/etc/apt/keyrings/`.
- Remova somente arquivos de repositório reconhecidamente gerenciados pelo
  SetupVibe.
- Não remova repositórios por busca genérica de conteúdo: eles podem pertencer
  ao usuário ou a outro gerenciador.
- Exclua entradas legadas antes de adicionar a fonte atual para evitar conflito
  de assinatura.
- O `server.sh` deve aguardar locks do APT, pois VMs recém-iniciadas podem estar
  executando `unattended-upgrades`.
- Para Debian instável (`forky`, `sid` ou `experimental`), o repositório Docker
  usa o codinome estável compatível definido pelo script.
- Em Debian, o Server usa `ansible-core`; não presuma que o metapacote
  `ansible` esteja disponível.

## Downloads e arquivos temporários

Downloads gerenciados devem:

1. usar HTTPS;
2. falhar em erros HTTP;
3. repetir de forma limitada erros transitórios de rede;
4. ter timeout de conexão, timeout total e tamanho mínimo;
5. rejeitar respostas HTML;
6. verificar SHA-256 quando o projeto publica ou fixa um checksum;
7. instalar o arquivo final com proprietário, grupo e modo explícitos.

Não baixe como root para um nome previsível em `/tmp` e depois execute
`user_do mv`. O sticky bit de `/tmp` permite que o usuário copie o conteúdo,
mas impede a remoção do arquivo criado por root; `mv` pode criar o destino e
ainda retornar erro ao tentar remover a origem.

Use `mktemp` e finalize com `sys_do install -o "$REAL_USER" -g "$REAL_GROUP"`.
O binário `ctop` deve ser verificado com o checksum correspondente à
arquitetura antes de ser instalado em `~/.local/bin/ctop`.

## Interatividade do Desktop

`desktop.sh` depende de um terminal real para:

- confirmar o roadmap;
- configurar nome e e-mail do Git quando ausentes.

O script deve validar `/dev/tty` antes de alterar o sistema e sair com erro
claro quando não houver terminal. Todo `read </dev/tty` também deve tratar
EOF ou perda do terminal. Sem essa proteção, o prompt de Git pode entrar em
loop infinito em automações sem TTY.

Em Docker, use `docker exec -it` para o Desktop. O Server pode usar execução
sem TTY com `--yes`.

## systemd e Docker dentro do container

Um container comum não representa adequadamente o instalador porque os scripts
habilitam e validam serviços como:

- `cron`;
- `docker`;
- `ssh`;
- `tailscaled`.

O teste de integração usa systemd como PID 1, container privilegiado e `tmpfs`
para `/run` e `/run/lock`.

Docker dentro de Docker não deve usar `overlay2` sobre o filesystem `overlay`
do container externo. Essa combinação falha e não representa um defeito do
SetupVibe. Somente no ambiente de teste aninhado, configure o daemon interno:

```json
{
  "features": {
    "containerd-snapshotter": false
  },
  "storage-driver": "vfs"
}
```

O driver `vfs` consome mais espaço e é mais lento, mas permite validar Docker
Compose e Portainer no container. Essa configuração não deve ser aplicada a
uma instalação Debian real.

## Receita reproduzível do container

Use uma imagem de teste descartável com um usuário comum e `sudo` sem senha:

```dockerfile
FROM debian:13

ENV container=docker

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        ca-certificates dbus systemd systemd-sysv sudo \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --shell /bin/bash tester \
    && printf '%s\n' 'tester ALL=(ALL) NOPASSWD:ALL' \
        > /etc/sudoers.d/tester \
    && chmod 0440 /etc/sudoers.d/tester

STOPSIGNAL SIGRTMIN+3
CMD ["/sbin/init"]
```

Crie o container com o repositório montado somente para leitura:

```bash
docker run -d \
    --name setupvibe-debian-desktop \
    --privileged \
    --security-opt label=disable \
    --tmpfs /run \
    --tmpfs /run/lock \
    -v "$PWD:/workspace:ro" \
    setupvibe/debian-systemd:13
```

Configure `vfs` no daemon Docker interno antes de testar a etapa DevOps. Em
seguida, execute:

```bash
# Desktop: TTY obrigatório
docker exec -it --user tester setupvibe-debian-desktop \
    sudo bash /workspace/desktop.sh

# Server: modo não interativo suportado
docker exec --user tester setupvibe-debian-server \
    sudo bash /workspace/server.sh --yes
```

O mount somente para leitura garante que o instalador não modifica
acidentalmente o checkout. Estado instalado e logs permanecem dentro do
container.

## Estratégia de validação

Uma mudança Linux não está validada apenas por `bash -n`. Use, conforme o
escopo:

1. `bash -n` para sintaxe.
2. `shellcheck` para problemas estáticos.
3. execução limpa no Debian suportado;
4. nova execução no mesmo container para testar idempotência;
5. resumo final com todas as etapas em `OK`;
6. validação dos comandos instalados;
7. validação dos serviços com `systemctl is-active`;
8. validação de Portainer com `docker ps`;
9. teste negativo do Desktop sem TTY;
10. execução da suíte de desenvolvimento com `bash tests/run_tests.sh`.

Avisos de dependências opcionais do npm não são por si só falhas. O critério é
o exit code da instalação e a execução bem-sucedida dos comandos gerenciados.

Versões atuais do npm controlam scripts de instalação de dependências em
instalações globais. Aprove somente os pacotes nativos necessários com
`--allow-scripts`; não use a opção ampla que libera todos os scripts.

## Aprendizados da investigação de julho de 2026

### Desktop sem TTY

Uma execução sem `docker exec -it` revelou que `read </dev/tty` falhava e o
prompt de Git repetia indefinidamente. O Desktop agora encerra antes de alterar
o sistema quando não encontra terminal interativo.

### Prefixo npm incorreto

O Desktop verificava o UID do processo root em vez de `REAL_USER`. Como
resultado, PNPM, PM2, n8n e todos os CLIs de IA tentavam usar `/usr` e falhavam
com `EACCES`. A decisão agora é baseada no usuário-alvo e cada comando é
validado com o `PATH` gerenciado.

### `ctop` e sticky bit de `/tmp`

O download era criado por root em `/tmp/ctop` e movido por `REAL_USER`. O
destino podia ser criado, mas a remoção da origem falhava com
`Operation not permitted`. O download agora usa arquivo temporário único,
checksum SHA-256 e instalação com propriedade explícita.

### Starship e CLIs instalados no diretório do usuário

Starship foi instalado corretamente em `~/.local/bin`, mas `user_do starship`
não o encontrou por causa do `secure_path` do sudo. O script passou a usar o
caminho absoluto. O mesmo princípio se aplica a Skills CLI e PM2.

### Fórmula Homebrew desativada

A fórmula `tldr` foi desativada pelo Homebrew em 24 de outubro de 2025 por
falta de manutenção. O Desktop instala a implementação mantida `tlrc`, que
continua oferecendo o comando `tldr`. Reruns devem desinstalar a fórmula
legada antes de instalar `tlrc`, pois ambas fornecem o mesmo executável.

### Scripts nativos bloqueados pelo npm

O npm bloqueou o script de instalação de `better-sqlite3`, dependência nativa
do Agentlytics. O pacote JavaScript era adicionado, mas o comando falhava ao
carregar o binding SQLite. A instalação agora aprova somente
`better-sqlite3` e valida a criação de um banco SQLite em memória. O
Agentlytics não oferece uma opção `--version`; usá-la inicia o fluxo
interativo e não é uma validação segura para automação.

O mesmo mecanismo bloqueou o `postinstall` do Claude Code e pode afetar outros
CLIs gerenciados. Cada pacote direto deve receber sua própria aprovação
`--allow-scripts` no comando de instalação. O n8n também exige aprovação
explícita para sua lista revisada de dependências com bindings ou geração de
artefatos.

### Idempotência do Starship e do `PATH`

`starship preset` recusa sobrescrever `~/.config/starship.toml` sem `--force`.
Como esse arquivo é gerenciado pelo Desktop, o rerun deve aplicar o preset com
essa opção.

Cada etapa adicionava novamente os mesmos diretórios ao `PATH`, o que gerava
uma variável enorme na unidade systemd do PM2. Diretórios gerenciados devem ser
inseridos somente quando ainda não estiverem presentes, e o caminho completo
deve ser deduplicado antes de ser fornecido ao PM2.

O gerador de startup do próprio PM2 acrescenta caminhos de sistema ao valor
recebido e pode reintroduzir duplicatas. No Linux, um drop-in systemd gerenciado
pelo SetupVibe deve sobrescrever o ambiente efetivo do serviço com o `PATH`
normalizado e executar `systemctl daemon-reload`.

### Falhas transitórias em downloads

Uma repetição completa chegou à etapa final e falhou ao baixar
`ecosystem.config.js` do GitHub. Downloads remotos são pontos de integração,
portanto uma resposta transitória não deve invalidar uma instalação longa sem
novas tentativas. O helper compartilhado agora exige HTTPS, aplica timeouts de
conexão e total e repete erros de transferência de forma limitada.

### PM2 e systemd

`pm2 startup systemd` executado como usuário comum apenas imprime o comando
administrativo necessário e retorna erro. A criação da unidade deve passar
por `sys_do`, com o caminho absoluto do PM2 disponível no `PATH`; operações
de estado como `pm2 save` continuam executando como `REAL_USER`.

### Tailscale em containers

Scripts de pacote podem respeitar `/usr/sbin/policy-rc.d` e instalar o
Tailscale sem iniciar o serviço. A etapa Linux deve executar e validar
explicitamente `systemctl enable --now tailscaled`.

### Resultado da execução

O `server.sh --yes` concluiu as 9 etapas no Debian 13, com Docker, Portainer,
Tailscale, SSH e cron ativos. A primeira execução limpa do Desktop reproduziu
as falhas acima. A primeira execução de regressão confirmou as correções de
npm, `ctop` e Starship e revelou os casos adicionais de migração `tldr`,
scripts nativos do npm e privilégios do PM2.

A execução integral final e idempotente do `desktop.sh` concluiu as 14 etapas
com `OK`. Docker, cron, SSH, Tailscale e `pm2-tester` ficaram ativos; Portainer
ficou em execução na porta HTTPS 9443; e a restauração do Agentlytics pelo
serviço PM2 foi testada depois de encerrar manualmente o daemon. O ambiente
efetivo do serviço ficou sem caminhos duplicados. Também passaram as
validações de versões dos ecossistemas, CLIs de IA, `tldr` via `tlrc`, checksum
e propriedade do `ctop`, binding SQLite do Agentlytics e saída antecipada do
Desktop sem TTY.

## Aprendizados do Ubuntu 26.04 (outubro de 2026)

### PHP 8.5 nativo no Ubuntu 26.04+ (resolute)

O Ubuntu 26.04 LTS inclui o PHP 8.5 nativamente em seus repositórios oficiais.
O PPA `ppa:ondrej/php` não possui pacotes para essa distribuição e retorna
HTTP 404, gerando falha no `apt-get update` e bloqueando todas as etapas
seguintes no modo estrito `set -Eeuo pipefail`. O instalador agora verifica se
`php${PHP_VERSION}` já existe no cache do APT antes de tentar adicionar o PPA.
Além disso, fontes legadas no formato deb822 (`ondrej-ubuntu-php-*.sources`)
são removidas na limpeza inicial de repositórios.

### Ansible nativo no Ubuntu 26.04+

O metapacote `ansible` está disponível no repositório `universe` do Ubuntu 26.04.
Assim como no PHP, a adição do PPA externo é evitada quando a distribuição
já fornece o pacote nativamente.

### Binário do utilitário Rust Trippy

O crate Cargo `trippy` instala o executável com o nome `trip`. A checagem de
existência de ferramenta agora avalia `command -v trip` para evitar
recompilações repetidas e garantir a idempotência da etapa de rede.

### Guarda Fail-Fast para distribuições não suportadas

O instalador `desktop.sh` agora valida antecipadamente a presença de `apt-get` e
`dpkg`, além de validar se a distribuição pertence à família Debian/Ubuntu
suportada (`ubuntu` ou `debian`). Tentativas de execução em distribuições não
suportadas (como Fedora, Arch, openSUSE ou Alpine) são rejeitadas de forma
imediata com erro claro, prevenindo falhas parciais por comandos inexistentes.

### Diagnóstico de systemd no WSL

Em ambientes WSL (Windows Subsystem for Linux), o `desktop.sh` detecta a
plataforma e emite um alerta orientativo caso o `systemd` não esteja em
execução como PID 1, informando como ativar `systemd=true` no `/etc/wsl.conf`
para o correto funcionamento de Docker, SSH, cron e PM2.

### Suporte a Bubblewrap na base do sistema

O pacote `bubblewrap` (`bwrap`) foi adicionado aos pacotes essenciais de sistema
tanto no `desktop.sh` quanto no `server.sh`. Isso estabelece o alicerce de
sandboxing para execução segura e isolada de agentes de codificação por IA
(como `ai-jail`).

### Ecossistema de IA: OpenCode, ai-memory e ai-jail

A etapa de ferramentas de IA (Etapa 13 no Desktop e Etapa 7 no Server) foi
expandida para incluir:

- `opencode`: instalado diretamente pelo instalador oficial e symlinkado para
  `~/.local/bin/opencode`;
- `ai-memory`: instalado pelo instalador oficial do GitHub Releases e ativado
  via `ai-memory setup --yes`;
- `ai-jail`: binário nativo pré-compilado via GitHub Releases (com verificação
  SHA-256) e bootstrap de permissões para os agentes (`ai-jail --bootstrap`);
- Aliases e wrapper de shell para execução com rede ativada e mapeamento de
  credenciais (`ai-jail agy`, `jagy`, `jclaude`, `jcodex`, `jopencode`).

#### Fluxo de Autenticação Manual das Ferramentas de IA (Host-First)

Por design de segurança estrito, o `ai-jail` isola sockets de exibição gráfica
(X11/Wayland), D-Bus e acessos ao sistema de arquivos hospedeiro (inclusive a
interoperabilidade com navegadores do Windows via `/mnt/c` no WSL 2). Dessa
forma, tentativas de autenticação interativa via navegador a partir de dentro
do sandbox falham ao disparar `xdg-open` ou redirecionar o callback OAuth
para o loopback `127.0.0.1`.

O modelo adotado pelo SetupVibe segue o padrão *Pre-authenticated Agent*:

1. **Login Inicial no Host:** O desenvolvedor executa o comando nativo
   diretamente no terminal do sistema hospedeiro (`claude`, `codex`, `opencode`,
   `agy`). O navegador do sistema abre normalmente, conclui o fluxo OAuth e
   grava as credenciais persistentes no `$HOME` (`~/.claude.json`, `~/.codex/`,
   `~/.config/opencode`, `~/.local/share/opencode`, `~/.gemini/`);
2. **Execução Segura no Sandbox:** O desenvolvedor encerra a ferramenta no host
   e passa a utilizá-la via aliases isolados (`jclaude`, `jcodex`, `jopencode`,
   `jagy`);
3. **Mapeamento e Renovação Contínua:** A função wrapper `ai-jail()` monta as
   pastas de credenciais em modo leitura e escrita (`rw`) usando `--agent-state`
   e `--rw-map`. Assim, os agentes iniciam já logados e renovam tokens de acesso
   via chamadas HTTPS de segundo plano gravando diretamente no disco do host.

#### Idempotência de Configurações de Shell (`.bashrc` e `.zshrc`)

Para que re-execuções do instalador em uma máquina ou VM existente não dupliquem
funções de shell nem deixem de atualizar versões anteriores, o SetupVibe adota:

- `~/.zshrc`: sobrescrita determinística a partir dos templates versionados
  (`conf/zshrc-linux.zsh`, `conf/zshrc-server.zsh`);
- `~/.bashrc`: injeção idempotente delimitada por marcadores exclusivos
  (`# >>> setupvibe ai-jail >>>` e `# <<< setupvibe ai-jail <<<`). Quando o
  bloco já existe, seu conteúdo é substituído de forma limpa sem tocar nas
  demais customizações do usuário.

### Captura granular de falhas e logs por etapa

Cada etapa agora transmite sua saída normalmente para o terminal enquanto
grava um log dedicado em `~/.setupvibe/logs/step_N.log`. Se a etapa for bem-sucedida,
o arquivo temporário é removido. Se houver falha (`status != 0`):

1. A etapa é marcada como `✘ Error`;
2. O log é preservado como `~/.setupvibe/logs/step_N_error.log` com propriedade
   do usuário real;
3. O resumo final lista o trecho exato de erro das últimas linhas e o caminho do
   log completo, eliminando a necessidade de rolar manualmente o terminal.

### Permissões de diretórios intermediários com `install -d` e auto-cura de `~/.setupvibe`

O utilitário GNU `install -d -o <user> -g <group> -m <mode> <dir>` aplica modo e
titularidade unicamente ao argumento final, criando componentes intermediários
ausentes com umask padrão e a identidade do processo executor (`root` ou
`nobody:nogroup` em namespaces WSL/contêineres).

Quando a preparação de logs em `run_section()` passava apenas `~/.setupvibe/logs`,
o diretório pai `~/.setupvibe` era gerado pertencendo ao `root`/`nobody` com modo
`0755`, impedindo qualquer comando desprivilegiado posterior de criar subpastas
(`portainer_data`, `bin`, `tmp`) com `Permission denied`.

A correção adotada:

- `run_section()` em `desktop.sh` e `server.sh` passa explicitamente o diretório
  pai e o diretório filho para o `sys_do install -d` e aplica um `chown` imediato
  para recuperar ambientes previamente corrompidos;
- `install_setupvibe_bin`, configuração de Portainer, `install_herdr` e
  `install_antigravity` asseguram o diretório gerenciado e suas subpastas com
  `install -d` explícito;
- A suíte de testes em `tests/` valida estaticamente e dinamicamente a ausência
  de `mkdir` desprotegido em `~/.setupvibe` e a capacidade de auto-cura de
  permissões restritivas.
