## 2.2 [Train the Neural Entity Extraction Model](3_Train_Neural_Entity_Extractor_GPU.py)

### Objective

The [companion script](3_Train_Neural_Entity_Extractor_GPU.py) describes how you can use [Keras](https://keras.io/) with [TensorFlow](https://www.tensorflow.org/) backend to train a deep neural network for entity recognition. We demonstrate how we can use the word embeddings extracted previously to initialize the embedding layer of the neural network. The main hypothesis of this work is that the incorporating of word embeddings as features into a
deep learning model could help to recognize unseen or very rare drug mentions in the training set. The task at hand is to identity drug and disease mentions in a given text. We are using an auto-labeled dataset which is the combination of SemEval 2013 - Task 9.1 (Drug Recognition) and BioCreative V CDR task corpus.

Before, proceeding forward make sure you have the word embeddings model trained. You can refer to this [Python script](../01_feature_engineering/2_Train_Word2Vec_Model_Spark.py) and its [documentation](../01_feature_engineering/ReadMe.md)
to see how to train your own word embedding model for the biomedical domain using Word2Vec implementation on Spark.

Once you have the embedding model ready you can start working on training the neural network for the entity recognition task.

### Execution Steps 

* Step 1: Copy the dataset and evaluation scripts to correct locations and Read Word Embeddings from Parquet Files:
We are using the [fastparquet package](https://pypi.python.org/pypi/fastparquet) as a way to read the embeddings from the parquet files and load them to a Pandas dataframe. You can then store this embedding matrix 
in any format and use it either as a TSV to visualize using the [Projector for Tensorflow](http://projector.tensorflow.org/) or use as a lookup table for downstream Deep Learning Tasks. The code provides a way to download the files from the Blob connected to your Spark Cluster. For more information about Azure blob storage, refer to [here](https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-how-to-use-blobs)

* Step 2: Prepare the data for training and testing in a format that is suitable for Keras. The DataReader/read_and_parse_training_data function is the one that does that.
 - It first reads the word embeddings and a word_to_index_map mapping each word in the embeddings to an index. It also creates a list where each item id refers to the the word vector corresponding to that index and hence to its word.
 - Next it reads the training and testing data line by line and appends a sentence to a list. It also creates one-hot vectors for each of the supported entity types (such as B-Disease, I-Disease, B-Drug etc.)
 - Once the list of sentences is ready, its time now to replace each word with its index from the above map. If we find a word which is not present in our vocabulary, we replace the word by the token "UNK".
 To generate the vector for "UNK" we sample a random vector, which has the same dimension as our embeddings, from a Normal Distribution. Since the number of words in each sentence might differ, we pad each sequence 
 to make sure that they have the same length. We add an additional tag "NONE" for each of the padded term. We also associate a zero vector with the paddings. The final shape of the train and test data should be (number of samples, max_sequence_length). This is the shape that can be fed to the [Embedding Layer](https://keras.io/layers/embeddings/) in Keras. Once we have this shape for our dataset we are ready for training our neural network (but first lets create one).
  
 * Step 3: This step decribes how to define the deep neural network architecture in Keras. We first create a [sequential](https://keras.io/getting-started/sequential-model-guide/) model for our neural network.
   1. We start by adding an [Embedding Layer](https://keras.io/layers/embeddings/) to our model and specify the input shape as created above. We load our pre-trained Embeddings for the weights of this layer and set the *trainable* flag as False since we do not want 
 to update the Embeddings (but this can change). 
   2. Next we add a [Bi-Directional LSTM layer](https://keras.io/layers/wrappers/#bidirectional) because we train the model to take into consideration not only the previous words but also next words when it predicts the output of teh current word. 
   3. Then we add a [dropout layer](https://keras.io/layers/core/#dropout) for regularization. 
   
   We can repeat the steps 2 and 3 several times to train a deeper model. Finaly, we add a [TimeDistributed Dense Layer](https://keras.io/layers/wrappers/#timedistributed). This layer is responsible for generating predictions for each word in the sentence.

 The following figure demonstrates the trained model architecture.     
        
   ![LSTM model](../../../docs/images/d-a-d-model.png)

   We optimize the [categorical_crossentropy](https://keras.io/losses/#categorical_crossentropy) loss and are using the [Adam](https://keras.io/optimizers/#adam) optimizer.

* Step 4: Now we have the data ready and the neural network architecture defined, so lets put them together and start the training. This step shows how to call the previously defined functions. We specify the paths of the training and the test files along with some parameters like 

    * vector size: this is the length of a word vector (50 for us).
    * num_classes:     this is the number of unique classes in the training and test sets (the clas would be B-Chemical, I-Disease, O etc.)
    *     seq_length:  this is the max_sequence_length found above
        num_layers:      number of LSTM layers 
        num_hidden_units: number of LSTM cells in each layer
        batch_size: number of training example at each weight update  
        num_epochs:      number of neural network training epochs

Once these are set, the model we start to train. 
Run the following command to ensure that the training is executed on GPU and to monitor the GPU utilization:

```
nvidia-smi -l
```
![GPU utilization](../../../docs/images/gpu-usage.png)

   Next step would be to obtain model predictions on the test set and evaluate the performance of the model.

The output of the training phase are two files: the trained model model.h5 file and the resources.pkl file. The resources.pkl file contains the metadata of the trained model and the word embedding lookup table. 

### Next Step
2.3. [Model evaluation](../03_model_evaluation/ReadMe.md)
