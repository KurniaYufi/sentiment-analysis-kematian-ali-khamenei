# sentiment-analysis-kematian-ali-khamenei

# 📰 Analisis Sentimen Pemberitaan Kematian Ali Khamenei di Media Online Indonesia

> **Studi komparatif analisis sentimen berbasis NLP terhadap 487 artikel berita dari 5 platform media online Indonesia menggunakan TextBlob (lexicon-based), POS Tagging (Stanza), TF-IDF, dan Named Entity Recognition.**

---

## 🔬 Tentang Penelitian

Penelitian ini mengkaji kecenderungan sentimen pemberitaan media online Indonesia terkait kematian Ali Khamenei, Pemimpin Tertinggi Republik Islam Iran. Dengan menggunakan pendekatan **komputasional berbasis NLP**, penelitian ini menganalisis 487 artikel berita dari 5 platform media besar Indonesia secara sistematis dan *reproducible*.

### Rumusan Masalah

| # | Pertanyaan Penelitian |
|---|----------------------|
| RQ1 | Bagaimana kecenderungan sentimen pemberitaan media online Indonesia terkait kematian Ali Khamenei pada 5 platform berita? |
| RQ2 | Apakah terdapat perbedaan yang signifikan dalam distribusi sentimen (positif, negatif, netral) antar platform? |
| RQ3 | Bagaimana hasil klasifikasi sentimen menggunakan pendekatan lexicon-based (TextBlob) dan pola sentimen apa yang dapat diidentifikasi? |

### Temuan Utama

- **60,8%** artikel bersentimen **netral** — mencerminkan standar jurnalisme yang berimbang
- **28,3%** artikel bersentimen **negatif** — umumnya terkait konteks geopolitik, bukan serangan personal
- **10,9%** artikel bersentimen **positif** — didominasi ekspresi duka cita diplomatik
- CNN Indonesia memiliki proporsi negatif tertinggi (39,6%), Antaranews.com memiliki netral tertinggi (69,1%)

---

## 📊 Dataset

| Platform | Artikel Dikumpulkan | Artikel Valid | Persentase Valid |
|----------|--------------------:|-------------:|----------------:|
| Kompas.com | 100 | 98 | 98% |
| Detik.com | 100 | 97 | 97% |
| CNN Indonesia | 100 | 96 | 96% |
| Tribunnews.com | 100 | 99 | 99% |
| Antaranews.com | 100 | 97 | 97% |
| **Total** | **500** | **487** | **97,4%** |

- **Kata kunci scraping:** `"Ali Khamenei death"`
- **Batas artikel:** 100 per platform
- **Format output:** CSV
- **Rata-rata panjang artikel (setelah preprocessing):** 180–420 kata

---

## ⚙️ Pipeline

Pipeline penelitian terdiri dari 6 tahap utama:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. PENGUMPULAN DATA                                            │
│     Link scraping → Content scraping → Integrasi CSV           │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  2. PREPROCESSING TEKS                                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 2a. Text Cleaning                                        │   │
│  │     Lowercase · Hapus URL · Tanda baca · Angka          │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │ 2b. Tokenisasi                                          │   │
│  │     NLTK word_tokenize (Punkt Tokenizer)                │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │ 2c. Penghapusan Stopword                               │   │
│  │     NLTK stopwords bahasa Indonesia                     │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │ 2d. Stemming                                            │   │
│  │     Sastrawi (Enhanced Confix Stripping)                │   │
│  └─────────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────────┘
                            │
          ┌─────────────────┴──────────────────┐
          │                                    │
┌─────────▼──────────┐              ┌──────────▼─────────┐
│  3. ANALISIS       │              │  4. POS TAG & NER  │
│     SENTIMEN       │              │                    │
│  TextBlob          │              │  Stanza · IndoBERT │
│  Terjemahan EN     │              │  PER · ORG · LOC   │
│  Positif/Neg/Netral│              │                    │
└─────────┬──────────┘              └──────────┬─────────┘
          │                                    │
          └─────────────────┬──────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  5. ANALISIS TF-IDF                                             │
