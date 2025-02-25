# Trim insignificant weights

This document provides an overview on model pruning to help you determine how it
fits with your usecase. To dive right into the code, see the
[Pruning with Keras](pruning_with_keras.ipynb) tutorial
and the [API docs](../../api_docs/python). For additional details on how to use
the Keras API, a deep dive into pruning, and documentation on more advanced
usage patterns, see the
[Train sparse models](train_sparse_models.md) guide.

## Overview

Magnitude-based weight pruning gradually zeroes out model weights during the
training process to achieve model sparsity. Sparse models are easier to
compress, and we can skip the zeroes during inference for latency improvements.

This technique brings improvements via model compression. In the future,
framework support for this technique will provide latency improvements. We've
seen up to 6x improvements in model compression with minimal loss of accuracy.

The technique is being evaluated in various speech applications, such as
speech recognition and text-to-speech, and has been experimented on across
various vision and translation models.

Users can apply this technique using APIs for Keras on Tensorflow 1.x for
versions 1.14+ and tf-nightly in both graph and eager execution. Note that in
tf-nightly, you must use tf.compat.v1 since 2.x is the default now.

Note: The pruning API is only compatible with `tf.distribute` when using graph
execution.

It is on our roadmap to bring full support to TF 2.x and eager execution.

## Results

### Image Classification

<figure>
  <table>
    <tr>
      <th>Model</th>
      <th>Non-sparse Top-1 Accuracy </th>
      <th>Sparse Accuracy </th>
      <th>Sparsity </th>
    </tr>
    <tr>
      <td rowspan=3>InceptionV3</td>
      <td rowspan=3>78.1%</td>
      <td>78.0%</td>
      <td>50%</td>
    </tr>
    <tr>
      <td>76.1%</td><td>75%</td>
    </tr>
    <tr>
      <td>74.6%</td><td>87.5%</td>
    </tr>
    <tr>
      <td>MobilenetV1 224</td><td>70.9%</td><td>69.5%</td><td>50%</td>
    </tr>
 </table>
</figure>

The models were tested on Imagenet.

### Translation

<figure>
  <table>
    <tr>
      <th>Model</th>
      <th>Non-sparse BLEU </th>
      <th>Sparse BLEU </th>
      <th>Sparsity </th>
    </tr>
    <tr>
      <td rowspan=3>GNMT EN-DE</td>
      <td rowspan=3>26.77</td>
      <td>26.86</td>
      <td>80% </td>
    </tr>
    <tr>
      <td>26.52</td><td>85%</td>
    </tr>
    <tr>
      <td>26.19</td><td>90%</td>
    </tr>
    <tr>
      <td rowspan=3>GNMT DE-EN</td>
      <td rowspan=3>29.47</td>
      <td>29.50</td>
      <td>80% </td>
    </tr>
    <tr>
      <td>29.24</td><td>85%</td>
    </tr>
    <tr>
      <td>28.81</td><td>90%</td>
    </tr>
 </table>
</figure>

The models use WMT16 German and English dataset with news-test2013 as the dev
set and news-test2015 as the test set.

## Examples

In addition to the [Prune with Keras](pruning_with_keras.ipynb)
tutorial, see the following examples:

* Train a CNN model on the MNIST handwritten digit classification task with
pruning:
[code](https://github.com/tensorflow/model-optimization/blob/master/tensorflow_model_optimization/python/examples/sparsity/keras/mnist/mnist_cnn.py)
* Train a LSTM on the IMDB sentiment classification task with pruning:
[code](https://github.com/tensorflow/model-optimization/blob/master/tensorflow_model_optimization/python/examples/sparsity/keras/imdb/imdb_lstm.py)

## Tips

1. Start with a pre-trained model or weights if possible. If not, create one
   without pruning and start after.
2. Do not prune very frequently to give the model time to recover. The toolkit
   provides a default frequency.
3. Try running an experiment where you prune a pre-trained model to the final
   sparsity with begin step 0.
4. Have a learning rate that's not too high or too low when the model is
   pruning. Consider the pruning schedule to be a hyperparameter.

<!-- TODO: include diagrams when learning rate is too high and when it's too low -->

For background, see *To prune, or not to prune: exploring the efficacy of
pruning for model compression* [[paper](https://arxiv.org/pdf/1710.01878.pdf)].
