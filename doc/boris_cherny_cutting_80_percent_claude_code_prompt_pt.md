# Boris Cherny sobre cortar 80% do prompt do Claude Code



> Este guia resume os principais conceitos, modelos e etapas apresentados no vídeo.

## Por que o Claude Code apaga o próprio prompt de sistema

Boris Cherny criou o Claude Code na Anthropic. Nesta palestra da Y Combinator, gravada um dia após o lançamento do Opus 5, ele explicou que o harness do Claude Code nunca está terminado. A equipe está constantemente adicionando e removendo instruções, e cada novo lançamento de modelo provoca uma grande reescrita.

### O harness muda a cada modelo

O motivo é simples. Cada modelo é diferente, então uma instrução escrita para um modelo de três meses atrás pode não funcionar no próximo.

- A equipe altera o prompt de sistema a cada lançamento de modelo.
- Eles mudam o conjunto de ferramentas disponíveis para o modelo.
- Eles alteram os prompts associados a cada ferramenta individual.
- Eles frequentemente descontinuam ferramentas e apagam completamente partes do código do harness.

### O que mudou com o Opus 5

Uma grande parte do antigo prompt de sistema existia para corrigir comportamentos que o modelo deveria saber executar, mas não sabia. O Opus 5 faz essas coisas sozinho, então essas correções se tornaram peso morto e a equipe removeu mais de 80% do prompt de sistema.

- O Opus 5 marcou 30% no Arc AGI 3. As melhores pontuações anteriores ficavam entre poucos pontos percentuais e a casa dos 10%.
- Combinado com o modo automático, ele pode funcionar por dias, semanas ou meses sem parar.
- Ele não precisa mais de comandos de scaffolding para continuar, porque entende que precisa concluir a tarefa.
- Boris também relatou que o modelo aparentemente não é mais vulnerável a prompt injection.

## As três camadas por trás da resistência a prompt injection

Prompt injection ocorre quando o modelo lê uma instrução de uma fonte não confiável, como uma página da web, e a segue. Boris descreveu três camadas que, juntas, tornam isso muito difícil de demonstrar.

1. Um modelo bem alinhado, resultado de aproximadamente três anos de pesquisa em alinhamento.
2. Um classificador de prompt injection executado em todo o tráfego, construído com base em trabalhos de interpretabilidade mecanicista que observam quais neurônios são ativados quando ocorre uma injeção.
3. O classificador do modo automático, que adiciona uma terceira verificação independente.

## O que restou no harness

Depois de todas as remoções, Boris disse que quase todo o código restante do Claude Code é voltado para segurança, permissões e análise estática, além de um conjunto de código de interface. A maior parte do restante já foi descontinuada.

## O método de ablação: como apagar e reconstruir

Ablação é um termo de pesquisa. Você remove algo para medir seu impacto. Aplicado a prompts, significa apagar todo o prompt de sistema e depois trazê-lo de volta linha por linha para descobrir o que cada linha realmente está fazendo. Boris descreveu isso como uma avaliação em que você remove coisas para descobrir o impacto.

### O conselho para usuários comuns

Boris deu uma recomendação direta para pessoas que usam Claude Code, mas não constroem produtos agênticos.

Aproximadamente a cada seis meses, e especialmente em um grande lançamento de modelo, apague sua configuração e veja o que acontece.

- Apague seu arquivo `CLAUDE.md`.
- Apague suas skills.
- Apague seus hooks.
- Depois use a ferramenta e observe o que o modelo faz sem eles.

### A reconstrução em quatro etapas

Apagar é apenas a primeira metade. A reconstrução é o que determina se a nova configuração será menor ou apenas diferente.

1. Apague a configuração.
2. Use a ferramenta em trabalho real, não em testes hipotéticos. Execute o produto de verdade ou a base de código real.
3. Observe onde ela se sai bem e onde tropeça.
4. Adicione uma instrução novamente somente depois de ver o modelo falhar repetidamente na mesma coisa.

Existem duas razões para essa disciplina na etapa quatro: você é um mau previsor de quais instruções o modelo realmente precisa, e cada linha mantida é relida pelo modelo em cada uso.

> **Princípio central:** não tente adivinhar o que o modelo precisa. Faça-o provar essa necessidade falhando na mesma coisa mais de uma vez.

## Formas de testar isso por conta própria

