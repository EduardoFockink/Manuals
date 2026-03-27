# Manual de Comandos de Shell para Navegação, Arquivos e Análise de Conteúdo

**Data de criação:** 2026-03-27  
**Descrição:** Documento enxuto de referência para comandos essenciais de shell no Linux, com foco em navegação, manipulação de arquivos, inspeção, leitura, filtragem e composição de comandos sob uma perspectiva operacional e prática.

---

## Sumário

- Navegação e localização
  - `pwd`
  - `cd`
  - `ls`
  - `find`
  - `locate`

- Manipulação de arquivos e permissões
  - `cp`
  - `mv`
  - `touch`
  - `chmod`
  - `chown`

- Consulta e inspeção
  - `man`
  - `file`
  - `stat`

- Leitura e visualização de conteúdo
  - `cat`
  - `less`
  - `head`
  - `tail`
  - `tac`

- Redirecionamento e composição
  - `|`
  - `>`

- Busca e filtragem
  - `grep`
  - `cut`
  - `sort`
  - `uniq`
  - `wc`

---

## Corpo do documento

## Navegação e localização

### pwd

**Para que serve:**  
Mostra o diretório atual da sessão.

**Como funciona:**  
Exibe o caminho absoluto do diretório de trabalho atual do shell.

**Principais parâmetros:**  
- `pwd` — mostra o diretório atual  
- `pwd -P` — mostra o caminho físico real, resolvendo symlinks  
- `pwd -L` — mostra o caminho lógico  

---

### cd

**Para que serve:**  
Altera o diretório atual da sessão.

**Como funciona:**  
Muda o diretório de trabalho do shell para o caminho informado.

**Principais formas de uso:**  
- `cd /caminho` — vai para um diretório específico  
- `cd ..` — sobe um nível  
- `cd ~` — vai para a home  
- `cd -` — volta ao diretório anterior  
- `cd` — retorna para a home do usuário  

---

### ls

**Para que serve:**  
Lista arquivos e diretórios.

**Como funciona:**  
Lê o conteúdo de um diretório e exibe os itens encontrados, podendo mostrar também permissões, donos, tamanhos e datas.

**Principais parâmetros:**  
- `ls` — listagem simples  
- `ls -l` — formato longo  
- `ls -a` — inclui ocultos  
- `ls -la` — formato longo com ocultos  
- `ls -lh` — tamanhos legíveis  
- `ls -R` — listagem recursiva  
- `ls -t` — ordena por data  

---

### find

**Para que serve:**  
Busca arquivos e diretórios com base em critérios como nome, tipo, tamanho e permissões.

**Como funciona:**  
Percorre recursivamente a árvore de diretórios e aplica filtros sobre cada entrada encontrada.

**Principais parâmetros:**  
- `find /caminho -name "arquivo"` — busca por nome  
- `find /caminho -iname "arquivo"` — busca ignorando maiúsculas/minúsculas  
- `find /caminho -type f` — apenas arquivos  
- `find /caminho -type d` — apenas diretórios  
- `find /caminho -perm 644` — busca por permissão  
- `find /caminho -size +10M` — busca por tamanho  
- `find /caminho -user usuario` — busca por dono  
- `find /caminho -exec comando {} \;` — executa ação em cada resultado  

---

### locate

**Para que serve:**  
Busca arquivos rapidamente por nome.

**Como funciona:**  
Consulta uma base indexada do sistema, em vez de percorrer o disco em tempo real.

**Principais parâmetros:**  
- `locate nome` — busca simples  
- `locate -i nome` — ignora maiúsculas/minúsculas  
- `locate -n 20 nome` — limita a quantidade de resultados  
- `locate -r 'regex'` — busca por expressão regular  

**Observação operacional:**  
Os resultados dependem da atualização do banco de dados do sistema.

---

## Manipulação de arquivos e permissões

### cp

**Para que serve:**  
Copia arquivos e diretórios.

**Como funciona:**  
Duplica o conteúdo de um arquivo ou estrutura de diretório para outro caminho.

**Principais parâmetros:**  
- `cp origem destino` — cópia simples  
- `cp -r dir1 dir2` — cópia recursiva de diretórios  
- `cp -a origem destino` — preserva atributos  
- `cp -v origem destino` — modo verboso  
- `cp -u origem destino` — copia apenas se necessário  

---

### mv

**Para que serve:**  
Move ou renomeia arquivos e diretórios.

**Como funciona:**  
Altera a localização ou o nome de um arquivo no sistema de arquivos.

**Principais parâmetros:**  
- `mv origem destino` — move ou renomeia  
- `mv -v origem destino` — modo verboso  
- `mv -i origem destino` — pede confirmação antes de sobrescrever  
- `mv -n origem destino` — não sobrescreve  

---

### touch

