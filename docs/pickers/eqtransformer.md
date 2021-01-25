# EQTransformer

Mousavi, S. M., Ellsworth, W. L., Zhu, W., Chuang, L. Y., & Beroza, G. C. (2020).
Earthquake transformer—an attentive deep-learning model for simultaneous earthquake
detection and phase picking. Nat. Commun., 11(1), 3952,
doi:[10.1038/s41467-020-17591-w](http://www.nature.com/articles/s41467-020-17591-w).

## Installation

The installation instructions for `EQTransformer` are found
[here](https://eqtransformer.readthedocs.io/en/latest/installation.html).

The installation in Artemisa was done using `venv`:

```console
$ git clone -b master https://git.csic.es/seismicai/eqtransformer.git
$ cd eqtransformer

$ python3 -m venv .venv
$ source .venv/bin/activate
$ pip install --upgrade pip
$ pip install tensorflow "h5py<3.0.0"
$ pip install EQTransformer -U
```

When installing `TensorFlow` it is important that the version of the
required package `h5py` is smaller than 3. Higher versions are not
compatible with `EQTransformer`.

## Run

How to run in Linux/macOS.


How to run on Artemisa
## Output

Output files
