# tmux

`tmux` é um multiplexador de terminal que permite executar múltiplas sessões de shell dentro de uma única janela, alternar entre elas e mantê-las em execução após a desconexão do terminal. É amplamente utilizado em servidores remotos via SSH e em fluxos de trabalho locais que exigem múltiplos contextos simultâneos.

O prefixo padrão de todos os atalhos é `Ctrl+b`, seguido da tecla correspondente ao comando.

## Sumário

- [Conceitos fundamentais](#conceitos-fundamentais)
- [Sessões](#sessões)
- [Janelas](#janelas)
- [Painéis](#painéis)
- [Modo de cópia](#modo-de-cópia)
- [Configuração](#configuração)

---

## Conceitos fundamentais

O tmux organiza o trabalho em três níveis hierárquicos: **sessões**, que representam um contexto completo de trabalho persistente; **janelas**, equivalentes a abas dentro de uma sessão; e **painéis**, divisões da área visível de uma janela. Uma sessão sobrevive ao fechamento do terminal e pode ser retomada a qualquer momento, o que torna o tmux útil para trabalhos longos em máquinas remotas.

```
Sessão
├── Janela 1
│   ├── Painel esquerdo
│   └── Painel direito
└── Janela 2
    └── Painel único
```

---

## Sessões

### new-session

Cria uma nova sessão independente. Sessões nomeadas facilitam a identificação e o `attach` posterior; a flag `-d` cria a sessão em segundo plano, sem anexá-la ao terminal atual.

- `tmux` — cria e anexa a uma nova sessão
- `tmux new -s nome` — cria sessão nomeada
- `tmux new -s nome -d` — cria em background

### attach-session

Reconecta o terminal a uma sessão existente, restaurando o estado das janelas e painéis tal como estavam no momento do `detach`.

- `tmux attach` — anexa à última sessão
- `tmux a -t nome` — anexa à sessão nomeada

### detach

Desvincula o terminal da sessão sem encerrá-la. Os processos continuam em execução em segundo plano.

- `Ctrl+b d` — atalho de detach
- `tmux detach` — equivalente via linha de comando

### kill-session

Encerra uma sessão e todos os processos associados aos seus painéis. `kill-server` finaliza o próprio servidor tmux, destruindo todas as sessões ativas.

- `tmux kill-session -t nome`
- `tmux kill-server` — encerra o servidor inteiro

### list-sessions

Lista as sessões ativas com nome, número de janelas e estado de anexação.

- `tmux ls` — lista no terminal
- `Ctrl+b s` — lista interativa dentro do tmux

---

## Janelas

### new-window

Cria uma nova janela na sessão atual, análoga a uma nova aba. A janela criada herda o diretório de trabalho da janela de origem.

- `Ctrl+b c` — nova janela
- `tmux new-window -n nome` — janela nomeada

### rename-window

Define o rótulo da janela exibido na barra de status, útil para identificar o propósito de cada aba.

- `Ctrl+b ,` — prompt interativo
- `tmux rename-window nome`

### next-window / prev-window

Alterna o foco entre janelas da sessão. O acesso direto por número é o atalho mais rápido quando há poucas janelas.

- `Ctrl+b n` / `Ctrl+b p` — próxima / anterior
- `Ctrl+b 0-9` — acesso direto por índice
- `Ctrl+b w` — seletor interativo

### kill-window

Fecha a janela atual e todos os seus painéis. Pede confirmação por padrão.

- `Ctrl+b &`
- `tmux kill-window`

---

## Painéis

### split-window

Divide a janela atual em dois painéis, horizontal ou verticalmente. O novo painel inicia no mesmo diretório do painel original quando configurado adequadamente.

- `Ctrl+b %` — divisão vertical (lado a lado)
- `Ctrl+b "` — divisão horizontal (acima/abaixo)
- `tmux split-window -h` / `-v`

### select-pane

Move o foco do teclado entre os painéis da janela. A navegação por setas é o padrão; atalhos vim requerem `mode-keys vi` no `~/.tmux.conf`.

- `Ctrl+b ←↑↓→` — direcional
- `Ctrl+b o` — próximo painel
- `Ctrl+b q` — exibe números; `Ctrl+b q N` salta ao painel N

### resize-pane

Ajusta as dimensões do painel atual. Os passos com `Ctrl` são finos; com `Alt`, grossos.

- `Ctrl+b Ctrl+←↑↓→` — passo de 1
- `Ctrl+b Alt+←↑↓→` — passo de 5
- `tmux resize-pane -L|R|U|D N` — N células na direção

### kill-pane

Encerra o painel atual. Sair do shell (`exit`) tem o mesmo efeito.

- `Ctrl+b x`
- `tmux kill-pane`

### zoom

Expande temporariamente o painel atual para ocupar toda a janela, preservando o layout original para restauração posterior.

- `Ctrl+b z` — alterna zoom

---

## Modo de cópia

O modo de cópia permite rolar o histórico do buffer e selecionar texto sem o mouse. Com `mode-keys vi`, a navegação e a seleção seguem as convenções do vim.

### Entrar e sair

- `Ctrl+b [` — entra no modo
- `q` — sai do modo

### Navegação (vi)

- `h j k l` — movimento do cursor
- `Ctrl+u` / `Ctrl+d` — meia página acima / abaixo
- `gg` / `G` — início / fim do buffer
- `/termo` `?termo` — busca adiante / atrás; `n` / `N` repete

### Seleção e cópia

- `v` — inicia seleção
- `y` — copia para o buffer do tmux
- `Ctrl+b ]` — cola

Para integrar com o clipboard do sistema:

```tmux
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard"
```

---

## Configuração

### ~/.tmux.conf

Arquivo lido na inicialização do servidor, define opções globais, atalhos e aparência. Opções comuns:

```tmux
set  -g mouse on
setw -g mode-keys vi
set  -g history-limit 10000
set  -g base-index 1
set  -g status-position top
bind r source-file ~/.tmux.conf
```

### source-file

Recarrega o arquivo de configuração sem reiniciar o servidor, aplicando alterações às sessões em execução.

- `tmux source-file ~/.tmux.conf`
- `Ctrl+b :source ~/.tmux.conf`
- `Ctrl+b r` — se vinculado no `~/.tmux.conf`
