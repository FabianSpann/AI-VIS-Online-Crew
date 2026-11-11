# Phase I – AI interaction record (AI-aided pair)

**Pair:** Muhammad Ejaz Habib Dar (k12443138) and Muhammad Abdullah (k12443710)
**Assistant:** Claude (Anthropic), used through claude.ai with code execution and web search.
**Period:** 10–11 Nov 2026

This file records the interactions that shaped our Phase I analysis. For each one it lists the prompt, what the AI produced, what we did with it, and why.

---

## Interaction 1: Turning the task into a concrete Phase I plan
**Prompt (summary):** "We are the AI-aided pair for Phase I. Our task is LLaVA hallucinating objects that aren't in the image. Our GitHub repo is empty and the deadline is tomorrow. Tell me what you need and let's do this." We also shared the Moodle task registration, the course's organisational requirements, and the repository.

**AI output:**
- Noticed that our task registration names no **dataset**.
- Proposed **POPE** (yes/no object probing on COCO) as the main dataset, because it gives ground truth for every answer and is large enough (9,000 questions) to show *systematic* patterns rather than cherry-picked examples.
- Proposed **COCO annotations** as context, used to test hypotheses about co-occurrence, object frequency, clutter and object size.

**Our decision:** Accepted. POPE matches the task and the course requirement that patterns be systematic. The three splits (random / popular / adversarial) contain a built-in hypothesis, co-occurrence, that we can test and later take into Phase II.

**Verification:** The AI could not open Hugging Face directly (blocked in its sandbox). It checked the dataset's columns, size (9,000 rows, 500 images) and splits through web search results instead of guessing.

---

## Interaction 2: Generating the analysis notebook
**Prompt (summary):** "Build the Phase I notebook for Google Colab (free T4 GPU)."

**AI output:** A complete notebook containing:
- 4-bit LLaVA-1.5-7B
- POPE inference, reading P(yes) from the next-token logits instead of parsing free text
- checkpointing to Google Drive
- COCO co-occurrence statistics
- 11 visualizations, including an interactive CLIP/UMAP image map with clustering and drill-down, and a CHAIR-style free-form caption experiment

**Our decisions:**
- **Accepted:** reading P(yes)/P(no) from logits. It is faster, deterministic, and gives a *confidence* for every answer, which makes confidence a behaviour we can study.
- **Accepted:** CLIP (not LLaVA's own encoder) for the image map. LLaVA's internals belong to Phase II, so Phase I stays black-box.
- **Accepted:** the caption experiment as an extension, since yes/no probing alone is a narrow view of hallucination. This uses the AI pair's expected extra breadth.
- **Modified:** nothing in the analysis design. Two bugs were fixed during the run (see Interaction 3).

**Failure patterns observed:**
- The first generated draft had code-quality problems that the AI found only when it re-read its own code: a useless always-true condition (`or True`) in the caption matcher, a convoluted merge that left junk columns (`key_0`), and a hard-to-read lambda.
- One comment pointed to the wrong section number.
- **Lesson:** generated code looks finished before it is. It needs a review pass.

**Verification:**
- The AI ran all analysis and plotting cells on synthetic data. All 9 analysis steps and 11 figures ran without errors.
- The LLaVA inference part **could not be tested by the AI** (no GPU in its sandbox). We test it ourselves with a quick run (`N_IMAGES = 10`) before the full run.

---

## Interaction 3: Debugging the real run
**Prompt (summary):** we pasted two Colab errors: an `AssertionError` in question parsing, and an `AttributeError: 'BaseModelOutputWithPooling' object has no attribute 'norm'` in the CLIP step.

**AI output:**
- Diagnosed POPE wording variants and made the parser tolerant, reporting unparsed questions instead of crashing.
- Diagnosed an API change in recent `transformers` (`get_image_features` now returns an object instead of a tensor) and switched to calling the vision tower and projection directly.

**Our decision:** accepted both fixes, then ran a 10-image test before the full 500-image run, as the AI recommended.

**Failure pattern:** the AI's code passed its own tests on synthetic data but failed on the real data and library versions. **Best practice:** always run a small real test first.

## Interaction 3b: Interpreting the results
**Prompt (summary):** we gave the AI `summary.json`, the result CSVs and the notebook after the full run.

**AI output:**
- Checked the headline claims with extra analyses before writing anything:
  - hallucination rate per co-occurrence bin *within each split* (showing that the adversarial gap is explained by co-occurrence),
  - AUROC of P(yes) for separating hallucinations from correct *yes* answers (0.85),
  - reading sample captions to check the hallucination matcher.
- Re-rendered the trigger map with rows and columns reordered by similarity, as suggested in the lab slides, which revealed kitchen and street blocks.

**Accepted:** the interpretations backed by these checks.

**Rejected / corrected:**
- the clutter hypothesis (no effect in the data),
- the "confident hallucination" framing (hallucinations turned out to be hesitant).

**Kept with a caveat:** the caption hallucination rates (approximate word matching).

## Interaction 4: Video script and reflection
**Prompt (summary):** "Give me the report and what to say in the video." Later: "Should we present slides or the notebook?"

**AI output:**
- A word-for-word script with timings for our part of the video, and an explanation of every chart.
- A 9-slide deck, after the AI argued that slides fit a strict 5-minute limit better than scrolling through a notebook. It also suggested a short live demo of the interactive image map.
- A draft of this reflection.

**What it changed in the analysis:** while selecting example images for a slide, the AI looked at the worst-cluster images and noticed that several flagged "hallucinations" are COCO label gaps (a visible but unlabelled car, a picnic table counted as a dining table). We added this to our findings, so presentation work led to a new insight.

**Our decisions:**
- **Accepted:** slides for the video.
- **Modified:** we removed the notebook's export cell before submission (our choice).
- **Added ourselves:** Pair 2's results, for the comparison section.

**Failure pattern:** the first script draft was written before the label-gap insight and had to be updated. AI-written narratives need re-checking whenever the analysis changes.
