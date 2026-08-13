Quero que você faça uma **AUDITORIA DE ABLAÇÃO** do meu `CLAUDE.md` e das minhas skills.

## OBJETIVO

Reduzir instruções desnecessárias, redundantes, antigas ou excessivamente prescritivas sem perder:

- contexto importante do projeto/negócio;
- regras realmente obrigatórias;
- preferências de marca e formato;
- segurança;
- integrações;
- critérios de qualidade;
- comportamentos que já demonstraram ser necessários.

## PRINCÍPIO CENTRAL

Não assuma que uma instrução é necessária só porque existe.

Considere cada linha **culpada de complexidade até que sua utilidade seja demonstrada**.

Ao mesmo tempo, não simplifique de forma irresponsável. Contexto específico do negócio, localização de arquivos, branding, compliance, segurança, integrações e procedimentos realmente determinísticos podem ser essenciais.

## 1. INVENTÁRIO

Leia:

- `CLAUDE.md`;
- todas as skills relevantes;
- arquivos diretamente referenciados por eles quando forem necessários para entender uma instrução.

Classifique cada instrução como:

- **CONTEXTO** — informação que o modelo precisa conhecer.
- **GUARDRAIL** — restrição obrigatória.
- **CRITÉRIO DE QUALIDADE** — define o que significa um bom resultado.
- **VERIFICAÇÃO** — define como confirmar que o resultado está correto.
- **INTEGRAÇÃO / FERRAMENTA** — necessária para acessar sistemas ou ferramentas.
- **PROCEDIMENTO REPETÍVEL** — processo que realmente justifica uma skill.
- **MICROGERENCIAMENTO** — detalha passos que o modelo provavelmente consegue decidir sozinho.
- **REDUNDÂNCIA** — repete algo existente em outro lugar.
- **LEGADO / OBSOLETA** — parece corrigir limitações de modelos anteriores.
- **AMBÍGUA / NÃO COMPROVADA** — não há evidência suficiente de que ajuda.

## 2. AUDITORIA LINHA A LINHA

Para cada instrução relevante, responda:

1. O que esta instrução tenta evitar ou garantir?
2. O modelo atual provavelmente precisa dela?
3. Ela descreve **o que deve acontecer** ou tenta determinar **como o modelo deve pensar/executar**?
4. Existe duplicação em outro arquivo?
5. Ela limita desnecessariamente a autonomia?
6. Existe uma forma mais curta de preservar sua intenção?
7. O que pode dar errado se ela for removida?

Dê uma decisão:

- `KEEP`
- `SIMPLIFY`
- `MOVE`
- `MERGE`
- `TEST`
- `REMOVE`

Quando não houver evidência suficiente, prefira `TEST` a `KEEP`.

## 3. PROCURE ESPECIFICAMENTE ESTES PROBLEMAS

Identifique:

- instruções passo a passo desnecessárias;
- regras duplicadas entre `CLAUDE.md` e skills;
- skills excessivamente grandes;
- skills tentando ensinar raciocínio genérico;
- instruções criadas para compensar modelos antigos;
- exemplos excessivos;
- formatação rígida sem necessidade;
- exceções acumuladas ao longo do tempo;
- contradições;
- regras que competem entre si;
- contexto carregado globalmente que só é útil em poucas tarefas;
- regras que deveriam ser critérios de saída;
- verificações que estão ausentes;
- regras que poderiam ser substituídas por teste ou validação objetiva.

## 4. PRESERVE O QUE REALMENTE É VALIOSO

Tenha cuidado para não remover:

- identidade e objetivos do projeto;
- arquitetura relevante;
- fontes de verdade;
- diretórios importantes;
- convenções internas essenciais;
- requisitos de segurança;
- compliance;
- branding;
- contratos de interface;
- critérios objetivos de qualidade;
- mecanismos de validação;
- conhecimento que o modelo não conseguiria descobrir sozinho.

## 5. CONVERTA MICROGERENCIAMENTO EM CRITÉRIOS

Sempre que possível, transforme:

> Faça A, depois B, depois C, depois D.

em:

> Produza X.  
> Respeite Y.  
> O resultado deve atingir Z.  
> Verifique usando W.  
> Escolha autonomamente a melhor estratégia.

