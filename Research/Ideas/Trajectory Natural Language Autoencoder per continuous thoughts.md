Secondo me il progetto va impostato così:

Costruire un **Natural Language Autoencoder** per sequenze di *continuous thoughts*, non per singole attivazioni, e verificare se il testo conserva davvero la struttura temporale della traiettoria.

Non partirei dal labeling di “deception”, “verification” o “constraint neglect”. Quelle diventano applicazioni di validazione, non la supervisione principale.

La pipeline centrale è:

$$
H=(h_1,\ldots,h_T)
\xrightarrow{\text{Trajectory AV}}
s
\xrightarrow{\text{Trajectory AR}}
\hat{H}=(\hat{h}_1,\ldots,\hat{h}_T)
$$

dove:

- $H$ è una sequenza ordinata di stati latenti;
- $s$ è una spiegazione testuale compatta;
- $\hat{H}$ è la traiettoria ricostruita dal solo testo.

La domanda scientifica primaria è:

> Un bottleneck testuale può preservare non solo il contenuto dei singoli stati, ma anche l’evoluzione temporale di un processo latente?

Anthropic fa *vector → text → vector*, iniettando un solo vettore come embedding speciale e ricostruendolo dal testo. Il loro confronto usa vettori normalizzati e quindi misura principalmente l’accordo direzionale; per Qwen2.5-7B rilasciano AV e AR sul layer 20, e la loro pipeline completa è SFT dell’AR, SFT dell’AV e poi GRPO con reward di ricostruzione.

## 1. La decisione più importante: cosa chiamare “traiettoria”

Ci sono almeno tre possibilità.

### A. Stati di token generati da un LLM normale

Prendete il residual stream dello stesso layer per token successivi:

$$
H=\left(h_1^{(\ell)},\ldots,h_T^{(\ell)}\right)
$$

È la strada più semplice, ma ha un problema: ogni $h_t$ corrisponde a un token osservabile. Il verbalizer potrebbe limitarsi a recuperare il testo generato, invece di interpretare un processo interno.

### B. Evoluzione attraverso i layer

Per uno stesso token osservate:

$$
H=\left(h^{(1)},h^{(2)},\ldots,h^{(L)}\right)
$$

È una traiettoria computazionale attraverso la profondità, ma non coincide con il vostro scenario iniziale di agenti che ragionano mediante diversi stati latenti successivi.

### C. Continuous thoughts autentici

Usate un modello come **Coconut**, che prende l’ultimo hidden state e lo reinserisce direttamente come embedding del passo successivo, senza convertirlo in token. In questo caso:

$$
h_t \rightarrow h_{t+1}
$$

è davvero una sequenza di pensieri continui non verbalizzati. Coconut è stato progettato proprio così e dispone di codice ufficiale per GSM8K, ProntoQA e ProsQA.

### Raccomandazione

Il paper principale dovrebbe usare **C: continuous thoughts autentici**.

Userei A solo come esperimento preliminare per riutilizzare immediatamente i checkpoint NLA di Anthropic.

## 2. Obiettivo scientifico preciso

Una formulazione pulita:

> We study whether an ordered sequence of continuous reasoning states can be compressed into a short natural-language description from which the original trajectory—and particularly its temporal evolution—can be reconstructed.

Tre sotto-domande:

1. Il testo conserva i singoli stati?
2. Conserva l’ordine e le transizioni tra gli stati?
3. Le descrizioni corrispondono a proprietà osservabili del processo, come esplorazione, convergenza, errore intermedio o mantenimento di un vincolo?

La terza domanda viene dopo le prime due. Non serve assumere che una traiettoria “significhi davvero verification”: serve dimostrare prima che il bottleneck conserva la dinamica.

## 3. Modelli da usare

### 3.1 Modello target: due livelli

#### Pilot di riproduzione

**GPT-2 124M + Coconut ufficiale**

Serve esclusivamente per:

