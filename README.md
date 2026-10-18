# SetupVibe Linux Edition

> Prepare. Revise. Desenvolva. — v0.41.10

SetupVibe é um projeto open source para instalar e configurar ferramentas de terminal, runtimes, CLIs de IA e utilitários de operação em um único comando.

Este repositório (`setupvibelinux`) é uma edição aprimorada com foco na família **Debian/Ubuntu** (Ubuntu 24.04+, Ubuntu 26.04+ Resolute, Debian 12+, Zorin OS, Mint e WSL 2) e macOS, trazendo compatibilidade avançada com kernels e pacotes modernos, suporte nativo a **Sandboxing Seguro para Agentes de IA (`ai-jail` + Bubblewrap)**, **Memória Contínua Multiagente (`ai-memory`)**, **OpenCode (`opencode`)**, **desacoplamento para execução local offline e remota direta via GitHub**, além de **captura granular de falhas e logs por etapa**.

Com suporte para Linux, WSL 2, macOS e Windows 11, o SetupVibe oferece três edições dedicadas:

- **Desktop Edition (`desktop.sh`):** Um ambiente de desenvolvimento completo para macOS 12+, distribuições Linux desktop (Ubuntu, Debian, Zorin OS, Mint) e WSL 2, incluindo ecossistemas completos de linguagens, shells de desenvolvimento, multiplexação de terminal e CLIs de IA.
- **Windows Edition (Beta) (`desktop.ps1`):** Uma configuração nativa para Windows 11 22H2+ de 64 bits, instalando utilitários essenciais, Python 3.14 e Node.js 24 LTS oficiais, CLIs de IA nativas, OpenSSH autônomo e configurando a base do WSL 2 sem alterar os perfis existentes do PowerShell.
- **Server Edition (`server.sh`):** Uma configuração enxuta e headless para servidores Linux e VMs na nuvem, com foco em Docker, cluster Swarm automatizado, Ansible, monitoramento e execução remota de agentes de IA.

---

## Quando Usar Cada Edição

A escolha da edição adequada garante uma configuração limpa e ajustada ao papel da máquina, sem sobrecarga desnecessária:

### 1. Estações de Trabalho e Desenvolvimento Local (`Desktop Edition`)

Use o `desktop.sh` ao configurar uma estação principal de desenvolvimento ou um ambiente Linux dentro do WSL 2:

- **Sistemas Suportados:** macOS 12 Monterey+, Ubuntu 24.04+/26.04+, Debian 12+, Zorin OS 18+, Linux Mint 21+ e distribuições WSL 2.
- **Melhor Uso:** Fluxos de trabalho de engenharia web, APIs, sistemas e engenharia de IA. Instala runtimes de linguagens completos, extensões de shell para desenvolvedores, bancos de dados/caches locais, gerenciamento de processos com PM2 e multiplexadores de terminal.

### 2. Máquinas Host com Windows 11 (`Windows Edition - Beta`)

Use o `desktop.ps1` ao preparar um host Windows 11 nativo:

- **Sistemas Suportados:** Windows 11 22H2+ nativo de 64 bits (Build 22621+).
- **Melhor Uso:** Instalação de runtimes oficiais de Python e Node.js, configuração de CLIs nativas de IA (Claude, Codex, Antigravity, Skills), configuração de serviços OpenSSH autônomos e preparação da máquina host para WSL 2 (rede espelhada, tunelamento de DNS, recuperação de memória RAM), mantendo os perfis nativos do PowerShell intactos. Inclui modo completo `-Uninstall` para reversões limpas.

### 3. Servidores em Nuvem, VPS e Executores de CI/CD (`Server Edition`)

Use o `server.sh` em máquinas Linux headless (sem interface gráfica):

- **Sistemas Suportados:** Ubuntu 24.04+, Debian 12+, Zorin OS 18+ (x86_64 e ARM64).
- **Melhor Uso:** Servidores de produção, ambientes de homologação (staging), nós de VPS e executores de CI/CD. Foco estrito em ferramentas de DevOps (Docker, cluster Docker Swarm opcional, Ansible, Tailscale, Tmux enxuto e execução de agentes de IA), sem instalar frameworks de desktop, Homebrew ou toolchains de linguagens desnecessárias.

---

## Recursos e Capacidades Principais

### Engenharia de IA e Ecossistema Multiagente

