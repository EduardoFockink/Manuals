# Comandos de shell

Referência de comandos essenciais de shell no Linux para navegação, manipulação de arquivos, inspeção, leitura e composição de pipelines. Para `grep`, `cut`, `sort`, `uniq`, `wc` e demais ferramentas de análise de texto, ver [`busca_filtragem.md`](busca_filtragem.md).

## Sumário

- [Navegação e localização](#navegação-e-localização)
- [Manipulação de arquivos e permissões](#manipulação-de-arquivos-e-permissões)
- [Consulta e inspeção](#consulta-e-inspeção)
- [Leitura e visualização de conteúdo](#leitura-e-visualização-de-conteúdo)
- [Redirecionamento e composição](#redirecionamento-e-composição)

---

## Navegação e localização

### pwd

Mostra o caminho absoluto do diretório de trabalho atual. Por padrão, exibe o caminho lógico (com symlinks); `-P` resolve para o caminho físico real.

- `pwd` — diretório atual
- `pwd -P` — resolve symlinks
- `pwd -L` — caminho lógico (padrão)

### cd

Altera o diretório de trabalho do shell. Sem argumento, volta para a home do usuário; o argumento `-` retorna ao diretório imediatamente anterior, útil para alternar rápido entre dois locais.

- `cd /caminho` — vai para um diretório
- `cd ..` — sobe um nível
- `cd ~` ou `cd` — vai para a home
- `cd -` — volta ao diretório anterior

### ls

Lista arquivos e diretórios. Sem flags, omite ocultos e mostra só nomes; `-l` traz formato longo com permissões, dono, tamanho e data. Combinações úteis viraram alias por hábito (`ll`, `la`).

- `ls` — listagem simples
- `ls -l` — formato longo
- `ls -a` — inclui ocultos
- `ls -la` — longo com ocultos
- `ls -lh` — tamanhos legíveis (`1.2K`, `5.4M`)
- `ls -lt` — ordena por data, mais recente primeiro
- `ls -R` — recursivo

### find

Percorre recursivamente uma árvore de diretórios aplicando filtros (nome, tipo, tamanho, permissão, idade). Diferente de `locate`, lê o disco em tempo real — sempre atualizado, mas mais lento em árvores grandes.

- `find /caminho -name "padrão"` — busca por nome
- `find /caminho -iname "padrão"` — ignora maiúsculas/minúsculas
- `find /caminho -type f` — só arquivos
- `find /caminho -type d` — só diretórios
- `find /caminho -size +10M` — maiores que 10 MB
- `find /caminho -mtime -7` — modificados nos últimos 7 dias
- `find /caminho -perm 644` — permissão exata
- `find /caminho -user nome` — por dono
- `find /caminho -exec comando {} \;` — executa ação em cada resultado
- `find /caminho -name "*.tmp" -delete` — remove resultados

### locate

Busca arquivos consultando uma base indexada (`updatedb`), em vez de varrer o disco. Muito mais rápido que `find`, mas só encontra o que estava no índice na última atualização.

- `locate nome` — busca simples
- `locate -i nome` — ignora maiúsculas/minúsculas
- `locate -n 20 nome` — limita a 20 resultados
- `locate -r 'regex'` — busca por regex
- `sudo updatedb` — força atualização do índice

---

## Manipulação de arquivos e permissões

### mkdir

Cria diretórios. Sem flags, falha se algum diretório intermediário não existir; `-p` cria toda a cadeia de uma vez e não reclama se o destino já existir.

- `mkdir nome` — cria diretório
- `mkdir -p a/b/c` — cria árvore inteira (idempotente)
- `mkdir -m 755 nome` — define permissões na criação
- `mkdir -v nome` — modo verboso

### cp

Copia arquivos e diretórios. Para diretórios, exige `-r` (recursivo). `-a` é o atalho para "preserve tudo" — atributos, links, timestamps —, ideal para backups fiéis.

- `cp origem destino` — cópia simples
- `cp -r dir1 dir2` — recursivo
- `cp -a origem destino` — preserva atributos, links, timestamps
- `cp -u origem destino` — só copia se origem for mais nova
- `cp -i origem destino` — pede confirmação antes de sobrescrever
- `cp -v origem destino` — verboso

### mv

Move ou renomeia. Renomear é apenas mover dentro do mesmo diretório. Em filesystems diferentes, vira cópia + remoção.

- `mv origem destino` — move ou renomeia
- `mv -i origem destino` — pede confirmação ao sobrescrever
- `mv -n origem destino` — nunca sobrescreve
- `mv -v origem destino` — verboso

### rm

Remove arquivos. **Não há lixeira** — a remoção é definitiva e imediata. Sem `-r`, recusa diretórios; sem `-f`, pede confirmação para arquivos sem permissão de escrita. O temido `rm -rf /` é bloqueado por padrão pelo GNU coreutils via `--preserve-root`.

- `rm arquivo` — remove arquivo
- `rm -i arquivo` — pede confirmação por arquivo
- `rm -I dir/*` — pede uma única confirmação antes de remover ≥4 itens
- `rm -r dir` — recursivo (remove diretório e conteúdo)
- `rm -f arquivo` — força, sem prompt e sem erro se não existir
- `rm -rf dir` — combinação clássica para remoção sem perguntas (cuidado)
- `rm -v arquivo` — verboso

### rmdir

Remove **somente** diretórios vazios — falha se houver qualquer conteúdo. Útil em scripts onde a presença de arquivos remanescentes deve abortar a operação (diferente de `rm -r`, que destrói tudo silenciosamente).

- `rmdir dir` — remove diretório vazio
- `rmdir -p a/b/c` — remove a cadeia inteira se cada nível ficar vazio
- `rmdir --ignore-fail-on-non-empty dir` — não falha se houver conteúdo

### touch

Cria arquivos vazios ou atualiza timestamps. Útil em scripts (criar marcadores), em `make` (forçar rebuild) e para reservar nomes.

- `touch arquivo` — cria vazio ou atualiza atime/mtime
- `touch arq1 arq2` — múltiplos
- `touch -a arquivo` — só access time
- `touch -m arquivo` — só modification time
- `touch -t YYYYMMDDhhmm arquivo` — define timestamp manualmente

### chmod

Altera bits de permissão (leitura/escrita/execução para dono/grupo/outros). Aceita notação numérica (octal: `755`) ou simbólica (`u+x`, `g-w`, `a=r`).

- `chmod 644 arquivo` — `rw-r--r--`
- `chmod 755 script.sh` — executável (padrão para scripts)
- `chmod u+x arquivo` — adiciona execução ao dono
- `chmod g-w arquivo` — remove escrita do grupo
- `chmod a+r arquivo` — leitura para todos
- `chmod -R 755 dir` — recursivo

### chown

Reatribui dono e/ou grupo. Geralmente requer `sudo`. A sintaxe `usuario:grupo` define ambos; `:grupo` muda só o grupo.

- `chown usuario arquivo` — só dono
- `chown usuario:grupo arquivo` — dono e grupo
- `chown :grupo arquivo` — só grupo
- `chown -R usuario:grupo dir` — recursivo

### ln

Cria links entre arquivos. **Hard link** (default) é uma segunda entrada de diretório apontando para o mesmo inode — o arquivo só some quando todos os hard links são removidos; não atravessa filesystems nem funciona com diretórios. **Symlink** (`-s`) é um pequeno arquivo que guarda o caminho de outro — pode apontar para qualquer lugar, mas quebra se o alvo move.

- `ln alvo nome` — hard link
- `ln -s alvo nome` — symlink (uso mais comum)
- `ln -sf alvo nome` — substitui symlink existente sem reclamar
- `ln -sn alvo dir/nome` — trata `dir/nome` como arquivo, não segue symlink
- `ln -sr alvo nome` — gera path relativo no symlink (mais portável)
- `ls -l nome` — mostra para onde o symlink aponta (`->`)
- `readlink -f nome` — resolve a cadeia até o destino final

---

## Consulta e inspeção

### man

Abre páginas de manual instaladas localmente. Cada manual pertence a uma seção numerada (1: comandos, 5: arquivos de config, 8: admin). `-k` busca por palavra-chave em todas as descrições curtas.

- `man comando` — abre o manual
- `man 5 arquivo` — seção específica (ex: `man 5 crontab`)
- `man -k termo` — busca por palavra-chave (`apropos`)
- `man -f comando` — descrição curta (`whatis`)
- `man -a comando` — itera por todas as seções com correspondência

### which

Mostra o caminho completo do executável que seria invocado ao digitar um comando, percorrendo o `$PATH` na ordem. Útil para descobrir qual versão de um binário está sendo usada quando há múltiplas instalações (ex: `python` do sistema vs. `pyenv`).

- `which comando` — mostra o caminho do primeiro executável encontrado
- `which -a comando` — mostra **todos** os executáveis com esse nome no `$PATH`
- `command -v comando` — alternativa portável (builtin do shell), também resolve aliases e funções
- `type comando` — mais completo: indica se é alias, função, builtin ou arquivo

### file

Identifica o tipo real de um arquivo analisando assinatura e conteúdo, sem confiar na extensão. Diferencia binário de texto, formato de imagem, codificação, etc.

- `file arquivo` — identifica o tipo
- `file *` — múltiplos
- `file -i arquivo` — tipo MIME
- `file -b arquivo` — saída sem o nome do arquivo (útil em scripts)

### stat

Exibe metadados do inode: tamanho, permissões, dono, grupo, e os três timestamps (access, modify, change). Mais detalhado que `ls -l`.

- `stat arquivo` — metadados completos
- `stat -c '%a %U %G %n' arquivo` — saída formatada (octal, dono, grupo, nome)
- `stat -f arquivo` — informações do filesystem em vez do arquivo

### du

Mede o uso de disco de arquivos e diretórios, somando recursivamente. Difere de `ls -l`: `du` conta blocos efetivamente alocados (inclui sub-árvores); `ls` mostra só o tamanho do próprio arquivo.

- `du -sh dir` — total do diretório, em formato legível (uso mais comum)
- `du -h dir` — todos os subdiretórios, recursivo
- `du -ah dir` — inclui arquivos individuais
- `du -sh *` — tamanho de cada item do diretório atual
- `du -sh * | sort -h` — ordenado do menor para o maior
- `du --max-depth=1 -h dir` — limita a profundidade da recursão
- `du -x dir` — não cruza pontos de montagem

---

## Leitura e visualização de conteúdo

### cat

Concatena e imprime arquivos no stdout. Para arquivos grandes prefira `less`; `cat` é mais útil em pipelines (`cat arquivo | comando`) ou para juntar vários arquivos.

- `cat arquivo` — exibe conteúdo
- `cat arq1 arq2` — concatena
- `cat -n arquivo` — numera todas as linhas
- `cat -b arquivo` — numera só linhas não vazias
- `cat -s arquivo` — colapsa linhas vazias consecutivas

### more

Paginador clássico do Unix. Avança página a página com `Espaço` e linha a linha com `Enter`, mas não rola para trás (em implementações modernas, `b` volta uma página, mas só funciona em arquivos, não em pipes). Sobreviveu por estar presente em qualquer sistema; para uso diário, prefira `less`.

- `more arquivo` — abre o arquivo
- `more +100 arquivo` — começa na linha 100
- `more -d arquivo` — exibe instruções de navegação no rodapé
- `comando | more` — pagina a saída de outro comando

Dentro do `more`:
- `Espaço` — próxima página
- `Enter` — próxima linha
- `/termo` — busca para frente
- `q` — sai

### less

Sucessor do `more` ("less is more"). Visualizador paginado interativo que permite rolar para trás, buscar bidirecionalmente e abrir arquivos enormes sem carregar tudo na memória. Atalhos seguem convenções do `vi`.

- `less arquivo` — abre o arquivo
- `less -N arquivo` — mostra números de linha
- `less -S arquivo` — não quebra linhas longas (rola horizontalmente)
- `less +F arquivo` — modo "follow", igual a `tail -f` (sai com `Ctrl+C`)

Dentro do `less`:
- `/termo` — busca para frente; `?termo` para trás
- `n` / `N` — próximo / anterior resultado
- `g` / `G` — início / fim do arquivo
- `q` — sai

### head

Imprime o início do arquivo (10 linhas por padrão). Útil para inspecionar cabeçalhos de CSVs, logs ou saídas longas.

- `head arquivo` — primeiras 10 linhas
- `head -n 20 arquivo` — primeiras 20 linhas
- `head -n -5 arquivo` — todas exceto as 5 últimas
- `head -c 100 arquivo` — primeiros 100 bytes

### tail

Imprime o fim do arquivo. `-f` acompanha em tempo real (clássico para logs); `-F` lida com rotação de arquivo.

- `tail arquivo` — últimas 10 linhas
- `tail -n 20 arquivo` — últimas 20 linhas
- `tail -n +5 arquivo` — da linha 5 em diante
- `tail -f arquivo` — segue o arquivo em tempo real
- `tail -F arquivo` — segue mesmo com rotação

### tac

`cat` ao contrário: imprime as linhas de baixo para cima. Útil para ler logs cronologicamente reversos sem carregar no editor.

- `tac arquivo` — inverte a ordem das linhas
- `tac -s 'sep' arquivo` — usa separador customizado em vez de quebra de linha

---

## Redirecionamento e composição

### Pipe (`|`)

Conecta o `stdout` de um comando ao `stdin` do próximo. Cada estágio roda em paralelo: o segundo já consome enquanto o primeiro ainda produz, sem precisar de arquivo intermediário.

- `cat arquivo | grep termo` — filtra conteúdo
- `ls -la | grep nome` — busca em listagem
- `ps aux | grep processo` — busca processos
- `comando1 | comando2 | comando3` — pipeline encadeado

### Redirecionamento (`>`, `>>`, `<`, `2>`)

Direciona streams entre comandos e arquivos. `>` sobrescreve; `>>` anexa; `<` injeta arquivo como stdin; `2>` redireciona stderr separadamente.

- `comando > arquivo` — grava stdout (sobrescreve)
- `comando >> arquivo` — anexa stdout ao final
- `comando < arquivo` — usa arquivo como stdin
- `comando 2> erros.log` — redireciona só stderr
- `comando > saida.log 2>&1` — junta stderr ao stdout
- `comando &> tudo.log` — atalho do bash/zsh para o caso acima
- `comando > /dev/null 2>&1` — descarta toda a saída
