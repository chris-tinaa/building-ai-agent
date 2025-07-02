# Use Case: Hospital System Chatbot

<img width="1470" alt="image" src="https://github.com/user-attachments/assets/8cd8b99e-faa3-4e8e-82ee-36b3cb0833aa" />


## Dokumentasi Teknis & Refleksi

### Alur Kerja Agent & Arsitektur

1. **Pengguna** berinteraksi melalui chat di FE (Streamlit).
2. **Frontend** mengirimkan pertanyaan pengguna ke **API Backend** (FastAPI) melalui endpoint `/hospital-rag-agent`.

<img width="1108" alt="image" src="https://github.com/user-attachments/assets/5a5c1cbe-c70f-4f14-83c8-17c0fc6b81cc" />

3. **API** memproses pertanyaan dengan memanggil **RAG Agent** (LangChain Agent Executor) yang terdiri dari beberapa "tool":
   - **Graph Tool**: Mengubah pertanyaan menjadi query Cypher untuk Neo4j, digunakan untuk pertanyaan terstruktur (misal: statistik, relasi antar entitas).
   - **Experiences Tool**: Melakukan semantic search pada review pasien menggunakan vektor embedding, digunakan untuk pertanyaan kualitatif (misal: pengalaman pasien).
   - **Waits/Availability Tool**: Mengambil data waktu tunggu rumah sakit secara real-time (simulasi).
4. **RAG Agent** memilih dan menjalankan tool yang paling relevan berdasarkan prompt dan instruksi.
5. **API** mengembalikan jawaban beserta penjelasan langkah-langkah intermediate ke Frontend.
6. **Frontend** menampilkan jawaban dan reasoning kepada pengguna.


### Pendekatan Memory (Episodic / Semantic)

- **Semantic Memory**: Sistem menggunakan semantic memory melalui vektor embedding pada data review pasien. Dengan pendekatan ini, pertanyaan yang bersifat kualitatif atau membutuhkan pemahaman konteks dapat dijawab dengan melakukan pencarian vektor (vector search) pada node review di Neo4j.
- **Episodic Memory**: Sistem tidak secara eksplisit menyimpan riwayat percakapan (episodic memory) antar sesi. Namun, setiap pertanyaan dalam satu sesi chat dapat diakses ulang oleh pengguna melalui chat history di frontend.

### Strategi RAG / Prompt yang Digunakan

- **Retrieval-Augmented Generation (RAG)**: Sistem menggabungkan hasil retrieval (baik structured query ke Neo4j maupun semantic search ke review) dengan LLM untuk menghasilkan jawaban yang relevan dan kontekstual.
  - Untuk Graph Tool, digunakan prompt template yang terstruktur untuk menghasilkan query Cypher yang valid dan aman, serta instruksi agar model tidak keluar dari skema yang diizinkan.
  - Untuk Experiences Tool, digunakan prompt yang mengarahkan model untuk hanya menggunakan konteks dari hasil semantic search dan tidak mengarang informasi.
  - Setiap tool memiliki deskripsi dan instruksi penggunaan yang jelas agar agent dapat memilih tool yang tepat.
- Setiap langkah reasoning agent (tool yang dipilih, query yang dijalankan, dsb) dikembalikan ke frontend sebagai penjelasan (explanation) untuk transparansi.

### Hasil & Refleksi (Kendala, Solusi)

**Hasil:**
- Sistem mampu menjawab pertanyaan kompleks yang membutuhkan reasoning 
- Pengguna mendapatkan jawaban yang explainable, dengan reasoning yang transparan.

**Kendala & Solusi:**
- **Kendala:** Integrasi antara LLM dan Neo4j membutuhkan prompt engineering yang ketat agar query yang dihasilkan valid dan tidak membahayakan database.
  - **Solusi:** Menggunakan prompt template yang membatasi skema, contoh query, dan instruksi eksplisit untuk mencegah query yang tidak diinginkan.
- **Kendala:** Tidak adanya memory percakapan jangka panjang (episodic memory) membatasi kemampuan agent
  - **Solusi:** Menyediakan chat history di frontend dan dapat dikembangkan dengan memory module di masa depan.



## Screenshots

Load data ke Neo4j
<img width="1095" alt="image" src="https://github.com/user-attachments/assets/3862831c-4e14-416f-bc99-1a35e17f6a55" />

<img width="1470" alt="image" src="https://github.com/user-attachments/assets/73eb3272-259f-40e8-89b8-8695aeba6684" />

RAG
<img width="1109" alt="image" src="https://github.com/user-attachments/assets/90f03dbd-d145-4517-b1c5-611e306a8617" />

Tools
<img width="1112" alt="image" src="https://github.com/user-attachments/assets/0459eae4-4215-4628-a3d0-21e5b24174fb" />

<img width="1099" alt="image" src="https://github.com/user-attachments/assets/02109e03-12b7-4d55-bde5-d88806a44e90" />

Cypher
<img width="1105" alt="image" src="https://github.com/user-attachments/assets/4ed8d4df-c15b-446d-917c-3c10751427dd" />


