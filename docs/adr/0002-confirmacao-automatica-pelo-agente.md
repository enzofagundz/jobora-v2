# 2. Confirmação automática de vaga pelo agente

- **Status:** aceito
- **Data:** 2026-11-01
- **Glossário:** GLOSSARY.md

## Contexto

Hoje `job_postings.status = confirmed` significa "vaga revisada e confirmada pelo usuário". A revisão acontece em `JobReviewService::confirm()` (chamado pela página de revisão e pela tool `confirm_job_posting`), que persiste os itens revisados, limpa a sugestão da IA e grava o status.

Vários consumidores leem `status = confirmed` como sinal de vaga válida, não de autoria humana: análises de mercado (`MarketAnalysisService`), estatísticas do painel, detecção de staleness das competências (`Staleness`), recursos MCP de mercado e o futuro digest diário. Nenhum deles pergunta quem confirmou.

O spec do V2 (#1) muda o fluxo: a descoberta calcula um score por vaga e, quando o score atinge o limite do usuário, a vaga passa a `confirmed` sem revisão humana, com a adequação de currículo enfileirada. O mesmo spec exige registrar a origem da confirmação em `job_postings.confirmed_by` (`user` ou `agent_policy`) e manter o status `confirmed`. Isso contradiz a semântica atual do glossário, que define vaga confirmada como revisada pelo usuário.

## Decisão

1. **`confirmed` continua sendo o único estado de vaga válida.** Não criar status `auto_confirmed` nem valor extra no enum. A vaga confirmada é a que permite adequar currículo, entra no digest e compõe as análises de mercado, independentemente da origem.
2. **`job_postings.confirmed_by` registra a origem:** `user` (revisão humana) ou `agent_policy` (política de score). A coluna fica nula enquanto o status não é `confirmed`; linhas existentes são preenchidas com `user` na migration.
3. **A auto-confirmação exige score calculado com sucesso e `score >= auto_adapt_threshold` do usuário** (default 75, por usuário). Score abaixo do limite, falha de extração ou falha de score mantêm a vaga em `needs_review`, sem adequação.
4. **Requisito legal ausente não impede a confirmação.** O blocker não entra na cobertura do score e sinaliza inelegibilidade para candidatura; ele bloqueia auto-envio, não a confirmação nem a adequação. O digest mostra o blocker.
5. **Captura por link explícito confirma com `confirmed_by = user`, ignorando o score.** Preferência do usuário não bloqueia; bloqueio legal e duplicidade continuam valendo.
6. **`confirmed` passa a significar "vaga válida", e não "revisada pelo usuário".** A revisão humana fica representada por `confirmed_by = user`; a UI mostra o selo "confirmada pelo agente" para `agent_policy`. Se o usuário revisar uma vaga `agent_policy`, `confirmed_by` passa a `user`, porque a última autoridade vence.
7. **O caminho de política grava o mesmo estado que a revisão humana.** Ao confirmar, persiste os itens e métodos extraídos, limpa `suggested_data` e define `status = confirmed`; a diferença é apenas `confirmed_by`.

## Consequências

- A migration de `job_postings` adiciona `confirmed_by` (string, nulável) com backfill `user` para as vagas já confirmadas; a validação dos dois valores fica no enum de aplicação, não em constraint de banco (SQLite e PostgreSQL).
- `JobReviewService::confirm()` grava `confirmed_by = user`. O caminho de auto-confirmação grava `agent_policy` e reutiliza a mesma persistência de itens, sem edição humana.
- As queries que filtram `status = confirmed` continuam válidas e passam a incluir vagas auto-confirmadas. Isso é intencional: mercado, estatísticas, staleness e digest devem enxergar as duas origens.
- A tool `confirm_job_posting`, as descrições do servidor MCP, a interface e o selo do painel precisam refletir a origem. As descrições que dizem "revisada pelo usuário" são ajustadas na fatia que implementar a auto-confirmação.
- O digest diário filtra por status e preferências, não por origem; uma vaga `agent_policy` pode aparecer nele.
- Testes futuros cobrem: auto-confirmação no limite, permanência em `needs_review` abaixo do limite, falha de score sem confirmação, link explícito com `user`, blocker legal sem bloquear confirmação e promoção para `user` após revisão humana.

## Alternativas descartadas

- **Criar status `auto_confirmed`.** Obrigaria todo consumidor de `confirmed` a checar dois valores e duplicaria filtros de mercado, digest e painel sem ganho semântico.
- **Booleano `reviewed_by_user`.** Não cobre a origem `agent_policy` como valor explícito e não escala se surgirem outras origens (importação, seed, migração).
- **Manter a confirmação humana obrigatória.** Contraria o objetivo do V2 de não revisar manualmente cada vaga acima do limite.
- **Confiar no score sem persistir a origem.** O selo "confirmada pelo agente" e a auditoria da decisão ficariam impossíveis.
- **Bloquear a confirmação quando houver requisito legal ausente.** Confundiria elegibilidade de candidatura com validade da vaga e esvaziaria o digest de vagas que ainda valem análise.
