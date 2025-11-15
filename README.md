# lerobot-SO-100

<img width="3794" height="1234" alt="image" src="https://github.com/user-attachments/assets/31b09632-cd12-4406-9bda-3a22959f6a47" />

[![Tests](https://github.com/huggingface/lerobot/actions/workflows/nightly.yml/badge.svg?branch=main)](https://github.com/huggingface/lerobot/actions/workflows/nightly.yml?query=branch%3Amain)
[![Python versions](https://img.shields.io/pypi/pyversions/lerobot)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/huggingface/lerobot/blob/main/LICENSE)
[![Status](https://img.shields.io/pypi/status/lerobot)](https://pypi.org/project/lerobot/)
[![Version](https://img.shields.io/pypi/v/lerobot)](https://pypi.org/project/lerobot/)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.1-ff69b4.svg)](https://github.com/huggingface/lerobot/blob/main/CODE_OF_CONDUCT.md)
https://github.com/huggingface/lerobot.git 의 내용을 참고하여 작성하였습니다.

<br/>
<br/>


#### Examples of pretrained models on simulation environments

<table>
  <tr>
    <td><img src="https://raw.githubusercontent.com/huggingface/lerobot/main/media/gym/aloha_act.gif" width="100%" alt="ACT policy on ALOHA env"/></td>
    <td><img src="https://raw.githubusercontent.com/huggingface/lerobot/main/media/gym/pusht_diffusion.gif" width="100%" alt="Diffusion policy on PushT env"/></td>
  </tr>
  <tr>
    <td align="center">ACT policy on ALOHA env</td>
    <td align="center">Diffusion policy on PushT env</td>
  </tr>
</table>

SO-100를 사용하여 ACT,Diffusion simulation을 적용



---


### Install LeRobot 🤗

#### From Source

First, clone the repository and navigate into the directory:

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
```

Then, install the library in editable mode. This is useful if you plan to contribute to the code.

```bash
pip install -e .
```

### Find the USB ports associated with each arm
To find the port for each bus servo adapter, run this script:

```bash
lerobot-find-port
```

### Give permission to port

e.g.
```bash
sudo chmod 666 /dev/ttyACM0
sudo chmod 666 /dev/ttyACM1
```

### calibrate

```bash
python -m lerobot.calibrate --teleop.type=so100_leader --teleop.port=/dev/ttyACM0 --teleop.id=my_awesome_leader_arm
python -m lerobot.calibrate --robot.type=so100_follower --robot.port=/dev/ttyACM1 --robot.id=my_awesome_follower_arm
```

### camera
```bash
python -m lerobot.find_cameras
```

### teleoperate
```bash
python -m lerobot.teleoperate \
--robot.type=so100_follower \
--robot.port=/dev/ttyACM1 \
--robot.id=my_follower_arm \
--teleop.type=so100_leader \
--teleop.port=/dev/ttyACM0 \
--teleop.id=my_leader_arm \
--display_data=true \
--robot.cameras='{
"hand": {"type": "opencv", "index_or_path": 2, "width": 640, "height": 480, "fps": 30},
"top":  {"type": "opencv", "index_or_path": 0, "width": 640, "height": 480, "fps": 30}
}'
```

### record 
```bash
python -m lerobot.record --robot.type=so100_follower --robot.port=/dev/ttyACM1 --robot.cameras="{hand: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}, top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}}" --robot.id=my_awesome_follower_arm --teleop.type=so100_leader --teleop.port=/dev/ttyACM0 --teleop.id=my_awesome_leader_arm --dataset.repo_id=Kimz1/so100-teleop-record-0827-2 --dataset.num_episodes=30 --dataset.single_task="Grab the box" --display_data=true --resume=true
```
처음 recording할때는 resume=true 끄고 진행 (or false로)

### huggingface login/upload
```bash
pip install huggingface_hub
huggingface-cli login
```
```bash
huggingface-cli upload Kimz1/so100-teleop-record-0827-4 /home/default/.cache/huggingface/lerobot/Kimz1/so100-teleop-record-0827-4 --repo-type=dataset
```

---

### train - ACT
```bash
python -m lerobot.scripts.train --dataset.repo_id=Kimz1/so100-teleop-record-0826-1 --policy.type=act --output_dir=outputs/train/act_so100-0826-1 --job_name=act_so100-0826-1 --policy.device=cuda --wandb.enable=true --steps=60000 --save_checkpoint=true --save_freq=10000 --resume=false --policy.repo_id=Kimz1/act-so100-policy-0826-1 --dataset.video_backend=pyav
```

### train - Diffusion
```bash
python -m lerobot.scripts.train \
--dataset.repo_id=Kimz1/so100-teleop-record-0827-4 \
--policy.type=diffusion \
--output_dir=outputs/train/diffusion_so100-0827-4 \
--job_name=diffusion_so100-0827-4 \
--policy.repo_id=Kimz1/diffusion-so100-policy-0827-4 \
--policy.n_obs_steps=2 \
--policy.n_action_steps=8 \
--policy.device=cuda \
--wandb.enable=true \
--steps=40000 \
--save_checkpoint=true \
--save_freq=10000 \
--resume=false \
--dataset.video_backend=pyav
```

### eval
```bash
python -m lerobot.record \
--robot.type=so100_follower \
--robot.port=/dev/ttyACM1 \
--robot.cameras="{hand: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}, top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}}" \
--robot.id=my_awesome_follower_arm \
--dataset.repo_id=Kimz1/eval_so100-teleop-record-0828-1 \
--policy.path=Kimz1/act-so100-policy-0828-1 \
--dataset.num_episodes=0 \
--dataset.single_task="Grab the box" \
--display_data=true
```
```bash
python -m lerobot.record \
--robot.type=so100_follower \
--robot.port=/dev/ttyACM1 \
--robot.cameras="{hand: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}, top: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}}" \
--robot.id=my_awesome_follower_arm \
--dataset.repo_id=Kimz1/eval_so100-teleop-record-0827-4 \
--policy.path=Kimz1/diffusion-so100-policy-0827-4-obs4-act8 \
--dataset.num_episodes=10 \
--dataset.single_task="Grab the box" \
--display_data=true
```

### video download
```bash
cp /home/default/.cache/huggingface/lerobot/Kimz1/so100-teleop-record-0825-1cam/videos/chunk-000/observation.images.hand/episode_000020.mp4 ~/Downloads/
```

<br/>

---


### Result
<br/>
ACT와 Diffusion로 train후 eval하는 영상입니다. 더 많은 video와 data는 아래 링크를 통해 확인하실 수 있습니다.

#### ACT

https://huggingface.co/datasets/Kimz1/eval_so100-teleop-record-0827-5

https://github.com/user-attachments/assets/6b789761-095e-49c8-b046-efa475ba6c64

<br/>

#### Diffusion

https://huggingface.co/datasets/Kimz1/so100-teleop-record-0827-4

https://github.com/user-attachments/assets/d5b6b8fe-ac19-47dd-b5a8-2b79d091fec7