│     Global · Per POS · Per Platform · Per Sentimen             │
│     Unigram · Bigram · Trigram                                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────┐
│  6. VISUALISASI                                                 │
│     Pie · Bar · WordCloud · Heatmap                            │
└─────────────────────────────────────────────────────────────────┘
```

### Detail Setiap Tahap

#### 1. Pengumpulan Data (Web Scraping)
- **Tahap 1 – Link Scraping:** Mengambil URL artikel dari fitur pencarian internal setiap platform menggunakan `Requests` + `BeautifulSoup`. Jeda 2 detik antar request untuk menghindari pemblokiran IP.
- **Tahap 2 – Content Scraping:** Mengakses setiap URL untuk mengambil judul dan isi artikel lengkap. Sistem secara bertingkat mencari elemen `article`, `div.article-content`, atau `div.detail_text`.

#### 2. Preprocessing Teks
| Langkah | Metode | Keterangan |
|---------|--------|------------|
| Text Cleaning | Regex | Hapus URL, mention, angka, tanda baca; lowercase |
| Tokenisasi | NLTK `word_tokenize` | Punkt Tokenizer |
| Stopword Removal | NLTK `stopwords('indonesian')` | Filter kata tidak informatif |
| Stemming | Sastrawi ECS | Reduksi ke bentuk dasar bahasa Indonesia |

#### 3. Analisis Sentimen
- Library: **TextBlob** (lexicon-based)
- Teks diterjemahkan ke Bahasa Inggris terlebih dahulu menggunakan **GoogleTranslator** (`deep-translator`)
- Threshold klasifikasi:
  - Polaritas `> 0.05` → **Positif**
  - Polaritas `< -0.05` → **Negatif**
  - Di antaranya → **Netral**
- Teks panjang dipecah menjadi segmen, dianalisis terpisah, lalu digabung dengan rata-rata tertimbang

#### 4. POS Tagging & Named Entity Recognition
- Library: **Stanza** (Stanford NLP), model bahasa Indonesia (IDN Treebank, Universal Dependencies)
- Tag yang digunakan: `NOUN`, `VERB`, `ADJ`, `ADV`, `PROPN`, `NUM`, `PUNCT`
- NER mengidentifikasi entitas: `PER` (orang), `ORG` (organisasi), `LOC` (lokasi)

#### 5. Analisis TF-IDF
Dilakukan secara berlapis menggunakan `TfidfVectorizer` (scikit-learn):

| Level | Deskripsi |
|-------|-----------|
| **Global** | Seluruh 487 artikel, `max_features=800`, `ngram_range=(1,3)` |
| **Per POS** | Terpisah untuk NOUN, PROPN, VERB, ADJ, ADV |
| **Per Platform** | Mengungkap "sidik jari editorial" tiap media |
| **Per Sentimen** | Kata khas per kategori positif / negatif / netral |

Parameter: `min_df=2`, `max_df=0.90`, `sublinear_tf=True`, `smooth_idf=True`

#### 6. Visualisasi
- Pie chart & bar chart distribusi sentimen per platform
- WordCloud term TF-IDF tertinggi
- Bar chart horizontal top-N term per POS
- Heatmap korelasi antar variabel

---

## 📁 Struktur Proyek

```
sentiment-khamenei/
│
├── notebooks/
│   ├── 1-Detik-ScrappingLink.ipynb        # Link scraping
│   ├── 2-Detik-PreprocessingNews.ipynb    # Content scraping + preprocessing
│   ├── 3-Article-POSTagging-NER.ipynb     # POS tagging & NER (Stanza)
│   └── 4-Detik-TFIDF-v3.ipynb            # Analisis TF-IDF & sentimen
│
├── data/
│   ├── raw/                               # Data mentah hasil scraping (.csv)
│   │   ├── kompas_links.csv
│   │   ├── detik_links.csv
│   │   ├── cnn_links.csv
│   │   ├── tribun_links.csv
│   │   └── antara_links.csv
│   ├── processed/                         # Data setelah preprocessing
│   └── final/                            # Dataset final siap analisis
│
├── outputs/
│   ├── figures/                           # Visualisasi (PNG, 150 DPI)
│   └── reports/                          # Tabel hasil & ringkasan
│
├── requirements.txt
└── README.md
```

---

## 🛠️ Instalasi

### Prasyarat
- Python 3.10+
- Google Colaboratory (direkomendasikan) atau lingkungan lokal

### Install Dependencies

```bash
pip install -r requirements.txt
```

**`requirements.txt`:**
```
requests==2.31.0
beautifulsoup4==4.12.2
pandas==2.0.3
numpy==1.24.4
nltk==3.8.1
PySastrawi==1.2.0
stanza==1.7.0
scikit-learn==1.3.0
textblob==0.17.1
deep-translator==1.11.4
polyglot==16.7.4
matplotlib==3.7.2
seaborn==0.12.2
wordcloud==1.9.2
```

### Download Model & Resource NLTK

```python
import nltk
import stanza

