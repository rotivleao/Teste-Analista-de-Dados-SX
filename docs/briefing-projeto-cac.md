# Briefing – Estrutura de dados de Aquisição (CAC)

## 1. Contexto e objetivo
- Meu gerente pediu o desenho de uma estrutura de dados ideal para o time de marketing/growth (Globo / Globoplay), no BigQuery.
- **Objetivo principal:** CAC. Também: CPA, CPC, CPM, CPV, CTR e CPI.
- **Entrega:** apenas o DESENHO (organograma, diagrama e estrutura de dados). **Não quero código SQL.**
- **Estilo:** rico, mas sóbrio e transparente para o gerente: poucas cores, uma por tabela de origem, sem excesso de informação.
- **Ferramenta:** board FigJam https://www.figma.com/board/vRgbrwP3PCqN3ANdE1M8dF (já tem a seção "1 · Organograma de montagem").

## 2. Escopo: só 4 tabelas
| Tabela | Papel |
|---|---|
| `gglobo-mkt-ins-hdg-prd.gflow.gflow_media_unified` | Investimento (custo). **Base do cruzamento.** |
| `gglobo-gplay-aquisicao-hdg-prd.Database_dashboard_growth.Base_GA4_Bronze` | Conversões web (GA4) |
| `gglobo-gplay-aquisicao-hdg-prd.Database_dashboard_growth.Base_Appsflyer_Bronze` | Conversões app (AppsFlyer) |
| `gglobo-mkt-ins-hdg-prd.reports.vendas_sf` | Vendas Salesforce. **Só para comparativo de CAC por canal** (sei que não conversa com as UDFs) |

## 3. Detalhe das tabelas

### gflow_media_unified (pré-clique)
- **Dimensões:** `day` (data), `bu_name` (conta de origem), `channel` (fonte: Google Ads, Meta Ads, TikTok Ads, Kwai, DV360, Twitter Ads, Amazon dsp, Amazon firetv, RTB House, Criteo, Afilio, Rocket Lab, Winclap), `campaign_id`/`campaign_name` (campanha), `ad_group_id`/`ad_group_name` (ad_group), `ad_id`/`ad_name` (content), `campaign_type`, `account_id`/`account_name`, `territorio`.
- **Métricas:** `cost` (sempre BRL), `impressions`, `clicks`, `video_views`.
- **Decisões:**
  - `ad_name` vazio vira "conteúdo desconhecido".
  - Métricas convertidas para inteiro.
  - `total_days_available` é ignorado.
  - Não precisa de timestamp.

### Base_GA4_Bronze (web)
- **Colunas:** `data`, `source_medium`, `campaign`, `content`, `ad_group`, `hostname`, `pagina_vitrine`, `stream_name_update`, `platform`, `periodicidade_hit_oficial`, `territorio`, `canal_agrupado`, `canal_total`, `canal_mkt`, `produto`, `nome_produto`, `user_pseudo_id`, `begin_checkout`, `add_to_cart`, `purchase`.
- `ad_group` só vem preenchido para Google Ads.

### Base_Appsflyer_Bronze (app)
- **Colunas:** `data`, `source_medium` (media_source), `campaign`, `content` (af_ad), `ad_group` (af_adset), `canal_agrupado`, `canal_total`, `canal_mkt`, `territorio`, `produto`, `nome_produto`, `user_pseudo_id` (customer_user_id), `pdp_experimento`, `add_to_cart` (checkout_form), `purchase` (deduplicado), `install`, `endpoint` (paid / retarget / organic), `Upgrade_venda_direta`.

### vendas_sf
- **Colunas:** `dt_venda`, `ts_venda`, `globo_id`, `nome_produto`, `produto`, `canal_compra` (WEB/APP), `origem_id`, `area_growth` (Mídia Paga, CRM, Social…), `canal` (da planilha de origem), `tipo_venda` (venda_direta, trial, cross_up, parceiro), `preco`.
- O canal vem da planilha "[2024] Repositório de OrigemIDs (Martech)" (aba TODOS, coluna `area_mkt`), cruzada pelo `origem_id`.

