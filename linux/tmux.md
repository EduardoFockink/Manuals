# tmux

`tmux` é um multiplexador de terminal: executa várias sessões de shell dentro de uma única janela, alterna entre elas e mantém os processos vivos após a desconexão. É a ferramenta padrão para trabalho persistente em servidores remotos via SSH e para fluxos locais com múltiplos contextos simultâneos.

Todos os atalhos são acionados após uma tecla **prefixo**, padrão `Ctrl+b`. O prefixo pode ser redefinido no `~/.tmux.conf` (`set -g prefix C-Space`, por exemplo).

## Sumário

- [Conceitos fundamentais](#conceitos-fundamentais)
- [Sessões](#sessões)
- [Janelas](#janelas)
- [Painéis](#painéis)
- [Modo de cópia](#modo-de-cópia)
- [Configuração](#configuração)

---

## Conceitos fundamentais

O tmux organiza o trabalho em três níveis hierárquicos:

- **Sessão** — contexto completo e persistente; sobrevive ao fechamento do terminal e pode ser retomada por `attach`.
- **Janela** — equivalente a uma aba dentro de uma sessão.
- **Painel** — divisão da área visível de uma janela, cada um rodando seu próprio shell.

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

### Criar

Sessões nomeadas facilitam o `attach` posterior. A flag `-d` cria em segundo plano, sem anexar ao terminal atual.

- `tmux` — cria e anexa a uma nova sessão sem nome
- `tmux new -s nome` — cria sessão nomeada
- `tmux new -s nome -d` — cria em background

### Anexar e desanexar

`attach` reconecta o terminal a uma sessão existente, restaurando janelas e painéis no estado em que estavam. `detach` desvincula o terminal sem encerrar a sessão — os processos continuam rodando em background.

- `tmux attach` — anexa à última sessão usada
- `tmux a -t nome` — anexa à sessão nomeada
- `Ctrl+b d` — desanexa
- `tmux detach` — equivalente via comando

### Listar e alternar

Diferente de `detach`/`attach`, `switch-client` apenas troca qual sessão é exibida sem desfazer a conexão — essencial quando o tmux é o processo principal da janela do terminal, pois evita que o `detach` feche a janela.

- `tmux ls` — lista as sessões ativas no terminal
- `Ctrl+b s` — seletor interativo de sessões (`choose-session`)
- `Ctrl+b w` — árvore de sessões, janelas e painéis (`choose-tree`); navega com setas, `Enter` seleciona, `x` mata, `/` busca
- `Ctrl+b (` / `Ctrl+b )` — sessão anterior / próxima (ordem alfabética)
- `tmux switch-client -t nome` — troca direto via comando

### Encerrar

`kill-session` encerra uma sessão e todos os processos dos seus painéis. `kill-server` finaliza o servidor inteiro, destruindo todas as sessões ativas.

- `tmux kill-session -t nome`
- `tmux kill-server`

---

## Janelas

### Criar e renomear

A janela criada herda o diretório de trabalho da janela de origem. O rótulo aparece na barra de status.

- `Ctrl+b c` — nova janela
- `tmux new-window -n nome` — janela nomeada
- `Ctrl+b ,` — renomeia (prompt interativo)
- `tmux rename-window nome`

### Navegar

O acesso por índice é o mais rápido com poucas janelas. Para muitas, prefira o seletor por árvore (`Ctrl+b w`, descrito em [Sessões](#listar-e-alternar)).

- `Ctrl+b n` / `Ctrl+b p` — próxima / anterior
- `Ctrl+b 0-9` — acesso direto por índice
- `Ctrl+b l` — alterna com a janela usada por último

### Encerrar

Fecha a janela atual e todos os seus painéis. Pede confirmação por padrão.

- `Ctrl+b &`
- `tmux kill-window`

---

## Painéis

### Dividir

Divide a janela atual. O novo painel inicia no mesmo diretório do original quando os binds usam `-c "#{pane_current_path}"`.

- `Ctrl+b %` — divisão vertical (linha divisória vertical, painéis lado a lado)
- `Ctrl+b "` — divisão horizontal (linha divisória horizontal, painéis empilhados)
- `tmux split-window -h` / `-v`

### Navegar

A navegação por setas é o padrão; binds vim-like (`h j k l`) requerem configuração explícita.

- `Ctrl+b ←↑↓→` — direcional
- `Ctrl+b o` — próximo painel
- `Ctrl+b ;` — alterna com o painel usado por último
- `Ctrl+b q` — exibe os números dos painéis; pressionar `N` em seguida salta ao painel N

### Redimensionar

Passos com `Ctrl` são finos; com `Alt`, grossos.

- `Ctrl+b Ctrl+←↑↓→` — passo de 1
- `Ctrl+b Alt+←↑↓→` — passo de 5
- `tmux resize-pane -L|R|U|D N` — N células na direção indicada

### Reorganizar

Troca a posição dos painéis sem refazer os splits.

- `Ctrl+b {` / `Ctrl+b }` — move o painel atual para a posição anterior / próxima
- `Ctrl+b Ctrl+o` / `Ctrl+b Alt+o` — rotaciona todos no sentido horário / anti-horário
- `Ctrl+b Space` — cicla layouts predefinidos (`even-horizontal`, `even-vertical`, `main-horizontal`, `main-vertical`, `tiled`)
- `tmux swap-pane -s N -t M` — troca o painel N com o M (use `Ctrl+b q` para ver os números)
- `tmux select-layout tiled` — aplica layout específico

### Mover entre janelas

- `Ctrl+b !` — extrai o painel atual em uma janela nova
- `tmux join-pane -s :2` — traz painel da janela 2 para a atual
- `tmux join-pane -t :1` — envia o painel atual para a janela 1

### Zoom

Expande o painel atual para ocupar toda a janela, preservando o layout original. Útil para foco temporário.

- `Ctrl+b z` — alterna zoom

### Encerrar

Sair do shell (`exit`) tem o mesmo efeito.

- `Ctrl+b x`
- `tmux kill-pane`

---

## Modo de cópia

Permite rolar o histórico do buffer e selecionar texto sem o mouse. Com `mode-keys vi`, navegação e seleção seguem as convenções do vim.

### Entrar e sair

- `Ctrl+b [` — entra no modo
- `q` — sai

### Navegação (vi)

- `h j k l` — movimento do cursor
- `Ctrl+u` / `Ctrl+d` — meia página acima / abaixo
- `gg` / `G` — início / fim do buffer
- `/termo` / `?termo` — busca adiante / atrás; `n` / `N` repete

### Seleção e cópia

- `v` — inicia seleção
- `y` — copia para o buffer do tmux
- `Ctrl+b ]` — cola

Integração com o clipboard do sistema (Wayland ou X11):

```tmux
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel \
  "wl-copy 2>/dev/null || xclip -selection clipboard"
```

---

## Configuração

### ~/.tmux.conf

Lido na inicialização do servidor; define opções globais, atalhos e aparência. Opções comuns:

```tmux
set  -g mouse on
setw -g mode-keys vi
set  -g history-limit 10000
set  -g base-index 1
set  -g status-position top
bind r source-file ~/.tmux.conf \; display "Config reloaded"
```

### Recarregar (source-file)

Aplica as alterações às sessões em execução sem reiniciar o servidor.

- `tmux source-file ~/.tmux.conf`
- `Ctrl+b :source ~/.tmux.conf`
- `Ctrl+b r` — se vinculado no `~/.tmux.conf`
