# PickNet

Wang, J., Xiao, Z., Liu, C., Zhao, D., & Yao, Z. (2019). Deep Learning for
Picking Seismic Arrival Times. Journal of Geophysical Research: Solid Earth,
124(7), 6612–6624, doi:[10.1029/2019JB017536](https://agupubs.onlinelibrary.wiley.com/doi/abs/10.1029/2019JB017536).


## Installation

According to the manual, the installation can be done with conda:

```console
$ conda create -n picknet python=3.6 tensorflow=1.8 cudatoolkit=X.X pyyaml matplotlib
```

However, this produces an error. We have tried this alternative:

```console
$ conda create -n picknet python=3.6
$ conda activate picknet
$ conda install -c conda-forge obspy       # this also installs matplotlib
$ conda install pyyaml
$ conda install pyproj                     # needed for gen_input_for_pick.py
$ pip install tensorflow==1.8

$ conda install numpy==1.16.2              # gen_tomo_arrivals_inputfile_p_s.py does not work with 1.19.5
```

We did not include the `cudatoolkit` option because we are testing the code in a
computer without a NVIDA GPU.

## Download waveform data files

The first step in the tutorial is to download waveform data from IRIS. For this we need
a list of events to extract (catalog), a list of stations, and a way to request the waveform
data to the data center. This is done with the script `download_data.py` located in the
directory `PickingDataFromIRISExample`:

```console
$ cd PickingDataFromIRISExample
$ python download_data.py
```

**NOTE:** directory `ori_data/events` must exist before running the script, but can be
empty (`catalog.xml` is created by the script). The other directories `ori_data/stations` and
`ori_data/waveforms` are created by the script.

Running this script produces the following files:

- `ori_data/events/catalog.xml`: catalog file in [QuakeML](https://quake.ethz.ch/quakeml/) format.
- `ori_data/stations/NET.STA.xml`: station files in [StationXML](https://www.fdsn.org/xml/station/) format.
   `NET` indicates the network code, and `STA` the station code (e.g. `BK.YBH.xml`).
- `ori_data/waveforms/En/*.mseed`: waveform files in miniSEED format. `En` is the directory
   that contains the waveform data for event number `n` in `catalog.xml`. The waveform files `*.mseed`
   have names such as: `UW.BST17..HHZ__20190623T223810Z__20190623T224910Z.mseed`

The event number `n` starts with 0. Running this script (as of 2021-01-22), it
obtains only 5 events, with the corresponding event directories `E0` to `E4`. 
Apparently when the script was created, it obtained 6 events. This will produce
a problem in the script of the following step because the number of events it expects (6) is
hard-wired in the code.

There seems to be an additional off-by-one logic problem in the scripts, because some output files
contain the prefix `E0_E6_` indicating that there are 7 events, from index 0 to 6, while in reality
the test data contains 6 events, with indices from 0 to 5 (or 5 event with indices from 0 to 4
in we re-run the data extraction script in 2021).

If we change the value of `end_e` to 5, then the script runs correctly, and generates the 
preprocessed files in directory `input_output`, some with prefixes `E0_E5_` (which should be `E0_E4_`).

## Waveform pre-processing

The next step is to prepare the extracted raw waveforms for picking. This pre-processing
consists of 3 steps:

1. Rotate the horizontal components to radial and transverse
2. Cut the waveforms to 12 s windowns centered on the theoretical P wave arrival
   for the vertical component and 16 s windows centered on the theoretical S wave arrival
   for the radial and transverse components.
3. Remove the mean and normalize by the absolute value of the maximum amplitude.

(The order of these steps is not clear, need to check the source code) 

```console
$ python gen_input_for_pick.py
```
This script generates its output files in the directory `input_output`.

First it generates the P and S time windows and writes them in `NumPy` binary format,
in the directories `P_slices` and `S_slices` respectively, with a subdirectory
for each event:

- `P_slices/E?/input.npy`
- `P_slices/E?/sta_info.npy`
- `S_slices/E?/input.npy`
- `S_slices/E?/sta_info.npy`

Then it combines the data from all the events into a single `NumPy` binary file, located
in the directories `P_input` and `S_input`:

- `P_input/E0_E6_info.npy`
- `P_input/E0_E6_input.npy`
- `S_input/E0_E6_RT_info.npy`
- `S_input/E0_E6_RT_input.npy`

As mentioned before, if we try to run this script with data extracted in 2021 it crashes.
The problem is that it expects 6 event directories (from `E0` to `E5`),
and the download script `download_data.py` only produced 5 event directorioes
(from `E0` to `E4`).

To make the script run correctly with 5 events, we need to edit line 406 in
`gen_input_for_pick.py` and set the variable `end_e` to 5.

```python
if __name__ == '__main__':
    print('START PROCESSING')
    start_e = 0
    end_e = 5    # originally set to 6
```
## Picking

In order to run the phase picking, two parameter files are needed inside the directory
`input_output`: `P_config.yaml` and `S_config.yaml`.

These files contains parameters for the training, etc. For the picking, the most relevant
difference between both files is the trained model to use. Here is the contents of `P_config.yaml`:

```
# location of snapshot and tensorbaord summary events
save_dir: fcn/trained_models/P_wave/
# training batch size, decide with your GPU size
batch_size_train: 20
# validation batch size, ran every val_interval
batch_size_val: 20
# test batch size
batch_size_test: 2000
# split training data for trainig/validation
train_split: 0.95
# maximum iterations to run epoc == 30k/batch_size
max_iterations: 800005
# optimizer params (not used currently Adam is used by default)
optimizer: 'adam'
optimizer_params:
    learning_rate: 0.001
    weight_decay: 0.0002
# Loss for layer fusion
loss_weights: 1.0
# save snapshot every save_interval iterations
save_interval: 1000
# validate on held out dataset
val_interval: 1000
# print loss every print_interval
print_interval: 100
# learning rate decay (Not used with Adam currently)
learning_rate_decay: 0.1
# Apply weighted_cross_entropy_loss to outputs from each side layer
# Setting to false only loss after last conv layer is computed
deep_supervision: True
# Targets are continous if True else binary {0, 1}
target_regression: True

training:
    filename: NOT_FOR_TRAINING
    #
    length_before: 400
    data_width: 1200
    data_height: 1
    n_channels: 1
    trace_per_collect: 4
    rand_dev: 300
# testing data
testing:
    filename: PickingDataFromIRISExample/input_output/P_input/E0_E6_input
    data_width: 1200
    data_height: 1
    n_channels: 1
    limit_left: 300
    limit_right: 900 
# use snapshot after test_snapshot intervals for testing
test_snapshot: 330000
# Apply testing_threshold after relu
testing_threshold: 0.0
#available choices: HED, SRN, PickNet
using_model: PickNet
#block1 conv size
b1_convh: 1
b1_convw: 32
#block2 conv size
b2_convh: 1
b2_convw: 16
#block3 conv size
b3_convh: 1
b3_convw: 8
#block4 conv size
b4_convh: 1
b4_convw: 4
#block5 conv size
b5_convh: 1
b5_convw: 2
```
The parameter `testing: filename` contains the `E0_E6_input` prefix. If we pick a different number of
events, this parameter has to be changed.

To run the picking go to the parent directory of the repository and type:

```console
$ cd ..

$ python seismic_pick_run.py --test \
  --config-file PickingDataFromIRISExample/input_output/P_config.yaml # for P waves

$ python seismic_pick_run.py --test \
  --config-file PickingDataFromIRISExample/input_output/S_config.yaml # for S waves
```

These scripts will produce the following files inside the directory `PickingDataFromIRISExample`:

For P waves:

- `input_output/P_input/E0_E6_input_fuse_picks.npy`
- `input_output/P_input/E0_E6_input_output.npy`

For S waves:

- `input_output/S_input/E0_E6_RT_input_fuse_picks.npy`
- `input_output/S_input/E0_E6_RT_input_output.npy`


## Post-processing

The output of the picking script `seismic_pick_run.py` are files in `NumPy` binary format. To obtain
the picks in readable form and see plots of the results we need to run additional scripts.
First we need to run:

```console
$ cd PickingDataFromIRISExample
$ python ViewAndGatherP.py input_output/P_input E0_E6_ 1         # for P waves
$ python ViewAndGatherS.py input_output/S_input E0_E6_RT_ 1      # for S waves
```

These two scripts produce the following output files.

For P waves:

- `P_input/E0_E6_msta_input.npy`
- `P_input/E0_E6_View/En.png` : png files for each event n

For S waves:

- `S_input/E0_E6_RT_msta_input.npy`
- `S_input/E0_E6_RT_View/En.png` : png files for each event n

Except for the `png`files, the results are in `NumPy` binary format and therefore
not yet in human readable format. In order to convert the output to text format we
have to run the following script:

```console
$ python gen_tomo_arrivals_inputfile_p_s.py 'input_output/P_input/' 'input_output/S_input/' 'txt_pickfile/' IRIS
```

This script will create a directory named `txt_pickfile` with
the following files:

- `txt_pickfile/IRISdata_code_dict.npy`
- `txt_pickfile/IRISpicks.txt`

However, the script in its original form does not work, giving the following error:

```
  File "gen_tomo_arrivals_inputfile_p_s.py", line 34, in <module>
    picks_dict = np.load(str(pickfile))[()]
  File "/Users/antonio/opt/anaconda3/envs/picknet/lib/python3.6/site-packages/numpy/lib/npyio.py", line 440, in load
    pickle_kwargs=pickle_kwargs)
  File "/Users/antonio/opt/anaconda3/envs/picknet/lib/python3.6/site-packages/numpy/lib/format.py", line 727, in read_array
    raise ValueError("Object arrays cannot be loaded when "
ValueError: Object arrays cannot be loaded when allow_pickle=False
```

This error occurs in the following line:

```python
    for pickfile in picks_dir.glob('*_msta_input.npy'):
        picks_dict = np.load(str(pickfile))[()]
```

This seems to be related to the `NumPy` version. The version originally installed is 1.19.5.
Downgrading `NumPy` to 1.16.2 (`conda install numpy==1.16.2`) fixes the issue.
This also eliminates most of the warnings produced when running `seismic_pick_run.py`.

One final step is to generate the station list:

```console
$ python gen_tomo_stations_inputfile.py 'ori_data/stations/' 'txt_pickfile/'
```
This script produces the following output:

- `txt_pickfile/stations.txt`