# NLTK resources
nltk.download('punkt')
nltk.download('stopwords')
nltk.download('averaged_perceptron_tagger')

# Stanza model bahasa Indonesia
stanza.download('id')
```

---

## 🚀 Cara Penggunaan

### 1. Scraping Data

```python
# Jalankan notebook: 1-Detik-ScrappingLink.ipynb
# Atur target platform dan query sebelum menjalankan

QUERY = "Ali Khamenei death"
TARGET_ARTICLES = 100
PLATFORM = "detik"  # kompas | detik | cnn | tribun | antara
```

### 2. Preprocessing

```python
from PySastrawi.Stemmer.StemmerFactory import StemmerFactory
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
import re

factory = StemmerFactory()
stemmer = factory.create_stemmer()
stop_words = set(stopwords.words('indonesian'))

def preprocess(text):
    text = text.lower()
    text = re.sub(r'https?://\S+|www\.\S+', '', text)
    text = re.sub(r'\d+', '', text)
    text = re.sub(r'[^\w\s]', '', text)
    tokens = word_tokenize(text)
    tokens = [t for t in tokens if t not in stop_words]
    tokens = [stemmer.stem(t) for t in tokens]
    return ' '.join(tokens)
```

### 3. Analisis Sentimen

```python
from textblob import TextBlob
from deep_translator import GoogleTranslator

def analyze_sentiment(text):
    translated = GoogleTranslator(source='id', target='en').translate(text)
    polarity = TextBlob(translated).sentiment.polarity
    if polarity > 0.05:
        return 'Positif'
    elif polarity < -0.05:
        return 'Negatif'
    else:
        return 'Netral'
```

### 4. TF-IDF

```python
from sklearn.feature_extraction.text import TfidfVectorizer
import pandas as pd

vectorizer = TfidfVectorizer(
    max_features=800,
    ngram_range=(1, 3),
    min_df=2,
    max_df=0.90,
    sublinear_tf=True,
    smooth_idf=True
)

X = vectorizer.fit_transform(df['clean_text'])
tfidf_df = pd.DataFrame(
    X.toarray(),
    columns=vectorizer.get_feature_names_out()
)