- verificare che il codice Coconut funzioni;
- estrarre correttamente le continuous thoughts;
- testare l’architettura multi-vector;
- effettuare debug senza spendere molto.

Un lavoro recente su continuous-thought misalignment ha usato GPT-2 124M con sei latent tokens e ha riportato un training nell’ordine di poche ore su una A100, mostrando che questa scala è utile per prototipi causali.

Non lo userei come unico modello del paper.

#### Modello principale

**Qwen2.5-1.5B-Instruct adattato a Coconut**

Motivazioni:

- abbastanza piccolo da poter essere addestrato e replicato;
- sufficientemente moderno da non rendere il lavoro un esperimento giocattolo;
- 1,54 miliardi di parametri e 28 layer;
- stessa famiglia di Qwen2.5-7B, per cui Anthropic ha rilasciato NLA e infrastruttura di riferimento.

Per il dominio matematico potete usare anche **Qwen2.5-Math-1.5B-Instruct**, che ha hidden size 1536.

#### Validazione di scala

Solo dopo che tutto funziona:

- Qwen2.5-3B;
- eventualmente Qwen2.5-7B;
- oppure Gemma-3-12B, per cui Anthropic ha rilasciato un altro NLA.

Non partirei dal 7B Coconut: raddoppierebbe il rischio tecnico prima di sapere se la domanda scientifica funziona.

## 4. Quali vettori estrarre

Coconut genera un continuous thought reinserendo nel modello l’ultimo hidden state. Per ogni passo $t$, salverei tre rappresentazioni:

$$
h_t^{\text{in}},\qquad h_t^{\text{mid}},\qquad h_t^{\text{out}}
$$

dove:

- $h_t^{\text{in}}$: vettore continuo ricevuto come embedding;
- $h_t^{\text{mid}}$: residual stream a circa due terzi della profondità;
- $h_t^{\text{out}}$: ultimo hidden state, reinserito nel passo successivo.

Il modello principale dovrebbe interpretare:

$$
H^{\text{mid}}=
\left(h_1^{\text{mid}},\ldots,h_T^{\text{mid}}\right)
$$

Perché Anthropic ha scelto layer a circa due terzi della profondità: abbastanza profondi da contenere semantica ricca, ma non così vicini all’unembedding da essere dominati dalla previsione dei token.

Gli *out* vanno comunque salvati perché sono gli effettivi continuous thoughts e servono per gli interventi causali.

### Lunghezza iniziale

Partirei con:

$$
T=6
$$

fisso per tutti gli esempi.

Non implementerei subito traiettorie di lunghezza variabile. Sei passi sono sufficienti per studiare:

- ordine;
- transizioni;
- early vs late state;
- perdita o convergenza del segnale.

Una volta funzionante, testate:

$$
T\in\{2,4,6,8\}
$$

## 5. Architettura del Trajectory NLA

### 5.1 Trajectory Activation Verbalizer

L’AV riceve sei vettori come sei embedding speciali:

```text
You are given an ordered sequence of continuous reasoning states.

<STATE_1>
<STATE_2>
<STATE_3>
<STATE_4>
<STATE_5>
<STATE_6>

Describe how the internal process evolves across the sequence.
```

Gli embedding normali dei token `<STATE_i>` vengono sostituiti dai vettori $h_i$.

Formalmente:

$$
e_i=A\!\left(\operatorname{RMSNorm}(h_i)\right)+p_i
$$

dove:

- $A$ inizialmente è un adattatore lineare condiviso;
- $p_i$ è un piccolo embedding temporale appreso;
- l’ordine è comunque codificato anche dalle posizioni del Transformer.

Anthropic inietta il singolo vettore quasi direttamente, dopo uno scaling fisso; per una sequenza e per vettori provenienti dall’ultimo hidden state di Coconut, un adattatore condiviso è una precauzione ragionevole.

L’output inizialmente può avere una forma semi-strutturata:

```text
Initial state: ...
Transition: ...
Late state: ...
Overall process: ...
```

