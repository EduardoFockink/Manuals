# Scan Tools

**Data de criação:** 2026-03-23  
**Descrição:** Documento enxuto de referência para comandos comuns de reconhecimento em offensive security, com foco em finalidade, funcionamento e parâmetros principais de uso.

---

## Sumário

- whois
- nslookup
- host
- traceroute
- whatweb
- dnsmap
- dirb

---

## Corpo do documento

## whois

**Para que serve:**  
Consulta informações de registro de domínios, blocos IP e ASNs, como registrador, datas, contatos e servidores responsáveis.

**Como funciona:**  
O cliente consulta um servidor WHOIS e, quando necessário, segue redirecionamentos até o servidor autoritativo que possui os dados do recurso consultado.

**Principais parâmetros:**  
- `whois alvo` — consulta básica de domínio ou IP  
- `-h HOST` — força um servidor WHOIS específico  
- `-I` — consulta primeiro a IANA  
- `--no-recursion` — não segue redirecionamentos  
- `-i ATTR` — busca por atributo  
- `-T TYPE` — filtra por tipo de objeto  

---

## nslookup

**Para que serve:**  
Realiza consultas DNS para validar resolução de nomes, identificar registros e testar servidores DNS específicos.

**Como funciona:**  
Envia queries DNS diretamente ao servidor configurado ou a um nameserver indicado manualmente, permitindo consulta interativa ou pontual.

**Principais parâmetros:**  
- `nslookup alvo` — consulta básica  
- `nslookup alvo servidor` — consulta usando um DNS específico  
- `-query=TYPE` — define o tipo de registro (`A`, `AAAA`, `MX`, `NS`, `TXT`, `PTR`)  
- `set norecurse` — desativa recursão  
- `set timeout=N` — ajusta timeout  
- `set retry=N` — ajusta tentativas  

---

## host

**Para que serve:**  
Faz consultas DNS rápidas e objetivas, sendo útil para enumeração simples e automação em shell.

**Como funciona:**  
Resolve nomes para IPs, IPs para nomes e permite consultar registros específicos diretamente em um nameserver escolhido.

**Principais parâmetros:**  
- `host alvo` — resolução básica  
- `host alvo servidor` — consulta em um nameserver específico  
- `-t TYPE` — escolhe o tipo de registro  
- `-a` — saída detalhada  
- `-l dominio` — tenta zone transfer  
- `-r` — desativa recursão  
- `-W N` — define timeout  

---

## traceroute

**Para que serve:**  
Mapeia o caminho de rede até um host, ajudando a identificar saltos, filtragens, latência e pontos de falha.

**Como funciona:**  
Envia pacotes com TTL crescente; cada roteador no caminho responde quando o TTL expira, revelando os hops intermediários.

**Principais parâmetros:**  
- `traceroute alvo` — execução padrão  
- `-I` — usa ICMP  
- `-T` — usa TCP SYN  
- `-p PORTA` — define a porta de destino  
- `-f N` — define o TTL inicial  
- `-m N` — define o número máximo de hops  
- `-n` — não resolve nomes  
- `-w N` — timeout por resposta  

---

## whatweb

**Para que serve:**  
Identifica tecnologias expostas por aplicações web, como CMS, frameworks, bibliotecas JS, servidor web e outros componentes.

**Como funciona:**  
Analisa respostas HTTP e aplica assinaturas e plugins para detectar tecnologias, versões e indícios de stack.

**Principais parâmetros:**  
- `whatweb URL` — fingerprinting básico  
- `-a LEVEL` — nível de agressividade  
- `-v` — saída detalhada  
- `-H "Header: valor"` — adiciona cabeçalho customizado  
- `-U "User-Agent"` — altera o User-Agent  
- `--follow-redirect` — segue redirecionamentos  
- `--proxy URL` — usa proxy  
- `-t N` — número de threads  
- `--log-json arquivo` — salva saída em JSON  

---

## dnsmap

**Para que serve:**  
Realiza brute force de subdomínios para ampliar a superfície de ataque visível do alvo.

**Como funciona:**  
Testa entradas de uma wordlist contra o domínio informado e registra os subdomínios que resolvem para endereços válidos.

**Principais parâmetros:**  
- `dnsmap dominio.com` — enumeração padrão  
- `-w wordlist.txt` — define wordlist customizada  
- `-r arquivo.txt` — salva saída em texto  
- `-c arquivo.csv` — salva saída em CSV  
- `-d N` — adiciona atraso entre consultas  
- `-i IP` — ignora IPs já conhecidos na saída  

---

## dirb

**Para que serve:**  
Descobre diretórios, arquivos e endpoints ocultos em aplicações web por meio de wordlists.

**Como funciona:**  
Gera caminhos com base em uma wordlist, envia requisições HTTP e interpreta os códigos de resposta para detectar conteúdo existente.

**Principais parâmetros:**  
- `dirb URL` — execução padrão  
- `dirb URL wordlist.txt` — usa uma wordlist específica  
- `-X .php,.txt,.bak` — testa extensões adicionais  
- `-x extensoes.txt` — carrega lista de extensões  
- `-N 404` — ignora um código HTTP  
- `-r` — desativa recursão  
- `-R` — recursão interativa  
- `-u usuario:senha` — autenticação  
- `-c "cookie=valor"` — envia cookie  
- `-H "Header: valor"` — cabeçalho customizado  
- `-p proxy` — usa proxy  
- `-o saida.txt` — salva resultados  