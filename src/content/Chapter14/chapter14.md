# 第14章生成对抗网络

生成对抗网络（GANs）是一种深层神经网络结构，它由两个相互竞争的网络组成（这也是将其称之为对抗网络的原因。）
2014年，蒙特利尔大学(University of Montreal)的伊恩•古德费勒(Ian Goodfellow)和包括约舒亚•本吉奥(yoshu Bengio)在内的研究人员在一篇论文(https://arxiv.org/abs/1406.2661)中介绍了GANs。说到GANs，脸书的人工智能研究主管，扬·勒恩（Yann LeCun）称，这是在过去十年的机器学习中最有趣的想法。GANs的潜力是巨大的，因为它们可以学习模拟任何数据分布。也就是说，GANs可以在任何领域(图像、音乐、演讲或散文)创建与我们自己的世界极其相似的世界。从某种意义上说，它们是机器人艺术家，它们的作品令人印象深刻。(http://www.nytimes.com/2017/08/14/arts/design/google-how-ai-create-new music and new-artists-project-magenta.html)。

本章将讨论以下内容:

 1、对GANs的一个直观介绍
 
 2、GANs的简单实现
 
 3、深度卷积生成式对抗网络
 

## 对GANs的一个直观介绍

在本节中，我们将以一种非常直观的方式介绍GANs。为了了解GANs是如何工作的，将模拟一个获得门票的情景。故事开始于一个非常有趣的活动，而你想参加这个活动。但是你听说这个活动时所有的票都卖完了，你还是要想尽办法去参加它。所以你想到了一个主意!你想自制一张与真正的门票完全相同或非常相似的票。然而这并不容易，一个很大的问题就是你不知道真正的门票是什么样子的，只能根据自己的想象来设计这张票。你要先设计好门票，然后到活动现场，把门票给保安看。希望他们能让你进去。但是你并不想引起保安的注意，所以你决定求助于你的朋友，他会带你先设计出来的票给保安看。如果未通过，他会观察进去的人拿着的真正的票的样子，给你一些相关信息。你将根据这些信息来修改门票，直到保安允许他进入。以此类推，你将设计一张能让自己通过的门票。

不管这个故事有多假，GANs的工作方式确实跟它很相近。现今GANs非常流行，人们在计算机视觉领域的许多应用中都使用它，并且在许多有趣的应用中都能看到GANs的应用，在此我们将提到并实现其中一些。GNAs包含两个在许多计算机视觉领域都已取得突破成果的主要组成部分。第一个组成部分称为生成器，第二称为判别器。生成器将尝试从特定的概率分布中生成数据样本，这与试图复制活动门票的人非常相似；判别器将会判断（就像保安试图从票中发现瑕疵以此确定票是真还是假）是否它的输入来自初始训练集（门票）或者来自生成器部分(即设计门票的人）。
![](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/图1.png)

## GANs的简单实现

  从伪造活动的门票来看，GNAs 的思想似乎是非常直观的。为了更加明白地理解GNAs 是如何工作的以及如何实现它，我们来展示一个GNAs基于MNIST 数据集的简单实现。
  首先，我们要构建核心GANs网络，这由两个部分组成：生成器和判别器。正如我们说的，生成器将尝试从一个特定的概率分布中想象或伪造数据样本;判别器通过对实际
数据样本的访问和查看，判断出生成器的输出在设计上是否有缺陷，或者是否与原始数据样本非常接近。与活动情景类似，生成器的全部目的是试图使判别器相信生成的
图像来自真实数据集，从而试图欺骗判别器。训练过程与活动故事有相似的结局，生成器将最终生成与原始数据样本非常相似的图像:
![](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/图2.png)
  任何GAN的典型结构如图2所示，并将在MNIST数据集上进行训练。图中Latent样本部分是一个随机的想法或向量，生成器使用它来用假图像复制真实图像。正如我们所提到的，判别器作为一种判断器，它会试着将真实的图像从生成器设计的虚假图像中分离出来。因此，这个网络的输出将是二进制的，它可以用一个sigmoid函数表示，
该函数的值为0(表示输入是假图像)和1(表示输入是真实图像)。让我们开始实现这个结构，看看它如何在MNIST数据集上执行。

首先导入实现所需的库:
```%matplotlib inline
import matplotlib.pyplot as plt
import pickle as pkl
import numpy as np
import tensorflow as tf
```
我们将使用MNIST数据集，因此需要用用TensorFlow helper获取数据集并将其存储在某处:
```
from tensorflow.example.tutorials.mnist import input_data
mnist_dataset = input_data.read_data_sets(‘MNIST_data’)
Output:
Extracting MNIST_data/train-images-idx3-ubyte.gz
Extracting MNIST_data/train-labels-idx1-ubyte.gz
Extracting MNIST_data/t10k-images-idx3-ubyte.gz
Extracting MNIST_data/t10k-labels-idx1-ubyte.gz
```

## 输入模型

  在深入构建由生成器和判别器表示的GAN核心之前，先定义计算图的输入。如图2所示，我们需要两个输入。第一个是真实的图像，它将被输入判别器。另一种输入
称为潜在空间，它将被输入到生成器，并用于生成它的虚假图像:

 ```Defining the model input for the generator and discrimator
def inputs_placeholders (discrimator_real_dim, gen_z_dim):
   real_discrminator_input = tf.placeholder(tf.float32, (None,
discrimator_real_dim), name=”real_discrminator_input” )
       generator_inputs_z = tf.placeholder(tf.float32, (None, gen_z_dim),
name = “generator_input_z”)
       return real_discrminator_input, generator_inputs_z 
 ```
 ![](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/图3.png)
  现在要深入构建结构的两个核心组件。我们将从构建生成器部分开始。如图3所示，生成器至少包含2个隐藏层，它将作为一个估算器工作。在这不使用一般的 ReLU
激活函数，而使用Leaky ReLU 激活函数。这将允许梯度值在不受任何约束的情况下通过该层(关于leaky ReLU的下一节将详细介绍)。

## 可变范围

可变范围是TensorFlow的一项功能，可帮助我们执行以下操作：
确保我们有一些命名约定以便以后检索它们，例如，通过使用单词生成器或判别器开始它们，这将在网络培训期间帮助我们。 我们可以使用这个名字范围功能，
但此功能无法帮助我们实现第二个目的；
能够重用或重新训练相同的网络但具有不同的输入。 例如，我们将从生成器中抽取假图像，看看它有多好生成器用于复制原始生成器。 此外，判别器将具
有访问真实和虚假的图像，这将使我们很容易重用变量而不是在构建计算图时创建新变量.

以下语句将说明如何使用TensorFlow的可变范围功能：

```tf.variable_scope('scopeName',reuse=False)```

**https://www.tensorflow.org/programmers_guide/variable_scope#the_problem可以阅读有关使用可变范围功能的好处的更多信息**

## Leaky ReLU
 
我们提到过我们将使用与ReLU激活功能不同的版本，这被称为Leaky ReLU.传统版本的ReLU激活功能就是这样取输入值和零之间的最大值，通过其他方式截断负值
值为零.Leaky ReLU，这是我们将使用的版本，允许一些负值存在，因此名称Leaky ReLU.
有时，如果我们使用传统的ReLU激活功能，网络就会陷入流行的状态叫做垂死的状态，那是因为网络什么也没产生所有输出都为零。
使用Leaky ReLU的想法是通过允许一些负值, 以防止这种死亡状态要传递的值。
使生成器工作背后的整个想法是从判别器接受梯度值, 如果网络陷入垂死的情况下, 学习过程不会发生。

下图说明了传统的ReLU与其Leaky版本之间的区别：

![deep](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/deep.png)

![deeo](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/deeo.png)

Leaky ReLU激活函数没有在TensorFlow中实现, 因此我们需要自己来实现它。如果输入是正值，此激活函数的输出将是正的,如果输入为负值, 则为受控负值。我们将控制一个名为alpha的参数的负值, 这将通过允许一些负值引入网络的容差。

下面的公式表示我们正在执行的Leaky ReLU:

![](https://latex.codecogs.com/gif.latex?f%28x%29%3Dmax%28a%5Ctimes%20x%2Cx%29)

## 生成器

mnist图像在0和1之间进行归一化, 其中sigmoid激活函数可以表现得最好。但在实践中发现，与其他函数相比，tanh激活函数能提供更好的性能。因此, 为了使用tanh激活函数, 我们需要将这些图像的像素值的范围重新缩放到-1 和1之间:
```python
def generator(gen_z, gen_out_dim, num_hiddern_units=128, reuse_vars=False, leaky_relu_alpha=0.01):
        Building the generator part of the network

        Function arguments
        gen_z : the generator input tensor
        gen_out_dim : the output shape of the generator
        num_hiddern_units : Number of neurons/units in the hidden layer
        reuse_vars : Reuse variables with tf.variable_scope
        leaky_relu_alpha : leaky ReLU parameter

        Function Returns
        tanh_output, logits_layer:
    with tf.variable_scope('generator', reuse=reuse_vars):
        # Defining the generator hidden layer
        hidden_layer_1 = tf.layers.dense(gen_z, num_hiddern_units, activation=None)

        # Feeding the output of hidden_layer_1 to leaky relu
        hidden_layer_1 = tf.maximum(hidden_layer_1, leaky_relu_alpha * hidden_layer_1)

        # Getting the logits and tanh layer output
        logits_layer = tf.layers.dense(hidden_layer_1, gen_out_dim, activation=None)
        tanh_output = tf.nn.tanh(logits_layer)

        return tanh_output, logits_layer

```
现在我们已经准备好了生成器部分。让我们继续前进, 并定义第二个组件的网络

## 判别器
 
 接下来, 我们将构建生成对抗网络中的第二个主要组件,这就是判别器。判别器与生成器大同小异, 但不是使用tanh激活函数, 我们将使用sigmoid激活
 功能;它将产生一个二进制输出, 将代表判断输入图像上的判别器:
  ```python
  def discriminator(disc_input, num_hiddern_units=128, reuse_vars=False, leaky_relu_alpha=0.01):
        Building the discriminator part of the network

        Function Arguments
        disc_input : discrminator input tensor
        num_hiddern_units : Number of neurons/units in the hidden layer
        reuse_vars : Reuse variables with tf.variable_scope
        leaky_relu_alpha : leaky ReLU parameter

        Function Returns
        sigmoid_out, logits_layer:
    with tf.variable_scope('discriminator', reuse=reuse_vars):
        # Defining the generator hidden layer
        hidden_layer_1 = tf.layers.dense(disc_input, num_hiddern_units, activation=None)

        # Feeding the output of hidden_layer_1 to leaky relu
        hidden_layer_1 = tf.maximum(hidden_layer_1, leaky_relu_alpha * hidden_layer_1)

        logits_layer = tf.layers.dense(hidden_layer_1, 1, activation=None)
        sigmoid_out = tf.nn.sigmoid(logits_layer)

        return sigmoid_out, logits_layer
```
## 建立GAN网络
 
 在定义构建生成器和判别器部件的主要功能后,实现把它们堆叠在一起并且定义模型丢失和优化器的功能。
 
## 模型超参数
 
 我们可以通过更改以下一组超参数来微调GANs:
 ```python
# size of discriminator input image
#28 by 28 will flattened to be 784
input_img_size = 784

# size of the generator latent vector
gen_z_size = 100

# number of hidden units for the generator and discriminator hidden layers
gen_hidden_size = 128
disc_hidden_size = 128

#leaky ReLU alpha parameter which controls the leak of the function
leaky_relu_alpha = 0.01

# smoothness of the label
label_smooth = 0.1
 ```
 
## 定义生成器和判别器
 
在定义了我们的体系结构的两个主要部分后, 将被用来生成假mnist 图像 (看起来与真实图像完全相同), 是时候构建网络使用的功能, 我们已经定义到目前为止。为了建立网络, 我们将按照以下步骤操作:
 1.定义模型的输入, 该模型将由两个变量组成。这些变量将是真实的图像中的一个, 这将被提供给判别器, 并第二个将是生成器用来复制原始图像的潜在空间。
 2.调用定义的生成器函数来构建网络的生成器部分。
 3.调用定义的判别器函数来构建网络的判别器部分，但我们将调用此函数两次。第一次调用将是真正的数据和第二次调用将是来自生成器的假数据。
 4.通过重用变量保持真实和假图像的权重不变:
 ```python
 tf.reset_default_graph()

# creating the input placeholders for the discrminator and generator
real_discrminator_input, generator_input_z = inputs_placeholders(input_img_size, gen_z_size)

#Create the generator network
gen_model, gen_logits = generator(generator_input_z, input_img_size, gen_hidden_size, reuse_vars=False,  leaky_relu_alpha=leaky_relu_alpha)

# gen_model is the output of the generator
#Create the generator network
disc_model_real, disc_logits_real = discriminator(real_discrminator_input, disc_hidden_size, reuse_vars=False, leaky_relu_alpha=leaky_relu_alpha)
disc_model_fake, disc_logits_fake = discriminator(gen_model, disc_hidden_size, reuse_vars=True, leaky_relu_alpha=leaky_relu_alpha)
```

## 判别器和生成器损耗

在这一部分中, 我们需要定义判别器和生成器损耗, 这可以被认为是此实现中最棘手的部分。
我们知道, 生成器试图复制原始图像。判别器对生成器生成的图像和原始输入的图像进行判别。因此, 在为每个部分设计我们的损失时, 我们需要针对两件事情。
首先, 我们需要网络的判别器部分能够区分生成器生成的假图像和来自原始的训练实例的真正图像。在训练期间, 我们将把判别器部分分成两类。第一个类是来自原始输入的图像。第二类是生成器生成的假图像。
因此, 判别器的最终一般损失将是其接受真实作为真实的能力,和检测到假的是假的的能力;那么最终的全部损失将是:

disc_loss=disc_loss_real+disc_loss_fake

```python
tf.reduce_mean(
    tf.nn.sigmoid_cross_entropy_with_logits(
        labels=tf.ones_like(disc_logits_fake),
        logits=disc_logits_fake))
```

因此, 我们需要计算两个损失, 才能得出最终的判别损失。        
因为知道这个minist批次中的所有图像都来自minist 数据集的真正的输入图像，第一种损失,disc_loss_real,根据从判别器中得到的logits和标签值进行计算。为了提高模型对测试集的泛化能力, 并给出更好的结果, 人们发现, 实际改变值1到0.9 更好。
这种对标签的更改引入了一种称为标签平滑的内容: 

```python
disc_labels_real = tf.ones_like(disc_logits_real) * (1 - label_smooth)
```

对于第二部分的判别器损耗, 即判别器检测假图像的能力，损失将在从判别器得到logits值和标签值之间;所有的这些都是零, 因为我们知道, 这个minist批次的所有图像来自生成器, 而不是来自原始输入。
现在我们已经讨论了判别器损耗, 我们需要计算生成器的损耗。生成器损耗将被称为gen_loss,这将是disc_logits_fake(判别器判别假图像的输出)和标签（这将是所有的, 因为生成器试图说服判别器假图像的设计是真的）之间的损失：

```python
# calculating the losses of the discrimnator and generator
disc_labels_real = tf.ones_like(disc_logits_real) * (1 - label_smooth)
disc_labels_fake = tf.zeros_like(disc_logits_fake)

disc_loss_real = tf.nn.sigmoid_cross_entropy_with_logits(labels=disc_labels_real, logits=disc_logits_real)
disc_loss_fake = tf.nn.sigmoid_cross_entropy_with_logits(labels=disc_labels_fake, logits=disc_logits_fake)

#averaging the disc loss
disc_loss = tf.reduce_mean(disc_loss_real + disc_loss_fake)

#averaging the gen loss
gen_loss = tf.reduce_mean(
    tf.nn.sigmoid_cross_entropy_with_logits(
        labels=tf.ones_like(disc_logits_fake),
        logits=disc_logits_fake))
```

## 优化器

最后, 优化器部分!在本节中, 我们将定义优化标准, 这些标准将在训练过程中使用。首先, 我们将分别更新生成器和判别器的变量, 所以我们需要能够检索每个部分的变量。
对于第一个优化器, 生成器, 我们将从名为生成器的计算图中的可训练变量开始检索所有的变量,。然后我们就可以通过引用它的名称来检查哪个变量是哪个变量。
我们将对判别变量做同样的, 通过从判别器开始判别所有变量。之后, 我们可以传递我们想要优化的变量列表到优化器。
因此, TensorFlow的可变作用域特征使我们能够从某个字符串开始检索变量,然后我们可以有两个不同的变量列表, 一个用于生成器,另一个用于判别器:

```python
# building the model optimizer

learning_rate = 0.002

# Getting the trainable_variables of the computational graph, split into Generator and Discrimnator parts
trainable_vars = tf.trainable_variables()
gen_vars = [var for var in trainable_vars if var.name.startswith("generator")]
disc_vars = [var for var in trainable_vars if var.name.startswith("discriminator")]

disc_train_optimizer = tf.train.AdamOptimizer().minimize(disc_loss, var_list=disc_vars)
gen_train_optimizer = tf.train.AdamOptimizer().minimize(gen_loss, var_list=gen_vars)
```

## 模型训练

现在让我们开始训练过程, 看看如何管理生成类似于 mnist 的图像:

```python
train_batch_size = 64
num_epochs = 300
generated_samples = []
model_losses = []

saver = tf.train.Saver(var_list=gen_vars)

with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())

    for e in range(num_epochs):
        for ii in range(mnist_dataset.train.num_examples // train_batch_size):
            input_batch = mnist_dataset.train.next_batch(train_batch_size)

            # Get images, reshape and rescale to pass to D
            input_batch_images = input_batch[0].reshape((train_batch_size, 784))
            input_batch_images = input_batch_images * 2 - 1

            # Sample random noise for G
            gen_batch_z = np.random.uniform(-1, 1, size=(train_batch_size, gen_z_size))

            # Run optimizers
            _ = sess.run(disc_train_optimizer,
                         feed_dict={real_discrminator_input: input_batch_images, generator_input_z: gen_batch_z})
            _ = sess.run(gen_train_optimizer, feed_dict={generator_input_z: gen_batch_z})

        # At the end of each epoch, get the losses and print them out
        train_loss_disc = sess.run(disc_loss,
                                   {generator_input_z: gen_batch_z, real_discrminator_input: input_batch_images})
        train_loss_gen = gen_loss.eval({generator_input_z: gen_batch_z})

        print("Epoch {}/{}...".format(e + 1, num_epochs),
              "Disc Loss: {:.3f}...".format(train_loss_disc),
              "Gen Loss: {:.3f}".format(train_loss_gen))

        # Save losses to view after training
        model_losses.append((train_loss_disc, train_loss_gen))

        # Sample from generator as we're training for viegenerator_inputs_zwing afterwards
        gen_sample_z = np.random.uniform(-1, 1, size=(16, gen_z_size))
        generator_samples = sess.run(
            generator(generator_input_z, input_img_size, reuse_vars=True),
            feed_dict={generator_input_z: gen_sample_z})

        generated_samples.append(generator_samples)
        saver.save(sess, './checkpoints/generator_ck.ckpt')

# Save training generator samples
with open('train_generator_samples.pkl', 'wb') as f:
    pkl.dump(generated_samples, f)
```

在运行了300次epoch的模型后, 我们有了一个训练模型, 将能够生成与我们提供给判别器的原始输入图像类似的图像:

```python
fig, ax = plt.subplots()
model_losses = np.array(model_losses)
plt.plot(model_losses.T[0], label='Disc loss')
plt.plot(model_losses.T[1], label='Gen loss')
plt.title("Model Losses")
plt.legend()
```

输出：

![](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/图6.png)

如上图所示, 你可以看到模型损失, 这是表示判别器和生成器的曲线是收敛的。

 
 ## 产生训练集样本

让我们测试模型的性能，看看在临近训练过程结束时，生成器的生成技能（设计门票这件事）是如何得到增强的：

```
def view_generated_samples (epoch_num, g_samples):
    fig, axes = plt.subplots(figsize=(7, 7), nrows=4, ncols=4, sharet = True, sharex = True)
    print(gen_samples[epoch_num][1].shape)
    for ax, gen_image in zip(axes.flatten(), g_samples[0][epoch_num]):
        ax.xaxis.set_visible(False)
        ax.yaxis.set_visible(Flase)
        img = ax.imshow(gen_image.reshape((28, 28)), cmap = 'Greys_r')
    return fig, axes
```

在绘制训练过程中最后一个时期的一些生成图像之前，我们需要在训练过程中加载包含每个时期生成的样本的持久文件：

```
# load samples from generator taken while trainin(在训练时从生成器中产生样本)
with open('train_generator_samples.pkl', 'rb') as f:
    gen_samples = pkl.load(f)
```
    
现在，让我们从训练过程的最后一个时期绘制16个生成的图像，看看生成器如何生成有意义的数字，如3,7和2：

```
_ = view_generated_samples(-1, gen_samples)
```
![4.1](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/4.1.PNG)

我们甚至可以看到在不同的时期生成器的设计技巧。 因此，我们可以设想到在每10个时期由生成器生成的图像：

```
rows, cols = 10, 6
fig, axes = plt.subplots(figsize=(7,12), nrows=rows, ncols=cols, sharex = True, sharey=True)
```

```
for gen_sample, ax_row in zip(gen_samples[::int(len(gen_sample)/rows)], axes):
   for gen_sample, ax_row in zip(gen_samples[::int(len(gen_samples)/cols)], ax_row):
       ax.imshow(image.reshape((28, 28)), camp='Greys_r')
       ax.xaxis.set_visible(False)
       ax.yaxis.set_visible(False)
````

![4.2](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/4.2.PNG)

正如你所看到的，生成器的设计技能及其生成假图像的能力起初非常有限，然后在训练过程结束时得到了增强。

## 从生成器中获取样本

在上一节中，我们介绍了在此GAN架构的训练过程中生成的一些示例。 我们还可以通过加载生成器变量，并为生成器提供可用于生成新图像的新潜在空间，从生成器中生成全新的图像：
```
# Sampling from the generator
saver = tf.train.Saver(var_list=g_vars)

with tf.Session() as sess:
    #restoring the saved checkpoints
    saver.restore(sess, tf.train.latest_checkpoint('checkpoints'))
    gen_sample_z = np.random.uniform(-1, 1, size=(16, z_size))
    generated_samples = sess.run(
            generator(generator_inout_z, input_img_size, reuse_vars=True),
            feed_dict={generator_input_z:gen_sample_z})
    view_generated_samples(0, [generated_samples])
```

![4.3](https://github.com/yanjiusheng2018/dlt/blob/master/src/content/Chapter14/chapter14_image/4.3.PNG)

在实现此示例时，你可以提出一些观察结果。在训练过程的第一个时期，生成器没有任何技能来产生与真正的相似的图像，因为它不知道它们的样子。甚至判别器也不知道如何区分由生成器产生的假图像和真实图像。 在训练开始时，会出现两种有趣的情况。首先，生成器不知道如何创建像我们最初传送到网络的真实图像。 其次，判别器不知道真实图像和假图像之间的区别。

在那之后，生成器开始伪造在某种程度上有意义的图像，这是因为生成器将学习来自原始输入图像的数据分布。 同时，判别器将能够区分假图像和真图像，并且在训练过程结束时将被欺骗。

## 总结

GAN现在被用于许多有趣的应用程序，并且GAN可用于不同的设置，例如半监督和无监督任务。 此外，由于大量研究人员致力于GAN的研究，这些模型正在日益发展，他们生成图像或视频的能力将会越来越好。

这些类型的模型可用于许多有趣的商业应用，例如向Photoshop添加插件可以执行命令，**（make my smile more appealing）使我的笑容更具吸引力**。 它们也可用于图像去噪。


学号|姓名|专业
--|:--:|--:
201802210497|李鑫|应用统计
201802210498|袁晓雨|应用统计
201802210499|朱璐微|应用统计
201802210500|唐倩|应用统计
