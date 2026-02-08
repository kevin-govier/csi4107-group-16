# Assignment 1: CSI4107

## Team Members

- Name: Yahya Osman  | Student Number: 30024009
- Name: Kevin Govier | Student Number: 300282040
- Name: Emily Cheng | Student Number: 300299745

## Task Division

- Yanya Osamn: Step 3
- Kevin Govier: Step 1 and 2
- Emily Cheng: README.md

## Program Functionality

### Preprocessing Module

The preprocessing module (preprocessing.py) transforms raw text into a list of tokens that represent meaningful terms in the text, suitable for indexing and retrieval. Markup, punctuation, and stopwords are removed from the raw text, and stemming to reduce tokens to their root forms is available. 

It consists of two functions, load_stopwords, which takes an html file containing stopwords and loads it as a set of individual stopwords, and preprocess, which preforms the preprocessing .

#### Input

load_stopwords
- filepath: str
  - Preset string of path to 'List of Stopwords.html'

preprocess
- text: str
  - Raw text to be tokenized.

- stop_words: Set[str]
  - Set of stopwords, obtained from load_stopwords.

- stem: boolean
  - A boolean flag indicating whether stemming should be applied.

#### Process

load_stopwords (Loading stopwords)
1. The stopword file is read as raw text.
2. HTML tags are removed.
3. The remaining content is split line by line.
4. Leading and trailing whitespace is removed, and empty lines are discarded.
5. The stopwords are stored in a set to ensure uniqueness and allow efficient lookup.

preprocess (Text Preprocessing)
1. All characters in the input text are converted to lowercase to ensure case-insensitive matching.
2. Non-alphabetic characters, excluding whitespace, are removed using a regular expression, leaving only lowercase alphabetic characters and spaces.
3. The cleaned text is tokenized into individual words using NLTK’s word_tokenize function.
4. Tokens that appear in the stopword set are removed to eliminate high-frequency terms with low semantic value.
5. If stemming is enabled, the Porter Stemmer is applied to each remaining token to reduce words to their root forms.

#### Output

load_stopwords
- query_ids: set[str]
  - Set of stopwords

preprocess 
- tokens: List[str]
  - List of tokenized words from text

### Indexing Module

The indexing module (indexing.py) builds an inverted index. The inverted index maps each word in a given set of strings to the documents in corpus body and the frequency it appears as. 

The structure of the inverted index is a dictionary which contains the word and another dictionary which holds key value pairs of the document id and the frequency the word appears in that document. 

#### Input 

- corpus_path: string
  - The filepath to the corpus body. Each document in the corpus contains a unique document identifier (_id), a title (title), and text (text) fields.

- stop_words: set of string
  - A set of stopwords

- stem: boolean
  - A boolean flag indicating if stemming has been applied to the stopwords or not

#### Process

1. The corpus is opened and read line by line.
2. For each line, the title and text fields are extracted and concatenated.
3. The function preprocess is used on the concatenated text to return tokens list
4. The document id is extracted from the unique docment indentifier field.
5. For each word in tokens list, the frequency value in the inverted index for that word and document id increments 

#### Output

- inverted_index: Dict[str, Dict[str, int]] -> Dict[(word), Dict[(doc_id), (frequency)]]
  - Inverted index

### Retrieval/Ranking Module

The retrieval and ranking module (query.py) computes the similarity between a query and documents in a collection using the vector space model with TF–IDF weighting and cosine similarity. 

It consists of 3 private functions:

_collect_doc_ids 
  
  Extracts and returns the set of document ids from the inverted index

_compute_idf 
  
  Calculates the inverse document frequence for a query based on document count and total number of documents

_compute_doc_norms 
  
  Calculates the vector norms using TF-IDF weights for the inverted index

And 1 public function:
queryData 
  
  Processes queries, computes similarity scores, and returns a ranked list of documents in order of decreasing similarity.

