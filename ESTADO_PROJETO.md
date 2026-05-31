# Estado do Projeto — Gestão Saúde

**Última atualização:** 2026-05-31
**Versão atual em produção:** v1.9.7
**URL:** https://gilenogestorsaude.github.io
**Repo:** https://github.com/gilenogestorsaude/gilenogestorsaude.github.io
**Firebase project:** gileno-gestao-saude

> Este documento é o **handoff vivo** do projeto. Qualquer nova sessão de trabalho começa lendo este arquivo pra entender estado atual, decisões já tomadas, e próximos passos.

---

## Resumo da sessão 2026-05-31 — v1.9.6 e v1.9.7 (refinos do plano alimentar, em uso real)

Ajustes que surgiram quando o Gileno foi importar/testar o plano da Débora de verdade.

**v1.9.6 — gesto do Modo Pro mais fácil no celular.** O alvo de 9px no "v1.9.x" do header era difícil de acertar 5× no telefone. Agora o gesto também funciona na linha **"Versão" em Configurações** (alvo grande, `onclick="proTap()"`); janela entre toques subiu de 1,5s → **2,5s**; e a partir do 2º toque aparece feedback *"faltam N toque(s)…"*. Mesma função `proTap`, só mais tolerante.

**v1.9.7 — ajustar a quantidade ao registrar item do plano** (feedback central do Gileno: "a quantidade indicada nem sempre é a consumida"). Antes, o ✓ de cada linha da sugestão registrava **direto** a grama do plano. Agora o ✓ chama **`openAddFood(foodId, prefillGrams)`** — o MESMO modal de "Adicionar Alimento", com a grama do plano **pré-preenchida e editável** + recálculo de macros ao vivo. `openAddFood` ganhou 2º param opcional `prefillGrams` (sem ele, cai na `food.ref` como antes — retrocompatível). A antiga `logPlanLine` (registro sem ajuste) foi **removida**; o botão **"✓ Comi tudo"** segue para registro em lote (1ª opção das substituições).

**Processo (importante p/ próximas sessões):** adotado **gate automático verify-then-commit** via JavaScriptCore — um script bash roda a suíte + checa sintaxe e SÓ commita/pusha se FAIL=0. O gate **barrou commits quebrados 2×** nesta leva (Edit com âncora errada deixando função ausente; teste cosmético desatualizado) — provou o valor. Sempre rodar o gate antes de subir.

**Como o Gileno usa o import (estado atual = concierge):** o app NÃO lê PDF ainda (é a feature Pro "em construção"). Fluxo real hoje: Claude gera o JSON do PDF → Gileno **liga o Modo Pro (5 toques em Versão)** → Metas → "📋 Importar plano" → 🥗 nutri → cola o JSON. O JSON do plano FASE 1 vive em `dieta-fase1.json`/`.min.json` (**gitignored**, dado pessoal); cópia pronta pra colar em `~/Downloads/plano-dieta-fase1-COLAR.txt`. ⚠️ Campo do whey/creatina no plano = **"Dejejum"** (sem s) pra casar com o campo real do Gileno.

---

## Resumo da sessão 2026-05-31 — v1.9.5 (MODO PRO preview + import PDF "em construção" + camada profissional)

**Decisão de produto (Gileno):** cada usuário precisa importar o **próprio** PDF (nutri e personal) — senão a feature não serve. Mas isso exige backend (a chave da IA NÃO pode ir no app, repo é público) → vira **Versão Pro paga, futura**. Pra não atrasar o lançamento: publicar já com a feature **anunciada como "em construção"** (feature flag + teaser).

**O que entrou:**
- **Modo Pro (preview)** — gesto secreto: **5 toques no "v1.9.x" do header** liga/desliga `D.proPreview` (toast + badge **PRO** roxo). `proTap()` com debounce 1,5s. É o "modo desenvolvedor" do Android. **Trava COSMÉTICA de propósito** — repo público; o cadeado REAL será no backend quando a Pro existir (cobrança/quem-é-Pro no servidor).
- **Card "📋 Importar plano" (PRO) em Metas** com 2 botões: **🥗 nutri** e **🏋️ personal**. Público vê "🔒 em breve" → modal "🚧 Em construção". Tester com preview ON: **nutri** abre o import real (colar-JSON concierge, `openDietImport`); personal ainda é teaser (treino Pro não construído). `openProFeature(kind)` roteia.
- **Camada profissional (semente da ideia de indicação/monetização):** `dietPlan.meta.professional {name, role, council, phone, instagram}`. `planProfessionalHtml()` rende contato **clicável** — WhatsApp (`wa.me/55…`), `tel:`, Instagram. Fallback p/ `meta.source`. O PDF da nutri já traz nome+CRN+telefone → o import (Pro) capturará isso de graça.
- `D.workoutPlan` reservado (espelha `dietPlan`, p/ o treino na Pro).

