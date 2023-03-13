# Runnig Alphafold2 on the BMC cluster


## Setup

### getting access to the cluster
Please contact Dr. Tobias Straub (tstraub@bmc.med.lmu.de)

### connecting to the cluster

	> ssh -t yourID@MBIOHW30.bio.med.uni-muenchen.de 

### conda

install conda/miniconda (<https://docs.conda.io/en/latest/miniconda.html>) in case you did not do so already.

	> cd .. # go $HOME
	> wget https://repo.anaconda.com/miniconda/Miniconda3-py39_23.1.0-1-Linux-x86_64.sh
	> bash Miniconda3-py39_23.1.0-1-Linux-x86_64.sh

follow the instructions and answer every prompt with yes.


## alphafold folder and environment setup
Copy the alphafold version 2.3 folder into your project folder.  In this folder you will find the scripts, the environment (.yml) file and the folders where you will store your input fasta file and your results.

	> cd /work/project/<your_project_folder>
	> cp -R /work/data/alphafold/alphafold-2.3.0 .

setup conda environment

	> cd alphafold-2.3.0
	> module load cuda/11.3
	> conda create --name alphafold_v2 python==3.8
	> conda activate alphafold_v2
	> conda install -y -c conda-forge openmm==7.5.1 cudnn==8.2.1.32 cudatoolkit==11.3.1 pdbfixer==1.7
	> conda install -y -c bioconda hmmer==3.3.2 hhsuite==3.3.0 kalign2==2.04
	> pip install absl-py==1.0.0 biopython==1.79 chex==0.0.7 dm-haiku==0.0.9 dm-tree==0.1.6 immutabledict==2.0.0 jax==0.3.25 ml-collections==0.1.0 numpy==1.21.6 scipy==1.7.0 tensorflow==2.11.0 pandas==1.3.4 tensorflow-cpu==2.11.0
	> pip install --upgrade jax==0.3.25 jaxlib==0.3.25+cuda11.cudnn805 -f https://storage.googleapis.com/jax-releases/jax_cuda_releases.html

## running alphafold 

activate conda environment

	> conda activate alphafold_v2.3

You need to create a fasta file for the amino acid sequence that you want to submit. There is a .fasta file for ‘14-3-3’ provided in the folder to serve as test example.

	> cd yourlocation/alphafold-2.3.0/fasta_in
	> nano yourproteinname.fasta

An empty text document will open. Here you can paste your protein as the uniprot download format, eg.:

	>sp|P29310|1433Z_DROME 14-3-3 protein zeta OS=Drosophila melanogaster OX=7227 GN=14-3-3zeta 	PE=1 SV=1
	MSTVDKEELVQKAKLAEQSERYDDMAQAMKSVTETGVELSNEERNLLSVAYKNVVGARRS
	SWRVISSIEQKTEASARKQQLAREYRERVEKELREICYEVLGLLDKYLIPKASNPESKVF
	YLKMKGDYYRYLAEVATGDARNTVVDDSQTAYQDAFDISKGKMQPTHPIRLGLALNFSVF
	YYEILNSPDKACQLAKQAFDDAIAELDTLNEDSYKDSTLIMQLLRDNLTLWTSDTQGDEA
	EPQEGGDN

CTRL + O saves a nano file. 
CTRL + X exits nano.

Go back to the main folder:

	> cd ..

### submit the script: 

For monomer:

	> sbatch run_alphafold.sh  -f ./fasta_in/ yourproteinname.fasta

For multimer: 

	> sbatch run_alphafold.sh  -f ./fasta_in/ yourproteinname.fasta -m multimer 


Other optional parameters can be found at the end of this document.

To check if your job is running, use

	> squeue

The results can be found in /yourlocation/alphafold-2.3.0/out_files/yourproteinname

When your job has finished, open a new terminal window on your desktop. To copy the data onto your computer, use:

	>scp yourID@MBIOHW30.bio.med.uni-muenchen.de: /yourlocation/alphafold-2.3.0/out_files/yourproteinname*/*.pdb .

You can now look at the files in pymol or similar software.

Optional Parameters:

	-d <data_dir>          Path to directory of supporting data (default:/work/project/ladlad_008/databases)
	-o <output_dir>        Path to a directory that will store the results (default: ./out_files/).
	-t <max_template_date> Maximum template release date to consider (ISO-8601 format - i.e. YYYY-MM-DD). 
			               Important if folding historical test sets (default:2022-12-31)
	-g <use_gpu>           Enable NVIDIA runtime to run with GPUs (default: true)
	-r <run_relax>         Whether to run the final relaxation step on the predicted models. Turning relax 
			               off might result in predictions with distracting stereochemical violations but 
			               might help in case you are having issues with the relaxation stage (default: true)
	-e <enable_gpu_relax>  Run relax on GPU if GPU is enabled (default: true)
	-n <openmm_threads>    OpenMM threads (default: all available cores)
	-a <gpu_devices>       Comma separated list of devices to pass to 'CUDA_VISIBLE_DEVICES' (default: 0)
	-m <model_preset>      Choose preset model configuration - the monomer model, the monomer model with 
			               extra ensembling, monomer model with pTM head, or multimer model (default: 
			               'monomer')
	-c <db_preset>         Choose preset MSA database configuration - smaller genetic database config 
			               (reduced_dbs) or full genetic database config (full_dbs) (default: 'full_dbs')
	-p <use_precomputed_msas> Whether to read MSAs that have been written to disk. WARNING: This will not 
			                  check if the sequence, database or configuration have changed (default: 'false')
	-l <num_multimer_predictions_per_model> How many predictions (each with a different random seed) will 
			                                 be generated per model. E.g. if this is 2 and there are 5 models then there will 
			                                 be 10 predictions per input. Note: this FLAG only applies if 
			                                 model_preset=multimer (default: 1)
	-b <benchmark>          Run multiple JAX model evaluations to obtain a timing that excludes the 
			                 compilation time, which should be more indicative of the time required for 
			                 inferencing many proteins (default: 'false')

## Problems

The software is not running, what do I do?

1)	Check the .out file:
cd /yourlocation/alphafold-2.3.0/out_files/
sort the files by date, the newest will be at the bottom:
	
	> ls -altr 

choose latest out file:
nano latestfile.out

It will tell you the problems it has been facing.

If you still don’t know what to do, contact: maren.heimhalt@bmc.med.lmu.de





	


