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

## 9. Anexar exames laboratoriais + extração por IA (Premium individual)

Upload de PDF do exame → Claude API ou GPT Vision extrai valores → gráfico histórico (glicemia, colesterol, vitamina D, etc).

**Esforço:** 3-5h. Inclui storage Firebase Storage (PDFs ficam aqui) + extração estruturada por IA + UI de visualização temporal.

Alto valor clínico — médico fica feliz quando paciente leva exames organizados.

---

## Prioridades sugeridas (ordem de implementação após Free)

1. **Notificações push** (#3) — concretiza promessa do site, atrai usuário a virar Premium
2. **PDF semanal/mensal** (#4) — valor visível imediato
3. **Detalhamento nutricional** (#1) — barato de implementar, dados já estão guardados
4. **Análise IA** (#2) — diferencial técnico forte
5. **Código de barras** (#8) — UX delightful
6. **Múltiplos perfis** (#7) — expande TAM (família)
7. **Apple Health / Google Fit** (#6) — atrai público fitness
8. **Exames laboratoriais** (#9) — diferencial clínico
9. **Marketplace de profissionais** (#5) — só depois de tudo acima + massa crítica de usuários

---

*Última atualização: 2026-05-23 (v1.1.2)*
