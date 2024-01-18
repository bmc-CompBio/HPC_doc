# Running Jupyter Notebook on the HPC

## Conda environment

Create a conda environment on the cluster that contains Jupyter Notebook and all software components required for your analyses

e.g defined in a yaml file env.yaml that contains

	name: sc_jupyter
	channels:
 	 - bioconda
 	 - conda-forge
	dependencies:
 	 - python
  	 - jupyter
  	 - pandas
  	 - scipy
  	 - r
  	 - rpy2
  	 - r-Seurat
  	 - anndata2ri
  	 - bioconductor-SingleCellExperiment
  	 - scanpy
  	 - python-igraph

then

	mamba env create -f env.yaml

## Launch script

Create a launch script called e.g. jupyter.sh with the following content and save it into the directory that will serve as working directory for your Jupyter session

	conda activate sc_jupyter
	unset XDG_RUNTIME_DIR
	echo "done."
	echo "*** Setting Jupyter interrupt character to Ctrl-T instead of Ctrl-C"
	echo "*** to avoid conflicts with Slurm."
	stty intr ^T
	echo ""
	echo "*** Starting Jupyter on: " $(hostname)
	jupyter notebook --no-browser --ip='0.0.0.0' # earlier versions of Jupyter 	allowed '*' instead of '0.0.0.0'

## Startup Jupyter Notebook on an HPC node

On the master node cd into your working directory and issue

	srun -p <partition> --pty jupyter.sh

where <partition> has to be replaced with the desired cluster partition slim16, slim18, fat, or gpu.

you will get standard output like this

	done.	
	*** Setting Jupyter interrupt character to Ctrl-T instead of Ctrl-C
	*** to avoid conflicts with Slurm.
		
	*** Starting Jupyter on:  gpu01
	[W 2024-01-18 13:23:24.552 ServerApp] A `_jupyter_server_extension_points` function was not found in jupyter_lsp. Instead, a `_jupyter_server_extension_paths` function was found and will be used for now. This function name will be deprecated in future releases of Jupyter Server.
	[W 2024-01-18 13:23:24.647 ServerApp] A `_jupyter_server_extension_points` function was not found in notebook_shim. Instead, a `_jupyter_server_extension_paths` function was found and will be used for now. This function name will be deprecated in future releases of Jupyter Server.
	[I 2024-01-18 13:23:24.648 ServerApp] jupyter_lsp | extension was successfully linked.
	[I 2024-01-18 13:23:24.652 ServerApp] jupyter_server_terminals | extension was successfully linked.
	[I 2024-01-18 13:23:24.656 ServerApp] jupyterlab | extension was successfully linked.
	[I 2024-01-18 13:23:24.659 ServerApp] notebook | extension was successfully linked.
	[I 2024-01-18 13:23:25.156 ServerApp] notebook_shim | extension was successfully linked.
	[I 2024-01-18 13:23:25.186 ServerApp] notebook_shim | extension was successfully loaded.
	[I 2024-01-18 13:23:25.188 ServerApp] jupyter_lsp | extension was successfully loaded.
	[I 2024-01-18 13:23:25.189 ServerApp] jupyter_server_terminals | extension was successfully loaded.
	[I 2024-01-18 13:23:25.198 LabApp] JupyterLab extension loaded from /home/tobiasst/miniconda3/envs/sc_jupyter/lib/python3.11/site-packages/jupyterlab
	[I 2024-01-18 13:23:25.198 LabApp] JupyterLab application directory is /home/tobiasst/miniconda3/envs/sc_jupyter/share/jupyter/lab
	[I 2024-01-18 13:23:25.199 LabApp] Extension Manager is 'pypi'.
	[I 2024-01-18 13:23:25.201 ServerApp] jupyterlab | extension was successfully loaded.
	[I 2024-01-18 13:23:25.204 ServerApp] notebook | extension was successfully loaded.
	[I 2024-01-18 13:23:25.205 ServerApp] Serving notebooks from local directory: /work/project/tobias/jupyter_test/scJupyter
	[I 2024-01-18 13:23:25.205 ServerApp] Jupyter Server 2.12.5 is running at:
	[I 2024-01-18 13:23:25.205 ServerApp] http://gpu01:8888/tree?token=73d138c645a3f74dffef7e59b9ffb8fa01875416b66330de
	[I 2024-01-18 13:23:25.205 ServerApp]     http://127.0.0.1:8888/tree?token=73d138c645a3f74dffef7e59b9ffb8fa01875416b66330de
	[I 2024-01-18 13:23:25.205 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
	[C 2024-01-18 13:23:25.226 ServerApp] 
	    
	    To access the server, open this file in a browser:
	        file:///home/tobiasst/.local/share/jupyter/runtime/jpserver-46610-open.html
	    Or copy and paste one of these URLs:
	        http://gpu01:8888/tree?token=73d138c645a3f74dffef7e59b9ffb8fa01875416b66330de
	        http://127.0.0.1:8888/tree?token=73d138c645a3f74dffef7e59b9ffb8fa01875416b66330de
	[I 2024-01-18 13:23:25.511 ServerApp] Skipped non-installed server(s): bash-language-server, dockerfile-language-server-nodejs, javascript-typescript-langserver, jedi-language-server, julia-language-server, pyright, python-language-server, python-lsp-server, r-languageserver, sql-language-server, texlab, typescript-language-server, unified-language-server, vscode-css-languageserver-bin, vscode-html-languageserver-bin, vscode-json-languageserver-bin, yaml-language-server

it is important to register three elements, NODE, PORT, TOKEN in the 3rd line from the bottom

	 http://gpu01:8888/tree?token=73d138c645a3f74dffef7e59b9ffb8fa01875416b66330de
 
which contains
 
 	 http://<NODE>:<PORT>/tree?token=<TOKEN>
	 

## Connect to Jupyter server from client computer

in a new client terminal window, connect to the cluster and its node serving the notebook 

	ssh <username>@MBIOHW30.bio.med.uni-muenchen.de -L 127.0.0.1:<PORT>:<NODE>:<PORT> -N
	
replacing \<username> with your HPC user name and \<PORT> & \<NODE> with PORT and NODE obtained from standard out above.

## Open the Notebook on your browser

Open browser and navigate to

	http://127.0.0.1:<PORT>/tree?token=<TOKEN>

replacing \<PORT> & \<TOKEN> with PORT and TOKEN obtained from standard out above.

## Shutdown

Terminate Jupyter notebook server with Ctrl-T in the session that you used to launch the server.

## Troubleshooting

1. make sure you use the correct NODE, PORT, TOKEN parameters

2. in case of connection problems use verbose output on the ssh connection 

		ssh <username>@MBIOHW30.bio.med.uni-muenchen.de -L 127.0.0.1:<PORT>:<NODE>:<PORT> -N -vvv
