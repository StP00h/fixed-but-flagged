# Fixed but Flagged, Wrong but Approved

*Testing Qwen3-0.6B’s feedback on Chinese short answers*

What happens when a small model grades short student answers — and the answer changes under it? This repository contains a complete, checkable case study: 129 real responses from a locally-run Qwen3-0.6B on fictional Chinese short-answer tasks, the full case logs, and the labels behind every claim. **The write-up is on this page; every link in it leads to the exact evidence.**

## What's in this repository

| Path | Contents |
|---|---|
| [`case-logs/set-f.md`](case-logs/set-f.md) | Set F case log — task, source material, student answer, and full model response for all 54 generations. |
| [`case-logs/set-g.md`](case-logs/set-g.md) | Set G case log — same format for all 75 generations, including seven control answers. |
| [`labels/set-f-labels.jsonl`](labels/set-f-labels.jsonl) | Set F labels: per-response target judgment from two independent AI coding passes, researcher adjudications (29 of 54 rows) with written rationale, and `label_source` marking which rows are human-confirmed. |
| [`labels/set-g-labels.jsonl`](labels/set-g-labels.jsonl) | Set G labels: same format (16 of 75 rows adjudicated). |
| [`data/prompts.v1.json`](data/prompts.v1.json) | The three prompt texts: ordinary, criteria, evidence-first. |
| [`environment/requirements.lock.txt`](environment/requirements.lock.txt) | Pinned Python dependencies — all 25 packages at exact versions (direct and transitive). |

Every response identifier (`packet_id`) used in the links below appears in both the case-log headers and the label files, so a claim and its evidence can be matched mechanically.

---

# Fixed but Flagged, Wrong but Approved

*Testing Qwen3-0.6B’s feedback on Chinese short answers*