Boris citou dois mecanismos para executar sua própria ablação.

- Passe uma flag de prompt de sistema ao iniciar o Claude Code para substituir o prompt de sistema por qualquer outro que você quiser.
- Defina a variável de ambiente `CLAUDE_CODE_SIMPLE=1`, uma opção pouco documentada que remove todos os prompts de sistema, incluindo os prompts associados às ferramentas.

A Anthropic usa a segunda opção internamente como linha de base para ablação. Boris observou um resultado contraintuitivo: o modelo fica ligeiramente mais inteligente sem os prompts. Os prompts são mantidos mesmo assim, porque fazem o Claude Code se comportar da forma que uma pessoa espera ao utilizá-lo como produto.

## O que acontece com as evals

As evals são mais duráveis que os prompts, mas não permanentemente. Boris disse que uma eval pode sobreviver de uma a três gerações de modelo antes que os modelos a saturem; nesse ponto, ela é descartada e substituída.

- Continue adicionando casos ao seu conjunto de evals conforme encontrar novas falhas.
- Espere aposentar evals quando os modelos atingirem o teto delas.
- Crie novas evals a partir dos pontos em que você realmente observou o modelo tendo dificuldades.

## Pare de especificar demais a tarefa

Boris identificou a especificação excessiva como um dos erros mais comuns que vê, e disse que ela é especialmente frequente entre pessoas que trabalham com engenharia há anos ou décadas.

### O modo de falha

As pessoas escrevem instruções que determinam a sequência exata de movimentos: faça isto, depois isto, depois isto, depois isto.

Essa abordagem funcionava com modelos antigos. Não funciona com os modernos e impede o modelo de encontrar um caminho melhor do que aquele que você imaginou.

Boris relacionou esse hábito à forma como software era construído anteriormente. Você projetava o sistema inteiro de antemão, escrevia uma grande suíte de testes unitários e tratava uma re-arquitetura como um projeto medido em meses ou anos. Modelos não funcionam assim.

### O que escrever em vez disso

A nova forma deve ser mais abstrata e muito mais curta.

- Descreva a tarefa.
- Descreva os guardrails.
- Descreva os critérios de saída, ou seja, como o modelo sabe que terminou.
- Deixe o modelo trabalhar e volte depois para verificar.

Ele também recomendou mirar mais alto do que parece confortável. Dê ao modelo uma tarefa um pouco mais difícil do que você acha que ele consegue executar, porque o teto muda a cada lançamento.

## Pense nisso como algo orgânico, não mecânico

Boris descreveu trabalhar com um modelo como algo mais próximo de conhecer uma criatura viva do que de configurar um sistema. Cada geração se comporta de forma diferente e tem uma personalidade ligeiramente distinta, então você passa tempo aprendendo como ela funciona e depois ajusta o harness ao redor do que aprendeu. Ele disse que o nível adequado das instruções é aproximadamente o mesmo que você usaria ao orientar um colega de trabalho.

## Verificação é o que a maioria das pessoas faz errado

Ao ser perguntado sobre qual habilidade importa agora que “prompt engineering” perdeu força como título profissional, Boris apontou duas coisas: dar ao Claude uma tarefa que pareça um pouco difícil demais e tornar possível para o Claude verificar o próprio trabalho ao longo do processo. Ele chamou a verificação de a coisa mais importante que as pessoas não acertam.

### O exemplo da reescrita em Swift

Boris queria saber como seria o aplicativo desktop do Claude, construído em Electron, se fosse transformado em uma aplicação nativa. Ele executou o experimento pelo Claude Tag, que é o Claude rodando dentro do Slack.

1. Perguntou se ele tinha acesso a um runner de macOS no GitHub. O sistema disse que não, então ele conectou um, dando acesso à capacidade de iniciar uma máquina virtual Mac.
2. Criou um repositório vazio para a reescrita em Swift e perguntou se havia acesso. A resposta foi não, então ele concedeu o acesso.
3. Deu uma única instrução: reescreva o aplicativo Electron em Swift, execute o aplicativo Electron na máquina virtual Mac, tire screenshots, compare pixel por pixel com a versão em Swift e não pare até terminar.

No momento da palestra, a tarefa estava rodando havia mais de duas semanas e ainda não havia terminado. Boris estimou que ela havia criado de milhares a dezenas de milhares de agentes. O Claude também decidiu sozinho criar um canal no Slack e fazer um live blog do progresso, com screenshots a cada poucos minutos.

