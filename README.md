# LIMU-BERT
LIMU-BERT, a novel representation learning model that can make use of unlabeled IMU data and extract generalized rather than task-specific features. 
LIMU-BERT adopts the principle of self-supervised training of the natural language model BERT to effectively capture temporal relations and feature distributions in IMU sensor data. With the representations learned via LIMU-BERT, task-specific models trained with limited labeled samples can achieve superior performances. The designed models are lightweight and easily deployable on mobile devices. 

Please check our paper for more details.
## File Overview
This project contains following folders and files.
- [`config`](./config) : config json files of models and training hyper-parameters.
- [`dataset`](./dataset) : the scripts for preprocessing four open datasets and a config file of key attributes of those datasets.
- [`benchmark.py`](./benchmark.py) : run DCNN, DeepSense, and R-GRU.
- [`classifier.py`](./classifier.py) : run LIMU-GRU that inputs representations learned by LIMU-BERT and output labels for target applications.
- [`classifier_bert.py`](./classifier_bert.py) : run LIMU-GRU that inputs raw IMU readings and output labels for target applications.
- [`config.py`](./config.py) : some helper functions for loading settings.
- [`embedding.py`](./embedding.py) : generates representation or embeddings for raw IMU readings given a pre-trained LIMU-BERT.
- [`models.py`](./models.py) : the implementations of LIMU-BERT, LIMU-GRU, and other baseline models.
- [`plot.py`](./plot.py) : some helper function for plotting IMU sensor data or learned representations.
- [`pretrain.py`](./pretrain.py) : pretrain LIMU-BERT.
- [`statistic.py`](./statistic.py) : some helper functions for evaluation.
- [`train.py`](./train.py) : several helper functions for training models.
- [`utils.py`](./utils.py) : some helper functions for preprocessing data or separating dataset.

## Setup
This repository has be tested for Python 3.7.7/3.8.5 and Pytorch 1.5.1/1.7.1. To install all dependencies, use the following command:

```
$ pip install -r requirements.txt
```

