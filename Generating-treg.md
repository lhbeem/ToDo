## Description

`treg` is time-registered data. 'treg' is a key product for generating time stamped profiles ('tpro').  It consists of PST organized transects with classes named for the position source, with data stored as ztim files.  Treg is generated from norm.  The hg repository for treg can be found at `$WAIS/syst/linux/src/treg`.

## Expected inputs

- Data in [norm](Generating-Norm-(Level1B))  (specifically what data??)
- The [XPED].params file (which contains projection and leap second information) in the `$WAIS/code/treg/[XPED]/params` directory
- the utility list_season is funtional which requires the expeditions [meta data to be complete](Updating-meta-data).
- is pik1 a dependency? 

## Set up code

Check out a working copy of the treg code for this season.

```
hg clone $WAIS/syst/linux/src/treg $WAIS/code/treg/[XPED]
```

If hg is unavailable, ask for help making it work. It is strongly discouraged to copy a similar directory in `$WAIS/syst/linux/src/treg` to `$WAIS/code/treg/[XPED]`, because this leads to undesired branching in the version history. -- GNG


### Edit code
Within `$WAIS/code/treg/[XPED]/GPS/make_gps.sh` verify "set" (of PST) search keyword is present within for loop near the end of code. If not, add it. The search keyword is used within grep so to have it be as general as possible (e.g. JKB2 for JKB2c or IBH for all IBH sets). 

Within `$WAIS/code/treg/[XPED]/CMP/make_gps.sh` (the code here might be named `make_pik1.sh`) verify the search keyword for the platform "Set" (e.g. JKB2, GCX0) is present within for loop near the end of code. If not, add it. The search keyword is used within grep so to have it be as general as possible (e.g. JKB2 for JKB2c). 



## Set up params file

In `$WAIS/code/treg/[XPED]/params` make a file named [XPED].params. It will contain three lines 

1. projection (Determine which projection to use, specified using peony shortcut name. Antarctic data typically uses `ps71s` = EPSG:3031 (Polar Stereographic 71S). Arctic seasons, including SRH and GOG, use `gis` = EPSG:3413 (NSIDC Sea Ice Polar Stereographic North).)
2. platform
3. [leap seconds](https://github.com/UTIG/DataReleaseTasking/wiki/GNSS-Expedition-Parameters#utc-gps-leap-seconds) (notice in the params file that the values are negative).

e.g.

```bash
prj=ps71s
PLATFORM=JKB
leap_seconds=-18
```
## Generate GPS treg

GPS treg is real time code-phase positions from ELSA.

To run all with PST list supplied by list_season utility

```
cd $WAIS/code/treg/[XPED]/GPS
./make_gps.sh
```


or to run specific PSTs

```
cd $WAIS/code/treg/[XPED]/GPS
./make_gps.sh [PST1] [PST2] ...
```

1 second ztim files are generated with longitude, latitude and elevation in WGS-84, as well as projected coordinates in either EPSG:3031 (Polar Stereographic 71S) or EPSG:3413 (Arctic Polar Stereographic 70N). 

## Generate PIK1 treg

PIK1 treg is real time code-phase positions from ELSA matched to pik1 radar traces.

(Does pik1 generation have to occur prior to this step?)

To make all psts supplied by list_season utility. The code name might vary between expeditions `make_pik1.sh` or `make_gps.sh`

```
cd $WAIS/code/treg/[XPED]/CMP
./make_pik1.sh
```


or to run specific PSTs

```
cd $WAIS/code/treg/[XPED]/CMP
./make_pik1.sh [PST1] [PST2] ...
```

Approximately 4 Hz ztim files are generated with longitude, latitude and elevation in WGS-84, as well as projected coordinates in either EPSG:3031 (Polar Stereographic 71S) or EPSG:3413 (Arctic Polar Stereographic 70N). 


## Generate POS treg

To generate POS treg (PPP solutions using wpt2), run all PSTs as supplied by list_season utility

```
cd $WAIS/code/treg/[XPED]/POS/wpt2
./make_treg
```

or to run specific PSTs

```
cd $WAIS/code/treg/[XPED]/POS/wpt2
./make_treg [PST1] [PST2] ...
```

2 Hz ztim files are generated with longitude, latitude and elevation in WGS-84, as well as projected coordinates in either EPSG:3031 (Polar Stereographic 71S) or EPSG:3413 (Arctic Polar Stereographic 70N).
Uncertainties are included. (<- ??? I think I meant I'm not sure if these directions are complete/accurate)

## Generate TRJ0

TRJ is loosely coupled IMU solutions integrating PPP from wpt1

To run all PSTs:

```
cd $WAIS/code/treg/[XPED]/TRJ
./combine_span.sh
```

50 Hz ztim files are generated with longitude, latitude and elevation in WGS-84, roll, pitch and heading, vertical standard deviation and north, west and vertical accelerations; as well as projected coordinates in either EPSG:3031 (Polar Stereographic 71S) or EPSG:3413 (Arctic Polar Stereographic 70N).


## To generate TRJ1 (combining available real time AVN and PPP from wpt2):

To run all PSTs

```
cd $WAIS/code/treg/[XPED]/TRJ
./combine_avn_pos.sh
```

or to run specific PSTs

```
cd $WAIS/code/treg/[XPED]/TRJ
./combine_avn_pos.sh [PST1] [PST2] ...
```

10 Hz ztim files are generated with longitude, latitude and elevation in WGS-84, roll, pitch and heading, vertical standard deviation and north, west and vertical accelerations; as well as projected coordinates in either EPSG:3031 (Polar Stereographic 71S) or EPSG:3413 (Arctic Polar Stereographic 70N).  Caution must be used with this product, as the real time AVN from either the MMQ-50 on JKB or the internal INS on GCX are unreliable.


## Verification 

- Files located in `$WAIS/targ/treg/PST/PIK1_[platform]a/`
- treg files are binary, but can be converted to ascii and viewed using the `zvert` utility (e.g. `zvert ztim_xyzrphsaaa.bin | head` will display the first few lines of a trajectory file in projected coordinates) 

