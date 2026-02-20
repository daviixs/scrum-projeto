# Fluxos do Sistema (Updates de Tarefas via WhatsApp)

Este documento detalha todos os fluxos possíveis do sistema no padrão flowchart TD.

## 1) Fluxo Principal (Happy Path): Lembrete -> Update -> Compliance

```mermaid
%% =========================================================
%% SISTEMA: Updates de Tarefas via WhatsApp (MVP)
%% OBJETIVO: "todos os fluxos possíveis" no padrão flowchart TD
%% =========================================================

%% ---------------------------------------------------------
%% 1) Fluxo Principal (Happy Path): Lembrete -> Update -> Compliance
%% ---------------------------------------------------------
flowchart TD
    A(["Inicio"]) --> B["Lead acessa Dashboard"]
    B --> C["Cria tarefas da semana (titulo/descricao/prazo/prioridade)"]
    C --> D["Atribui tarefas aos devs"]
    D --> E["Configura janelas de lembrete (2 horarios/dia) + timezone"]
    E --> F["Backend persiste Team/User/Task/ReminderPolicy"]
    F --> G["Scheduler cria jobs (time_1 e time_2) por equipe"]
    G --> H(["Aguardando horario do lembrete"])
    H --> I["Worker dispara lembrete do periodo"]
    I --> J["Backend monta lista de tarefas pendentes por dev (fonte: dashboard)"]
    J --> K["Envia mensagem WhatsApp com CTA 'Atualizar agora' + lista curta"]
    K --> L{"WhatsApp entregue?"}
    L -- Sim --> M["Dev clica CTA / escolhe /atualizar"]
    M --> N["Bot exibe tarefas abertas (top 3 por prazo/prioridade)"]
    N --> O["Dev seleciona tarefa"]
    O --> P["Dev informa % (0/25/50/75/100 ou digita)"]
    P --> Q{"Ha bloqueio?"}
    Q -- Nao --> R["Salva TaskUpdate (% + timestamp)"]
    Q -- Sim --> S["Coleta motivo curto do bloqueio"]
    S --> T["Salva TaskUpdate (% + blocked_reason + timestamp)"]
    R --> U["Recalcula compliance do dia (dev em dia?)"]
    T --> U
    U --> V["Dashboard atualiza painel (quase realtime)"]
    V --> W(["Fim: Dev em dia + tarefa atualizada"])
```

## 2) Fluxo /hoje: Dev puxa status sem esperar lembrete

```mermaid
%% ---------------------------------------------------------
%% 2) Fluxo /hoje: Dev puxa status sem esperar lembrete
%% ---------------------------------------------------------
flowchart TD
    A2(["Inicio"]) --> B2["Dev envia comando /hoje no WhatsApp"]
    B2 --> C2["Webhook recebe mensagem"]
    C2 --> D2["Backend autentica remetente (phone -> User)"]
    D2 --> E2{"User encontrado e ativo?"}
    E2 -- Nao --> F2["Responde: 'Nao reconhecido/convite necessario'"]
    E2 -- Sim --> G2["Busca tarefas abertas do dev (dashboard = fonte)"]
    G2 --> H2{"Existe tarefa aberta?"}
    H2 -- Nao --> I2["Responde: 'Nenhuma tarefa pendente hoje'"]
    H2 -- Sim --> J2["Responde lista curta + CTA 'Atualizar agora'"]
    J2 --> K2(["Fim"])
```

## 3) Fluxo /atualizar: Dev inicia update manualmente

```mermaid
%% ---------------------------------------------------------
%% 3) Fluxo /atualizar: Dev inicia update manualmente
%% ---------------------------------------------------------
flowchart TD
    A3(["Inicio"]) --> B3["Dev envia /atualizar"]
    B3 --> C3["Webhook recebe comando"]
    C3 --> D3["Backend valida User e horario permitido (janela de trabalho)"]
    D3 --> E3{"Dentro do horario de trabalho?"}
    E3 -- Nao --> F3["Responde: 'Fora do horario. Tente mais tarde'"]
    E3 -- Sim --> G3["Mostra tarefas abertas (top 3 + opcao 'ver mais')"]
    G3 --> H3["Dev seleciona tarefa"]
    H3 --> I3["Dev informa %"]
    I3 --> J3{"Bloqueado?"}
    J3 -- Nao --> K3["Grava TaskUpdate e confirma"]
    J3 -- Sim --> L3["Coleta motivo e grava TaskUpdate"]
    K3 --> M3["Atualiza compliance e painel do lead"]
    L3 --> M3
    M3 --> N3(["Fim"])
```

