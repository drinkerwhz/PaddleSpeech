# Parallel WaveGAN with the LJSpeech-1.1
This example contains code used to train a [parallel wavegan](http://arxiv.org/abs/1910.11480) model with [LJSpeech-1.1](https://keithito.com/LJ-Speech-Dataset/).
## Dataset
### Download and Extract the datasaet
Download LJSpeech-1.1 from the [official website](https://keithito.com/LJ-Speech-Dataset/).
### Get MFA results for silence trim
We use [MFA](https://github.com/MontrealCorpusTools/Montreal-Forced-Aligner) results to  cut silence in the edge of audio.
You can download from here [ljspeech_alignment.tar.gz](https://paddlespeech.bj.bcebos.com/MFA/LJSpeech-1.1/ljspeech_alignment.tar.gz), or train your own MFA model reference to [use_mfa example](https://github.com/PaddlePaddle/DeepSpeech/tree/develop/examples/other/use_mfa) of our repo.

## Get Started
Assume the path to the dataset is `~/datasets/LJSpeech-1.1`.
Assume the path to the MFA result of LJSpeech-1.1 is `./ljspeech_alignment`.
Run the command below to
1. **source path**.
2. preprocess the dataset,
3. train the model.
4. synthesize wavs.
    - synthesize waveform from `metadata.jsonl`.
```bash
./run.sh
```

### Preprocess the dataset
```bash
./local/preprocess.sh ${conf_path}
```
When it is done. A `dump` folder is created in the current directory. The structure of the dump folder is listed below.

```text
dump
├── dev
│   ├── norm
│   └── raw
├── test
│   ├── norm
│   └── raw
└── train
    ├── norm
    ├── raw
    └── feats_stats.npy
```

The dataset is split into 3 parts, namely `train`, `dev` and `test`, each of which contains a `norm` and `raw` subfolder. The `raw` folder contains log magnitude of mel spectrogram of each utterances, while the norm folder contains normalized spectrogram. The statistics used to normalize the spectrogram is computed from the training set, which is located in `dump/train/feats_stats.npy`.

Also there is a `metadata.jsonl` in each subfolder. It is a table-like file which contains id and paths to spectrogam of each utterance.

### Train the model
`./local/train.sh` calls `${BIN_DIR}/train.py`.
```bash
CUDA_VISIBLE_DEVICES=${gpus} ./local/train.sh ${conf_path} ${train_output_path}
```
Here's the complete help message.

```text
usage: train.py [-h] [--config CONFIG] [--train-metadata TRAIN_METADATA]
                [--dev-metadata DEV_METADATA] [--output-dir OUTPUT_DIR]
                [--ngpu NGPU] [--verbose VERBOSE] [--batch-size BATCH_SIZE]
                [--max-iter MAX_ITER] [--run-benchmark RUN_BENCHMARK]
                [--profiler_options PROFILER_OPTIONS]

Train a ParallelWaveGAN model.

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG       config file to overwrite default config.
  --train-metadata TRAIN_METADATA
                        training data.
  --dev-metadata DEV_METADATA
                        dev data.
  --output-dir OUTPUT_DIR
                        output dir.
  --ngpu NGPU           if ngpu == 0, use cpu.
  --verbose VERBOSE     verbose.

benchmark:
  arguments related to benchmark.

  --batch-size BATCH_SIZE
                        batch size.
  --max-iter MAX_ITER   train max steps.
  --run-benchmark RUN_BENCHMARK
                        runing benchmark or not, if True, use the --batch-size
                        and --max-iter.
  --profiler_options PROFILER_OPTIONS
                        The option of profiler, which should be in format
                        "key1=value1;key2=value2;key3=value3".
```

1. `--config` is a config file in yaml format to overwrite the default config, which can be found at `conf/default.yaml`.
2. `--train-metadata` and `--dev-metadata` should be the metadata file in the normalized subfolder of `train` and `dev` in the `dump` folder.
3. `--output-dir` is the directory to save the results of the experiment. Checkpoints are save in `checkpoints/` inside this directory.
4. `--ngpu` is the number of gpus to use, if ngpu == 0, use cpu.

### Synthesize
`./local/synthesize.sh` calls `${BIN_DIR}/synthesize.py`, which can synthesize waveform from `metadata.jsonl`.
```bash
CUDA_VISIBLE_DEVICES=${gpus} ./local/synthesize.sh ${conf_path} ${train_output_path} ${ckpt_name}
```
```text
usage: synthesize.py [-h] [--config CONFIG] [--checkpoint CHECKPOINT]
                     [--test-metadata TEST_METADATA] [--output-dir OUTPUT_DIR]
                     [--ngpu NGPU] [--verbose VERBOSE]

Synthesize with parallel wavegan.

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG       parallel wavegan config file.
  --checkpoint CHECKPOINT
                        snapshot to load.
  --test-metadata TEST_METADATA
                        dev data.
  --output-dir OUTPUT_DIR
                        output dir.
  --ngpu NGPU           if ngpu == 0, use cpu.
  --verbose VERBOSE     verbose.
```

1. `--config` parallel wavegan config file. You should use the same config with which the model is trained.
2. `--checkpoint` is the checkpoint to load. Pick one of the checkpoints from `checkpoints` inside the training output directory.
3. `--test-metadata` is the metadata of the test dataset. Use the `metadata.jsonl` in the `dev/norm` subfolder from the processed directory.
4. `--output-dir` is the directory to save the synthesized audio files.
5. `--ngpu` is the number of gpus to use, if ngpu == 0, use cpu.

## Pretrained Models
Pretrained models can be downloaded here. [pwg_ljspeech_ckpt_0.5.zip](https://paddlespeech.bj.bcebos.com/Parakeet/pwg_ljspeech_ckpt_0.5.zip)

Parallel WaveGAN checkpoint contains files listed below.

```text
pwg_ljspeech_ckpt_0.5
├── pwg_default.yaml              # default config used to train parallel wavegan
├── pwg_snapshot_iter_400000.pdz  # generator parameters of parallel wavegan
└── pwg_stats.npy                 # statistics used to normalize spectrogram when training parallel wavegan
```
## Acknowledgement
We adapted some code from https://github.com/kan-bayashi/ParallelWaveGAN.