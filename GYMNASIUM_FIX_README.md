# Gymnasium API Compatibility Fix

## Problem Description

The training script `train_gym.py` was failing with a `ValueError: too many values to unpack (expected 4)` error when running with Gymnasium v1.2.0.

### Root Cause
The issue occurred because of API changes between the old OpenAI Gym and the newer Gymnasium library:

- **Old Gym API**: `env.step(action)` returns 4 values: `(observation, reward, done, info)`
- **New Gymnasium API**: `env.step(action)` returns 5 values: `(observation, reward, terminated, truncated, info)`

The codebase was written for the old API but was running with Gymnasium v1.2.0, causing the unpacking error.

## Error Stack Trace
```
ValueError: too many values to unpack (expected 4)
  File "/Users/mustafa/Desktop/wvr/CloseAirCombatPettingZoo/scripts/train/train_gym.py", line 36, in step
    observation, reward, done, info = self.env.step(action)
```

## Solution

We updated three key locations in the codebase to handle both the old and new APIs:

### 1. GymEnv.step() method in `train_gym.py`
**File**: `/scripts/train/train_gym.py` (lines 34-47)

```python
def step(self, action):
    action = np.array(action).reshape(self.action_shape)
    result = self.env.step(action)
    if len(result) == 5:
        # New Gymnasium API: observation, reward, terminated, truncated, info
        observation, reward, terminated, truncated, info = result
        done = terminated or truncated
    else:
        # Old Gym API: observation, reward, done, info
        observation, reward, done, info = result
    observation = np.array(observation).reshape((1, -1))
    done = np.array(done).reshape((1,-1))
    reward = np.array(reward).reshape((1, -1))
    return observation, reward, done, info
```

### 2. GymHybridEnv.step() method in `train_gym.py`
**File**: `/scripts/train/train_gym.py` (lines 73-89)

```python
def step(self, action):
    action = np.array(action).reshape(self.action_shape)
    discrete_a, continuous_a = action[:self.discrete_dims].astype(np.int32), action[self.discrete_dims:]
    action = (discrete_a, continuous_a)
    result = self.env.step(action)
    if len(result) == 5:
        # New Gymnasium API: observation, reward, terminated, truncated, info
        observation, reward, terminated, truncated, info = result
        done = terminated or truncated
    else:
        # Old Gym API: observation, reward, done, info
        observation, reward, done, info = result
    observation = np.array(observation).reshape((1, -1))
    done = np.array(done).reshape((1,-1))
    reward = np.array(reward).reshape((1, -1))
    return observation, reward, done, info
```

### 3. step_env function in `env_wrappers.py`
**File**: `/envs/env_wrappers.py` (lines 191-213)

```python
def step_env(env, action):
    result = env.step(action)
    if len(result) == 5:
        # New Gymnasium API: observation, reward, terminated, truncated, info
        obs, reward, terminated, truncated, info = result
        done = terminated or truncated
    else:
        # Old Gym API: observation, reward, done, info
        obs, reward, done, info = result
        
    if 'bool' in done.__class__.__name__:
        if done:
            obs = env.reset()
    elif isinstance(done, (list, tuple, np.ndarray)):
        if np.all(done):
            obs = env.reset()
    elif isinstance(done, dict):
        if np.all(list(done.values())):
            obs = env.reset()
    else:
        raise NotImplementedError("Unexpected type of done!")
    return obs, reward, done, info
```

## Key Changes

1. **Detect API version**: Check if `env.step()` returns 5 values (new API) or 4 values (old API)
2. **Handle both cases**: Unpack the correct number of values based on the detected API version
3. **Combine terminated and truncated**: In the new API, combine `terminated` and `truncated` into a single `done` boolean using logical OR
4. **Maintain backward compatibility**: The fix works with both old Gym and new Gymnasium versions

## Environment Details

- **Gymnasium Version**: v1.2.0
- **Python Version**: 3.13
- **Operating System**: MacOS
- **Shell**: zsh 5.9

## Testing

The fix was tested by running:
```bash
python train_gym.py
```

The script now runs successfully without ValueError exceptions and shows proper training progress:
```
choose to use cpu...
Scenario CartPole-v1 Algo ppo Exp check updates 0/12500 episodes, total num timesteps 800/10000000, FPS 3643.
average episode rewards is 25.0
...
```

## Impact

This fix ensures the codebase works with modern Gymnasium environments while maintaining compatibility with older Gym versions. The training pipeline now runs smoothly with the latest Gymnasium API.