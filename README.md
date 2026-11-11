# AI-VIS-Online-Crew

**Group:** Online Crew — AI and Visualization Lab, JKU Linz, WS 2026/27

Exploring why large vision-language models hallucinate visual details that aren't present in images.

## Task
Investigate why large vision-language models (VLMs) hallucinate objects that are not in an image (for example, describing an object in the background that does not exist).

- **Phase I – How does the model behave?** Treat the model as a black box: probe it with images and questions, and find systematic patterns in *when* and *which* objects it hallucinates.
- **Phase II – What did the model learn?** Open the model (attention, representations, saliency) to explain the behaviours found in Phase I.

## Model
- **LLaVA-1.5-7B** ([project page](https://llava-vl.github.io/), Hugging Face: [`llava-hf/llava-1.5-7b-hf`](https://huggingface.co/llava-hf/llava-1.5-7b-hf)): a CLIP ViT-L/14 image encoder connected to a Vicuna-7B language model.
- Run in 4-bit (NF4) on a free Google Colab T4 GPU.
- Helper model for the image map: CLIP ViT-B/32 (`openai/clip-vit-base-patch32`), used only to embed images, independently of LLaVA.

## Datasets
- **POPE** (Li et al., 2023, [paper](https://arxiv.org/abs/2305.10355), Hugging Face: [`lmms-lab/POPE`](https://huggingface.co/datasets/lmms-lab/POPE)): 500 COCO val2014 images and 9,000 yes/no questions of the form *"Is there a {object} in the image?"*, balanced 50/50 between present and absent objects. It has three splits of 3,000 questions each, which differ in how the absent objects are chosen:
  - *random*: any COCO object
  - *popular*: the most frequent COCO objects
  - *adversarial*: objects that frequently co-occur with the objects in the image
- **COCO 2014 annotations** ([cocodataset.org](https://cocodataset.org)):
  - *val2014*: ground-truth objects, their sizes, and scene clutter for the 500 test images
  - *train2014*: object frequency and co-occurrence statistics

## Repository structure
| Path | Content |
| --- | --- |
| `pair1/` | Pair 1 (Muhammad Ejaz Habib Dar, Muhammad Abdullah). Phase I with GenAI: `phase1_llava_pope.ipynb` (POPE probing, co-occurrence, confidence, image map, captions) |
| `pair1/outputs/` | Results of the full run: `summary.json`, per-question results, interactive (HTML) and static (PNG) figures |
| `pair2/` | Pair 2 (Fabian Spann, Ronald Pohl). Phase I without GenAI: `Phase1_Pair2_Without.ipynb` (3,000-question stratified POPE sample, error types, calibration, per-object analysis, frequency vs. co-occurrence regression) |
| `ai-interactions/` | Records of our interactions with GenAI |
| `ai_reflection_midpoint.md` | Phase I reflection on working with GenAI |
| `team-info.json`, `workload.json` | Course metadata |

## How to run (Phase I, AI pair)
1. Open `pair1/phase1_llava_pope.ipynb` in Google Colab and choose **Runtime → Change runtime type → T4 GPU**.
2. Optional: set `N_IMAGES = 10` for a quick test run, then set it back to `None`.
3. Run all cells. Results are checkpointed to Google Drive. A full run takes about 1 hour.
