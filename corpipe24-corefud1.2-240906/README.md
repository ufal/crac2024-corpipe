# The `corpipe24-corefud1.2-240906` Model

The `corpipe24-corefud1.2-240906` is a `mT5-large`-based multilingual model for
coreference resolution usable in CorPipe 24 <https://github.com/ufal/crac2024-corpipe>.
It is released at http://hdl.handle.net/11234/1-5672 under the CC BY-NC-SA 4.0
license.

The model is language agnostic (no _corpus id_ on input), so it can be in theory
used to predict coreference in any `mT5` language.

This model jointly predicts also the empty nodes needed for zero coreference.
The paper introducing this model also presents an alternative two-stage approach
first predicting empty nodes (via
https://www.kaggle.com/models/ufal-mff/crac2024_zero_nodes_baseline/) and then
performing coreference resolution (via https://hdl.handle.net/11234/1-5673),
which is circa twice as slow but slightly better.

The model was trained using the following command (see the CorPipe 24 repository
for more information):
```sh
tb="ca_ancora cs_pcedt cs_pdt cu_proiel de_parcorfull de_potsdamcc en_gum en_litbank en_parcorfull es_ancora fr_democrat grc_proiel hbo_ptnk hu_korkor hu_szegedkoref lt_lcc no_bokmaalnarc no_nynorsknarc pl_pcc ru_rucor tr_itcc"
ratios_sqrt="7.3 12.3 10.3 2.8 1.2 2.1 5.2 5.2 1.2 7.7 6.1 3.0 1.1 1.8 4.0 2.2 5.7 5.3 8.4 4.5 2.7"

corpipe24.py --train --dev --treebanks $(for c in $tb; do echo data/$c/$c-corefud-train.conllu; done) --resample 10000 $ratios_sqrt --epochs=15 --batch_size=8 --adafactor --learning_rate=6e-4 --learning_rate_decay --encoder=google/mt5-large --segment=512 --right=50 --label_smoothing=0.2 --exp=corpipe24-corefud1.2
```

## CorefUD 1.2 Test Sets Results

The model achieves the following CorefUD 1.2 test set results (as reported in
the paper); segment size 2560 was used, with the exception for `cu_proiel` and
`grc_proiel` where it was 512:

| avg   | ca   | cs_pce | cs_pdt | cu   | de_par | de_pot | en_gum | en_lit | en_par | es   | fr   | grc  | hbo  | hu_kor | hu_sze | lt   | no_bok | no_nyn | pl   | ru   | tr   |
|-------|------|--------|--------|------|--------|--------|--------|--------|--------|------|------|------|------|--------|--------|------|--------|--------|------|------|------|
| 70.18 | 80.4 | 72.8   | 74.8   | 57.1 | 61.6   | 67.0   | 74.4   | 78.1   | 58.6   | 79.8 | 67.9 | 66.0 | 67.2 | 60.1   | 67.3   | 75.2 | 78.9   | 76.6   | 75.2 | 81.2 | 53.4 |


## Running the Model on Plain Text

To run the model on plain text, first the plain text needs to be tokenized and
converted to CoNLL-U (and optionally parsed if you also want mention heads),
by using for example UDPipe 2:

```sh
curl -F data="Eve came home and Peter greeted her there. Then Peter and Paul set out to a trip and Eve waved them off." \
  -F model=english -F tokenizer= -F tagger= -F parser=  https://lindat.mff.cuni.cz/services/udpipe/api/process \
  | python -X utf8 -c "import sys,json; sys.stdout.write(json.load(sys.stdin)['result'])" >input.conllu
```

Then the CoNLL-U file can be processed by CorPipe 24, by using for example
```sh
python3 corpipe24.py --load corpipe24-corefud1.2-240906/model.h5 --exp . --epoch 0 --test input.conllu
```
which would generate the following predictions in `input.00.conllu`:
```
# generator = UDPipe 2, https://lindat.mff.cuni.cz/services/udpipe
# udpipe_model = english-ewt-ud-2.12-230717
# udpipe_model_licence = CC BY-NC-SA
# newdoc
# global.Entity = eid-etype-head-other
# newpar
# sent_id = 1
# text = Eve came home and Peter greeted her there.
1	Eve	Eve	PROPN	NNP	Number=Sing	2	nsubj	_	Entity=(c1--1)
2	came	come	VERB	VBD	Mood=Ind|Number=Sing|Person=3|Tense=Past|VerbForm=Fin	0	root	_	_
3	home	home	ADV	RB	_	2	advmod	_	Entity=(c2--1)
4	and	and	CCONJ	CC	_	6	cc	_	_
5	Peter	Peter	PROPN	NNP	Number=Sing	6	nsubj	_	Entity=(c3--1)
6	greeted	greet	VERB	VBD	Mood=Ind|Number=Sing|Person=3|Tense=Past|VerbForm=Fin	2	conj	_	_
7	her	she	PRON	PRP	Case=Acc|Gender=Fem|Number=Sing|Person=3|PronType=Prs	6	obj	_	Entity=(c1--1)
8	there	there	ADV	RB	PronType=Dem	6	advmod	_	Entity=(c2--1)|SpaceAfter=No
9	.	.	PUNCT	.	_	2	punct	_	_

# sent_id = 2
# text = Then Peter and Paul set out to a trip and Eve waved them off.
1	Then	then	ADV	RB	PronType=Dem	5	advmod	_	_
2	Peter	Peter	PROPN	NNP	Number=Sing	5	nsubj	_	Entity=(c4--1(c3--1)
3	and	and	CCONJ	CC	_	4	cc	_	_
4	Paul	Paul	PROPN	NNP	Number=Sing	2	conj	_	Entity=(c5--1)c4)
5	set	set	VERB	VBD	Mood=Ind|Number=Sing|Person=3|Tense=Past|VerbForm=Fin	0	root	_	_
6	out	out	ADP	RP	_	5	compound:prt	_	_
7	to	to	ADP	IN	_	9	case	_	_
8	a	a	DET	DT	Definite=Ind|PronType=Art	9	det	_	Entity=(c6--2
9	trip	trip	NOUN	NN	Number=Sing	5	obl	_	Entity=c6)
10	and	and	CCONJ	CC	_	12	cc	_	_
11	Eve	Eve	PROPN	NNP	Number=Sing	12	nsubj	_	Entity=(c1--1)
12	waved	wave	VERB	VBD	Mood=Ind|Number=Sing|Person=3|Tense=Past|VerbForm=Fin	5	conj	_	_
13	them	they	PRON	PRP	Case=Acc|Number=Plur|Person=3|PronType=Prs	12	obj	_	Entity=(c4--1)
14	off	off	ADP	RP	_	12	compound:prt	_	SpaceAfter=No
15	.	.	PUNCT	.	_	5	punct	_	SpaceAfter=No

```

## How to Cite

During the model release, only the preprint was available; please cite the
CRAC 2024 paper once published.

```
@misc{straka-2024-corpipe,
  title={{CorPipe at CRAC 2024: Predicting Zero Mentions from Raw Text}},
  author={Milan Straka},
  year={2024},
  eprint={2410.02756},
  archivePrefix={arXiv},
  primaryClass={cs.CL},
  url={https://arxiv.org/abs/2410.02756},
}
```
