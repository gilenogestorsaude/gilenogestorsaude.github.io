# Estado do Projeto — Gestão Saúde

**Última atualização:** 2026-05-23
**Versão atual em produção:** v1.4.0
**URL:** https://gilenogestorsaude.github.io
**Repo:** https://github.com/gilenogestorsaude/gilenogestorsaude.github.io
**Firebase project:** gileno-gestao-saude

> Este documento é o **handoff vivo** do projeto. Qualquer nova sessão de trabalho começa lendo este arquivo pra entender estado atual, decisões já tomadas, e próximos passos.

---

## Resumo da sessão 2026-05-23

Sessão maratona que evoluiu o app de v1.0.6 (esqueleto parado há 78 dias) pra **v1.4.0 cobrindo 5/6 promessas do site plenamente + 1/6 parcial**. Pronto pra lançamento oficial assim que a S6 (lançamento) for executada.

### Commits desta sessão (10 versões)

| Versão | Commit | O que entregou |
|---|---|---|
| v1.0.7 | 894ef16 | PWA fix (manifest.json) + Firestore rules + remoção de auto-save redundante |
| v1.1.0 | 90a685e | S2 — Medicamentos & suplementos (schema + página + widget + alertas) |
| v1.1.1 | 297d62a | S2.1 — Cadastro e gestão de alimentos (gap funcional crítico) |
| v1.1.2 | 4be3141 | S2.2 — TACO oficial UNICAMP (576 alimentos) + 4 macros + autocomplete |
| v1.1.3 | 0247ec5 | Display dos 4 macros consistente em toda UI |
| v1.1.4 | 17323b7 | Proteção tripla contra erro de referência (escala automática + preview por 100g + validador sanidade) |
| v1.1.5 | dc12a9d | Migration corretiva: defaults antigos → valores TACO |
| v1.2.0 | e3d1818 | S3 — Registro de treino (exercícios, séries, reps, cargas) |
| v1.3.0 | 2979c41 | S4 — Histórico de consultas médicas |
| v1.4.0 | 3b08064 | S5 — Histórico de sinais vitais (peso, PA, FC) |

---

## Stack confirmada

- **Frontend:** Vanilla JS + HTML + CSS (sem framework, sem build) — single-file `index.html` ~2500 linhas
- **Backend:** Firebase 10.12.0 (Auth + Firestore)
- **Storage de dados:** Firestore — coleção `users/{uid}` doc único com campo `data: JSON.stringify(D)` + `updatedAt`
- **Hosting:** GitHub Pages
- **PWA:** Service Worker network-first + manifest.json + cache de `taco.json`
- **Auth:** Email/senha + Google OAuth popup

### Repos relevantes locais
- Clone: `/Users/gilenopaiva/Documents/Gileno_Gestao/Gestao_Saude/`
- Remote SSH (chave do usuário GitHub `gilenogestorfinanceiro` tem permissão de push): `git@github.com:gilenogestorsaude/gilenogestorsaude.github.io.git`

---

## Schema de dados (objeto D persistido em Firestore)

```
D = {
  userName, peso, metaPeso,
  days: { 'YYYY-MM-DD': { slots: { pretreino|postreino|cafe|almoco|lanche|jantar: {items, time} } } },
  water: { 'YYYY-MM-DD': [{time, vol, source, auto}] },
  goals: {
    treino:   {kcal, prot, water, carbo?, gord?},
    descanso: {kcal, prot, water, carbo?, gord?}
  },
  dayType: { 'YYYY-MM-DD': 'treino'|'descanso' },
  vitals: { 'YYYY-MM-DD': {peso, pa, fc} },
  foods: [{id, name, ref, unit, kcal, prot, carbo, gord, nota}],
  meds: [{id, nome, dosagem, horarios[], estoque?, notas?, ativo}],
  medsTaken: { 'YYYY-MM-DD': {medId: ['HH:MM', ...]} },
  treinos: [{id, data, nome, exercicios[{id, nome, series[{reps, carga}]}], notas?, duracao?, createdAt}],
  consultas: [{id, data, medico, especialidade, local?, queixa, conduta, prescricao?, proximaConsulta?, notas?, createdAt}],
  theme, bphCutoff
}
```

---

## Cobertura das 6 promessas do site

| Promessa | Status | Versão que fechou |
|---|---|---|
| Diário nutricional completo | ✅ pleno | v1.1.2 (TACO + 4 macros) |
| Controle de medicamentos | ✅ pleno | v1.1.0 |
| Registro de atividade física | ✅ pleno | v1.2.0 |
| Histórico de consultas médicas | ✅ pleno (Free básico) | v1.3.0 |
| Monitoramento de sinais vitais | ✅ pleno | v1.4.0 |
| Alertas e lembretes personalizados | ⚠️ parcial (9 alertas in-app) | — (push é Premium) |

---

## Estratégia comercial — modelo freemium qualitativo

Decidido em 2026-05-23. **Free entrega o básico das 6 promessas; Premium entrega a camada de inteligência/automação que torna o app indispensável.**

