# Contexto de domínio

Vocabulário compartilhado do projeto. Só termos e significados — não é spec, plano nem decisão de implementação. A UI é pt-BR e o código é inglês; o termo canônico é o de UI, com o identificador do código entre parênteses.

## Currículo

- **Currículo** (`Resume`): o texto do perfil profissional do candidato. Não há histórico: um novo envio substitui a versão anterior.
- **Versão de currículo**: o currículo em um idioma específico. Hoje existem duas: PT-BR e EN. Cada versão tem texto próprio, editável.
- **Currículo canônico**: a versão PT-BR. É a fonte de verdade do perfil — comparação de mercado, competências estratégicas e análise de mercado leem sempre ela.
- **Versão EN**: versão em inglês, gerada pelo sistema a partir do PT-BR por **tradução de IA**. Editável à mão. Não é fonte de verdade: existe para candidatura em inglês.
- **Tradução do currículo** (`ResumeTranslation`): operação de IA que gera ou regenera a versão EN a partir do PT-BR. Substitui o texto EN inteiro.
- **Drift da versão EN**: estado da versão EN quando o PT-BR mudou depois da última tradução. Não bloqueia uso; é sinalizado e pode ser regenerado.

## Descoberta e decisão

- **Execução de descoberta** (`DiscoveryRun`): registro de uma rodada de descoberta do Hermes, com fonte, início e fim, contagens (capturadas, triadas, pontuadas, adaptadas), status e erro.
- **Score de vaga** (`JobScore`): nota de 0 a 100 calculada sobre os requisitos da vaga, `round(100 * cobertura * confiança)`, com breakdown por requisito e blockers. Uma linha por vaga; reprocessar substitui a anterior.

## Vaga e candidatura

- **Vaga** (`JobPosting`): anúncio de emprego capturado e triado. Só uma vaga **confirmada** permite adequar currículo.
- **Confirmação da vaga** (`confirmed_by`): origem da confirmação — `user` (revisão humana) ou `agent_policy` (política de score). Só existe em vaga confirmada.
- **Vaga auto-confirmada** (`confirmed_by = agent_policy`): vaga que atingiu o limite de score e foi confirmada pelo agente, sem revisão humana. O status continua `confirmed`.
- **Adequação de currículo** (`ResumeAdaptation`): versão do currículo adaptada a uma vaga. Existe **uma por (vaga, idioma)** — no máximo duas: PT-BR e EN. Regenerar substitui a daquele idioma, sem histórico.
- **Adaptação PT-BR** / **Adaptação EN**: as duas adaptações da mesma vaga, uma por idioma. A UI mostra uma por vez, escolhida pelo seletor de idioma.
- **Candidatura** (`Application`): o acompanhamento da candidatura a uma vaga, iniciado sob demanda. O idioma não é atributo da candidatura — quem carrega idioma é a adequação. A candidatura é o agregado de acompanhamento; a execução fica na **tentativa de candidatura**.
- **Tentativa de candidatura** (`ApplicationAttempt`): registro de execução do preenchimento e envio de uma candidatura, nos modos `manual`, `shadow` ou `automatic`. Guarda o score do momento, a adequação usada, o plano de campos e as evidências; não substitui a **candidatura**.
- **Modo sombra** (`shadow`): modo de tentativa que preenche o formulário e para antes do submit, guardando as evidências. Não envia candidatura.
- **Movimentação** (`ApplicationEvent`): registro append-only do histórico da candidatura (status, observação, contato).
- **Forma de candidatura** (`ApplicationMethod`): canal pelo qual se candidatar (LinkedIn, e-mail, URL, telefone…).
- **Otimização ATS** (`ats_prompt`): bloco de palavras-chave sugerido para vencer triagens automáticas, **por adaptação** — logo, por idioma. O bloco sai no idioma da adaptação.
- **Simulação ATS** (`ats_score`): nota estimada de aderência ao ATS, **por adaptação**; a justificativa exibida permanece em pt-BR (UI).
- **Competência estratégica** (`ProfessionalCompetency`): conceito do perfil classificado contra o mercado; derivado do currículo canônico.
- **Comparação de mercado**: leitura do currículo canônico contra a frequência de conceitos das vagas analisadas.
