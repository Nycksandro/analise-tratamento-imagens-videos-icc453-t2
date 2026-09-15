# Detecção de Cortes de Cena e Seleção de Quadros-Chave (Shot Boundary Detection & Keyframes)

Este projeto realiza a **detecção automática de cortes de cena** em vídeos e a **extração de quadros-chave (*key frames*)** para cada cena identificada. O sistema utiliza duas abordagens de descritores visuais — **Histograma Local** e **BIC (Border/Interior Pixel Classification) Local** — com partição espacial dos quadros.

---

## Estrutura do Repositório

Ao final da execução completa do pipeline, todas as pastas do projeto estarão populadas conforme a estrutura abaixo:

```text
imagens_tp2/
 ┣ 📂 cortes_chaves            # Arquivos .txt contendo os quadros-chave calculados
 ┣ 📂 cortes_identificados     # Arquivos .txt contendo os quadros onde cortes foram detectados
 ┣ 📂 cortes_manuais           # Referência manual dos cortes (Ground Truth) em formato .txt
 ┣ 📂 docs
 ┃ ┗ 📜 TP02_Deteccao_Cortes.pdf  # Especificação técnica e diretrizes do trabalho
 ┣ 📂 estatisticas             # Relatórios com métricas de acurácia, matrizes e erros (TP, FP, FN, VN)
 ┣ 📂 videos                   # Arquivos de vídeo originais em formato .mp4
 ┣ 📜 detector_corte_BIC.py            # Algoritmo de detecção de cortes via BIC Local
 ┣ 📜 detector_corte_histograma.py     # Algoritmo de detecção de cortes via Histograma Local
 ┣ 📜 estatisticas_quadros.py          # Script de cálculo de métricas comparativas com o Ground Truth
 ┣ 📜 faz_tudo_de_uma_vez.py           # Pipeline de automação (executa todos os testes em lote)
 ┣ 📜 README.md                        # Documentação do repositório
 ┗ 📜 selecinaQuadro.py                # Ferramenta interativa OpenCV para marcação manual de quadros
```

---

Aqui está a seção de **Entregáveis** formatada e pronta para ser adicionada ao final do seu **README.md**:

---

## Entregáveis do Projeto

Os artefatos e componentes entregues neste projeto estão subdivididos em três categorias principais conforme as exigências da disciplina:

**Documentação Técnica**

* [Relatório Teórico](/docs/TP02_Deteccao_Cortes.pdf): Contém a análise detalhada dos resultados, tabelas comparativas com os frames identificados, discussão dos limiares e matrizes de confusão.

* [Guia do Repositório](/README.md): Guia completo de estrutura, instruções de execução, decisões teóricas e detalhamento de saídas.

**Códigos-Fonte**

* [detector_corte_histograma.py](/detector_corte_histograma.py): Algoritmo de extração de características via Histograma Local e cálculo de distância Euclidiana.  

* [detector_corte_BIC.py](/detector_corte_BIC.py): Algoritmo de quantização (64 cores), classificação de pixels (Borda/Interior) e cálculo da distância dLog.  

* [selecinaQuadro.py](/selecinaQuadro.py): Interface interativa OpenCV utilizada para navegar nos vídeos e construir o mapeamento manual do Ground Truth.  

* [estatisticas_quadros.py](/estatisticas_quadros.py): Módulo responsável por comparar as detecções automáticas contra o Ground Truth e calcular métricas detalhadas.  

* [faz_tudo_de_uma_vez.py](/faz_tudo_de_uma_vez.py): Script de orquestração que roda todos os algoritmos e limiares em lote para toda a base de vídeos.  

**Artefatos Gerados**

* **1. Arquivos com Quadros de Corte Identificados (`cortes_identificados/`)**

    * Arquivos texto contendo a lista dos quadros detectados automaticamente como limites de cena para cada vídeo, algoritmo e limiar aplicado.

    * *Nomenclatura:* `<nome_do_video>_<limiar>_<algoritmo>.txt`

* **2. Arquivos de Quadros-Chave (*Keyframes*) (`cortes_chaves/`)**
    
    * Arquivos texto com os índices dos quadros-chave representativos extraídos do ponto médio de cada cena delimitada.

    * *Nomenclatura:* `<nome_do_video>_cortes_chaves_<limiar>_<algoritmo>.txt`