Lunghezza massima: **48–64 token**.

Il limite è importante: senza una restrizione, il modello potrebbe codificare la traiettoria in un testo lunghissimo o quasi steganografico.

### 5.2 Trajectory Activation Reconstructor

L’AR riceve solo la spiegazione e sei token di ricostruzione:

```text
Explanation:
{generated explanation}

Reconstructed states:
<REC_1> <REC_2> <REC_3> <REC_4> <REC_5> <REC_6>
```

Per ciascun token:

$$
\hat{h}_i=W r_i+b
$$

dove $r_i$ è l’hidden state dell’AR in corrispondenza di `<REC_i>`.

Userei:

- un’unica testa $W$ condivisa tra i passi;
- embedding temporali distinti per i sei reconstruction slots;
- nessuna testa separata per posizione nella versione principale.

Una testa diversa per ogni $i$ rischierebbe di memorizzare statistiche posizionali invece di ricostruire una dinamica.

## 6. Funzione obiettivo

### 6.1 Ricostruzione dei singoli stati

Come Anthropic:

$$
\mathcal{L}_{\text{state}}
=
\frac{1}{T}
\sum_{t=1}^{T}
\left[1-\cos\!\left(h_t,\hat{h}_t\right)\right]
$$

È la loss fondamentale.

### 6.2 Ricostruzione delle transizioni

Definite:

$$
\Delta h_t=h_{t+1}-h_t
$$

e:

$$
\mathcal{L}_{\Delta}
=
\frac{1}{T-1}
\sum_{t=1}^{T-1}
\left[
1-\cos\!\left(\Delta h_t,\Delta \hat{h}_t\right)
\right]
$$

Questa loss obbliga il testo a conservare come cambia la traiettoria, non solo il suo tema medio.

### 6.3 Geometria globale

Costruite la matrice delle similarità interne:

$$
G_{ij}(H)=\cos(h_i,h_j)
$$

e minimizzate:

$$
\mathcal{L}_{\text{geom}}
=
\frac{1}{T^2}
\left\|G(H)-G(\hat{H})\right\|_F^2
$$

Questo preserva:

- quali stati sono vicini;
- dove si verifica un cambiamento;
- se la traiettoria converge o diverge.

### Loss complessiva

$$
\mathcal{L}_{\text{AR}}
=
\mathcal{L}_{\text{state}}
+\alpha\mathcal{L}_{\Delta}
+\beta\mathcal{L}_{\text{geom}}
$$

Partirei con:

$$
\alpha=0.5,\qquad \beta=0.1
$$

ma vanno trattati come valori iniziali, non come scelta definitiva.

Durante il training dell’AV:

$$
R(s,H)
=
-\mathcal{L}_{\text{AR}}
-\gamma\,\operatorname{Length}(s)
-\eta\,\operatorname{Unnaturalness}(s)
$$

## 7. Il rischio più grave: un codice testuale non umano

AV e AR potrebbero colludere.

Il testo potrebbe sembrare inglese ma usare:

- punteggiatura;
- sinonimi insoliti;
- ordine delle parole;
- token rari;

come canale nascosto per trasmettere numeri all’AR.

La buona ricostruzione, da sola, non dimostra interpretabilità.

Inserirei quindi tre vincoli.

### Paraphrase bottleneck

Durante parte del training:

$$
H\rightarrow s\rightarrow \operatorname{Paraphrase}(s)\rightarrow \hat{H}
$$

La ricostruzione deve sopravvivere a una parafrasi.

Se collassa dopo una parafrasi semanticamente equivalente, il sistema sta usando la superficie del testo e non il significato.

### Due reconstructor indipendenti

Addestrate:

$$
\operatorname{AR}_1(s)\rightarrow \hat{H}_1
$$

$$
\operatorname{AR}_2(s)\rightarrow \hat{H}_2
$$

da inizializzazioni diverse.

Il reward dell’AV è la media delle due ricostruzioni. Questo rende più difficile sviluppare un codice arbitrario condiviso con un unico AR.

