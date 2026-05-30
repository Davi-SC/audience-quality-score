# Indicador Multidimensional de Qualificação de Audiência no Instagram

> **Artigo associado:** _Indicador Multidimensional para Qualificação de Audiência de Perfis no Instagram_  
> **Autor:** Davi Silva da Cruz, Francisco Alan de Oliveira Santos
> **Instituição:** Instituto Federal do Maranhão (IFMA) - Campus Coelho Neto

---

## Visão Geral

Este repositório contém a implementação completa do pipeline de dados descrito no Trabalho de Conclusão de Curso (TCC) que propõe e valida um **indicador multidimensional para qualificação de audiência de perfis no Instagram**.

O indicador combina duas dimensões complementares:

| Dimensão                        | Métrica                                        | Interpretação                                              |
| ------------------------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| **Nível de engajamento**        | Mediana da Taxa de Engajamento (TE) por perfil | Interação típica da audiência com as publicações           |
| **Estabilidade do engajamento** | Coeficiente de Variação (CV) da TE por perfil  | Consistência temporal do engajamento ao longo do histórico |

Essas dimensões são normalizadas em **percentis de rank intrassegmento** (categoria × porte de seguidores) e combinadas em um **score composto**, que classifica cada perfil em quatro níveis de qualificação: **A**, **B**, **C** e **D**.

---

## Base de Dados

O estudo utiliza o conjunto de dados de influenciadores do Instagram disponibilizado por:

> **Kim, S. et al.** _Multimodal Post Attentive Profiling for Influencer Marketing._ WWW '20, ACM, 2020.  
> Disponível em: [https://doi.org/10.1145/3366423.3380052](https://doi.org/10.1145/3366423.3380052)

A base contém:

- **~50.000 influenciadores** rotulados em 9 categorias (beauty, fashion, fitness, food, travel, family, interior, pet, other)
- **Até 300 publicações por perfil**, com metadados de engajamento (curtidas, comentários, tipo de post, patrocínio, hashtags, etc.)
- Cobertura temporal de **2012 a 2019** (análise restrita a **2017–2019**)

> **Nota:** Os dados brutos do dataset original de **Kim et al. (2020)** não estão incluídos neste repositório por questões de tamanho e licenciamento. O pesquisador deve obtê-los diretamente dos autores originais. Você pode solicitar os dados brutos por meio do [formulário](https://sites.google.com/site/sbkimcv/dataset/instagram-influencer-dataset#h.m325vwh142wd).

---

## Arquitetura do Pipeline

O fluxograma abaixo ilustra as etapas do pipeline, desde a extração dos dados brutos até a validação estatística do indicador:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PIPELINE DE IMPLEMENTAÇÃO                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. EXTRAÇÃO (extracao_dados.ipynb)                                 │
│     ├── Leitura dos arquivos .info (JSON) do dataset Kim et al.     │
│     ├── Extração paralela com ThreadPoolExecutor                    │
│     ├── Geração de posts.parquet e comments.parquet                 │
│     └── Extração do dataframe_influencers.csv                       │
│                         │                                           │
│                         ▼                                           │
│  2. PROCESSAMENTO (processamento_posts.ipynb)                       │
│     ├── Critério 1: Exclusão de followers = 0 ou nulo               │
│     ├── Critério 2: Filtro temporal 2017–2019                       │
│     ├── Critério 3: Exclusão de posts sem interação                 │
│     ├── Cálculo da TE (clássica e ponderada)                        │
│     ├── Segmentação por porte (nano → mega)                         │
│     ├── Cálculo do CV por perfil                                    │
│     └── Gravação das camadas (base, core, cv)                       │
│                         │                                           │
│                         ▼                                           │
│  3. EDA (eda.ipynb)                                                 │
│     ├── Estatísticas descritivas                                    │
│     ├── Distribuição por segmento e categoria                       │
│     └── Análise de correlação TE × CV                               │
│                         │                                           │
│                         ▼                                           │
│  4. QUALIFICAÇÃO (perfil_qualificacao.ipynb)                        │
│     ├── Critérios de elegibilidade (≥30 posts, ≥10 válidos, etc.)   │
│     ├── Rank percentil intra-segmento (TE e CV)                     │
│     ├── Score composto: 0.60·rank_ER + 0.40·(1 − rank_CV)          │
│     ├── Classificação em tiers (A, B, C, D) por quartis             │
│     └── Análise de sensibilidade dos pesos                          │
│                         │                                           │
│                         ▼                                           │
│  5. VALIDAÇÃO (validacao_hipoteses.ipynb)                           │
│     ├── Teste de normalidade (Shapiro-Wilk)                         │
│     ├── H1: Kruskal-Wallis + Dunn (TE entre tiers)                 │
│     ├── H2: Kruskal-Wallis + Dunn (CV entre tiers)                 │
│     └── Visualizações estatísticas (boxplots, forest plots, etc.)   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Estrutura do Repositório

```
influencers_kim/
│
├── extracao_dados.ipynb          # Etapa 1: Extração dos dados brutos (.info → Parquet)
├── processamento_posts.ipynb     # Etapa 2: Pré-processamento, métricas e segmentação
├── eda.ipynb                     # Etapa 3: Análise exploratória de dados
├── perfil_qualificacao.ipynb     # Etapa 4: Construção do score e classificação em tiers
├── validacao_hipoteses.ipynb     # Etapa 5: Validação estatística das hipóteses
│
├── pyproject.toml                # Configuração do projeto e dependências (uv)
├── uv.lock                       # Lockfile de dependências
├── .python-version               # Versão do Python (3.13)
├── .gitignore                    # Regras de exclusão do Git
```

> **Nota:** Arquivos `.parquet` e `.csv` gerados pela execução sequencial dos notebooks estão no `.gitignore` por questões de tamanho.

---

## Pré-requisitos

| Requisito                        | Versão                                     |
| -------------------------------- | ------------------------------------------ |
| Python                           | ≥ 3.13                                     |
| [uv](https://docs.astral.sh/uv/) | ≥ 0.7 (gerenciador de pacotes e ambientes) |

O projeto utiliza o **uv** como gerenciador de dependências. Se você ainda não o tem instalado:

```bash
# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# Linux / macOS
curl -LsSf https://astral.sh/uv/install.sh | sh
```

---

## Instalação e Configuração

1. **Clone o repositório:**

```bash
git clone https://github.com/Davi-SC/influencers_kim.git
cd influencers_kim
```

2. **Crie o ambiente virtual e instale as dependências:**

```bash
uv sync
```

3. **Abra o Jupyter Notebook:**

```bash
uv run jupyter notebook
```

---

## Guia de Execução — Passo a Passo

Os notebooks devem ser executados **na ordem sequencial** descrita abaixo. Cada etapa depende dos artefatos gerados pela etapa anterior.

---

### Etapa 1 — Extração de Dados (`extracao_dados.ipynb`)

**O que faz:**

1. Lê o arquivo `influencers.txt` (metadados dos influenciadores separados por TAB) e gera `dataframe_influencers.csv`
2. Percorre recursivamente todos os arquivos `.info` do dataset original usando `ThreadPoolExecutor` com processamento paralelo
3. Para cada publicação, extrai: identificação, métricas de engajamento, tipo de post, patrocínio, hashtags, etc.
4. Grava os resultados em batches de 100.000 registros, consolidando em `posts.parquet` e `comments.parquet`

**Pré-condição:** O dataset original de Kim et al. (2020) deve estar disponível localmente. Os caminhos `BASE`, `SAIDA` e `TXT` nas células de configuração devem ser ajustados para o diretório local do pesquisador.

---

### Etapa 2 — Processamento de Posts (`processamento_posts.ipynb`)

**O que faz:**

1. **Critério 1:** Remove perfis com `followers = 0` ou nulo
2. **Critério 2:** Filtra publicações do período **2017–2019**
3. **Critério 3:** Exclui publicações sem nenhuma interação (likes = 0 **e** comments = 0)
4. Calcula a **Taxa de Engajamento (TE)** clássica e ponderada por post:
   - `TE = (curtidas + comentários) / seguidores × 100`
5. Realiza a **segmentação por porte** (bucket de seguidores):

   | Segmento | Faixa de seguidores |
   | -------- | ------------------- |
   | Nano     | ≤ 10.000            |
   | Micro    | 10.001 – 100.000    |
   | Mid-tier | 100.001 – 500.000   |
   | Macro    | 500.001 – 1.000.000 |
   | Mega     | > 1.000.000         |

6. Calcula o **Coeficiente de Variação (CV)** da TE por perfil: `CV = σ(TE) / μ(TE)`
7. Gera as camadas do dataset:
   - `base_completa` → todos os posts com followers > 0
   - `base_2017_2019` → filtro temporal aplicado
   - `core_2017_2019` → base robusta com comments > 0

---

### Etapa 3 — Análise Exploratória (`eda.ipynb`)

**O que faz:**

1. Carrega as bases processadas (`base_2017_2019` e `core_2017_2019`)
2. Calcula estatísticas descritivas gerais (nº de posts, perfis únicos, período, tipos de post)
3. Analisa a distribuição do engajamento por segmento e categoria
4. Gera heatmaps de cobertura de perfis por `(categoria × bucket_followers)`
5. Analisa a correlação entre TE e CV
6. Gera visualizações para o artigo (mapas de calor, boxplots, histogramas)

---

### Etapa 4 — Perfil de Qualificação (`perfil_qualificacao.ipynb`)

**O que faz:**

1. **Critérios de elegibilidade:** Exige ≥ 30 posts totais, ≥ 10 posts válidos para CV, perfil completo no lookup, categoria e bucket definidos, CV não-nulo
2. **Rank percentil intra-segmento:** Para cada perfil elegível, calcula o rank percentil da `TE mediana` e do `CV` dentro do seu segmento `(categoria × bucket_followers)`
3. **Score composto:**
   ```
   score = W_ER × rank_percentil(TE) + W_CV × (1 − rank_percentil(CV))
   ```
   Onde `W_ER = 0.60` e `W_CV = 0.40`. O CV é invertido porque menor CV (maior estabilidade) é desejável.
4. **Classificação em tiers** por quartis globais do score:

   | Tier | Quartil      | Interpretação                                      |
   | ---- | ------------ | -------------------------------------------------- |
   | A    | Q4 (75–100%) | Alta qualificação — engajamento elevado e estável  |
   | B    | Q3 (50–75%)  | Qualificação moderada-alta                         |
   | C    | Q2 (25–50%)  | Qualificação moderada-baixa                        |
   | D    | Q1 (0–25%)   | Baixa qualificação — engajamento baixo ou instável |

5. **Análise de sensibilidade dos pesos:** Recalcula o score em 4 cenários de pesos (40/60, 50/50, 60/40, 70/30) e avalia:
   - Correlação de Spearman entre rankings (> 0.89 em todos os pares)
   - Proporção de perfis que mantêm o mesmo tier (63,67% em todos os cenários)

---

### Etapa 5 — Validação de Hipóteses (`validacao_hipoteses.ipynb`)

**Hipóteses testadas:**

| Hipótese | Descrição                                                          | Resultado                                            |
| -------- | ------------------------------------------------------------------ | ---------------------------------------------------- |
| **H1**   | As distribuições de TE diferem significativamente entre os 4 tiers | Confirmada (Kruskal-Wallis H = 19.620,86; p < 0,001) |
| **H2**   | As distribuições de CV diferem significativamente entre os 4 tiers | Confirmada (Kruskal-Wallis H = 13.277,43; p < 0,001) |

**Testes estatísticos aplicados:**

1. **Shapiro-Wilk** - Verificação de normalidade (rejeitada justifica testes não-paramétricos)
2. **Kruskal-Wallis** - Teste omnibus para diferenças entre os 4 grupos
3. **Post-hoc de Dunn** com correção de Bonferroni - Comparações par-a-par entre tiers (todos significativos com p < 0,001)

---

## Métricas e Indicadores Utilizados

### Taxa de Engajamento (TE)

```
TE_post = (Curtidas + Comentários) / Seguidores × 100
```

Para análises em nível de perfil, utiliza-se a **mediana** da TE das publicações (robusta a outliers).

### Coeficiente de Variação (CV)

```
CV_perfil = σ(TE) / μ(TE)
```

Mede a dispersão relativa: valores baixos indicam engajamento estável; valores altos indicam volatilidade.

### Score de Qualificação

```
Score = 0.60 × rank_percentil(TE_mediana) + 0.40 × (1 − rank_percentil(CV))
```

Os ranks percentis são calculados **dentro de cada segmento** (categoria × bucket de seguidores), garantindo comparabilidade justa entre perfis de diferentes portes e nichos.

## Referência Bibliográfica Principal

```bibtex
@inproceedings{kim2020multimodal,
  title     = {Multimodal Post Attentive Profiling for Influencer Marketing},
  author    = {Kim, Seungbae and Han, Jinyoung and Yoo, Seunghyun and Gerber, Matthew},
  booktitle = {Proceedings of The Web Conference 2020 (WWW '20)},
  pages     = {2878--2884},
  year      = {2020},
  publisher = {Association for Computing Machinery},
  doi       = {10.1145/3366423.3380052}
}
```

---

## Licença

Este projeto é parte de um Trabalho de Conclusão de Curso (TCC) do Instituto Federal do Maranhão — Campus Coelho Neto. O código é disponibilizado para fins acadêmicos e de pesquisa.
