# BERT-ranking
A simple BERT-based ranking model. The score of a query-document-pair is computed from the output corresponding to the CLS token of BERT.

## Requirements
Install the requirements from `requirements.txt`. Note that [ranking_utils](https://github.com/mrjleo/ranking-utils) has additional dependencies.

## Usage
The following datasets are currently supported:
* [ANTIQUE](https://ciir.cs.umass.edu/downloads/Antique/)
* [FiQA Task 2](https://sites.google.com/view/fiqa/home)
* [InsuranceQA V2](https://github.com/shuzi/insuranceQA)
* [TREC-DL 2019](https://microsoft.github.io/msmarco/TREC-Deep-Learning-2019)
* Any dataset in generic TREC format

### Preprocessing
First, preprocess your dataset by running the preprocessing script. Use
```
python -m ranking_utils.scripts.preprocess -h
```
and
```
python -m ranking_utils.scripts.preprocess [...] <YOUR DATASET> -h
```
for additional help. Example usage:
```
python -m ranking_utils.scripts.preprocess /where/to/save/files antique /path/to/dataset/files
```

### Training and Evaluation
Use the training script to train a new model and save checkpoints. For help, run:
```
python train.py -h
```
Make sure to set `CUDA_VISIBLE_DEVICES` and use the `--gpus` argument when you train. For example, in order to train on GPUs `0` and `2`:
```
CUDA_VISIBLE_DEVICES=0,2 python train.py /preprocessed/files fold_1 --gpus 0 1 --batch_size 32 --precision 16
```
Note that the IDs in `--gpus 0 1` index only the __visible GPUS__, in this case `CUDA_VISIBLE_DEVICES=0,2`.

You can use the `--predict` argument to run the model on the testset using the best checkpoint after the training has finished. This should create output files (one per GPU) in your experiment directory. You can then use the following script to create a TREC runfile that can be evaluated with the TREC evaluation tool:
```
python -m ranking_utils.scripts.convert_to_trec -h
```

### Ranking
You can also use a trained model to re-rank any existing testsets or TREC runfiles:
```
python re_rank.py -h
```
This will again create a new TREC runfile which can be evaluated. __If you use this script, make sure that the query and document IDs in the data file (created by the pre-processing script) and the testsets/runfiles match!__ Again, make sure to set `CUDA_VISIBLE_DEVICES` to control which GPUs are used.
