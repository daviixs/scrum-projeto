# Design de Produto (Base de Decisão)

Projeto: Bot WhatsApp para Gestão de Tarefas (equipes de tecnologia)  
Data: 2026-02-13  
Status: Validado em brainstorming (Sessões 1-5)

---

## Contexto consolidado do histórico

- Foco atual: validação de produto com MVP bem estruturado para uso básico.
- Público inicial: 2-3 equipes pequenas de empresas parceiras.
- Métrica primária de sucesso do MVP: `% de tarefas atualizadas no prazo via WhatsApp`.
- Regra operacional definida: 1 atualização obrigatória por dev por dia útil.
- Lembretes: 2 vezes por dia útil.
- Fonte oficial das tarefas: dashboard web (tech lead cria e atribui tudo).
- Objetivo de negócio implícito: reduzir desperdício de tempo em dailies longas e melhorar controle de execução semanal.

---

## Sessão 1 - Escopo funcional do MVP

### Insights principais

- O MVP deve focar em execução diária e consistência de atualização, não em amplitude de features.
- Para validação com clientes parceiros, previsibilidade operacional é mais importante que "efeito wow".
- Centralizar criação e atribuição no dashboard reduz ambiguidade e risco de inconsistência.
- WhatsApp deve operar como interface de update rápido, com baixa fricção.
- O núcleo de valor para o tech lead é visibilidade confiável de andamento por pessoa e por tarefa.

### Hipóteses a validar

- Devs aceitam rotina de 1 update por dia no WhatsApp sem sensação de vigilância excessiva?
- Dois lembretes por dia atingem boa taxa de atualização sem gerar fadiga?
- O lead realmente usa o painel para reduzir incerteza de status e encurtar alinhamentos?
- A regra de compliance diária é suficiente para criar disciplina operacional?

### Recomendações práticas (agora vs depois)

- Fazer agora (MVP):
- Dashboard para tech lead:
- cadastrar equipe e horário de trabalho;
- criar tarefas da semana com título, descrição, responsável, prioridade e prazo;
- configurar dois horários de lembrete por dia útil;
- visualizar compliance diário/semanal por dev e equipe.
- WhatsApp para dev:
- receber lembrete com tarefas abertas;
- atualizar progresso com fluxo curto (`%`, bloqueio opcional);
- comandos mínimos: `/hoje`, `/atualizar`, `/ajuda`.
- Regra de compliance:
- "em dia" = pelo menos 1 update válido por dia útil.
- Fazer depois:
- subtarefas/checklists aninhadas;
- dependências e desbloqueio avançado;
- colaboração completa (handoff/menções/comentários ricos).

---

## Sessão 2 - Arquitetura e fluxo de dados (MVP)

### Insights principais

- Separar API, worker de notificação e webhook de entrada reduz acoplamento e facilita operação.
- Idempotência no webhook é obrigatória para evitar duplicação de updates.
- Logs de entrega e erro precisam existir desde o MVP para operar com segurança.
- A plataforma deve priorizar robustez de fluxo sobre sofisticação de feature nesta fase.

### Hipóteses a validar

- Horários fixos de lembrete por equipe cobrem a rotina real dos pilotos?
- Fluxo guiado no WhatsApp reduz erro de entrada em comparação a texto livre?
- Atualização quase em tempo real no dashboard aumenta confiança e ação do lead?

### Recomendações práticas (agora vs depois)

- Fazer agora (arquitetura mínima):
- `Dashboard Web` (tech lead);
- `API Backend` (regras de negócio e autenticação);
- `Worker de Notificação` (agendamento e disparo);
- `Webhook WhatsApp` (recebimento e processamento de respostas);
- `Banco relacional` (PostgreSQL).
- Fluxo principal:
1. Lead cria tarefas e atribuições no dashboard.
2. Backend agenda lembretes por equipe conforme política definida.
3. Worker envia mensagem com contexto de tarefas pendentes.
4. Dev responde no WhatsApp com update.
5. Webhook valida, registra e evita duplicidade.
6. Dashboard reflete compliance e situação das tarefas.
- Entidades mínimas:
- `Team`, `User (lead/dev)`, `Task`, `TaskUpdate`, `ReminderPolicy`, `MessageLog`.
- Confiabilidade:
- retry com backoff para mensagens críticas;
- trilha de auditoria de envio e recebimento.
- Fazer depois:
- event bus dedicado;
- multi-tenant avançado;
- regras complexas de orquestração.

---

## Sessão 3 - UX conversacional, anti-fatigue e edge cases

### Insights principais

- O update precisa acontecer em menos de 30 segundos para adesão consistente.
- Notification fatigue é um risco central; sem política de volume o produto perde engajamento.
- Edge cases operacionais devem ser tratados no MVP para evitar quebra de confiança.

### Hipóteses a validar

- Opções rápidas de percentual (`0/25/50/75/100`) reduzem abandono de fluxo?
- Supressão do segundo lembrete quando já houve update melhora percepção?
- A experiência no WhatsApp é vista como prática e não intrusiva?

### Recomendações práticas (agora vs depois)

