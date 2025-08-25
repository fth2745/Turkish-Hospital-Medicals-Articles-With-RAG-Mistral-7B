# Türkçe Tıbbi QA Chatbot & RAG Mimarisi (Generative Mistral 3B + FAISS)

Bu proje, **Türkçe tıbbi soruları yanıtlayabilen bir QA chatbot** oluşturmayı amaçlamaktadır. Modern **Retrieval-Augmented Generation (RAG)** yaklaşımı kullanılarak, kullanıcı sorularına hem doğru hem de bağlama uygun yanıtlar üretilir.  

---

## 🔹 Proje Hedefi

- Kullanıcının Türkçe tıbbi sorularına **doğru, bağlamlı ve okunabilir cevaplar** sunmak.  
- En alakalı tıbbi makaleleri otomatik olarak bulmak ve referans olarak göstermek.  
- Modern NLP teknikleri ile **retrieval + generative pipeline** tasarlamak.  

---

## 🔹 Teknik Mimari

Proje üç ana modülden oluşur:  

### 1. Veri ve Ön İşleme
- **Dataset:** [umutertugrul/turkish-hospital-medical-articles](https://huggingface.co/datasets/umutertugrul/turkish-hospital-medical-articles) kullanılmıştır.  
- Boş veya anlamsız içerikler filtrelenir.  
- Her makale başlık, URL ve metin olarak yapılandırılır.  

---

### 2. Retrieval & Reranking
- **SentenceTransformers** ile makaleler embeddinglenir (vektörleştirilir).  
- **FAISS** kullanılarak hızlı ve verimli bir vektör tabanlı arama sistemi oluşturulur.  
  - **Neden FAISS?** Büyük veri setlerinde benzerlik tabanlı aramalar CPU/GPU üzerinde çok hızlıdır.  
- Kullanıcının sorusu embeddinglenir ve **en yakın 100 makale** retrieval ile bulunur.  
- **CrossEncoder** ile en alakalı 10 pasaj yeniden sıralanır (rerank).  

> Bu yapı, **RAG (Retrieval-Augmented Generation)** yaklaşımının temelidir: Model yalnızca öğrenilmiş dil bilgisine dayanmaz, aynı zamanda **güncel ve doğrulanabilir kaynaklardan bilgi alır**.  

---

### 3. Generative QA
- **Mistral-7B-Instruct** modeli ile kullanıcıya bağlama uygun yanıt üretilir.  
- Üretilen metin:
  - Tekrarlayan cümlelerden arındırılır.  
  - Eğer son cümle token sınırından dolayı tamamlanmamışsa kaldırılır.  
- Maksimum giriş uzunluğu ve maksimum çıkış token sayısı kullanıcı tarafından ayarlanabilir.  

---

### 4. Arayüz
- **Gradio** kullanılarak etkileşimli bir chatbot arayüzü sunulur.  
- Kullanıcılar:
  - Sorularını girebilir  
  - Maksimum giriş uzunluğu ve maksimum çıkış token sayısını ayarlayabilir  
- Arayüz, aynı zamanda **top 5 en alakalı makaleyi** listeler.  

---

## 🔹 Kullanılan Teknolojiler

| Katman                  | Teknoloji / Kütüphane |
|-------------------------|----------------------|
| Veri                     | Hugging Face Datasets |
| Embedding / Retrieval    | SentenceTransformers, FAISS |
| Reranking               | CrossEncoder |
| Generative Model        | Transformers (Mistral-7B-Instruct) |
| Arayüz                   | Gradio |
| Pipeline yönetimi       | Python, PyTorch |

---

## 🔹 Neden Bu Yaklaşım?
1. **RAG (Retrieval-Augmented Generation):**  
   - Büyük dil modelleri her zaman güncel bilgi içermez.  
   - Retrieval adımı ile model, doğru ve güncel makalelerden bilgi alır.  

2. **FAISS:**  
   - Milyonlarca dokümanı çok hızlı bir şekilde taramak mümkün.  
   - GPU ile çalıştırıldığında vektör benzerlik hesaplamaları oldukça verimlidir.  

3. **CrossEncoder Reranking:**  
   - Retrieval ile gelen pasajlar, kullanıcının sorusuna göre yeniden sıralanır.  
   - Böylece generative model sadece **en alakalı içerik** üzerinden yanıt üretir.  

4. **Mistral-7B:**  
   - Büyük ve açık kaynaklı bir dil modeli.  
   - Türkçe ve tıbbi terminolojiye uygun şekilde fine-tune edilmiştir (Instruction modeli).  

---

## 🔹 Projenin Avantajları

- **Doğru ve güvenilir cevaplar:** Sadece dil modeline değil, gerçek makalelere dayalı bilgi.  
- **Hızlı retrieval:** FAISS sayesinde anında en alakalı makaleler bulunur.  
- **Kullanıcı dostu arayüz:** Gradio ile kolay kullanım.  
- **Esnek parametreler:** Maksimum giriş/çıkış uzunluğu kullanıcı tarafından ayarlanabilir.  

---
## 🔹 Gelecek Geliştirme Alanları
- **Multi-GPU ve daha büyük veri setleri ile ölçeklendirme.
- **Konu özelinde daha iyi embeddingler ve tıbbi terminoloji adaptasyonu.
- **Chat geçmişi ve bağlamlı uzun sohbet desteği.
- **Hugging Face Spaces üzerinden deploy ve paylaşım.

## 🔹 Proje Workflow Diyagramı

```mermaid
flowchart TD
    A[Soru: Kullanıcı Girişi] --> B[Embedding Oluşturma: SentenceTransformer]
    B --> C[Retrieval: FAISS - Top 100 Makale]
    C --> D[Reranking: CrossEncoder - Top 10 Makale]
    D --> E[Top 5 Alakalı Makale Seçimi]
    E --> F[Generative Model: Mistral-7B]
    F --> G[Cevap: Kullanıcıya Gönderim]

