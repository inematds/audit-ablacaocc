---
name: audit-ablacao
description: Auditoria de ablação (somente diagnóstico) do CLAUDE.md, das skills e dos hooks de um projeto ou da config global. Classifica cada instrução (contexto, guardrail, critério, verificação, microgerenciamento, redundância, legado) e entrega relatório + proposta de versão mínima + plano de teste de ablação. NUNCA edita, move ou apaga arquivos. Gatilhos - "auditoria de ablação", "audita meu CLAUDE.md", "audita minhas skills", "meu prompt está inchado", "cortar instruções", "ablation audit", "enxugar skills", "o que dá pra remover da config".
---

# Auditoria de Ablação — CLAUDE.md, skills e hooks

Diagnóstico da configuração do agente. Descobre quais instruções ainda pagam o próprio custo e quais viraram peso morto de modelos antigos.

**Somente leitura.** Esta skill lê, classifica e propõe. Não escreve, não edita, não move, não apaga, não roda `git`. A aplicação das mudanças é decisão e ação do usuário, em outra sessão/pedido.

Escopo: configuração de agente (`CLAUDE.md`, `~/.claude/skills/*/SKILL.md`, `.claude/settings*.json`, hooks, `AGENTS.md`). Para auditar a **memória** guardada da sessão, use `memory-audit` — é outra coisa.

## Princípio central

Toda linha é **culpada de complexidade até provar utilidade**. Uma instrução existir não prova que ela é necessária.

Mas o objetivo não é reduzir ao máximo. É maximizar:

**qualidade + autonomia + verificabilidade ÷ complexidade**

Não corte contexto que o modelo não consegue inferir sozinho: identidade do projeto, caminhos de arquivos, fontes de verdade, branding, segurança, compliance, contratos de interface, integrações, convenções internas.

## Inventário

Leia o que estiver no escopo pedido (se o usuário não disser, pergunte em texto: config global, um projeto, ou ambos):

- `CLAUDE.md` do projeto e o global (`~/.claude/CLAUDE.md`);
- as skills relevantes;
- hooks e `settings.json`, quando existirem;
- arquivos referenciados, só quando forem necessários para entender uma instrução.

Classifique cada instrução em uma destas categorias:

`CONTEXTO` · `GUARDRAIL` · `CRITÉRIO DE QUALIDADE` · `VERIFICAÇÃO` · `INTEGRAÇÃO/FERRAMENTA` · `PROCEDIMENTO REPETÍVEL` · `MICROGERENCIAMENTO` · `REDUNDÂNCIA` · `LEGADO/OBSOLETA` · `AMBÍGUA/NÃO COMPROVADA`

E decida: `KEEP` · `SIMPLIFY` · `MOVE` · `MERGE` · `TEST` · `REMOVE`.

Sem evidência suficiente, prefira `TEST` a `KEEP`.

## Perguntas por instrução

1. O que ela tenta evitar ou garantir?
2. O modelo atual ainda precisa dela?
3. Ela diz **o que deve acontecer** ou tenta dirigir **como o modelo pensa/executa**?
4. Está duplicada em outro arquivo?
5. Limita autonomia sem necessidade?
6. Existe forma mais curta de preservar a intenção?
7. O que quebra se ela sumir?

## Procure especificamente

Passo a passo desnecessário · regras duplicadas entre `CLAUDE.md` e skills · skills grandes demais · skills ensinando raciocínio genérico · instruções que compensam modelos antigos · excesso de exemplos · formatação rígida sem motivo · exceções acumuladas · contradições e regras que competem · contexto global que só serve a poucas tarefas · regras que deveriam ser critérios de saída · **verificações ausentes** · regras substituíveis por teste objetivo.

## Converta microgerenciamento em critério

De "faça A, depois B, depois C" para:

> Produza X. Respeite Y. O resultado deve atingir Z. Verifique usando W. Escolha a estratégia.

## Cada skill

Ela precisa existir? Que problema recorrente resolve? O que nela é contexto, o que é procedimento, o que é microgerenciamento? Caberia numa instrução curta? Deveria carregar só sob demanda? Dá pra dividir ou fundir? Contém coisas que o modelo já faz sozinho?

Classifique: `KEEP` · `SIMPLIFY` · `MERGE` · `SPLIT` · `LOAD-ON-DEMAND` · `CONVERT-TO-CONTEXT` · `DELETE-CANDIDATE`.

## Formato da entrega

Relatório em chat ou, se o usuário pedir arquivo, num `.md` **novo** dentro do diretório de trabalho (nunca sobrescrevendo config). Seções, nesta ordem:

1. **Resumo executivo** — os 5 maiores problemas.
2. **Métricas** — instruções analisadas; contagem de `KEEP`/`SIMPLIFY`/`MOVE`/`MERGE`/`TEST`/`REMOVE`; redução estimada em %.
3. **Problemas por arquivo** — tabela: Arquivo | Problema | Severidade | Recomendação | Justificativa.
4. **Candidatas à remoção** — tabela: Trecho | Arquivo | Motivo | Risco | Como testar.
5. **Redundâncias e conflitos** — duplicações e contradições, com os arquivos envolvidos.
6. **Skills** — tabela: Skill | Função atual | Diagnóstico | Recomendação | Redução estimada.
7. **`CLAUDE.md` proposto** — versão mínima completa, em bloco de código (proposta, não aplicada).
8. **Skills propostas** — versões reduzidas das skills mantidas, em bloco de código.
9. **Plano de ablação** — 5 a 10 tarefas reais e representativas do projeto, comparando: **A** config atual · **B** simplificada · **C** mínima (contexto + objetivo + guardrails + critérios + verificação). Avalie qualidade, aderência, autonomia, número de correções humanas, consistência, tempo, uso desnecessário de ferramentas, complexidade e capacidade de verificar o próprio trabalho.
10. **Top 10 mudanças** — ordenadas por impacto esperado ÷ risco.

## Regra de reintrodução

Uma instrução removida só volta quando: (1) uma falha real acontecer, (2) a mesma classe de falha se repetir, (3) ficar claro que uma instrução específica resolve, e (4) ela couber na forma mais curta possível.

## Regras

1. **Não altere nada.** Nenhum edit, nenhum `rm`, nenhum `mv`, nenhum commit. Só leitura e relatório.
2. **Leia antes de opinar.** Nada de diagnóstico por suposição sobre arquivo não lido; se não conseguiu ler, diga isso.
3. **Cite o trecho.** Toda recomendação de remoção aponta arquivo e trecho.
4. **Curto não é sempre melhor.** Explique cada remoção relevante e o risco dela.
5. **Na dúvida, `TEST`.**
6. Sem emoji, sem hype. Terso e concreto.