**Arquitetura do treino (descoberto nesta sessão):** o treino JÁ tem as 3 camadas prontas de antes — `D.workoutTemplates[]` (plano A/B/C) + `D.treinos[]` (execução) + `executeTemplate()` ("fazer a partir do plano"). Ou seja, o "sugestão+execução" do treino **já funciona**; só falta a mesma porta de entrada (PDF→template) = a feature Pro. Mesmo princípio da dieta.

**Verificação:** JavaScriptCore com **gate automático verify-then-commit** (só commita se sintaxe OK + 0 FAIL + ≥20 PASS + proTap presente sem duplicata). O gate **barrou** um commit onde o Edit do `proTap` falhou por âncora errada (header tinha onclick mas a função não existia) — provou seu valor. Após corrigir: **22/22 checks** (gesto 5-toques toggle, roteamento nutri/personal com e sem preview, contato WhatsApp/tel, fallbacks).

**Deploy:** `APP_VERSION`+`sw.js` → **1.9.5**.

### 🔭 ROADMAP — Versão Pro (import de PDF) — decisão pendente de QUANDO/COMO
Caminhos pra transformar PDF→plano (cada usuário, self-service):
- **A — Concierge (atual):** usuário manda PDF, Claude (eu) gero o JSON, ele cola. Zero infra, não escala. Bom pra fase família.
- **B — Builder manual no app:** tela pro usuário montar o plano à mão (reusa busca TACO). Self-service, sem backend/custo, mas trabalhoso.
- **C — IA lê o PDF:** PDF → **VPS do Gileno** (pasta `VPS_Diagnostico`) ou Cloud Function → Claude API → JSON → `processDietImport`. Self-service real. Precisa backend + custo/PDF + **revisão obrigatória** (saúde: nunca importar cego — usuário confere/corrige/confirma).
- Recomendação registrada: validar parsing com 2-3 formatos de PDF **antes** de fixar infra.

