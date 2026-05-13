# Compactação, codificação e inspeção binária

Comandos para empacotar e comprimir arquivos (`tar`, `gzip`, `bzip2`, `xz`, `zstd`, `zip`), converter dados entre representações textuais (`base64`) e inspecionar arquivos binários (`xxd`, `strings`). Combinam-se bem em pipelines: empacotar uma árvore com `tar` e comprimir com `zstd`, ou enviar binário por canal de texto codificado em `base64`.

## Sumário

- [Compactação](#compactação)
  - [tar](#tar)
  - [gzip / gunzip](#gzip--gunzip)
  - [bzip2 / bunzip2](#bzip2--bunzip2)
  - [xz / unxz](#xz--unxz)
  - [zstd / unzstd](#zstd--unzstd)
  - [zip / unzip](#zip--unzip)
  - [Comparação](#comparação)
- [Codificação](#codificação)
  - [base64](#base64)
- [Inspeção binária](#inspeção-binária)
  - [xxd](#xxd)
  - [strings](#strings)
- [Pipelines comuns](#pipelines-comuns)

---

## Compactação

### tar

Empacota múltiplos arquivos em um só (tarball) preservando estrutura, permissões e timestamps. Por si só **não comprime** — combina-se com `gzip`, `bzip2`, `xz` ou `zstd` via flags. A sintaxe agrupada (`tar -czvf` etc.) é notoriamente confusa; vale memorizar as combinações.

Mnemônico das letras: **c**reate / e**x**tract / lis**t**, **f**ile (próximo argumento é o nome), **v**erbose, **z** (gzip), **j** (bzip2), **J** (xz), `--zstd` ou `-I zstd` (zstd).

Para identificar o tipo real de um arquivo sem extensão clara, ver [`file`](comandos_shell.md#file).

- `tar -cf arquivo.tar dir/` — cria tarball sem compressão
- `tar -czf arquivo.tar.gz dir/` — cria + comprime com gzip (`.tgz`)
- `tar -cjf arquivo.tar.bz2 dir/` — cria + bzip2
- `tar -cJf arquivo.tar.xz dir/` — cria + xz
- `tar --zstd -cf arquivo.tar.zst dir/` — cria + zstd
- `tar -xf arquivo.tar.gz` — extrai (GNU tar detecta o compressor automaticamente; BSD tar exige flag explícita)
- `tar -xf arquivo.tar.gz -C destino/` — extrai em outro diretório
- `tar -tf arquivo.tar.gz` — lista o conteúdo sem extrair
- `tar -czvf saida.tar.gz dir/ --exclude='*.log'` — exclui padrões
- `tar -czf - dir/ | ssh host 'cat > backup.tar.gz'` — empacota e envia via stream (sem arquivo intermediário)

### gzip / gunzip

Compressor de arquivo único, baseado em DEFLATE. Rápido e onipresente; substitui o arquivo original pela versão `.gz` (use `-k` para preservar). Para múltiplos arquivos, combine com `tar` antes. Para `grep` direto em arquivos comprimidos, ver [`grep`](busca_filtragem.md#grep) e use o wrapper `zgrep`.

- `gzip arquivo` — comprime, gera `arquivo.gz` e remove o original
- `gzip -k arquivo` — mantém o original
- `gzip -9 arquivo` — compressão máxima (mais lenta)
- `gzip -1 arquivo` — compressão mínima (mais rápida)
- `gzip -t arquivo.gz` — testa integridade (silencioso se OK)
- `gunzip arquivo.gz` — descomprime (= `gzip -d`)
- `zcat arquivo.gz` — imprime conteúdo descomprimido sem desempacotar
- `zless arquivo.gz` — pagina conteúdo comprimido
- `zgrep padrão arquivo.gz` — `grep` direto em arquivo comprimido

### bzip2 / bunzip2

Compressor de arquivo único, baseado em Burrows-Wheeler. Comprime mais que `gzip` mas é mais lento. Mesma interface; a extensão é `.bz2`. Wrappers análogos a [`grep`](busca_filtragem.md#grep): `bzgrep`, `bzcat`, `bzless`.

- `bzip2 arquivo` — comprime
- `bzip2 -k arquivo` — preserva original
- `bzip2 -t arquivo.bz2` — testa integridade
- `bunzip2 arquivo.bz2` — descomprime
- `bzcat arquivo.bz2` — imprime descomprimido
- `bzgrep padrão arquivo.bz2` — `grep` em arquivo bzip2

### xz / unxz

Compressor moderno baseado em LZMA2. Razão de compressão ainda melhor que `bzip2`, ao custo de mais CPU e RAM. Padrão de fato para tarballs do kernel Linux e muitos pacotes. Wrappers de leitura: `xzcat`, `xzless`, `xzgrep` (ver [`grep`](busca_filtragem.md#grep)).

- `xz arquivo` — comprime, gera `arquivo.xz`
- `xz -k arquivo` — preserva original
- `xz -T 0 arquivo` — usa todos os cores (paralelo)
- `xz -t arquivo.xz` — testa integridade
- `unxz arquivo.xz` — descomprime
- `xzcat arquivo.xz` — imprime descomprimido
- `xzgrep padrão arquivo.xz` — `grep` em arquivo xz

### zstd / unzstd

Compressor moderno (Facebook), baseado em LZ77 + Huffman. Excelente trade-off: descompressão muito rápida e razão competitiva com `xz` em níveis altos. Vem ganhando espaço em pacotes (Arch, Fedora) e ferramentas de backup. Suporta paralelismo nativo.

- `zstd arquivo` — comprime, gera `arquivo.zst`
- `zstd -k arquivo` — preserva original (default na verdade — diferente do gzip)
- `zstd -19 arquivo` — máxima compressão padrão; `--ultra -22` libera os níveis extremos
- `zstd -T 0 arquivo` — usa todos os cores
- `zstd --long arquivo` — ativa modo de janela longa (melhor razão em arquivos grandes)
- `zstd -t arquivo.zst` — testa integridade
- `unzstd arquivo.zst` — descomprime (= `zstd -d`)
- `zstdcat arquivo.zst` — imprime descomprimido
- `tar --zstd -cf out.tar.zst dir/` — integração com tar (GNU)
- `tar -I zstd -cf out.tar.zst dir/` — forma portável usando `-I` (qualquer compressor externo)

### zip / unzip

Formato cross-platform por excelência — padrão em downloads do Windows, releases do GitHub e pacotes Office. Diferente do `tar`, **junta empacotamento e compressão em um só passo** e cada arquivo é comprimido individualmente (permite extração parcial sem ler tudo).

- `zip arquivo.zip a.txt b.txt` — comprime arquivos
- `zip -r arquivo.zip dir/` — recursivo (necessário para diretórios)
- `zip -9 arquivo.zip dados/` — compressão máxima
- `zip -e arquivo.zip dados/` — pede senha e cria zip criptografado (criptografia fraca, **não use para sigilo real**)
- `unzip arquivo.zip` — extrai no diretório atual
- `unzip arquivo.zip -d destino/` — extrai em outro diretório
- `unzip -l arquivo.zip` — lista o conteúdo sem extrair
- `unzip -P senha arquivo.zip` — extrai zip criptografado (senha em CLI fica no histórico — cuidado)
- `unzip -o arquivo.zip` — sobrescreve sem perguntar

### Comparação

Valores aproximados ao comprimir o mesmo dataset textual (logs, código). Sempre meça no seu caso real — texto, binário e mídia comprimem muito diferentes.

| Ferramenta | Razão típica | Compressão | Descompressão | Notas                              |
|------------|--------------|------------|---------------|------------------------------------|
| `gzip`     | ~3:1         | rápida     | muito rápida  | onipresente, baseline universal    |
| `bzip2`    | ~3.5:1       | lenta      | lenta         | obsoleto na prática frente a xz/zstd |
| `xz`       | ~4:1         | muito lenta| média         | melhor razão clássica; alto uso de RAM |
| `zstd`     | ~3.5:1 (alto: ~4:1) | muito rápida | muito rápida | trade-off ótimo, paralelo nativo |
| `zip`      | ~3:1         | rápida     | rápida        | cross-platform; comprime arquivo a arquivo |

Regra de bolso: **`zstd`** para a maioria dos casos novos; **`gzip`** para máxima compatibilidade; **`xz`** quando o tamanho final pesa mais que o tempo; **`zip`** para interop com Windows.

---

## Codificação

### base64

Codifica dados binários em texto ASCII de 64 caracteres (A–Z, a–z, 0–9, `+`, `/`, com `=` como padding). Usado para enviar binário por canais que só aceitam texto: e-mail, JSON, variáveis de ambiente, headers HTTP. **Não é compressão nem criptografia** — aumenta o tamanho em ~33%.

A variante **URL-safe** troca `+/` por `-_` (e às vezes remove `=`) para ser usada em URLs e tokens JWT sem precisar escapar.

- `base64 arquivo` — codifica e imprime
- `base64 < arquivo > saida.b64` — codifica para arquivo
- `base64 -d arquivo.b64` — decodifica
- `echo 'texto' | base64` — codifica string
- `echo 'dGV4dG8K' | base64 -d` — decodifica string
- `base64 -w 0 arquivo` — sem quebra de linha (uma linha só, útil para JSON/headers)
- `base64 -w 0 arquivo | tr '+/' '-_' | tr -d '='` — gera variante URL-safe (manualmente)
- `echo 'YWJj' | tr '_-' '/+' | base64 -d` — decodifica URL-safe (inverso)

---

## Inspeção binária

### xxd

Gera dump hexadecimal de um arquivo, mostrando offset, bytes em hex e representação ASCII lado a lado. Útil para inspecionar binários, debugar protocolos e diferenciar formatos. Tem modo reverso (`-r`) que reconstrói o binário a partir do dump — permite editar bytes em um editor de texto.

`xxd` vem com o pacote do `vim`, **não é POSIX**. Em sistemas mínimos use `od` como alternativa: `od -A x -t x1z -v arquivo` produz saída comparável (offset em hex, bytes em hex, ASCII).

Para comparar dois binários byte a byte de forma legível, combine com [`diff`](busca_filtragem.md#diff): `diff <(xxd a.bin) <(xxd b.bin)`.

- `xxd arquivo` — dump hex padrão
- `xxd -l 64 arquivo` — limita a 64 bytes (cabeçalho)
- `xxd -s 0x100 arquivo` — começa no offset 0x100
- `xxd -c 8 arquivo` — 8 bytes por linha (em vez de 16)
- `xxd -p arquivo` — saída "plain hex" (sem offset/ASCII)
- `xxd -i arquivo` — emite array C com os bytes do arquivo (útil para embutir em código)
- `xxd -r dump.hex > arquivo.bin` — reconstrói binário a partir do dump
- `echo -n 'AB' | xxd` — vê o hex de uma string
- `od -A x -t x1z -v arquivo` — equivalente POSIX

### strings

Extrai sequências de caracteres imprimíveis de um arquivo binário. Útil para inspeção rápida: descobrir versões embutidas em executáveis, mensagens de erro, paths hardcoded ou pistas em CTFs e malware analysis.

- `strings binario` — extrai sequências ≥ 4 caracteres (default)
- `strings -n 8 binario` — sequências ≥ 8 caracteres
- `strings -a binario` — varre o arquivo inteiro (não só seções de dados)
- `strings -t x binario` — prefixa cada linha com offset em hex
- `strings -e l binario` — codificação UTF-16 little-endian (encontra strings Windows)
- `strings binario | grep -i version` — busca padrão dentro do binário

---

## Pipelines comuns

```sh
# Backup comprimido de um diretório, com data no nome
tar -czf "backup-$(date +%F).tar.gz" projeto/

# Transferir diretório para máquina remota sem arquivo intermediário
tar -czf - projeto/ | ssh host 'tar -xzf - -C /destino'

# Tarball com barra de progresso (requer pv)
tar -cf - dir/ | pv | gzip > out.tar.gz

# Fatiar tarball grande em pedaços de 100 MB
tar -czf - dir/ | split -b 100M - backup.tar.gz.part-
# ... e reconstruir
cat backup.tar.gz.part-* | tar -xzf - -C destino/

# Tarball seletivo via find (só logs modificados nos últimos 7 dias)
find . -name '*.log' -mtime -7 | tar -czf logs.tar.gz -T -

# Smoke test antes de descartar o original
gzip -t arquivo.gz && echo OK

# Codifica chave SSH para colar em um secret manager
base64 -w 0 ~/.ssh/id_ed25519

# Compara dois binários byte a byte de forma legível
diff <(xxd a.bin) <(xxd b.bin)

# Procura URLs embutidas em um executável
strings binario | grep -E 'https?://'

# Tamanho original vs comprimido
ls -l arquivo arquivo.gz
```