**Para que serve:**  
Cria arquivos vazios ou atualiza timestamps de arquivos existentes.

**Como funciona:**  
Se o arquivo não existir, ele é criado; se já existir, seu horário de acesso/modificação é atualizado.

**Principais parâmetros:**  
- `touch arquivo` — cria arquivo vazio ou atualiza timestamp  
- `touch arq1 arq2` — cria ou atualiza múltiplos arquivos  
- `touch -a arquivo` — altera apenas o tempo de acesso  
- `touch -m arquivo` — altera apenas o tempo de modificação  
- `touch -t YYYYMMDDhhmm arquivo` — define timestamp manualmente  

---

### chmod

**Para que serve:**  
Altera permissões de arquivos e diretórios.

**Como funciona:**  
Modifica os bits de permissão de leitura, escrita e execução para dono, grupo e outros.

**Principais parâmetros:**  
- `chmod 644 arquivo` — define permissões numéricas  
- `chmod 755 arquivo` — leitura/execução ampla e escrita para o dono  
- `chmod u+x arquivo` — adiciona execução ao dono  
- `chmod g-w arquivo` — remove escrita do grupo  
- `chmod -R 755 diretorio` — aplica recursivamente  

---

### chown

**Para que serve:**  
Altera o dono e o grupo de arquivos e diretórios.

**Como funciona:**  
Reatribui ownership no sistema de arquivos.

**Principais parâmetros:**  
- `chown usuario arquivo` — altera dono  
- `chown usuario:grupo arquivo` — altera dono e grupo  
- `chown :grupo arquivo` — altera apenas o grupo  
- `chown -R usuario:grupo diretorio` — aplica recursivamente  

---

## Consulta e inspeção

### man

**Para que serve:**  
Exibe o manual de comandos, bibliotecas e arquivos de configuração.

**Como funciona:**  
Abre a página de manual instalada localmente no sistema.

**Principais parâmetros:**  
- `man comando` — abre o manual  
- `man 5 arquivo` — abre uma seção específica  
- `man -k termo` — busca por palavra-chave  
- `man -f comando` — descrição curta  
- `man -a comando` — mostra todas as entradas correspondentes  

---

### file

**Para que serve:**  
Identifica o tipo real de um arquivo.

**Como funciona:**  
Analisa assinatura, formato e conteúdo do arquivo, sem depender só da extensão.

**Principais parâmetros:**  
- `file arquivo` — identifica o tipo  
- `file *` — verifica múltiplos arquivos  
- `file -i arquivo` — mostra tipo MIME  
- `file -b arquivo` — saída resumida sem o nome do arquivo  

---

### stat

**Para que serve:**  
Mostra metadados detalhados de um arquivo ou diretório.

**Como funciona:**  
Lê informações do inode e exibe tamanho, permissões, dono, grupo e timestamps.

**Principais parâmetros:**  
- `stat arquivo` — mostra metadados completos  
- `stat -c '%a %U %G %n' arquivo` — saída formatada  
- `stat -f arquivo` — mostra dados do sistema de arquivos  

---

## Leitura e visualização de conteúdo

### cat

**Para que serve:**  
Exibe e concatena arquivos.

**Como funciona:**  
Lê um ou mais arquivos e escreve seu conteúdo na saída padrão.

**Principais parâmetros:**  
- `cat arquivo` — exibe o conteúdo  
- `cat arq1 arq2` — concatena arquivos  
- `cat -n arquivo` — numera todas as linhas  
- `cat -b arquivo` — numera apenas linhas não vazias  
- `cat -s arquivo` — comprime linhas vazias repetidas  

---

### less

**Para que serve:**  
Permite visualizar arquivos longos de forma paginada.

**Como funciona:**  
Abre o arquivo em modo de leitura interativa, com navegação por páginas e busca interna.

**Principais formas de uso:**  
- `less arquivo` — abre o arquivo  
- `/termo` — busca dentro do conteúdo  
- `n` — próximo resultado da busca  
- `q` — sai do visualizador  
- `less -N arquivo` — mostra número das linhas  
- `less -S arquivo` — evita quebra de linha longa  

---

### head

**Para que serve:**  
Mostra o início de um arquivo.

**Como funciona:**  
Lê as primeiras linhas da entrada ou do arquivo informado.

**Principais parâmetros:**  
- `head arquivo` — mostra as 10 primeiras linhas  
- `head -n 20 arquivo` — mostra as 20 primeiras linhas  
- `head -c 100 arquivo` — mostra os primeiros 100 bytes  

---

### tail

**Para que serve:**  
Mostra o fim de um arquivo, muito útil para logs.

**Como funciona:**  
Lê as últimas linhas ou bytes do arquivo informado.