### 💰 ROADMAP — Monetização "espaço profissional" (ideia do Gileno) — FUTURO, NÃO agora
Vender espaço/indicação pra nutri/personal/**médicos** (vínculo com módulo Consultas). Separar SEMPRE: **(A) guardar o contato do profissional do próprio usuário** = feito agora (útil, baixo risco) vs. **(B) vender indicação a terceiros** = receita só com audiência + **risco regulatório** (publicidade de saúde é regulada por CFM/CRN/CREF; "anúncio disfarçado de recomendação do app" é perigoso). Decidir com público + checagem jurídica. NÃO implementar (B) na fase de testes.

---

## Resumo da sessão 2026-05-31 — v1.9.4 (PLANO ALIMENTAR — Parte 2: execução em paralelo)

Fecha o objetivo "dieta como sugestão + execução em paralelo". A Parte 1 (v1.9.3) trouxe o plano como leitura; esta traz o **registro de 1 toque** e a **aderência** (planejado × realizado), sem nunca misturar as duas camadas.

**No card "📋 Plano da nutri" de cada refeição:**
- **✓ por linha** — registra aquele item do plano na execução (`logPlanLine`). Numa substituição ("ou"), cada opção tem seu ✓ → você escolhe qual comeu (frango? camarão?).
- **✓ Comi tudo** — registra a refeição inteira conforme o plano (`logWholePlanMeal`); substituições usam a **1ª opção**.
- **Aderência** — "Previsto X kcal/prot · Comido Y kcal/prot (Z% das kcal)", comparando o plano com o que foi de fato registrado no dia (`slotRealTotals`).

**Núcleo reaproveitável:** `addMealItemToSlot(slotId,foodId,grams)` — adiciona à execução com escala de macros, hora real auto no 1º item do dia, e re-ordena alfabético (igual `addFoodToMeal`, mas sem depender de modal/`selectedMealSlot`). É a base do registro de 1 toque. A execução continua sendo `D.days[dt].slots` — registro via plano é idêntico a registrar na mão; o plano (`D.dietPlan`) nunca é alterado pelo ato de comer.

**Verificação:** JavaScriptCore — **28 checks** com funções reais × `dieta-fase1.json`: comi-tudo cria 4 itens no almoço (subst.=frango 1ª opção, não as outras), escolha de substituição (camarão 193g→174kcal/36,7g), aderência comido=previsto, hora real auto, ordem alfabética, bordas (foodId inválido / grama 0 → false). Todos passaram.

**Deploy:** `APP_VERSION`+`sw.js` → **1.9.4**.

**PRÓXIMO:** importar o plano real (passar o JSON da `dieta-fase1.json` pro Gileno colar) e validar no aparelho. Depois: treino (mesmo padrão plano/execução). Possível futuro: aderência do **dia inteiro** no dashboard (hoje é por refeição).

---

## Resumo da sessão 2026-05-31 — v1.9.3 (PLANO ALIMENTAR — Parte 1: sugestão)

**Import da dieta da nutri como SUGESTÃO** (objetivo do Gileno: "dieta como sugestão + execução em paralelo" = planejado vs. realizado). Esta é a **Parte 1 de 2**.

**Arquitetura — 3 camadas:** (1) Plano `D.dietPlan` = prescrição, NOVA estrutura separada; (2) Execução `D.days[dt].slots` = o que comeu (já existia, intacta); (3) Comparação = sobrepõe as duas. Parte 1 entrega (1) + leitura; **Parte 2 (PENDENTE)** = "✓ comi" 1-toque, seletor de substituição, "comi tudo", aderência Plano vs. Comido.

**Estrutura `D.dietPlan`:** `{ meta:{name,source,date,kcalTarget,waterTargetMl,note,importedAt}, slots:[{slotId,name,time,lines:[ {foodId,grams} | {label,options:[{foodId,grams}]} ]}] }`. As `options` modelam as substituições "ou" do PDF (ex.: proteína do almoço = frango/bife/camarão/boi/atum). `slotId` aponta pra campo real de `D.mealSlots`.

**Funções novas** (após `saveMeal`): `getPlanSlot`, `planLineMacros`, `planSlotSubtotal`, `planCardHtml` (card 📋 read-only no topo de cada refeição), `processDietImport` (lógica pura/testável — muta foods/slots/dietPlan), `openDietImport`/`doDietImport` (modal cola-JSON), `clearDietPlan`. CSS: `.plan-card` (borda verde esq.), `.modal-actions`, `.modal-body`.

**Import (cola-JSON, decisão de privacidade do Gileno):** repo é público → plano NÃO vai no código. Botão "Importar" em Metas (card 📋 Plano alimentar) → cola o JSON → vai pro Firestore privado. `processDietImport`: adiciona só foods novos (não duplica por id), casa slots por `match`(id)→nome, cria os que faltam logo após o último resolvido (Sobremesa entra após Almoço), guarda o plano. **Idempotente** (reimport = 0 dups). NÃO mexe em metas (água/kcal) — decisão do Gileno: metas são editáveis pelo usuário.

**Fonte de dados:** `dieta-fase1.json` (gerado por script, **gitignored** — tem dado pessoal). Plano da Débora Teixeira (CRN 29204), FASE 1: 7 refeições, 21 alimentos, total **1.756 kcal** (PDF: 1.764 — dif. arredondamento). PDF só tinha kcal → **macros enriquecidos via TACO** (P/C/G). ⚠️ Proteína do plano ~150g < meta do app (180/160) — registrado, não alterado.

**Verificação:** JavaScriptCore — **31 checks** (sintaxe do arquivo + `processDietImport` REAL × `dieta-fase1.json` real: não-duplicação, criação/ordem de Sobremesa, substituições, subtotais, total 1.756, idempotência). Todos passaram.

**Deploy:** `APP_VERSION`+`sw.js` → **1.9.3**.

**PRÓXIMO (Parte 2):** "✓ comi" copia linha do plano → execução (com seletor de substituição); botão "comi tudo conforme o plano"; indicador de aderência (Plano vs. Comido por refeição e no dia). Depois: treino (mesmo padrão plano/execução).

---

## Resumo da sessão 2026-05-31 — v1.9.2

**Mover alimento/refeição entre os campos** (feedback do Gileno: registrou no Almoço, era Café da manhã). Antes só dava pra apagar e recadastrar. Agora há **duas** formas:

1. **Mover um item** — cada alimento na lista "Itens — X" ganhou um botão ⇄ azul (à direita, longe do × vermelho de remover). Abre um seletor com as outras refeições do dia; o item vai pra escolhida.
2. **Mover a refeição inteira** — botão "⇄ Mover" no cabeçalho do card "Itens — X" move **todos** os itens daquele campo de uma vez.

**Regras:**
- Destinos = `getDaySlots(dt)` menos o slot atual (os mesmos chips que aparecem no topo).
- Ao mover, o destino é re-ordenado alfabético (mantém o ajuste da v1.9.1). Se o destino já tinha itens, faz **merge**.
- **Item individual:** o slot de origem continua existindo (mesmo vazio). **Refeição inteira:** o slot de origem é removido e a navegação vai pro destino; se o destino for novo, herda o **horário real** da origem (preserva quando você comeu).
- Helpers novos: `ensureDaySlot`, `slotDisplayName`, `openMealDestPicker` (modal genérico de destino), `openMoveItem`/`moveMealItem`, `openMoveMeal`/`moveMealAll`. CSS: `.added-move`.

**Verificação:** JavaScriptCore (sandbox sem Node/preview) — parse de sintaxe do arquivo inteiro + **15 checks** rodando o **texto real** das funções de mover + o `getDaySlots` real extraídos do arquivo (destinos sem o slot atual, mover tudo, mover item, merge, preservação de horário, nada perdido, índice inválido sem quebrar). Todos passaram.

**Deploy:** pendente — `APP_VERSION` e `sw.js` em **1.9.2**.

---

## Resumo da sessão 2026-05-31 — v1.9.1

**Ordenação alfabética dos alimentos** (feedback do Gileno). Antes, alimento novo (criado no "+ Novo" ou trazido da TACO) ia pro **fim** da lista. Agora:

1. **Catálogo "Adicionar Alimento"** — renderizado em ordem alfabética (cópia ordenada via `D.foods.slice().sort(byNameAsc)`; a ordem salva em `D.foods` não muda). Vale também para a busca (`filterFoods`).
2. **Itens da refeição montada** — cada slot (Café da manhã, Almoço…) exibe os itens em ordem alfabética. `slot.items` é ordenado **in-place** no render de `rMeal` (o índice continua alinhado com `removeMealItem`) **e** ao adicionar (`addFoodToMeal`, antes do `save()`), então o histórico salvo já fica ordenado.

Comparador único **`byNameAsc`** (definido perto de `fmt`/`fmtN`): `localeCompare(b, 'pt-BR', { sensitivity:'base' })` — ignora acento e maiúscula. Sem migração de dados: refeições antigas se reordenam ao serem abertas.

**Verificação:** JavaScriptCore (sandbox sem Node/preview) — parse de sintaxe do arquivo inteiro + **15 checks** (lógica + funções **reais** `filterFoods`/`foodItemHtml`/`getDayData` em ambiente stub, incluindo alinhamento de índice do `removeMealItem`). Todos passaram.

**Deploy:** pendente — `git push` no repo `gilenogestorsaude.github.io`. `APP_VERSION` e `sw.js` em **1.9.1** → cache busting + reload automático no aparelho.

---

## Resumo da sessão 2026-05-30 — v1.9.0

Ajustes de uso surgidos no dia a dia (feedback do próprio Gileno), focados em **refeições e horários**:

1. **Dias de descanso não mostram mais Pré/Pós-treino** — campos marcados `treinoOnly` somem nos dias `descanso`, mas reaparecem se já houver item registrado neles naquele dia (nunca esconde dado).
2. **Campos de refeição configuráveis** — antes fixos (constante `MEAL_SLOTS`), agora vivem em `D.mealSlots` e são editáveis em Metas → card "🍽 Campos de refeição": renomear, horário previsto, marcar "só treino", ligar/desligar, **reordenar (↑↓), adicionar e remover**. Remoção preserva histórico.
3. **Horário previsto + realizado nas refeições** — cada refeição mostra o previsto (do campo) e a hora real (auto-preenchida ao registrar o 1º alimento no dia de hoje, sempre editável).
4. **Horário previsto + realizado na medicação** — a hora real já era capturada por `toggleMedTaken`; agora é **exibida** (`prev · tomado`) e **editável** (botão 🕐, `editMedTakenTime`).

**Integridade de dados:** `getDayMeals()` passou a somar iterando todos os slots com itens (`Object.keys(day.slots)`), não a config — então customizar/remover campos **nunca** altera os totais de dias passados.

**Regra central** — `getDaySlots(dt)`: exibe um campo se (`ativo` E passa no filtro treino) **OU** se já tem item naquele dia. Helpers novos: `getMealSlots`, `getMealSlotById`, `getDaySlots`, `mealTimeInfo`, `nowHHMM`, `openTimeEditor`, `editMealRealTime`, `editMedTakenTime`, e CRUD de slots (`add/save/delete/toggle/moveMealSlot`).

**Verificação:** sem Node nem preview no browser neste ambiente (sandbox bloqueia o preview por Python) — validado via **JavaScriptCore** (`osascript -l JavaScript`): sintaxe do arquivo inteiro + **14 testes** (lógica real + render de `rDash`/`rMeal`/`rGoals`). Detalhes na memória `gestao-saude-verificacao`.

> **Lacuna documental:** entre v1.4.0 e v1.8.2 houve várias versões (redesign de topo/rodapé, navegação de dias anteriores, sono + IMC com recência, água editável, anéis >100%, etc.) que não foram registradas aqui — ver `git log`. Esta seção retoma a documentação a partir da v1.9.0.

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

- **Frontend:** Vanilla JS + HTML + CSS (sem framework, sem build) — single-file `index.html` ~4780 linhas
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
  userName, peso, altura, metaPeso,
  days: { 'YYYY-MM-DD': { slots: { <slotId>: {items[], time, realTime, name} } } },   // realTime/name v1.9.0
  mealSlots: [{id, name, time, treinoOnly, ativo}],   // campos de refeição configuráveis (v1.9.0); seed em DEFAULT_MEAL_SLOTS
  water: { 'YYYY-MM-DD': [{time, vol, source, auto}] },
  goals: {
    treino:   {kcal, prot, water, carbo?, gord?},
    descanso: {kcal, prot, water, carbo?, gord?}
  },
  dayType: { 'YYYY-MM-DD': 'treino'|'descanso' },     // default: domingo = descanso
  vitals: { 'YYYY-MM-DD': {peso, pa, fc, dormir, acordar} },   // dormir/acordar = sono
  foods: [{id, name, ref, unit, kcal, prot, carbo, gord, nota}],
  meds: [{id, nome, dosagem, horarios[], estoque?, notas?, ativo}],
  medsTaken: { 'YYYY-MM-DD': {medId: {'HH:MM_prescrito': 'HH:MM_real'}} },   // dict prescrito→real (migrado de array)
  treinos: [{id, data, nome, exercicios[{id, nome, series[{reps, carga}]}], notas?, duracao?, createdAt}],
  workoutTemplates: [{id, nome, exercicios[...], notas?, createdAt}],   // Treino A/B/C reutilizáveis
  consultas: [{id, data, medico, especialidade, local?, queixa, conduta, prescricao?, proximaConsulta?, notas?, createdAt}],
  modules: { refeicao, hidratacao, medicamentos, treino, vitais, consultas },   // ligar/desligar áreas
  theme, bphCutoff, bphEnabled, _uiCollapsed
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
pra contexto, e o IDEIAS_PREMIUM.md no mesmo dir pra roadmap. Versão atual em produção: v1.9.0.

[Aqui descreve o que quer fazer: bug encontrado no teste, próxima feature, S6 lançamento, etc]
```

---

## Outros projetos em pausa (do mesmo ecossistema Gileno Gestão)

Não relacionados a Gestão Saúde, mas anotados pra você não perder:

- **VPS 2.0'-D** (`~/Documents/Gileno_Gestao/VPS_Diagnostico/`): instalação CC nativo na VPS Hostinger. Pausada na Fase C (briefing executável pronto, falta executar token longa-duração + smoke DW). Decisão pendente sobre como resolver coleta automatizada do Lado B (extratos bancários) da conciliação Agrodel.

- **Wait state Andressa Chianca:** aguardando resposta dela sobre whitelist IP + CA cert + adesão Bitwarden Send. Email enviado em 22/05 18:05. Deadline 26/05.