## Prepare dataset
In the [`dataset`](./dataset) folder, we provide four scripts that preprocess the corresponding datasets. Those datasets are widely adopted in the previous studies:
- [HHAR](http://archive.ics.uci.edu/ml/datasets/heterogeneity+activity+recognition)
- [UCI](http://archive.ics.uci.edu/ml/datasets/Smartphone-Based+Recognition+of+Human+Activities+and+Postural+Transitions)
- [MotionSense](https://github.com/mmalekzadeh/motion-sense)
- [Shoaib](https://www.utwente.nl/en/eemcs/ps/research/dataset/)

Each script has a kernel function which transform the raw IMU data and output preprocessed data and label. You can set the sampling rate and window size (sequence length).
- Data: a numpy array with the shape of (N\*W\*F), N is the number of samples, W is the windows size, and F is the number of features (6 or 9).
- Label: a numpy array with the shape of (N\*W\*L), N is the number of samples, W is the windows size, and L is the number of label types (e.g., activity and user label). The detailed label information is provied in [`data_config.json`](./dataset/data_config.json).

The two numpy arrays are saved as "data_X_Y.npy" and "label_X_Y.npy" in each dataset folder, where X represents the sampling rate and Y is the window size. 
For example, all data and label are saved as "data_20_120.npy" and "label_20_120.npy" in our experiments and the data and label arrays of HHAR dataset are saved in the _dataset/hhar_ folder.

## Framework
In our framework, there are two phases:
- Self-supervised training phase: train LIMU-BERT with unlabeled IMU data.
- Supervised training phase: train LIMU-GRU based on the learned representations.

In implementation, there are three steps to run the codes:
- [`pretrain.py`](./pretrain.py) : pretrain LIMU-BERT.
- [`embedding.py`](./embedding.py) : generates and save representations learned by LIMU-BERT.
- [`classifier.py`](./classifier.py) : load representations and train a task-specific classifier.

For other baseline models, directly run [`benchmark.py`](./benchmark.py) or [`tpn.py`](./tpn.py).
## Usage
[`pretrain.py`](./pretrain.py), [`embedding.py`](./embedding.py), [`classifier.py`](./classifier.py), 
[`benchmark.py`](./benchmark.py), and [`classifier_bert.py`](./classifier_bert.py) share the same usage pattern.
```
usage: xxx.py [-h] [-g GPU] [-f MODEL_FILE] [-t TRAIN_CFG] [-a MASK_CFG]
                   [-l LABEL_INDEX] [-s SAVE_MODEL]
                   model_version {hhar,motion,uci,shoaib} {10_100,20_120}

positional arguments:
  model_version         Model config, e.g. v1
  {hhar,motion,uci,shoaib}
                        Dataset name
  {10_100,20_120}       Dataset version

optional arguments:
  -h, --help            show this help message and exit
  -g GPU, --gpu GPU     Set specific GPU
  -f MODEL_FILE, --model_file MODEL_FILE
                        Pretrain model file, default: None
  -t TRAIN_CFG, --train_cfg TRAIN_CFG
                        Training config json file path
  -a MASK_CFG, --mask_cfg MASK_CFG
                        Mask strategy json file path, default: config/mask.json
  -l LABEL_INDEX, --label_index LABEL_INDEX
                        Label Index setting the task, default: -1
  -s SAVE_MODEL, --save_model SAVE_MODEL
                        The saved model name, default: 'model'
```
### [`pretrain.py`](./pretrain.py)
Example:
```
pretrain.py v1 uci 20_120 -s limu_v1 
```
For this command, we will train a LIMU-BERT, whose settings are defined in the _based_v1_ of [`limu_bert.json`](./config/limu_bert.json),
with the UCI dataset "data_20_120.npy" and "label_20_120.npy". The trained model will be saved as "limu_v1.pt" in the _saved/pretrain_base_uci_20_120_ folder.
The mask and train settings are defined in the [`mask.json`](./config/mask.json) and [`pretrain.json`](./config/pretrain.json), respectively.

In the main function of [`pretrain.py`](./pretrain.py), you can set following parameters:
- _training_rate_: float, defines the proportion of unlabeled training data we want to use. The default value is 0.8.
### [`embedding.py`](./embedding.py)
Example:
```
embedding.py v1 uci 20_120 -f limu_v1
```
For this command, we will load the pretrained LIMU-BERT from file "limu_v1.pt" in the _saved/pretrain_base_uci_20_120_ folder.
And embedding.py will save the learned representations as "embed_limu_v1_uci_20_120.npy" in the _embed_ folder.

### [`classifier.py`](./classifier.py)
Example:
```
classifier.py v2 uci 20_120 -f limu_v1 -s limu_gru_v1 -l 0
```
For this command, we will load the embeddings or representations from "embed_limu_v1_uci_20_120.npy" and train the GRU classifier
, whose settings are defined in the _gru_v2_ of [`classifier.json`](./config/classifier.json). 
The trained GRU classifier will saved as "limu_gru_v1.pt" in the _saved/classifier_base_uci_20_120_ folder.
The target task corresponds to the first label in "label_20_120.npy" of UCI dataset, which is a human activity recognition task defined in [`data_config.json`](./dataset/data_config.json).
The train settings are defined in the [`train.json`](./config/train.json).

In the main function of [`classifier.py`](./classifier.py), you can set following parameters:
- _training_rate_: float, defines the proportion of unlabeled data that the pretrained LIMU-BERT uses. The default value is 0.8. 
Note that this value must equal to the _training_rate_ in the [`pretrain.py`](./pretrain.py).
- _label_rate_: float, defines the proportion of labeled data to the unlabeled training data that the pretrained LIMU-BERT uses.
- _balance_: bool, defines whether it should use balanced labeled sample among the multiple classes. Default: True.
- _method_: str, defines the classifier type from {gru, lstm, cnn1, cnn2, attn}. Default: gru.

If you are confused about the above settings, please check the Section 4.1.1 in our paper for more details.

### [`classifier_bert.py`](./classifier_bert.py)
Example:
```
classifier_bert.py v1_v2 uci 20_120 -f limu_v1 -s limu_gru_v1 -l 0
```
For this command, we will train the a composite classifier with pretrained LIMU-BERT and GRU classifier
, whose settings are defined in the _gru_v2_ of [`classifier.json`](./config/classifier.json). 
The trained LIMU-GRU classifier will saved as "limu_gru_v1.pt" in the _saved/bert_classifier_base_uci_20_120_ folder.
The train settings are defined in the [`bert_classifier_train.json`](./config/bert_classifier_train.json). 
Note that "v1_v2" defines two model versions, which corresponds to the LIMU-BERT and GRU classifier, respectively.
### [`benchmark.py`](./benchmark.py)
Example:
```
benchmark.py v1 uci 20_120 -s dcnn_v1 -l 0
```
For this command, we will train a DCNN model, whose settings are defined in the _dcnn_v1_ of [`classifier.json`](./config/classifier.json). 
The trained DCNN classifier will saved as "dcnn_v1.pt" in the _saved/bench_dcnn_uci_20_120_ folder.

In the main function of [`benchmark.py`](./benchmark.py), the parameters are same to those in [`classifier.py`](./classifier.py).


## Citation
LIMU-BERT: Unleashing the Potential of Unlabeled Data for IMU Sensing Applications

## Reference
The implementation of BERT part refers the codes of [ALBERT](https://github.com/dhlee347/pytorchic-bert).

## Contact
huatao001@e.ntu.edu.sg (preferred)

735820057@qq.com

