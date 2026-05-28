# movieNight — TODO



NAPRAW TMDB POSTER PATH NA SERWERZE

## 🟢 Przyszłość

- [ ] **Rating endpoint** — model `Rating` istnieje w DB ale brak routera
- [ ] **GhostUser** — użytkownik bez konta w sesji
- [ ] **Metryki rekomendacji** — jak dobrze poleca (precision, kliknięcia, rating po seansie)
- [ ] **LLM as judge** — automatyczna ocena jakości rekomendacji
- [ ] **`user_taste` learning** — aktualizacja wektora użytkownika po każdym ratingu
- [ ] **Deploy (production)** — po zebraniu feedbacku z beta
- [ ] **OpenAI Embeddings** — text-embedding-3-small zamiast BAAI/bge-base-en
- [ ] **Cohere Reranker** — migracja na zewnętrzne API (Cohere Rerank 3)
- [ ] **Baza as a Service** — przejście na Supabase (Postgres + pgvector)
- [ ] **RRF (Reciprocal Rank Fusion)** — lepsze łączenie wyników przy konfliktach w grupie
---