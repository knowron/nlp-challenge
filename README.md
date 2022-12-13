<h3 align="center">
    <img width="400" src="https://user-images.githubusercontent.com/22967053/206445506-51202ec8-eea7-477e-9661-aed615139b5e.png" alt="KNOWRON Logo">
</h3>

<h1 align="center">NLP Challenge</h1>
<p align="center">Welcome to the KNOWRON challenge for NLP Engineers!</p>

<br>

## Goals 🎯

The main objective of this challenge is to test your skills in Natural Language
Processing.

```console
Note: There's more to NLP than Transformers ;)
```

<br>

However, apart from NLP knowledge, as NLP *Engineer* is also important to have
good skills on:

- Problem solving.
- Coding style.
- Knowledge on frameworks and other technologies in the industry.
- Good practices.
- Attention to detail.

<br>

## Rules 🫱🏻‍🫲🏼

There are only 3 simple rules to follow:

1. Your code should be made available in a private repository on your personal
   GitHub account. Check the section
   [Sending the Challenge](https://github.com/knowron/nlp-challenge-v2#sending-the-challenge-)
   for the submission details.
2. Ideally, you can deliver the challenge within 7 days. But if you need more
   time, let us know :)
3. Have fun! 😀

<br>

## The Challenge 🎲

<br>

> **⚠️ Please, read everything carefully before starting to code!**


### Problem statement

```console
We want to build a search engine similar to Google, Bing, DuckDuckGo, etc.
```

However, there are **two notable differences**:

1. **Our knowledge base** doesn't come from scraping the web, but rather from
   **documents that our clients upload to our system**.
2. The **nature of the texts** is mostly **technical**. Our domain is mainly
   **industry-related** (e.g., a manual for an industrial cooling system).

<br>

The pipeline for this challenge is simplified and will roughly look like
follows:

1. All the relevant documents are stored in a local directory.
2. Upon server startup, the text from these documents is extracted and stored in
   memory.
