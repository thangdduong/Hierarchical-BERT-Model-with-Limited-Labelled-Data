# A Sentence-level Hierarchical BERT Model for Document Classification with Limited Labelled Data

This repository is temporarily associated with paper [Lu, J., Henchion, M., Bacher, I. and Mac Namee, B., 2021. A Sentence-level Hierarchical BERT Model for Document Classification with Limited Labelled Data. arXiv preprint arXiv:2106.06738.](https://arxiv.org/pdf/2106.06738.pdf)


### Dependencies
Tested Python 3.6, and requiring the following packages, which are available via PIP:

* Required: [numpy >= 1.19.5](http://www.numpy.org/)
* Required: [scikit-learn >= 0.21.1](http://scikit-learn.org/stable/)
* Required: [pandas >= 1.1.5](https://pandas.pydata.org/)
* Required: [gensim >= 3.7.3](https://radimrehurek.com/gensim/)
* Required: [matplotlib >= 3.3.3](https://matplotlib.org/)
* Required: [torch >= 1.9.0](https://pytorch.org/)
* Required: [transformers >= 4.8.2](https://huggingface.co/transformers/)
* Required: [Keras >= 2.0.8](https://keras.io/)
* Required: [Tensorflow >= 1.14.0](https://www.tensorflow.org/)
* Required: [FastText model trained with Wikipedia 300-dimension](https://fasttext.cc/docs/en/pretrained-vectors.html)
* Required: [GloVe model trained with Gigaword and Wikipedia 200-dimension](https://nlp.stanford.edu/projects/glove/)
* Required: packaging >= 20.0


### Step 1. Data Processing

The first step is encoding raw text data into different high-dimensional vectorised representations. The raw text data should be stored in directory __"raw_corpora/"__, each dataset should have its individual directory, for example, the "longer_moviereview/" directory under folder "raw_corpora/". The input corpus of documents should consist of plain text files stored in csv format (two files for one corpus, one for documents belong to class A and one for documents for class B) with a columan named as __text__. It should be noted that the csv file must be named in the format _#datasetname_neg_text.csv_ or _#datasetname_pos_text.csv_. Each row corresponding to one document in that corpus, the format can be refered to the csv file in the sample directory "raw_corpora/longer_moviereview/". Then we can start preprocessing text data and converting them into vectors by:

    python encode_text.py -d dataset_name -t encoding_methods
    
The options of -t are `hbm` (corresponding to the sentence representation generated by the token-level RoBERTa encoder in the paper), `roberta-base` and `fasttext`, for example `-t roberta-base,fasttext` means encoding documents by RoBERTa and FastText respectively. The encoded documents are stored in directory __"dataset/"__, while the FastText document representations are stored in __"dataset/fasttext/"__ and other representations are stored in __"dataset/roberta-base/"__. It should be noted that the sentence representations for hbm is suffixed by ".pt" and the document representations generated by RoBERTa are suffixed by ".csv"(average all tokens to represent a document) or "\_cls.csv" (using classifier token "\<s\>" to represent a document). Due to the upload file size limit, we did not upload sample ".pt" files but you can generate yours. For encoding by FastText, you need to download the pretrained FastText model in advance (see __Dependencies__).

### Step 2. Run Hierarchical BERT Model (HBM)(our approach)

We can evaluate the Hierarchical BERT Model (HBM) with limited number of labelled data (in this experiment, we subsample the fully labelled dataset to simulate this low-shot scenario) by:

    python run_hbm.py -d dataset_name -l learning_rate -e num_of_epochs -r random_seeds -s training_set_size

The `training_set_size` can be randome numbers up to 200 (also, you can customise the maximum number by editing the script), for example  `-s 50,100,150` means the training HBM with 50, 100 and 150 labelled instances respectively.  The `random_seeds` are random state for subsampling training set from the whole dataset. For example `-r 1988,1999 -s 50,100` will training HBM with four different training sets, i.e. 50 labelled instances sampled by seed 1988, 50 labelled instances sampled by seed 1999, 100 labelled instances sampled by seed 1988 and 100 labelled instances sampled by seed 1999.
The script then evaluate the performance of HBM in the rest testing set (i.e. the whole dataset minus the 200 instances that sampled out as the training set, the details can be referred in the paper). The evaluation results are stored in directory "outputs/". Furthermore, the concrete results of each step are stored in "outputs/hbm_results/".

### Step 3. Run fine-tuned RoBERTa (baseline)

Similar to the above settings, we can evaluate the fine-tuned RoBERTa performance with limited number of labelled data by:

    python run_fine_tuned_roberta.py -d dataset_name -l learning_rate -e num_of_epochs -r random_seeds -s training_set_size
    
Similarly, the evaluation results are stored in directory "outputs/". Furthermore, the concrete results of each step are stored in "outputs/fine_tuned_results/". It should be noted that the directories "fine_tuned_data/", "fine_tuned_outputs/" and "fine_tuned_cache/" are used for stored auxiliary information generated during the fine-tuning and hence these three directories should be created in advance.

### Step 4. Run Hierarchical Attention Networks (HAN)(baseline)

Similar to the above settings, we can evaluate the Hierarchical Attention Networks with limited number of labelled data by:
    
    python run_han.py -d dataset_name -l learning_rate -e num_of_epochs -r random_seeds -s training_set_size
    
The preprocessing and text encoding (with GloVe) are also integrated into this script and you should download the GloVe model in advance (see __dependencies__). The evaluation results are stored in dictory "outputs/".

## Step 5. Run SVM-based methods (i.e. pretrained RoBERTa + SVM and FastText + SVM) (baseline)

