Achieving Open Vocabulary Neural Machine Translation with Hybrid Word-Character Models
====================================================================

Code to train hybrid word-character neural machine translation systems described in our ACL paper
<a href="http://nlp.stanford.edu/pubs/luong2016acl_hybrid.pdf">Achieving Open Vocabulary Neural Machine Translation with Hybrid Word-Character Models</a>, which obtain state-of-the-art results in translating English-Czech.


## Features:
- All of the features of the attention-based NMT codebase here
  https://github.com/lmthang/nmt.matlab.
- Train hybrid word-character models.
- Beam-search decoder that can ensembles models including hybrid ones.
- Code to compute source word representations and evaluate on the word
  similarity tasks or do tsne plots.
- Code to compute sentence representations and rerank scores.

## Citations:
If you make use of this code in your research, please cite our paper
```
@inproceedings{luong2016acl_hybrid,
 address = {Berlin, Germany},
 author = {Luong, Minh-Thang  and  Manning, Christopher D.},
 booktitle = {Association for Computational Linguistics (ACL)},
 month = {August},
 title = {Achieving Open Vocabulary Neural Machine Translation with Hybrid Word-Character Models},
 year = {2016}
}

```

- Thang Luong <lmthang@stanford.edu>, 2015, 2016

## Files

```
README.md       - this file
code/           - main Matlab code
  trainLSTM.m: train models
  testLSTM.m: decode models
  computeSentRepresentations.m: compute encoder representations.
  computeRerankScores.m: compute decoding scores.
data/           - toy data
scripts/        - utility scripts
```

The code directory further divides into sub-directories:
```
  basic/: define basic functions like sigmoid, prime. It also has an efficient way to aggreate embeddings.
  layers/: we define various layers like attention, LSTM, etc. with forward and backprop code.
  misc/: things that we haven't categorized yet.
  preprocess/: deal with data.
  print/: print results, logs for debugging purposes.
  wordsim/: word similarity task 
```

## Examples

- Prepare the data
```
./scripts/run_prepare_data.sh ./data/train.10k.en ./data/valid.100.en ./data/test.100.en 1000 ./output/id.1000
./scripts/run_prepare_data.sh ./data/train.10k.de ./data/valid.100.de ./data/test.100.de 1000 ./output/id.1000
```
Here, we convert train/valid/test files in text format into integer format that can be handled efficiently in Matlab. The script syntax is:
```
run_prepare_data.sh <trainFile> <validFile> <testFile> <vocabSize> <outDir>
```

- Train a bilingual, encoder-decoder model

In Matlab, go into the code/ directory and run:
```
trainLSTM('../output/id.1000/train.10k', '../output/id.1000/valid.100', '../output/id.1000/test.100', 'de', 'en', '../output/id.1000/train.10k.de.vocab.1000', '../output/id.1000/train.10k.en.vocab.1000', '../output/basic', 'isResume', 0)
```
This trains a very basic model with all the default settings. We set 'isResume' to 0 so that it will train a new model each time you run the command instead of loading existing models. See trainLSTM.m for more.

(To run directly from your terminal, checkout scripts/train.sh)

The syntax is:
```
trainLSTM(trainPrefix,validPrefix,testPrefix,srcLang,tgtLang,srcVocabFile,tgtVocabFile,outDir,varargin)
% Arguments:
%   trainPrefix, validPrefix, testPrefix: expect files trainPrefix.srcLang,
%     trainPrefix.tgtLang. Similarly for validPrefix and testPrefix.
%     These data files contain sequences of integers one per line.
%   srcLang, tgtLang: languages, e.g. en, de.
%   srcVocabFile, tgtVocabFile: one word per line.
%   outDir: output directory.
%   varargin: other optional arguments.
```

The trainer code outputs logs such as:
```
1, 20, 6.38K, 1, 6.52, gN=11.62
```
which means: at epoch 1, mini-batches 20, training speed is 6.38K words/s, learning rate is 1, train cost is 6.52, and grad norm is 11.62.

Once in a while, the code will evaluate on the valid and test sets:
```
# eval 34.40, 2, 144, 6.19K, 1.00, train=4.76, valid=3.63, test=3.54, W_src{1}=0.050 W_tgt{1}=0.081 W_emb_src=0.050 W_emb_tgt=0.056 W_soft=0.076, time=0.57s
```
which tells us additional information such as the current test perplexity, 34.40, the valid / test costs, and the average abs values of the model paramters.

- Decode
```
testLSTM('../output/basic/modelRecent.mat', 2, 10, 1, '../output/basic/translations.txt')
```
Decode with beamSize 2, collect maximum 10 translations, batchSize 1. 

