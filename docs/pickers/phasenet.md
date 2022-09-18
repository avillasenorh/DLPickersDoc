# PhaseNet

Zhu, W., & Beroza, G. C. (2019). PhaseNet: A Deep-Neural-Network-Based Seismic
Arrival Time Picking Method. Geophysical Journal International, 216(1), 261–273,
doi:[10.1093/gji/ggy423](https://academic.oup.com/gji/article/216/1/261/5129142).

New version added since original repo was downloaded.

## Installation

Requirements


New version (tried on 2022-09-18):

```bash
$ git clone https://github.com/wayneweiqiang/PhaseNet.git
$ cd PhaseNet
$ conda env create -f env.yml
$ conda activate phasenet
```


## Run

How to run in Linux/macOS.

How to run on Artemisa

### Run the new version

Here we assume that we have installed PhaseNet in the directory `${HOME}/repos/PhaseNet`
and we want to run the predictions for the miniSEED data in `${HOME}/repos/PhaseNet/demo`.

First we need to create a `csv` file with the list of miniSEED files to process:

```
fname,E,N,Z
CCC.mseed,HHE,HHN,HHZ
...
```

To run the test data in the directory `demo`:

```
$ cd
$ cd repos/PhaseNet
$ python phasenet/predict.py --model=model/190703-214543 --data_list=demo/fname.csv \
         --data_dir=demo/mseed --result_dir=demo/results --format=mseed --plot_figure
```

This will produce the output in the directory `${HOME}/repos/PhaseNet/demo/results`, 

## Old output

Old version outputs a `csv` and a `pkl` (pickle) file. The format of the csv file is:

```
fname,itp,tp_prob,its,ts_prob
CA.CBEU..HH.mseed_0,[],[],[],[]
CA.AVIN..HN.mseed_0,[],[],[],[]
CA.CBRU..HH.mseed_0,[],[],[],[]
CA.BAIN..HN.mseed_0,[],[],[],[]
CA.BLAN..HN.mseed_0,[],[],[],[]
CA.BAJU..HN.mseed_0,[],[],[],[]
CA.CAVN..HH.mseed_0,[],[],[],[]
CA.CBUD..HH.mseed_0,[],[],[],[]
CA.CBEU..HH.mseed_3000,[],[],[],[]
CA.AVIN..HN.mseed_3000,[2072],[0.36653],[],[]
CA.BLAN..HN.mseed_3000,[],[],[],[]
CA.BAIN..HN.mseed_3000,[],[],[],[]
CA.BAJU..HN.mseed_3000,[],[],[],[]
CA.CAVN..HH.mseed_3000,[2284],[0.884308],[],[]
CA.CBUD..HH.mseed_3000,[251],[0.78194],[530],[0.934841]
CA.CBRU..HH.mseed_3000,[],[],[],[]
CA.BLAN..HN.mseed_6000,[863],[0.733137],[],[]
...
```

This format can not be used and is re-written to the so-called `pick` format:

```
CA.CMAS..HH P 2020-10-19T19:53:09.600000000 0.94159900 1
CA.CMAS..HH S 2020-10-19T19:53:19.370000000 0.69820100 1
```

Also `markers` format to plot with `snuffler`:

```
# Snuffler Markers File Version 0.2
phase: 2020-10-19 19:53:09.600000000   0   CA.CMAS..HHZ  None   None   None   P            None   False
phase: 2020-10-19 19:53:19.370000000   0   CA.CMAS..HHN  None   None   None   S            None   False
```

## New output

```
file_name,begin_time,station_id,phase_index,phase_time,phase_score,phase_type
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,18021,2019-07-04T17:03:00.208,0.956,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,24742,2019-07-04T17:04:07.418,0.838,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,30658,2019-07-04T17:05:06.578,0.78,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,35298,2019-07-04T17:05:52.978,0.442,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,38846,2019-07-04T17:06:28.458,0.493,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,38900,2019-07-04T17:06:28.998,0.435,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,51138,2019-07-04T17:08:31.378,0.971,P
...
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,2516038,2019-07-04T23:59:20.378,0.37,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,2516598,2019-07-04T23:59:25.978,0.961,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,2519071,2019-07-04T23:59:50.708,0.838,P
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,18412,2019-07-04T17:03:04.118,0.927,S
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,25121,2019-07-04T17:04:11.208,0.629,S
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,31022,2019-07-04T17:05:10.218,0.309,S
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,31401,2019-07-04T17:05:14.008,0.815,S
CCC.mseed,2019-07-04T16:59:59.998,CCC.mseed,35681,2019-07-04T17:05:56.808,0.537,S
...
