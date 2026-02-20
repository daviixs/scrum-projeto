# Produto: Updates de Tarefas via WhatsApp

## 🎯 Contexto Inicial

Público inicial: 2–3 equipes parceiras  
Objetivo: validação de produto  
Métrica primária: % de tarefas atualizadas no prazo via WhatsApp  
Operação diária:

- 1 update obrigatório por dev por dia útil
- 2 lembretes por dia
  Fonte oficial de tarefas: apenas dashboard web

---

# 🧩 Estratégia de MVP

## MVP Ultra-Enxuto (recomendado)

Inclui:

- Criação e atribuição de tarefas no dashboard
- Notificação 2x/dia via WhatsApp
- Update simples no chat (% / bloqueado / concluído)
- Painel de compliance para o lead

Prós:

- Validação rápida
- Menor risco técnico
- Menor ruído com cliente piloto

Contras:

- Menor “wow factor” inicial

Decisão: executar por 4–6 semanas antes de expandir.

---

# 🏗️ Arquitetura e Fluxo de Dados (MVP)

## Arquitetura

- Dashboard Web (lead)
  CRUD de tarefas, equipes e regras de lembrete

- API Backend
  Autenticação, regras de negócio, agendamento e métricas

- Worker de Notificação
  Dispara lembretes 2x/dia por equipe

- Webhook WhatsApp
  Recebe respostas dos devs e grava updates

- Banco relacional (ex: PostgreSQL)
  Equipes, usuários, tarefas, updates, logs

---

## Fluxo Principal

1. Lead cria tarefas e atribui responsáveis
2. Backend agenda lembretes
3. Worker envia mensagem no WhatsApp
4. Dev responde via fluxo guiado
5. Webhook valida e registra percentual/status
6. Dashboard atualiza compliance quase em tempo real

---

## Modelo de Dados Mínimo

Team
User (role: lead/dev)
Task (assignee, priority, due_date, status)
TaskUpdate (task_id, user_id, %, blocked_reason, created_at)
ReminderPolicy (team_id, time_1, time_2, timezone)
MessageLog (status_entrega, falha, tentativa)

---

## Decisões Técnicas

- Fonte única de verdade = dashboard
- WhatsApp apenas para update rápido
- Idempotência no webhook
- Retry com backoff
- Logs detalhados de envio

---

# 💬 UX Conversacional

## Objetivo

Update em menos de 30 segundos

## Fluxo

1. Bot envia lembrete
2. Dev escolhe tarefa
3. Dev informa percentual (0/25/50/75/100 ou manual)
4. Bot pergunta se há bloqueio
5. Se sim, coleta motivo
6. Confirmação final e salva

---

## Estratégia Anti-Fatigue

- Máximo 2 lembretes/dia útil
- Skip se já atualizou
- Respeitar horário da equipe
- Escalonamento só após ausência recorrente

---

## Edge Cases

- Dev em férias → suspende cobrança
- Mudança de prioridade → reflete no próximo lembrete
- Tarefa reaberta → histórico preservado
- Muitas tarefas → sugerir top 3
- Resposta ambígua → fallback guiado
- Falha de entrega → retry + log + alerta

---

## Critérios de Validação

- Tempo mediano ≤ 30s
- Abandono < 15%
- Feedback qualitativo semanal

---

# 🚀 Roadmap

## Fase 1 — MVP (4–6 semanas)

- Criar/atribuir tarefas
- Configurar 2 lembretes/dia
- Update rápido via WhatsApp
- Painel de compliance
- Logs de entrega

Métrica primária:

- % tarefas atualizadas no prazo

Métricas secundárias:

- Tempo de daily
- Throughput semanal

---

## Fase 2 — V1

- Alertas de deadline 24h
- Filtros por urgência
- Resumo diário automático
- Escalonamento simples de blocker

---

## Fase 3 — V2

- Sugestão da próxima tarefa
- Retrospectiva semanal acionável
- Integração Jira/GitHub (sync inicial unidirecional)
- Benchmark de equipe

---

# 📌 Must Have

- Dashboard como fonte oficial
- Lembretes 2x/dia
- Update rápido
- Painel de compliance
- Logs de entrega

# ✨ Nice to Have

- Pomodoro
- Subtarefas
- Handoff
- IA generativa ampla
- Menções

---

# 💰 Modelo de Negócio

- Cliente pagante: empresa/equipe
- Pricing: por equipe com faixa de usuários
- Modelo simples para piloto

## Prova de Valor

- Aumento da taxa de updates em dia
- Redução do tempo de daily
- Redução de tarefas sem status claro

---

# 🧠 Síntese

O produto não é sobre tarefas.
É sobre disciplina operacional mensurável.

Resolve:

- Falta de clareza diária
- Dailies longas
- Tarefas sem atualização
- Falta de accountability leve

Com:

- Fricção mínima
- Canal já usado (WhatsApp)
- Métrica clara de valor (compliance)