* **3. Mapeamento Manual - Ground Truth (`cortes_manuais/`)**
    
    * Marcação dos quadros de corte realizada manualmente para validação e comparação estatística.

    * *Nomenclatura:* `<nome_do_video>_manual.txt`

* **4. Relatórios Estatísticos e Métricas (`estatisticas/`)**

    * Relatórios comparativos gerados para cada combinação de teste, contendo:

        * Métrica de Acurácia (%)

        * Contagem total de quadros (Manual vs. Algoritmo)

        * Classificação detalhada: Verdadeiros Positivos (TP), Falsos Positivos (FP), Falsos Negativos (FN) e Verdadeiros Negativos (VN)

        * Listagem explícita dos quadros corretamente identificados, não identificados e detectados incorretamente

    * *Nomenclatura:* `<nome_do_video>_estatistica_<limiar>.txt`

---

## Pré-requisitos e Preparação dos Arquivos

Antes de executar os algoritmos automáticos, certifique-se de configurar as entradas nas seguintes pastas:

* **`videos/`**: Insira os vídeos a serem analisados (ex.: `homem_aranha.mp4`, `trivago.mp4`, `flamengo_jornal.mp4`, `minecraft_trailer.mp4`, `old_town_road.mp4`).


* **`cortes_manuais/`**: Contém os arquivos de marcação manual (*Ground Truth*). Para cada vídeo `<nome>.mp4`, deve existir o correspondente `<nome>_manual.txt`.

---

## Diretrizes Técnicas e Algoritmos

1. **Particionamento Espacial em 5 Regiões:**
* Cada quadro é dividido em **5 partições**: uma partição central que compreende 60% da imagem e 4 partições de canto cobrindo 10% cada.


2. **Descritores Utilizados:**
* **Histograma Local:** Calcula o histograma de cores RGB para cada uma das 5 partições e aplica a distância Euclidiana máxima entre quadros consecutivos.

* **BIC Local:** Aplica quantização de cores (64 níveis) e classifica pixels entre Borda e Interior, calculando a distância dLog de cada partição.

3. **Pré-processamento & Filtragem:**
* Quadros de transição como **telas pretas** (≥90% dos pixels $\le 10$) ou **telas brancas** (≥90% dos pixels $\ge 240$) são desconsiderados para evitar falsos positivos.

4. **Salto de Amostragem (*Frame Skip*):**
* A análise avança em intervalos iguais à taxa de quadros por segundo ($\text{salto} = \text{FPS}$ do vídeo), aproximando-se de 1 segundo de intervalo.

5. **Cálculo de Quadros-Chave (*Key Frames*):**
* O quadro-chave é determinado pelo ponto médio entre dois cortes sucessivos:

$$QC = \lfloor \frac{\text{Quadro}_A + \text{Quadro}_B}{2} \rfloor$$

6. **Métricas e Janela de Tolerância:**

* A validação estatística em `estatisticas_quadros.py` classifica uma detecção como **Verdadeiro Positivo (TP)** se ela for exata ou estiver dentro do intervalo de tolerância de $\pm 1$ salto ($\pm \text{FPS}$) do valor manual.

---

## Como Executar

### 1. Automação Completa (Recomendado)

Para rodar todos os testes de Histograma (limiares 50000 e 100000) e BIC (limiares 5 e 1) nos vídeos cadastrados e gerar os relatórios estatísticos automaticamente:

```bash
python3 faz_tudo_de_uma_vez.py
```
### 2. Execução Individual dos Scripts

* **Marcação Manual Interativa:**
Utilize as teclas `A`/`D` ou Setas para navegar, `S` para salvar o quadro, `R` para remover e `Q` para encerrar:

```bash
python3 selecinaQuadro.py videos/homem_aranha.mp4 cortes_manuais/homem_aranha_manual.txt
```

* **Detector por Histograma:**
```bash
python3 detector_corte_histograma.py videos/homem_aranha.mp4 50000
```

* **Detector por BIC:**
```bash
python3 detector_corte_BIC.py videos/homem_aranha.mp4 5
```

* **Geração de Estatísticas Individual:**
```bash
python3 estatisticas_quadros.py cortes_manuais/homem_aranha_manual.txt cortes_identificados/homem_aranha_50000_hist.txt
```