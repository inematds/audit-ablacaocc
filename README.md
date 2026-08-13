# audit-ablacaocc — auditoria de ablação para Claude Code

Skill de **diagnóstico** (somente leitura) que audita seu `CLAUDE.md`, suas skills e seus hooks, classifica cada instrução e propõe uma versão mínima — sem tocar em nenhum arquivo.

> Toda linha da sua configuração é **culpada de complexidade até provar utilidade**.

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/audit-ablacaocc/guia/**

- Skill: [`SKILL.md`](SKILL.md)
- Base conceitual: [`doc/`](doc/) — resumo da palestra do Boris Cherny (Y Combinator, um dia após o lançamento do Opus 5)

---

## Índice

1. [A visão do Boris: por que apagar a própria config](#1-a-visão-do-boris-por-que-apagar-a-própria-config)
2. [Por que essa faxina é útil na prática](#2-por-que-essa-faxina-é-útil-na-prática)
3. [O que a skill faz (e o que ela nunca faz)](#3-o-que-a-skill-faz-e-o-que-ela-nunca-faz)
4. [Instalação](#4-instalação)
5. [Como usar](#5-como-usar)
6. [Dica de uso: chame skills o tempo todo](#6-dica-de-uso-chame-skills-o-tempo-todo)
7. [Ciclo recomendado](#7-ciclo-recomendado)
8. [Guia prático: publicar no Git e no portal](#8-guia-prático-publicar-no-git-e-no-portal)

---

## 1. A visão do Boris: por que apagar a própria config

Boris Cherny criou o Claude Code na Anthropic. A tese dele, resumida:

**Configuração envelhece.** Cada instrução que você escreve é o conserto da fraqueza de um modelo específico. Quando o modelo seguinte deixa de ter aquela fraqueza, a instrução vira peso morto — e continua sendo relida a cada uso, ocupando contexto e limitando o modelo.

Foi literalmente o que aconteceu no Claude Code: boa parte do prompt de sistema existia só para corrigir comportamentos que o modelo deveria saber executar sozinho. Com o Opus 5, a equipe **removeu mais de 80% do prompt de sistema**. O que sobrou é quase todo segurança, permissões, análise estática e interface.

**Ablação** é o método. É termo de pesquisa: você remove algo para medir o impacto. Aplicado a prompt, significa apagar tudo e trazer de volta linha por linha, descobrindo o que cada linha realmente faz. O conselho do Boris para quem só *usa* Claude Code:

> A cada ~6 meses, e principalmente em lançamento grande de modelo: apague seu `CLAUDE.md`, apague suas skills, apague seus hooks — e veja o que o modelo faz sem eles.

A reconstrução tem quatro etapas, e a quarta é a que importa:

1. Apague a config.
2. Use a ferramenta em **trabalho real**, não em teste hipotético.
3. Observe onde ela vai bem e onde tropeça.
4. Só devolva uma instrução **depois de ver o modelo falhar repetidamente na mesma coisa**.

Duas razões para essa disciplina: você é um péssimo previsor de qual instrução o modelo precisa, e cada linha mantida custa contexto em toda execução.

Os outros pilares da visão:

- **Especificação excessiva é o erro mais comum** — e é mais frequente em quem tem anos de engenharia. Roteirizar "faça A, depois B, depois C" funcionava com modelos antigos; hoje impede o modelo de achar um caminho melhor. Escreva **tarefa + guardrails + critérios de saída**, e volte depois para conferir.
- **Verificação é a alavanca que quase todo mundo erra.** Prompt curto com um jeito real de o modelo conferir o próprio trabalho ganha de prompt gigante sem verificação. O exemplo clássico: reescrever o app Electron em Swift, rodar o original numa VM, tirar screenshot, comparar pixel a pixel, não parar até terminar. Produzir → observar → comparar → condição de parada.
- **Mire acima do teto.** Dê uma tarefa um pouco mais difícil do que você acha que o modelo aguenta, e reteste seus fracassos antigos a cada versão nova — o motivo da falha pode ter desaparecido.
- **Trabalhar com modelo virou ciência empírica**, não teórica. É mais parecido com conhecer uma criatura viva do que configurar um sistema. O nível certo de instrução é o mesmo que você usaria orientando um colega de trabalho.
- **Não existe truque secreto.** O ciclo é: tarefa difícil → meios de verificar → observar onde trava → corrigir *aquilo* (prompt melhor, ou uma skill, ou um MCP) → repetir.

Para testar sozinho, Boris citou duas alavancas: a flag de prompt de sistema na inicialização, e `CLAUDE_CODE_SIMPLE=1` (remove todos os prompts de sistema, inclusive os das ferramentas — é a linha de base de ablação usada internamente). Detalhe contraintuitivo: sem os prompts, o modelo fica *ligeiramente mais inteligente*; eles são mantidos porque fazem o produto se comportar como a pessoa espera.

## 2. Por que essa faxina é útil na prática

Quem usa Claude Code há um tempo acumula a mesma camada de sedimento:

| Sintoma | O que costuma ser |
|---|---|
| `CLAUDE.md` de 300 linhas com exceções datadas | correções de modelos que já saíram de cena |
| A mesma regra no `CLAUDE.md` e dentro de 3 skills | redundância que gera conflito quando uma das cópias muda |
| Skill que ensina "como pensar" em 400 linhas | raciocínio genérico que o modelo já faz sozinho |
| Passo a passo rígido de 12 etapas | microgerenciamento que trava a rota melhor |
| Duas regras que se contradizem | o modelo escolhe uma, e você não sabe qual |
| Nada dizendo *como conferir* que ficou certo | o buraco mais caro de todos |

O custo é real e composto: contexto queimado em toda chamada, autonomia reduzida, comportamento inconsistente, e regras zumbis que ninguém ousa apagar porque ninguém sabe mais o que elas seguram.

O ganho da faxina:

- **Contexto de volta** para o trabalho de verdade.
- **Menos contradição** — regra que só existe em um lugar não briga com nada.
- **Mais autonomia** — critérios de saída deixam o modelo achar caminho melhor que o seu.
- **Config auditável** — você passa a saber *por que* cada linha está lá, porque cada uma sobreviveu a uma pergunta.
- **Verificação entra na conta** — a auditoria expõe onde falta um jeito objetivo de provar que terminou certo.

O objetivo **não** é maximizar a redução. É maximizar:

**qualidade + autonomia + verificabilidade ÷ complexidade**

Por isso a skill preserva explicitamente o que o modelo não consegue inferir: identidade do projeto, caminhos e fontes de verdade, branding, segurança, compliance, integrações, contratos de interface e convenções internas.

## 3. O que a skill faz (e o que ela nunca faz)

**Faz:**

- lê `CLAUDE.md` (projeto + global), skills, hooks e `settings.json`;
- classifica cada instrução — `CONTEXTO`, `GUARDRAIL`, `CRITÉRIO DE QUALIDADE`, `VERIFICAÇÃO`, `INTEGRAÇÃO/FERRAMENTA`, `PROCEDIMENTO REPETÍVEL`, `MICROGERENCIAMENTO`, `REDUNDÂNCIA`, `LEGADO/OBSOLETA`, `AMBÍGUA/NÃO COMPROVADA`;
- decide `KEEP` / `SIMPLIFY` / `MOVE` / `MERGE` / `TEST` / `REMOVE` (na dúvida, `TEST`);
- audita skill por skill — `KEEP` / `SIMPLIFY` / `MERGE` / `SPLIT` / `LOAD-ON-DEMAND` / `CONVERT-TO-CONTEXT` / `DELETE-CANDIDATE`;
- entrega relatório em 10 seções: resumo executivo, métricas, problemas por arquivo, candidatas à remoção, redundâncias e conflitos, skills, `CLAUDE.md` mínimo proposto, skills propostas, plano de teste de ablação (versões A/B/C) e top 10 mudanças por impacto ÷ risco.

**Nunca faz:** editar, mover, apagar, sobrescrever config ou commitar. É diagnóstico. Aplicar é decisão sua, num pedido separado — e é de propósito: auditoria que já sai mexendo é auditoria em que você não confia.

Diferença de vizinhança: `memory-audit` cuida da **memória** guardada da sessão. Esta aqui cuida da **configuração do agente**.

## 4. Instalação

```bash
git clone https://github.com/inematds/audit-ablacaocc.git
mkdir -p ~/.claude/skills/audit-ablacao
cp audit-ablacaocc/SKILL.md ~/.claude/skills/audit-ablacao/SKILL.md
```

Ou, para instalar só no projeto atual:

```bash
mkdir -p .claude/skills/audit-ablacao
cp /caminho/audit-ablacaocc/SKILL.md .claude/skills/audit-ablacao/SKILL.md
```

Reinicie a sessão do Claude Code para ele carregar a skill.

## 5. Como usar

Chamada direta:

```
/audit-ablacao
```

Ou por gatilho, em linguagem natural:

- "faz uma auditoria de ablação do meu CLAUDE.md global"
- "audita as skills deste projeto, meu prompt tá inchado"
- "o que dá pra remover da minha config sem quebrar nada?"

Escopos que valem a pena:

| Escopo | Quando |
|---|---|
| Global (`~/.claude/`) | a cada ~6 meses ou em lançamento grande de modelo |
| Um projeto | quando o `CLAUDE.md` dele passa de ~150 linhas |
| Um conjunto de skills | quando 3+ skills brigam pelo mesmo gatilho |

Depois do relatório, o fluxo saudável é: escolher o **Top 10 por impacto ÷ risco**, aplicar em outra sessão, e rodar o **plano de ablação** (versões A / B / C) nas tarefas reais do projeto antes de dar a mudança por boa.

## 6. Dica de uso: chame skills o tempo todo

Essa é uma prática que eu apliquei e que muda o resultado mais do que qualquer ajuste de prompt: **use skills ao máximo — chamando direto ou dentro dos projetos.**

Por quê:

- **Skill é procedimento, não instrução global.** Regra que vive no `CLAUDE.md` é lida em *toda* execução, inclusive nas 90% que não têm nada a ver com ela. A mesma regra dentro de uma skill só custa contexto quando a tarefa é aquela. Mover procedimento do `CLAUDE.md` para uma skill é a forma mais barata de emagrecer a config sem perder nada — é exatamente o `MOVE` e o `LOAD-ON-DEMAND` do relatório.
- **Chamar direto (`/nome-da-skill`) elimina a loteria do gatilho.** Você para de depender de a descrição casar com a sua frase. Quando várias skills competem pelo mesmo assunto, invocação explícita é o desempate — e mata a necessidade de escrever regra de roteamento no `CLAUDE.md`.
- **Skill dentro do projeto (`.claude/skills/`) versiona junto com o código.** O procedimento anda com o repo, entra no PR, é revisável e não vaza para os outros projetos.
- **Skill é a unidade certa de conserto.** No ciclo do Boris, quando o modelo tropeça você tem três remédios: prompt melhor (instrução estava obscura), **skill** (falta procedimento repetível) ou MCP (falta contexto que ele não alcança). Escolher o remédio certo evita o reflexo de despejar mais uma regra no `CLAUDE.md`.
- **Skill é fácil de auditar e de aposentar.** Ela tem fronteira, nome e escopo. Dá para desativar uma skill e medir; desativar "aquele parágrafo do meio do `CLAUDE.md`" é bem mais difícil.

Na prática: `CLAUDE.md` fica só com o que é verdade **sempre** (identidade, guardrails, fontes de verdade, segurança). Todo o resto — procedimento, formato, receita, integração — vira skill, invocada direto quando você sabe o que quer.

## 7. Ciclo recomendado

1. **Rodar** `/audit-ablacao` no escopo escolhido.
2. **Cortar** o Top 10 por impacto ÷ risco (numa sessão separada — a skill não aplica).
3. **Usar em trabalho real** por alguns dias. Não em teste hipotético.
4. **Anotar as falhas** e devolver instrução só quando a mesma falha se repetir — e na forma mais curta possível.
5. **Repetir** a cada ~6 meses ou a cada lançamento grande de modelo.

## 8. Guia prático: publicar no Git e no portal

Receita que este próprio repositório seguiu. Vale para qualquer skill/projeto INEMA.

### 8.1 Estrutura do repo

```
audit-ablacaocc/
├── SKILL.md        # a skill (raiz = instalável com um cp)
├── README.md       # este arquivo
├── capa/capa.png   # capa 1280x720 (skill capa-inema)
├── guia/index.html # landing + guia self-contained (skill projetos-landing-guia)
└── doc/            # material de base
```

Regra que evita retrabalho: **o guia vai em `guia/`, nunca na raiz** — a raiz é do código/skill. E é **uma página dentro do próprio repo**, nunca um repositório separado. Nome do repo remoto = nome da pasta local.

### 8.2 Git — do zero ao push

```bash
cd ~/projetos/audit-ablacaocc

git init -b main
git config user.name  "inematds"
git config user.email "inematds@gmail.com"   # autor acompanha a conta de destino
git add -A
git commit -m "feat: skill audit-ablacao + guia"

gh repo create inematds/audit-ablacaocc --public --source=. --remote=origin --push
```

Checagem que salva tempo: confira `git config user.email` **antes** de commitar. Autor divergente da conta de destino é o motivo nº 1 de deploy bloqueado depois.

Se o push falhar por causa de `.github/workflows/*.yml` (falta de escopo `workflow` no token HTTPS), empurre esse commit por SSH:

```bash
git push git@github.com:inematds/audit-ablacaocc.git main
```

### 8.3 GitHub Pages

Servir da **raiz do branch `main`** (sem Actions — assim o problema de escopo `workflow` nem aparece):

```bash
gh api -X POST repos/inematds/audit-ablacaocc/pages \
  -f 'source[branch]=main' -f 'source[path]=/'
```

Confirme antes de dizer que publicou:

```bash
curl -sI https://inematds.github.io/audit-ablacaocc/guia/ | head -1   # espera 200
```

A primeira publicação leva ~1 min. Como o guia está em `guia/`, a URL pública termina em `/guia/`.

### 8.4 Portal (inema.club)

Com a URL do Pages respondendo, publique nas 3 superfícies INEMA (portal, inemabuscas e catálogo PRO):

```
/atualiza-portal https://inematds.github.io/audit-ablacaocc/guia/
```

A skill cuida do card, da capa e do commit + push nos três repositórios.

### 8.5 Definição de "publicado"

Publicar = **commit + push no `origin`**. O deploy é automático via webhook git → Vercel (e GitHub Pages, quando for o caso) e não está sob nosso controle: nada de abrir dashboard, ficar checando status ou re-disparar deploy. Terminou quando o push entrou no `origin` e, no caso do Pages, quando o arquivo responde.

---

## Créditos

Método e visão: Boris Cherny (Anthropic), palestra na Y Combinator sobre o corte de 80% do prompt de sistema do Claude Code. Resumos e material de base em [`doc/`](doc/). Empacotado como skill por INEMA.
