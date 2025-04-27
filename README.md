# ASR-Allosaurus
Allosaurus is a pretrained universal phone recognizer. It can be used to recognize phones in more than 2000 languages.
# Prepare Data
To fine-tune your data, you need to prepare audio files and their transcriptions. First, please create one data directory (name can be arbitrary), inside the data directory, create a train directory and a validate directory. Obviously, the train directory will contain your training set, and the validate directory will be the validation set.

Each directory should contain the following two files:

wave: this is a file associating utterance with its corresponding audios
text: this is a file associating utterance with its phones.
# wave
wave is a txt file mapping each utterance to your wav files. Each line should be prepared as follows:

utt_id /path/to/your/audio.wav
Here utt_id denotes the utterance id, it can be an arbitrary string as long as it is unique in your dataset. The audio.wav is your wav file as mentioned above, it should be a mono-channel wav format, but sampling rate can be arbitrary (the tool would automatically resample if necessary) The delimiter used here is space.

To get the best fine-tuning results, each audio file should not be very long. We recommend to keep each utterance shorter than 10 seconds.

# text
text is another txt file mapping each utterance to your transcription. Each line should be prepared as follows

utt_id phone1 phone2 ...
Here utt_id is again the utterance id and should match with the corresponding wav file. The phone sequences came after utterance id is your phonetic transcriptions of the wav file. The phones here should be restricted to the phone inventory of your target language. Please make sure all your phones are already registered in your target language by the list_phone command

# Feature Extraction
Next, we will extract feature from both the wave file and text file. We assume that you already prepared wave file and text file in BOTH train directory and validate directory

# Audio Feature
To prepare the audio features, run the following command on both your train directory and validate directory.

# command to prepare audio features
python -m allosaurus.bin.prep_feat --model=some_pretrained_model --path=/path/to/your/directory (train or validate)
The path should be pointing to the train or the validate directory, the model should be pointing to your traget pretrained model. If unspecified, it will use the latest model. It will generate three files feat.scp, feat.ark and shape.

The first one is an file indexing each utterance into a offset of the second file.

The second file is a binary file containing all audio features.

The third one contains the feature dimension information

If you are curious, the scp and ark formats are standard file formats used in Kaldi.

# Text Feature
To prepare the text features, run the following command again on both your train directory and validate directory.

# command to prepare token
python -m allosaurus.bin.prep_token --model=<some_pretrained_model> --lang=<your_target_language_id> --path=/path/to/your/directory (train or validate)
The path and model should be the same as the previous command. The lang is the 3 character ISO language id of this dataset. Note that you should already verify the the phone inventory of this language id contains all of your phone transcriptions. Otherwise, the extraction here might fail.

After this command, it will generate a file called token which maps each utterance to the phone id sequences.

# Training
Next, we can start fine-tuning our model with the dataset we just prepared. The fine-tuning command is very simple.

# command to fine_tune your data

python -m allosaurus.bin.adapt_model --pretrained_model=<pretrained_model> --new_model=<your_new_model> --path=/path/to/your/data/directory --lang=<your_target_language_id> --device_id=<device_id> --epoch=<epoch>

There are couple of other optional arguments available here, but we describe the required arguments.
pretrained_model should be the same model you specified before in the prep_token and prep_feat.
new_model can be an arbitrary model name (Actually, it might be easier to manage if you give each model the same format as the pretrained model (i.e. YYMMDD))
The path should be pointing to the parent directory of your train and validate directories.
The lang is the language id you specified in prep_token
The device_id is the GPU id for fine-tuning, if you do not have any GPU, use -1 as device_id. Multiple GPU is not supported.
epoch is the number of your training epoch

During the training, it will show some information such as loss and phone error rate for both your training set and validation set. After each epoch, the model would be evaluated with the validation set and would save this checkpoint if its validation phone error rate is better than previous ones. After the specified epoch has finished, the fine-tuning process will end and the new model should be available.

# Testing
After your training process, the new model should be available in your model list. use the list_model command to check your new model is available now

# command to check all your models

python -m allosaurus.bin.list_model

If it is available, then this new model can be used in the same style as any other pretrained models. Just run the inference to use your new model.

python -m allosaurus.run --lang <language id> --model <your new model> --device_id <gpu_id> -i <audio>
