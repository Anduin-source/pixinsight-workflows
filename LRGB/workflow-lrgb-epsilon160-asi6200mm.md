# Workflow LRGB — PixInsight 1.9+

**Setup:** Takahashi Epsilon 160ED + ASI6200MM — Pier 1  
**Observatório:** X93 — Munhoz, MG, Brasil  
**Última revisão:** Abril 2026

---

## Índice

1. [Canal RGB — Processamento Linear e Stretch](#1-canal-rgb--processamento-linear-e-stretch)
2. [Canal L — Processamento Linear e Stretch](#2-canal-l--processamento-linear-e-stretch)
3. [Combinação Final](#3-combinação-final)
4. [Configurações do Setup](#4-configurações-do-setup)
5. [Tabelas de Referência](#5-tabelas-de-referência)
6. [Regras Inquebráveis](#6-regras-inquebráveis)
7. [Premissas e Limitações](#7-premissas-e-limitações)

---

## 1. Canal RGB — Processamento Linear e Stretch

| # | Etapa | Ferramenta | Observações |
|---|-------|------------|-------------|
| 1 | Stack + Crop + Plate Solving | WBPP | Masters lineares com WCS embutido e bordas limpas. Verificar presença de WCS antes de continuar. |
| 2 | Inspecionar bordas | Visual | DynamicCrop manual se sobraram artefatos de borda. |
| 3 | LinearFit entre R, G, B | LinearFit | Referência = G. Maior SNR, menor contaminação por poluição luminosa. |
| 4 | Channel Combination | ChannelCombination | Gera RGB linear. **NÃO usar LRGBCombination aqui** — opera em espaço Lab, inadequado para dados lineares. |
| 5 | Calibração de fluxo | SPFC | Pré-requisito do MGC. Mesmo que vá usar GC, não prejudica. Ver [config. SPFC](#41-spectrophotometricfluxcalibration-spfc). |
| 6 | Gradient Correction | MGC ou GC | MGC: wide-field, hemisfério norte, fora do plano galáctico. GC: objetos compactos, hemisfério sul (Dec < −30°), plano galáctico (\|b\| < 10°). Ver [tabela de decisão](#51-decisão-mgc-vs-gc). |
| 7 | Calibração de cor | SPCC | Sempre após gradient correction. Ver [config. SPCC](#42-spectrophotometriccolorcalibration-spcc). |
| 8 | Deconvolução leve | BlurXTerminator | Sharpen Stars = 0.25 / Sharpen Nonstellar = 0.40–0.50 / Automatic PSF = marcado. Intensidade menor que no L. |
| 9 | Redução de ruído | NoiseXTerminator | Intensity/Color Separation = **MARCADO** / Denoise = 0.80–0.90 / Iterations = 1. Pode ser agressivo — detalhe vem do L. |
| 10 | Stretch | MultiscaleAdaptiveStretch | Target background = 0.12–0.15. **ANOTAR valor exato** — usar o mesmo no L. Contrast Recovery Intensity = 0.50–0.60 / Scale separation = 512. |
| 11 | SCNR | SCNR | Color = Green / Method = Average Neutral / Amount = 0.70–0.80 / Preserve lightness = marcado. |
| 12 | Ajustes criativos | Curves, ColorSaturation | Ajustes finos de cor e contraste. Não exagerar — o L vai trazer contraste adicional após LRGBCombination. |

---

## 2. Canal L — Processamento Linear e Stretch

| # | Etapa | Ferramenta | Observações |
|---|-------|------------|-------------|
| 1 | Stack + Crop + Plate Solving | WBPP | Mesmo template, mesmo frame de referência dos canais RGB. |
| 2 | Inspecionar bordas | Visual | DynamicCrop manual se necessário. |
| 3 | Calibração de fluxo | SPFC | Somente se usar MGC. Mesma configuração do RGB. |
| 4 | Gradient Correction | MGC ou GC | Mesma escolha do RGB. Para M6 e objetos no plano galáctico sul: usar GC. |
| 5 | Deconvolução forte | BlurXTerminator | Sharpen Stars = 0.50–0.70 / Sharpen Nonstellar = 0.70–0.90 / Automatic PSF = marcado. Intensidade maior que no RGB — L carrega todo o detalhe. |
| 6 | Redução de ruído conservadora | NoiseXTerminator | Intensity/Color Separation = **DESMARCADO** (L é monocromático) / Denoise = 0.65–0.70 / Iterations = 1. Conservador — preservar detalhe. |
| 7 | Stretch | MultiscaleAdaptiveStretch | Target background = **MESMO VALOR DO RGB** (ex: 0.150) / Contrast Recovery Intensity = 0.50 / Scale separation = 512 / Color Saturation = DESMARCADO (mono). |
| 8 | Ajustes de detalhe | Curves, LHE | LocalHistogramEqualization com Amount baixo — fácil de exagerar. |
| 9 | LinearFit L → RGB *(opcional)* | LinearFit | Referência = luminância extraída do RGB. Reject High = 0.85 (evita estrelas gordas). Menos crítico se MAS alinhou os backgrounds com mesmo Target background. Pode ser omitido se backgrounds visualmente compatíveis. |

---

## 3. Combinação Final

| # | Etapa | Ferramenta | Observações |
|---|-------|------------|-------------|
| 1 | Fusão L + RGB | LRGBCombination | **NÃO usar ChannelCombination aqui.** Apenas L marcado, R/G/B desmarcados. Chrominance NR = ATIVADO / Saturation = 0.5. Aplicar sobre a imagem RGB. |
| 2 | Ajustes finais | Curves, ColorSaturation | Saturação, hue, contraste final. |

---

## 4. Configurações do Setup

### 4.1 SpectrophotometricFluxCalibration (SPFC)

| Campo | Valor | Observação |
|-------|-------|------------|
| QE curve | Sony IMX411/455/461/533/571 | Sensor IMX455 da ASI6200MM |
| Gray filter | Astronomik UV-IR Block L-2 | Melhor opção para Chroma L |
| Red filter | Chroma R | |
| Green filter | Chroma G | |
| Blue filter | Chroma B | |
| Narrowband filters mode | Desmarcado | LRGB broadband |
| Generate graphs | Marcado | Verificar qualidade da calibração |
| Catalog | Gaia DR3/SP | |
| Automatic limit magnitude | Marcado | |

### 4.2 SpectrophotometricColorCalibration (SPCC)

| Campo | Valor | Observação |
|-------|-------|------------|
| QE curve | Sony IMX411/455/461/533/571 | Mesmo do SPFC |
| Red filter | Chroma R | |
| Green filter | Chroma G | |
| Blue filter | Chroma B | |
| White reference | Photon Flux | Para aglomerados estelares (ex: M6) |
| | Average Spiral Galaxy | Para nebulosas e galáxias |
| Background Neutralization | Marcado | Neutraliza fundo junto com calibração |
| Catalog | Gaia DR3/SP | |
| Automatic limit magnitude | Marcado | |
| Narrowband filters mode | Desmarcado | |

### 4.3 BlurXTerminator (BXT)

| Parâmetro | Canal RGB | Canal L |
|-----------|-----------|---------|
| Sharpen Stars | 0.25 | 0.50–0.70 |
| Adjust Star Halos | 0.00 | 0.00 |
| Automatic PSF | Marcado | Marcado |
| Sharpen Nonstellar | 0.40–0.50 (leve) | 0.70–0.90 (forte) |
| Correct Only | Desmarcado | Desmarcado |
| Luminance Only | Desmarcado | Desmarcado |

### 4.4 NoiseXTerminator (NXT)

| Parâmetro | Canal RGB | Canal L |
|-----------|-----------|---------|
| Intensity/Color Separation | **MARCADO** | **DESMARCADO** (mono) |
| Denoise Intensity | 0.80–0.90 (agressivo) | 0.65–0.70 (conservador) |
| Denoise Color | 0.80–0.90 | N/A — modo mono |
| Iterations | 1 | 1 |

### 4.5 MultiscaleAdaptiveStretch (MAS)

| Parâmetro | Valor | Observação |
|-----------|-------|------------|
| Target background | 0.12–0.15 | **USAR O MESMO VALOR** no RGB e no L |
| Aggressiveness | 0.70 | |
| Dynamic range compression | 0.40 | |
| Contrast Recovery | Marcado | Intensity = 0.50–0.60 / Scale sep. = 512 |
| Color Saturation | Marcado (RGB) / Desmarcado (L) | Desmarcado no L — mono sem saturação |

---

## 5. Tabelas de Referência

### 5.1 Decisão MGC vs GC

| Critério | Use MGC | Use GC |
|----------|---------|--------|
| Telescópio | Epsilon 160 (wide-field) | RC 12" (objetos compactos) |
| Alvo | Nebulosas extensas, cirros, IFN, campos difusos | Galáxias isoladas, aglomerados, planetárias |
| Fundo no quadro | Pouco fundo limpo disponível | Fundo dominante e limpo |
| Declinação | Hemisfério norte — boa cobertura MARS | Hemisfério sul (Dec < −30°) — cobertura MARS incompleta |
| Posição galáctica | Fora do plano (\|b\| > 10°) | Plano galáctico (\|b\| < 10°) |
| Pré-requisitos | ImageSolver + SPFC + XMARS (~1.35 GB) | Nenhum adicional |
| Risco | Pode falhar sem cobertura MARS | Baixo — fallback confiável |

### 5.2 Os dois LinearFits — não confundir

| LinearFit | Momento | Referência | Objetivo |
|-----------|---------|------------|----------|
| LinearFit 1 | Entre R, G, B antes do ChannelCombination | Canal G | Igualar nível de fundo dos canais de cor entre si |
| LinearFit 2 *(opcional)* | Entre L e RGB antes do LRGBCombination | Luminância extraída do RGB | Igualar brilho do L ao perfil do RGB. Reject High = 0.85 |

### 5.3 Os dois Channel Combinations — não confundir

| Processo | Momento | Espaço de cor | Objetivo |
|----------|---------|---------------|----------|
| ChannelCombination | R+G+B lineares → RGB linear | RGB direto | Montar a imagem colorida linear |
| LRGBCombination | L+RGB esticados → LRGB final | CIE L\*a\*b\* | Fundir luminância processada no RGB já esticado |

### 5.4 Distribuição de Tempo de Captura LRGB

| Tipo de alvo | L | RGB total | Justificativa |
|--------------|---|-----------|---------------|
| Aglomerado aberto (ex: M6) | 40% | 60% | Cor das estrelas é o elemento principal |
| Nebulosa de emissão/reflexão | 50% | 50% | Equilíbrio padrão |
| Galáxia com estrutura fina | 60–70% | 30–40% | Detalhe estrutural domina |
| Objeto muito fraco | 70% | 30% | Máximo SNR para detalhe sutil |

---

## 6. Regras Inquebráveis

| Regra | Motivo |
|-------|--------|
| Gradient Correction **antes** de SPCC | SPCC precisa de fundo plano para calibrar cor corretamente |
| SPFC **antes** de MGC | MGC precisa de fluxo físico para comparar com MARS |
| BXT **sempre antes** de NXT | BXT precisa do ruído gaussiano original para funcionar corretamente |
| NXT no RGB combinado, nunca canal por canal | Modo Intensity/Color Separation só funciona em imagem colorida |
| ChannelCombination para R+G+B lineares | LRGBCombination opera em Lab — inadequado para dados lineares |
| LRGBCombination para L+RGB esticados | ChannelCombination não faz fusão correta em espaço Lab |
| Mesmo Target background no MAS (L e RGB) | Garante compatibilidade no LRGBCombination sem ajuste manual |
| Tudo linear **antes** do MAS | Após o stretch o ruído muda de natureza — processos lineares perdem validade |
| NXT no L com Denoise conservador (0.65–0.70) | L carrega o detalhe — redução agressiva destrói informação irrecuperável |
| BXT mais forte no L que no RGB | L carrega o detalhe — deconvolução forte só aqui. RGB recebe tratamento leve |

---

## 7. Premissas e Limitações

- WBPP configurado com plate solving habilitado e parâmetros corretos de escala de placa por rig.
- XMARS database baixado (~1.35 GB) e configurado no MGC (necessário apenas para MGC).
- Catálogo Gaia DR3/SP instalado localmente para SPFC e SPCC.
- Em céu com poluição luminosa forte, NXT linear pode causar manchas após o stretch — nesse caso mover NXT para após o MAS.
- MAS é ferramenta nova (dez/2025) — se resultado não agradar, GHS é alternativa robusta e estabelecida.
- Para narrowband (Ha, OIII, SII): workflow muda — cobertura MARS pode ser limitada, e SPFC requer curvas de transmissão configuradas por filtro.
- MGC não recomendado para alvos no hemisfério sul profundo (Dec < −30°) ou no plano galáctico (|b| < 10°) — usar GC nesses casos.
- LinearFit 2 (L → RGB) é opcional se MAS foi aplicado com o mesmo Target background nos dois canais.

---

*Observatório X93 — Munhoz, MG, Brasil | PixInsight 1.9+ | Abril 2026*