- **CLIs Nativas de IA:** Instala Anthropic Claude Code, OpenAI Codex CLI, Google Antigravity CLI (`agy`), OpenCode (`opencode`), a [Vercel Labs Skills CLI](https://github.com/vercel-labs/skills) e Spec-Kit (`specify`).
- **Sandbox Seguro e Isolamento (ai-jail):** Instala o `ai-jail` com `bubblewrap` e Landlock, incluindo gerador automático de permissões (`--bootstrap`), wrapper inteligente com rede ativada e atalhos rápidos de terminal (`jagy`, `jclaude`, `jcodex`, `jopencode`).
- **Memória Contínua Multiagente (ai-memory):** Instala e inicializa o `ai-memory` nativo com servidor MCP e serviço de fundo, conectando sessões e histórico entre todos os agentes.
- **Multiplexador de Terminal Herdr:** Instala o [Herdr](https://github.com/herdrdev/herdr) nas edições Unix Desktop e Server, permitindo executar, observar e gerenciar múltiplos agentes de codificação por IA simultaneamente em paralelo. Consulte o [Guia do Herdr](docs/pt-br/HERDR.md).
- **Instalação Segura do Antigravity:** Na Desktop Edition para Unix, resolve e valida o manifesto oficial da versão via SHA-512 diretamente para executar `agy install --skip-aliases --skip-path`, garantindo que os perfis de shell do usuário permaneçam limpos.

### Runtimes e Gerenciadores de Pacotes Verificados

- **Ecossistema Node.js:** Node.js 24 LTS (verificado via SHA-256 e Authenticode no Windows, APT no Debian/Ubuntu, Homebrew no macOS), juntamente com Bun, PNPM e ferramentas globais do npm.
- **Ecossistema Python:** Python 3.14 no macOS e Windows (instalado para todos os usuários com assinatura Authenticode oficial); `uv` e pacote `qrcode` do PyPI global no nível do usuário com o comando de terminal `qr` no Unix.
- **Ecossistema PHP:** PHP 8.5 com detecção nativa de pacotes no Ubuntu 26.04+ (evitando PPAs externos incompatíveis), conjunto completo de extensões, Composer e instalador do Laravel.
- **Ruby, Go e Rust:** Ruby 3.4.10 via `rbenv` e `ruby-build` com Rails; Go 1.26.5; toolchain Rust via `rustup` e `cargo`.
- **Gerenciadores de Pacotes do Sistema:** Gerenciamento nativo de pacotes via WinGet e Chocolatey no Windows, Homebrew no macOS/Linux e APT em distribuições baseadas em Debian.

### Terminal Moderno e Multiplexação

- **Ambiente de Shell Unix:** ZSH configurado com Oh My Zsh, prompt Starship, navegação inteligente `zoxide`, ativação do gerenciador de runtimes `mise`, sugestões automáticas (autosuggestions), realce de sintaxe (syntax highlighting) e busca fuzzy com fzf.
- **Shell do Windows Preservado:** Mantém os perfis do Windows PowerShell e PowerShell 7 completamente intocados, evitando substituições do Starship ou de políticas de execução na máquina host Windows. Instala e valida o Windows Terminal.
- **Tmux e Plugins TPM:** Tmux pré-configurado nas edições Unix com mais de 20 plugins, tema Onedark, persistência contínua de sessões (`tmux-resurrect` e `tmux-continuum`) e numeração de janelas/painéis iniciando em 1. Consulte o [Guia do Tmux](docs/desktop/pt-br/tmux.md).
- **Gerenciamento de Processos:** Gerenciamento automatizado de processos com PM2 e inicialização automática nativa da plataforma (LaunchAgents no macOS, systemd no Linux). Consulte o [Guia do PM2](docs/desktop/pt-br/pm2.md).

### Operações, Containers e Redes

- **Containerização e Clusters:** Instala Docker Engine, Docker Compose e oferece suporte à inicialização automatizada de clusters Docker Swarm na Server Edition via `--manager`.
- **Automação de Infraestrutura:** Ansible com verificações nativas de pacotes em distribuições modernas, além do GitHub CLI (`gh`) para fluxos de trabalho ágeis com repositórios.
- **OpenSSH Enterprise no Windows:** Baixa o MSI oficial do Win32-OpenSSH diretamente dos releases do GitHub, valida assinaturas Authenticode, instala cliente e servidor, inicia o `sshd` e habilita a porta TCP 22 no firewall sem dependência do Windows Update ou de Recursos sob Demanda (Features on Demand).
- **Rede Mesh e Diagnósticos:** Rede segura com Tailscale, além de utilitários modernos de diagnóstico de rede, incluindo `bandwhich`, `gping`, `trippy` (`trip`) e `rustscan`.
- **Utilitários Unix Modernos:** Ferramentas de terminal rápidas e modernas, incluindo `bat`, `eza`, `fzf`, `ripgrep` e `glow`.

### Otimização para WSL 2

- **Configuração Base Sem Impor Distribuição:** Configura o motor do WSL 2 sem forçar uma distribuição Linux específica na máquina host Windows.
- **Integração com o Host:** Configura o `.wslconfig` com rede espelhada (compatível com VPN e LAN), acesso de entrada no firewall Hyper-V, tunelamento de DNS, integração com proxy do Windows, liberação automática de memória RAM e armazenamento VHD esparso.

### Diagnóstico Granular de Falhas e Logs

- **Transmissão e Captura em Tempo Real:** Toda etapa transmite a saída ao vivo para o terminal e grava simultaneamente em `~/.setupvibe/logs/step_N.log`.
- **Relatório Preciso de Erros:** Se uma etapa falhar (`exit code != 0`), o log é preservado em `~/.setupvibe/logs/step_N_error.log` com permissão do usuário. O resumo final lista o código de saída, as últimas 6 linhas da falha e o caminho exato do log.
- **Fail-Fast e Diagnóstico WSL:** Rejeita imediatamente distribuições fora da família Debian/Ubuntu suportada e emite aviso orientativo se o `systemd` não estiver ativo como PID 1 no WSL 2.

### Manutenção Segura, Separação de Privilégios e Reversão

- **Gerenciamento Inteligente de Privilégios:** Utiliza `sudo` estritamente quando privilégios elevados são exigidos; todos os binários em nível de usuário são posicionados de forma limpa em `$HOME/.local/bin`.
- **Segurança na Manutenção do Windows:** Solicita confirmação antes de encerrar instaladores conflitantes, executa `sfc.exe /scannow`, rejeita reinicializações pendentes do CBS/Windows Update e valida o repositório de componentes do Windows (component store).
- **Modo Completo de Desinstalação:** A Windows Edition inclui o parâmetro `-Uninstall` para remover de forma limpa os utilitários gerenciados, restaurar as regras de firewall anteriores do WSL/OpenSSH e limpar entradas obsoletas do PATH, preservando os dados do usuário e as distribuições WSL.

---

## Matriz de Edições

| Recurso | Desktop Edition | Windows Edition (Beta) | Server Edition |
| --- | --- | --- | --- |
| Sistema Operacional Alvo | macOS 12+, Linux, WSL 2 | Windows 11 22H2+ (x64) | Ubuntu 24.04+, Debian 12+ |
| Script de Execução | `desktop.sh` | `desktop.ps1` | `server.sh` |
| Gerenciador de Pacotes Principal | Homebrew / APT | WinGet + Chocolatey | APT |
| OpenSSH Cliente e Servidor | Cliente (Servidor no Linux) | MSI oficial Win32-OpenSSH | Cliente + Servidor |
| Base Espelhada WSL 2 | No WSL / Pronto para o host | Configurado (`.wslconfig`) | ✗ |
| Node.js 24 LTS | ✔ | ✔ | ✔ (APT) |
| Bun e PNPM | ✔ | ✗ | ✗ |
| Python 3.14 / uv | ✔ | ✔ (MSI 3.14) | ✗ |
| PHP 8.5, Composer, Laravel | ✔ | ✗ | ✗ |
| Ruby 3.4.10 e Rails | ✔ | ✗ | ✗ |
| Go 1.26.5 e Rust | ✔ | ✗ | ✗ |
| Claude Code e Codex CLI | ✔ | ✔ | ✔ |
| Google Antigravity CLI (`agy`) | ✔ | ✔ | ✗ |
| OpenCode (`opencode`) | ✔ | ✗ | ✔ |
| Sandbox Seguro (`ai-jail` + `bubblewrap`) | ✔ | ✗ | ✔ |
| Memória Contínua MCP (`ai-memory`) | ✔ | ✗ | ✔ |
| Vercel Labs Skills CLI | ✔ | ✔ | ✔ |
| Multiplexador de Agentes Herdr | ✔ | ✗ | ✔ |
| ZSH + Oh My Zsh + Starship | ✔ | ✗ (Preserva PowerShell) | ✔ |
| Tmux + Plugins TPM | ✔ (20+ plugins) | ✗ | ✔ (Configuração enxuta) |
| Gerenciador de Processos PM2 | ✔ (Inicialização automática) | ✗ | ✗ |
| Docker e Ansible | ✔ | Somente GitHub CLI | ✔ |
| Gerenciador Docker Swarm (`--manager`) | ✗ | ✗ | ✔ |
| VPN Mesh Tailscale | ✔ | ✔ | ✔ |
| Diagnóstico Granular de Erros e Logs | ✔ | ✔ (Logs Windows) | ✔ |
| Modo de Desinstalação Reversível | Manual | ✔ (`-Uninstall`) | Manual |

---

## Fluxo de Trabalho Recomendado

O SetupVibe segue um fluxo de trabalho verificável: **Prepare. Revise. Desenvolva.**

1. **Escolha a Edição e Revise:** Revise o script correspondente à plataforma de destino para verificar cada modificação de pacote e arquivo antes da execução.
2. **Execute a Instalação:** Execute o instalador canônico em uma linha ou execute a partir de um clone local.
3. **Valide a Configuração:** Confirme as versões dos runtimes (`node -v`, `python -v`, `agy --version`, `claude --version`, etc.) e verifique se os serviços relevantes (`sshd`, `pm2`, `docker`) estão em execução.
4. **Desenvolva e Orquestre:** Inicie seu shell de desenvolvimento ou multiplexador de terminal (`tmux`, `herdr`) para coordenar fluxos de trabalho locais e multiagente.

---

## Início Rápido

### Desktop Edition (macOS, Linux e WSL)

Execute o script diretamente via `curl`:

```bash
curl -fsSL https://raw.githubusercontent.com/jjeffersonlima/setupvibelinux/main/desktop.sh | bash
```

Ou execute localmente a partir deste repositório clonado:

```bash
git clone https://github.com/jjeffersonlima/setupvibelinux.git
cd setupvibelinux
bash desktop.sh
```

### Windows Edition (Beta)

Execute o instalador de utilitários do Windows diretamente via PowerShell (ele solicita elevação padrão via UAC sem alterar a política do sistema):

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; irm https://raw.githubusercontent.com/jjeffersonlima/setupvibelinux/main/desktop.ps1 | iex
```

Ou execute localmente a partir deste repositório clonado:

```powershell
git clone https://github.com/jjeffersonlima/setupvibelinux.git
cd setupvibelinux
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\desktop.ps1
```

Para remover todos os utilitários e configurações gerenciados pela Windows Edition:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\desktop.ps1 -Uninstall
```

### Server Edition (Somente Linux)

Execute o instalador enxuto para servidores via `curl`:

```bash
curl -fsSL https://raw.githubusercontent.com/jjeffersonlima/setupvibelinux/main/server.sh | bash
```

Para inicializar automaticamente um nó gerenciador (manager) do Docker Swarm após a conclusão:

```bash
curl -fsSL https://raw.githubusercontent.com/jjeffersonlima/setupvibelinux/main/server.sh | bash -s -- --manager
```

Ou execute localmente a partir deste repositório clonado:

```bash
git clone https://github.com/jjeffersonlima/setupvibelinux.git
cd setupvibelinux
bash server.sh
```

Opções adicionais para automação em servidores:

- `--yes`: Instalação não assistida, sem solicitações interativas de confirmação.
- `--advertise-addr ENDEREÇO`: Especifica explicitamente o endereço IP ou interface de rede para o Docker Swarm.

---

## Documentação

| Documento | Descrição | Link |
| --- | --- | --- |
| Visão Geral | Comparações abrangentes de edições e guias | [docs/README.md](docs/README.md) |
| Desktop Edition | Guia detalhado de configuração de workstation em 14 etapas | [docs/desktop/README.md](docs/desktop/README.md) |
| Windows Edition (Beta) | Configuração nativa do Windows 11, manutenção e reversão | [docs/windows/README.md](docs/windows/README.md) |
| Server Edition | Guia de operações e Swarm enxuto em 9 etapas | [docs/server/README.md](docs/server/README.md) |
| Multiplexador Herdr | Referência de orquestração de agentes de terminal | [docs/pt-br/HERDR.md](docs/pt-br/HERDR.md) |
| Guia do Tmux | Atalhos, plugins e gerenciamento de sessões | [docs/desktop/pt-br/tmux.md](docs/desktop/pt-br/tmux.md) |
| Guia do PM2 | Gerenciamento de processos, clusters e persistência | [docs/desktop/pt-br/pm2.md](docs/desktop/pt-br/pm2.md) |
| Guia do Cronboard | Monitoramento de tarefas cron e referência do dashboard | [docs/desktop/pt-br/cronboard.md](docs/desktop/pt-br/cronboard.md) |
| Utilitários Executáveis | Scripts CLI compartilhados implantados em `~/.setupvibe/bin` | [docs/pt-br/EXECUTABLES.md](docs/pt-br/EXECUTABLES.md) |
| Engenharia Debian | Compatibilidade com Debian e Ubuntu, PPAs e testes em containers | [DEBIAN.md](DEBIAN.md) |
| Identidade de Marca | Identidade visual, tokens de design, tom de voz e logos | [brand/README.md](brand/README.md) |

---

## Marca

Os logos oficiais claro e escuro, tokens de cores, fontes web locais, descrição de marca e guia de voz estão em [`brand/`](brand/). Utilize os assets SVG e tokens semânticos por padrão e siga o checklist de publicação em [`brand/checklist.md`](brand/checklist.md) antes de publicar materiais públicos.

---

## Como Contribuir

Contribuições de todos os tamanhos são bem-vindas! Leia nosso [Guia de Contribuição](CONTRIBUTING.md) para começar.

---

Fork mantido por [Jefferson Lima](https://github.com/jjeffersonlima/setupvibelinux) (baseado no [SetupVibe](https://github.com/promovaweb/setupvibe)) · Licenciado sob [GPL-3.0](LICENSE)
