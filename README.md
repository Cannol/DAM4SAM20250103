<div align="center">

# A Distractor-Aware Memory (DAM) for <br> Visual Object Tracking with SAM2

[Jovana Videnović](https://www.linkedin.com/in/jovana-videnovi%C4%87-5a5b08169/), [Alan Lukežič](https://www.vicos.si/people/alan_lukezic/), and [Matej Kristan](https://www.vicos.si/people/matej_kristan/)

Faculty of Computer and Information Science, University of Ljubljana

[[`Preprint`](https://arxiv.org/abs/2411.17576)]  [[`Project page`](https://jovanavidenovic.github.io/dam-4-sam/) ] [[`DiDi dataset`](#didi-a-distractor-distilled-dataset)]



https://github.com/user-attachments/assets/e90158ba-5c02-489d-9401-26f77f0592b0



https://github.com/user-attachments/assets/0203a96a-c5c9-46f8-90d6-2445d2c5ad73




</div>

## Abstract
Memory-based trackers such as SAM2 demonstrate remarkable performance, however still struggle with distractors. We propose a new plug-in distractor-aware memory (DAM) and management strategy that substantially improves tracking robustness. The new model is demonstrated on SAM2.1, leading to SAM2.1++, which sets a new state-of-the-art on six benchmarks, including the most challenging VOT/S benchmarks without additional training. We also propose a new distractor-distilled (DiDi) dataset to better study the distractor problem. See the [preprint](https://arxiv.org/abs/2411.17576) for more details.

## Installation

To set up the repository locally, follow these steps:

1. Clone the repository and navigate to the project directory:
    ```bash
    git clone https://github.com/jovanavidenovic/DAM4SAM.git
    cd DAM4SAM
    ```
2. Create a new conda environment and activate it:
   ```bash
    conda create -n dam4sam_env python=3.10.15
    conda activate dam4sam_env
    ```
3. Install torch and other dependencies:
   ```bash
   pip install torch==2.1.0 torchvision==0.16.0 --index-url https://download.pytorch.org/whl/cu121
   pip install -r requirements.txt
   ```

If you experience problems as mentioned here, including `ImportError: cannot import name '_C' from 'sam2'`, run the following command in the repository root:
    ```
    python setup.py build_ext --inplace
    ```
Note that you can still use the repository even with the warning above, but some postprocessing SAM2 steps may be skipped. For more information, consult [SAM2 installation instructions]().

## Getting started

Model checkpoints can be downloaded by running:
```bash
cd checkpoints && \
./download_ckpts.sh 
```

Our model configs are available in `sam2/` folder. 

## Running and evaluation

This repository supports evaluation on the following datasets: DiDi, VOT2020, VOT2022, LaSot, LaSoText and GoT-10k. Support for running on VOTS2024 will be added soon. 

### DiDi dataset

Run on a single sequence and visualize results:
```bash
CUDA_VISIBLE_DEVICES=0 python run_on_didi.py --dataset_path <path-to-didi> --sequence <sequence-name>
```

Run on the whole dataset and save results to disk:
```bash
CUDA_VISIBLE_DEVICES=0 python run_on_didi.py --dataset_path <path-to-didi> --output_dir <output-dir-path>
```

After obtaining the raw results on DiDi using previous command, you can compute performance measures. This is done using the VOT toolkit. We thus provide the empty vot workspace in the `didi-workspace` directory. The sequences from DiDi dataset should be placed into the `didi-workspace/sequences` directory. Alternatively, you can just create a symbolic link named `sequences` in the `didi-workspace`, pointed to the DiDi dataset on your disk. The raw results must be placed in the `results` subfolder, e.g. `didi-workspace/results/DAM4SAM`. If the results were obtained using `run_on_didi.py` you should move them to the workspace using the following command:

```bash
python move_didi_results.py --dataset_path <path-to-didi> --src <source-results-directory> --dst ./didi-workspace/results/DAM4SAM
```

The `<source-results-directory>` is the path to the directory used as `output_dir` argument in `run_on_didi.py` script. The `move_didi_results.py` script does not only move the results, but also convert them into bounding boxes since DiDi is a bounding box dataset. Finally, the performance measures are computed using the following commands:

```bash
vot analysis --workspace <path-to-didi-workspace> --format=json DAM4SAM
vot report --workspace <path-to-didi-workspace> --format=html DAM4SAM
```

Performance measures are available in the generated report under `didi-workspace/reports`. Note: if running the analysis multiple times, remember to clear the `cache` directory. 

### VOT2020 and VOT2022 Challenges

Create VOT workspace (for more info see instructions [here](https://www.votchallenge.net/howto/)). For VOT2020 use:
```bash
vot initialize vot2020/shortterm --workspace <workspace-dir-path>
```
and for VOT2022 use:
```bash
vot initialize vot2022/shortterm --workspace <workspace-dir-path>
```

You can use integration files from `vot_integration/vot2022_st` folder to run only on the selected experiment. We provided two stack files: one for the baseline and one for the real-time experiments. After workspace creation and tracker integration you can evaluate the tracker on VOT using the following commands:

```bash
vot evaluate --workspace <path-to-vot-workspace> DAM4SAM
vot analysis --workspace <path-to-vot-workspace> --format=json DAM4SAM
vot report --workspace <path-to-vot-workspace> --format=html DAM4SAM
```

### Bounding box datasets
Running our tracker is supported on LaSot, LaSoText and GoT-10k datasets. Tracker is initialized with masks, which are obtained using SAM2 image predictor, from ground truth initialization bounding boxes. You can download them for all datasets at [this link](https://data.vicos.si/alanl/sam2_init_masks.zip). Before running the tracker, set the corresponding paths to the datasets and the directory with ground truth masks in dam4sam_config.yaml (in the repo root directory).

Run on the whole dataset and save results to disk (arguments for the argument <dataset-name> can be: `got | lasot | lasot_ext`):
```bash
CUDA_VISIBLE_DEVICES=0 python run_on_box_dataset.py --dataset_name=<dataset-name> --output_dir=<output-dir-path>
```

Run on a single sequence and visualize results:
```bash
CUDA_VISIBLE_DEVICES=0 python run_on_box_dataset.py --dataset_name=<dataset-name> --sequence=<sequence-name>
```
## DiDi: A distractor-distilled dataset
DiDi is a distractor-distilled tracking dataset created to address the limitation of low distractor presence in current visual object tracking benchmarks. To enhance the evaluation and analysis of tracking performance amidst distractors, we have semi-automatically distilled several existing benchmarks into the DiDi dataset. The dataset is available for download at [this link](https://go.vicos.si/didi).

<p align="center"> <img src="imgs/didi-examples.jpg" width="80%"> </p>
<div align="center">
  <i>Example frames from the DiDi dataset showing challenging distractors. Targets are denoted by green bounding boxes.</i>
</div>

### Experimental results on DiDi
See [the project page](https://jovanavidenovic.github.io/dam-4-sam/) for qualitative comparison.
| Model         | Quality | Accuracy | Robustness |
|---------------|---------|----------|------------|
| TransT        | 0.465   | 0.669    | 0.678      |
| KeepTrack     | 0.502   | 0.646    | 0.748      |
| SeqTrack      | 0.529   | 0.714    | 0.718      |
| AQATrack      | 0.535   | 0.693    | 0.753      |
| AOT           | 0.541   | 0.622    | 0.852      |
| Cutie         | 0.575   | 0.704    | 0.776      |
| ODTrack       | 0.608   | 0.740 :1st_place_medal:	 | 0.809    |
| SAM2.1Long    | 0.646   | 0.719    | 0.883      |
| SAM2.1   | 0.649 :3rd_place_medal:	 | 0.720    | 0.887 :3rd_place_medal:	 |
| SAMURAI       | 0.680 :2nd_place_medal:	  | 0.722 :3rd_place_medal:	   | 0.930 :2nd_place_medal:	    |
| **SAM2.1++** (ours) | 0.694 :1st_place_medal:	 | 0.727 :2nd_place_medal:	 | 0.944 :1st_place_medal:	 |

## Acknowledgments

Our work is built on top of [SAM 2](https://github.com/facebookresearch/sam2?tab=readme-ov-file) by Meta FAIR.