#### Input
queryData
- inverted_index: (Dict[str, Dict[str, int]])
  - An inverted index mapping terms to document identifiers and term frequencies.

- query: (str)
  - Raw query text.

- stop_words (Set[str])
  - A set of stopwords.

- stem (bool)
  - A boolean flag indicating whether stemming should be applied to the query.

- doc_norms ([Dict[str, float]])
  - Optional precomputed document vector norms for cosine normalization.

#### Process

1. Query text is preprocessed using the preprocess function. Lowercasing, tokenization, stopword removal, and stemming is applied.
2. Term frequencies in the query are computed using a frequency counter. For each query term, a TF–IDF weight is calculated.
3. If document vector norms are not provided, they are computed here by iterating over the inverted index and calculating the TF–IDF weights for all document terms.
4. For each query term that appears in the inverted index, the module retrieves the corresponding postings list and accumulates the dot product between the query vector and document vectors using TF–IDF weights.
5. Accumulated dot products are normalized by the product of the query norm and the corresponding document norm, resulting in cosine similarity scores.
6. Documents with non-zero similarity scores are sorted in descending order of cosine similarity to produce the final ranked list.

#### Output

- ranked
  - List[Tuple[str, float]]


### Output Format and Results Generation

Generation is finalized in the file main.py, putting together the final retrieval results for all test queries and writing them to a single output file in the standard TREC evaluation format.

#### Input

load_test_query_ids
- path: str
  - string for filepath to "test.tsv"

prompt_non_empty
- prompt: str
  - Ensures prompt is not empty and strips prompt of whitespace

main
- corpus.jsonl
  - The document collection used to build the inverted index.

- queries.jsonl
  - A JSONL file containing all queries.

- test.tsv
  - A relevance judgments file used to identify which queries belong to the test set.

- run_name (str)
  - A user-provided identifier for the retrieval run, included in every output line.

#### Process

1. Stopword list is loaded from the preprocessing module.
2. Inverted index is built from the document corpus using the indexing module.
3. Prompts for run name and checks entered string with prompt_non_empty
4. Query ids are loaded from test.tsv with load_test_query_ids.
5. Read queries.jsonl line by line and skips queries not in the test set. For each test query, the retrieval and ranking module is used to compute and order similarity scores between the query and documents in the corpus.
6. Top 100 retrieved documents are retrieved and written to the RESULTS.txt file.

#### Output

The ranked results are written to the output file, RESULTS.txt, using the TREC run format.
    
    query_id Q0 document_id rank score run_name


## How to Run

- Environment setup:
  - Python 3.11+ recommended
  - If you cannot install `nltk` system-wide, use a virtual environment:
    - `python3 -m venv Assignment-1/.venv`
    - `source Assignment-1/.venv/bin/activate`
    - `python -m pip install --upgrade pip`
    - `python -m pip install nltk`
  - NLTK data downloads:
    - `python -c "import nltk; nltk.download('punkt')"`
    - `python -c "import nltk; nltk.download('punkt_tab')"`
- Run commands:
  - `python Assignment-1/main.py > Assignment-1/RESULTS.txt`
- Output:
  - Results file path: `Assignment-1/RESULTS.txt`
  - Note: each run overwrites `Assignment-1/RESULTS.txt`
  - Note: only queries listed in `Assignment-1/qrels/test.tsv` are processed

## Algorithms, Data Structures, and Optimizations

### Step 1: Preprocessing

#### Tokenization

Input text is lowercased, then tokenized into words using NLTK’s word_tokenize to split text on whitespace and punctuation.

#### Stopword Removal

Tokens matching any word in a predefined stopword set loaded from “List of Stopwords.html” are filtered out to remove high frequency, low semantic value words.

#### Stemming

Porter Stemmer is used to reduce tokens to their root forms to improve search and matching.

#### Regex/Normalization

Regex is used to remove all non-alphabetic characters except whitespace, cleaning punctuation and numbers before tokenization.

### Step 2: Indexing

