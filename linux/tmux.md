# Manual de tmux — Multiplexador de Terminal

**Data de criação:** 2026-03-27
**Descrição:** Documento de referência para uso do tmux no dia a dia, cobrindo sessões, janelas, painéis, modo de cópia e configuração, com foco operacional e prático.

---

## Sumário

- Conceitos fundamentais
  - Sessões, janelas e painéis

- Sessões
  - `new-session`
  - `attach-session`
  - `detach`
  - `kill-session`
  - `list-sessions`

- Janelas
  - `new-window`
  - `rename-window`
  - `next-window` / `prev-window`
  - `kill-window`

- Painéis
  - `split-window`
  - `select-pane`
  - `resize-pane`
  - `kill-pane`
  - `zoom`

- Modo de cópia
  - Entrar no modo
  - Navegação vi
  - Seleção e cópia

- Configuração
  - `~/.tmux.conf`
  - `source-file`

---

## Corpo do documento

## Conceitos fundamentais

### Sessões, janelas e painéis

**Para que serve:**
Entender a hierarquia do tmux é essencial para usá-lo bem.

**Como funciona:**
O tmux organiza o trabalho em três níveis:

- **Sessão** — contexto completo de trabalho, persiste mesmo após fechar o terminal
- **Janela** — equivalente a uma aba dentro de uma sessão
- **Painel** — divisão da tela dentro de uma janela

```
Sessão
└── Janela 1
│   ├── Painel esquerdo
│   └── Painel direito
└── Janela 2
    └── Painel único
```

**Observação operacional:**
O prefixo padrão do tmux é `Ctrl+b`. Todos os atalhos são acionados pressionando o prefixo e depois a tecla indicada.

---

## Sessões

### new-session

**Para que serve:**
Cria uma nova sessão tmux.

**Como funciona:**
Inicia um novo contexto de trabalho independente, que pode ser retomado posteriormente.

**Principais formas de uso:**
- `tmux` — cria e entra em uma nova sessão
- `tmux new-session` — equivalente ao anterior
- `tmux new-session -s nome` — cria sessão com nome
- `tmux new -s nome` — forma abreviada
- `tmux new -s nome -d` — cria em background sem entrar

---

### attach-session

**Para que serve:**
Reconecta a uma sessão existente.

**Como funciona:**
Retoma uma sessão que foi detachada ou criada em background, restaurando o estado exato das janelas e painéis.

**Principais formas de uso:**
- `tmux attach` — reconecta à última sessão
- `tmux attach -t nome` — reconecta a uma sessão específica
- `tmux a` — forma abreviada de attach
- `tmux a -t nome` — forma abreviada com nome

---

### detach

**Para que serve:**
Sai da sessão sem encerrá-la.

**Como funciona:**
Desvincula o terminal da sessão, que continua rodando em background.

**Principais formas de uso:**
- `Ctrl+b d` — atalho para detach
- `tmux detach` — via linha de comando

---

### kill-session

**Para que serve:**
Encerra uma sessão e todos os seus processos.

**Como funciona:**
Mata todos os painéis e janelas da sessão especificada.

**Principais formas de uso:**
- `tmux kill-session -t nome` — encerra sessão pelo nome
- `Ctrl+b :kill-session` — via prompt interno do tmux
- `tmux kill-server` — encerra todas as sessões e o servidor tmux

---

### list-sessions

**Para que serve:**
Lista todas as sessões ativas.

**Como funciona:**
Exibe nome, número de janelas e status de cada sessão em execução.

**Principais formas de uso:**
- `tmux list-sessions` — lista completa
- `tmux ls` — forma abreviada
- `Ctrl+b s` — abre lista interativa dentro do tmux

---

## Janelas

### new-window

**Para que serve:**
Cria uma nova janela na sessão atual.

**Como funciona:**
Abre uma nova aba dentro da sessão, mantendo as janelas anteriores intactas.

**Principais formas de uso:**
- `Ctrl+b c` — atalho para nova janela
- `tmux new-window` — via linha de comando
- `tmux new-window -n nome` — com nome definido

---

### rename-window

**Para que serve:**
Renomeia a janela atual.

**Como funciona:**
Altera o rótulo exibido na barra de status inferior do tmux.

**Principais formas de uso:**
- `Ctrl+b ,` — abre prompt para renomear
- `tmux rename-window nome` — via linha de comando

---

### next-window / prev-window

**Para que serve:**
Navega entre janelas da sessão.

**Como funciona:**
Alterna o foco para a janela seguinte ou anterior na ordem da barra de status.

**Principais formas de uso:**
- `Ctrl+b n` — próxima janela
- `Ctrl+b p` — janela anterior
- `Ctrl+b 0-9` — vai direto para a janela pelo número
- `Ctrl+b w` — abre lista interativa de janelas

---

### kill-window

**Para que serve:**
Fecha a janela atual e todos os seus painéis.

**Como funciona:**
Encerra todos os processos rodando nos painéis da janela.

**Principais formas de uso:**
- `Ctrl+b &` — encerra com confirmação
- `tmux kill-window` — via linha de comando

