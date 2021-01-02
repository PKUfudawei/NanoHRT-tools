# NanoHRT-tools

### Set up CMSSW and offcial NanoAOD-tools

```bash
cmsrel CMSSW_11_1_0_pre5
cd CMSSW_11_1_0_pre5/src
cmsenv

git clone https://github.com/cms-nanoAOD/nanoAOD-tools.git PhysicsTools/NanoAODTools
```

**Alternatively, set up CMSSW and offcial NanoAOD-tools with Python3 support. This is needed to re-run the taggers w/ `ONNXRuntime`.**

```bash
cmsrel CMSSW_11_1_0_pre5_PY3
cd CMSSW_11_1_0_pre5_PY3/src
cmsenv

git clone https://github.com/hqucms/nanoAOD-tools.git PhysicsTools/NanoAODTools
```

### Get customized NanoAOD tools for HeavyResTagging (NanoHRT-tools)

```bash
git clone https://github.com/hqucms/NanoHRT-tools.git PhysicsTools/NanoHRTTools -b dev/unify-producer
```

### Compile

```bash
scram b -j8
```

### Test

Instructions to run the nanoAOD postprocessor can be found at [nanoAOD-tools](https://github.com/cms-nanoAOD/nanoAOD-tools#nanoaod-tools). 

### Production

```bash
cd PhysicsTools/NanoHRTTools/run
```

##### Make trees for MC performance study:

```bash
python runPostProcessing.py [-i /path/of/input] -o /path/to/output -d datasets.yaml --friend 
-I PhysicsTools.NanoHRTTools.producers.hrtMCTreeProducer hrtMCTree -n 1
```

To merge the trees, run the same command but add `--post -w ''` (i.e., set `-w` to an empty string (`''`) -- we do not add the cross sections, but simply reweight signals to match the QCD spectrum afterwards).


##### Make trees for heavy flavour tagging (bb/cc) or top/W data/MC comparison and scale factor measurement:

```bash
python runHeavyFlavTrees.py -i /eos/uscms/store/user/lpcjme/noreplica/NanoHRT/path/to/input -o /path/to/output 
(--sample-dir custom_samples) --jet-type [ak8,ak15] --channel [photon|qcd|muon|inclusive] --year [2016|2017|2018] -n 10 
(--batch) (--run-data) (--run-syst)
(--run-tagger) (--run-mass-regression) (--sfbdt 0.5)
(--condor-extras '+AccountingGroup = "group_u_CMST3.all"')
```

Command line options:

  - the preselection for each channel is coded in `runHRTTrees.py`
  - add `--run-data` to make data trees
  - add `--run-syst` to make the systematic trees
  - can run data & MC for multiple years together w/ e.g., `--year 2016,2017,2018`. The `--run-data` option will be ignored in this case. Add also `--run-syst` to make the systematic trees.
  - use `--sample-dir` to specify the directory containing the sample lists. Currently we maintain two sets of sample lists: the default one is under [samples](run/samples) which is used for running over official NanoAOD datasets remotely, and the other one is [custom_samples](run/custom_samples) which is used for running over privately produced NanoAOD datasets locally. To run over the private produced samples, ones needs to add `--sample-dir custom_samples` to the command line.
  - the `--batch` option will submit jobs to condor automatically without confirmation
  - remove `-i` to run over remote files (e.g., official NanoAOD, or private NanoAOD published on DAS); consider adding `--prefetch` to copy files first before running
  - **[NEW]** add `--run-tagger` (`--run-mass-regression`) to run new ParticleNet tagger (mass regression) on-the-fly. Check `HeavyFlavBaseProducer.py` for the model configuration.
  - **[NEW]** use `--sfbdt` to change the sfBDT cut value. This affects only QCD and photon samples. By default, sfBDT > 0.5 is applied to QCD and photon samples.
  - **[NEW]** use `--condor-extras` to pass extra options to condor job description file.
     
More options of `runPostProcessing.py` or `runHRTTrees.py` (a wrapper of `runPostProcessing.py`) can be found with `python runPostProcessing.py -h` or `python runHRTTrees.py -h`, e.g.,

 - To resubmit failed jobs, run the same command but add `--resubmit`.

 - To add cross section weights and merge output trees according to the config file, run the same command but add `--post`. The cross section file to use can be set with the `-w` option. 

