# Hyprland

**Data de criação:** 2026-03-28
**Descrição:** Manual de uso do Hyprland — compositor Wayland com tiling dinâmico, baseado na instalação atual (Fedora 43, NVIDIA, Catppuccin Mocha).

---

## Sumário

- [Conceitos fundamentais](#conceitos-fundamentais)
- [Keybinds essenciais](#keybinds-essenciais)
- [Gerenciamento de janelas](#gerenciamento-de-janelas)
- [Workspaces](#workspaces)
- [Layout Dwindle](#layout-dwindle)
- [Monitores](#monitores)
- [Waybar](#waybar)
- [Aplicativos do sistema](#aplicativos-do-sistema)
- [hyprctl — controle em tempo real](#hyprctl--controle-em-tempo-real)

---

## Conceitos fundamentais

**Para que serve:**
Entender o modelo mental do Hyprland antes de usar — evita confusão com os atalhos e o comportamento das janelas.

**Como funciona:**
O Hyprland é um compositor Wayland com tiling dinâmico. Isso significa que ele gerencia tanto o desenho das janelas na tela (papel do compositor) quanto o posicionamento automático delas em grade (papel do tiling WM). Cada janela aberta é colocada automaticamente em uma divisão no workspace ativo. O `$mod` configurado é a tecla **Super** (tecla Windows/Command).

**Principais conceitos:**
- **Workspace** — área de trabalho virtual; cada workspace existe por monitor
- **Tiling** — janelas dividem o espaço disponível automaticamente
- **Floating** — janela solta, não participa do tiling
- **Focus** — janela ativa no momento; os atalhos atuam sobre ela
- **Dwindle** — algoritmo de divisão em espiral (cada nova janela divide o espaço da anterior ao meio)

---

## Keybinds essenciais

**Para que serve:**
Referência rápida de todos os atalhos configurados para o dia a dia.

**Como funciona:**
Os binds ficam em `~/.config/hypr/keybinds.conf`. A tecla modificadora `$mod` é o **Super**. Combinações com `SHIFT` movem janelas; com `CTRL` redimensionam.

**Principais formas de uso:**

| Atalho | Ação |
|--------|------|
| `Super + Return` | Abre terminal (Alacritty) |
| `Super + Space` | Abre launcher de apps (wofi) |
| `Super + E` | Abre gerenciador de arquivos (Nautilus) |
| `Super + L` | Bloqueia a tela (hyprlock) |
| `Super + Q` | Fecha a janela ativa |
| `Super + F` | Alterna fullscreen |
| `Super + V` | Alterna floating da janela ativa |
| `Super + P` | Alterna pseudotile |
| `Super + H/J/K/L` | Move o foco (esq/baixo/cima/dir) — estilo vim |
| `Super + Shift + H/J/K/L` | Move a janela ativa no layout |
| `Super + Ctrl + ←/→/↑/↓` | Redimensiona a janela ativa (50px por vez) |
| `Super + 1–9` | Vai para o workspace N |
| `Super + Shift + 1–9` | Move a janela ativa para o workspace N |
| `Super + arrastar (botão esq)` | Move janela flutuante |
| `Super + arrastar (botão dir)` | Redimensiona janela flutuante |
| `Print` | Captura área selecionada (copia para clipboard) |
| `Shift + Print` | Captura tela inteira (copia para clipboard) |
| Teclas de mídia | Volume ±5%, mute, brilho ±5% |

---

## Gerenciamento de janelas

**Para que serve:**
Controlar posição, tamanho e modo das janelas no workspace atual.

**Como funciona:**
No modo tiling (padrão), o Hyprland posiciona as janelas automaticamente conforme o algoritmo Dwindle. É possível alternar qualquer janela para floating (`Super+V`), tornando-a independente do grid. Redimensionamento no tiling afeta a divisão entre as janelas vizinhas; no floating afeta só a janela em questão. Também é possível redimensionar arrastando a borda da janela com o mouse (configurado via `resize_on_border = true`).

**Principais formas de uso:**
- `Super + Q` — fecha a janela ativa (envia sinal de encerramento)
- `Super + F` — fullscreen completo (sem borda, sem waybar)
- `Super + V` — toggle floating; a janela sai do tiling e flutua livremente
- `Super + P` — pseudotile: mantém a janela no grid mas a exibe no tamanho preferido dela (útil para apps com tamanho fixo)
- `Super + Ctrl + setas` — redimensiona 50px por tecla pressionada
- Arrastar borda — redimensiona clicando e arrastando a borda (sem atalho)
- `Super + arrastar mouse esq` — move janelas flutuantes
- `Super + arrastar mouse dir` — redimensiona janelas flutuantes

**Janelas com float automático** (configurado em `rules.conf`):
- `pavucontrol` — mixer de áudio
- `nm-connection-editor` — gerenciador de rede
- `blueman-manager` — Bluetooth
- Diálogos "Open File" e "Save As"

---

## Workspaces

**Para que serve:**
Organizar apps em áreas de trabalho separadas para não acumular tudo em uma tela só.

**Como funciona:**
O Hyprland cria workspaces sob demanda — não existem até serem acessados pela primeira vez. Cada workspace está associado a um monitor. Ao mover uma janela para outro workspace (`Super+Shift+N`), ela some da tela atual e aparece no workspace de destino. Workspaces são numerados de 1 a 9.

**Principais formas de uso:**
- `Super + 1–9` — navega diretamente para o workspace N
- `Super + Shift + 1–9` — move a janela ativa para o workspace N (sem seguir ela)
- A waybar exibe os workspaces com número; clicar em um vai direto para ele
- Workspace com janela urgente (notificação) aparece em vermelho na waybar

---

## Layout Dwindle

**Para que serve:**
Entender como o Hyprland posiciona automaticamente as janelas ao abrir novos apps.

**Como funciona:**
O Dwindle funciona como uma árvore binária em espiral. A primeira janela ocupa a tela toda. A segunda divide o espaço da primeira ao meio (vertical ou horizontal). A terceira divide o espaço da segunda, e assim por diante. O resultado é um layout em espiral que vai se subdividindo. Com `preserve_split = true`, a direção da divisão é mantida entre reaberturas.

```
┌───────────────┐    ┌───────┬───────┐    ┌───────┬───┬───┐
│               │    │       │       │    │       │ 2 │ 3 │
│       1       │ →  │   1   │   2   │ →  │   1   ├───┴───┤
│               │    │       │       │    │       │   4   │
└───────────────┘    └───────┴───────┘    └───────┴───────┘
```

**Principais formas de uso:**
- `Super + P` — pseudotile: a janela fica no slot do grid mas não expande para preenchê-lo
- `Super + Shift + H/J/K/L` — reordena as janelas dentro da árvore
- A direção da próxima divisão alterna automaticamente (horizontal ↔ vertical)

---

## Monitores

**Para que serve:**
Gerenciar tela interna do notebook e monitor externo via HDMI.

**Como funciona:**
A configuração base em `monitors.conf` desativa a tela interna (`eDP-1`) e usa o monitor externo como principal. O script `monitor-handler.sh` fica rodando em background ouvindo o socket de eventos do Hyprland via `socat`. Quando um monitor é conectado, desativa o eDP-1; quando é desconectado, reativa o eDP-1 com resolução preferida.

**Principais formas de uso:**
- Conectar HDMI → tela interna desliga automaticamente, externo assume
- Desconectar HDMI → tela interna reativa automaticamente com resolução preferida
- Para forçar manualmente via terminal:
  ```sh
  hyprctl keyword monitor "eDP-1, preferred, auto, 1"   # ativa interna
  hyprctl keyword monitor "eDP-1, disable"               # desativa interna
  ```
- Para listar monitores conectados:
  ```sh
  hyprctl monitors
  ```

---

## Waybar

**Para que serve:**
Barra de status que exibe workspaces, janela ativa, relógio, rede, volume e bateria.

**Como funciona:**
A waybar roda como processo separado, iniciado pelo `exec-once` do Hyprland. Ela se integra nativamente com o protocolo Hyprland para mostrar workspaces e o título da janela ativa. Configuração em `~/.config/waybar/config.jsonc`; visual em `~/.config/waybar/style.css`.

**Layout da barra:**
```
[workspaces] [título da janela]    [horário]    [rede] [volume] [bateria] [tray]
```

**Principais formas de uso:**
- Clicar em número de workspace → vai para aquele workspace
- Clicar no ícone de rede → abre `nm-connection-editor`
- Clicar no ícone de volume → abre `pavucontrol`
- Clicar no relógio → alterna entre formato curto (`14:32`) e completo (`sáb 28 mar 14:32`)
- Hover na bateria → tooltip com tempo restante e potência atual
- Bateria crítica (≤15%) → pisca em vermelho

Para recarregar a waybar sem reiniciar o Hyprland:
```sh
pkill waybar && waybar &
```

---

## Aplicativos do sistema

**Para que serve:**
Referência dos utilitários que fazem parte do ambiente — cada um com seu papel.

**Como funciona:**
Cada ferramenta é um processo independente iniciado pelo Hyprland no boot via `exec-once`. Eles se comunicam com o compositor via protocolos Wayland ou sockets.

**Principais formas de uso:**

**wofi** — launcher de aplicativos
- `Super + Space` → abre em modo `drun` (lista apps instalados)
- Digitar para filtrar, Enter para abrir

**mako** — notificações
- Exibe notificações no canto da tela
- Clicar na notificação a dispensa
- `makoctl dismiss` — dispensa a notificação mais recente
- `makoctl dismiss --all` — dispensa todas

**hyprpaper** — papel de parede
- Roda em background, sem interação direta
- Para trocar o wallpaper em tempo real:
  ```sh
  hyprctl hyprpaper wallpaper ", /caminho/para/imagem.png"
  ```
- Arquivo de config: `~/.config/hypr/hyprpaper.conf`

**hyprlock** — bloqueio de tela
- `Super + L` → bloqueia imediatamente
- Senha do usuário para desbloquear

**hyprpicker** — seletor de cor
- ```sh
  hyprpicker   # clica em qualquer pixel e copia o hex para clipboard
  ```

**grimblast** — screenshots
- `Print` → seleciona área com mouse, copia para clipboard
- `Shift + Print` → captura tela inteira, copia para clipboard
- Para salvar em arquivo em vez de clipboard:
  ```sh
  grimblast save area ~/Pictures/screenshot.png
  ```

---

## hyprctl — controle em tempo real

**Para que serve:**
Inspecionar o estado do Hyprland e modificar configurações sem reiniciar.

**Como funciona:**
O `hyprctl` se comunica com o socket do Hyprland em execução. Permite consultar estado (janelas, workspaces, monitores) e enviar comandos de controle (`dispatch`) ou alterar configurações temporariamente (`keyword`).

**Principais formas de uso:**
```sh
hyprctl monitors              # lista monitores e suas configurações
hyprctl workspaces            # lista workspaces ativos
hyprctl clients               # lista todas as janelas abertas (app, classe, workspace)
hyprctl activewindow          # info da janela com foco

hyprctl dispatch killactive   # fecha janela ativa (mesmo que Super+Q)
hyprctl dispatch workspace 3  # vai para workspace 3

hyprctl keyword general:gaps_out 10       # muda gap externo temporariamente
hyprctl keyword decoration:rounding 12    # muda arredondamento temporariamente

hyprctl reload                # recarrega hyprland.conf completo
```

Mudanças feitas com `keyword` são temporárias — somem ao recarregar ou reiniciar. Para persistir, editar o arquivo de config correspondente.
