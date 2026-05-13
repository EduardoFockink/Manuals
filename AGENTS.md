# Diretrizes para Agentes

Padrões para escrita e manutenção de manuais neste repositório.

## Idioma e tom

- PT-BR. Termos técnicos consagrados em inglês (pipe, stream, inode, symlink).
- Prosa direta. Sem introduções, conclusões ou metacomentários.
- Sem emojis.
- Foco no "porquê" e nas pegadinhas, não só na sintaxe — o `man` já descreve flags.

## Estrutura

```
linux/             shell, terminal, ambiente
networks/          rede e segurança
operating-systems/ conceitos de SO
```

Um manual por arquivo `.md`, em `snake_case` ou `kebab-case` (manter consistência por diretório). Editar arquivo existente antes de criar novo. Quando um tópico cresce demais dentro de um doc, extrair para arquivo dedicado e deixar uma referência cruzada no original.

## Formato

Estilo wiki, baseado em `linux/tmux.md`, `linux/busca_filtragem.md` e `linux/compactacao.md`.

### Esqueleto

```markdown
# nome

Parágrafo único de 1–3 frases: o que é, para quê serve, quando usar. Mencionar
docs relacionados via link inline quando fizer sentido.

## Sumário

- [Seção](#seção)
  - [Subseção](#subseção)

---

## Seção

Prosa curta de contexto da seção (1–2 frases) quando a seção agrupa vários
comandos relacionados.

### comando

Descrição em 1–2 frases que explica o porquê e a nuance importante (ex: "não
há lixeira", "lê todo o arquivo na memória", "não é POSIX, vem com o vim").

- `comando args` — efeito
- `comando -f` — efeito da flag

---

## Pipelines comuns (opcional)

Bloco de código `sh` com receitas práticas comentadas, quando o doc trata de
ferramentas que se encadeiam.
```

### Regras de marcação

- `#` apenas no título do doc. `##` para seções. `###` para comandos/subitens. Não pular níveis.
- Sumário com links de âncora (`[texto](#anchor)`). Anchors em minúsculas, espaços viram `-`, acentos preservados.
- Comandos inline em backticks. Blocos com linguagem declarada (`sh`, `tmux`, `lua`).
- Bullets de comandos no formato `` `comando args` — descrição curta `` (use travessão `—`, não hífen).
- Separador `---` entre seções H2.
- Sem metadados (data, autor) no corpo — Git registra.
- Sem placeholders ou seções vazias.

## Conteúdo

- Cada entrada de comando deve explicar **quando usar** e **o que diferencia** das alternativas, não apenas o que faz. "Igual a `cp` mas...", "diferente de `more`...", "para múltiplos arquivos use `tar` antes".
- Destacar pegadinhas: comportamento destrutivo, diferenças GNU vs BSD, requisitos não-POSIX, limites de portabilidade.
- Listas para variações de flags ou formas de uso. Tabelas só para comparações reais entre alternativas.
- Exemplos executáveis, não pseudocódigo.
- Diagramas ASCII só quando a hierarquia for essencial (ex: árvore tmux).
- Cross-links inline para outros docs do repo: `[\`grep\`](busca_filtragem.md#grep)`, não notas de rodapé.

## README

Atualizar `README.md` sempre que:

- Criar novo manual — adicionar bullet na seção do diretório correspondente, com 1 linha de descrição.
- Mudar escopo significativo de um doc existente (novos comandos centrais, renomeação, divisão em arquivos).

A descrição no README é uma linha resumindo os tópicos principais, não a intro do doc.

## Commits

- PT-BR. Imperativo no assunto, ≤50 caracteres.
- Sem footer de IA. Sem `Co-Authored-By`.
- Corpo apenas quando o porquê não for óbvio pelo título; cada bullet começa por verbo no infinitivo descrevendo a mudança.
- **Granularidade: um commit por arquivo modificado**, salvo quando a mudança é coordenada (ex: novo doc + entrada no README; renomeação que afeta cross-links).
- Atualizações de README que acompanham um novo doc entram no mesmo commit.
- Não misturar mudanças de conteúdo com refatorações estruturais.

### Exemplos de assunto

- `Adiciona manual de compactação`
- `Documenta switch-client no tmux`
- `Reescreve comandos_shell em estilo wiki`
- `Corrige mnemônico do tar`

## Evitar

- Frases de abertura genéricas ("Este documento descreve...").
- Repetir o nome da ferramenta em toda frase.
- Labels repetitivos por entrada (`**Para que serve:**`, `**Como funciona:**`).
- Tabelas decorativas.
- Traduzir termos técnicos consagrados.
- Commits "chore: várias melhorias" — preferir mensagens específicas.
