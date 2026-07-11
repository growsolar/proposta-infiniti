# Proposta Comercial — Infiniti Energy (Template HTML/CSS)

Recriação fiel do template de proposta comercial da Infiniti Energy, em **HTML + CSS puro**,
pronto para ser preenchido com dados do CRM e convertido em PDF.

> O PDF original é gerado por **openhtmltopdf** (HTML→PDF). Este template segue o mesmo caminho:
> um HTML + CSS que vira PDF. Não é Word/PowerPoint/Figma.

## Estrutura de arquivos

```
proposta-infiniti/
├── index.html            → as 9 páginas da proposta (com dados de EXEMPLO)
├── style.css             → todo o layout, cores e tipografia
├── proposta-exemplo.pdf  → resultado renderizado (para conferência)
├── README.md             → este arquivo
└── assets/
    ├── logo-infiniti.png     (2000×2000, fundo transparente)
    ├── capa-fundo.png        (594×842, foto de fundo da capa — já traz a logo WEG)
    ├── bancos.png            (logos dos bancos de financiamento)
    └── fonts/                (Poppins 400/500/600/700, embutida offline)
```

## Como visualizar

Basta abrir `index.html` no navegador. Para servir com as fontes/imagens:

```bash
python -m http.server 8080 --directory proposta-infiniti
# abrir http://localhost:8080
```

## Como gerar o PDF

**Opção A — no fluxo do sistema (recomendado):** o backend injeta os dados nos campos e
passa o HTML final para o `openhtmltopdf` (mesma lib do PDF original).

**Opção B — via navegador headless** (usado para gerar o `proposta-exemplo.pdf`):

```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    b = p.chromium.launch(); pg = b.new_page()
    pg.goto("http://localhost:8080/index.html", wait_until="networkidle")
    pg.pdf(path="proposta.pdf", format="A4", print_background=True,
           margin={"top":"0","bottom":"0","left":"0","right":"0"})
    b.close()
```

## Campos dinâmicos × fixos

- **Campos dinâmicos** (mudam por cliente) estão marcados no HTML com o atributo
  `data-field="..."`. Ex.: `data-field="cliente.nome"`, `data-field="proposta.valor_total"`.
  Basta o backend localizar por esse atributo e trocar o conteúdo.
- **Blocos fixos e calculados** estão sinalizados por comentários `<!-- FIXO -->`,
  `<!-- DINÂMICO -->` e `<!-- CALCULADO -->` ao longo do `index.html`.

Campos `data-field` já mapeados:

| data-field | Página | Descrição |
|---|---|---|
| `proposta.data_elaboracao` / `proposta.data_validade` / `proposta.numero` | 1 | datas e nº |
| `cliente.nome` | 1, 9 | nome do cliente |
| `dim.localidade` / `dim.estrutura` / `dim.irradiacao` / `dim.potencia` / `dim.modulos` / `dim.geracao` / `dim.area` | 3 | dimensionamento |
| `proposta.valor_total` | 5 | valor total |
| `analise.tir` / `analise.vpl` / `analise.payback` | 8 | indicadores |
| `amb.co2` / `amb.arvores` / `amb.custo` | 8 | retorno ambiental |

> As **tabelas** (contas, produtos, geração ano-a-ano, retorno do investimento) e os
> **gráficos** são repetições de linhas/barras — no sistema real serão gerados em loop a
> partir dos dados. No `index.html` estão com os valores de exemplo do PDF original.

## Cálculos que o backend precisa fornecer

Os números das páginas 4, 7 e 8 **não são fixos — são calculados**. É preciso implementar
as fórmulas com estes parâmetros (extraídos do PDF original):

- Inflação anual: **4,00%**
- Simultaneidade: **30,00%**
- Degradação dos módulos: até **20% em 25 anos**
- Após 2028: continuidade do pagamento de **90% do Fio B**
- Taxa DI (CDB): **4,40% a.a.** · CDB a **130% do CDI**
- Rendimento da poupança: **3,15% a.a.**
- Indicadores: TIR, VPL (TMA 4%), Payback

## Notas técnicas

- **Formato:** A4 retrato (210×297mm). Cada `<section class="page">` = 1 página.
- **Fonte:** Poppins (embutida em `assets/fonts`, funciona offline e no PDF).
- **Gráficos:** feitos em CSS puro (barras) — funcionam no openhtmltopdf sem JavaScript.
- **Cores da marca:** laranja `#F5851F`, azul `#0E2A72`, azul decorativo `#2436B8`, verde `#3B763D`.
- **Emojis** do retorno ambiental (☁️🌲💲) podem ser trocados por ícones SVG se preferir
  aparência consistente entre sistemas.
