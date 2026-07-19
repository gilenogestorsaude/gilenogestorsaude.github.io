# Ideias para versão Premium / futuro

Documento vivo — anotações de features e modelos de negócio que ficam fora do MVP Free, mas têm valor estratégico.

---

## 1. Detalhamento nutricional avançado (Premium individual)

**Status:** roadmap próximo (S2.3+)

Free já entrega 4 macros (kcal, prot, carbo, gord). Premium adiciona:

- Gorduras saturadas (g) — relevante p/ colesterol e saúde cardiovascular
- Fibras (g) — relevante p/ saciedade e função intestinal (especialmente BPH)
- Sódio (mg) — relevante p/ pressão arterial
- Cálcio (mg) — relevante p/ saúde óssea após 50 anos
- Ferro (mg) — relevante p/ prevenção de anemia

Todos esses campos já estão guardados em `taco.json` (foram normalizados em v1.1.2) — só falta UI pra exibi-los e cobrança.

**Esforço:** 1-2h. Adicionar campos no modal de food, somar no dashboard quando habilitado, esconder atrás de feature flag.

---

## 2. Análise por IA dos hábitos (Premium individual)

Usar Claude API ou GPT pra gerar análises personalizadas:

- "Como meu mês foi comparado ao anterior?"
- "Padrões de hidratação fracos identificados — sugiro alterações"
- "Aderência a medicamentos: 87% — Addera com 3 esquecimentos recorrentes às terças"
- "Refeição com proteína mais baixa que a meta há 8 dias seguidos"

**Esforço:** 2-3h pra integração inicial. Custo: ~R$ 0,05-0,15 por relatório (manageable).

Aproveita a skill `assistente-performance` já existente, que gera relatório semanal sábado.

---

## 3. Notificações push (Premium individual)

Firebase Cloud Messaging pra lembretes reais (mesmo com app fechado):

- Horário de medicamento se passou >15 min
- Lembrete de hidratação por bloco BPH
- Sinais vitais: "registre seu peso semanal"
- Treino: "você não treinou hoje — agendar?"
- Refeição: "almoço previsto pras 12:30 — registre"

**Esforço:** 2-3h. Inclui setup VAPID keys, service worker push handler, modal de permissão.

Concretiza a promessa do site: "Alertas e lembretes personalizados" (hoje só alertas visuais in-app).

---

## 4. Exportar PDF semanal/mensal formatado (Premium individual)

