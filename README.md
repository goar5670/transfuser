# TransFuser: Imitation with Transformer-Based Sensor Fusion for Autonomous Driving

## [Paper](https://arxiv.org/abs/2205.15997) | [Supplementary]() 

<img src="figures/demo.gif">

This repository contains the code for the paper [TransFuser: Imitation with Transformer-Based Sensor Fusion for Autonomous Driving](https://arxiv.org/abs/2205.15997). This work is a journal extension of the CVPR 2021 paper [Multi-Modal Fusion Transformer for End-to-End Autonomous Driving](https://arxiv.org/abs/2104.09224). If you find our code or papers useful, please cite:

```bibtex
@article{Chitta2022ARXIV,
  author = {Chitta, Kashyap and
            Prakash, Aditya and
            Jaeger, Bernhard and
            Yu, Zehao and
            Renz, Katrin and
            Geiger, Andreas},
  title = {TransFuser: Imitation with Transformer-Based Sensor Fusion for Autonomous Driving},
  journal = {arXiv},
  volume  = {2205.15997},
  year = {2022},
}
```

```bibtex
@inproceedings{Prakash2021CVPR,
  author = {Prakash, Aditya and
            Chitta, Kashyap and
            Geiger, Andreas},
  title = {Multi-Modal Fusion Transformer for End-to-End Autonomous Driving},
  booktitle = {Conference on Computer Vision and Pattern Recognition (CVPR)},
  year = {2021}
}
```


## ToDos

- [x] Autopilot
- [x] Training scenarios and routes
- [x] Longest6 benchmark
- [x] Inference code
- [x] Data generation
- [x] Pretrained agents
- [x] Training script
- [ ] Dataset upload
- [ ] Leaderboard submission instructions
- [ ] Additional tools


## Contents

1. [Setup](#setup)
2. [Training](#training)
2. [Evaluation](#evaluation)


## Setup

Clone the repo, setup CARLA 0.9.10.1, and build the conda environment:

```Shell
git clone https://github.com/autonomousvision/transfuser.git
cd transfuser
git checkout 2022
chmod +x setup_carla.sh
./setup_carla.sh
conda env create -f environment.yml
conda activate tfuse
pip install torch-scatter -f https://data.pyg.org/whl/torch-1.11.0+cu102.html
pip install mmcv-full==1.5.3 -f https://download.openmmlab.com/mmcv/dist/cu102/torch1.11.0/index.html
```

## Training

### Training scenarios and routes
See the [tools/dataset](./tools/dataset) folder for detailed documentation regarding the training routes and scenarios. We will soon upload the dataset used in our paper, which is generated from all the routes in [this folder.](../../leaderboard/data/training/)

### Data generation
We have provided a script for data generation via a privileged agent which we call the autopilot (`/team_code_autopilot/autopilot.py`). To generate data, the first step is to launch a CARLA server:

```Shell
./CarlaUE4.sh --world-port=2000 -opengl
```

Once the CARLA server is running, use the script below for generating training data:
```Shell
./leaderboard/scripts/datagen.sh <carla root> <working directory of this repo (*/transfuser/)>
```

The main variables to set are `SCENARIOS` and `ROUTES`. 

### Training script

The code for training via imitation learning is provided in [train.py.](./team_code_latest/train.py)


## Evaluation

### Longest6 benchmark
We make some minor modifications to the CARLA leaderboard code for the Longest6 benchmark, which are documented [here](./leaderboard). See the [leaderboard/data/longest6](./leaderboard/data/longest6/) folder for a description of Longest6 and how to evaluate on it.

### Pretrained agents
Pre-trained agent files for all 4 methods can be downloaded from [AWS](https://s3.eu-central-1.amazonaws.com/avg-projects/transfuser/models_2022.zip):

```Shell
mkdir model_ckpt
wget https://s3.eu-central-1.amazonaws.com/avg-projects/transfuser/models_2022.zip -P model_ckpt
unzip model_ckpt/models_2022.zip -d model_ckpt/
rm model_ckpt/models_2022.zip
```

### Running an agent
To evaluate a model, we first launch a CARLA server:

```Shell
./CarlaUE4.sh --world-port=2000 -opengl
```

Once the CARLA server is running, evaluate an agent with the script:
```Shell
./leaderboard/scripts/local_evaluation.sh <carla root> <working directory of this repo (*/transfuser/)>
```

By editing the arguments in `local_evaluation.sh`, we can benchmark performance on the Longest6 routes. You can evaluate both privileged agents (such as [autopilot.py]) and sensor-based models. To evaluate the sensor-based models use [submission_agent.py](./team_code_latest/submission_agent.py) as the `TEAM_AGENT` and point to the folder you downloaded the model weights into for the `TEAM_CONFIG`. The code is automatically configured to use the correct method based on the args.txt file in the model folder.

### Parsing longest6 results
To compute additional statistics from the results of evaluation runs we provide a parser script [tools/result_parser.py](./tools/result_parser.py).

```Shell
${WORK_DIR}/tools/result_parser.py --xml ${WORK_DIR}/leaderboard/data/longest6/longest6.xml --results /path/to/folder/with/json_results/ --save_dir /path/to/output --town_maps ${WORK_DIR}/leaderboard/data/town_maps_xodr
```

It will generate a results.csv file containing the average results of the run as well as additional statistics. It also generates town maps and marks the locations where infractions occurred.

<!-- ### Building docker image

Add the following paths to your ```~/.bashrc```
```
export CARLA_ROOT=<path_to_carla_root>
export SCENARIO_RUNNER_ROOT=<path_to_scenario_runner_in_this_repo>
export LEADERBOARD_ROOT=<path_to_leaderboard_in_this_repo>
export PYTHONPATH="${CARLA_ROOT}/PythonAPI/carla/":"${SCENARIO_RUNNER_ROOT}":"${LEADERBOARD_ROOT}":${PYTHONPATH}
```

Edit the contents of ```leaderboard/scripts/Dockerfile.master``` to specify the required dependencies, agent code and model checkpoints. Add all the required information in the area delimited by the tags ```BEGINNING OF USER COMMANDS``` and ```END OF USER COMMANDS```. The current Dockerfile works for all the models in this repository.

Specify a name for the docker image in ```leaderboard/scripts/make_docker.sh``` and run:
```
leaderboard/scripts/make_docker.sh
```

Refer to the Transfuser example for the directory structure and where to include the code and checkpoints.

### Testing the docker image locally

Spin up a CARLA server:
```
SDL_VIDEODRIVER=offscreen SDL_HINT_CUDA_DEVICE=0 ./CarlaUE4.sh -world-port=2000 -opengl
```

Run the docker container:  
Docker 19:  
```
docker run -it --rm --net=host --gpus '"device=0"' -e PORT=2000 <docker_image> ./leaderboard/scripts/run_evaluation.sh
```
If the docker container doesn't start properly, add another environment variable ```SDL_AUDIODRIVER=dsp```.

### Submitting docker image to the leaderboard

Register on [AlphaDriver](https://app.alphadrive.ai/), create a team and apply to the CARLA Leaderboard.

Install AlphaDrive cli:
```
curl http://dist.alphadrive.ai/install-ubuntu.sh | sh -
```

Login to alphadrive and submit the docker image:
```
alpha login
alpha benchmark:submit --split <2/3> <docker_image>
```
Use ```split 2``` for MAP track and ```split 3``` for SENSORS track. -->