### Controlli semantici

Confrontate:

- testo originale;
- parafrasi;
- sostituzione con sinonimi;
- word shuffle;
- rimozione delle frasi temporali;
- testo genericamente plausibile.

Una vera descrizione semantica dovrebbe essere robusta alla parafrasi, ma non al word shuffle o alla rimozione delle informazioni temporali.

## 8. Dati

### 8.1 Dataset principali per addestrare il modello Coconut e generare traiettorie

#### ProsQA

È il dataset più importante.

Coconut lo usa per task di reasoning su grafi/DAG che richiedono pianificazione e branching. Nel paper, il vantaggio del latent reasoning emerge soprattutto su ProsQA, e gli autori interpretano i primi continuous thoughts come esplorazione parallela che si restringe nei passi successivi.

Uso:

- training principale del target latent reasoner;
- studio *exploration → convergence*;
- valutazione dell’importanza dell’ordine.

#### ProntoQA

Task logici multi-hop generabili sinteticamente. Il repository Coconut fornisce la procedura per produrre esempi a cinque hop.

Uso:

- traiettorie con una progressione logica nota;
- generalizzazione fuori da ProsQA;
- perturbazioni controllate della catena inferenziale.

#### GSM8K

Problemi matematici con ragionamenti step-by-step; Coconut fornisce una pipeline di preprocessing e usa una versione aumentata/sintetica per il training.

Uso:

- corretto vs errato;
- traiettorie con lunghezze e strutture più varie;
- confronto con process-verification datasets.

#### Quantità da generare

Punterei a circa **100.000 traiettorie**, non necessariamente 100.000 prompt unici:

| Dominio | Traiettorie |
|---|---:|
| ProsQA | 40.000 |
| ProntoQA | 30.000 |
| GSM8K | 30.000 |

Per ciascun prompt generate 2–4 rollout con temperature diverse, conservando sia successi sia fallimenti.

Non bisogna filtrare via tutti gli errori: le traiettorie sbagliate sono fondamentali.

### 8.2 Dataset di valutazione interpretativa

#### PRM800K

Contiene circa 800.000 giudizi umani a livello di singolo passaggio matematico ed è stato rilasciato con *Let’s Verify Step by Step*.

Uso:

- creare esempi con ragionamento corretto fino a un certo passo e poi errato;
- testare se la descrizione della traiettoria contiene un cambiamento coerente con il primo errore;
- eventualmente addestrare solo un piccolo evaluator testuale, non il TNLA.

#### ProcessBench

Contiene 3.400 soluzioni matematiche con l’annotazione del primo step errato, oppure l’indicazione che l’intera soluzione è corretta.

Uso:

- evaluation held-out;
- error localization;
- confronto early/middle/late descriptions.

Non lo userei per addestrare il verbalizer.

#### IFEval

Comprende 541 prompt costruiti usando 25 tipi di istruzioni automaticamente verificabili, come lunghezza, parole obbligatorie e formati specifici.

Uso:

- constraint tracking;
- confronto tra output che rispettano o violano i vincoli;
- verifica automatica, senza annotatori umani.

Per usarlo con Coconut, dovrete generare reasoning traces sintetiche e fare un piccolo fine-tuning del target model sul dominio instruction-following.

#### MoralChain

Un lavoro del 2026 ha già costruito MoralChain, 12.000 scenari con percorsi morali e immorali, addestrando un Coconut con un dual trigger per separare ragionamento latente misallineato e output osservabile. Ha già mostrato che probe lineari distinguono condizioni armate da condizioni normali e che il segnale è più forte nei primi latent tokens.

Uso:

- solo case study finale di safety;
- confronto con il probe lineare del lavoro esistente;
- domanda nuova: il TNLA riesce a verbalizzare il pattern “plan then suppress”? 

Non lo userei come contributo principale: il lato detection/probing è già molto vicino al vostro problema.