### Por que esse prompt funcionou

O prompt era curto e não continha nenhuma técnica especial. O que ele tinha era um ciclo completo de verificação: uma forma de produzir a saída, uma forma de observar o alvo, uma forma de comparar os dois e uma condição para parar.

- **Produzir:** reescrever o aplicativo em Swift.
- **Observar:** executar o original em uma máquina virtual e tirar screenshots.
- **Comparar:** verificar pixel por pixel em relação à nova versão.
- **Condição de parada:** não parar até terminar.

> **Regra de verificação:** dê ao modelo uma forma de conferir o próprio resultado da mesma maneira que você faria. Sem isso, ele fica travado. Com isso, pode trabalhar por semanas.

## Não existe um único truque secreto

Perguntaram a Boris o que diferencia o 1% dos melhores usuários de Claude Code. A resposta dele foi: pare de procurar um truque. Ele aconselhou especificamente a não tomar influenciadores de redes sociais como referência sobre o assunto.

O método que descreveu é um ciclo. Dê ao modelo uma tarefa difícil demais. Dê ferramentas para verificar o trabalho. Observe onde ele tem dificuldade. Corrija especificamente aquela dificuldade. Repita.

Para corrigir uma dificuldade, ele citou três opções dependendo da causa.

- Prompt melhor, se a instrução estava pouco clara.
- Uma skill, se o modelo precisa de um procedimento repetível.
- Um servidor MCP, se o modelo está sem contexto que não consegue acessar.

## Product overhang e unhobbling

Esses são dois lados da mesma ideia, e Boris os apresentou como a maior fonte de oportunidade ainda não explorada para builders atualmente.

- **Product overhang:** o modelo já consegue fazer algo valioso, mas não existe produto que permita expressar essa capacidade.
- **Hobbling:** o produto atrapalha ativamente aquilo que o modelo conseguiria fazer.

Unhobbling significa remover o scaffolding que restringe o modelo para que a capacidade existente possa aparecer. Boris enfatizou que isso se refere às capacidades dos modelos atuais, não dos futuros.

### Como o Claude Code nasceu

O próprio Claude Code é o exemplo mais claro. Quando Boris começou a construí-lo, o melhor modelo de código disponível era o Sonnet 3.5. Os produtos de programação daquela época ofereciam autocomplete de uma linha, algum autocomplete de múltiplas linhas e chat somente leitura sobre uma base de código.

A avaliação dele era que o modelo já conseguia escrever um arquivo inteiro de uma vez, e nada no mercado permitia isso. A resposta de design foi remover scaffolding e entregar ao modelo o harness mais simples possível, com acesso completo ao terminal.

### A reescrita do Bun

O Claude Code roda sobre Bun, um runtime JavaScript open source e uma alternativa mais rápida ao Node.js. O Bun foi escrito em Zig, uma linguagem de sistemas de baixo nível que exige gerenciamento manual de memória, o que torna vazamentos de memória fáceis de introduzir.

- Inicialmente, a equipe do Bun usava Claude para fazer fuzzing na base de código e provocar vazamentos de memória, encontrando um caso por vez.
- Jared, da equipe do Bun, continuou propondo um teste mais difícil a cada nova geração de modelo: reescrever tudo.
- A partir do Fable, o modelo conseguiu. Ele definiu o sucesso usando as suítes de testes existentes do Bun e do Node.js.
- Um único prompt, executado como um workflow dinâmico, reescreveu a base de código de Zig para Rust ao longo de 11 dias.
- A base de código tem mais de 100.000 linhas. Boris disse que o mesmo trabalho teria levado bem mais de um ano para engenheiros qualificados.
- Não foi totalmente one-shot. Houve direcionamento ao longo do caminho, mas modelos anteriores não conseguiam fazer isso nem com orientação.
- O resultado está em produção e é a base sobre a qual o Claude Code roda hoje.

### Ensinando um modelo a desenhar

Um segundo exemplo surgiu de experimentação interna, não de um problema de negócio. Alguém na Anthropic descobriu que, se você der ao Opus 5 acesso ao OpenCV, uma biblioteca de visão computacional, e pedir para ele desenhar, ele produz retratos, animais e paisagens convincentes.