Note that the testLSTM implicitly decodes the test file specified during training. To specifiy a different test file, use 'testPrefix', see testLSTM.m for more.
```
testLSTM('../output/basic/modelRecent.mat', 2, 10, 1, '../output/basic/translations.txt', 'testPrefix', '../output/id.1000/valid.100')
```

(To run directly from your terminal, checkout scripts/test.sh)

Syntax:
```
testLSTM(modelFiles, beamSize, stackSize, batchSize, outputFile,varargin)
% Arguments:
%   modelFiles: single or multiple models to decode. Multiple models are
%     separated by commas.
%   beamSize: number of hypotheses kept at each time step.
%   stackSize: number of translations retrieved.
%   batchSize: number of sentences decoded simultaneously. We only ensure
%     accuracy of batchSize = 1 for now.
%   outputFile: output translation file.
%   varargin: other optional arguments.
```

- Grad check
```
trainLSTM('', '', '', '', '', '', '', '../output/gradcheck', 'isGradCheck', 1)
```

## Advanced Examples
- Profiling
```
trainLSTM('../output/id.1000/train.10k', '../output/id.1000/valid.100', '../output/id.1000/test.100', 'de', 'en', '../output/id.1000/train.10k.de.vocab.1000', '../output/id.1000/train.10k.en.vocab.1000', '../output/basic', 'isProfile', 1, 'isResume', 0)
```

- Train a monolingual language model
```
trainLSTM('../output/id.1000/train.10k', '../output/id.1000/valid.100','../output/id.1000/test.100', '', 'en', '','../output/id.1000/train.10k.en.vocab.1000', '../output/mono', 'isBi', 0, 'isResume', 0)
```

- Train multi-layer model with dropout
```
trainLSTM('../output/id.1000/train.10k', '../output/id.1000/valid.100', '../output/id.1000/test.100', 'de', 'en', '../output/id.1000/train.10k.de.vocab.1000', '../output/id.1000/train.10k.en.vocab.1000', '../output/advanced', 'numLayers', 2, 'lstmSize', 100, 'dropout', 0.8, 'isResume', 0)
```

We train a 2 stacking LSTM layers with 100 LSTM cells and 100-dim embeddings. Here, 0.8 means the dropout "keep" probability.

- More control of hyperparameters
```
trainLSTM('../output/id.1000/train.10k', '../output/id.1000/valid.100', '../output/id.1000/test.100', 'de', 'en', '../output/id.1000/train.10k.de.vocab.1000', '../output/id.1000/train.10k.en.vocab.1000', '../output/hypers', 'learningRate', 1.0, 'maxGradNorm', 5, 'initRange', 0.1, 'batchSize', 128, 'numEpoches', 10, 'finetuneEpoch', 5, 'isResume', 0)
```

Apart from "obvious" hyperparameters, we want to scale our gradients whenever its norm averaged by batch size (128) is greater than 5. After training for 5 epochs, we start halving our learning rate each epoch. To have further control of learning rate schedule, see 'epochFraction' and 'finetuneRate' options in trainLSTM.m

- Train attention model:
```
trainLSTM('../output/id.1000/train.10k', '../output/id.1000/valid.100', '../output/id.1000/test.100', 'de', 'en', '../output/id.1000/train.10k.de.vocab.1000', '../output/id.1000/train.10k.en.vocab.1000', '../output/attn', 'attnFunc', 1, 'attnOpt', 1, 'isReverse', 1, 'feedInput', 1, 'isResume', 0)
```
Here, we also use source reversing 'isReverse' and the input feeding approach 'feedInput' as described in the paper. Other attention architectures can be specified as follows:
```
  % attnFunc=0: no attention.
  %          1: global attention
  %          2: local attention + monotonic alignments
  %          4: local attention  + regression for absolute pos (multiplied distWeights)
  % attnOpt: decide how we generate the alignment weights:
  %          0: location-based
  %          1: content-based, dot product
  %          2: content-based, general dot product
  %          3: content-based, concat Montreal style
```
'isResume' is set to 0 to avoid loading existing models (done by default), so that you can try different attention architectures.

- More grad checks:
```
./scripts/run_grad_checks.sh > output/grad_checks.txt 2>&1
```
Then compare with the provided grad check outputs data/grad_checks.txt. They
should look similar.

Note: many different configurations will be run with the run_grad_checks.sh script. For many configuration, we set the 'initRange' to a large value 10, so you will notice the total gradient differences are large. This is to debug subtle mistakes; and if the total diff < 10, you can mostly be assured. We do note that with attnFunc=4, attnOpt=1, the diff is quite large; this is something to be checked though the model seems to work in practice.