3. At this point, queries can be run against our indexed documents in order to
   search for relevant information. The queries can be formulated in natural
   language (e.g., "What does coolant mean?", or using keywords (e.g., "coolant
   meaning").

```console
Note: you can assume the language of all documents is English.
```

<br>

### Cool, but too high-level...

You're right, there are still a lot of missing details. Let's make it a bit more
concrete.


#### REST API

Our content extraction system and search engine will be run on a server. For
this, you can use [Flask](https://flask.palletsprojects.com/en/2.2.x/). It's
simple yet powerful enough for our purposes. Don't worry, below you can find
some code snippets of a minimal Flask server setup.

<br>


#### Text Extraction and Indexing

For text extraction, we recommend using
[PyMuPDF](https://pymupdf.readthedocs.io/en/latest/). Upon server startup, all
the documents situated in
[`/docs`](https://github.com/knowron/nlp-challenge-v2/tree/main/docs) have to be
read in order to extract their text.

When it comes to indexing and search, our the goal for the challenge is not to
develop the next Google. To keep things as simple as possible, you can use the
module [rank-bm25](https://pypi.org/project/rank-bm25/), which implements the
BM25 algorithm. This module is enough to store the extracted text (in memory)
and be able to search it (yes, BM25 only considers syntactic overlap, but that's
enough for the challenge ;).

You can find below an example that opens a single document and indexes its
content. Keep in mind that for the challenge, all documents in
[`/docs`](https://github.com/knowron/nlp-challenge-v2/tree/main/docs) have to be
opened and extracted.

```python
import fitz
from rank_bm25 import BM25Plus
from flask import Flask, request
from typing import List

FILE_PATH: str = "docs/doc_1.pdf"

app = Flask(__name__)


def _extract_text_from_doc(doc: fitz.fitz.Document) -> List[str]:
    pass  # TODO


def _index_passages(passages: List[str]) -> BM25Plus:
    preprocessed_passages = [_preprocess_passage(p) for p in passages]
    return BM25Plus(preprocessed_passages)


def _preprocess_passage(passage: str) -> List[str]:
    # You can use fancier pre-processing if you want :)
    # Some sort of tokenization is needed though. Here we just do a
    # very simple whitespace tokenization.
    return passage.split()


# The following code is run upon server startup.
doc = fitz.open(FILE_PATH)
corpus: List[str] = _extract_text_from_doc(doc)
bm25 = _index_passages(corpus)
```

<br>

#### Search

Then we can add a search endpoint:

##### 📡 `GET /search`

This endpoint lets users upload a document. It admits a query parameter `q` with
the search query. E.g.: `GET /search?q=What does coolant mean?`. Following the
previous example, we provide some extra code (the ellipses "..." mean omitted
code from above).

```python
...

from io import StringIO
from typing import List, Optional

FILE_PATH: str = "docs/doc_1.pdf"

app = Flask(__name__)


...


def _search(
    bm25: BM25Plus,
    query: str,
    corpus: List[str],
    top_k: Optional[int] = 5
) -> List[str]:
    """Run query on our knowledge base.

    Args:
        bm25 (:obj:`BM25Plus`):
            The indexed corpus, i.e., our knowledge base.
        corpus (:obj:`List[str]`):
            The unprocessed corpus.
        query (:obj:`str`):
            The query to use for the search.
        top_k (:obj:`Optional[int], defaults to 5`):
            The maximum number of matching passages to
            return.

    Returns:
        :obj:`List[str]`: The passages found, if any.
    """
    pass  # TODO: check rank-bm25 docs :)


# The following code is run upon server startup.
doc = fitz.open(FILE_PATH)
corpus: List[str] = _extract_text_from_doc(doc)
bm25 = _index_passages(corpus)


@app.route("/search", methods=['GET'])
def search_view():
    query = request.args.get("q")
    top_passages = None
    if query:
        top_passages = _search(bm25, query, corpus)
    if top_passages:
        response = StringIO()
        response.write("<p>Here is what I found:</p><br>")
        for passage in top_passages:
            response.write(f"<p>{passage}</p><br>")
        return response.getvalue()
    return "<p>Nothing found.</p>"
```

<br>

### One last thing

We now have all the ingredients we need to solve the challenge. However, we
would like to point out one last thing. We are especially interested in how well
the content is extracted.

As you might have guessed from the code snippets above, the text has to be
separated into passages. **Each passage will have a title and a text.** This
way, we can present the search results to users in a more meaningful way (think
how search results in popular search engines are served, for example).

**How you split passages is up to you, but make sure the title is meaningful!**
Also, keep in mind that BM25 penalizes length, and in any case, if a search
result is too long it will be hard for the user to know exactly which
information matched. You can use third-party libraries for the text splitting.

Since the rank-bm25 only admits strings, we need a way of converting Passage
objects into strings and vice versa. How you do this is up to you. This is how a
Passage can look like:

```python
from dataclasses import dataclass


@dataclass
class Passage:
    title: str
    text: str


def _passage2str(passage: Passage) -> str:
    # TODO: format passage.title and passage.text into a
    # single string in a way that can be then be turned
    # back into a Passage.
    pass


def _str2passage(string_: str) -> Passage:
    # TODO: turn a string formatted with `_passage2str`
    # back into a Passage.
    pass
```

<br>

With this, we can now slightly modify the `search` endpoint in order to return
more nicely formatted results, e.g.:

```python
@app.route("/search", methods=['GET'])
def search_view():
    query = request.args.get("q")
    top_passages = None
    if query:
        top_passages: List[str] = _search(bm25, query, corpus)
    if top_passages:
        response = StringIO()
        response.write("<p>Here is what I found:</p><br>")
        for p in top_passages:
            passage: Passage = _str2passage(p)
            response.write(f"<h3>{passage.title}</h3>")
            response.write(f"<p>{passage.text}</p><br>")
        return response.getvalue()
    return "<p>Nothing found.</p>"
```

```console
Note:

All the code snippets are provided as a reference. You can modify the
parameters and return types of the provided functions, or include others
of your own.

As long as the requested functionality is provided, you're free to come up
with your own judgments! :)
```

<br>

## Sending the Challenge 📤

After finishing the challenge, please provide repo access to:

- [@alikareemraja](https://github.com/alikareemraja)
- [@dmlls](https://github.com/dmlls)

Also, send an email to [ali@knowron.com](mailto:ali@knowron.com), with:

- Title: **[NLP Engineer] Name & Surname**.
- Repository link with your solution.
- Information about you: Github, LinkedIn and everything you consider important.

<br>

## Questions 🙋🏻

We are pleased to receive your questions or suggestions at
[ali@knowron.com](mailto:ali@knowron.com). You can also open an issue, but
before that, make sure that your question hasn't been answered in a previous
[issue](https://github.com/knowron/nlp-challenge-v2/issues).