O modelo nunca foi treinado para desenhar. Boris chamou isso de **elicitation gap**: a capacidade existe, e basta perguntar da forma correta para acessá-la. A hipótese dele é que existem dezenas ou centenas de descobertas comparáveis ainda não exploradas.

> **Método prático:** mantenha uma lista de problemas em que modelos falharam no passado e teste-os novamente a cada novo lançamento. A falha do modelo anterior não diz nada sobre o atual.

## Orquestrando milhares de agentes

Boris descreveu dois mecanismos distintos para colocar grandes quantidades de capacidade computacional por trás de uma única intenção, e a diferença entre eles importa.

### Workflows dinâmicos

Um workflow dinâmico pega uma grande tarefa e a divide em etapas coordenadas executadas por muitos agentes. Boris disse que, para ativá-lo, basta dizer “use um workflow”, e o Claude cuida do restante.

- O workflow roda em um sandbox, usando uma máquina virtual iniciada dentro do runtime Bun.
- Ele não executa apenas um agente ou dez agentes em paralelo. Ele orquestra em etapas.
- Uma primeira onda de agentes faz uma análise inicial do trabalho.
- Uma segunda etapa verifica ou resume o que a primeira onda produziu.
- Uma terceira etapa distribui novamente o trabalho, e assim por diante, até a conclusão.

Bons casos de uso incluem reescrever uma base de código, executar análises profundas sobre dados complexos e construir uma funcionalidade complexa que atravesse várias etapas e dezenas de pull requests.

### Uma álgebra para agentes

Boris vem de uma formação em programação funcional e projetou o sistema em torno de composição. Existe uma forma de executar agentes em sequência e outra de executá-los em paralelo, e o Claude tem ferramentas para combinar essas abordagens dentro do sandbox e usar tokens de forma eficiente em trabalhos muito complexos.

### Uma nova forma de test-time compute

Historicamente, a capacidade dos modelos aumentou com o tamanho da rede neural, a quantidade de dados de treinamento e o número de FLOPs usados no treinamento. Mais recentemente, surgiu o conceito de test-time compute, que se refere à quantidade de tokens que o modelo gera enquanto trabalha.

Boris apresentou workflows dinâmicos como uma nova forma de orquestrar test-time compute e uma maneira de aumentar drasticamente quanto desse compute pode ser aplicado a uma única tarefa difícil. Ele observou que isso ainda foi pouco discutido publicamente.

## Loops e routines

Loops e routines resolvem um problema diferente dos workflows dinâmicos. Eles tratam de uma tarefa repetitiva executada várias vezes em um cronograma, em vez de uma grande tarefa dividida em partes.

- Um loop é essencialmente um cron job executando Claude localmente.
- Uma routine é a mesma coisa rodando na nuvem, para que você possa fechar o notebook.
- Cada execução não compartilha contexto com a anterior, embora possa compartilhar memória.
- Elas podem ser agendadas a cada cinco minutos, a cada hora ou diariamente.

## Bases de código que fazem a própria manutenção

A Anthropic agora usa Claude para manter os próprios produtos. A equipe configurou um canal no Slack e começou a executar routines sobre o Claude Code CLI, o aplicativo iOS, o Android e o desktop.

### As routines que eles executam

Boris enfatizou que cada uma dessas routines é composta por um único prompt, frequentemente com apenas uma frase. O modelo descobre sozinho os detalhes da implementação.

- **Limpar código morto.** Roda diariamente, usa análise estática e dinâmica e abre um pull request removendo o que encontra. Boris observou que ninguém pediu explicitamente para usar esses métodos de análise; o modelo descobriu isso sozinho.
- **Publicar experimentos concluídos.** Encontra experimentos que já foram liberados para 100% dos usuários, remove o scaffolding do experimento da base de código e publica o resultado.
- **Escrever testes ausentes** para áreas da base de código com baixa cobertura.
- **Apagar testes inúteis**, incluindo testes de baixo valor adicionados por modelos antigos ou por pessoas ao longo do tempo.
- **Polícia de abstrações.** Encontra abstrações quase duplicadas que divergiram ao longo do tempo em uma grande base de código e as unifica novamente em uma só.

### A escala disso

Hoje existem aproximadamente 20 a 30 dessas routines rodando diariamente em todas as bases de código, lançando centenas e às vezes milhares de agentes por dia.