---

## Painéis

### split-window

**Para que serve:**
Divide a janela atual em painéis.

**Como funciona:**
Cria um novo painel na mesma janela, dividindo o espaço horizontal ou verticalmente.

**Principais formas de uso:**
- `Ctrl+b %` — divide verticalmente (painel ao lado)
- `Ctrl+b "` — divide horizontalmente (painel abaixo)
- `tmux split-window -h` — divisão horizontal via comando
- `tmux split-window -v` — divisão vertical via comando

**Observação operacional:**
Com a configuração vim do `~/.tmux.conf`, pode-se usar `Ctrl+b v` para vertical e `Ctrl+b s` para horizontal.

---

### select-pane

**Para que serve:**
Move o foco para outro painel.

**Como funciona:**
Altera qual painel recebe entrada do teclado.

**Principais formas de uso:**
- `Ctrl+b seta` — move pelo painel na direção da seta
- `Ctrl+b h/j/k/l` — navegação vim (requer configuração)
- `Ctrl+b q` — exibe números dos painéis temporariamente
- `Ctrl+b q N` — vai para o painel de número N
- `Ctrl+b o` — alterna para o próximo painel

---

### resize-pane

**Para que serve:**
Redimensiona um painel.

**Como funciona:**
Ajusta o tamanho do painel atual na direção especificada.

**Principais formas de uso:**
- `Ctrl+b Ctrl+seta` — redimensiona em passos de 1
- `Ctrl+b Alt+seta` — redimensiona em passos de 5
- `tmux resize-pane -L 10` — reduz 10 colunas à esquerda
- `tmux resize-pane -R 10` — expande 10 colunas à direita
- `tmux resize-pane -U 5` — expande 5 linhas para cima
- `tmux resize-pane -D 5` — reduz 5 linhas para baixo

---

### kill-pane

**Para que serve:**
Fecha o painel atual.

**Como funciona:**
Encerra o processo do painel e libera o espaço na janela.

**Principais formas de uso:**
- `Ctrl+b x` — fecha com confirmação
- `exit` — fecha o painel ao sair do shell
- `tmux kill-pane` — via linha de comando

---

### zoom

**Para que serve:**
Expande um painel para ocupar a janela inteira temporariamente.

**Como funciona:**
Alterna entre o painel em tela cheia e o layout original com múltiplos painéis.

**Principais formas de uso:**
- `Ctrl+b z` — ativa/desativa zoom no painel atual

---

## Modo de cópia

### Entrar no modo

**Para que serve:**
Permite rolar o histórico e copiar texto sem usar o mouse.

**Como funciona:**
Entra em um modo de leitura onde o teclado controla a navegação no buffer de saída.

**Principais formas de uso:**
- `Ctrl+b [` — entra no modo de cópia
- `q` — sai do modo de cópia
- `PgUp` — também entra no modo e sobe uma página

---

### Navegação vi

**Para que serve:**
Navegar pelo histórico do terminal com teclas vim.

**Como funciona:**
Com `mode-keys vi` ativo no `~/.tmux.conf`, o modo de cópia usa atalhos do vim.

**Principais formas de uso:**
- `h j k l` — mover cursor
- `Ctrl+u` — sobe meia página
- `Ctrl+d` — desce meia página
- `gg` — vai para o início do histórico
- `G` — vai para o final
- `/termo` — busca para frente
- `?termo` — busca para trás
- `n` — próxima ocorrência
- `N` — ocorrência anterior

---

### Seleção e cópia

**Para que serve:**
Selecionar e copiar texto do terminal.

**Como funciona:**
Marca uma região do buffer e copia para a área de transferência interna do tmux.

**Principais formas de uso:**
- `v` — inicia seleção (requer configuração vi)
- `y` — copia a seleção
- `Ctrl+b ]` — cola o texto copiado

**Observação operacional:**
Para integrar com a área de transferência do sistema, pode-se adicionar ao `~/.tmux.conf`:
`bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard"`

---

## Configuração

### ~/.tmux.conf

**Para que serve:**
Arquivo de configuração persistente do tmux.

**Como funciona:**
Lido na inicialização do servidor tmux, define opções, atalhos e aparência.

**Principais opções:**
- `set -g mouse on` — habilita suporte ao mouse
- `setw -g mode-keys vi` — ativa modo vi no modo de cópia
- `set -g history-limit 10000` — aumenta o histórico do buffer
- `set -g base-index 1` — janelas começam no índice 1
- `set -g status-position top` — move a barra de status para cima
- `bind r source-file ~/.tmux.conf` — atalho para recarregar config

---

### source-file

**Para que serve:**
Recarrega as configurações do `~/.tmux.conf` sem reiniciar o tmux.

**Como funciona:**
Relê o arquivo de configuração e aplica as mudanças na sessão em execução.

**Principais formas de uso:**
- `tmux source-file ~/.tmux.conf` — via terminal
- `Ctrl+b :source ~/.tmux.conf` — via prompt interno do tmux
- `Ctrl+b r` — se o atalho estiver configurado no `~/.tmux.conf`

---
