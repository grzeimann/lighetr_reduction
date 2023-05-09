# LIGHETR Team HET Reduction Manual

## Initial TACC SETUP

### Step 1 - Get a TACC account
To execute reductions in near real-time for LIGHETR observations at the HET you will need a TACC account.  Email grzeimann@gmail.com for help.

### Step 2 - Set TACC Python Environment
```
ssh username@stampede2.tacc.utexas.edu
cd $HOME
ln -s $WORK work-stampede2
module load python3
pip3 install astropy --user
pip3 install seaborn --user
pip3 install specutils --user
pip3 install scikit-learn --user
pip3 install photutils=='0.7.2' --user
```

## VIRUS Reductions

To start VIRUS reductions we'll need an idev session

```
idev -N 1 -m 120 -p skx-dev
```

After creating the session, we can examine what observations have been taken and if our target is on TACC yet.

```
python /work/03730/gregz/maverick/Remedy/get_objects.py 20230509 virus
```

Finally, we can begin a reduction
```
python3 /work/03730/gregz/maverick/Remedy/quick_reduction.py 20230509 9 34 /work/03730/gregz/maverick/output/20230301_20230401.h5 -nd 3 -fp /work/03730/gregz/maverick/fplaneall.txt -nD -q -mc
```

## LRS2 Reductions

Once you have an account, you can go start visualization portal in a browser: https://vis.tacc.utexas.edu/jobs/

<p align="center">
  <img src="TACC_VIZ_portal.png" width="650"/>
</p>

Go to the utilities tab at the bottom and activate python3 for stampede2 (you should only need to do this once). Then, you can request a job as shown in the attached image (just click submit when you pull up the same left hand settings). After a small wait time, a new screen will show up and you will click connect.  Sometimes there are not enough nodes initially and you have to wait a bit longer. After you connect, you should be in your work directory, which will allow you to navigate to LRS2Multi/notebooks.  There will be a file called example_reduction.ipynb.  Open that notebook and follow the instructions to get started.
