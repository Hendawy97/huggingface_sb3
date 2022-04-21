# Hugging Face 🤗 x Stable-baselines3 v2.0

A library to load and upload Stable-baselines3 models from the Hub.

## Installation
### With pip
```
pip install huggingface-sb3
```

## Examples
We wrote a tutorial on how to use 🤗 Hub and Stable-Baselines3 [here](https://github.com/huggingface/huggingface_sb3/blob/main/Stable_Baselines_3_and_Hugging_Face_%F0%9F%A4%97_tutorial.ipynb)

If you use **Colab or a Virtual/Screenless Machine**, you can check Case 3 and Case 4.

### Case 1: I want to download a model from the Hub
```python
import gym

from huggingface_sb3 import load_from_hub
from stable_baselines3 import PPO
from stable_baselines3.common.evaluation import evaluate_policy

# Retrieve the model from the hub
## repo_id = id of the model repository from the Hugging Face Hub (repo_id = {organization}/{repo_name})
## filename = name of the model zip file from the repository
checkpoint = load_from_hub(
    repo_id="sb3/demo-hf-CartPole-v1",
    filename="ppo-CartPole-v1.zip",
)
model = PPO.load(checkpoint)

# Evaluate the agent and watch it
eval_env = gym.make("CartPole-v1")
mean_reward, std_reward = evaluate_policy(
    model, eval_env, render=False, n_eval_episodes=5, deterministic=True, warn=False
)
print(f"mean_reward={mean_reward:.2f} +/- {std_reward}")
```

### Case 2: I trained an agent and want to upload it to the Hub
With `package_to_hub()` **we'll save, evaluate, generate a model card and record a replay video of your agent before pushing the repo to the hub**.
It currently **works for Gym and Atari environments**. If you use another environment, you should use `push_to_hub()` instead.

First you need to be logged in to Hugging Face:
- If you're using Colab/Jupyter Notebooks:
```python
from huggingface_hub import notebook_login
notebook_login()
```
- Else:
```
huggingface-cli login
```
Then

**With `package_to_hub()`**:

```python
import gym

from stable_baselines3 import PPO
from stable_baselines3.common.vec_env import DummyVecEnv
from huggingface_sb3 import load_from_hub, package_to_hub

# Create the environment
env_id = 'LunarLander-v2'
env = DummyVecEnv([lambda: gym.make(env_id)])

# Create the evaluation env
eval_env = DummyVecEnv([lambda: gym.make(env_id)])

# Instantiate the agent
model = PPO('MlpPolicy', env, verbose=1)

# Train the agent
model.learn(total_timesteps=int(5000))

# This method save, evaluate, generate a model card and record a replay video of your agent before pushing the repo to the hub
package_to_hub(model=model, 
               model_name="ppo-LunarLander-v2",
               model_architecture="PPO",
               env_id=env_id,
               eval_env = eval_env,
               repo_id="ThomasSimonini/TEST2-Colab-ppo-LunarLander-v2",
               commit_message="Test commit")
```


**With `push_to_hub()`**:
```python
from huggingface_sb3 import push_to_hub
from stable_baselines3 import PPO

env = make_atari_env('SpaceInvadersNoFrameskip-v4', n_envs=1, seed=0)
env = VecFrameStack(env, n_stack=4)

# Define a PPO model with CNN policy network
model = PPO("CnnPolicy", "CartPole-v1", verbose=1)

# Train it for 10000 timesteps
model.learn(total_timesteps=10_000)

# Save the model
model.save("ppo-CartPole-v1")

# Push this saved model to the hf repo
# If this repo does not exists it will be created
## repo_id = id of the model repository from the Hugging Face Hub (repo_id = {organization}/{repo_name})
## filename: the name of the file == "name" inside model.save("ppo-CartPole-v1")
push_to_hub(
    repo_id="sb3/demo-hf-CartPole-v1",
    filename="ppo-CartPole-v1.zip",
    commit_message="Added Cartpole-v1 model trained with PPO",
)
```
### Case 3: I use Google Colab with Classic Control/Box2D Gym Environments
- You can use xvbf (virtual screen)
```
!apt-get install -y xvfb python-opengl > /dev/null 2>&1
```
- Just put your code inside a python file and run
```
!xvfb-run -s "-screen 0 1400x900x24" <your_python_file>
```

### Case 4: I use a Virtual/Remote Machine
- You can use xvbf (virtual screen)

```
xvfb-run -s "-screen 0 1400x900x24" <your_python_file>
```