#### Inverted Index Structure
Uses dictionary mapping for dynamic index building. Dictionary maps terms to a second dictionary, where the second dictionary maps document IDs to term frequencies:

Dict[str, Dict[str, int]]

#### Term Frequency Storage
Term frequencies within each document are mapped within the dictionary for efficient incremenation.

### Step 3: Retrieval and Ranking

#### Candidate Document Selection
Only documents containing at least one query term are retrieved by accessing the inverted index entries for those terms.

#### TF-IDF Weighting
Term frequency-inverse document frequency weights using smoothed idf:

    idf = log((doc_count + 1) / (doc_freq + 1)) + 1

#### Cosine Similarity
Scores are calculated as the normalized dot product between TF-IDF weighted query and document vectors, with precomputed document norms for efficiency.

## Vocabulary Statistics

- Vocabulary size: 
    
    29953
    
- Sample 100 tokens: 
    
    ['microstructural', 'development', 'human', 'newborn', 'cerebral', 'white', 'matter', 'assessed', 'vivo', 'diffusion', 'tensor', 'magnetic', 'resonance', 'imaging', 'alterations', 'architecture', 'developing', 'brain', 'affect', 'cortical', 'result', 'functional', 'disabilities', 'line', 'scan', 'weighted', 'mri', 'sequence', 'analysis', 'applied', 'measure', 'apparent', 'coefficient', 'calculate', 'relative', 'anisotropy', 'delineate', 'dimensional', 'fiber', 'preterm', 'full', 'term', 'infants', 'assess', 'effects', 'prematurity', 'early', 'gestation', 'studied', 'central', 'mean', 'wk', 'microm', 'decreased', 'posterior', 'limb', 'internal', 'capsule', 'coefficients', 'versus', 'closer', 'birth', 'absolute', 'values', 'areas', 'compared', 'nonmyelinated', 'fibers', 'corpus', 'callosum', 'visible', 'marked', 'differences', 'organization', 'data', 'indicate', 'quantitative', 'assessment', 'water', 'insight', 'living', 'induction', 'myelodysplasia', 'myeloid', 'derived', 'suppressor', 'cells', 'myelodysplastic', 'syndromes', 'mds', 'age', 'dependent', 'stem', 'cell', 'malignancies', 'share', 'biological', 'features', 'activated', 'adaptive']

## Results (Qualitative)

First 10 answers for Query 1:
 [doc_id] [score]
- 10608397 0.089398
- 10607877 0.083132
- 31543713 0.074067
- 10931595 0.065463
- 13231899 0.064990
- 25404036 0.060988
- 6863070  0.059027
- 9580772  0.058704
- 40212412 0.058345
- 16939583 0.055992

First 10 answers for Query 2:

[doc_id] [score]
- 23389795 0.363334
- 2739854  0.327502
- 14717500 0.256770
- 4632921  0.216234
- 8411251  0.191888
- 3672261  0.156045
- 4378885  0.153725
- 32181055 0.149510
- 4414547  0.143268
- 14019636 0.139733

## Evaluation

Creating qrels.txt:
    
    awk '{print $1, 0, $2, $3}' "/qrelstest.tsv" > "/qrels/qrels.txt"

trec_eval command used: 
    
    ./trec_eval "/qrels/qrels.txt" "/RESULTS.txt"

MAP score: 
    
    0.5300

## Discussion

The MAP score of 0.5300 indicates that the retrieval algorithm is effective at ranking relevant documents. The top 10 results for the first two queries show a smooth decrease in similarity scores, suggesting that the TF–IDF weighting and cosine similarity are functioning as intended.

The relatively large vocabulary size (29,953 terms) reflects the diversity of the document collection and helps explain why similarity scores are relatively small, as term weights are distributed across many unique words. Stemming and stopword removal helps reduce noise in the index but some relevant documents may still be missed when they do not share exact query terms with the documents.

Most retrieval errors are likely due the inability of TF-IDF to capture semantic relationships or synonymy between terms.
