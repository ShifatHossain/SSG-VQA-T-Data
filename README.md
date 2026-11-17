# SSG-VQA-T: A Temporal QA Dataset for CholecT45

**SSG-VQA-T** is a temporal visual question answering (VQA) dataset built on the
CholecT45 laparoscopic cholecystectomy videos. Every question is tied to a video
and can only be answered by reasoning over **multiple frames of the same surgical
video** — what happened before/after an event, what was happening at the same time,
how long a relation lasted, which object appeared or disappeared, and so on.

This repository contains only the temporal QA annotations, their class-indexed
counterparts and answer vocabularies, the notebooks that generated them, and this
README. **No video frames and no other dataset are shipped here** — see
[Downloading CholecT45](#downloading-cholect45).

---

## Repository layout

```text
SSG-VQA-T/
├── README.md
├── Temporal.ipynb                        # builds the in-template temporal QAs
├── Untemplate.ipynb                      # rephrases them into out-of-template temporal QAs
├── ssgqa_qa_temporal/                    # In-template temporal, raw answers     (VIDxx.txt)
├── ssgqa_qa_temporal_cls/                # In-template temporal, class-indexed   (VIDxx/temporal.txt)
├── ssgqa_qa_temporal_oot/                # Out-of-template temporal, raw answers (VIDxx.txt)
├── ssgqa_qa_temporal_oot_cls/            # Out-of-template temporal, class-indexed (VIDxx/temporal.txt)
└── vocab/
    ├── vocab_in_template_temporal.json   # 29 answer classes (in-template)
    └── vocab_out_of_template_temporal.json
```

| Set | Videos | Questions |
| :-- | :---: | ---: |
| `ssgqa_qa_temporal` / `ssgqa_qa_temporal_cls` | 50 | 22,374 |
| `ssgqa_qa_temporal_oot` / `ssgqa_qa_temporal_oot_cls` | 50 | 22,374 |

The two sets contain the same questions with the same answers — the out-of-template
set is a paraphrased, free-form rewording of the in-template set.

---

## Question types

Each question carries a `QType` field:

| `QType` | Meaning |
| :---: | :--- |
| `0` | Temporal state / action identification |
| `1` | Temporal event order / sequencing (before / after / immediately before) |
| `2` | Concurrent / simultaneous action queries |

---

## File formats

### Raw text (`ssgqa_qa_temporal/`, `ssgqa_qa_temporal_oot/`)

One file per video, `VIDxx.txt`, one question per line:

```text
Question|Answer|QType|Action_Start|Action_End|Related_Action_Start|Related_Action_End
```

Example (`ssgqa_qa_temporal/VID01.txt`):

```text
What action did the bipolar perform immediately before the grasper began retracting the gallbladder?|coagulate|1|1010|1025|1037|1038
```

The four trailing integers are the frame indices (0-based, as in CholecT45) of the
action and of the related action that the question refers to.

### Class-indexed (`ssgqa_qa_temporal_cls/`, `ssgqa_qa_temporal_oot_cls/`)

Same layout, one file per video at `VIDxx/temporal.txt`, with the answer replaced
by its index in the corresponding vocabulary:

```text
Question|Answer_Index|QType|Action_Start|Action_End|Related_Action_Start|Related_Action_End
```

Example (`ssgqa_qa_temporal_cls/VID01/temporal.txt`):

```text
What action did the bipolar perform immediately before the grasper began retracting the gallbladder?|7|1|1010|1025|1037|1038
```

### Answer vocabularies (`vocab/`)

JSON arrays of answer strings; the class index in a `*_cls` file is the position of
the answer in the array (answers are sorted alphabetically, multi-token answers are
indexed by their first token). Each temporal vocabulary has 29 classes.

---

## Downloading CholecT45

The CholecT45 videos are **not** included in this repository. CholecT45 is an
officially released CAMMA dataset; request access and download it from its GitHub
release page — the same route used for the SSG-VQA dataset:

**<https://github.com/CAMMA-public/cholect45>**

Expected layout once downloaded (the QA files reference these frame indices):

```text
CholecT45/
├── data/
│   ├── VID01/000000.png
│   ├── VID01/000001.png
│   ├── ...
│   └── VID45/...
├── triplet/  instrument/  verb/  target/  dict/
├── LICENSE
└── README.txt
```

Notes:

- CholecT45 contains 45 videos. This QA set covers 50 video IDs; the 45 CholecT45
  videos are all present, and `VID92`, `VID96`, `VID103`, `VID110`, `VID111` are
  outside the CholecT45 release (no frames are provided for them here either).
- CholecT45 is released under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/);
  please cite the dataset when using these questions:

```bibtex
@article{nwoye2022rendezvous,
  title={Rendezvous: Attention Mechanisms for the Recognition of Surgical Action Triplets in Endoscopic Videos},
  author={Nwoye, C. I. and Yu, T. and Gonzalez, C. and Seeliger, B. and Mascagni, P. and Mutter, D. and Marescaux, J. and Padoy, N.},
  journal={Medical Image Analysis},
  year={2022},
  doi={10.1016/j.media.2022.102433}
}
```

---

## Regenerating the questions (optional)

The annotations are provided as-is; regeneration only needs the two notebooks.

1. **`Temporal.ipynb`** — groups per-frame scene-graph JSON files into video
   timelines and emits the in-template temporal questions into `ssgqa_qa_temporal/`.
   The input directory is set inside the notebook (`SSGVQA/scene_graph/`); those
   scene graphs are distributed with the SSG-VQA release
   (<https://github.com/CAMMA-public/SSG-VQA>).
2. **`Untemplate.ipynb`** — rephrases the questions of `ssgqa_qa_temporal/` with an
   LLM into free-form wording and writes them to `ssgqa_qa_temporal_oot/`
   (cell headed `# SSG-VQA Temporal`). The same notebook also holds a cell that
   rephrases the non-temporal questions, which are not part of this repository.
3. **Class indexing** — to rebuild a `*_cls` file, replace the answer field of each
   raw line with the index of its first token in the matching `vocab/*.json` array.