- Fazer agora (fluxo ideal <30s):
1. Bot envia lembrete com CTA de atualização.
2. Dev escolhe a tarefa pendente na lista.
3. Dev informa `%` (rápido ou digitado).
4. Bot pergunta se há bloqueio (`Sim/Não`).
5. Se houver bloqueio, captura motivo curto.
6. Bot confirma registro.
- Anti-fatigue:
- no máximo 2 lembretes por dia útil por dev;
- supressão inteligente se dev já atualizou;
- respeito estrito ao horário de trabalho;
- mensagens curtas e contextuais.
- Escalonamento:
- alerta para lead apenas em ausência recorrente (ex.: 2 dias sem update).
- Edge cases MVP:
- férias/folga (`ausente` suspende cobrança);
- reabertura de tarefa com histórico preservado;
- múltiplas tarefas (priorizar top 3 por urgência/prazo);
- resposta ambígua (fallback guiado);
- falha de entrega (retry + sinalização no dashboard).
- Fazer depois:
- entendimento avançado de texto livre;
- recomendação inteligente de próxima tarefa por IA.

---

## Sessão 4 - Priorização, roadmap e modelo de valor

### Insights principais

- O roadmap deve começar em disciplina operacional, depois expandir para diferenciação.
- Em validação B2B com poucos clientes, simplicidade comercial e clareza de ROI são essenciais.
- A métrica primária (`A`) deve guiar decisões de produto nas primeiras semanas.

### Hipóteses a validar

- Empresas pagariam por ganho de previsibilidade e redução de custo de alinhamento?
- Leads adotam o painel como fonte central de status diário?
- Melhora de compliance de update se traduz em ganho prático de execução?

### Recomendações práticas (agora vs depois)

- Fase 1 - MVP (4-6 semanas):
- dashboard com criação/atribuição;
- duas notificações por dia útil;
- fluxo rápido de update no WhatsApp;
- painel de compliance.
- Fase 2 - V1 (6-10 semanas):
- alerta de deadline (24h);
- priorização visual e filtros;
- resumo diário para lead;
- escalonamento básico de blockers.
- Fase 3 - V2:
- sugestão de próxima tarefa;
- retrospectiva semanal acionável;
- integrações Jira/GitHub (inicialmente sync simples).
- Must have (MVP):
- cadastro/atribuição no dashboard;
- política de lembrete;
- update WhatsApp;
- compliance e logs.
- Nice to have:
- pomodoro;
- colaboração avançada;
- retro completa;
- IA generativa ampla.
- Modelo de negócio inicial:
- pagador principal: empresa/equipe;
- pricing inicial sugerido: por equipe/faixa de usuários (mais simples para piloto).

---

## Sessão 5 - Riscos técnicos e operacionais (WhatsApp) + mitigação

### Insights principais

- Operação via WhatsApp depende de política rígida de envio e relevância de mensagem.
- Risco de ban/bloqueio aumenta com comportamento de spam e baixo engajamento.
- Entrega crítica requer observabilidade e retry, não apenas tentativa única.
- Governança de consentimento (opt-in) e histórico de interação é requisito operacional.

### Hipóteses a validar

- 2 lembretes/dia mantêm taxa de resposta alta sem degradar experiência?
- Mensagens curtas com CTA performam melhor que texto longo?
- Retry + monitoramento evitam perda de atualização em casos de falha?
- A política de opt-in/opt-out está clara e auditável por usuário?

### Recomendações práticas (agora vs depois)

- Fazer agora:
- política conservadora de envio:
- máximo de 2 lembretes por dia útil;
- supressão quando já houve update;
- respeito a timezone e horário da equipe.
- templates utilitários curtos e contextuais;
- `MessageLog` completo por tentativa (enviado, entregue, falha, erro permanente);
- retry com backoff e limite de tentativas;
- painel operacional com:
- falhas por equipe/dev;
- devs sem update no dia;
- tendência semanal de engajamento;
- trilha de consentimento e descadastro.
- Fazer depois:
- fallback de canal secundário (email/Slack) para eventos críticos;
- score interno de risco de ban;
- automações de governança para escala.

---

## Riscos e conflitos de escopo (resumo crítico)

- Risco de overbuild: incluir colaboração complexa cedo pode atrasar validação.
- Conflito potencial: excesso de notificações para elevar throughput pode reduzir adoção.
- Redundância potencial: dailies automatizadas completas no MVP podem competir com meta de simplicidade.
- Mitigação: manter foco estrito em update diário + visibilidade de compliance até comprovar valor.

---

## Critérios de sucesso da validação (go/no-go)

- Primário (decisão):
- `% de tarefas atualizadas no prazo via WhatsApp`.
- Secundários:
- redução de tempo em daily/status;
- variação de throughput semanal;
- percepção qualitativa de utilidade por dev e lead.

Sugestão de regra de decisão:
- Go: melhora consistente da métrica primária durante 4-6 semanas + feedback positivo de leads.
- No-go/pivot: baixa adesão recorrente ao update diário mesmo com ajuste de UX e política de lembrete.

---

## Decisões pendentes para próxima etapa

- Definir baseline das equipes piloto antes do início.
- Definir meta numérica mínima para a métrica primária.
- Definir política formal de ausência/férias e exceções.
- Definir rotina de revisão semanal com clientes piloto.