# Top terms
top_terms = tfidf_df.mean().sort_values(ascending=False).head(15)
print(top_terms)
```

---

## 📈 Hasil Penelitian

### Distribusi Sentimen per Platform

| Platform | Positif | Negatif | Netral | Total |
|----------|--------:|--------:|-------:|------:|
| Kompas.com | 12 (12,2%) | 28 (28,6%) | 58 (59,2%) | 98 |
| Detik.com | 9 (9,3%) | 31 (32,0%) | 57 (58,8%) | 97 |
| CNN Indonesia | 7 (7,3%) | 38 (39,6%) | 51 (53,1%) | 96 |
| Tribunnews.com | 14 (14,1%) | 22 (22,2%) | 63 (63,6%) | 99 |
| Antaranews.com | 11 (11,3%) | 19 (19,6%) | 67 (69,1%) | 97 |
| **Total** | **53 (10,9%)** | **138 (28,3%)** | **296 (60,8%)** | **487** |

### Top 5 Term TF-IDF Global

| Peringkat | Term | Tipe | Skor TF-IDF |
|-----------|------|------|------------:|
| 1 | khamenei | Unigram | 0,1823 |
| 2 | iran | Unigram | 0,1654 |
| 3 | pemimpin tertinggi | Bigram | 0,1421 |
| 4 | wafat | Unigram | 0,1387 |
| 5 | israel | Unigram | 0,1203 |

### Term Khas per Platform (Editorial Fingerprint)

| Platform | Term Khas |
|----------|-----------|
| Kompas.com | analisis, perspektif, hubungan diplomatik, warisan, transformasi |
| Detik.com | konfirmasi, meninggal dunia, berita terkini, laporan, pejabat |
| CNN Indonesia | dampak global, kawasan, ketegangan, sanksi, program nuklir |
| Tribunnews.com | reaksi, duka cita, tokoh, ucapan belasungkawa, pemimpin dunia |
| Antaranews.com | pernyataan resmi, pemerintah Indonesia, sikap, diplomasi, tanggapan |

---

## 🧰 Teknologi yang Digunakan

| Kategori | Library / Tool |
|----------|---------------|
| Bahasa Pemrograman | Python 3.10+ |
| Lingkungan | Google Colaboratory |
| Web Scraping | `BeautifulSoup4`, `Requests` |
| Manipulasi Data | `Pandas`, `NumPy` |
| NLP Dasar | `NLTK` (tokenisasi, stopword) |
| Stemming | `PySastrawi` (Enhanced Confix Stripping) |
| POS Tagging & NER | `Stanza` (Stanford NLP, model `id`) |
| TF-IDF | `scikit-learn` (`TfidfVectorizer`) |
| Analisis Sentimen | `TextBlob` |
| Penerjemahan | `deep-translator` (GoogleTranslator) |
| Deteksi Bahasa | `Polyglot` |
| Visualisasi | `Matplotlib`, `Seaborn`, `WordCloud` |

---

## ⚠️ Keterbatasan

1. **Akurasi Terjemahan:** Penerjemahan otomatis Indonesia → Inggris menggunakan GoogleTranslator tidak selalu sempurna, terutama untuk kalimat kompleks, idiom, dan terminologi khas berita.
2. **Cakupan TextBlob:** TextBlob dirancang untuk bahasa Inggris umum dan mungkin kurang optimal untuk teks berita formal.
3. **Threshold Arbitrer:** Ambang batas sentimen (`> 0.05` / `< -0.05`) bersifat arbiter dan dapat mempengaruhi distribusi hasil.
4. **Tidak Ada Validasi Manual:** Belum dilakukan *human annotation* untuk mengukur akurasi klasifikasi secara kuantitatif (precision, recall, F1-score).

### Rekomendasi Pengembangan

- Gunakan **IndoBERT** atau **fine-tuned mBERT** untuk analisis sentimen langsung dalam Bahasa Indonesia
- Tambahkan dimensi **temporal** untuk melihat evolusi sentimen antar fase pemberitaan
- Perluas ke **media sosial** (Twitter/X, YouTube) untuk perbandingan dengan diskursus publik
- Terapkan **ensemble method** untuk klasifikasi sentimen yang lebih robust

---

## 📚 Referensi

- Liu, B. (2012). *Sentiment Analysis and Opinion Mining.* Synthesis Lectures on Human Language Technologies, 5(1), 1–167.
- Qi, P., et al. (2020). Stanza: A Python NLP Toolkit for Many Human Languages. *ACL 2020.*
- Tala, F. Z. (2003). *A Study of Stemming Effects on Information Retrieval in Bahasa Indonesia.* Universiteit van Amsterdam.
- Entman, R. M. (1993). Framing: Toward clarification of a fractured paradigm. *Journal of Communication*, 43(4), 51–58.
- Pratama, B. Y., & Wahyudi, R. (2021). Analisis Sentimen Pemberitaan Pemilu 2019 di Media Online Indonesia. *Jurnal Teknologi dan Sistem Komputer*, 9(3), 201–210.

---

## 📄 Lisensi

Proyek ini dikembangkan untuk keperluan penelitian akademis. Data berita yang digunakan merupakan milik platform masing-masing dan diakses sesuai kebijakan penggunaan publik masing-masing situs.

---

<p align="center">
  Dibuat untuk keperluan penelitian &nbsp;·&nbsp; NLP &nbsp;·&nbsp; Media Indonesia &nbsp;·&nbsp; Analisis Sentimen
</p>