## 6. AUDITE CADA SKILL

Para cada skill, determine:

- ela realmente precisa existir?
- qual problema recorrente resolve?
- qual parte é contexto?
- qual parte é procedimento?
- qual parte é microgerenciamento?
- poderia ser apenas uma instrução curta?
- deveria ser carregada apenas sob demanda?
- pode ser dividida?
- pode ser fundida com outra?
- contém instruções que o modelo atual provavelmente já executa sem ajuda?

Classifique cada skill como:

- `KEEP`
- `SIMPLIFY`
- `MERGE`
- `SPLIT`
- `LOAD-ON-DEMAND`
- `CONVERT-TO-CONTEXT`
- `DELETE-CANDIDATE`

## 7. CRIE UMA VERSÃO MÍNIMA

Depois da análise, gere:

### A. Diagnóstico do `CLAUDE.md` atual

Explique seus principais problemas.

### B. `CLAUDE.md MINIMAL`

Crie uma versão significativamente menor preservando apenas:

- contexto necessário;
- guardrails;
- critérios de qualidade;
- verificações;
- integrações;
- regras comprovadamente úteis.

### C. `SKILLS MINIMAL`

Para cada skill mantida, gere uma versão reduzida.

**Não altere nenhum arquivo ainda. Apenas apresente a proposta.**

## 8. PLANO DE ABLAÇÃO

Escolha entre 5 e 10 tarefas reais e representativas deste projeto.

Para cada uma, proponha comparar:

### VERSÃO A
Configuração atual.

### VERSÃO B
`CLAUDE.md` simplificado + skills simplificadas.

### VERSÃO C
Configuração mínima:

**contexto essencial + objetivo + guardrails + critérios + verificação**

Avalie cada versão em:

- qualidade final;
- aderência ao objetivo;
- autonomia;
- quantidade de correções humanas;
- consistência;
- tempo;
- uso desnecessário de ferramentas;
- complexidade;
- capacidade de verificar o próprio trabalho.

## 9. REGRA PARA REINTRODUZIR INSTRUÇÕES

Uma instrução removida só deve voltar se:

1. uma falha real ocorrer;
2. a mesma classe de falha se repetir;
3. ficar claro que uma instrução específica resolveria o problema;
4. ela puder ser escrita da forma mais curta possível.

## 10. FORMATO DA RESPOSTA

Entregue exatamente estas seções:

### 1. Resumo executivo
Os 5 maiores problemas encontrados.

### 2. Métricas
- total de instruções analisadas;
- `KEEP`;
- `SIMPLIFY`;
- `MOVE`;
- `MERGE`;
- `TEST`;
- `REMOVE`;
- redução estimada em %.

### 3. Problemas por arquivo

| Arquivo | Problema | Severidade | Recomendação | Justificativa |
|---|---|---|---|---|

### 4. Instruções candidatas à remoção

| Trecho | Arquivo | Motivo | Risco | Como testar |
|---|---|---|---|---|

### 5. Redundâncias e conflitos

Liste todas as duplicações e contradições importantes.

### 6. Skills

| Skill | Função atual | Diagnóstico | Recomendação | Redução estimada |
|---|---|---|---|---|

### 7. `CLAUDE.md` proposto

Forneça a versão minimalista completa.

### 8. Skills propostas

Forneça as versões minimalistas completas.

### 9. Plano de testes de ablação

Mostre as tarefas, versões e métricas.

### 10. Top 10 mudanças de maior impacto

Ordene por:

**impacto esperado / risco**

## REGRAS IMPORTANTES

- Não faça alterações automaticamente.
- Não apague arquivos durante a auditoria.
- Não assuma que “mais curto” é sempre melhor.
- Não preserve uma instrução apenas porque ela parece inteligente.
- Não remova contexto específico que o modelo não conseguiria inferir.
- Prefira critérios de resultado e verificação a sequências rígidas.
- Prefira evidência observada a suposição.
- Quando estiver incerto, use `TEST`.
- Explique cada remoção relevante.
- O objetivo não é maximizar a redução.

O objetivo é maximizar:

**qualidade + autonomia + verificabilidade ÷ complexidade**