**Principais parâmetros:**  
- `tail arquivo` — mostra as 10 últimas linhas  
- `tail -n 20 arquivo` — mostra as 20 últimas linhas  
- `tail -f arquivo` — acompanha o arquivo em tempo real  
- `tail -c 100 arquivo` — mostra os últimos 100 bytes  

---

### tac

**Para que serve:**  
Exibe um arquivo em ordem inversa de linhas.

**Como funciona:**  
Lê o conteúdo e imprime as linhas de baixo para cima.

**Principais parâmetros:**  
- `tac arquivo` — inverte a ordem das linhas  
- `tac -s 'sep' arquivo` — usa separador customizado  
- `tac -b -s 'sep' arquivo` — associa o separador ao início do bloco  

---

## Redirecionamento e composição

### |

**Para que serve:**  
Encadeia comandos, enviando a saída de um para a entrada do outro.

**Como funciona:**  
Conecta o `stdout` do comando à esquerda ao `stdin` do comando à direita.

**Principais formas de uso:**  
- `cat arquivo | grep termo` — filtra conteúdo  
- `ls -la | grep nome` — procura na listagem  
- `ps aux | grep processo` — busca processos  
- `comando1 | comando2 | comando3` — pipeline em múltiplas etapas  

---

### >

**Para que serve:**  
Redireciona a saída de um comando para um arquivo, sobrescrevendo o conteúdo.

**Como funciona:**  
Envia o `stdout` do comando para um arquivo, em vez de exibir no terminal.

**Principais formas de uso:**  
- `comando > arquivo.txt` — grava a saída  
- `ls -la > lista.txt` — salva listagem  
- `echo texto > arquivo.txt` — escreve em arquivo  

**Observação operacional:**  
Para anexar ao final sem sobrescrever, usa-se `>>`.

---

## Busca e filtragem

### grep

**Para que serve:**  
Busca padrões em arquivos ou fluxos de texto.

**Como funciona:**  
Lê a entrada linha por linha e imprime as que correspondem ao padrão especificado.

**Principais parâmetros:**  
- `grep termo arquivo` — busca simples  
- `grep -i termo arquivo` — ignora maiúsculas/minúsculas  
- `grep -r termo diretorio` — busca recursiva  
- `grep -n termo arquivo` — mostra número da linha  
- `grep -v termo arquivo` — inverte o filtro  
- `grep -E 'regex' arquivo` — regex estendida  
- `grep -o 'regex' arquivo` — mostra apenas o trecho casado  
- `grep -A N termo arquivo` — N linhas após  
- `grep -B N termo arquivo` — N linhas antes  
- `grep -C N termo arquivo` — contexto antes e depois  

---

### cut

**Para que serve:**  
Extrai campos, colunas ou faixas de caracteres de cada linha.

**Como funciona:**  
Seleciona partes do texto com base em delimitador ou posição.

**Principais parâmetros:**  
- `cut -d ':' -f 1 arquivo` — extrai o campo 1 usando `:`  
- `cut -d ' ' -f 2-4 arquivo` — extrai intervalo de campos  
- `cut -c 1-10 arquivo` — extrai por posição de caracteres  
- `cut -f 3 arquivo` — extrai campo com TAB como delimitador padrão  

---

### sort

**Para que serve:**  
Ordena linhas de texto.

**Como funciona:**  
Lê as linhas da entrada e as organiza conforme critério alfabético, numérico ou customizado.

**Principais parâmetros:**  
- `sort arquivo` — ordenação padrão  
- `sort -n arquivo` — ordenação numérica  
- `sort -r arquivo` — ordem reversa  
- `sort -u arquivo` — ordena e remove duplicatas  
- `sort -k 2 arquivo` — ordena pela coluna 2  
- `sort -t ':' -k 1,1 arquivo` — define delimitador e coluna  
- `sort -h arquivo` — ordena valores legíveis como `1K`, `1M`  

---

### uniq

**Para que serve:**  
Remove ou identifica linhas repetidas consecutivas.

**Como funciona:**  
Compara linhas adjacentes e consolida repetições; por isso normalmente é usado após `sort`.

**Principais parâmetros:**  
- `uniq arquivo` — remove duplicatas consecutivas  
- `uniq -c arquivo` — conta ocorrências  
- `uniq -d arquivo` — mostra apenas duplicadas  
- `uniq -u arquivo` — mostra apenas únicas  
- `sort arquivo | uniq` — fluxo comum para deduplicação geral  

---

### wc

**Para que serve:**  
Conta linhas, palavras, caracteres e bytes.

**Como funciona:**  
Lê a entrada ou arquivo e retorna métricas básicas de volume de texto.

**Principais parâmetros:**  
- `wc arquivo` — mostra linhas, palavras e bytes  
- `wc -l arquivo` — conta linhas  
- `wc -w arquivo` — conta palavras  
- `wc -c arquivo` — conta bytes  
- `wc -m arquivo` — conta caracteres  

---