Boris estimou que isso cobre trabalho que anteriormente exigiria dezenas ou centenas de engenheiros. Ele disse que ainda não chegaram totalmente lá, mas estão no caminho para automatizar por completo a manutenção dos aplicativos, liberando os engenheiros para criar novos produtos e conversar com usuários.

> **Padrão que vale copiar:** as routines de maior alavancagem são tarefas de manutenção óbvias, repetitivas e fáceis de deixar para depois. Uma frase cada, executadas diariamente, sem necessidade de gatilho humano.

## A mentalidade que diferencia os melhores usuários

Boris voltou repetidamente a um mesmo tema: trabalhar com modelos se tornou uma ciência empírica, não teórica.

### Esqueça seus pressupostos

- Deixe de lado o que você aprendeu sobre o comportamento de modelos anteriores.
- Deixe de lado suposições da teoria da computação sobre o que deveria ser difícil.
- Teste a tarefa, observe onde o modelo tem dificuldade e ajuste com base no que você observou.
- Continue disposto a testar novamente ideias que falharam no passado, porque o motivo da falha pode ter desaparecido.

Ele descreveu overengineering como um modo recorrente de falha, e desaprender esse hábito como uma verdadeira jornada para builders experientes.

## Onde programação ainda não está resolvida

Boris já disse publicamente que programação está resolvida, mas fez uma ressalva: está resolvida para o tipo de programação que ele faz, não para todo mundo. Ele citou áreas em que o Claude ainda tem dificuldades.

- Bases de código de sistemas muito profundas.
- Sistemas distribuídos.
- Verificação visual detalhada, como detectar algo deslocado por um pixel. O Opus 5 representou um grande avanço em visão e uso de computador, mas ainda não é perfeito.

Quando ele fez uma enquete com a sala, um número relevante de fundadores disse que 100% do código deles já é escrito por agentes, e uma quantidade semelhante afirmou estar acima de 50%.

## O que ainda vale aprender manualmente

Boris aprendeu a programar em uma calculadora TI-83 no ensino fundamental, escrevendo programas em BASIC para resolver problemas de álgebra em provas, e depois aprendeu assembly sozinho quando os problemas de cálculo ficaram mais difíceis. Ele chegou a escrever um guia online sobre isso, que ainda está disponível. O ponto da história é que ele sempre aprendeu em função de um problema concreto.

O conselho dele para estudantes é aprender ciência da computação, mas dar o mesmo peso às habilidades ao redor dela, porque são essas habilidades que tornam a capacidade técnica valiosa.

- Aplicar conhecimento a um problema real, em vez de estudá-lo isoladamente.
- Construir produtos e startups.
- Desenvolver senso de design.
- Desenvolver senso de negócios.
- Aprender ciência de dados.
- Aprender a conversar com usuários.

## Principais aprendizados

1. **Configuração envelhece.** Cada instrução que você escreve é uma correção para a fraqueza de um modelo específico, e se torna peso morto quando o próximo modelo deixa de ter aquela fraqueza.
2. **Apague periodicamente.** Aproximadamente a cada seis meses, e em grandes lançamentos de modelo, limpe seu `CLAUDE.md`, suas skills e seus hooks, e veja o que o modelo faz sem eles.
3. **Reconstrua com base em evidências, não em previsão.** Adicione uma instrução novamente somente depois que o modelo tiver falhado na mesma coisa mais de uma vez. Somos ruins em adivinhar de quais linhas ele realmente precisa.
4. **Especificação excessiva é o erro comum.** Dê a tarefa, os guardrails e os critérios de saída; depois deixe o modelo trabalhar. Não roteirize cada etapa.
5. **Verificação é o elemento de maior alavancagem que você pode adicionar.** Um prompt curto, com uma forma real de conferir o próprio trabalho, supera um prompt longo sem mecanismo de verificação.
6. **Mire acima do teto.** Dê ao modelo uma tarefa um pouco mais difícil do que você acha que ele consegue executar e repita antigos fracassos a cada nova versão.
7. **Product overhang é onde está a oportunidade.** Os modelos já conseguem fazer coisas valiosas que nenhum produto permite expressar, e o próprio Claude Code nasceu da identificação exatamente desse tipo de oportunidade.
8. **Automatize manutenção com routines de uma frase.** A Anthropic executa de 20 a 30 routines diárias que limpam código morto, unificam abstrações duplicadas e gerenciam testes em todos os seus aplicativos.

---