**Status: PARCIALMENTE ENTREGUE na v1.11.0 (18/07/2026)** — Relatório Semanal (Pro) no app: tabelas, médias, metas, gráficos SVG, alertas por regra, anotações da semana e exportação em HTML imprimível (salvar como PDF). Falta: versão mensal e a camada de IA (#2, Etapa 2 via VPS).

Botão que gera PDF rico:

- Capa com período
- Gráficos de tendência (peso, PA, FC, adesão de medicamentos)
- Tabela de refeições x macros
- Score de adesão de medicamentos
- Anotações livres do usuário no período

**Esforço:** 3-4h. Possíveis libs: `jsPDF` + `Chart.js`.

Útil pra levar pro médico/nutricionista — diferencial real.

---

## 5. Marketplace de profissionais de saúde (B2B2C — produto à parte)

**Status:** projeto separado — não é feature do app, é produto que CONSOME o app como onboarding/audiência.

### Modelo de negócio

- **App Gestão Saúde** agrega usuários finais (lado C)
- **Profissionais** (nutricionistas, personal trainers, médicos, fisioterapeutas, psicólogos) pagam pra estar listados ou racham comissão por agendamento
- **Gileno Gestão** é a infra que conecta — lado B² central

Receita possível:
- Plano mensal pago pelos profissionais (R$ 49-149/mês — modelo Doctoralia)
- Comissão por consulta agendada (10-15% — modelo Vittude)
- Permuta (eles indicam o app pros pacientes, app destaca o perfil deles)
- Misto (assinatura básica + comissão sobre primeiros agendamentos)

### Precedentes brasileiros validados

| Empresa | Foco | Modelo |
|---|---|---|
| Doctoralia | Médicos | Assinatura + plano premium pro profissional |
| Conexa Saúde | Telemedicina | Comissão + B2B (planos de saúde) |
| Vittude | Psicólogos online | Comissão por consulta (~15%) |
| GoodWill / GympassWellz | Wellness | Assinatura corporativa |

### Componentes técnicos

| Componente | Esforço | Risco |
|---|---|---|
| Perfil de profissional (CRM/CREF/CRN/CRP, bio, foto, especialidade, localização) | Médio | Baixo |
| Sistema de agendamento (slots, calendário, fuso horário, conflitos, cancelamento) | Grande | Médio |
| Pagamento (Stripe Connect ou Mercado Pago split) — antes ou depois da consulta | Grande | Médio (gateways exigem documentação fiscal) |
| Verificação de credenciais (CFM, CREF, CRN, CRP reais) | Médio | **Alto** (responsabilidade legal se profissional falso) |
| Chat ou link pra consulta (Zoom/Meet/Daily.co integrado) | Médio | Baixo |
| Sistema de reviews + denúncia | Médio | Médio |
| Compliance LGPD pra dados de saúde compartilhados | Médio | **Alto** (dados sensíveis) |
| Admin do marketplace (você gerenciar profissionais cadastrados) | Médio | Baixo |
| Termos legais (contrato profissional × Gileno Gestão, responsabilidade civil) | — | **Alto** — precisa advogado especializado |
| Notas fiscais / repasses | Médio | Médio (contabilidade B2B2C é complexa) |

**Estimativa total para MVP funcional:** 3-6 meses de trabalho dedicado.

### Pré-requisitos antes de começar

1. **App Free lançado e estável** (cobertura promessa do site = 6/6)
2. **Massa crítica de usuários** — marketplace sem usuários não atrai profissionais (efeito de rede). Mínimo realista: 200-500 usuários ativos. Sem isso, profissionais não pagam.
3. **Decisão sobre escopo geográfico inicial** — só PB? Nordeste? Brasil? Cada um exige rede de profissionais diferente.
4. **Advogado especializado em saúde digital** consultado (LGPD em saúde, responsabilidade civil, termos de uso de profissionais de saúde)
5. **Conta empresarial e CNPJ ativo** pra Gileno Gestão receber via gateway (Stripe Connect exige)

### Sequenciamento recomendado

Depois que app Free atingir uns 100-200 usuários:

1. **Fase 0 (1 mês):** validação de demanda — talk com 10 profissionais e 30 usuários sobre interesse
2. **Fase 1 (2 meses):** MVP do marketplace — perfis estáticos + link de contato (sem agendamento integrado, sem pagamento — só vitrine)
3. **Fase 2 (2 meses):** agendamento + pagamento via Stripe Connect
4. **Fase 3 (1 mês):** verificação de credenciais + audit trail + LGPD compliance check
5. **Fase 4 (ongoing):** marketing, captação de profissionais, ajustes

Total: ~6 meses do início (Fase 0) ao produto em produção com 20-50 profissionais ativos.

---

## 6. Integração Apple Health / Google Fit / Strava (Premium individual)

Importar automaticamente:

- Peso (balança smart Bluetooth)
- Passos
- Frequência cardíaca de repouso
- Treinos do Strava/Garmin
- Sono do Apple Watch

**Esforço:** 4-6h. HealthKit JS API tem limitações; provavelmente exige wrapper Capacitor pra app iOS nativo. Google Fit é mais aberto.

---

## 7. Múltiplos perfis (Premium familiar)

Um único usuário pagante gerencia perfis pra família (esposa, filhos, pais idosos). Compartilhamento opcional de dados de saúde.

**Modelo:** plano "Família" 50-80% mais caro que individual.

**Esforço:** 3-4h. Schema Firestore muda (não é mais `users/{uid}` mas `accounts/{uid}/profiles/{profileId}`).

---

## 8. Reconhecimento de código de barras (Premium individual)

Câmera do celular lê código de barras de embalagem → busca OpenFoodFacts → preenche food automaticamente.

**Esforço:** 2-3h. `QuaggaJS` ou `@zxing/browser` no front. OpenFoodFacts já tem API por barcode.

Diferencial concreto pra industrial izados (whey de marca, barrinhas, suplementos).

---

## 9. Consultas Médicas — Camada Premium (alto valor clínico)

**Status:** decidido em 2026-05-23 que Free entrega cadastro básico de consultas (S4, já implementado em v1.3.0) e Premium entrega a camada avançada que torna o app indispensável pra cuidar da saúde.

### Free (já entregue na v1.3.0)
- Cadastro: data, médico, especialidade, queixa, conduta, prescrição, próxima consulta, notas
- Lista cronológica das **últimas 50 consultas**
- Alerta no dashboard de consulta nos próximos 7 dias
- Separação Agendadas × Histórico
- Modal de edição completo

### Premium (roadmap)

#### 9.1. Histórico ilimitado
Free limita visualização das últimas 50; Premium remove limite.
Implementação: paginação lazy + remoção do slice no `rConsultas()`.
**Esforço:** 30 min. Trivial. Só vale a pena quando há paywall implementado.

#### 9.2. Anexar PDFs de exames + extração por IA *(killer feature)*
Upload de PDF do exame laboratorial → Claude API ou GPT Vision extrai valores estruturados → guarda no Firestore relacionado à consulta → mostra evolução temporal (glicemia, colesterol total, HDL, LDL, vitamina D, TSH, PSA, hemoglobina, etc).

**Esforço:** 4-6h. Inclui:
- Firebase Storage pra hospedar PDFs (cota free: 5 GB)
- Extração via Claude API (~R$ 0,02-0,08 por PDF de 1-2 páginas)
- UI de visualização: lista de exames anexos por consulta + dossiê histórico de marcadores
- Tratamento de extração ambígua (review pelo user antes de salvar)

**Valor pra cliente:** médico fica MUITO feliz quando paciente leva exames organizados + evolução temporal — é a coisa que cardiologista, endocrino, urologista mais pedem e raramente recebem.

#### 9.3. Notificação push de consulta próxima
Free tem alerta in-app (precisa abrir o app pra ver). Premium tem push real via Firebase Cloud Messaging que chega no celular mesmo com app fechado.

**Esforço:** 2-3h. VAPID keys + service worker handler + modal de permissão. Compartilhado com #3 do roadmap geral.

#### 9.4. Gerar dossiê médico em PDF *(diferencial REAL)*
Botão "Exportar histórico médico em PDF" que gera dossiê profissional com:
- Identificação (nome, idade, contatos de emergência)
- Medicamentos atuais em uso (consome `D.meds`)
- Alergias e condições crônicas (campo novo)
- Última PA / FC / peso (consome vitals)
- Últimas 10 consultas resumidas (especialidade, data, queixa, conduta)
- Últimos exames anexados (resumo de marcadores)
- Sinais vitais — gráfico de tendência (se Premium)

**Esforço:** 4-5h. jsPDF + Chart.js + template profissional + opção "compartilhar via WhatsApp/email".

**Valor:** leva pra consulta com qualquer médico → "Doutor, aqui está meu histórico organizado" → experiência transformadora pro paciente.

#### 9.5. Lembretes inteligentes via IA
Claude API roda análise periódica (semanal/mensal) sobre o histórico:
- "Última visita ao urologista foi há 8 meses — agendar?"
- "PA elevada em 3 das últimas 5 medições — considere consulta com cardio"
- "Glicose vem subindo gradualmente — vale pedir hemoglobina glicada"

**Esforço:** 3-4h. Função Cloud (Firebase Functions) + prompt engineering + integração com `D.consultas + D.vitals + exames`.

**Custo operacional:** ~R$ 0,10-0,30 por análise por usuário (Claude Haiku barato). Inclui no preço Premium.

#### 9.6. Compartilhamento com cuidadores/família
Permite usuário Premium compartilhar leitura do histórico médico com email autorizado (esposa, filho, cuidador). Útil pra idosos ou pacientes em tratamento sério.

**Esforço:** 3-4h. Schema Firestore muda (autorização separada por email), UI de gestão de compartilhamento, isolamento de leitura (cuidador vê mas não edita).

#### 9.7. Comparativo temporal de exames
Quando há múltiplos exames do mesmo tipo, mostra gráfico de evolução: colesterol total ao longo de 2 anos, TSH ao longo de 5 consultas, PSA por consulta urológica, etc.

**Esforço:** 2-3h. Depende de #9.2 estar implementado (precisa dos valores extraídos). Chart.js + UI de seleção de marcador.

#### 9.8. Múltiplas tags/categorias por consulta
Free tem só especialidade. Premium permite categorizar com tags: "rotina anual", "preventivo", "urgência", "follow-up", "segunda opinião", etc. Útil pra filtrar e analisar padrões.

**Esforço:** 1-2h. Trivial.

### Por que essa estratégia funciona

**Free hookat** com cadastro completo (resolve a dor básica: "ter histórico médico organizado").
**Premium diferencia** com IA + automação + PDF dossiê (resolve dor profunda: "ser um paciente bem preparado, com dados na mão").
**Paciente fideliza** porque cancelar Premium = perder exames anexados, gráficos, dossiê = friction alto.

### Sequenciamento sugerido (depois do Free lançado)

1. **9.3 Push notifications** (compartilhado com roadmap geral, baixo custo) — primeiro
2. **9.2 Anexar PDFs + extração IA** — feature ÂNCORA, o que justifica a assinatura
3. **9.4 Dossiê PDF** — multiplica valor das #9.2
4. **9.5 Lembretes IA** — retenção, hábito
5. **9.7 Comparativo de marcadores** — depende de #9.2 ter histórico
6. **9.1 Histórico ilimitado** — quando precisar
7. **9.6 Família** — quando tiver demanda
8. **9.8 Tags** — refinamento

---

## Prioridades sugeridas (ordem de implementação após Free)

Reordenado em 2026-05-23 após decisão estratégica de fazer "Consultas Médicas" como pilar Premium (item #9 robusto).

1. **Notificações push** (#3) — concretiza promessa "alertas e lembretes" do site, retém Free, atrai Premium. Compartilhado entre roadmap geral e item #9.3.
2. **Anexar PDFs de exames + IA** (#9.2) — **feature âncora** do Premium. O que justifica pagar.
3. **Dossiê médico em PDF** (#9.4) — multiplica valor de #9.2. Diferencial REAL pra paciente.
4. **PDF semanal/mensal de saúde geral** (#4) — valor visível imediato, formato similar a #9.4.
5. **Detalhamento nutricional** (#1) — barato, dados já guardados no taco.json.
6. **Análise IA dos hábitos** (#2) — diferencial técnico. Compartilhado com #9.5 (lembretes médicos).
7. **Comparativo temporal de exames** (#9.7) — depende de #9.2 ter histórico acumulado.
8. **Código de barras** (#8) — UX delightful no diário nutricional.
9. **Múltiplos perfis** (#7) — expande TAM (família).
10. **Apple Health / Google Fit** (#6) — atrai público fitness.
11. **Compartilhar família** (#9.6) — pra idosos / cuidadores.
12. **Marketplace de profissionais** (#5) — só depois de tudo acima + massa crítica de usuários (100-500 ativos).

### Pilares do Premium pra justificar assinatura mensal

| Pilar | Componentes |
|---|---|
| **Inteligência** | #2 análise hábitos, #9.5 lembretes médicos IA, #9.2 extração de exames |
| **Automação** | #3 push, #6 integrações, #8 código de barras |
| **Portabilidade** | #4 PDF semanal, #9.4 dossiê médico, exportações |
| **Amplitude** | #1 nutricional avançado, #7 família, #9.1 histórico ilimitado |
| **Profundidade clínica** | #9.2/9.4/9.5/9.7 — toda a camada de consultas Premium |

A camada **Profundidade clínica** (item #9 expandido) é o que torna o Gileno Gestão Saúde **diferente** de qualquer app de tracking de nutrição/treino do mercado.

---

*Última atualização: 2026-05-23 (após decisão Free × Premium pra Consultas — v1.3.0)*
