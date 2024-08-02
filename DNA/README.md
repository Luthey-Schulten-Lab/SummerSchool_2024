# Simulating DNA using btree_chromo and LAMMPS

## Description:

The purpose of this tutorial is to give you a crash course on modeling and simulating the bacterial chromosome. You will be introduced to the minimal cell chromosome: it’s structure, interactions it has with proteins, and how it replicates. We will walk you through how to set up and run a simulation using the program btree_chromo, on the Delta HPC cluster. The coarse-grained model of the DNA, ribosomes and cell membrane will be discussed, as well as the use of LAMMPS to perform energy minimizations and Brownian dynamics. We will also go into greater detail about how we model biological mechanisms such as SMC looping and topoisomerase. Finally, you will get a chance to visualize and analyze a simulation trajectory in VMD.

*This tutorial was prepared for the STC QCB Summer School, held during August of 2024.*

## Outline of tutorial:

1. Introduction to DNA simulation with btree_chromo and LAMMPS
2. Running btree_chromo: Setting up on Delta and running a simulation
3. Modeling the minimal cell
4. Modeling chromosome dynamics
5. Understanding btree_chromo commands
6. Visualization and analysis with VMD

## 1. Introduction to DNA Simulation with btree_chromo and LAMMPS

<img align="right" width="300" src="./figures/1. Introduction to simulation with btree_chromo and LAMMPS/initial_state.png">

