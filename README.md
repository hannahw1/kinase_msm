# kinase_msm
[![Build Status](https://travis-ci.org/msultan/kinase_msm.svg?branch=master)](https://travis-ci.org/msultan/kinase_msm)
Framework for rapid multi-sequence analysis

Kinase_MSM provides a framework for analyzing multi-sequence protein sets. It was originally conceived for handling simulation data from families of kinases. It can be used to build single models from many simulations, project models from one protein to another protein, and more.

Kinase_MSM is intended to work with the following directory structure for input data. The base directory "base_dir" contains subdirectories associated with each protein in the series. Each of these subdirectories contains raw trajectory data associated with a given project. Each protein directory should contain a directory of featurized data, "features_dir", generated via other means.

Example input structure:

+ base_dir 
  + kinase1
    + kinase1_proj_1 
       + trajectories 
    + kinase1_proj_2
       + trajectories 
    + features_dir 
    + protein_only_traj 
  + kinase2
    + kinase2_proj_1 
       + trajectories 
    + features_dir 
    + protein_only_traj TODO: What is protein_only_traj?
  + series.yaml (generated by kinase_msm)

Kinase_MSM will generate a directory to hold output from model analysis, "mdl_dir", in the base_dir, with subdirectories for each protein (along with other data in the mdl dir TODO: what does this mean?).

Example output directory:
  
+ mdl_dir
   + project.yaml #Contains model details
   + kinase1 #subdirectory containing model output for kinase1
   + kinase2

TODO: have setup_sim.py, fit_transform.py, pull_tics.py all be from command line and the following can just be instructions for command line usage

Below is an example script that will generate the necessary framework for model analysis.  In this case, the two proteins being modelled are rcsb_mdl and rcsb_hmdl, and each has two respective sources of data, indicated in projects. Parameters for model fitting are input in the script from the results of an Osprey hyperparameter optimization.
 
``` python
from kinase_msm.series_setup import setup_series_analysis
from kinase_msm.fit_transform_kinase_series import fit_pipeline
import os 

base_dir = "/hsgs/nobackup/msultan/research/kinase/btk_kinase/fah_data/rcsb"

protein_list =["rcsb_mdl", "rcsb_hmdl"]

proj_dict={}

proj_dict={}
proj_dict["rcsb_mdl"] =["PROJ9145","PROJ9151"]
proj_dict["rcsb_hmdl"] = ["PROJ9146","PROJ9152"]

#using her2 mdl osprey results
osprey_stride = 100 

osprey_mdl_params = {'tica__n_components': 5, 'tica__lag_time': 3, 'tica__weighted_transform': True, 'tica__shrinkage': None, 'cluster__n_clusters': 500}


mdl_params = osprey_mdl_params
mdl_params["tica__lag_time"] = mdl_params["tica__lag_time"]*osprey_stride
mdl_params["msm__lag_time"] = 3*osprey_stride

mdl_dir = os.path.join(base_dir,"mdl_dir")
feature_dir = "feature_dir"
series_name = "btk_series"
project_dict = proj_dict

def main():
    yaml_file = setup_series_analysis(base_dir,mdl_dir,feature_dir,series_name,protein_list, proj_dict, mdl_params)
    featurize_series(yaml_file)
    fit_pipeline(base_dir)

if __name__=="__main__":
    main()

```
