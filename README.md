# pixinsight-workflows

Workflows de processamento de astrofotografia em PixInsight 1.9+, desenvolvidos para o **Observatório X93** (Munhoz, MG, Brasil — hemisfério sul).

O objetivo é documentar de forma precisa e reproduzível cada etapa do processamento, incluindo parâmetros numéricos, justificativas técnicas e decisões condicionais por tipo de alvo.

Sugestões de melhoria são bem-vindas via [Issues](../../issues) ou Pull Requests.

---

## Setup

| Componente | Especificação |
|------------|---------------|
| Telescópio | Takahashi Epsilon 160ED |
| Câmera | ZWO ASI6200MM (sensor Sony IMX455) |
| Filtros | Chroma LRGB + narrowband Ha/OIII/SII 3nm |
| Observatório | X93 — Munhoz, MG, Brasil |
| Software | PixInsight 1.9+ |

---

## Workflows disponíveis

| Workflow | Status | Arquivo |
|----------|--------|---------|
| LRGB broadband | ✅ Completo | [workflows/lrgb/workflow-lrgb-epsilon160-asi6200mm.md](workflows/lrgb/workflow-lrgb-epsilon160-asi6200mm.md) |
| Narrowband SHO | 🚧 Em desenvolvimento | [workflows/narrowband/](workflows/narrowband/) |

---

## Estrutura do repositório

```
pixinsight-workflows/
│
├── README.md
│
├── workflows/
│   ├── lrgb/
│   │   └── workflow-lrgb-epsilon160-asi6200mm.md
│   └── narrowband/
│       └── workflow-sho-epsilon160-asi6200mm.md      ← em desenvolvimento
│
└── process-icons/                                     ← reservado para .xpsm futuros
    └── .gitkeep
```

---

## Ferramentas RC Astro utilizadas

- **BlurXTerminator (BXT)** — deconvolução e correção de PSF
- **NoiseXTerminator (NXT)** — redução de ruído
- **StarXTerminator** — separação de estrelas
- **MAS (MultiscaleAdaptiveStretch)** — stretch não-linear (desde dez/2025)

---

## Notas sobre hemisfério sul

Alguns cuidados específicos para observatórios no hemisfério sul estão documentados nos workflows, em particular:

- Cobertura do catálogo MARS para MGC pode ser incompleta para Dec < −30°
- GC é o fallback recomendado para alvos no plano galáctico sul e narrowband

---

*Observatório X93 — Munhoz, MG, Brasil*