## 4. UDFs (projeto `gglobo-gplay-aquisicao-hdg-prd.Functions`)
| UDF | Usada em | Resultado |
|---|---|---|
| `case_when_canais_geral_mkt` | GA4 | canal_agrupado |
| `case_when_canais_geral_mkt_agrupado` | GA4 | canal_total (Meta, Google, Kwai…) |
| `case_when_canais_mkt_final` | GA4 | canal_mkt (Mídia paga, CRM, Social, Search, Direto, Mídia Orgânica, Outros) |
| `case_when_canais_geral_mkt_AF` | AppsFlyer | canal_agrupado / canal_total |
| `case_when_canais_geral_mkt_agrupado_AF` | AppsFlyer | canal_mkt |
| `case_when_territorio(content, ad_group, campaign)` | GA4, AppsFlyer e **aplicar na gflow** | território (Esporte, Filme, Série, Novela, Reality, Institucional…). Precedência: content → ad_group → campaign |
| `case_when_tipo_campanha(content, ad_group, campaign)` | **aplicar** em gflow, GA4 e AppsFlyer | Performance / Awareness / Consideração. Precisa de melhoria |

Falta criar um **de-para de `channel` da gflow para o `canal_total`** (Google Ads → Google, Meta Ads → Meta, TikTok Ads → Tiktok…).

## 5. Regra do cruzamento (o coração do CAC)
Cruzar `gflow_media_unified` contra as demais tabelas pelas chaves **data, campanha, ad_group e content**, em níveis:

| Nível | Chave | Participa | Leitura |
|---|---|---|---|
| 1 · Canal | data + canal | gflow, GA4, AppsFlyer **e vendas_sf** | CAC por canal (atribuição × Salesforce) |
| 2 · Campanha | + campanha | gflow, GA4, AppsFlyer | tipo de campanha |
| 3 · Ad group | + ad_group | gflow, AppsFlyer (GA4 só Google Ads) | – |
| 4 · Content | + content | gflow, GA4, AppsFlyer | território |

**Regras:**
- **Aquisição** = compras de venda direta.
- **Upgrade** fica fora.
- **Retarget** NÃO entra no CAC.
- `globo_id` (GA4) = `customer_user_id` (AppsFlyer) = `globo_id` (Salesforce).
- Datas no fuso America/Sao_Paulo.
- Normalizar as chaves e somar cada lado na mesma chave antes de cruzar.
- O cruzamento é completo: custo sem venda e venda sem custo aparecem.

## 6. Taxonomia de nomes (referência)
- **utm_campaign** (10 posições, separadas por "_"): origem, direcionamento, funil, formato da campanha, web/app, produto, conteúdo, temporada, território, nome da campanha.
- **utm_ad_group**: conteúdo, temporada, formato, detalhamento do anúncio.
- **utm_content**: conteúdo, temporada, território, funil, origem, formato do criativo, secundagem, detalhamento, plano, produto.
- "na" = não se aplica.
- Os nomes de mídia paga (ex.: `aq_google_app-android_perf_...`) parecem seguir outra versão. Fica para depois.

## 7. Entregas esperadas (no FigJam)
1. **Organograma de montagem:** tabelas → UDFs → cruzamento com a gflow → 4 níveis → CAC/CPA/CPC… *(feito; conferir se há texto cortado)*
2. **Diagrama de chaves:** as 4 tabelas com as colunas-chave, ligadas à gflow, com a chave escrita em cada ligação (vendas_sf tracejada, só canal).
3. **Estrutura de dados:** dicionário de cada tabela: coluna, tipo, papel (chave / métrica / atributo) e uso no cruzamento.

## 8. Pendências (a confirmar)
- **Retarget:** como identificar o custo de retarget na gflow, para excluir do CAC.
- **Trial:** entra ou não como aquisição.
- **Schema real:** print do schema e 5 linhas de exemplo de GA4_Bronze, Appsflyer_Bronze e vendas_sf.
- **Valores de canal:** lista de valores distintos de `channel` (gflow), `canal_total`/`canal_mkt` (GA4 e AppsFlyer) e `area_growth`/`canal` (vendas_sf), para fechar o de-para.

## 9. Pontos de atenção já identificados nas bases
- **UDF de canal do GA4:** classifica DV360 como Google.
- **UDFs de canal da AppsFlyer:** "Outros Midia paga", "Midia Organica" e "Smart snackbar" caem em "Outros".
- **GA4:** `purchase_WEB` não filtra o stream WEB (risco de contar duas vezes). As compras são contadas por evento, não por `transaction_id`.
- **AppsFlyer:** a deduplicação perde o horário do evento. Compras sem `customer_user_id` são deduplicadas juntas.
- **vendas_sf:** a data está em UTC. O join de primeiro play pode duplicar vendas. Um `origem_id` duplicado na planilha também duplica vendas.