A prompt asks for one benefit and one cost of keeping the library open later. The revised answer explains that students who return late could still borrow books, and that staff would have to work longer hours with no extra budget. Qwen3-0.6B says the answer “does not clearly state the specific benefits or costs.” It then lists the very benefit and cost the answer already gives. [Original F04 response](case-logs/set-f.md#f04_repaired--criteria--81a4d05023f02ac3)

In another task, the source says that the east reading corner received 30 picture books and the west reading corner received 20 popular science books. The answer assigns the 30 picture books to the west corner. Given the same criteria prompt, the model says, “Grounded target issue: the number of new books matches the source material.” [Original G05 response](case-logs/set-g.md#g05_problem--criteria--3ac818883fb43526)

*The examples and excerpts are translated from Chinese. The links lead to the original inputs and responses.*

Following the first comment would mean rewriting information already in the answer. Following the second would leave the mix-up in place. Research on formative feedback connects the task goal, current performance, and the next action [1], [2]. Comparisons of human and ChatGPT feedback on essays also distinguish accuracy, guidance for revision, and whether the feedback addresses what matters most [3]. We wanted to see whether the model would drop a criticism once the issue was fixed and still catch it when the wording changed.

Prior work on feedback grounded in student submissions has documented models generating feedback that misrepresents what the submission contains [4]. In propositional logic tutoring, Yasir et al. observed models rejecting valid but suboptimal steps and accepting incorrect ones [5]. Here, each task gives us one requirement we can check against the answer and the source.

## Three versions of the same answer

The library prompt stays the same across all three versions. It asks for at least one benefit and one cost. Before running the model, we can specify what the feedback should pick up in each version.

| Version | What the answer says | What the feedback should pick up |
|---|---|---|
| Original with an issue | Students who return late could still borrow books, so the library should stay open later. | The answer gives a benefit and omits the cost. |
| Fixed | Later opening would help those students, but it would increase staff hours with no extra budget. | The answer gives both a benefit and a cost. |
| Reworded with the issue intact | Extending opening hours would improve access to library loans for students who return late. | The wording changes and the cost is still missing. |

The setup works like a behavioral test. Decide which edit should change the judgment and which should leave it in place, then run the model. CheckList applies this idea to NLP models [6]. Wang et al. compared feedback on original and modified essays [7], while Dycke and Gurevych compared edits that change the soundness of a paper’s reasoning with surface edits in automated peer review [8]. In our short answers, the same requirement applies to all three versions, so the reader can check each judgment against the task and the answer.

We ran the model locally on two sets of AI-drafted Chinese short-answer tasks and collected 129 responses. All tasks are fictional scenarios written for this project; no real student submissions are involved. Set F contains six task families focused on argumentation, source fidelity, and decision tasks. Set G contains six new families, including unit conversion, step order, and if–then conditions. It also includes seven control answers that meet the target requirement. We submitted each answer on its own under three prompts: ordinary feedback, explicit criteria, and evidence first. Each call received the task, source material, and current answer. We used the same revision of Qwen3-0.6B throughout, running on MPS in non-thinking mode with one sample per input [9], [10]. [Run details](#how-the-runs-were-done) · [Prompt texts](data/prompts.v1.json)

## The two sets show different errors

| Model judgment on the target issue | Set F | Set G |
|---|---:|---:|
| Explicitly identified the issue in the original answer | 9/18 | 5/18 |
| Explicitly flagged the same issue after it was fixed | 4/18 | 0/18 |
| Explicitly confirmed that the issue was absent in the fixed answer | 8/18 | 18/18 |
| Explicitly approved the reworded answer with the issue still present | 4/18 | 13/18 |

Each denominator of 18 represents six task families under three prompts. We kept unclear and contradictory judgments in separate categories. The two runs had an AI-assisted coding pass and a second AI review of each target judgment. A researcher adjudicated 29 of 54 responses in Set F and 16 of 75 in Set G; the remaining labels retain the first-pass coding, so each count in this section mixes human-confirmed rows with AI-coded ones. The imbalance matters most for Set G's fixed answers: only 1 of those 18 rows was human-adjudicated, against 11 of 18 in Set F. Of the 21 control responses in Set G, 19 were coded as explicit confirmations that the target issue was absent and two gave no clear judgment on it. [Set F final labels](labels/set-f-labels.jsonl) · [Set G final labels](labels/set-g-labels.jsonl)

The same model shows opposite failure modes on the two sets. In Set F, it sometimes kept criticizing a problem the answer had already fixed. In Set G, every fixed answer was coded as explicitly confirmed. The other versions help explain why those confirmations tell only part of the story. In nine task-family and prompt combinations in Set G, the labels approve both the original and the reworded answer. In three combinations, it caught the issue in the original and approved the reworded answer. Reading the versions side by side shows which mistakes the model was already missing before the rewrite.

## The evidence can be right while the verdict is wrong

The evidence-first prompt asks the model to show its evidence before giving a judgment. On the reading-corner task, it returned these three lines.

> Evidence: The west reading corner received 20 popular science books.  
> Judgment: Meets the requirement.  
> Suggestion: No changes needed.

The evidence accurately reflects the source. Put it next to the answer, which says the west corner received 30 picture books, and both the quantity and the type of book conflict. The model supplied the fact that should have led it to flag the error. Research on citation evaluation also treats factual correctness and whether the cited evidence supports a claim as distinct checks [11]. [Original G05 evidence-first response](case-logs/set-g.md#g05_problem--evidence_first--690469ace6d94cc2)

Two other answers show why the suggested action needs a check of its own. In a reading-preference task, the answer says most students preferred light music. The model correctly reports that 12 preferred quiet reading and eight preferred light music. It calls the answer a violation and then says no changes are needed. In a library checkout task, the answer puts pickup before reservation. The model's evidence notes that no confirmation text had arrived yet, and it judges the answer a violation—but its suggestion is to add information about the confirmation message, when the needed fix is to reverse the order of the steps. FeedEval treats validity, specificity, and helpfulness as distinct properties of essay feedback [12]. These short answers show what that distinction looks like in a single response. [Original F03 response](case-logs/set-f.md#f03_problem--evidence_first--ac9424402f6b45be) · [Original G02 response](case-logs/set-g.md#g02_problem--evidence_first--fe0c728531e3be31)

## Add these three versions to your feedback tests

If you are building a tool that gives feedback on short answers, pick one task and one requirement. Write an answer with a specific issue, a version that fixes it, and a rewrite that leaves it in place. Run the tool on each version separately, then check the replies.

1. **Did it read the answer correctly?** Check what the model says the answer includes or omits against the text the student has in front of them.
2. **Does the evidence support the verdict?** Compare the answer with the source and decide whether the evidence supports asking for a change or confirming that the requirement is met.
3. **Would the suggestion fix the issue?** Try following it. If the answer already meets the requirement, an explicit confirmation can be the whole response.

For tasks with a checkable unit, sequence, or entity-to-fact match, you can verify that part separately and pass the result to the model that writes the feedback. Daheim et al. studied this separation of error verification and response generation in math tutoring [13]. The same three test inputs can help you check whether a change to your tool makes it flag correct work, miss an error, or recommend a revision that leaves the error in place.

## What this does not show

- **One model, one revision, one language, one sample per input.** Nothing here says anything about small models as a class, other Qwen revisions, or English tasks.
- **The counts are small and not independent.** Three versions of one task share the same source material, so “18” means six tasks times three prompts, not eighteen independent trials.
- **Most labels are AI-coded.** Every target judgment went through two independent AI coding passes, and a researcher adjudicated 29 of 54 responses in Set F and 16 of 75 in Set G; the rest keep the first-pass coding.
- **The two sets changed more than one thing at a time.** Task type and material type moved together, so we can describe the flip in failure modes but not attribute it to a single cause.
- **The tasks are fiction.** Single-requirement short answers with a checkable target are cleaner than real student writing. A model that misses errors on the clean case gives little reason to trust it on messier ones, but the reverse is not guaranteed.

The case logs include the task, source, answer, and full model response. Each response has a corresponding final label, with a rationale for every human adjudication. The next time the model says “meets the requirement” or “add more detail,” put the answer beside the feedback and point to the sentence that makes the judgment stand or fall. [Set F case log](case-logs/set-f.md) · [Set G case log](case-logs/set-g.md)

## References

[1] J. Hattie and H. Timperley, “The power of feedback,” *Rev. Educ. Res.*, vol. 77, no. 1, pp. 81–112, Mar. 2007, doi: [10.3102/003465430298487](https://doi.org/10.3102/003465430298487).

[2] V. J. Shute, “Focus on formative feedback,” *Rev. Educ. Res.*, vol. 78, no. 1, pp. 153–189, Mar. 2008, doi: [10.3102/0034654307313795](https://doi.org/10.3102/0034654307313795).

[3] J. Steiss *et al*., “Comparing the quality of human and ChatGPT feedback of students’ writing,” *Learn. Instr.*, vol. 91, Art. no. 101894, Jun. 2024, doi: [10.1016/j.learninstruc.2024.101894](https://doi.org/10.1016/j.learninstruc.2024.101894).

[4] Q. Jia *et al*., “On assessing the faithfulness of LLM-generated feedback on student assignments,” in *Proc. 17th Int. Conf. Educ. Data Mining*, 2024, pp. 491–499, doi: [10.5281/zenodo.12729868](https://doi.org/10.5281/zenodo.12729868).

[5] T. Yasir, W. Li, S. Gilson, S. Tithi, X. Tian, and T. Barnes, “Confirming correct, missing the rest: LLM tutoring agents struggle where feedback matters most,” in *Proc. 21st Workshop Innov. Use NLP Building Educ. Appl.*, 2026, pp. 819–840, doi: [10.18653/v1/2026.bea-1.56](https://doi.org/10.18653/v1/2026.bea-1.56).

[6] M. T. Ribeiro, T. Wu, C. Guestrin, and S. Singh, “Beyond accuracy: Behavioral testing of NLP models with CheckList,” in *Proc. 58th Annu. Meeting Assoc. Comput. Linguistics*, 2020, pp. 4902–4912, doi: [10.18653/v1/2020.acl-main.442](https://doi.org/10.18653/v1/2020.acl-main.442).

[7] Y. Wang, R. Hu, and Z. Zhao, “Beyond agreement: Diagnosing the rationale alignment of automated essay scoring methods based on linguistically-informed counterfactuals,” in *Findings Assoc. Comput. Linguistics: EMNLP*, 2024, pp. 8906–8925, doi: [10.18653/v1/2024.findings-emnlp.520](https://doi.org/10.18653/v1/2024.findings-emnlp.520).

[8] N. Dycke and I. Gurevych, “Automatic reviewers fail to detect faulty reasoning in research papers: A new counterfactual evaluation framework,” 2025, *arXiv*:2508.21422, accepted at *Trans. Assoc. Comput. Linguist.*, 2026. [Online]. Available: [arxiv.org/abs/2508.21422](https://arxiv.org/abs/2508.21422).

[9] Qwen Team, “Qwen3 technical report,” 2025, *arXiv*:2505.09388. [Online]. Available: [arxiv.org/abs/2505.09388](https://arxiv.org/abs/2505.09388).

[10] Qwen, “Qwen3-0.6B,” Hugging Face model card, revision `c1899de289a04d12100db370d81485cdf75e47ca`. [Online]. Available: [fixed revision model card](https://huggingface.co/Qwen/Qwen3-0.6B/blob/c1899de289a04d12100db370d81485cdf75e47ca/README.md).

[11] T. Gao, H. Yen, J. Yu, and D. Chen, “Enabling large language models to generate text with citations,” in *Proc. Conf. Empirical Methods Natural Lang. Process.*, 2023, pp. 6465–6488, doi: [10.18653/v1/2023.emnlp-main.398](https://doi.org/10.18653/v1/2023.emnlp-main.398).

[12] S. Chu, J. Kim, and M. Y. Yi, “FeedEval: Pedagogically aligned evaluation of LLM-generated essay feedback,” in *Findings Assoc. Comput. Linguistics: ACL*, 2026, pp. 12648–12674, doi: [10.18653/v1/2026.findings-acl.615](https://doi.org/10.18653/v1/2026.findings-acl.615).

[13] N. Daheim, J. Macina, M. Kapur, I. Gurevych, and M. Sachan, “Stepwise verification and remediation of student reasoning errors with large language model tutors,” in *Proc. Conf. Empirical Methods Natural Lang. Process.*, 2024, pp. 8386–8411, doi: [10.18653/v1/2024.emnlp-main.478](https://doi.org/10.18653/v1/2024.emnlp-main.478).

## How the runs were done

- **Model**: Qwen3-0.6B, fixed revision `c1899de289a04d12100db370d81485cdf75e47ca` ([model card for that revision](https://huggingface.co/Qwen/Qwen3-0.6B/blob/c1899de289a04d12100db370d81485cdf75e47ca/README.md)), Apache-2.0, downloaded from Hugging Face, loaded from local weights with no network access at run time.
- **Stack**: Python 3.12.12, torch 2.9.0, transformers 4.57.1 (recreate with `pip install -r environment/requirements.lock.txt`), Apple-silicon MPS, float32, eager attention, MPS-to-CPU fallback explicitly disabled.
- **Decoding**: the model's chat template with thinking disabled; one sample per input; temperature 0.7, top_p 0.8, top_k 20, max_new_tokens 512. All 129 responses ended with an EOS token; none were truncated, none errored.
- **Inputs**: each call saw exactly one version of one answer — task, source material, and the answer itself — under one of the three prompts. No revision history, no evaluation labels, no variant names in any input.
- **Seeds**: derived deterministically from each packet ID, so different inputs got different fixed seeds; one sample per input means sampling noise and prompt effects are not separated.

## How the labels were made

Each response was coded for one question — did the model explicitly identify, explicitly deny, or fail to judge the task's target issue — plus evidence, suggestion, and scope fields. Coding went through two independent AI passes (target-judgment agreement between passes: 53/54 in Set F, 68/75 in Set G), and a researcher adjudicated the disputed and ambiguous rows — 29 of 54 in Set F, 16 of 75 in Set G — each with a written rationale in the label files. Rows marked `label_source: ai_assisted_unconfirmed` keep first-pass coding and are not treated as human-confirmed.

## What is intentionally not in this repository

To keep the repository minimal, the raw generation logs (token-ID streams and machine-specific environment dumps), the task seed files, and the pipeline scripts are omitted. Nothing in the write-up depends on the omissions: the case logs contain every prompt-side fact and every full response verbatim, and the label files contain every judgment with its rationale. The task and answer texts are fictional fixtures drafted with AI assistance for this project; no real student work is involved.

## License

Released under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — see [`LICENSE`](LICENSE) for the full text. You may share and adapt the write-up, case logs, fixtures, and labels with attribution. Model responses are quoted verbatim from Qwen3-0.6B (Apache-2.0); use of the model and its outputs is additionally governed by that model license. If pipeline code is added to this repository later, it should carry its own software license.

### How to Cite
```bibtex
@misc{li2026fixedbutflagged,
  author       = {Xuao Li},
  title        = {{Fixed but Flagged, Wrong but Approved}: Testing {Qwen3-0.6B}'s
                  Feedback on Chinese Short Answers},
  year         = {2026},
  month        = nov,
  url          = {https://github.com/StP00h/fixed-but-flagged}
}
```
