# Hyprland

Compositor Wayland com tiling dinâmico. Gerencia o desenho das janelas (papel do compositor) e o posicionamento automático em grid (papel do tiling WM). Este manual reflete a configuração atual do sistema (Fedora, NVIDIA, Catppuccin Mocha) com `$mod` mapeado para a tecla **Super**.

## Sumário

- [Conceitos fundamentais](#conceitos-fundamentais)
- [Keybinds essenciais](#keybinds-essenciais)
- [Gerenciamento de janelas](#gerenciamento-de-janelas)
- [Workspaces](#workspaces)
- [Layout Dwindle](#layout-dwindle)
- [Monitores](#monitores)
- [Waybar](#waybar)
- [Aplicativos do sistema](#aplicativos-do-sistema)
- [hyprctl](#hyprctl)

---

## Conceitos fundamentais

Vocabulário mínimo para entender atalhos e comportamento. Cada janela aberta cai automaticamente em uma divisão do workspace ativo.

- **Workspace** — área de trabalho virtual; cada workspace pertence a um monitor
- **Tiling** — janelas dividem o espaço disponível automaticamente
- **Floating** — janela solta, fora do tiling
- **Focus** — janela ativa; atalhos atuam sobre ela
- **Dwindle** — algoritmo de divisão em espiral (cada nova janela divide o espaço da anterior ao meio)

---

## Keybinds essenciais

Binds em `~/.config/hypr/keybinds.conf`. Modificador `$mod` é **Super**. Combinações com `Shift` movem janelas; com `Ctrl` redimensionam.

| Atalho | Ação |
|--------|------|
| `Super + Return` | Abre terminal (Alacritty) |
| `Super + Space` | Launcher de apps (wofi) |
| `Super + E` | Gerenciador de arquivos (Nautilus) |
| `Super + L` | Bloqueia tela (hyprlock) |
| `Super + Q` | Fecha janela ativa |
| `Super + F` | Toggle fullscreen |
| `Super + V` | Toggle floating |
| `Super + P` | Toggle pseudotile |
| `Super + H/J/K/L` | Move foco (esq/baixo/cima/dir) |
| `Super + Shift + H/J/K/L` | Move janela ativa no layout |
| `Super + Ctrl + ←↑↓→` | Redimensiona janela (50px por vez) |
| `Super + 1–9` | Vai para workspace N |
| `Super + Shift + 1–9` | Move janela ativa para workspace N |
| `Super + arrastar (esq)` | Move janela flutuante |
| `Super + arrastar (dir)` | Redimensiona janela flutuante |
| `Print` | Captura área (clipboard) |
| `Shift + Print` | Captura tela inteira (clipboard) |
| Teclas de mídia | Volume ±5%, mute, brilho ±5% |

---

## Gerenciamento de janelas

No tiling padrão, o Hyprland posiciona conforme o algoritmo Dwindle. `Super+V` torna a janela independente do grid; o redimensionamento no tiling afeta a divisão entre vizinhas, no floating afeta só a própria. `resize_on_border = true` habilita arrastar borda com mouse.

- `Super + Q` — fecha (envia sinal de encerramento)
- `Super + F` — fullscreen completo (sem borda, sem waybar)
- `Super + V` — toggle floating
- `Super + P` — pseudotile: mantém slot no grid mas exibe no tamanho preferido do app (útil para apps com tamanho fixo)
- `Super + Ctrl + ←↑↓→` — redimensiona 50px por tecla
- `Super + arrastar mouse esq/dir` — move / redimensiona janelas flutuantes

Janelas com float automático (definido em `rules.conf`): `pavucontrol`, `nm-connection-editor`, `blueman-manager`, diálogos "Open File" e "Save As".

---

## Workspaces

Workspaces são criados sob demanda (não existem até o primeiro acesso) e ficam associados ao monitor onde foram acessados. Ao mover uma janela para outro workspace, ela some da tela atual e aparece no destino sem trocar o foco.

- `Super + 1–9` — vai para o workspace N
- `Super + Shift + 1–9` — move janela ativa para workspace N (sem seguir)
- Clique no número da waybar — vai direto para aquele workspace
- Workspace com janela urgente — pisca em vermelho na waybar

---

## Layout Dwindle

Árvore binária em espiral. A primeira janela ocupa a tela toda; cada nova divide o espaço da anterior ao meio, alternando entre divisão horizontal e vertical. Com `preserve_split = true`, a direção da divisão é mantida entre reaberturas.

```
┌───────────────┐    ┌───────┬───────┐    ┌───────┬───┬───┐
│               │    │       │       │    │       │ 2 │ 3 │
│       1       │ →  │   1   │   2   │ →  │   1   ├───┴───┤
│               │    │       │       │    │       │   4   │
└───────────────┘    └───────┴───────┘    └───────┴───────┘
```

- `Super + P` — pseudotile: janela ocupa o slot mas não expande para preenchê-lo
- `Super + Shift + H/J/K/L` — reordena janelas dentro da árvore

---

## Monitores

Configuração base em `monitors.conf` desativa a tela interna (`eDP-1`) e usa o monitor externo como principal. O script `monitor-handler.sh` roda em background ouvindo o socket de eventos via `socat`: ao conectar HDMI, desativa eDP-1; ao desconectar, reativa com resolução preferida.

- Conectar/desconectar HDMI — alternância automática via script
- `hyprctl monitors` — lista monitores conectados e configurações
- `hyprctl keyword monitor "eDP-1, preferred, auto, 1"` — ativa interna manualmente
- `hyprctl keyword monitor "eDP-1, disable"` — desativa interna manualmente

---

## Waybar

Barra de status com workspaces, janela ativa, relógio, rede, volume e bateria. Roda como processo separado (iniciado pelo `exec-once` do Hyprland). Configuração em `~/.config/waybar/config.jsonc`; visual em `~/.config/waybar/style.css`.

```
[workspaces] [título da janela]    [horário]    [rede] [volume] [bateria] [tray]
```

- Clicar em workspace — vai para ele
- Clicar em rede — abre `nm-connection-editor`
- Clicar em volume — abre `pavucontrol`
- Clicar no relógio — alterna formato curto / completo
- Hover na bateria — tooltip com tempo restante e potência
- Bateria ≤15% — pisca em vermelho

Recarregar sem reiniciar o Hyprland:

```sh
pkill waybar && waybar &
```

---

## Aplicativos do sistema

Cada utilitário é um processo independente iniciado pelo Hyprland no boot via `exec-once` e se comunica com o compositor via protocolos Wayland ou sockets.

### wofi

Launcher de aplicativos. Modo padrão `drun` (lista apps instalados); digitar filtra, Enter abre.

- `Super + Space` — abre wofi

### mako

Daemon de notificações. Exibe no canto da tela; clique dispensa a notificação.

- `makoctl dismiss` — dispensa a mais recente
- `makoctl dismiss --all` — dispensa todas

### hyprpaper

Papel de parede. Roda em background, sem interação direta. Config em `~/.config/hypr/hyprpaper.conf`.

- `hyprctl hyprpaper wallpaper ", /caminho/imagem.png"` — troca em tempo real

### hyprlock

Bloqueio de tela. Senha do usuário para desbloquear.

- `Super + L` — bloqueia imediatamente

### hyprpicker

Seletor de cor. Clique em qualquer pixel copia o hex para o clipboard.

- `hyprpicker` — invoca o picker

### grimblast

Captura de tela. Por padrão copia para o clipboard; `save` grava em arquivo.

- `Print` — captura área selecionada
- `Shift + Print` — captura tela inteira
- `grimblast save area ~/Pictures/screenshot.png` — salva em arquivo

---

## hyprctl

Comunicação com o socket do Hyprland em execução. Permite consultar estado e enviar comandos sem reiniciar o compositor. Mudanças via `keyword` são temporárias — somem ao recarregar; para persistir, editar o arquivo de config.

```sh
hyprctl monitors              # lista monitores e configurações
hyprctl workspaces            # lista workspaces ativos
hyprctl clients               # lista janelas abertas (app, classe, workspace)
hyprctl activewindow          # info da janela com foco

hyprctl dispatch killactive   # fecha janela ativa (= Super+Q)
hyprctl dispatch workspace 3  # vai para workspace 3

hyprctl keyword general:gaps_out 10       # muda gap externo (temporário)
hyprctl keyword decoration:rounding 12    # muda arredondamento (temporário)

hyprctl reload                # recarrega hyprland.conf completo
```
