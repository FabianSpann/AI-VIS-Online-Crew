# Midpoint reflection: Phase I with GenAI

**AI-aided pair (Phase I):**
- Muhammad Ejaz Habib **Dar** (k12443138)
- Muhammad **Abdullah** (k12443710)

**Human-only pair (Phase I):** Fabian Spann (k12122592), Ronald Pohl (k12105567)

---

## 1. How we used GenAI
We used Claude as an active analysis partner:
- choosing the dataset (POPE + COCO),
- proposing hypotheses (co-occurrence, object frequency, clutter, object size, overconfidence),
- writing the full Colab notebook,
- interpreting the figures.

The important prompts and our decisions are documented in `ai-interactions/phase1_ai_pair_log.md`.

## 2. What GenAI changed in our process
- **Speed and breadth:** a working pipeline with 11 visualizations, including an interactive image map and a caption experiment, was ready within one evening. Without AI we would probably have covered only the split-level metrics and one or two plots.
- **Hypotheses came from the AI's prior knowledge of the literature.** For example, it suggested that co-occurrence statistics drive hallucination. This sped us up, but it also meant we were testing ideas we did not come up with ourselves. We had to make sure we actually understood them.
- **Design choices we would not have made ourselves:** reading the yes/no probability from the logits (which makes confidence measurable), and using an independent CLIP encoder for the image map.

## 3. Failure patterns
- **Code that looks finished but is not:** the first draft contained dead code, a messy merge, and a wrong cross-reference. These were caught only by a review pass.
- **Untestable parts:** the AI could not run LLaVA in its own environment. It tested the analysis on synthetic data, but the model-loading code was unverified until we ran it.
- **Tool limits:** the AI could not access Hugging Face directly and had to verify the dataset format through web search.
- **First code ran but broke in real use:** the question parser crashed on wording variants in POPE, and the CLIP embedding call failed with the newest `transformers` version. Each was fixed after we pasted the error back. The AI's synthetic tests could not catch these.
- **A suggested hypothesis did not hold:** the AI expected cluttered scenes to drive hallucination. The data showed almost no clutter effect. The difference comes from the scene *type*, not the number of objects.
- **Over-eager framing:** chart titles drafted before the results ("hallucinations are often made with high confidence?") assumed an outcome. The data showed the opposite: hallucinations are hesitant. We kept the conclusion the data supports.
- **Label gaps found only by looking at images:** some flagged hallucinations are objects COCO did not annotate (a distant car) or near-synonyms (picnic table vs. dining table, bench vs. chair). Aggregate charts hid this; we only saw it by drilling into the example images.
- **Measurement limits the AI stated, which we checked:** the caption hallucination matcher uses word matching (e.g. "table" counts as "dining table"). We read sample captions to confirm most flagged objects really are absent.

## 4. Emerging best practices
1. Ask the AI to **test its code** (even on synthetic data) and to say clearly what it could *not* test.
2. **Run a small version first** (10 images) before trusting a full pipeline.
3. Keep **hypotheses separate from findings**: the AI proposes a hypothesis, and the data decides.
4. Check every number the AI quotes in an interpretation against `summary.json`.
5. Log decisions as we go, not afterwards.

## 5. Comparison with the human-only pair
Both pairs agreed on the same direction: LLaVA-1.5-7B in 4-bit, POPE plus COCO annotations, and the question of what makes an absent object "plausible" to the model. After that, each pair worked separately (Pair 2: `pair2/Phase1_Pair2_Without.ipynb`).

- **Same core finding, reached independently.** Both pairs found that **co-occurrence, not object frequency or scene clutter**, explains hallucination.
  - Pair 2's logistic regression: co-occurrence lift +1.24 (p < 0.001); frequency −0.04 (n.s.); clutter n.s.
  - Ours: co-occurrence +0.86, the largest coefficient.
  - Both pairs measured similar false-yes rates on the three splits (Pair 2: 2.2 / 3.0 / 9.4%; ours: 2.3 / 5.7 / 10.2%), and the same median P(yes) of about 0.69 for hallucinations.
  - Arriving at this without AI makes us more confident it is real and not an artefact of AI-written code.
- **Insights the human-only pair had that we (and the AI) overlooked:**
  - **Pre-processing.** The Hugging Face processor centre-crops images, while the original LLaVA pads them to a square. Cropping can cut off objects at the border, which would show up as "misses". Pair 2 replicated the original padding; we did not. Our miss rate (22.2%) is close to theirs (22.7%), so the effect seems small, but it is a real methodological gap in our AI-generated pipeline.
  - **Statistical rigour.** Pair 2 reported Wilson confidence intervals, chi² and binomial tests, and p-values for every claim. We mostly reported rates and one CI plot.
- **Insights we had that they did not:**
  - the reordered trigger map (kitchen and street blocks),
  - the CLIP/UMAP image map with cluster drill-down, which revealed **COCO label gaps** (bench vs. chair, a visible but unlabelled car),
  - the free-caption experiment (hallucinations grow from 13% to 52% across a caption).
- **Breadth vs. depth:**
  - With AI we covered more ground: 9,000 questions vs. 3,000, 12 interactive figures, and two extra experiments.
  - Pair 2 went deeper on each claim: careful hypotheses (H1–H5), interpretation written in their own words, and significance tests.
- **Visualization choices:** Pair 2 used static matplotlib/seaborn figures (violin plots, an object × split heatmap, and the model's hallucination map next to the COCO co-occurrence map). We used interactive Plotly figures with hover details.
- **Speed vs. verification:** AI got us a full pipeline in one evening. The time we saved partly went into debugging AI code on real data and checking AI interpretations, and one AI hypothesis (clutter) and one framing ("confident hallucinations") turned out wrong.
- **Confidence in conclusions:** where both pairs agree (co-occurrence, hesitant hallucinations), we are confident. Our caption numbers rest on approximate word matching, and our miss rates may be slightly affected by cropping, so we hold those more loosely.

## 6. Key Phase I findings (AI pair)
Full details are in `pair1/phase1_llava_pope.ipynb` (Findings cell).
1. Hallucination depends on the queried object: 2.3% (random) → 5.7% (popular) → 10.2% (adversarial).
2. **Co-occurrence is the driver:** 1.3% hallucination for rarely co-occurring objects vs. about 15% for strongly co-occurring ones. At equal co-occurrence, the splits behave alike.
3. Two trigger blocks, **kitchen** and **street**: LLaVA "completes" typical scenes.
4. Hallucinations are **hesitant** (median P(yes) 0.69 vs 0.93 for correct *yes*; AUROC 0.85).
5. The more common error is **missing small objects** (36% missed below 0.5% of the image area vs 4% above 15%).
6. In free captions, 67% of captions contain a hallucinated object, and the hallucinated share grows from 13% early in the caption to 52% at the end.
