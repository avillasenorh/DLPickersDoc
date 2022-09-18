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

New `csv` output file now writes times in ISO 8601 format, so there is no need for conversion.

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
```


## Scripts for batch processing


### Data extraction

```bash
$ sfile_extract_all.sh sfile destination_directory
```


```
#!/bin/bash
#: Title       : se_sfile_wf_extract.sh
#: Purpose     : Reads a hypocenter defined in an S-file, extracts all the waveform
#+               data available in a SeisComP directory structure (SDS) and runs
#+               PhaseNet for the extracted miniSEED files.
#: Usage       : se_sfile_wf_extract.sh S-file SDS_dir
#: Date        : 2021-03-26
#: Author      : "Antonio Villasenor" <antonio.villasenor@csic.es>
#: Version     : 1.0
#: Requirements: awk
#+               GNU date command (not macOS date)
#+               dataselect (https://github.com/iris-edu/dataselect)
#+               run.py (PhaseNet prediction script)
#: IMPORTANT!! : if PhaseNet is installed using a Python virtual environment
#+               (venv or conda) it must be activated before running this script
#: Arguments   : S-file SDS-dir
#: Options     : none
set -euo pipefail

progname=${0##*/}

[[ $# -lt 2 ]] && { echo "usage: $progname sfile dest_dir"; exit 1; }

[[ ! -s $1 ]] && { echo "ERROR: S-file does not exist: $1"; exit 1; }
[[ ! -d $2 ]] && { echo "ERROR: destination directory does not exist: $2"; exit 1; }

sfile="$1"
dest_dir="$2"

if command -v date > /dev/null && date --version > /dev/null 2>&1; then
    DATE=date
elif command -v gdate > /dev/null && gdate --version > /dev/null 2>&1; then
    DATE=gdate
else
    echo "ERROR: no GNU date command in this system"
    exit 1
fi

[[ $( command -v dataselect ) ]] || { echo "ERROR: dataselect executable is missing"; exit 1; }

pre_event=30
post_event=210

origin_time=$(awk '/1$/ { \
	year =   1*substr($0,2,4); \
	month =  1*substr($0,7,2); \
	day =    1*substr($0,9,2); \
	hour =   1*substr($0,12,2); \
	minute = 1*substr($0,14,2); \
	second = 1*substr($0,17,4); \
	printf("%4d-%02d-%02dT%02d:%02d:%04.1f",year,month,day,hour,minute,second); \
}' $sfile)

year=${origin_time%%-*}
jday=$($DATE -u --date="$origin_time" +%j)
event_dir=$($DATE -u --date="$origin_time" +%Y.%j.%H%M%S)
out_dir=$dest_dir/$year/$event_dir
[[ -d $out_dir ]] && { echo "WARNING: event $sfile already processed in $out_dir"; exit 1; }

seconds=$($DATE -u --date="$origin_time" +%s)
start_time=$(bc -l <<< "$seconds - $pre_event")
end_time=$(bc -l <<< "$seconds + $post_event")

ts=$($DATE -u --date="@$start_time" +%Y,%j,%H,%M,%S,%N)
te=$($DATE -u --date="@$end_time" +%Y,%j,%H,%M,%S,%N)

echo " "
echo Processing $sfile
echo Origin time: $origin_time t_start: $ts t_end: $te

# /home/antonio/atlantico/RAW_DATA2/IBERIA 
#sds=( /tank/PYROPE /tank/IBERARRAY /tank/SISCAN /tank/MISTERIOS /home/antonio/atlantico/PROCESSED_DATA/IBERIA )
sds=( /tank/GEOMARGEN3 /home/antonio/atlantico/PROCESSED_DATA/IBERIA )

/bin/rm -f mseed.list
touch mseed.list

for datadir in "${sds[@]}"; do
    [[ -d $datadir/SDS/$year ]] && find $datadir/SDS/$year -name "*.[BEHS]?Z.?.$year.$jday" -print >> mseed.list
done

[[ ! -s mseed.list ]] && { echo "ERROR: no miniSEED files for $sfile"; /bin/rm -f mseed.list; exit 1; }

mkdir -p $out_dir
cp $sfile $out_dir/.
$DATE -u --date="@$start_time" +%Y-%m-%dT%H:%M:%S.%N > $out_dir/start_time.txt

start_string=$($DATE -u --date="@$start_time" +%Y-%m-%dT%H%M%SZ)
end_string=$($DATE -u --date="@$end_time" +%Y-%m-%dT%H%M%SZ)

while read -r zfile; do

   filename=${zfile##*/}        # miniSEED file withouth path: GS.CA06.00.HHZ.D.2019.003
   prefix=${filename::-12}      # GS.CA06.00.HH
   dum=${filename#*.}           # CA06.00.HHZ.D.2019.003
   sta=${dum%%.*}               # CA06
   zcmp=${zfile:(-14):5}        # HHZ
   ecmp=${zcmp/Z/E}             # HHE
   ncmp=${zcmp/Z/N}             # HHN
   h1=${zfile//$zcmp/$ecmp}     # /full_path/GS.CA06.00.HHE.D.2019.003
   h2=${zfile//$zcmp/$ncmp}     # /full_path/GS.CA06.00.HHN.D.2019.003

   [[ -s $zfile ]] && dataselect -ts $ts -te $te -lso -Pe -o $out_dir/${prefix}Z__${start_string}__${end_string}.mseed $zfile
   [[ -s $h1 ]]    && dataselect -ts $ts -te $te -lso -Pe -o $out_dir/${prefix}E__${start_string}__${end_string}.mseed $h1
   [[ -s $h2 ]]    && dataselect -ts $ts -te $te -lso -Pe -o $out_dir/${prefix}N__${start_string}__${end_string}.mseed $h2

done < mseed.list

/bin/mv mseed.list $out_dir/.

#nfiles=$( cat $out_dir/fname.csv | wc -l )
#[[ $nfiles -lt 2 ]] && { echo "ERROR: no data for $sfile"; /bin/rm -rf $out_dir; exit 1; }
```

### Run phasenet for all miniSEED files in a directory

```bash
$ event_pn_pick_new.sh $event_directory $destination_directory
```

```
#!/bin/bash
#: Title       : se_sfile_wf_extract.sh
#: Purpose     : Reads a hypocenter defined in an S-file, extracts all the waveform
#+               data available in a SeisComP directory structure (SDS) and runs
#+               PhaseNet for the extracted miniSEED files.
#: Usage       : se_sfile_wf_extract.sh S-file SDS_dir
#: Date        : 2021-03-26
#: Author      : "Antonio Villasenor" <antonio.villasenor@csic.es>
#: Version     : 1.0
#: Requirements: awk
#+               GNU date command (not macOS date)
#+               dataselect (https://github.com/iris-edu/dataselect)
#+               run.py (PhaseNet prediction script)
#: IMPORTANT!! : if PhaseNet is installed using a Python virtual environment
#+               (venv or conda) it must be activated before running this script
#: Arguments   : S-file SDS-dir
#: Options     : none
set -euo pipefail

progname=${0##*/}

[[ $# -lt 2 ]] && { echo "usage: $progname event_dir dest_dir"; exit 1; }

[[ ! -d $1 ]] && { echo "ERROR: event directory does not exist: $1"; exit 1; }
[[ ! -d $2 ]] && { echo "ERROR: destination directory does not exist: $2"; exit 1; }

datadir="$1"
destdir="$2"
eventdir=${datadir##*/extract/}

if command -v date > /dev/null && date --version > /dev/null 2>&1; then
    DATE=date
elif command -v gdate > /dev/null && gdate --version > /dev/null 2>&1; then
    DATE=gdate
else
    echo "ERROR: no GNU date command in this system"
    exit 1
fi

pn_dir=${HOME}/repos/PhaseNet
model=${pn_dir}/model/190703-214543

[[ ! -d $pn_dir ]] && { echo "ERROR: invalid directory for PhaseNet repo: $pn_dir"; exit 1; }
[[ ! -d $model ]] && { echo "ERROR: PhaseNet model does not exist: $model"; exit 1; }

# ES.ELOR.00.HHZ__2020-12-26T121519Z__2020-12-26T121919Z.mseed
/bin/ls -1 $datadir/*__*Z__*Z.mseed > mseed.list.$$
nfiles=$( cat mseed.list.$$ | wc -l )

[[ $nfiles -eq 0 ]] && { echo "ERROR: no miniSEED files in $datadir"; /bin/rm -f mseed.list.$$; exit 1; }

awk -F/ '{print $NF}' mseed.list.$$ | awk -F_ '{print substr($1,1,length($1)-1)}' - | sort -u > streams.list.$$

curdir=$PWD
outdir=$destdir/$eventdir
[[ -d $outdir ]] && { echo "Output event directory already exists: $outdir"; /bin/rm -f *.list.$$; exit 1; }
mkdir -p $outdir
cp $datadir/*L.S?????? $outdir/.
echo "fname,E,N,Z" > $outdir/fname.csv

while read -r stream; do

    /bin/rm -f tmp.$$
    grep "${stream}[ENZ12]" mseed.list.$$ > tmp.$$
    n_channels=$( cat tmp.$$ | wc -l )

	if [[ $n_channels -eq 3 ]]; then

        printf "%s" $stream.mseed >> $outdir/fname.csv
        while read -r cmpfile; do

            cat $cmpfile >> $outdir/$stream.mseed

            fname=${cmpfile##*/}     # ES.ERTA..HHE__2005-02-15T014307Z__2005-02-15T014707Z.mseed ES.ERTA..HHE
            cmpname=${fname%%__*}    # ES.ERTA..HHE
            cmp=${cmpname##*.}       # HHE
            printf ",%s" $cmp >> $outdir/fname.csv

        done < tmp.$$
        printf "\n" >> $outdir/fname.csv

    elif [[ $n_channels -eq 1 ]]; then
        echo only one component for $stream $n_channels
        /bin/rm -f tmp.$$
        continue
    else
        echo unusual number of components for $stream : $n_channels
        /bin/rm -f tmp.$$
        continue
    fi
    /bin/rm -f tmp.$$

done < streams.list.$$

/bin/rm -f mseed.list.$$ streams.list.$$

nfiles=$( cat $outdir/fname.csv | wc -l )
[[ $nfiles -lt 2 ]] && { echo "ERROR: no stations for $eventdir"; /bin/rm -rf $outdir; exit 1; }

cd $pn_dir
#timeout 10m \
python phasenet/predict.py --model=model/190703-214543 \
                           --data_list=${outdir}/fname.csv \
                           --data_dir=${outdir} \
                           --result_dir=${outdir}/phasenet \
                           --format=mseed --plot_figure > $outdir/phasenet.log 2>&1

#cd $outdir
#pnout2picks.sh

cd $curdir
```