### Free (lançável já, depois da S6)
- Cadastro completo das 6 áreas: nutrição, medicamentos, treino, consultas, vitais, hidratação
- TACO oficial UNICAMP (576 alimentos brasileiros)
- Limites: 30 dias de histórico de vitais, últimas 50 consultas, sem gráficos, sem push, sem IA

### Premium (futuro, R$ 9,90/mês ou R$ 79/ano — referência)
Roadmap detalhado em `IDEIAS_PREMIUM.md`. Pilares:
- **Inteligência:** análise IA dos hábitos, lembretes médicos IA, extração de exames
- **Automação:** push notifications, integrações Apple Health/Google Fit, código de barras
- **Portabilidade:** PDF semanal/mensal, dossiê médico, exportações
- **Amplitude:** detalhamento nutricional avançado, múltiplos perfis (família), histórico ilimitado
- **Profundidade clínica:** camada de consultas Premium (anexar PDFs de exames + IA, gráficos de evolução de marcadores, lembretes inteligentes)

**Killer feature do Premium:** anexar PDFs de exames laboratoriais + extração via Claude API + dossiê médico em PDF pra levar ao médico.

---

## Decisões estratégicas já tomadas

1. **Modelo freemium qualitativo** (não quantitativo) — Free completo, Premium adiciona poderes
2. **Consultas médicas no Free** com camada Premium avançada (anexar PDFs + IA + dossiê) — decisão de 23/05 após discussão Free × Premium
3. **TACO embarcada** (UNICAMP) como fonte primária de dados nutricionais; OpenFoodFacts pode ser adicionado depois como fallback pra industrializados
4. **Sem Apple Health integration no Free** — fica como Premium (exige Capacitor/wrapper iOS)
5. **Bottom-nav fixa em 5 itens** (Início, Refeição, Água, Meds, Metas); features adicionais (Treino, Consultas, Vitais) acessadas via dashboard/header
6. **PWA, sem app nativo store** por enquanto — instalável via "Adicionar à Tela Inicial" no iOS/Android
7. **Marketplace de profissionais** (item #5 do IDEIAS_PREMIUM.md) é projeto separado de 3-6 meses, só faz sentido depois de massa crítica de usuários (100-500 ativos)

---

## Pendências críticas pra S6 (lançamento oficial)

Bloqueante:
- [ ] **Site `gilenogestao.com.br`**: mudar "100% gratuito" → "Comece grátis"
- [ ] **Site:** remover claim "48-bit encryption" (tecnicamente errado)
- [ ] **Site:** mudar status "Em Breve" → "Disponível" + adicionar link/botão pro app
- [ ] **Política de privacidade** (obrigatório LGPD — dados de saúde)
- [ ] **Termos de uso** (limita responsabilidade — "não substitui consulta médica")
- [ ] **Página FAQ / Sobre** no app

Importante mas não-bloqueante:
- [ ] Auditoria Lighthouse PWA (alvo: score > 90)
- [ ] SEO básico no site (meta tags, sitemap, Open Graph)
- [ ] Onboarding pra novo usuário (tour 3-4 telas)
- [ ] Métricas (Google Analytics ou Plausible)
- [ ] Canal de feedback (email ou WhatsApp pra bug reports)

Marketing:
- [ ] Post Instagram do lançamento
- [ ] Mensagem WhatsApp pra rede próxima
- [ ] Story de lançamento

**Estimativa S6 total: 5-7h dividido em 2-3 sub-sessões (S6a site+legal, S6b técnico+onboarding, S6c marketing)**

---

## Bugs conhecidos / observações

- Hospedagem do site é **Readdy.ai** (no-code AI builder) — não WordPress. Edições via editor visual proprietário; não há REST API. Pra mudanças no site, fluxo é "Claude descreve mudança específica → Gileno aplica no editor → Publish".
- Modal `prompt()` nativo ainda usado em `editGoal()` — UX pode ser melhorada (não-crítico)
- innerHTML extensivo — vetor XSS teórico se food.name contiver script (não-crítico pra uso pessoal)
- Service Worker cacheia indefinidamente — sem limite de tamanho (não-crítico pra escala atual)

---

## Frase pra retomar este projeto em nova sessão

```
Estou retomando o app Gestão Saúde. Lê /Users/gilenopaiva/Documents/Gileno_Gestao/Gestao_Saude/ESTADO_PROJETO.md
pra contexto, e o IDEIAS_PREMIUM.md no mesmo dir pra roadmap. Versão atual em produção: v1.4.0.

[Aqui descreve o que quer fazer: bug encontrado no teste, próxima feature, S6 lançamento, etc]
```

---

## Outros projetos em pausa (do mesmo ecossistema Gileno Gestão)

Não relacionados a Gestão Saúde, mas anotados pra você não perder:

- **VPS 2.0'-D** (`~/Documents/Gileno_Gestao/VPS_Diagnostico/`): instalação CC nativo na VPS Hostinger. Pausada na Fase C (briefing executável pronto, falta executar token longa-duração + smoke DW). Decisão pendente sobre como resolver coleta automatizada do Lado B (extratos bancários) da conciliação Agrodel.

- **Wait state Andressa Chianca:** aguardando resposta dela sobre whitelist IP + CA cert + adesão Bitwarden Send. Email enviado em 22/05 18:05. Deadline 26/05.
