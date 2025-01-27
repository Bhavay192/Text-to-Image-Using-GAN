This is the Tensorflow implementation of TAC-GAN model presented in
[https://arxiv.org/abs/1703.06412](https://arxiv.org/abs/1703.06412).

Text Conditioned Auxiliary Classifier Generative Adversarial Network,
(TAC-GAN) is a text to image Generative Adversarial Network (GAN) for
synthesizing images from their text descriptions. TAC-GAN builds upon the
[AC-GAN](https://arxiv.org/abs/1610.09585) by conditioning the generated images
on a text description instead of on a class label. In the presented TAC-GAN
model, the input vector of the Generative network is built based on a noise
vector and another vector containing an embedded representation of the
textual description. While the Discriminator is similar to that of
the AC-GAN, it is also augmented to receive the text information as
input before performing its classification.

For embedding the textual descriptions of the images into vectors we used
[skip-thought vectors](https://arxiv.org/abs/1506.06726)

The following is the architecture of the TAC-GAN model

<img src="https://chalelele.files.wordpress.com/2017/05/tac-gan-1.png"
height="700" width="400" style="float:center">

# Prerequisites
Some important dependencies are the following and the rest can be installed 
using the ```requirements.txt```
1. Python 3.5
2. [Tensorflow 1.2.0](https://github.com/tensorflow/tensorflow)
4. [Theano 0.9.0](https://github.com/Theano/Theano) : for skip thought vectors
5. [scikit-learn](http://scikit-learn.org/stable/index.html) : for skip thought vectors
6. [NLTK 3.2.1](http://www.nltk.org/) : for skip thought vectors

It is recommended to use a virtual environment for running this project and
installing the required dependencies in it by using the
[***requirements.txt***](https://github.com/dashayushman/TAC-GAN/blob/master/requirements.txt) file.

The project has been tested on a Ubuntu 14.04 machine with an 12 GB NVIDIA
Titen X GPU

# 1. Setup and Run

## 1.1. Clone the Repository

```
git clone https://github.com/dashayushman/TAC-GAN.git
cd TAC-GAN
```

## 1.2. Download the Dataset

The model presented in the paper was trained on the
[flowers dataset](http://www.robots.ox.ac.uk/~vgg/data/flowers/102/ ). This
To train the TAC-GAN on the flowers dataset, first, download the dataset by
doing the following,

1. **Download the flower images** from
[here](http://www.robots.ox.ac.uk/~vgg/data/flowers/102/102flowers.tgz).
Extract the ```102flowers.tgz``` file and copy the extracted ```jpg``` folder
 to ```Data/datasets/flowers```

2. **Download the captions** from
[here](https://drive.google.com/file/d/0B0ywwgffWnLLcms2WWJQRFNSWXM/).
Extract the downloaded file, copy the text_c10 folder and paste it in ```
Data/datasets/flowers``` directory

3. **Download the pretrained skip-thought vectors model** from
[here](https://github.com/ryankiros/skip-thoughts#getting-started) and copy
the downloaded files to ```Data/skipthoughts```

**NB:** *It is recommended to keep all the images in an SSD if available. This
 makes the batch loading and processing operation faster.*

## 1.3. Data Preprocessing
Extract the skip-thought features for the captions and prepare the dataset
for training by running the following script

```
python dataprep.py --data_dir=Data --dataset=flowers
```

This script will create a set of pickled files in the dataset directory which
will be used during training. The following are the available flags for data preparation:

FLAG | VALUE TYPE | DEFAULT VALUE | DESCRIPTION
--- | --- | --- | ---
data_dir | str | Data | The data directory |
dataset | str | flowers | Dataset to use. For Eg., "flowers" |

## 1.4. Training

To train TAC-GAN with the default hyper parameters run the following script

```
python train.py --dataset="flowers" --model_name=TAC-GAN
```

While training, you can montor samples generated by the model in the ```Data/training/TAC_GAN/samples``` directory. Notice that a directory is created according to the ***"model_name"*** that you provide. This directory contains all the data related to a particular experiment. This can also be considered as an ***"experiment name"*** too.

The following flags can be set to change the hyperparameters of the network.

FLAG | VALUE TYPE | DEFAULT VALUE | DESCRIPTION
--- | --- | --- | ---
z-dim | int | 100 | Number of dimensions of the Noise vector |
t_dim | int | 256 | Number of dimensions for the latent representation of the text embedding.
batch_size | int | 64 | Mini-Batch Size
image_size | int | 128 | Batch size to use during training.
gf_dim | int | 64 | Number of conv filters in the first layer of the generator.
df_dim | int | 64 | Number of conv filters in the first layer of the discriminator.
caption_vector_length | int | 4800 | Length of the caption vector embedding (vector generated using skip-thought vectors model).
n_classes | int | 102 | Number of classes
data_dir | String | Data | Data directory
learning_rate | float | 0.0002 | Learning rate
beta1 | float | 0.5 | Momentum for Adam Update
epochs | int | 200 | Maximum number of epochs to train
save_every | int | 30 | Save model and samples after this many number.of iterations
resume_model | Boolean | False | To Load the pre-trained model
data_set | String | flowers | Which dataset to use: "flowers"
model_name | String | model_1 | Name of the model: Can be anything
train | bool | True | This is True while training and false otherwise. Used for batch normalization

We used the following script (hyper-parameters) in for the results that we show in our paper

```
python train.py --t_dim=100 --image_size=128 --data_set=flowers --model_name=TAC_GAN --train=True --resume_model=True --z_dim=100 --n_classes=102 --epochs=400 --save_every=20 --caption_vector_length=4800 --batch_size=128
```

## 1.5. Monitoring

While training, you can monitor the updates on the *terminal* as well as by using [*tensorboard*](https://www.tensorflow.org/get_started/summaries_and_tensorboard)

### 1.5.1 The Terminal:
![Terminal log](https://chalelele.files.wordpress.com/2017/06/terminal_log.png)

### 1.5.1 Tensorboard:

You can use the following script to start tensorboard and visualize realtime changes:

```
tensorboard --logdir=Data/training/TAC_GAN/summaries
```

![Tensorboard](https://chalelele.files.wordpress.com/2017/06/tb.png)

# 2. Generating Images for the text in the dataset

Once you have trained the model for certain epochs you can generate images for all the text descriptions in the dataset use the following script. This will create a synthetic dataset with images generated by the generator.

```
python train.py --data_set=flowers --epochs=100 --output_dir=Data/synthetic_dataset --checkpoints_dir=Data/training/TAC_GAN/checkpoints
```

Notice that the ***checkpoints*** directory is ls created automatically created inside the ***model*** directory after you run the training script.

This script will create the following directory structure:

```
Data
  |__synthetic_dataset
        |___ds
             |___train
             |___val
```

the ***train*** directory will contain all the images generated from the text descriptions of the images in the training set and the same goes for the ***val*** directory.

# 3. Generating Images from any Text

To generate images from any text, do the following

## 3.1 Add Text Descriptions:

Write your text descriptions in a file or use the example file ```Data/text.txt``` that we have provided in the Data directory.
The text description file should contain one text description per line. For example,

```
a flower with red petals which are pointed
many pointed petals
A yellow flower
```

## 3.2 Extract Skip-Thought Vectors:

Run the following script for extracting the Skip-Thought vectors for the text descriptions

```
python encode_text.py --caption_file=Data/text.txt --data_dir=Data
```

This script will create a pickle file called ```Data/enc_text.pkl``` with features extracted from the text descriptions.

## 3.3 Generate Images:

To generate images for the text descriptions, run the following script,

```
python generate_images.py --data_set=flowers --checkpoints_dir=Data/training/TAC_GAN/checkpoints --images_per_caption=30 --data_dir=Data
```

This will create a directory  ```Data/images_generated_from_text/``` with a folder corresponding to every row of the ***text.txt*** file. Each of these folders will contain images for that text.

The following are the parameters you need to set, in case you have used different parameters for training the model.

FLAG | VALUE TYPE | DEFAULT VALUE | DESCRIPTION
--- | --- | --- | ---
z-dim | int | 100 | Number of dimensions of the Noise vector |
t_dim | int | 256 | Number of dimensions for the latent representation of the text embedding.
batch_size | int | 64 | Mini-Batch Size
image_size | int | 128 | Batch size to use during training.
gf_dim | int | 64 | Number of conv filters in the first layer of the generator.
df_dim | int | 64 | Number of conv filters in the first layer of the discriminator.
caption_vector_length | int | 4800 | Length of the caption vector embedding (vector generated using skip-thought vectors model).
n_classes | int | 102 | Number of classes
data_dir | String | Data | Data directory
learning_rate | float | 0.0002 | Learning rate
beta1 | float | 0.5 | Momentum for Adam Update
images_per_caption | int | 30 | Maximum number of images that you want to generate for each of the text descriptions
data_set | String | flowers | Which dataset to use: "flowers"
checkpoints_dir | String | /tmp | Path to the checkpoints directory which will be used to generate the images

# 4. Evaluation

We have used two metrics for evaluating TAC-GAN,

1. [Inception-Scope](https://github.com/openai/improved-gan/tree/master/inception_score)
2. [MS-SSIM score](https://github.com/tensorflow/models/blob/master/compression/image_encoder/msssim.py)

The links are from where we adapted the code for evaluating TAC-GAN. Before evaluating the model, generate a synthetic dataset by referring to [**Section 6**](#6-generating-images-for-text-in-the-dataset)

## 4.1 Inception Score

To calculate the inception score, use the following script,

```
python inception_score.py --output_dir=Data/synthetic_dataset --data_dir=Data --n_images=30000 --image_size=128
```

This will create a collection of all the generated images in ```Data/synthetic_dataset/ds_inception``` and show the inception score on the terminal.

The following are the set of available parameters/flags

FLAG | VALUE TYPE | DEFAULT VALUE | DESCRIPTION
--- | --- | --- | ---
output_dir | str | Data/ds_inception | Directory to dump all the images for calculating the inception score |
data_dir | str | Data/synthetic_dataset/ds | The root directory of the synthetic dataset |
n_images | int | 30000 | Number of images to consider for calculating inception score |
image_size | int | 128 | Size of the image to consider for calculating inception score |


## 4.2 MS-SSIM

To calculate the MS-SSIM score, use the following script,

```
python inception_score.py --output_dir=Data --data_dir=Data --dataset=flowers --syn_dataset_dir=Data/synthetic_datset/ds
```

This will create a ```Data/msssim.tsv``` tab separated file. The data in this file is structured as follows

```
<class_name>  <mean_msssim_real_images>   <mean_msssim_generated_images>
```

Once you have generated the ***msssim.tsv*** file, you can use the following script to generate a figure to compare the MS-SSIM score of the images in the real dataset with the images in the synthetic dataset belonging to the same class,

```
python utility/plot_msssim.py --input_file=Data/msssim.tsv --output_file=Data/msssim
```

This will create ```Data/msssim.pdf```, which is the ***.pdf*** file of the generated figure.

# 5. Generate Interpolated Images

In our paper we show the effect of interpolating the noise and the text embedding vectors on the generated image. Images are randomply selected and their text descriptions are used to generate synthetic images. The following sub-sections will elaborate on how to do it and which scripts will help you in doing it.

## 5.1 Z (Noise) Interpolation

For interpolating the noise vector and generating images, use the following scripts

```
python z_interpolation.py --output_dir=Data/synthetic_dataset --data_set=flowers --checkpoints_dir=Data/training/TAC_GAN/checkpoints --n_images=500
```

This will generate the interpolated images in ```Data/synthetic_dataset/z_interpolation/```.

## 5.1 T (Text Embedding) Interpolation

For interpolating the text embedding vectors and generating images, use the following scripts

```
python t_interpolation.py --output_dir=Data/synthetic_dataset --data_set=flowers --checkpoints_dir=Data/training/TAC_GAN/checkpoints --n_images=500
```

This will generate the interpolated images in ```Data/synthetic_dataset/t_interpolation/```.

***NOTE:*** Both the above mentioned scripts have the same flags/arguments, which are the following,

FLAG | VALUE TYPE | DEFAULT VALUE | DESCRIPTION
--- | --- | --- | ---
z-dim | int | 100 | Number of dimensions of the Noise vector |
t_dim | int | 256 | Number of dimensions for the latent representation of the text embedding.
batch_size | int | 64 | Mini-Batch Size
image_size | int | 128 | Batch size to use during training.
gf_dim | int | 64 | Number of conv filters in the first layer of the generator.
df_dim | int | 64 | Number of conv filters in the first layer of the discriminator.
caption_vector_length | int | 4800 | Length of the caption vector embedding (vector generated using skip-thought vectors model).
n_classes | int | 102 | Number of classes
data_dir | String | Data | Data directory
learning_rate | float | 0.0002 | Learning rate
beta1 | float | 0.5 | Momentum for Adam Update
data_set | str | flowers | The dataset to use: "flowers"
output_dir | String | Data/synthetic_dataset | The directory in which the t_interpolated images will be generated
checkpoints_dir | String | /tmp | Path to the checkpoints directory which will be used to generate the images
n_interp | int | 100 | The factor difference between each interpolation (Should ideally be a multiple of 10)
n_images | int | 500 | Number of images to randomply sample for generating interpolation results

# 6. References

### TAC-GAN

If you find this code usefull, then please use the following BibTex to cite our work.

```
@article{dash2017tac,
  title={TAC-GAN-Text Conditioned Auxiliary Classifier Generative Adversarial Network},
  author={Dash, Ayushman and Gamboa, John Cristian Borges and Ahmed, Sheraz and Afzal, Muhammad Zeshan and Liwicki, Marcus},
  journal={arXiv preprint arXiv:1703.06412},
  year={2017}
}
```

### Oxford-102 Flowers Dataset

If you use the Oxford-102 Flowers Dataset, then please cite their work using the following BibTex.

```
@InProceedings{Nilsback08,
   author = "Nilsback, M-E. and Zisserman, A.",
   title = "Automated Flower Classification over a Large Number of Classes",
   booktitle = "Proceedings of the Indian Conference on Computer Vision, Graphics and Image Processing",
   year = "2008",
   month = "Dec"
}
```

### Skip-Thought

If you use the Skip-Thought model in your work like us, then please cite their work using the following BibTex

```
@article{kiros2015skip,
  title={Skip-Thought Vectors},
  author={Kiros, Ryan and Zhu, Yukun and Salakhutdinov, Ruslan and Zemel, Richard S and Torralba, Antonio and Urtasun, Raquel and Fidler, Sanja},
  journal={arXiv preprint arXiv:1506.06726},
  year={2015}
}
```

### Code

We have referred to the [text-to-image](https://github.com/paarthneekhara/text-to-image) and [DCGAN-tensorflow](https://github.com/carpedm20/DCGAN-tensorflow) repositories for developing our code, and we are extremely thankful to them.
