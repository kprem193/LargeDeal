References:
https://github.com/mozilla/DeepSpeech

## DeepSpeech - CPU Training:

Install Git-LFS :
``` 
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
```
Clone DeepSpeech Repo :
```
git clone https://github.com/mozilla/DeepSpeech.git
```
Download the pre-built models (graph file and vocab):

```
wget -O - https://github.com/mozilla/DeepSpeech/releases/download/v0.1.1/deepspeech-0.1.1-models.tar.gz | tar xvfz -
```
### Install the requirements:
```
pip3 install -r requirements.txt
```
If there is an error in installing cryptography run this command:
```
sudo apt-get install build-essential libssl-dev libffi-dev python-dev
pip install cryptography
```
Install sox mp3 handler
```
sudo apt-get install libsox-fmt-mp3
```
Install libpthread
```
sudo apt-get install libpthread-stubs0-dev

```
Download the pre-built binaries:
```
python3 util/taskcluster.py --target .
```
Prepare the data for training
Make 3 csv files train.csv, test.csv, dev.csv

Format of csv : 'wav_filename', 'wav_filesize', 'transcript'

Run the following command to start the training from scratch:
``` 
python3 ./DeepSpeech.py --checkpoint_dir provide_checkpoint_directory/ --epoch provide_number_of_epoch(10) --train_files train.csv --dev_files dev.csv --test_files test.csv --export_dir provide_the_directory_to_get_output_graph --decoder_library_path ./libctc_decoder_with_kenlm.so
```
Run the following command to start the training the pretrained model :
```
python3 ./DeepSpeech.py --n_hidden 2048 --initialize_from_frozen_model models/output_graph.pb --checkpoint_dir provide_checkpoint_directory/ --epoch provide_number_of_epoch(10) --train_files train.csv --dev_files dev.csv --test_files test.csv --export_dir provide_the_directory_to_get_output_graph --decoder_library_path ./libctc_decoder_with_kenlm.so
```
## DeepSpeech - GPU Training
You need to have a NVIDIA_GPU which supports CUDA and CUDNN (most of the latest GPU's support it)

Install the NVIDIA_GPU Driver if you don't have it

Install CUDA toolkit 9.0 and CUDNN 7.1.4 for CUDA 9.0

References: https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html, https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07

Install Git-LFS :
``` 
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
```
Clone DeepSpeech Repo :
```
git clone https://github.com/mozilla/DeepSpeech.git
```
Download the pre-built models (graph file and vocab):

```
wget -O - https://github.com/mozilla/DeepSpeech/releases/download/v0.1.1/deepspeech-0.1.1-models.tar.gz | tar xvfz -
```
### Install the requirements:
```
pip3 install -r requirements.txt
```
If there is an error in installing cryptography run this command:
```
sudo apt-get install build-essential libssl-dev libffi-dev python-dev
pip install cryptography
```
Install sox mp3 handler
```
sudo apt-get install libsox-fmt-mp3
```
Install libpthread
```
sudo apt-get install libpthread-stubs0-dev

```
Download the pre-built binaries for GPU:
```
python3 util/taskcluster.py --arch gpu --target .
```
Remove normal tensorflow and install tensorflow-gpu:
```
pip3 uninstall tensorflow
pip3 install 'tensorflow-gpu==1.6.0'
```

Prepare the data for training
Make 3 csv files train.csv, test.csv, dev.csv

Format of csv : 'wav_filename', 'wav_filesize', 'transcript'

Run the following command to start the training from scratch:
``` 
python3 ./DeepSpeech.py --checkpoint_dir provide_checkpoint_directory/ --epoch provide_number_of_epoch(10) --train_files train.csv --dev_files dev.csv --test_files test.csv --export_dir provide_the_directory_to_get_output_graph --decoder_library_path ./libctc_decoder_with_kenlm.so
```
Run the following command to start the training the pretrained model :
```
python3 ./DeepSpeech.py --n_hidden 2048 --initialize_from_frozen_model models/output_graph.pb --checkpoint_dir provide_checkpoint_directory/ --epoch provide_number_of_epoch(10) --train_files train.csv --dev_files dev.csv --test_files test.csv --export_dir provide_the_directory_to_get_output_graph --decoder_library_path ./libctc_decoder_with_kenlm.so
```
## After Training
You will get the output file in export_directory provided,

To run you trained model run this command in your DeepSpeech Directory:
```
./deepspeech --model path_output_file_you_get_in_export_directory(output_graph.pb) --alphabet models/alphabet.txt --lm models/lm.binary --trie models/trie --audio audio_input.wav
```

## Notes:
To test if your setup is right, run:
```
./bin/run-ldc93s1.sh
```
About Checkpoints:

During training of a model so-called checkpoints will get stored on disk. This takes place at a configurable time interval. The purpose of checkpoints is to allow interruption (also in the case of some unexpected failure) and later continuation of training without losing hours of training time. Resuming from checkpoints happens automatically by just (re)starting training with the same --checkpoint_dir of the former run.

Be aware however that checkpoints are only valid for the same model geometry they had been generated from. In other words: If there are error messages of certain Tensors having incompatible dimensions, this is most likely due to an incompatible model change. One usual way out would be to wipe all checkpoint files in the checkpoint directory or changing it before starting the training.
