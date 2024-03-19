# Summary

This repository contains code, examples and data accompanying the "Diffusion, viscosity and linear rheology of valence-limited disordered fluids" paper.

## The simulation code

All simulations have been run with [oxDNA](https://github.com/lorenzo-rovigatti/oxDNA). The repository contains a snapshot dated 19/03/2024, which can be compiled with the following commands:

```
cd oxDNA
mkdir build
cd build
cmake ..
make install -j6
make rovigatti -j6
```

At the end of the compilation stage, the `oxDNA` executable will be placed in the `build/bin` folder. Check the [oxDNA docs](https://lorenzo-rovigatti.github.io/oxDNA/) for additional information.

Note that the patchy interaction used in the paper is implemented as an [oxDNA plugin](https://lorenzo-rovigatti.github.io/oxDNA/input.html#plugins-options), which means that input files should be updated to contain the correct `plugin_search_path` values.

### Simulation options

All the simulation options are specified in the input file that `oxDNA` takes as an argument. The most important options (specified as `key = value`) are:

* `T` is the temperature at which the simulation will be run
* `dt` is the integration time step
* `DPS_lambda` is the $\lambda$ value: use $\lambda = 1$ and $\lambda = 10$ to simulate systems with and without swap, respectively.
* `thermostat` sets the way the simulation will be coupled to a thermostat. Use `thermostat = no` to run NVE simulations.

### File formats

The configuration file has the [same format](https://lorenzo-rovigatti.github.io/oxDNA/configurations.html) as oxDNA, while the topology format is unique to these simulations and should have the following layout:

```
N N_species
N_1 N_patches_1 patch_type_a,patch_type_b,...
N_2 N_patches_2 patch_type_c,patch_type_d,...
```

where the first line contains the total number of particles and the number of particle species. Each following line pertains to a particle species and contains 3 fields: the number of particles of that species, its number of patches and then a comma-separated list of patch types. Note that the particle data stored in configurations is listed according to the order specified here.

As an example, the topology of a system made of 200 tetravalent particles and 800 divalent particles, all having patches of the same type would read

```
1000 2
200 4 0,0,0,0
800 2 0,0
```

The model also needs a file that contains an "interaction matrix" specifying the strength of patch-patch interactions. If not otherwise specified, by default the strength is 0. In the paper only same-type patches appear, and the interaction matrix file has always the following content:

```
patchy_eps[0][0] = 1.0
```

## Examples

The `examples` folder contains all the files required to run simulations of two of the systems investigated in the paper: a pure $M = 4$ system (`examples/M4`) and a binary mixture made of 200 trivalent and 800 divalent particles (`examples/M_2.2_M32`). Each leaf subfolder contains the following files:

* `input` is the oxDNA input file.
* `topology.dat` is the topology file.
* `interaction_matrix.dat` specifies the interaction matrix.

You will first have to generate an initial configuration by using oxDNA's `confGenerator`, which takes an input file and a density value (in units of the inverse of the bead diameter cubed, $\sigma^{-3}$) as arguments, like this:

```
PATH/TO/OXDNA/build/bin/confGenerator input 0.6
```

A simulation can then be run by using the `oxDNA` executable, as follows:

```
PATH/TO/OXDNA/build/bin/oxDNA input
```

**Nota Bene:** if you copy the files somewhere else you'll also have to update the `plugin_search_path` values specified in the input files.

## Data

The `data` folder contains most of the raw data used to draw the figures in the paper. The data is organised in the following hierarchy of folders: `SYSTEM/LAMBDA/DENSITY/TEMPERATURE/SAMPLE`, where

1. `SYSTEM` refers to the investigated system: `M_2_NVE`, `M_3_NVE` and `M_4_NVE` are the pure systems, the rest are binary mixtures specified by the average valence and the valence of the particles they are composed of. For instance, `M_2.4_M32_NVE` is a system of average valence `2.4` composed of trivalent and divalent particles.
2. `LAMBDA` can be either `LAMBDA_1.0` or `LAMBDA_10.0` and refers to systems with and without swap, respectively.
3. `DENSITY` is always equal to 0.6.
4. `TEMPERATURE` is the temperature at which the simulations were initially equilibrated.
5. `SAMPLE` has a name like `TRY_Y`, with all folders containing data generated with independent simulations of the same state point.

The folders store `.dat` files whose name should be self-explanatory. All files with prefix `arr_` are quantities expressed as a function of $1 / T$. Files prefixed with `avg_` are obtained by analysing time series obtained by averaging over all the samples data (*e.g.* first average all mean-squared displacements and then compute $D$), while files without it are obtained by first analysing the single time series and then taking averaging over the results (*e.g.* compute $D$ for each sample and then take the average). The two sets of results are very similar to each other.

Contact the authors if you have any questions about the data.

