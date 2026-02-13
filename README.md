
# Create cisTarget Track Databases (AertsLab)

Pipeline oficial da **AertsLab** para criação de **cisTarget track databases** a partir de dados **ChIP-seq** (formatos `.BED`, `.bigWig`).

Este repositório automatiza a configuração do ambiente, organiza dependências e fornece os **scripts originais** necessários para gerar bancos de dados compatíveis com:

* **cisTarget**
* **pycistarget**
* **SCENIC / SCENIC+**

Referências e documentação oficial:
Repositório do AertsLab → [https://github.com/aertslab/create_cisTarget_databases](https://github.com/aertslab/create_cisTarget_databases)

---

## 1. Requisitos e Ambiente Recomendada

Para usuários Windows recomenda-se:

1. **WSL2 + Ubuntu**

   * Instalar via Microsoft Store: Ubuntu
     [https://apps.microsoft.com/detail/9pdxgncfsczv](https://apps.microsoft.com/detail/9pdxgncfsczv)

2. **Miniconda**
   Instalação sugerida:

   ```bash
   wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   chmod +x Miniconda3-latest-Linux-x86_64.sh
   ./Miniconda3-latest-Linux-x86_64.sh
   ```

---

## 2. Clonar o Repositório Oficial

```bash
git clone https://github.com/aertslab/create_cisTarget_databases.git
cd create_cisTarget_databases
```

---

## 3. Criar e Ativar o Ambiente Conda

Criação do ambiente:

```bash
conda create -n create_cistarget_databases -c conda-forge --strict-channel-priority \
    "python=3.10" \
    "numpy>=1.22.4,<2" \
    "pandas<2" \
    "pyarrow>=7.0.0" \
    "numba>=0.58" \
    "python-flatbuffers"

```

Ativação:

```bash
conda activate create_cistarget_databases
```

---

## 4. Instalar o Cluster-Buster

Ferramenta essencial para processamento de sequências:

```bash
cd "${CONDA_PREFIX}/bin"
wget https://resources.aertslab.org/cistarget/programs/cbust
chmod a+x cbust
```

---

## 5. Instalar Ferramentas UCSC

Necessárias para manipular tracks:

```bash
cd "${CONDA_PREFIX}/bin"

wget http://hgdownload.soe.ucsc.edu/admin/exe/linux.x86_64/liftOver
wget http://hgdownload.soe.ucsc.edu/admin/exe/linux.x86_64/bigWigAverageOverBed

chmod a+x liftOver bigWigAverageOverBed
```

---

## 6. Exportar Variável de Ambiente do Repositório

Na pasta clonada:

```bash
conda activate create_cistarget_databases
export create_cistarget_databases_dir=$(pwd)
```

---

## 7. Testar Instalação

```bash
${create_cistarget_databases_dir}/create_cistarget_track_databases.py --help
```

Se o help for exibido, a instalação está correta.

---
## 8. Baixando arquivos necessários
```bash
wget https://resources.aertslab.org/cistarget/regions/hg38-limited-upstream10000-tss-downstream10000-full-transcript.bed
```
```bash
wget https://resources.aertslab.org/cistarget/regions hg38-limited-upstream500-tss-downstream100-full-transcript.bed
```

## 9. Gerando a Database a partir de ChIP-seq

### Estrutura necessária:

1. Criar a pasta onde ficarão os arquivos `.bigWig`:

```
bigwigs/
```

2. Dentro dessa pasta, gerar um arquivo `.txt` contendo todos os bigWigs:

```bash
ls *.bigWig > ../lista_tracks.txt
```

A partir daqui utilizamos o script principal:

### Exemplo para SCENIC

```bash
${create_cistarget_databases_dir}/create_cistarget_track_databases.py \
    -b hg38-limited-upstream10000-tss-downstream10000-full-transcript.bed \
    -T bigwigsE2F1 \
    -d lista_tracksE2F1.txt \
    -o hg38_chipseq \
    -g "#[0-9]+$"
```

### Exemplo para SCENIC+

```bash
${create_cistarget_databases_dir}/create_cistarget_track_databases.py \
  -b hg38-limited-upstream10000-tss-downstream10000-full-transcript.bed \
  -T bigwigs \
  -d lista_tracks.txt \
  -o hg38_chipseq
```

**Observação importante:**

* Se **não** for usado o parâmetro `-g`, o pipeline criará uma matriz `regions × tracks`, formato utilizado no **SCENIC+**.

---

## 9. Estrutura Recomendada de Diretórios

```bash
project/
├── create_cisTarget_databases/
├── data/
│   ├── tracks/
│   ├── track_ids.txt
│   └── hg38-limited-upstream500-tss-downstream100-full-transcript.bed
└── README.md
```

---

## 10. Integração com o pySCENIC / SCENIC+

Após gerar os rankings, é possível integrá-los com o pySCENIC.

Exemplo generativo de ranking final:

```
pyscenic ctx \
    dados/adj.tsv \
    databases/hg38__refseq-r80__10kb_up_and_down_tss.mc9nr.feather \
    databases/hg38_chipseq.tracks_vs_genes.rankings.feather \
    --annotations_fname databases/motifs-v9-nr.hgnc-m0.001-o0.0.tbl \
    --expression_mtx_fname dados/PBMC10k_filtered.loom \
    --mode custom_multiprocessing \
    --nes_threshold 2.5 \
    --output output/reg.csv \
    --num_workers 10 \
    --mask_dropouts
```

---

## 11. Principais Componentes e Fontes de Dados

| Item             | Descrição                   | Fonte                                                |
| ---------------- | --------------------------- | ---------------------------------------------------- |
| regions.bed      | Intervalos genômicos        | resources.aertslab.org/cistarget/regions/            |
| bigWigs ChIP-seq | Tracks experimentais        | ENCODE, ChIP-Atlas etc.                              |
| track_ids.txt    | Lista de paths para bigWigs | Gerado manualmente via `ls *.bigWig > track_ids.txt` |

---

## 12. Considerações Finais

Após a geração:

* Para **SCENIC**, utiliza-se matriz de rankings **gene × motif/tracks**
* Para **SCENIC+**, utiliza-se **regions × tracks**

Os bancos podem ser usados em pipelines downstream para:

* Inferência regulatória
* Enriquecimento de motivos
* Integração RNA + ATAC
* Construção de regulons


