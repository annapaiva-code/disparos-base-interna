# Disparos Base Interna — Dashboard de Funil

## O que é
Dashboard HTML de funil de conversão dos disparos MIA (SZI + Marketplace + Gestão/SZS).
Cobre leads gerados por campanhas de WA via MIA a partir de 11/06/2026.

## Links
- **Público:** https://disparos-base-interna.vercel.app
- **GitHub:** https://github.com/annapaiva-code/disparos-base-interna (branch `main`)
- **Drive:** "Disparos SZI — Funil Diário" (sobrescreve diariamente)
- **Arquivo local:** `C:\Users\compu\Desktop\disparos_szi_funil.html`

## Rotina automática
- **Trigger:** `trig_01Y2KJjp3bipYmAyjzK1WfE5`
- **Horário:** `0 10 * * *` = 7h BRT todo dia
- Puxa `nekt_operacional_silver.pipedrive_marketing_deals_disparos_mia`
- Gera HTML → salva no Drive → push GitHub → Vercel auto-deploya

## Estrutura das abas
| Aba | ID | Cor |
|-----|-----|-----|
| SZI | `tab-szi` | azul `#3b82f6` |
| Marketplace | `tab-mkt` | roxo `#8b5cf6` |
| Gestão / Outros | `tab-out` | âmbar `#f59e0b` |
| Todos | `tab-all` | neutro |

Cada aba tem: funil do mês → 3 cards insight → tabela de campanhas → seção SQL→OPP ⚡ (análise do mês).

## Dados
- **Fonte (desde 27/07/2026):** Nekt → `pipedrive_deals_readable` + joins bronze
  (`pipedrive_pipelines`, `pipedrive_stages`), filtro `rd_campanha LIKE '%_MIA%' OR LIKE '%_SIA%'`.
  A lógica está inlinada em `C:\Users\compu\disparos-auto\atualizar.mjs` (const `DISPAROS`).
- **NÃO usar mais `pipedrive_marketing_deals_disparos_mia`:** a transformação que a gera
  (`query-qB7h`) filtra só `rd_campanha LIKE '%_MIA%'` e é **cega aos disparos da SIA**, que
  substituiu a MIA em jul/2026 (última campanha MIA 24/07; SIA desde 17/07 na Gestão/SZS e
  27/07 no SZI). Em 27/07 a tabela pronta perdia 447 leads, 34 SQL e 13 OPP do mês.
- **Mês corrente é reconstruído a cada rodada** (não mais congelado por dia): SQL/OPP/WON são
  preenchidos dias depois da criação do lead, então o snapshot congelado subcontava para sempre
  (julho marcava 34 OPPs contra 82 reais). `--congelar` volta ao comportamento antigo.
- **Período:** a partir de `2026-06-11` (atualizar no prompt da rotina ao iniciar nova campanha)
- **Pipelines:** Vendas Spot = SZI · Marketplace = MKT · outros = Gestão/Outros
- **ATENÇÃO — bug de case no cálculo de "Responderam":** `stage_name` do estágio inicial
  vem como `'Lead in'` (i minúsculo) nos pipelines Vendas Spot, Comercial SZS e Comercial
  Decor, mas como `'Lead In'` (I maiúsculo) no pipeline Marketplace. A fórmula
  `responderam = COUNT(CASE WHEN stage_name != 'Lead In' THEN 1 END)` é case-sensitive no
  Athena e portanto SUBESTIMA quem não respondeu (mostra ~100% de resposta) para
  SZI/Gestão/Decor. Use sempre `lower(stage_name) != 'lead in'` para o cálculo de
  "responderam" em qualquer frente. (Descoberto e corrigido retroativamente em 02/07/2026 —
  ver commit "Corrigir bug de case-sensitivity no cálculo de Resp".)

## SDRs vs Closers
- **SDRs (Pré-venda):** Jeniffer Correa, Karoane Soares, Cesar Araujo — fazem cadência de contato via MIA até o agendamento
- **Closers:** Luana, Hellen, Jonathan — atuam após a reunião acontecer

## Vercel
- Projeto: `disparos-base-interna` (team `annapaiva`)
- Token: `C:\Users\compu\AppData\Roaming\xdg.data\com.vercel.cli\auth.json`
- Deploy via push no GitHub (auto-deploy configurado)
