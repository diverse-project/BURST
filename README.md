# BURST a benchmarking platform for uniform random sampling techniques

BURST provides Python scripts and Docker environment for evaluating state-of-the-art feature model and SAT samplers (KUS, SPUR, Unigen, etc.) as well as proven statistical test (Barbarik). 
BURST comes with an extensive --- and extensible --- benchmark dataset comprising 500+ SAT formuale and feature models, including challenging, real-world models of the Linux kernel. A demonstration of the tool is available: 
[![Alt
text](https://img.youtube.com/vi/sSKosyrfitA/0.jpg](https://www.youtube.com/watch?v=sSKosyrfitA)

## Usage 

You only need to pull the Docker image `macher/usampling:squashed` that contains both scripts and benchmarks. 

To verify that your installation is correct, please run the following command: 
`docker run macher/usampling:squashed /bin/bash -c 'cd /home/usampling-exp/; echo STARTING; python3 barbarikloop.py -flas /home/samplingfm/Benchmarks/FeatureModels/FM-3.6.1-refined.cnf --ref-sampler 6 --sampler 2 --seed 1 --timeout 50000; echo END'` 

A message with `NOT UNIFORM` should appear at the end. 
The sampler `QuickSampler` has been used over the formula `/home/samplingfm/Benchmarks/FeatureModels/FM-3.6.1-refined.cnf` 

In addition to pull the Docker image, we recommand to clone this repo. Like this, you can benefit from the most recent updates (or simply edit the scripts) by mounting the repo within the Docker image.
For instance, you can interactively used the image `docker run -it -v $(pwd):/home/usampling-exp:z macher/usampling:squashed /bin/bash` 
and the scripts are located in `/home/usampling-exp/` 


### Usage (Sampling)

The previous example was to check uniformity. 
You can also generate samples with some samplers.
For instance:

`docker run -v $(pwd):/home/usampling-exp:z macher/usampling:squashed /bin/bash -c 'cd /home/usampling-exp/; echo STARTING; python3 usampling-experiments.py -flas /home/samplingfm/Benchmarks/Blasted_Real/blasted_case141.cnf /home/samplingfm/Benchmarks/Blasted_Real/blasted_case142.cnf --spur -t 1; echo END'`

is calling SPUR sampler, with a timeout of 1 second, and with formulas explicitly given (here two formulas: useful to focus on specific formulas). 
You can also specify a folder.
Without `flas` default formulas contained in the Docker folder/subfolders `/home/samplingfm/` are processed (around 500 files).

Typical outcomes are:

```
cat usampling-data/experiments-KUS.csv
formula_file,timeout,execution_time_in,dnnf_time,sampling_time,model_count,counting_time,dnnfparsing_time
/home/samplingfm/Benchmarks/FeatureModels/FM-3.6.1-refined.cnf,False, 0.1399824619293213, 0.011404275894165039, 0.0007951259613037109, 26256, 0.0008006095886230469, 0.0012776851654052734
```

The meaning of columns of the CSV file is as follows: `formula_file': the name of the processed model; `timeout': whether a timeout has been reached or not; `execution_time_in' overall execution time; `dnnf_time': the time required by KUS to compile the corresponding DNNF formula; `sampling_time': time taken to produce the samples; `model_count': the number of solutions in the model; `counting\_time': time taken to count solutions; `dnnfparsing_time': time taken to parse the compiled DNNF formula. All times are reported in seconds.

### Usage (Uniformity)

We assess uniformity with Barbarik (https://github.com/meelgroup/barbarik).  To compute uniformity for a set of models, inside the Docker: `python3 barbarikloop.py -flas gilles --sampler 10  --seed 1 --timeout 60` where sampler is the sampler to be assessed (1=Unigen, 2=QuickSampler, 3=STS, 4=CMS, 5=UniGen3, 6=SPUR, 7=SMARCH, 8=UniGen2,9=KUS, 10=Distance-based Sampling), seed an integer seed and a timeout in seconds. it supports all the parameters of barbarik (use --help to see a description of all the options). We can also specify the sampler used as reference the same way with `--ref-sampler` followed by the sampler to use.

A full usage example is as follows: 

`docker run -v $(pwd):/home/usampling-exp:z macher/usampling:squashed /bin/bash -c 'cd /home/usampling-exp/; echo STARTING; python3 barbarikloop.py -flas /home/samplingfm/Benchmarks/FeatureModels/FM-3.6.1-refined.cnf --sampler 9 --seed 1 --timeout 100; echo END'` 

It's QuickSampler with JHipster feature model and timeout=100 seconds... end eta/epsilon defaut values `cmd: ['python3', 'barbarik.py', '--seed', '1', '--verb', '1', '--eta', '0.9', '--epsilon', '0.3', '--delta', '0.05', '--reverse', '0', '--exp', '1', '--minSamples', '0', '--maxSamples', '9223372036854775807', '--sampler', '9', '--ref-sampler','6', '/home/samplingfm/Benchmarks/FeatureModels/FM-3.6.1-refined.cnf']` 

## Samplers and data used 

```
SAMPLER_UNIGEN = 1
SAMPLER_QUICKSAMPLER = 2
SAMPLER_STS = 3
SAMPLER_CMS = 4
SAMPLER_UNIGEN3 = 5
SAMPLER_SPUR = 6
SAMPLER_SMARCH = 7
SAMPLER_UNIGEN2 = 8
SAMPLER_KUS = 9
SAMPLER_DISTAWARE = 10
```

With BURST, large study and results of different SAT-based samplers are possible:
 * KUS https://github.com/meelgroup/KUS (new!)
 * SPUR https://github.com/ZaydH/spur (new!) 
 * Unigen2 and QuickSampler https://github.com/diverse-project/samplingfm/
 * SMARCH https://github.com/jeho-oh/Kclause_Smarch/tree/master/Smarch (new!)
 * other: Unigen3, CMS, STS (new!)
over different data:
 * https://github.com/diverse-project/samplingfm/ (including SAT formulas and hard feature models)
 * https://github.com/PettTo/Feature-Model-History-of-Linux (new!)

### Pre-built Docker image 

We recommend to use `macher/usampling:squashed` but other variants are possible (eg `macher/usampling:fmlinux` for a Docker image with th 5Gb dataset of Linux feature model)
Everything is available here https://cloud.docker.com/repository/docker/macher/usampling

### Requirements

  * Docker image with Python 3, pandas, numpy, setuptools, pycoSAT, anytree 
  * solvers above and a proper installation 
  * time and resources ;) 



## Architecture and extensibility 

### Organization of the repo 

 * All samplers are in `samplers` directory (and all utilities/dependencies are also in this folder)
 * `usampling-experiments.py` pilots the scalability study of samplers over different datasets 
 * `barbarik.py` pilots the uniformity checking of samplers over different datasets. It is based on the barbarik tool from Kuldeep Meel et al: https://github.com/meelgroup/barbarik. This version supports uniformity check for all the 10 solvers above and uses KUS as a reference uniform solver, if not specified in the command line above.
 * `barbarikloop.py` allows to run uniformity checks on set fo files (using the same flas technique as above) and report the results in a CSV file

### Extensibility 

To configure a new sampler to work with Barbarik, here are the basic steps in `barbarik.py` (see also comments in the script): 

 * add a new number/enumeration eg `SAMPLER_XXX = 11`
 * expand the class `SolutionRetriver`. This is the class where solution obtained from samplers are fed to Barbarik. you need to: 
    * 1. define a method `get_solution_from_XXX(*topass_withseed)` in this class. This method needs to parse the output of the sampler and create an internal representation fo the list of solutions. Each solution is a list of literals preceded by '-' if is not selected.
    * 2. Wrap the call at the beginning of the Class: if/elif blocks.
Note that since all samplers have different output formats, most of the work consists of parsing the output to create a list of solutions in a format acceptable by Barbarik. This step can be very specific and ad-hoc; we provide facilities to ease the effort and integration.   
   
