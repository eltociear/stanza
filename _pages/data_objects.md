---
layout: page
title: Data Objects and Annotations
keywords: data objects
permalink: '/data_objects.html'
nav_order: 2
parent: Neural Pipeline
---

This page describes the data objects and annotations used in Stanza, and how they interact with each other.

## Document

A [`Document`](data_objects.md#document) object holds the annotation of an entire document, and is automatically generated when a string is annotated by the [`Pipeline`](pipeline.md#pipeline). It contains a collection of [`Sentence`](data_objects.md#sentence)s and entities (which are represented as [`Span`](data_objects.md#span)s), and can be seamlessly translated into a native Python object.

[`Document`](data_objects.md#document) contains the following properties:

| Property | Type | Description |
| --- | --- | --- |
| text | `str` | The raw text of the document. |
| sentences | `List[Sentence]` | The list of sentences in this document. |
| entities (ents) | `List[Span]` | The list of entities in this document. |
| num_tokens | `int` | The total number of tokens in this document. |
| num_words | `int` | The total number of words in this document. |

[`Document`](data_objects.md#document) also contains the following method(s):

| Method | Return Type | Description |
| --- | --- | --- |
| to_dict | `List[List[Dict]]` | Dumps the whole document into a list of list of dictionaries, each dictionary representing a token, which are grouped by sentences in the document. |

## Sentence

A [`Sentence`](data_objects.md#sentence) object represents a sentence (as is segmented by the [TokenizeProcessor](/tokenize.html) or provided by the user), and contains a list of [`Token`](data_objects.md#token)s in the sentence, a list of all its [`Word`](data_objects.md#word)s, as well as a list of entities in the sentence (represented as [`Span`](data_objects.md#span)s).

[`Sentence`](data_objects.md#sentence) contains the following properties:

| Property | Type | Description |
| --- | --- | --- |
| doc | `Document` | A "back pointer" to the parent doc of this sentence. |
| text | `str` | The raw text for this sentence. |
| dependencies | `List[(Word, str, Word)]` | The list of dependencies for this sentence, where each item contains the head `Word` of the dependency relation, the type of dependency relation, and the dependent `Word` in that relation. |
| tokens | `List[Token]` | The list of tokens in this sentence. |
| words | `List[Word]` | The list of words in this sentence. |
| entities (ents) | `List[Span]` | The list of entities in this sentence. |

[`Sentence`](data_objects.md#sentence) also contains the following methods:

| Method | Return Type | Description |
| --- | --- | --- |
| to_dict | `List[Dict]` | Dumps the sentence into a list of dictionaries, where each dictionary represents a token in the sentence. |
| print_dependencies | `None` | Print the syntactic dependencies for this sentence. |
| print_tokens | `None` | Print the tokens for this sentence. |
| print_words | `None` | Print the words for this sentence. |

## Token

A [`Token`](data_objects.md#token) object holds a token, and a list of its underlying syntactic [`Word`](data_objects.md#word)s. In the event that the token is a [multi-word token](https://universaldependencies.org/u/overview/tokenization.html) (e.g., French _au = à le_), the token will have a range `id` as described in the [CoNLL-U format specifications](https://universaldependencies.org/format.html#words-tokens-and-empty-nodes) (e.g., `3-4`), with its `words` property containing the underlying [`Word`](data_objects.md#word)s corresponding to those `id`s. In other cases, the [`Token`](data_objects.md#token) object will function as a simple wrapper around one [`Word`](data_objects.md#word) object, where its `words` property is a singleton.

[`Token`](data_objects.md#token) contains the following properties:

| Property | Type | Description |
| --- | --- | --- |
| id | `Tuple[int]` | The index of this token in the sentence, 1-based. This index contains two elements (e.g., `(1, 2)`) when the corresponding token is a multi-word token, otherwise it contains just a single element (e.g., `(1, )`). |
| text | `str` | The text of this token. Example: 'The'. |
| misc | `str` | Miscellaneous annotations with regard to this token. Used in the pipeline to store whether a token is a multi-word token, for instance. |
| words | `List[Word]` | The list of syntactic words underlying this token. |
| start_char | `int` | The start character index for this token in the raw text of the document. Particularly useful if you want to detokenize at one point, or apply annotations back to the raw text. |
| end_char | `int` | The end character index for this token in the raw text of the document. Particularly useful if you want to detokenize at one point, or apply annotations back to the raw text. |
| ner | `str` | The NER tag of this token, in [BIOES format](https://en.wikipedia.org/wiki/Inside%E2%80%93outside%E2%80%93beginning_(tagging)). Example: 'B-ORG'. |

[`Token`](data_objects.md#token) also contains the following methods:

| Method | Return Type | Description |
| --- | --- | --- |
| to_dict | `List[Dict]` | Dumps the token into a list of dictionares, each dictionary representing one of the words underlying this token. |
| pretty_print | `str` | Print this token with the words it expands into in one line. |

## Word

A [`Word`](data_objects.md#word) object holds a syntactic word and all of its word-level annotations. In the event of multi-word tokens (MWT), words are generated as a result of applying the [MWTProcessor](mwt.md), and are used in all downstream syntactic analyses such as tagging, lemmatization, and parsing. If a [`Word`](data_objects.md#word) is the result from an MWT expansion, its `text` will usually not be found in the input raw text. Aside from multi-word tokens, [`Word`](data_objects.md#word)s should be similar to the familiar "tokens" one would see elsewhere.

[`Word`](data_objects.md#word) contains these properties:

| Property | Type | Description |
| --- | --- | --- |
| id | `int` | The index of this word in the sentence, 1-based (index 0 is reserved for an artificial symbol that represents the root of the syntactic tree). |
| text | `str` | The text of this word. Example: 'The'. |
| lemma | `str` | The lemma of this word. |
| upos (pos) | `str` | The universal part-of-speech of this word. Example: 'NOUN'. |
| xpos | `str` | The treebank-specific part-of-speech of this word. Example: 'NNP'. |
| feats | `str` | The morphological features of this word. Example: 'Gender=Fem&#124;Person=3'. |
| head | `int` | The id of the syntactic head of this word in the sentence, 1-based for actual words in the sentence (0 is reserved for an artificial symbol that represents the root of the syntactic tree). |
| deprel | `str` | The dependency relation between this word and its syntactic head. Example: 'nmod'. |
| deps | `str` | The combination of head and deprel that captures all syntactic dependency information. Seen in CoNLL-U files released from Universal Dependencies, not predicted by our [`Pipeline`](pipeline.md#pipeline).
| misc | `str` | Miscellaneous annotations with regard to this word. The pipeline uses this field to store character offset information internally, for instance. |
| parent | `Token` | A "back pointer" to the parent token that this word is a part of. In the case of a multi-word token, a token can be the parent of multiple words. |

[`Word`](data_objects.md#word) also contains the following methods:

| Method | Return Type | Description |
| --- | --- | --- |
| to_dict | `Dict` | Dumps the word into a dictionary with all its information. |
| pretty_print | `str` | Prints the word in one line with all its information. |

## Span

A [`Span`](data_objects.md#span) object stores attributes of a contiguous span of text. A range of objects (e.g., named entities) can be represented as a [`Span`](data_objects.md#span).

[`Span`](data_objects.md#span) contains the following properties:

| Property | Type | Description |
| --- | --- | --- |
| doc | `Document` | A "back pointer" to the parent document of this span. |
| text | `str` | The text of this span. |
| tokens | `List[Token]` | The list of tokens that correspond to this span. |
| words | `List[Word]` | The list of words that correspond to this span. |
| type | `str` | The entity type of this span. Example: 'PERSON'. |
| start_char | `int` | The start character offset of this span in the document. |
| end_char | `int` | The end character offset of this span in the document. |

[`Span`](data_objects.md#span) also contains the following methods:

| Method | Return Type | Description |
| --- | --- | --- |
| to_dict | `Dict` | Dumps the span into a dictionary containing all its information. |
| pretty_print | `str` | Prints the span in one line with all its information. |

## Adding new properties to Stanza data objects

New in v1.1
{: .label .label-green }

All Stanza data objects can be extended easily should you need to attach new annotations of interest to them, either through a new [`Processor`](pipeline.md#processor) you are developing, or from some custom code you're writing.

To add a new annotation or property to a Stanza object, say a `Document`, simply call

```python
Document.add_property('char_count', default=0, getter=lambda self: len(self.text), setter=None)
```

And then you should be able to access the `char_count` property from all instances of the `Document` class. The interface here should be familiar if you have used class properties in Python or other object-oriented language -- the first and only mandatory argument is the name of the property you wish to create, followed by `default` for the default value of this property, `getter` for reading the value of the property, and `setter` for setting the value of the property.

By default, all created properties are read-only, unless you explicitly assign a `setter`. The underlying variable for the new property is named `_{property_name}`, so in our example above, Stanza will automatically create a class variable named `_char_count` to store the value of this property should it be necessary. This is the variable your `getter` and `setter` functions should use, if needed.