# Diretrizes para Agentes

Padrões para escrita e manutenção de manuais neste repositório.

## Idioma e tom

- PT-BR. Termos técnicos consagrados em inglês.
- Prosa direta. Sem introduções, conclusões ou metacomentários.
- Sem emojis.

## Estrutura

```
linux/             shell, terminal, ambiente
networks/          rede e segurança
operating-systems/ conceitos de SO
```

Um manual por arquivo `.md`, em `kebab-case`. Editar arquivo existente antes de criar novo.

## Formato

Estilo wiki, baseado em `linux/tmux.md`.

```markdown
# nome

Parágrafo único: o que é e quando usar. Até 3 frases.

## Sumário

- [Seção](#seção)

---

## Seção

Prosa curta de contexto.

### subcomando

Descrição em uma frase.

- `comando args` — efeito
- `comando -f` — efeito da flag
```

Regras:

- `#` apenas no título; `##` para seções; `###` para subitens. Não pular níveis.
- Comandos inline em crases. Blocos com linguagem declarada.
- Sem metadados (data, autor) no corpo — Git registra.
- Sem placeholders ou seções vazias.

## Conteúdo

- Cada seção responde: o que faz, quando usar, como invocar.
- Listas para variações de flags ou formas de uso.
- Exemplos executáveis, não pseudocódigo.
- Diagramas ASCII apenas quando a hierarquia for essencial.

## Commits

PT-BR, imperativo, ≤50 caracteres no assunto. Sem footer de IA, sem `Co-Authored-By`. Corpo apenas quando o porquê não for óbvio.

## Evitar

- Frases de abertura genéricas.
- Repetir o nome da ferramenta em toda frase.
- Tabelas decorativas.
- Traduzir termos técnicos consagrados.
