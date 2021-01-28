# FilterPicker

Lomax, A., Satriano, C., & Vassallo, M. (2012). Automatic Picker Developments
and Optimization: FilterPicker-a Robust, Broadband Picker for Real-Time Seismic
Monitoring and Earthquake Early Warning. Seismological Research Letters, 83(3),
531–540, doi:[10.1785/gssrl.83.3.531](http://doi.org/10.1785/gssrl.83.3.531).

## Installation

Download software [(C version)](http://alomax.free.fr/FilterPicker/) and compile.

FilterPicker executable is called `picker_func_test`.


## Run

FilterPicker takes two arguments:

1. Input file in SAC format
2. Output file name 

```console
$ picker_func_test ES.CTAB..EHZ.D.2011.300 ES.CTAB..EHZ.D.2011.300.pick
```

The output file has the format of the location code [NonLinLoc](http://alomax.free.fr/nlloc/):

```
CTAB   -12345 EHZ  ? P0_    + 20110922 0013   15.7600 GAU 6.000e-02 0.000e+00 1.188e+01 6.400e-01
CTAB   -12345 EHZ  ? P1_    ? 20110922 0019   40.3100 GAU 1.000e-02 0.000e+00 1.874e+01 1.000e-02
CTAB   -12345 EHZ  ? P2_    - 20110922 0023   11.7600 GAU 5.000e-02 0.000e+00 1.151e+01 1.600e-01
CTAB   -12345 EHZ  ? P3_    - 20110922 0031   22.3400 GAU 4.000e-02 0.000e+00 1.120e+01 3.200e-01
CTAB   -12345 EHZ  ? P4_    + 20110922 0038   57.8799 GAU 8.000e-02 0.000e+00 1.030e+01 6.400e-01
CTAB   -12345 EHZ  ? P5_    + 20110922 0100   23.5599 GAU 2.000e-02 0.000e+00 1.005e+01 8.000e-02
CTAB   -12345 EHZ  ? P6_    ? 20110922 0112   26.7699 GAU 1.000e-02 0.000e+00 1.639e+01 8.000e-02
CTAB   -12345 EHZ  ? P7_    + 20110922 0121   24.9399 GAU 4.000e-02 0.000e+00 1.223e+01 1.600e-01
CTAB   -12345 EHZ  ? P8_    - 20110922 0134   19.6099 GAU 4.000e-02 0.000e+00 1.014e+01 1.600e-01
```

The meaning of the columns is:

- Station name: station name or code (char*6)
- Instrument: instument identification for the trace for which the time pick
  corresponds (i.e. SP, BRB, VBB) -12345 means `UNDEFINED` (char*4)
- Component: component identification for the trace for which the time pick
  corresponds (i.e. Z, N, E, H) (char *4)
- P phase onset (char*1)
description of P phase arrival onset; i, e
- Phase descriptor (char*6)
- Phase identification (i.e. P, S, PmP)
- First Motion: first motion direction of P arrival; c, C, u, U = compression;
  d, D = dilatation; +, -, Z, N; . or ? = not readable (char*1)
- Date (yyyymmdd) (int*6)
- Hour/minute (hhmm) (int*4)
- Seconds: seconds of phase arrival (float*7.4)
- Err: Error/uncertainty type; GAU (char*3)
- ErrMag: Error/uncertainty magnitude in seconds (expFloat*9.2)
- Coda duration: coda duration reading (expFloat*9.2)
- Amplitude: Maxumim peak-to-peak amplitude (expFloat*9.2)
- Period: Period of amplitude reading (expFloat*9.2)

```bash
#!/bin/bash

set -u # error if variable undefined
set -o pipefail

arcdir=/Lake/arclink

year=2017
day1=60
day2=299

net=XD

for ((day = $day1; day <= $day2; day++ )); do

    jday=$( printf "%03d" $day)

    daydir=$year.$jday
    mkdir $daydir
    cd $daydir
    echo ""
    echo ""
    echo "Processing $daydir"
    /bin/rm -f mseed.list
    find  $arcdir/$year/$net -name "$net.*.$year.$jday" -print > mseed.list
    while read -r msfile; do
        /bin/rm -f *.SAC sac.list
        echo ""
        filename=${msfile##*/}
        echo $filename
        mseed2sac $msfile

        /bin/ls -1 *.SAC > sac.list 2> /dev/null
        nsac=$( cat sac.list | wc -l )

        if [[ $nsac -eq 0 ]]; then
            echo "ERROR: no SAC files obtained from $filename"
        elif [[ $nsac -eq 1 ]]; then
            sacfile=$( head -1 sac.list )
            picker_func_test $sacfile $filename.pick
        else
            echo "WARNING: multiple SAC files for $filename"
            (( nfile = 1 ))
            for sacfile in *.SAC; do
                npts=$( saclhdr -NPTS $sacfile )
                if [[ $npts -gt 75000 ]]; then
                   picker_func_test $sacfile temp.$nfile.pick
                else
                   echo "WARNING: segment too small: $sacfile $npts"
                fi
                (( nfile++ ))
            done
            cat temp.[0-9]*.pick | sort -n -k7 -k8 -k9 > $filename.pick
            /bin/rm -f temp.[0-9]*.pick
        fi

        /bin/rm -f *.SAC sac.list
    done < mseed.list
    /bin/rm -f mseed.list
    cd ..

done
```

## Codes and results in Artemisa

The previous script `loop_lomax_picker.sh` is located in `~ific0331/scripts`

The FilterPicker executable `picker_func_test` is located in `~ific0331/bin`

The utility program `mseed2sac` is also located in `ific0331/bin`

The output of running FilterPicker for the El Hierro dataset is stored in
`/lustre/ific.uv.es/ml/ific033/data/Hierro/FilterPicker`.