## 4) Anti-fatigue: Segundo lembrete suprimido ou vira soft ping

```mermaid
%% ---------------------------------------------------------
%% 4) Anti-fatigue: Segundo lembrete suprimido ou vira soft ping
%% ---------------------------------------------------------
flowchart TD
    A4(["Inicio"]) --> B4(["Horario do 2o lembrete do dia"])
    B4 --> C4["Worker avalia compliance do dev no dia"]
    C4 --> D4{"Dev ja fez 1 update valido hoje?"}
    D4 -- Sim --> E4{"Politica: suprimir ou soft ping?"}
    E4 -- Suprimir --> F4["Nao envia mensagem (silencio)"]
    E4 -- Soft ping --> G4["Envia ping curto: 'Tudo certo por ai? (ja esta em dia)'" ]
    D4 -- Nao --> H4["Envia lembrete normal com CTA atualizar"]
    F4 --> I4(["Fim"])
    G4 --> I4
    H4 --> I4
```

## 5) Idempotencia: Evitar update duplicado no webhook

```mermaid
%% ---------------------------------------------------------
%% 5) Idempotencia: Evitar update duplicado no webhook
%% ---------------------------------------------------------
flowchart TD
    A5(["Inicio"]) --> B5["Webhook recebe resposta do dev (button/list/text)"]
    B5 --> C5["Extrai message_id + payload (task_id/%/blocked_reason)"]
    C5 --> D5{"message_id ja processado?"}
    D5 -- Sim --> E5["Ignora duplicado e responde 'Ok (ja registrado)'"]
    D5 -- Nao --> F5["Valida payload e grava TaskUpdate"]
    F5 --> G5["Marca message_id como processado (MessageLog/IdempotencyKey)"]
    G5 --> H5(["Fim"])
    E5 --> H5
```

## 6) Fallback de texto ambíguo: 'nao entendi' -> fluxo guiado

```mermaid
%% ---------------------------------------------------------
%% 6) Fallback de texto ambíguo: 'nao entendi' -> fluxo guiado
%% ---------------------------------------------------------
flowchart TD
    A6(["Inicio"]) --> B6["Dev envia texto livre (ex: '50% na tarefa X')"]
    B6 --> C6["Parser tenta extrair tarefa e percentual"]
    C6 --> D6{"Conseguiu mapear com confianca?"}
    D6 -- Sim --> E6["Confirma: 'Entendi 50% na tarefa X. Ha bloqueio?'"]
    E6 --> F6{"Bloqueio?"}
    F6 -- Nao --> G6["Grava TaskUpdate e confirma"]
    F6 -- Sim --> H6["Coleta motivo, grava e confirma"]
    D6 -- Nao --> I6["Responde: 'Nao entendi. Selecione uma opcao:'"]
    I6 --> J6["Mostra lista guiada de tarefas + botoes de %"]
    J6 --> K6["Dev segue fluxo guiado normal"]
    G6 --> L6(["Fim"])
    H6 --> L6
    K6 --> L6
```

## 7) Falha de entrega no WhatsApp: retry + log + alerta no dashboard

```mermaid
%% ---------------------------------------------------------
%% 7) Falha de entrega no WhatsApp: retry + log + alerta no dashboard
%% ---------------------------------------------------------
flowchart TD
    A7(["Inicio"]) --> B7["Worker envia lembrete para dev"]
    B7 --> C7{"Provider retornou erro?"}
    C7 -- Nao --> D7["Registra MessageLog: delivered/pending"]
    D7 --> E7(["Fim"])
    C7 -- Sim --> F7["Registra MessageLog: failed + motivo"]
    F7 --> G7["Retry com backoff (ex: 1m, 5m, 15m)"]
    G7 --> H7{"Recuperou no retry?"}
    H7 -- Sim --> I7["Atualiza MessageLog: delivered"]
    I7 --> E7
    H7 -- Nao --> J7["Marca como 'critical failure'"]
    J7 --> K7["Alerta no dashboard do lead (falha de entrega)"]
    K7 --> E7
```

