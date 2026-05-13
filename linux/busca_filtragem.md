# Busca e filtragem em shell

Comandos para localizar padrões em arquivos e fluxos de texto, extrair campos, ordenar, deduplicar e contar. São as peças mais combinadas em pipelines do shell — `grep | cut | sort | uniq -c | sort -rn` é o canivete suíço da análise rápida de logs e dados tabulares.

Complementa [`comandos_shell.md`](comandos_shell.md), que cobre navegação, manipulação de arquivos e leitura de conteúdo.

## Sumário

- [grep](#grep)
- [cut](#cut)
- [tr](#tr)
- [sed](#sed)
- [awk](#awk)
- [sort](#sort)
- [uniq](#uniq)
- [wc](#wc)
- [diff](#diff)
- [Pipelines comuns](#pipelines-comuns)

---

## grep

Busca padrões em arquivos ou na entrada padrão. Lê linha por linha e imprime as que casam com o padrão. Sem flags, trata o padrão como texto literal; com `-E`, como regex estendida.

- `grep termo arquivo` — busca literal
- `grep -i termo arquivo` — ignora maiúsculas/minúsculas
- `grep -v termo arquivo` — inverte (mostra o que **não** casa)
- `grep -n termo arquivo` — prefixa com número da linha
- `grep -r termo dir/` — recursivo na árvore de diretórios
- `grep -l termo *` — lista só os arquivos que contêm o termo
- `grep -c termo arquivo` — conta ocorrências
- `grep -E 'regex' arquivo` — regex estendida (`+`, `?`, `|` sem escape)
- `grep -o 'regex' arquivo` — imprime apenas o trecho casado, não a linha inteira
- `grep -A N` / `-B N` / `-C N` — N linhas de contexto após / antes / ambos

---

## cut

Extrai pedaços fixos de cada linha — campos delimitados ou faixas de caracteres. Útil quando o formato é regular (CSV, `/etc/passwd`, saída de `ls -l`). Para parsing mais complexo, prefira `awk`.

- `cut -d ':' -f 1 arquivo` — campo 1, delimitado por `:`
- `cut -d ',' -f 2-4 arquivo` — intervalo de campos
- `cut -d ',' -f 1,3,5 arquivo` — campos não contíguos
- `cut -c 1-10 arquivo` — caracteres da posição 1 a 10
- `cut -f 3 arquivo` — TAB é o delimitador padrão (sem `-d`)

---

## tr

Substitui, remove ou colapsa caracteres em um stream. Opera caractere a caractere — não entende campos nem linhas. Importante: **só lê stdin**, não aceita arquivo como argumento; use `<` ou pipe.

- `tr 'a' 'A' < arquivo` — substitui `a` por `A`
- `tr 'a-z' 'A-Z' < arquivo` — converte para maiúsculas
- `tr '[:lower:]' '[:upper:]' < arquivo` — equivalente com classes POSIX
- `tr -d '[:digit:]' < arquivo` — remove todos os dígitos
- `tr -s ' ' < arquivo` — colapsa espaços consecutivos em um só
- `tr -s '[:space:]' '\n' < arquivo` — substitui qualquer whitespace por quebra de linha
- `tr -c '[:alnum:]' ' ' < arquivo` — `-c` complementa: troca tudo que **não** for alfanumérico por espaço
- `tr -d '\r' < arquivo` — remove CR (útil para arquivos de Windows)

Classes úteis: `[:alpha:]`, `[:digit:]`, `[:alnum:]`, `[:space:]`, `[:upper:]`, `[:lower:]`, `[:punct:]`.

---

## sed

Stream editor — aplica edições programáticas linha a linha. O caso de longe mais comum é substituição (`s/x/y/`); o resto da linguagem (delete, append, hold space) raramente aparece no dia a dia. **Atenção:** sintaxe difere entre GNU sed (Linux) e BSD sed (macOS), especialmente em `-i`.

- `sed 's/x/y/' arquivo` — substitui a **primeira** ocorrência de `x` por `y` em cada linha
- `sed 's/x/y/g' arquivo` — global (todas as ocorrências da linha)
- `sed -E 's/(foo|bar)/baz/g' arquivo` — regex estendida
- `sed -i 's/x/y/g' arquivo` — edita o arquivo no lugar (GNU)
- `sed -i.bak 's/x/y/g' arquivo` — in-place com backup `.bak` (portável Linux/macOS)
- `sed -n '5,10p' arquivo` — imprime apenas as linhas 5 a 10 (`-n` silencia, `p` imprime)
- `sed '/regex/d' arquivo` — deleta linhas que casam com o regex
- `sed '1d' arquivo` — remove a primeira linha (cabeçalho de CSV)
- `sed '$d' arquivo` — remove a última linha
- `sed 's|/path/old|/path/new|g'` — `|` como delimitador (evita escapar barras)

---

## awk

Linguagem de processamento de texto orientada a campos. Cada linha é dividida em campos (`$1`, `$2`, ..., `$NF` = último); blocos `BEGIN { }` e `END { }` rodam antes e depois do processamento. Para extração simples, `cut` basta; para somar, filtrar por condição ou reformatar, `awk` brilha.

- `awk '{print $1}' arquivo` — primeira coluna
- `awk '{print $NF}' arquivo` — última coluna
- `awk -F ':' '{print $1}' /etc/passwd` — define delimitador
- `awk 'NR==5' arquivo` — imprime apenas a linha 5 (`NR` = número da linha)
- `awk 'NR>1' arquivo` — pula a primeira linha (cabeçalho)
- `awk '/regex/ {print $2}' arquivo` — só linhas que casam com o regex
- `awk '$3 > 100 {print}' arquivo` — filtro por valor numérico
- `awk '{s+=$1} END {print s}' arquivo` — soma a coluna 1
- `awk '{s+=$1} END {print s/NR}' arquivo` — média
- `awk 'BEGIN {FS=","; OFS="\t"} {print $1,$3}' arquivo.csv` — converte CSV para TSV (campos 1 e 3)
- `awk '!seen[$0]++' arquivo` — remove duplicatas preservando ordem original

---

## sort

Ordena linhas. Sem flags, é alfabético por linha inteira. Lê o arquivo todo na memória antes de imprimir, então a ordem da saída só aparece quando a entrada termina (importante em pipes).

- `sort arquivo` — alfabético
- `sort -n arquivo` — numérico (10 vem depois de 9, não antes)
- `sort -h arquivo` — numérico legível (`1K`, `2M`, `3G`)
- `sort -r arquivo` — ordem reversa
- `sort -u arquivo` — remove duplicatas
- `sort -k 2 arquivo` — ordena pela coluna 2
- `sort -t ':' -k 1,1 arquivo` — define delimitador e restringe a coluna 1

---

## uniq

Colapsa ou identifica linhas repetidas **consecutivas**. Por isso quase sempre vem depois de `sort` — só assim duplicatas espalhadas pelo arquivo ficam adjacentes.

- `uniq arquivo` — remove duplicatas consecutivas
- `uniq -c arquivo` — prefixa cada linha com a contagem de ocorrências
- `uniq -d arquivo` — mostra **apenas** as duplicadas
- `uniq -u arquivo` — mostra **apenas** as únicas (que não se repetem)
- `uniq -i arquivo` — ignora maiúsculas/minúsculas na comparação

---

## wc

Conta linhas, palavras, caracteres e bytes. Sem flags, imprime os três primeiros. Aceita entrada via pipe ou arquivo direto.

- `wc arquivo` — linhas, palavras e bytes
- `wc -l arquivo` — só linhas
- `wc -w arquivo` — só palavras
- `wc -c arquivo` — bytes
- `wc -m arquivo` — caracteres (difere de bytes em UTF-8)

---

## diff

Compara dois arquivos linha a linha e mostra o que difere. O formato `unified` (`-u`) é o mesmo dos patches e dos `git diff`. Para diretórios, `-r` percorre recursivamente. Em código, `git diff`, `delta` ou `vimdiff` costumam render melhor que o `diff` cru.

- `diff a.txt b.txt` — diff no formato clássico
- `diff -u a.txt b.txt` — formato unified (padrão de patches)
- `diff -q a.txt b.txt` — só diz **se** diferem, sem mostrar o quê
- `diff -r dir1/ dir2/` — recursivo entre diretórios
- `diff -i a.txt b.txt` — ignora maiúsculas/minúsculas
- `diff -w a.txt b.txt` — ignora diferenças de whitespace
- `diff -y a.txt b.txt` — saída lado a lado
- `diff --color=always a.txt b.txt` — colorido
- `diff -u a.txt b.txt > patch.diff` — gera patch; aplica com `patch < patch.diff`

---

## Pipelines comuns

Combinações frequentes que vale memorizar:

```sh
# Top 10 IPs mais frequentes em um log
cut -d ' ' -f 1 access.log | sort | uniq -c | sort -rn | head -10

# Linhas únicas preservando a ordem original
awk '!seen[$0]++' arquivo

# Contar quantos arquivos .py existem no projeto
find . -name '*.py' | wc -l

# Buscar termo em arquivos versionados, ignorando node_modules
grep -rn termo --exclude-dir=node_modules .

# Extrair domínios únicos de uma lista de URLs
grep -oE 'https?://[^/]+' urls.txt | sort -u

# Frequência de palavras em um texto livre
tr -s '[:space:]' '\n' < texto.md | tr '[:upper:]' '[:lower:]' \
  | sort | uniq -c | sort -rn | head -20

# Tamanho total dos arquivos do diretório atual (em bytes)
ls -l | awk 'NR>1 {s+=$5} END {print s}'

# Substitui 'foo' por 'bar' em todos os .py do projeto
find . -name '*.py' -exec sed -i 's/foo/bar/g' {} +
```