#### TruthfulQA

Contiene 817 domande in 38 categorie, progettate per indurre risposte basate su false credenze diffuse.

Uso:

- test secondario OOD;
- corretto, falso, astensione;
- nessuna pretesa di avere ground truth sul “processo interno”.

## 9. Come ottenere i primi target testuali

Il TNLA deve essere fondamentalmente auto-supervisionato, ma un training direttamente da zero con RL sarebbe instabile.

Userei una fase di warm-start.

Coconut viene addestrato mediante un curriculum che sostituisce progressivamente gli step testuali del CoT con continuous thoughts. Ciò vi dà, per ogni traiettoria, una versione esplicita del ragionamento che l’ha supervisionata.

Un teacher riceve:

- domanda;
- CoT esplicito originale;
- risposta;
- esito corretto/errato;

e genera una descrizione compressa del processo:

```text
The process initially maintains two candidate paths.
The middle states eliminate one branch.
The final states commit to the remaining path.
```

Il teacher non vede le attivazioni. Il testo non è considerato verità interna: serve solo a inizializzare il linguaggio dell’AV.

Create circa **50.000 descrizioni warm-start**.

Poi la ricostruzione delle attivazioni diventa il vero segnale di training.

## 10. Procedura di training

### Fase 0 — Riproduzioni

Prima di modificare qualsiasi cosa:

- eseguite inferenza con il checkpoint NLA Qwen2.5-7B layer 20;
- riproducete Coconut su almeno un dataset;
- verificate che answer accuracy e latent-thought extraction siano ragionevoli;
- salvate e reiniettate una continuous thought senza cambiare l’output.

**Criterio di uscita:** la reiniezione dei vettori salvati deve riprodurre quasi esattamente la generazione originale.

### Fase 1 — Train del target Coconut

1. CoT fine-tuning iniziale.
2. Curriculum Coconut:
   - stage 0: tutti gli step testuali;
   - stage 1: primo step sostituito da continuous thoughts;
   - …
   - stage finale: sei latent thoughts, poi risposta.
3. Valutazione su ProsQA, ProntoQA e GSM8K.
4. Congelamento del target model.

Il repository Coconut assume quattro A100 da 80 GB per le riproduzioni principali, anche se il consumo può essere ridotto adattando batch size e gradient accumulation.

### Fase 2 — Raccolta delle traiettorie

Per ogni rollout salvate:

```text
example_id
dataset
split
prompt_hash
seed
correct
final_answer
latent_length
h_in[1:T]
h_mid[1:T]
h_out[1:T]
explicit_cot_id
constraint_metadata
first_error_step
```

#### Formato dati

Non metterei tutte le matrici come liste annidate in un unico Parquet.

Userei:

- Zarr o memory-mapped NumPy per le attivazioni;
- Parquet per metadata, indici e label;
- bf16 per lo storage principale;
- fp32 solo durante il calcolo delle metriche.

Per Qwen2.5-1.5B, assumendo $d=1536$, sei vettori bf16 occupano circa:

$$
6\times1536\times2\approx18\ \text{KB}
$$

Per 100.000 traiettorie:

$$
\approx1.8\ \text{GB}
$$

per ogni tipo di rappresentazione. Salvando *in*, *mid* e *out*, siete intorno a 5–6 GB, prima dei metadata.

### Fase 3 — AR SFT

Addestrate prima:

$$
s\rightarrow\hat{H}
$$

con le spiegazioni teacher.

Configurazione iniziale:

- bf16;
- sequence length 256;
- learning rate $10^{-5}$ per il backbone;
- learning rate più alto per adapter e reconstruction head;
- 3 epoch;
- early stopping su state cosine + delta cosine;
- gradient checkpointing;
- FSDP o DeepSpeed ZeRO-3.

### Fase 4 — AV SFT

Addestrate:

$$
H\rightarrow s
$$

sui target teacher.

Configurazione:

- massimo 64 output token;
- LoRA inizialmente;
- teacher forcing;
- mix dei tre dataset;
- batch bilanciati tra corretto ed errato.

Dopo questo passaggio fate già il primo round trip:

$$
H\rightarrow s\rightarrow\hat{H}
$$

### Fase 5 — Best-of-$N$ distillation

Prima di usare RL pesante:

1. AV genera $N=8$ spiegazioni per traiettoria.
2. AR assegna a ciascuna il reconstruction score.
3. Applicate paraphrase robustness e naturalness filter.
4. Tenete le migliori 1–2.
5. Fate un altro SFT dell’AV.

Questa procedura è molto meno costosa e più facile da debuggare del GRPO.

### Fase 6 — GRPO facoltativo

Solo se best-of-$N$ mostra un plateau.

Anthropic usa Miles, Ray/FSDP o Megatron, SGLang per i rollout e GRPO per l’AV; per la configurazione Qwen2.5-7B riporta SFT su due H100-80GB e una configurazione RL su due nodi da otto H100.

Per il vostro 1.5B dovrebbe essere molto più leggero, ma partirei comunque con:

- 4 GPU H100/A100;
- rollout asincroni;
- AR congelato per i primi update;
- aggiornamenti alternati AV/AR;
- KL penalty verso l’AV SFT;
- reward clipping.

## 11. Esperimenti del paper

### Esperimento 1 — Una traiettoria è più di un singolo stato?

Confrontate:

#### TNLA completo

$$
H_{1:T}\rightarrow s\rightarrow\hat{H}_{1:T}
$$

#### Ultimo stato

$$
h_T\rightarrow s\rightarrow\hat{H}
$$

#### Mean pooling

$$
\bar{h}\rightarrow s\rightarrow\hat{H}
$$

Altri confronti:

- NLA indipendente per ogni stato + sintesi testuale;
- autoencoder numerico non testuale con bottleneck di capacità confrontabile;
- traiettoria shuffled.

Metriche:

- cosine medio per stato;
- delta cosine;
- errore sulla Gram matrix;
- performance distinta su early/middle/late states;
- numero di token del bottleneck.

**Risultato necessario:** TNLA deve ricostruire le transizioni meglio di mean pooling, ultimo stato e shuffled trajectory.

Altrimenti non avete dimostrato che la sequenza serve.

### Esperimento 2 — Il modello usa davvero l’ordine?

Testate:

$$
H=(h_1,\ldots,h_T)
$$

contro:

$$
H_{\text{reverse}}=(h_T,\ldots,h_1)
$$

e permutazioni casuali.

Misurate:

- se la spiegazione cambia;
- se l’AR ricostruisce l’ordine corretto;
- se le espressioni “initially”, “then”, “finally” sono coerenti;
- se un classifier dal testo recupera la permutazione temporale.

Un modello che produce la stessa spiegazione per ordered e shuffled non interpreta una traiettoria: interpreta un insieme di stati.

### Esperimento 3 — Robustezza semantica del testo

Per ogni descrizione $s$:

- parafrasi;
- sinonimi;
- traduzione inglese → italiano → inglese;
- word shuffle;
- rimozione di “early/middle/late”;
- testo casuale della stessa lunghezza.

Il pattern desiderato:

$$
R(s)\approx R\!\left(\operatorname{Paraphrase}(s)\right)
$$

ma:

$$
R(s)\gg R\!\left(\operatorname{Shuffle}(s)\right)
$$

Questo è uno degli esperimenti più importanti del paper.

### Esperimento 4 — Proprietà operative del processo

Qui entrano le label, ma solo per validare.

#### ProsQA

Dal testo si può prevedere:

- branching elevato vs basso;
- soluzione corretta vs errata;
- convergenza precoce vs tardiva.

#### ProcessBench

Dal testo si può prevedere:

- soluzione corretta;
- primo step errato;
- errore early vs late.

#### IFEval

Dal testo si può prevedere:

- rispetto completo dei vincoli;
- quale vincolo viene violato;
- violazione semplice vs multipla.

#### MoralChain

Dal testo si può prevedere:

- baseline;
- armed;
- release;
- pattern *early-misaligned → late-aligned*.

Confrontate sempre:

- prompt-only;
- output-only;
- prompt + output;
- probe diretto sugli stati;
- TNLA text.

Il probe diretto probabilmente sarà più accurato. Non è un fallimento: il vantaggio del TNLA è produrre una rappresentazione leggibile.

### Esperimento 5 — Generalizzazione tra task

Train:

- ProsQA + ProntoQA.

Test:

- GSM8K;
- ProcessBench.

Poi:

- train math;
- test logic.

Una descrizione che funziona solo dentro lo stesso dataset potrebbe aver imparato scorciatoie del dominio. Una descrizione che trasferisce è evidenza di primitive più generali.

### Esperimento 6 — Text-to-latent editing causale

Questo sarebbe il risultato più forte.

Supponete che l’AV produca:

> The trajectory maintains several alternatives initially and commits only in the final states.

Editate il testo:

> The trajectory commits to one alternative immediately and does not revisit it.

Fate:

$$
s_{\text{edit}}
\xrightarrow{\text{AR}}
\widetilde{H}
$$

e reiniettate gradualmente:

$$
H'=(1-\lambda)H+\lambda\widetilde{H}
$$

con:

$$
\lambda\in\{0.1,0.25,0.5\}
$$

Misurate se cambia:

- answer accuracy;
- numero di alternative rappresentate;
- capacità di backtracking;
- constraint adherence;
- comportamento morale nel case study safety.

Controlli:

- rumore norm-matched;
- vettori di un altro esempio;
- traiettoria shuffled;
- edit semantico irrilevante;
- stessa edit senza modificare il latente.

Se le edit testuali producono cambiamenti specifici e ripetibili, avete una dimostrazione causale molto più forte della sola ricostruzione.

## 12. Infrastruttura

### Stack consigliato

- PyTorch;
- Hugging Face Transformers;
- repository ufficiale Coconut;
- fork del repository NLA;
- SGLang per rollout con `input_embeds`;
- FSDP per training distribuito;
- Miles/Ray solo nella fase RL;
- Zarr + Parquet per dataset;
- W&B o MLflow;
- Hydra per configurazioni;
- Apptainer/container riproducibile sul cluster.

Anthropic usa proprio SGLang per passare `input_embeds` all’AV e Miles con FSDP/Megatron per il training distribuito.

### Struttura repository

```text
trajectory-nla/
├── configs/
│   ├── coconut/
│   ├── datagen/
│   ├── av_sft/
│   ├── ar_sft/
│   └── rl/
├── target_model/
│   ├── coconut_adapter.py
│   └── activation_hooks.py
├── data/
│   ├── generate_rollouts.py
│   ├── build_teacher_targets.py
│   └── storage.py
├── tnla/
│   ├── verbalizer.py
│   ├── reconstructor.py
│   ├── adapters.py
│   ├── losses.py
│   └── prompts.py
├── experiments/
│   ├── reconstruction.py
│   ├── order_ablation.py
│   ├── paraphrase_test.py
│   ├── concept_eval.py
│   └── causal_editing.py
└── tests/
```

### Job organization

- job array per dataset, per seed e per decoding setting;
- un job separato per activation extraction;
- checksum su modello, tokenizer e dataset;
- salvataggio periodico dei sample decodificati;
- tre seed per ogni risultato principale;
- split per template/problema, non solo per rollout.

## 13. Risorse di calcolo

### Pilot

- 1–2 A100/H100;
- GPT-2 Coconut;
- 5.000–10.000 traiettorie;
- AV/AR piccoli;
- nessun RL.

Obiettivo: verificare entro due settimane che ordered trajectory batta shuffled trajectory.

### Esperimento principale 1.5B

Una configurazione ragionevole:

- target Coconut: $4\times$ A100/H100;
- generation/extraction: 2–4 GPU in parallelo;
- AV SFT: 2–4 GPU;
- AR SFT: 2–4 GPU;
- best-of-8: 4 GPU;
- GRPO: 4–8 GPU, solo alla fine.

Stima ingegneristica, non benchmark garantito:

| Fase | GPU-hours indicative |
|---|---:|
| Coconut 1.5B | 150–400 |
| Generazione traiettorie | 50–150 |
| AR SFT | 50–150 |
| AV SFT | 50–150 |
| Best-of-$N$ | 50–200 |
| Esperimenti/ablation | 150–300 |
| GRPO opzionale | 200–600 |

Totale senza GRPO:

$$
\approx 450\text{–}1{,}350\ \text{GPU-hours}
$$

Con Cineca è realistico, ma non lancerei il blocco grande finché il pilot non supera i criteri minimi.

## 14. Criteri go/no-go

### Gate 1 — Il target latent reasoner funziona

- accuracy non collassa passando da CoT a continuous thoughts;
- le traiettorie non sono tutte quasi identiche;
- salvataggio e reiniezione sono deterministici.

### Gate 2 — Esiste informazione sequenziale

TNLA ordered deve battere:

- mean pool;
- last state;
- shuffled input;

sulla delta reconstruction.

Se non succede, il paper “trajectory” non regge.

### Gate 3 — Il bottleneck è semantico

La parafrasi deve conservare buona parte della reconstruction performance.

Se una minima parafrasi distrugge tutto, avete costruito un codec testuale, non un metodo interpretativo.

### Gate 4 — Il testo è utile

Le descrizioni devono aiutare a distinguere almeno alcune proprietà operative su dataset held-out.

### Gate 5 — Causalità

Almeno un text edit deve produrre un cambiamento specifico superiore a rumore e controlli casuali.

Questo è desiderabile, ma non indispensabile per il primo paper se i primi quattro risultati sono forti.

## 15. Timeline realistica

### Settimane 1–2

- riproduzione NLA;
- riproduzione Coconut;
- activation hooks;
- pilot GPT-2;
- ordered/shuffled sanity check.

### Settimane 3–5

- Coconut Qwen2.5-1.5B;
- generazione dei 100k rollout;
- teacher descriptions;
- infrastruttura dati.

### Settimane 6–8

- AR SFT;
- AV SFT;
- round-trip;
- tuning di state/delta/geometry losses.

### Settimane 9–10

- best-of-$N$;
- paraphrase robustness;
- baselines e ablation.

### Settimane 11–13

- ProcessBench;
- IFEval;
- cross-task generalization;
- MoralChain case study.

### Settimane 14–16

- causal text editing;
- human evaluation;
- figures;
- scrittura.

## 16. Il paper minimo e quello ambizioso

### Paper minimo credibile

- Introduzione del Trajectory Natural Language Autoencoder.
- Continuous-thought trajectories da Coconut.
- Ricostruzione di stati e transizioni.
- Ordered batte shuffled, mean pool e last state.
- Robustezza alla parafrasi.
- Descrizioni utili su reasoning correctness/error localization.

Questo è già un contributo metodologico coerente.

### Paper ambizioso

Aggiunge:

- generalizzazione tra task;
- MoralChain;
- editing in linguaggio naturale;
- ricostruzione e reiniezione causale;
- scoperta di pattern ricorrenti come *exploration → narrowing → commitment*.

La frase centrale diventerebbe:

> Natural language can serve not only as a readable bottleneck for individual activations, but as a semantically robust and causally editable representation of continuous latent reasoning trajectories.

La mia scelta operativa netta sarebbe quindi:

> Pilot GPT-2 Coconut → main model Qwen2.5-1.5B Coconut → train unsupervised TNLA su ProsQA/ProntoQA/GSM8K → validate temporal semantics con ProcessBench e IFEval → safety solo come case study MoralChain → causal text editing come risultato finale forte.