## 8) Dev ausente (ferias/folga): suspende cobrança automática

```mermaid
%% ---------------------------------------------------------
%% 8) Dev ausente (ferias/folga): suspende cobrança automática
%% ---------------------------------------------------------
flowchart TD
    A8(["Inicio"]) --> B8["Lead marca dev como 'Ausente' no dashboard"]
    B8 --> C8["Backend salva status do user (absent=true, periodo)"]
    C8 --> D8(["Horario de lembrete"])
    D8 --> E8["Worker monta lista de destinatarios"]
    E8 --> F8{"Dev esta ausente?"}
    F8 -- Sim --> G8["Nao envia lembrete + nao conta no compliance"]
    F8 -- Nao --> H8["Envia lembrete normal"]
    G8 --> I8(["Fim"])
    H8 --> I8
```

## 9) Tarefa concluída e reaberta: histórico preservado

```mermaid
%% ---------------------------------------------------------
%% 9) Tarefa concluída e reaberta: histórico preservado
%% ---------------------------------------------------------
flowchart TD
    A9(["Inicio"]) --> B9["Dev atualiza tarefa para 100%"]
    B9 --> C9["Backend grava TaskUpdate(100%) + marca status DONE"]
    C9 --> D9(["Dias depois..."])
    D9 --> E9["Lead reabre tarefa no dashboard (status volta OPEN)"]
    E9 --> F9["Backend preserva TaskUpdate antigo (historico)"]
    F9 --> G9["Proximo lembrete inclui tarefa reaberta"]
    G9 --> H9(["Fim"])
```

## 10) Mudança de prioridade no dia: refletir sem flood

```mermaid
%% ---------------------------------------------------------
%% 10) Mudança de prioridade no dia: refletir sem flood
%% ---------------------------------------------------------
flowchart TD
    A10(["Inicio"]) --> B10["Lead altera prioridade/prazo no dashboard"]
    B10 --> C10["Backend salva Task (priority/due_date)"]
    C10 --> D10{"Enviar mensagem imediata?"}
    D10 -- Nao (MVP) --> E10["Nao notifica agora (evita flood)"]
    E10 --> F10(["Proximo lembrete usa dados atualizados"])
    D10 -- Sim (futuro) --> G10["(V1+) Notifica mudanca de prioridade com rate limit"]
    F10 --> H10(["Fim"])
    G10 --> H10
```

## 11) Escalonamento ao lead: ausencia recorrente (ex: 2 dias sem update)

```mermaid
%% ---------------------------------------------------------
%% 11) Escalonamento ao lead: ausencia recorrente (ex: 2 dias sem update)
%% ---------------------------------------------------------
flowchart TD
    A11(["Inicio"]) --> B11(["Fim do dia util"])
    B11 --> C11["Backend calcula compliance do dev no dia"]
    C11 --> D11{"Dev ficou sem update hoje?"}
    D11 -- Nao --> E11["Zera contador de ausencia consecutiva"]
    D11 -- Sim --> F11["Incrementa contador de ausencia consecutiva"]
    F11 --> G11{"Ausencia >= 2 dias?"}
    G11 -- Nao --> H11["Nao escalona (apenas registra)"]
    G11 -- Sim --> I11["Notifica lead no dashboard (alerta de ausencia)"]
    E11 --> J11(["Fim"])
    H11 --> J11
    I11 --> J11
```

## 12) Regra de compliance: definição de 'em dia' por dia útil

```mermaid
%% ---------------------------------------------------------
%% 12) Regra de compliance: definição de 'em dia' por dia útil
%% ---------------------------------------------------------
flowchart TD
    A12(["Inicio"]) --> B12["Recebe TaskUpdate valido do dev (hoje)"]
    B12 --> C12{"Dia util para a equipe?"}
    C12 -- Nao --> D12["Nao conta compliance (fim de semana/feriado)"]
    C12 -- Sim --> E12["Marca dev 'em dia' (>=1 update valido)"]
    E12 --> F12["Atualiza taxa diaria/semanal no dashboard"]
    D12 --> G12(["Fim"])
    F12 --> G12
```