Here, we simulate DNA replication and dynamics using the C++ program btree_chromo, available online at  [github.com/brg4/btree_chromo](https://github.com/brg4/btree_chromo). This program was created mainly for the purposes of simulating the minimal cell chromosome, but it can be used to simulate any circular chromosome. The main purpose of the program is to model replication states of the chromosome, as well as perform simulation of chromosome dynamics by calling [LAMMPS](https://www.lammps.org/#gsc.tab=0) (Large-scale Atomic/Molecular Massively Parallel Simulator), a molecular dynamics program from Sandia National Laboratories.  

The DNA that btree_chromo simulates is coarse-grained at a 10 bp resolution. This means that a single, 3.4 nm diameter bead is used to represent 10 base pairs. We also use beads to represent ribosomes (10 nm), and the cell membrane. The program emulates the effects of SMC (structural maintenence of chromosomes) proteins that extrude loops of DNA to effect chromosome organization, as well as type II topoisomerases which allow for DNA strand crossings when they become tangled.

ZLS group member Ben Gilbert (recently graduated) wrote the program and used it to investigate chromosome disentanglement of replicated DNA strands and partitioning of the daughter strands into separate halves of a toy model cell (50 kbp genome), as well as create chromosome contact maps based on the simulated trajectories ([Gilbert 2023](https://www.frontiersin.org/journals/cell-and-developmental-biology/articles/10.3389/fcell.2023.1214962/full)). Ongoing research using btree_chromo involves integrating btree_chromo with Lattice Microbes, as well as investigating disentanglement and partitioning of a full minimal cell model (543 kbp genome). 

| <img src="./figures/1.%20Introduction%20to%20simulation%20with%20btree_chromo%20and%20LAMMPS/division_2chromo_5000bp_separation.png" width="300"/>  | <img src="./figures/1.%20Introduction%20to%20simulation%20with%20btree_chromo%20and%20LAMMPS/partition.png" width="300"/> |
|:--:|:--:|
| Figure 1: Cell division of 50 kbp toy model | Figure 2: DNA partitioning of 543 kbp model |


Today you will be running simulations using a variant of LAMMPS which utilizes the GPUs on the Delta HPC cluster. We will simulate the entire minimal cell including the effects of SMC proteins, topoisomerase, and Brownian dynamics. 

## 2. Running btree_chromo: Setting up on Delta and running a simulation

**Step 1: Log in to Delta**

Connect to the Delta HPC using the following command. You will need to type your password and do 2FA.

```bash
ssh $USERNAME@login.delta.ncsa.illinois.edu
```

> [!WARNING]
You will need to replace $USERNAME with your own.
> 

**Step 2: Create workspace and copy examples folder**

```bash
bash /projects/bcuj/sharefile/Workshop_2024/DNA_model/prelaunch_btree_chromo.sh

```

This bash script creates a workspace `/projects/bcuj/${USER}/btree_chromo_workspace`. It also copies the `examples` directory into the workspace, which contains example input and output files for **btree_chromo**.

**Step 3: Launch the container**

```bash
bash /projects/bcuj/sharefile/Workshop_2024/DNA_model/launch_btree_chromo.sh

```

This will run the container in interactive mode. You should now see the `Apptainer>` prompt which indicates your have entered the container. We will be running btree_chromo and viewing the terminal output in the container.

**Important:** The btree_chromo  exectuable is within the container in `/Software/btree_chromo/build/apps/`. \
**Important:** We have mounted `/projects/bcuj/${USER}/btree_chromo_workspace/examples` into the container in `/mnt/examples`. All changes made in the workspace will be reflected in the container.

**Step 4: Open a new terminal window**\
In a new terminal window, repeat step 1 and

```bash
cd  /projects/bcuj/${USER}/btree_chromo_workspace/examples

```

Viewing and editing of the example files will be done within this terminal window.

## Typical Workflow:

The general workflow for running **btree_chromo** is as follows:

1. Prepare an input file (*directives.inp*) containing the directives to be executed by the binary tree program. Lines beginning with '#' are ignored.
2. Prepare any auxiliary input files needed for the chosen directives.
3. Run with: `./btree_chromo (some location)/(some name)_directives.inp`

> [!IMPORTANT]
We will henceforth refer to the terminal running the container as "**Apptainer terminal**", and the teminal for viewing and editing files, such as directives files, as the "**Editor terminal**".
> 

Next, we will run the program in the **Apptainer terminal**. First, change directories to where the executable is located:

In the **Apptainer terminal**:

```bash
cd /Software/btree_chromo/build/apps

```

Next, in the **Apptainer terminal** run 'full_model.inp'.

In the **Apptainer terminal**:

```bash
./btree_chromo /mnt/examples/full_model.inp

```

## 3. Modeling the Minimal Cell
### Growing the DNA
<img align="right" width="300" src="./figures/3. Modeling the minimal cell/sc_growth_composite.pdf">

### Modeling Replication States

In this section, you will learn how to represent replication states, including nested theta structures, with a binary tree model. You will also get some practice with navigating directories in a linux terminal.

In **Editor terminal**,

```bash
ls /preparing_chromosome

```

You should see the following files:

| File name | Description |
| --- | --- |
| preparing_chromosome_directives.inp | contains directives (commands to be executed by btree_chromo) |
| chromo_state_0.dat | contains the initial replication state |
| chromo_state_1.dat | contains the final replication state |
| transforms.dat | contains a set of replication events |

Using a text exitor such as **vim**, open 'preparing_chromosome_directives.inp'. Scroll through the file to see all of the commands. Here, **btree_chromo** first creates a chromosome with 1000 monomers, and then applies a series of transforms, printing the replication state at each step, and finally outputs the final state.

> [!WARNING]
Make sure that the "terminate'' at the top is commented to execute all of the commands.
> 

Next, we will run the program in the **Apptainer terminal**. In the **Apptainer terminal** run 'preparing_chromosome_directives.inp'.

In the **Apptainer terminal**:

```bash
./btree_chromo /mnt/examples/preparing_chromosome/preparing_chromosome_directives.inp

```

Check out the terminal output. Notice how **btree_chromo** starts by testing the validity of all the commands and parameters and prints them to the terminal output. It then executes each of the commands. For example, you should see this output corresponding to the first replication transformation:

```bash
COMMAND: transform
	param_0: m_cw100_ccw200

COMMAND: print

printing tree with 2 leaves and 1 forks
fork breakdown: 0 completed, 1 active
leaves:
ml mr
active forks:
m
total_size = 1300
| generation = 0
| rho_t = 300/1000, rho_cw = 100/800, rho_ccw = 200/900
| start = 0, mid = 500, end = 999
| start_link = 999, end_link = 0
| left branch
  | generation = 1
  | start = 0, mid = 500, end = 999
  | start_link = 999, end_link = 0
| right branch
  | generation = 1
  | start = 1000, mid = 1200, end = 1299
  | start_link = 299, end_link = 600

```

Next, in the **Editor terminal**, move to the next example in `examples/preparing_physical_structure`.
There are several file types in this directory. Here are what they are used for:

| File type | Description |
| --- | --- |
| .inp | directives (commands to be executed by btree_chromo) |
| .dat | replication state |
| .xyz | monomer coordinates for quick visualization |
| .bin | monomer/ribosome coordinates (binary) |
| data. | LAMMPS data file |

## 4. Modeling Chromosome Dynamics

Take a look in the provided README for btree_chromo: [github.com/brg4/btree_chromo](https://github.com/brg4/btree_chromo). Scroll past the description and installation steps to see the list of possible directives.
Let's focus our attention on the commands that we can use to toggle various aspects of the Brownian dynamics simulation in LAMMPS, under Spatial System for Simulations: bonds, bending angles, and twisting angles.

| Directive | Description |
| --- | --- |
| switch_bonds:(T/F) | enable/disable bonds between DNA monomers (default T) |
| switch_bending_angles:(T/F) | enable/disable bending angles between DNA monomers (default T) |
| switch_twisting_angles:(T/F) | enable/disable twisting angles between DNA monomers (default T) |

*Note: these directives must be used prior to 'write_LAMMPS_data_file' to take effect*.
By turning off these switches, we are effectively removing terms from the whole energy function that correspond to adjacent-monomer interactions in the DNA polymer:
$$U= \sum_{i=1}^{N_{\mathrm{DNA}}}\left[U_i^b+U_i^t+U_i^a+U_i^s\right] +\sum_{i=1}^{N_{\mathrm{DNA}}-1} \sum_{j=i+1}^{N_{\mathrm{DNA}}} U_{i j}^{\mathrm{DNA}-\mathrm{DNA}}+\sum_{i=1}^{N_{\mathrm{DNA}}} \sum_j^{N_{\text {ribo }}} U_{i j}^{\mathrm{DNA}-\text { ribo }} +\sum_{i=1}^{N_{\text {ribo }}-1} \sum_{j=i+1}^{N_{\text {ribo }}} U_{i j}^{\text {ribo-ribo }} +\sum_{i=1}^{N_{\text {bdry }}} \sum_j^{N_{\mathrm{DNA}}} U_{i j}^{\text {bdry-DNA }}+\sum_{i=1}^{N_{\text {bdry }}} \sum_j^{N_{\text {ribo }}} U_{i j}^{\text {bdry-ribo }}.$$
Turning off bending removes the $U_i^b$ (cosine potential for bending), turning off twisting removes the $U_i^t$ and $U_i^a$ (cosine potentials for twisting and aligning), and turning off bonds will remove all of these as well as the $U_i^s$ (FENE potentials for stretching) resulting in separate diffusing DNA monomers rather than a DNA polymer. The other five terms in the energy function are for excluded volume interactions (purely repulsive Weeks-Chandler-Andersen (WCA) pair potentials).

|**Bending:** |**Twisting** | **Stretching** | **Excluded Volume** |
|:--:|:--:|:--:|:--:|
| <img src="./figures/4.%20Modeling%20chromosome%20dynamics/DNA_model_bending_0.png/" width="300"/>  | <img src="./figures/4.%20Modeling%20chromosome%20dynamics/DNA_model_twisting_0.png/" width="300"/> |  <img src="./figures/4.%20Modeling%20chromosome%20dynamics/DNA_model_stretching_0.png/" width="200"/> |  <img src="./figures/4.%20Modeling%20chromosome%20dynamics/DNA_model_LJ_0.png/" width="120"/> |
|$U_i^b=\kappa_b\left[1-\cos \left(\pi-\theta_i\right)\right]$|$U_i^t=\kappa_t\left[1-\cos \left(\alpha_i+\gamma_i\right)\right]$|$U_i^s= -\frac{\kappa_s L_0^2}{2} \log \left[1-\left(l_i / L_0\right)^2\right]$ $+4 \epsilon_s\left[\left(\frac{\sigma_s^s}{l_i}\right)^{12}-\left(\frac{\sigma_s}{l_i}\right)^6\right]$ $\times \Theta\left(2^{\frac{1}{6}} \sigma_s-l_i\right)$  |$U_{i j}^{e . v .}=  4 \epsilon_{e . v}\left[\left(\frac{\sigma_{e . v}}{r_{i j}}\right)^{12}-\left(\frac{\sigma_{e . v}}{r_{i j}}\right)^6\right]$ $\times  \Theta\left(2^{\frac{1}{6}} \sigma_{{e.v. }}-r_{i j}\right)$|

For simulating the influence of loops and topoisomerase, the relevant directives are described in the [README](https://github.com/brg4/btree_chromo/) under Simulator:

| Directive | Description |
| --- | --- |
| simulator_load_loop_params:loop_params_file | read a file (loop_params_file) containing the parameters for the looping interactions |
| simulator_run_loops:Nloops,Nsteps,Tfreq,Dfreq,append_option,skip_option | run Brownian dynamics with the hard/FENE potential for Nsteps with Nloops randomly placed, while printing thermodynamic information every Tfreq steps and dumping every Dfreq steps - append_option = noappend/append and skip_option = first/skip_first |

If we take a look at loop_params.txt, we find that it specifies the minimum number of monomers separating anchor and hinge, the distribution of extrusion steps, hinge unbinding probability and grab radius, loop update frequency and topoisomerase relaxation frequency.

```bash
# system parameters

# minimum distance separating anchor and hinge on strand - [# monomers]
min_dist=5

# distribution family for steps
family=poisson
# average extrusion distance [# monomers]
ext_avg=20.0
# max extrusion distance [# monomers]
ext_max=30

# hinge unbinding probability
p_unbinding=0.0
# grab radius - [A]
r_g=5.0E+2

# simulator parameters

# loop update frequency - [# timesteps]
freq_loop=10000

# topoisomerase relaxation (soft DNA-DNA pairs) frequency - [# timesteps]
freq_topo=50000
# topoisomerase relaxation (soft DNA-DNA pairs) interval - [# timesteps]
dNt_topo=50000

```

The effect of SMC looping during the minimal cell replication cycle is not fully understood. While magnetic tweezer experiments have been done to determine loop extrusion step size of ~200 bp/s (Ryu et al, NAR 2022) and simulations indicate an extrusion frequency of ~2.5 steps/s (Nomidis et al, NAR 2022), we have limited experimental results for SMC looping in the crowded in-cell environment. btree_chromo allows us to investigate SMC looping through adjustment of simulation parameters.

| Parameter | Description |
| --- | --- |
| Total number of loops | Number of active anchor+hinge pairs that are extruding loops |
| Loop extrusion frequency (s^-1) | How often does loop extrusion occur? Our best estimate (Nomidis et al) is around every 0.4 s |
| Unbind/Rebind frequency (s^-1) | How often does the anchor move to a new location? After 90 extrusion steps? Less? |
| Extrusion step size (bp) | ~200 bp (Ryu et al) |

Recall that 

## 5. Understanding btree_chromo Commands

## 6. Visualization and analysis with VMD

**Visualizing LAMMPS trajectories with VMD:**\
You will now copy over the .lammpstrj files from Delta to your local machine in order to visualize them in vmd.

In the **Local terminal**:

```bash
scp $USERNAME@login.delta.ncsa.illinois.edu:/projects/bcuj/$USERNAME/btree_chromo_workspace/examples/simulating_chromosome_with_replication/\\*.lammpstrj .

```

(Alternatively, instead of `.` you may choose to specify path to a local directory.)
We will also copy over the .tcl scripts which will create nice representations for the DNA, ribosomes and boundary particles.

In the **Local terminal**:

```bash
scp $USERNAME@login.delta.ncsa.illinois.edu:/projects/bcuj/sharefile/Workshop_2024/DNA_model/\\*.tcl .

```

In VMD, delete the previous two molecules. Open the VMD TkConsole and do (Extensions->Tk Console). In the Tk Console, do `source load_example_full.tcl`. See the DNA polymer replicate!

| Monomer type | Color | Bead Size |
| --- | --- | --- |
| DNA | viridis color scale | 13.0 |
| Ori | red | 26.0 |
| Ter | orange | 26.0 |
| Fork | magenta | 26.0 |
| Ribosome | mauve | 70.0 |
| Boundary | gray | 32.5 |
| Anchor | black | 19.5 |
| Hinge | white | 19.5 |

## Links

1. [Public version of btree_chromo code](https://github.com/brg4/btree_chromo/tree/master), including in-depth README with explanation of code, input commands, and numerous examples of input files
2. [Ben’s paper](https://www.frontiersin.org/journals/cell-and-developmental-biology/articles/10.3389/fcell.2023.1214962/full) (goes along with code; investigates effect of topoisomerases and SMC looping)
3. [Ryu et al](https://watermark.silverchair.com/gkab1268.pdf?token=AQECAHi208BE49Ooan9kkhW_Ercy7Dm3ZL_9Cf3qfKAc485ysgAAA3QwggNwBgkqhkiG9w0BBwagggNhMIIDXQIBADCCA1YGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMA59DoPedmilVf1DbAgEQgIIDJ8aawyI4vnwQVACfC477JCaN79RsA-0xR99FeUs_UrNv19E7fbB8MzTIoryPiah0rwNRefWkEijhBjOlQnLrTFWjdk5NofDT1Wsr-HC8S93hljm99r9JttImcTpmSpaMJJUeVfbWk2AjbwcfaENTPUYZ0IDMo8Hm7JG2_b1f6FixKjMXruoIMWRwICge2xphz31oDIWveArL9HxWF2MHuJyJOA4JQRRcS6v5NiBPtUb5_lpo5UpJmARTx4-P4Zv2v0fydjXJZHMaybmJBUapbfTFg2kOQ1Amra9tixr1qiaMw3T0_IISEJmQH5R1wL24F03jRx80NziAxEzxBq-BDQPn_-qbWFHIZSi1oCmmirw3BTS6aBmlv5D2vbbj6U28YtQuQ0VWc8RDuWZfnIHs1jQRc1wB4iDfX_4JmXGUpGq6Rs8f6RsX7jFL-y0sEi095YYGXABzIAFPa5kPowsluykXfDThHpGBGKOzx-sgVGF5fDsicafsPA6pSLNt1wbKGAcZkY8O1Ks9ZpiYdd9DNbO1X_doMkcO83uDD4RLqPDT59chj4ndUvz2qJ99XDrAq5bvuOj3z_y8BTJs4fn2lXphlAlGwQVCLBfg3sUV1HUZtK5SqTYm3Z8c9HuZvsxl0fBdNm-ustJftwNJV9E26lhfB0wGazO4WmY2UTyEIJ9Q0rpyDh4UOn9uNCtXybabKo2u7BRtirEOoP1JCT3-hRO5PUkJUToJXn9mRxJ03eVXtPqwic_oTxqwgVf0vqgs-Q_M5gRtAqvhN8QeZYB8Udi9Pv9ID3sknbfxOoy7SjjRiR5i4AsHBvyvAK2BSwFlwQ37o-wsRHV-fr-oIyGRZ9mFQikR6zNgF2Ry0oZtSCB2eRJwi3ABbVKR6qBu9XPVm1q3dDfTrm_s6ycYu2Ues_zaiNwp_QfMPN8nEuYHnl5WlVsbSKzbHhCbSboICGNbnklDxODznTLEvb-oObNcY6LXT7VheXl3iT_gapR7FoYf8H2-dAmo6FW51zwX5EwtCG1cFTo2QbQpXCXAE3nceOzm3lE2oqIXAJN7b_H7HR4DDFTFfGrnfA) (magnetic tweezers)
4. [Nomidis et al](https://watermark.silverchair.com/gkac268.pdf?token=AQECAHi208BE49Ooan9kkhW_Ercy7Dm3ZL_9Cf3qfKAc485ysgAAA3MwggNvBgkqhkiG9w0BBwagggNgMIIDXAIBADCCA1UGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMTq-Fwt4IdPb2QsVkAgEQgIIDJnrY-47zEepn0CV_7_8nYkuAVVtWC5E7SEkZEIcc0LCf3MU1VrXVqrtSfiFexlwwUQUsY4NelEgiNVhH7Ci6RI_dhshO4kUrnkkZus1SlA_ShFjtwOdJp00JGvCejyFlafdStptVBByxY4pTITUcyW388ezKzqCnkG6jrXod7Jz4slJ0YTGkx72HijxUi9_03gUHb_tNfWVGV-NNg4eNM-DYgNTclMBp2jZ3k8t4iHsWVQJrNifKblG6Y-Af-Abg610YMn693SOqyF1V2H-cJIZD9pxL8PGUDInbggyb2y0qhTp0tMzRYmuBy7lhneiJodHJC8bgymGb08bssROO90dSENKVJ4w71ASTonM0ZTmDKk27DoQ_jWe2Y58grA6K_y-Ec_KTBsPF2XtltSpO6QZapEOlpOzMIq1bvMn2Z49ws8tEC7MSd4Bxgb-zvk7rx14nXEro4JnE1GDqWyIi_L7ctUpduMx_nq6txZonhgp-M9w7RIofbRAlp1lVO4uDyXdUAzSRUeTKd978jFsqMMHlimwyB2mDp2BeLmIfUH35EioDQBnIwpZrSmSXc2TC6KYlyQ3QrGhUuNx58eT3SPP3LbXa-D8hnEcdIx47GF_TSwnkx3bRO_J5FhVjAuUbi3-_EOVGaCscrs7iAAj2ZPTiYRsQUSTVcX-N_jExdmC52qGRdE0ujirZVsEBszbfcrX-7ZXAxf4vBvEIRM3KRjEsZW6RMJXCoIgfCgeYtTAz9m7oeDQXmEC04kwHcVxI3w9oqCjoKoZfDHjJQKNmBIvRbwd2JlQes-FloHz_0gLnEVORwEbIfSeLHCrIMCvMRPIE5RWVD3vZ1qjzSfmMDMuPEg-hLo6IQvhvi_CDjepcfDNqxsTk52pZT1os1aeK6JQ9awzKer0Ks0LYWAtZDvf79mGtve1TUTZdEu-DYH2WPrF80CnhbZALFgjvT4pTa1XAp_kliKAEVYLRqmenQmYqpVOCghn_MPc9fqw_yQXB_m3Q-ZxebGMX2udsZy_MGapfjy21UMD6kVrF7RB5EABMqboR_K1yQ5jjGLkB0mYjcTQ3hm9J) (MD simulation on ratcheting motion)
5. Broedersz [google scholar page](https://scholar.google.com/citations?hl=en&user=xSgWTTQAAAAJ&view_op=list_works&sortby=pubdate)
6. [Zawadzki et al](https://www.sciencedirect.com/science/article/pii/S2211124715013492?via%3Dihub) (topo coordination with SMC)