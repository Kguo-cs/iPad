Code for paper: "iPad: Iterative Proposal-centric End-to-End Autonomous Driving"


# Install
```bash
conda create -n ipad python=3.8
conda activate ipad
pip install torch==2.1.0 torchvision==0.16.0 torchaudio==2.1.0 --index-url https://download.pytorch.org/whl/cu121
pip install -e ./Bench2DriveZoo
pip install -e ./nuplan-devkit
pip install -e .
```

# Set environment variable
set the environment variable based on where you place the PAD directory. 
```bash
export NUPLAN_MAP_VERSION="nuplan-maps-v1.0"
export NUPLAN_MAPS_ROOT="$HOME/pad_workspace/dataset/maps"
export NAVSIM_EXP_ROOT="$HOME/pad_workspace/exp"
export NAVSIM_DEVKIT_ROOT="$HOME/pad_workspace/navsim"
export OPENSCENE_DATA_ROOT="$HOME/pad_workspace/dataset"
export Bench2Drive_ROOT="$HOME/pad_workspace/Bench2Drive"
```


# Navsim training and evaluation
1. download the navtrain dataset and map as [Navsim](https://github.com/autonomousvision/navsim) 
```bash
bash download/download_maps.sh
bash download/download_navtrain.sh
bash download/download_test.sh
```
Put the downloaded maps in "dataset/maps", and dataset in "dataset/navsim_logs" and "dataset/sensor_blobs" 

2. cache training data and metric
```bash
python navsim/planning/script/run_training_metric_caching.py
python navsim/planning/script/run_dataset_caching.py
```
Change "cache_data=True" in [pad_agent](navsim/agents/pad/pad_agent.py) during running "run_dataset_caching.py".

3. train navsim model
```bash
python navsim/planing/script/run_training.py
```
4. test navsim model

Change the checkpoint path in [agent_config](navsim/planning/script/config/common/agent/navsim_agent.yaml)
```bash
python navsim/planing/script/run_create_submission_pickle.py
```
Then, submit the created "submission.pkl" to the [official leaderboard](https://huggingface.co/spaces/AGC2024-P/e2e-driving-navsim) on HuggingFace.
[Pretrained model](https://drive.google.com/drive/folders/1b8HwN4v4mpM-3T7bUYPwvA9jaMTHCEAu?usp=drive_link)


# Bench2drive
1. download the base dataset as [Bench2Drive](https://github.com/Thinklab-SJTU/Bench2Drive) 
```bash
huggingface-cli download --repo-type dataset --resume-download rethinklab/Bench2Drive --local-dir Bench2Drive-Base
```
2. download and setup CARLA 0.9.15
```bash
    mkdir carla
    cd carla
    wget https://carla-releases.s3.us-east-005.backblazeb2.com/Linux/CARLA_0.9.15.tar.gz
    tar -xvf CARLA_0.9.15.tar.gz
    cd Import && wget https://carla-releases.s3.us-east-005.backblazeb2.com/Linux/AdditionalMaps_0.9.15.tar.gz
    cd .. && bash ImportAssets.sh
    export CARLA_ROOT=YOUR_CARLA_PATH
    echo "$CARLA_ROOT/PythonAPI/carla/dist/carla-0.9.15-py3.7-linux-x86_64.egg" >> YOUR_CONDA_PATH/envs/YOUR_CONDA_ENV_NAME/lib/python3.8/site-packages/carla.pth # python 3.8 also works well, please set YOUR_CONDA_PATH and YOUR_CONDA_ENV_NAME
```
3. [Prepare Dataset](Bench2DriveZoo/docs/DATA_PREP.md)

4. cache training data and metric
```bash
python Bench2Drive/leaderboard/pad_team_code/b2d_datacache.py
python Bench2Drive/leaderboard/pad_team_code/gen_mapinfo.py
```

5. train Bench2drive model
```bash
python navsim/planning/script/run_b2d_training.py
```

6. closeloop evaluation
```bash
cd Bench2Drive
python leaderboard/leaderboard/pad_eval.py
python tools/merge_route_json.py
python tools/ability_benchmark.py
python tools/efficiency_smoothness_benchmark.py
```
