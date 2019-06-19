# StackNN
This project implements differentiable stacks and queues in PyTorch. The data structures are implemented in such a way that it should be easy to integrate them into your own models. For example, to construct a differentiable stack and perform a push:

```python
from StackNN.structs import Stack
stack = Stack(BATCH_SIZE, STACK_VECTOR_SIZE)
read_vectors = stack(value_vectors, pop_strengths, push_strengths)
```

For examples of more complex use cases of this library, refer to the [industrial-stacknns](https://github.com/viking-sudo-rm/industrial-stacknns) repository.

All the code in this repository is associated with the paper [Context-Free Transductions with Neural Stacks](https://arxiv.org/abs/1809.02836), which appeared at the Analyzing and Interpreting Neural Networks for NLP workshop at EMNLP 2018. Refer to our paper for more theoretical background on differentiable data structures.

## Running a demo

*Check [example.ipynb](example.ipynb) for the most up-to-date demo code.*

There are several experiment configurations pre-defined in [configs.py](configs.py). To train a model on one of these configs, do:

```shell
python run.py CONFIG_NAME
```

For example, to train a model on the string reversal task:

```shell
python run.py final_reverse_config
```

In addition to the experiment configuration argument, [run.py](run.py) takes several flags:
* `--model`: Model type (`BufferedModel` or `VanillaModel`)
* `--controller`: Controller type (`LinearSimpleStructController`, `LSTMSimpleStructController`, etc.)
* `--struct`: Struct type (`Stack`, `NullStruct`, etc.)
* `--savepath`: Path for saving a trained model
* `--loadpath`: Path for loading a model

## Documentation

You can find auto-generated documentation [here](https://stacknn.readthedocs.io/en/latest/index.html).

## Contributing

This project is managed by [Computational Linguistics at Yale](http://clay.yale.edu/). We welcome contributions from outside in the form of pull requests. Please report any bugs in the GitHub issues tracker. If you are a Yale student interested in joining our lab, please contact Bob Frank.

## Citations

If you use this codebase in your research, please cite the associated paper:

```
@inproceedings{hao-etal-2018-context,
    title = "Context-Free Transductions with Neural Stacks",
    author = "Hao, Yiding  and
      Merrill, William  and
      Angluin, Dana  and
      Frank, Robert  and
      Amsel, Noah  and
      Benz, Andrew  and
      Mendelsohn, Simon",
    booktitle = "Proceedings of the 2018 {EMNLP} Workshop {B}lackbox{NLP}: Analyzing and Interpreting Neural Networks for {NLP}",
    month = nov,
    year = "2018",
    address = "Brussels, Belgium",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/W18-5433",
    pages = "306--315",
    abstract = "This paper analyzes the behavior of stack-augmented recurrent neural network (RNN) models. Due to the architectural similarity between stack RNNs and pushdown transducers, we train stack RNN models on a number of tasks, including string reversal, context-free language modelling, and cumulative XOR evaluation. Examining the behavior of our networks, we show that stack-augmented RNNs can discover intuitive stack-based strategies for solving our tasks. However, stack RNNs are more difficult to train than classical architectures such as LSTMs. Rather than employ stack-based strategies, more complex stack-augmented networks often find approximate solutions by using the stack as unstructured memory.",
}
```

## Dependencies

The core implementation of the data structures is stable in Python 2 and 3. The specific tasks that we have implemented require Python 2.7. We use PyTorch version 0.4.1, with the following additional dependencies:

* `numpy`
* `scipy` (for data processing)
* `matplotlib` (for visualization)
* `nltk`

Using pip or conda should suffice for installing most of these dependencies. To get the right command for installing PyTorch, refer to the installation widget on the PyTorch website.

## Models

A model is a pairing of a controller network with a neural data structure. There are two kinds of models:

* `models.VanillaModel` is a simple controller-data structure network. This means there will be one step of computation per input.
* `models.BufferedModel` adds input and output buffers to the vanilla model. This allows the network to run for extra computation steps.

To use a model, call `model.forward()` on every input and `model.init_controller()` whenever you want to reset the stack between inputs. You can find example training logic in the `tasks` package.

## Data structures

* `structs.Stack` implements the differentiable stack data structure.
* `structs.Queue` implements the differentiable queue data structure.

The buffered models use read-only and write-only versions of the differentiable queue for their input and output buffers.

## Tasks

The `Task` class defines specific tasks that models can be trained on. Below are some formal language tasks that we have explored using stack models.

### String reversal

The `ReverseTask` trains a feed-forward controller network to do string reversal. The code generates 800 random binary strings which the network must reverse in a sequence-to-sequence fashion:

~~~
Input:   1 1 0 1 # # # #
Label:   # # # # 1 0 1 1
~~~

By 10 epochs, the model tends to achieve 100% accuracy. The config for this task is called `final_reverse_config`.

### Context-free language modelling

`CFGTask` can be used to train a context-free language model. Many interesting questions probing linguistic structure can be reduced to special cases of this general task. For example, the task can be used to model a language of balanced parentheses. The configuration for the parentheses task is `final_dyck_config`.

### Evaluation tasks

We also have a class for evaluation tasks. These are tasks where output i can be succintly expressed as some function of inputs 0, .., i. Some applications of this are evaluation of parity and reverse polish boolean formulae.

### Real datasets

The data folder contains several real datasets that the stack can be trained on. We should implement a task for reading in these datasets.
