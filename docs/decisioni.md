# Registro delle decisioni e delle assunzioni

Qui teniamo traccia delle scelte fatte nel progetto, del perché le abbiamo fatte e di cosa comportano. Serve da base per scrivere le descrizioni del notebook e per preparare la presentazione.

**Come si usa**

- Ogni voce ha un ID (`D-<capitolo>-<n>`). Da qui in avanti non si rinumera e non si cancella: se una decisione cambia, si segna come *superata* e si aggiunge una voce nuova che la richiama, così resta traccia di come ci siamo arrivati.
- **Stato**: `concordata` (decisa dal gruppo), `proposta` (da confermare), `superata`.
- **Fonte**: da dove viene la decisione (documento, riunione, esperimento).
- I punti ancora da chiarire stanno in fondo, in [Questioni aperte](#questioni-aperte).

## Indice

| ID | Capitolo | Decisione | Stato |
|---|---|---|---|
| [D-1-01](#d-1-01-unire-i-dataset-a-e-b-in-un-unico-dataset) | 1 | Unire i dataset A e B in un unico dataset | concordata |
| [D-1-02](#d-1-02-colonne-di-provenienza-e-rimozione-dei-duplicati) | 1 | Colonne di provenienza `in_A` / `in_B` e un'unica copia dei prompt duplicati | concordata |
| [D-1-03](#d-1-03-etichette-numeriche-jailbreak--1-benigno--0) | 1 | Etichette numeriche: jailbreak = 1, benigno = 0 | proposta |
| [D-1.1-01](#d-11-01-prima-lettura-qualitativa-dei-prompt) | 1.1 | Leggere a mano un campione di prompt prima di qualsiasi analisi | concordata |
| [D-1.1-02](#d-11-02-analisi-delle-caratteristiche-di-base) | 1.1 | Analizzare le caratteristiche di base (lunghezza, termini, tipo di richiesta) | concordata |
| [D-1.1-03](#d-11-03-classificatori-naive-costruiti-passo-per-passo) | 1.1 | Classificatori naive costruiti passo per passo, come baseline | concordata |
| [D-1.2-01](#d-12-01-preprocessing-del-testo-con-nltk) | 1.2 | Preprocessing con `nltk`: lowercasing, tokenizzazione, lemmatizzazione | concordata |
| [D-1.2-02](#d-12-02-le-stop-word-non-vengono-rimosse) | 1.2 | Le stop word non vengono rimosse (cambio rispetto al piano iniziale) | concordata |
| [D-1.2-03](#d-12-03-rappresentazioni-del-testo-da-confrontare) | 1.2 | Quattro rappresentazioni del testo da confrontare | concordata |
| [D-1.2-04](#d-12-04-separabilità-misurata-con-k-means-a-k--2) | 1.2 | Separabilità delle classi misurata con k-means a k = 2 | concordata |
| [D-1.2-05](#d-12-05-metriche-omogeneità-completezza-e-v-measure) | 1.2 | Metriche: omogeneità, completezza e v-measure | proposta |
| [D-1.2-06](#d-12-06-ogni-clustering-è-anche-visualizzato) | 1.2 | Ogni clustering è accompagnato da una visualizzazione | concordata |
| [D-1.2-07](#d-12-07-ipotesi-separabilità--prestazioni-del-classificatore) | 1.2 | Ipotesi: la rappresentazione che separa meglio dà il classificatore migliore | concordata |
| [D-1.3-01](#d-13-01-clustering-dei-soli-prompt-di-jailbreak) | 1.3 | Clustering dei soli jailbreak per individuarne i tipi | concordata |
| [D-1.3-02](#d-13-02-tre-algoritmi-di-clustering-a-confronto) | 1.3 | Tre algoritmi a confronto: k-means, gerarchico, DBSCAN | concordata |
| [D-1.3-03](#d-13-03-scelta-dei-cluster-con-metriche-e-ispezione-visiva) | 1.3 | Scelta dei cluster con metriche più ispezione visiva | concordata |
| [D-1.3-04](#d-13-04-tassonomia-dei-tipi-di-jailbreak-riusata-nei-capitoli-successivi) | 1.3 | Tassonomia dei tipi di jailbreak riusata nei capitoli 2 e 3 | concordata |
| [D-1.4-01](#d-14-01-campioni-di-c-e-d-usati-come-verifica) | 1.4 | Campioni di C e D usati per verificare le conclusioni ottenute su A e B | concordata |
| [D-1.4-02](#d-14-02-regola-per-decidere-se-rifare-il-clustering) | 1.4 | Regola per decidere se rifare il clustering includendo C e D | concordata |

---

## Capitolo 1: esplorazione, visualizzazione e clustering

### Il filo logico del capitolo

Il capitolo 1 procede per domande successive, e ogni sezione usa quello che ha trovato la precedente:

1. **Preparazione (D-1-01 … D-1-03).** Costruiamo un unico dataset pulito da A e B, senza duplicati e con etichette numeriche. Tutto il resto del progetto lavora su questa base.
2. **1.1: com'è fatto un jailbreak?** Prima leggiamo i prompt, poi misuriamo caratteristiche semplici, poi vediamo quanto si riesce a classificare con sole regole scritte a mano. Le osservazioni di questa sezione guidano anche il preprocessing di 1.2 (è da qui che nasce la scelta di tenere le stop word).
3. **1.2: quale rappresentazione separa le due classi?** Trasformiamo i testi in vettori in quattro modi diversi e controlliamo, senza usare le etichette, se jailbreak e prompt benigni formano due gruppi distinti. Ne esce un'ipotesi da verificare nel capitolo 2.
4. **1.3: quali tipi di jailbreak esistono?** Raggruppiamo solo i jailbreak e diamo un nome a ogni gruppo. Questa tassonomia viene riusata nei capitoli 2 e 3.
5. **1.4: le conclusioni reggono su dati nuovi?** Usiamo campioni di C e D, mai visti prima, per verificare sia la rappresentazione scelta in 1.2 sia la tassonomia di 1.3.

### Decisioni trasversali al capitolo 1

#### D-1-01: Unire i dataset A e B in un unico dataset

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo ([proposta-michelangelo.md](proposta-michelangelo.md))
- **Decisione:** prima di qualsiasi analisi uniamo A (*Prompt Injections Benchmark*) e B (*Jailbreak Classification*) in un unico dataset, e da lì in avanti lavoriamo solo su quello.
- **Motivazione:**
  - I due dataset hanno la stessa struttura: un prompt e la sua etichetta. Unirli non richiede di riconciliare colonne diverse.
  - Analizzarli separatamente vorrebbe dire ripetere due volte le stesse analisi senza un vantaggio chiaro.
  - Studiare un dataset alla volta rischia di farci prendere per caratteristica generale una particolarità di uno solo dei due (overfitting sul dataset), o di non vedere schemi che emergono solo con più dati.
- **Conseguenze:** tutte le analisi del capitolo 1 e l'addestramento del capitolo 2 usano il dataset unito. Unire non vuol dire perdere l'informazione sulla provenienza: la tracciamo con D-1-02, così possiamo comunque confrontare A e B quando serve (ad esempio per controllare che una caratteristica non valga solo per uno dei due).

#### D-1-02: Colonne di provenienza e rimozione dei duplicati

- **Stato:** concordata
- **Fonte:** discussione di gruppo
- **Decisione:**
  - Nel dataset unito aggiungiamo due colonne binarie (one-hot), `in_A` e `in_B`, che valgono 1 se il prompt compare rispettivamente in A o in B.
  - Un prompt presente in entrambi i dataset compare una sola volta, con `in_A = 1` e `in_B = 1`. Le copie in più vengono eliminate.
- **Motivazione:**
  - Le colonne ci permettono di ricostruire in ogni momento da quale dataset viene un prompt, anche dopo l'unione, e di misurare quanto A e B si sovrappongono.
  - I duplicati vanno tolti perché altrimenti pesano il doppio nelle statistiche e nel clustering. Soprattutto, nel capitolo 2 lo stesso prompt potrebbe finire sia nel training sia nel test set, gonfiando le prestazioni misurate (*data leakage*).
- **Conseguenze:** nel notebook riportiamo quanti prompt cadono in ciascuna delle tre combinazioni (solo A, solo B, entrambi): è un dato utile da commentare, perché una sovrapposizione ampia vorrebbe dire che i due dataset sono in parte la stessa fonte.
- **Da chiarire:** Q-01 (prompt uguali con etichette diverse) e Q-02 (cosa conta come duplicato).

#### D-1-03: Etichette numeriche: jailbreak = 1, benigno = 0

- **Stato:** proposta
- **Fonte:** discussione di gruppo
- **Decisione:** convertiamo le etichette testuali in numeri con pandas (ad esempio con `Series.map`): `jailbreak` → 1, prompt benigno ("sano") → 0.
- **Motivazione:**
  - Con etichette numeriche si calcolano direttamente medie e proporzioni (la media della colonna è la percentuale di jailbreak), e le metriche di scikit-learn funzionano senza altre conversioni.
  - Mettere il jailbreak come classe positiva (1) è coerente con la scelta di dare priorità alla *recall* sui jailbreak nel capitolo 2: non vogliamo lasciarne passare nessuno.
- **Da chiarire:** verificare quali sono esattamente i valori delle etichette in A e in B (nome e maiuscole, ad esempio `benign` o altro) prima di scrivere la mappatura. Dopo la conversione conviene controllare che non sia rimasto nessun valore mancante, segno di un'etichetta non prevista.

### 1.1 Caratteristiche di base

Lo scopo di questa sezione è capire *a occhio e con misure semplici* in cosa un jailbreak differisce da un prompt normale, prima di usare qualsiasi tecnica di machine learning. Le tre decisioni vanno lette in sequenza: si osserva (D-1.1-01), si misura (D-1.1-02), si prova a classificare con quello che si è misurato (D-1.1-03).

#### D-1.1-01: Prima lettura qualitativa dei prompt

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo e discussione di gruppo
- **Decisione:** prima di calcolare qualsiasi statistica leggiamo a mano un campione di prompt delle due classi, per farci un'idea di come sono scritti e di cosa contengono.
- **Motivazione:** una lettura diretta suggerisce quali caratteristiche vale la pena misurare (ad esempio giochi di ruolo, finte autorizzazioni, istruzioni del tipo "ignora le istruzioni precedenti") e aiuta a interpretare i numeri che verranno dopo. Senza questo passaggio rischiamo di misurare caratteristiche scelte a caso.
- **Conseguenze:** nel notebook mostriamo alcuni esempi rappresentativi per classe, con un breve commento su cosa si nota. È da questa lettura che è emerso il ruolo delle stop word nei jailbreak, poi formalizzato in D-1.2-02.

#### D-1.1-02: Analisi delle caratteristiche di base

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** dopo la lettura qualitativa confrontiamo prompt di jailbreak e prompt benigni su un insieme di caratteristiche semplici:
  - lunghezza del prompt (in caratteri e in parole), con media e distribuzione per classe;
  - parole ed espressioni molto frequenti nei jailbreak e rare o assenti nei prompt benigni, cercando anche sequenze di 2–3 parole (n-gram) e non solo parole singole;
  - tipo di richiesta fatta nel prompt;
  - altre caratteristiche che emergono dalla lettura (D-1.1-01).
- **Motivazione:** sono caratteristiche facili da calcolare e da spiegare. Se già queste separano bene le classi è un risultato interessante, e in ogni caso forniscono le variabili per i classificatori naive (D-1.1-03).
- **Conseguenze:** anche in questa analisi le stop word restano nel testo, per le ragioni spiegate in D-1.2-02: espressioni come *"you are now"* o *"I am the owner"* sono fatte quasi solo di stop word, e toglierle le renderebbe invisibili. Per lo stesso motivo gli n-gram sono importanti: il segnale sta nella formula intera più che nella singola parola.

#### D-1.1-03: Classificatori naive costruiti passo per passo

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo e discussione di gruppo
- **Decisione:** con le caratteristiche trovate in D-1.1-02 costruiamo dei classificatori naive, cioè regole semplici scritte a mano e non addestrate (ad esempio: "se il prompt supera N parole è un jailbreak", oppure "se contiene almeno una delle espressioni X, Y, Z è un jailbreak"). Partiamo da una sola caratteristica e ne aggiungiamo una alla volta, misurando ogni volta le prestazioni.
- **Motivazione:**
  - Il percorso a passi mostra quanto si riesce a ottenere con caratteristiche banali e quanto aggiunge ciascuna. Se una regola sulla sola lunghezza raggiunge già buone prestazioni, è una cosa da dire esplicitamente: significa che il problema, almeno su questi dati, è in parte facile.
  - I classificatori naive fanno da *baseline* per il capitolo 2: un classificatore addestrato ha senso solo se fa meglio di queste regole. È anche lo schema seguito nei tutorial del corso.
- **Conseguenze:**
  - Valutiamo i classificatori naive con le stesse metriche e sugli stessi dati di test del capitolo 2 (recall sui jailbreak in primo luogo), così i confronti sono omogenei.
  - Le soglie delle regole (ad esempio il valore di N) vanno scelte guardando solo i dati di training, altrimenti la baseline risulterebbe migliore di quanto è davvero.
  - Nel capitolo 2 verifichiamo se le parole su cui si basano le regole naive coincidono con quelle che LIME indica come decisive per i classificatori addestrati.

### 1.2 Clustering e separabilità tra prompt benigni e jailbreak

In 1.1 abbiamo guardato caratteristiche scelte da noi. Qui facciamo un passo in più: rappresentiamo ogni prompt come un vettore e lasciamo che sia un algoritmo non supervisionato a dirci se jailbreak e prompt benigni occupano zone diverse dello spazio. L'ordine delle decisioni segue l'ordine del lavoro: si prepara il testo (D-1.2-01, D-1.2-02), lo si rappresenta (D-1.2-03), si fa il clustering e lo si valuta (D-1.2-04 … D-1.2-06), e se ne ricava un'ipotesi per il capitolo 2 (D-1.2-07).

#### D-1.2-01: Preprocessing del testo con nltk

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo e discussione di gruppo
- **Decisione:** prima di costruire le rappresentazioni basate sulle parole applichiamo, nell'ordine:
  1. lowercasing;
  2. tokenizzazione;
  3. lemmatizzazione (preferita allo stemming).

  Per tokenizzazione e lemmatizzazione usiamo `nltk` (ad esempio `word_tokenize` e `WordNetLemmatizer`).
- **Motivazione:**
  - Riduce il rumore e la dimensione del vocabolario: forme diverse della stessa parola (*run*, *running*, *ran*) diventano una sola caratteristica.
  - Preferiamo la lemmatizzazione allo stemming perché produce parole vere (*running* → *run*, invece di troncamenti come *studi* per *studies*): le caratteristiche restano leggibili, il che conta nei grafici dei centroidi e soprattutto nelle spiegazioni LIME del capitolo 2.
- **Conseguenze:**
  - Il preprocessing si applica solo alle rappresentazioni basate sulle parole (conteggi e TF-IDF, D-1.2-03). I modelli di embedding pre-addestrati vanno invece usati sul testo originale: sono stati addestrati su testo normale e hanno un loro tokenizer, quindi lemmatizzare o mettere in minuscolo prima li peggiorerebbe.
  - `WordNetLemmatizer` lavora meglio se gli si indica la parte del discorso (verbo, nome…); senza, tratta tutto come nome e lascia invariati molti verbi. Valutare se aggiungere il POS tagging di `nltk`.
  - `nltk` richiede di scaricare alcune risorse (ad esempio `punkt` e `wordnet`) con `nltk.download`: va fatto in una cella iniziale del notebook, così il notebook gira da zero.
- **Nota:** `nltk` non compare ancora nell'elenco delle librerie consentite in `CLAUDE.md`. È un'omissione nostra: la usiamo, e l'elenco verrà aggiornato in seguito.

#### D-1.2-02: Le stop word non vengono rimosse

- **Stato:** concordata
- **Fonte:** discussione di gruppo (modifica la proposta di Michelangelo)
- **Decisione:** non rimuoviamo le stop word, né nel preprocessing delle rappresentazioni (D-1.2-01) né nell'analisi delle caratteristiche di 1.1 (D-1.1-02).
- **Come ci siamo arrivati:** il piano iniziale, nella proposta di Michelangelo, prevedeva di togliere le stop word come passo standard di pulizia del testo. Leggendo i prompt (D-1.1-01) ci siamo resi conto che nei jailbreak molte di queste parole fanno parte delle formule che caratterizzano l'attacco, e abbiamo cambiato idea.
- **Motivazione:** nei jailbreak ricorrono formule come:
  - chiedere al modello di impersonare qualcuno: *"you are now…"*, *"pretend you are…"*, *"act as…"*;
  - attribuirsi un'autorità o un diritto: *"I am the owner of this copyrighted material"*, *"I am your developer"*.

  Pronomi come *I*, *you* e *your* e verbi come *am* e *are* stanno di solito nelle liste di stop word. Toglierli cancellerebbe proprio questi schemi, che sono tra i segnali più utili per riconoscere un jailbreak.
- **Conseguenze:**
  - Senza filtro, le stop word saranno le parole più frequenti in assoluto. Nei conteggi semplici (`CountVectorizer`) rischiano di dominare; TF-IDF invece ne riduce automaticamente il peso, perché le parole presenti in quasi tutti i prompt ricevono un IDF basso. È un motivo in più per confrontare le due rappresentazioni.
  - Per sostenere la scelta con un dato, oltre che con un ragionamento, possiamo ripetere una volta il clustering a k = 2 (D-1.2-04) con le stop word rimosse e mostrare che la separazione peggiora. È facoltativo, ma è un buon argomento in presentazione.

#### D-1.2-03: Rappresentazioni del testo da confrontare

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** confrontiamo quattro rappresentazioni:
  1. vettori di conteggio delle parole (`CountVectorizer`);
  2. vettori TF-IDF (`TfidfVectorizer`);
  3. embedding prodotti da un modello di text embedding pre-addestrato;
  4. embedding presi dallo strato interno di un classificatore già fine-tuned per riconoscere i jailbreak.
- **Motivazione:** vanno da rappresentazioni semplici e interpretabili (1, 2), che guardano solo quali parole compaiono, a rappresentazioni che catturano il significato della frase (3, 4). Il confronto dice quanto conta la scelta della rappresentazione e se vale la pena usare modelli più pesanti.
- **Conseguenze:**
  - Le rappresentazioni 1 e 2 usano il testo preprocessato (D-1.2-01); la 3 e la 4 il testo originale.
  - Le stesse quattro rappresentazioni vengono riusate nel capitolo 2 per addestrare i classificatori. Solo la 1 e la 2 hanno parole come caratteristiche, quindi solo i classificatori addestrati su di esse si possono spiegare con LIME in modo leggibile.
  - La rappresentazione 4 parte avvantaggiata, perché il modello è già stato addestrato proprio a distinguere i jailbreak. Va presentata come termine di confronto e non come rappresentazione "pari" alle altre.
- **Da chiarire:** quali modelli usare per la 3 e la 4 (Q-07).

#### D-1.2-04: Separabilità misurata con k-means a k = 2

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** per ciascuna rappresentazione applichiamo k-means con k = 2, senza usare le etichette, e poi controlliamo quanto i due cluster corrispondono alle due classi.
- **Motivazione:** ci dice se una rappresentazione separa i jailbreak dai prompt benigni già nello spazio delle caratteristiche, prima di addestrare qualsiasi classificatore. Se i due gruppi emergono da soli, un classificatore avrà vita facile.
- **Conseguenze:** fissiamo il seed di k-means (`random_state`) per avere risultati riproducibili. Se le due classi sono molto sbilanciate, k-means tende a non trovare un cluster piccolo: va tenuto presente nell'interpretare i risultati.

#### D-1.2-05: Metriche: omogeneità, completezza e v-measure

- **Stato:** proposta
- **Fonte:** proposta di Michelangelo
- **Decisione:** valutiamo la corrispondenza tra cluster e classi con tutte e tre le metriche: omogeneità, completezza e v-measure.
- **Motivazione:** misurano aspetti diversi. L'omogeneità dice se ogni cluster contiene una sola classe; la completezza dice se ogni classe sta tutta in un solo cluster; la v-measure è la loro media armonica. Riportarle tutte e tre evita di nascondere un difetto dietro un solo numero: ad esempio, un cluster molto grande che contiene quasi tutto ha completezza alta ma omogeneità bassa.
- **Da chiarire:** confermare se riportarle tutte o sceglierne una principale (Q-03).

#### D-1.2-06: Ogni clustering è anche visualizzato

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** accanto alle metriche mostriamo ogni clustering in un grafico 2D o 3D, dopo una riduzione di dimensionalità.
- **Motivazione:** le metriche danno un numero, il grafico mostra il perché. Con il grafico rispondiamo a domande come: le due classi appaiono separate? Si sovrappongono? E se sì, in quale zona e con quali prompt? I prompt che cadono nella zona di sovrapposizione sono quelli da leggere: probabilmente sono anche quelli che i classificatori del capitolo 2 sbaglieranno.
- **Conseguenze:** conviene colorare i punti due volte, una per cluster assegnato e una per etichetta vera, così il confronto è immediato.
- **Da chiarire:** quale tecnica di riduzione di dimensionalità usare (Q-04).

#### D-1.2-07: Ipotesi: separabilità → prestazioni del classificatore

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** formuliamo un'ipotesi esplicita, da verificare nel capitolo 2: la rappresentazione che meglio divide prompt benigni e jailbreak nei due cluster sarà anche quella con cui i classificatori ottengono le prestazioni migliori.
- **Motivazione:** lega il capitolo 1 al capitolo 2 e dà al notebook una domanda chiara a cui rispondere. Anche una smentita è un risultato: un classificatore supervisionato può trovare un confine tra le classi anche quando k-means, che non conosce le etichette, non lo vede.
- **Idea extra (non decisa):** k-means con valori di k diversi, o DBSCAN, sull'intero dataset per vedere quanti tipi generali di prompt esistono. Si sovrappone in parte con 1.3, quindi è a bassa priorità.

### 1.3 Tipi di jailbreak tramite clustering

In 1.2 la domanda era "jailbreak o no?". Qui la domanda diventa "quali famiglie di jailbreak esistono?". Il risultato di questa sezione, una tassonomia dei tipi di attacco, è uno degli strumenti più riusati nel resto del progetto.

#### D-1.3-01: Clustering dei soli prompt di jailbreak

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** applichiamo il clustering solo ai prompt etichettati come jailbreak, per suddividerli in tipi diversi.
- **Motivazione:** i prompt benigni non servono a distinguere un tipo di attacco da un altro e introdurrebbero solo rumore.
- **Conseguenze:** usiamo la rappresentazione risultata migliore in 1.2, o al massimo un paio delle migliori, per non moltiplicare gli esperimenti.

#### D-1.3-02: Tre algoritmi di clustering a confronto

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** proviamo tre algoritmi:
  - k-means, con il numero di cluster scelto tramite il metodo del gomito (*elbow*);
  - clustering gerarchico, tenendo il taglio del dendrogramma che separa meglio i cluster;
  - DBSCAN.
- **Motivazione:** fanno assunzioni diverse. k-means cerca cluster compatti e di forma sferica; il gerarchico mostra la struttura a più livelli (tipi e sottotipi di attacco); DBSCAN trova cluster di forma qualsiasi e isola i prompt anomali, che non appartengono a nessuna famiglia. Confrontarli evita che il risultato dipenda solo dalle assunzioni di un algoritmo.
- **Da chiarire:** quali metriche usare per il gomito e per il gerarchico (Q-05).

#### D-1.3-03: Scelta dei cluster con metriche e ispezione visiva

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** scegliamo la suddivisione migliore combinando le metriche con l'ispezione visiva del risultato più promettente, e non affidandoci a un solo numero.
- **Motivazione:** su testo le metriche interne di clustering danno spesso indicazioni deboli o contrastanti (il "gomito" a volte non si vede). La visualizzazione e la lettura dei prompt nei cluster dicono se la suddivisione ha senso anche per un umano.

#### D-1.3-04: Tassonomia dei tipi di jailbreak riusata nei capitoli successivi

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** una volta scelta la suddivisione leggiamo i prompt di ciascun cluster, ne guardiamo le parole più caratteristiche (ad esempio quelle con peso maggiore nel centroide) e diamo a ogni cluster un nome che descriva il tipo di attacco (ad esempio "jailbreak di role-playing").
- **Motivazione:** la tassonomia serve dopo:
  - nel capitolo 2, per misurare su quali tipi di jailbreak i classificatori funzionano bene e su quali sbagliano;
  - nel capitolo 3, per capire a quali tipi l'LLM è più vulnerabile e quali difese funzionano meglio contro ciascun tipo.
- **Conseguenze:** salviamo il tipo assegnato come colonna del dataset, così è disponibile nei capitoli successivi senza rifare il clustering.

### 1.4 Verifica con i dataset C e D

Fin qui tutte le conclusioni vengono da A e B. Questa sezione le mette alla prova su dati che non abbiamo usato per costruirle, e decide se vanno corrette.

#### D-1.4-01: Campioni di C e D usati come verifica

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** aggiungiamo campioni dei dataset C (*Mosscap*) e D (*Pliny HackAPrompt*) per verificare se le conclusioni ottenute su A e B restano valide. Le domande sono due:
  - **Rappresentazione (da 1.2):** la rappresentazione migliore separa correttamente anche i prompt di C e D? Se no, dove finiscono i nuovi punti? Con i nuovi prompt una rappresentazione diventa migliore o peggiore delle altre?
  - **Tipi di jailbreak (da 1.3):** i nuovi jailbreak cadono nei cluster trovati?
- **Motivazione:** C e D non sono stati usati per costruire cluster e rappresentazioni, quindi fanno da verifica indipendente. Vengono anche da contesti diversi (giochi e competizioni di prompt injection), quindi è probabile che contengano attacchi che in A e B non ci sono.
- **Conseguenze:**
  - C e D sono grandi, quindi ne usiamo dei campioni (come consentito dalla consegna), estratti con un seed fisso.
  - I nuovi prompt vanno trattati esattamente come quelli di A e B: stesso preprocessing (D-1.2-01) e stessi vettorizzatori, già addestrati su A e B (si usa `transform`, non `fit_transform`). Altrimenti non si verifica la rappresentazione costruita su A e B, ma una nuova.

#### D-1.4-02: Regola per decidere se rifare il clustering

- **Stato:** concordata
- **Fonte:** proposta di Michelangelo
- **Decisione:** fissiamo in anticipo come interpretare il risultato della verifica sui tipi di jailbreak:
  - **Se i nuovi jailbreak cadono nei cluster esistenti:** i tipi trovati su A e B coprono la maggior parte dello spazio dei jailbreak, e la tassonomia si tiene così com'è.
  - **Se molti cadono fuori:** la tassonomia di A e B non è sufficiente, e rifacciamo il clustering dei jailbreak includendo i campioni di C e D, per ottenere una classificazione dei tipi più completa.
- **Motivazione:** decidere la regola prima di vedere i risultati evita di adattare l'interpretazione a posteriori.
- **Conseguenze:** se la tassonomia cambia, i capitoli 2 e 3 usano quella nuova, e lo diciamo esplicitamente nel notebook.
- **Da chiarire:** cosa significa in pratica "cadere in un cluster" e quale soglia fa scattare il nuovo clustering (Q-06).

---

## Questioni aperte

| ID | Questione | Legata a |
|---|---|---|
| Q-01 | Se lo stesso prompt compare in A e in B con etichette diverse, come lo trattiamo? Va scelta una regola (scartarlo, tenere una delle due etichette, controllarlo a mano) e va riportato quanti casi ci sono. | D-1-02 |
| Q-02 | Cosa conta come duplicato: solo testo identico, o anche testo che differisce per spazi o maiuscole? E come trattiamo i duplicati presenti dentro uno stesso dataset? | D-1-02 |
| Q-03 | Riportiamo omogeneità, completezza e v-measure tutte e tre, o ne scegliamo una come metrica principale? | D-1.2-05 |
| Q-04 | Quale tecnica di riduzione di dimensionalità usiamo per i grafici? `TruncatedSVD` è adatta alle matrici sparse di conteggi e TF-IDF, PCA agli embedding densi, t-SNE dà grafici più leggibili ma distorce le distanze. Tutte disponibili in scikit-learn. | D-1.2-06 |
| Q-05 | Quali metriche usiamo per il metodo del gomito (WSS, BSS, Variance Ratio Criterion / Calinski-Harabasz) e per il clustering gerarchico (ad esempio silhouette)? | D-1.3-02 |
| Q-06 | Come misuriamo se un nuovo jailbreak "cade" in un cluster esistente (ad esempio distanza dal centroide entro una soglia, o etichetta di rumore in DBSCAN), e quale percentuale di prompt fuori dai cluster ci spinge a rifare il clustering? | D-1.4-02 |
| Q-07 | Quali modelli usiamo per le rappresentazioni 3 (text embedding generico) e 4 (classificatore fine-tuned per jailbreak)? Per la 4 un candidato è `meta-llama/Llama-Prompt-Guard-2-86M`, già citato nella proposta per il capitolo 4. | D-1.2-03 |
