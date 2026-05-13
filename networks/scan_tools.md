# Ferramentas de scan

Comandos de reconhecimento usados em offensive security e auditoria de rede: consulta de registro (`whois`), DNS (`nslookup`, `host`), traçado de rota (`traceroute`), fingerprinting web (`whatweb`) e enumeração por wordlist (`dnsmap`, `dirb`). A maioria depende apenas de DNS/HTTP, então roda em qualquer máquina sem privilégios especiais.

## Sumário

- [whois](#whois)
- [nslookup](#nslookup)
- [host](#host)
- [traceroute](#traceroute)
- [whatweb](#whatweb)
- [dnsmap](#dnsmap)
- [dirb](#dirb)

---

## whois

Consulta dados de registro de domínios, blocos IP e ASNs: registrador, datas, contatos, servidores responsáveis. O cliente fala com um servidor WHOIS e segue redirecionamentos até o servidor autoritativo do recurso. Útil na fase inicial de reconhecimento para mapear ownership e infraestrutura.

- `whois alvo` — consulta domínio ou IP
- `whois -h HOST alvo` — força servidor WHOIS específico
- `whois -I alvo` — consulta primeiro a IANA
- `whois --no-recursion alvo` — não segue redirecionamentos
- `whois -i ATTR valor` — busca por atributo
- `whois -T TYPE alvo` — filtra por tipo de objeto

---

## nslookup

Consulta DNS interativa ou pontual. Aceita servidor de DNS específico no segundo argumento — útil para comparar respostas entre resolvers e detectar DNS poisoning ou split-horizon.

- `nslookup alvo` — consulta básica usando o resolver do sistema
- `nslookup alvo 8.8.8.8` — consulta usando DNS específico
- `nslookup -query=TYPE alvo` — define tipo de registro (`A`, `AAAA`, `MX`, `NS`, `TXT`, `PTR`)
- `nslookup -query=ANY alvo` — todos os tipos (muitos resolvers bloqueiam)
- Modo interativo: `set norecurse`, `set timeout=N`, `set retry=N`

---

## host

Resolução DNS objetiva e direta — alternativa rápida ao `nslookup`, melhor para shell scripts. Suporta tentativa de zone transfer, que pode revelar a zona inteira se o servidor estiver mal configurado.

- `host alvo` — resolução básica
- `host alvo 8.8.8.8` — consulta em nameserver específico
- `host -t TYPE alvo` — escolhe tipo de registro
- `host -a alvo` — saída detalhada (todos os registros)
- `host -l dominio.com ns.dominio.com` — tenta zone transfer (AXFR)
- `host -r alvo` — desativa recursão
- `host -W N alvo` — timeout em segundos

---

## traceroute

Mapeia o caminho de rede até o destino enviando pacotes com TTL crescente — cada roteador no caminho responde quando o TTL expira. Revela hops, latência e pontos de filtragem. Pacotes UDP por padrão; muitos firewalls os bloqueiam, então `-I` (ICMP) ou `-T` (TCP) costuma ir mais longe.

- `traceroute alvo` — execução padrão (UDP)
- `traceroute -I alvo` — usa ICMP (igual ao traceroute do Windows)
- `traceroute -T -p 443 alvo` — TCP SYN na porta 443 (atravessa firewalls que liberam HTTPS)
- `traceroute -n alvo` — não resolve nomes (mais rápido)
- `traceroute -f N alvo` — TTL inicial
- `traceroute -m N alvo` — número máximo de hops
- `traceroute -w N alvo` — timeout por resposta

---

## whatweb

Fingerprinting de aplicações web. Analisa headers HTTP, conteúdo HTML e padrões de URL para identificar CMS, frameworks, bibliotecas JS, servidores e versões. Útil antes de escolher exploits ou wordlists específicas.

- `whatweb URL` — fingerprinting básico
- `whatweb -a 3 URL` — agressividade 1 a 4 (4 = mais consultas, mais barulho)
- `whatweb -v URL` — saída detalhada com plugins acionados
- `whatweb -H "Header: valor" URL` — header customizado
- `whatweb -U "User-Agent" URL` — altera User-Agent
- `whatweb --follow-redirect URL` — segue redirecionamentos
- `whatweb --proxy http://127.0.0.1:8080 URL` — usa proxy (Burp/ZAP)
- `whatweb -t N URL` — número de threads
- `whatweb --log-json saida.json URL` — saída JSON

---

## dnsmap

Brute force de subdomínios contra uma wordlist para ampliar a superfície visível do alvo. Diferente de varreduras ativas no host, só faz consultas DNS — invisível para o servidor web final, mas pode aparecer em logs do DNS autoritativo.

- `dnsmap dominio.com` — enumeração com wordlist embutida
- `dnsmap dominio.com -w wordlist.txt` — wordlist customizada
- `dnsmap dominio.com -r saida.txt` — salva em texto
- `dnsmap dominio.com -c saida.csv` — salva em CSV
- `dnsmap dominio.com -d N` — atraso entre consultas (evita rate-limiting)
- `dnsmap dominio.com -i IP` — ignora IPs já conhecidos no relatório

---

## dirb

Descobre diretórios, arquivos e endpoints ocultos enviando requisições HTTP baseadas em wordlist. Interpreta códigos de resposta para distinguir o que existe do que não existe. Para alvos modernos, prefira `gobuster`, `feroxbuster` ou `ffuf` (mais rápidos e flexíveis); `dirb` segue útil em ambientes mínimos.

- `dirb URL` — execução padrão com wordlist embutida
- `dirb URL wordlist.txt` — wordlist específica
- `dirb URL -X .php,.txt,.bak` — testa extensões adicionais
- `dirb URL -x extensoes.txt` — carrega lista de extensões de arquivo
- `dirb URL -N 404` — ignora código HTTP específico
- `dirb URL -r` — desativa recursão
- `dirb URL -R` — recursão interativa (pergunta a cada subdiretório encontrado)
- `dirb URL -u usuario:senha` — autenticação básica
- `dirb URL -c "cookie=valor"` — envia cookie
- `dirb URL -H "Header: valor"` — header customizado
- `dirb URL -p http://127.0.0.1:8080` — usa proxy
- `dirb URL -o saida.txt` — salva resultados
