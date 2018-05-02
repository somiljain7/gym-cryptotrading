# Gym CryptoTrading Environment

[![license](https://img.shields.io/packagist/l/doctrine/orm.svg)](https://github.com/samre12/deep-trading-agent/blob/master/LICENSE)
[![dep2](https://img.shields.io/badge/python-2.7-red.svg)](https://www.python.org/download/releases/2.7/)
[![dep3](https://img.shields.io/badge/status-in%20progress-green.svg)](https://github.com/samre12/gym-cryptotrading/)
[![dep4](https://img.shields.io/circleci/project/github/RedSparr0w/node-csgo-parser.svg)](https://github.com/samre12/gym-cryptotrading/)

Gym Environment API based Bitcoin trading simulator with continuous observation space and discrete action space. It uses real world transactions from **CoinBaseUSD** exchange to sample *per minute closing, lowest and highest prices along with volume of the currency traded* in the particular minute interval.

**Contents of this document**

- [Installation](#installation)
- [Usage](#usage)
- [Basics](#basics)
    - [Obsevation Space](#obs)
    - [Action Space](#action)
    - [Parameters](#params)
    - [Simulator](#simulator)
- [Important Information](#inf)
- Environments
    - [Realized PnL Environment](https://github.com/samre12/gym-cryptotrading/wiki/Realized-PnL-Trading-Environment)
    - [Unrealized PnL Environment](https://github.com/samre12/gym-cryptotrading/wiki/Unrealized-PnL-Trading-Environment)
    - [Weighted Unrealized PnL Environment](https://github.com/samre12/gym-cryptotrading/wiki/Weighted-Unrealized-PnL-Trading-Environment)
- [Examples](#exp)
- [Recent Updates and Breaking Changes](#changes)

<a name="introduction"></a>

## Installation

```bash
git clone https://github.com/samre12/gym-cryptotrading.git
cd gym-cryptotrading
pip install -e .
```

<a name="usage"></a>

## Usage

Importing the module into the current session using `import gym_cryptotrading` will register the environment with `gym` after which it can be used as any other gym environment.

### Environments

- `'RealizedPnLEnv-v0'`

- `'UnRealizedPnLEnv-v0'`

- `'WeightedPnLEnv-v0'`

```python
import gym
import gym_cryptotrading
env = gym.make('RealizedPnLEnv-v0')
```

- Use `env.reset()` to start a new random episode.

    - returns history of observations prior to the starting point of the episode. Look [Parameters](#params) for more information.

    ```python
    state = env.reset() # use state to make initial prediction
    ```

    **Note:** Make sure to reset the environment before first use else `gym.error.ResetNeeded()` will be raised.

- Use `env.step(action)` to take one step in the environment.

    - returns `(observation, reward, is_terminal)` in respective order

    ```python
    observation, reward, is_terminal = env.step(action)
    ```

    **Note:** Calling `env.step(action)` after the terminal state is reached will raise `gym.error.ResetNeeded()`.

- With the current implementation, the environment does not support `env.render()`.

Setting the logging level of `gym` using `gym.logger.set_level(level)` to a value less than or equal 10 will allow to track all the logs (`debug` and `info` levels) generated by the environment.</br>
These include human readable timestamps of Bitcoin prices used to simulate an episode.</br>
For information, visit [**here**](https://github.com/openai/gym/blob/293eea787a662f501b0e4aab512d3769e830ece2/gym/logger.py#L11) .


<a name="basics"></a>

## Basics

<a name="obs"></a>

### Observation Space

- Observation at a time step is the relative `(closing, lowest, highest, volume)` of Bitcoin in the corresponding minute interval.

- Since the price of Bitcoin varies from a few dollars to 15K dollars, the observation for time step i + 1 is normalized by the prices at time instant i.

Each entry in the observation is the ratio of *increase (value greater than 1.0)* or *decrease (value lessar than 1.0)* from the price at previos time instant.

<a name="action"></a>

### Action Space

At each time step, the agent can either go **LONG** or **SHORT** in a `unit` (for more information , refer to [Parameters](#params)) of Bitcoin or can stay **NEUTRAL**.</br>
Action space thus becomes *discrete* with three possible actions:

- `NEUTRAL` corresponds to `0`

- `LONG` corresponds to `1`

- `SHORT` corresponds to `2`

**Note:** Use `env.action_space.get_action(action)` to lookup action names corresponding to their respective values.

<a name="params"></a>

### Parameters

The basic environment is characterized with these parameters:

- `history_length` lag in the observations that is used for the state representation of the trading agent.</br>

    - every call to `env.reset()` returns a numpy array of shape `(history_length,) + shape(observation)` that corresponds to observations of length `history_length` prior to the starting point of the episode.

    - trading agent can use the returned array to predict the first action

    - defaults to `100`.

    - supplied value must be greater than or equal to `0`

- `horizon` alternatively **episode length** is the number trades that the agent does in a single episode

    - defaults to `5`.

    - supplied value must be greater than `0`

- `unit` is the fraction of Bitcoin that can be traded in each time step

    - defaults to `5e-4`.

    - supplied value must be greater than `0`

### Usage

```python
env = gym.make('RealizedPnLEnv-v0')
env.env.set_params(history_length, horizon, unit)
```

**Note:** parameters can only be set before first reset of the environment, that is, before the first call to `env.reset()`, else `gym_cryptotrading.errors.EnvironmentAlreadyLoaded` will be raised.

Some environments contain their own specific parameters due to the nature of their reward function.</br>
These parameters can be passed using `env.env.set_params(history_length, horizon, unit, **kwargs)` as keyworded arguements alongside setting *history length*, *horizon* and *unit*.

<a name="simulator"></a>

### Simulator

- Dataset for per minute prices of Bitcoin is not continuos and compute due to the downtime of the exchanges.

- Current implementation does not make any assumptions about the missing values.

- It rather finds continuos blocks with lengths greater than `history_length + horizon + 1` and use them to simulate episodes. This avoids any discrepancies in results due to random subsitution of missing values

<a name="inf"></a>

## Important Information

Upon first use, the environment downloads latest transactions dataset from the exchange which are then cached in *tempory directory* of the operating system for future use.</br>

- A user can also update the latest transactions dataset by calling `gym_cryptotrading.Generator.update_gen()` **prior** to making the environment.

    - Updating the latest transactions won't reflect in environments made earlier.

- If you are running the environment behind a proxy, export suitalble **http proxy settings** to allow the environment to download transactions from the exchange

<a name="exp"></a>

## Examples
Coming soon.

<a name="changes"></a>

## Recent Updates and Breaking Changes

Listing changes from [**`b9af98db728230569a18d54dcfa87f7337930314`**](https://github.com/samre12/gym-cryptotrading/commit/b9af98db728230569a18d54dcfa87f7337930314) commit. Visit [**here**](https://github.com/samre12/gym-cryptotrading/tree/b9af98db728230569a18d54dcfa87f7337930314) to browse the repository with head at this commit.

- Added support for trading environments with **Realized PnL** and **Weighted Unrealized PnL** reward functions

- Renamed `cryptotrading.py` to `unrealizedPnL.py` to emphasize the specific reward function of the environment

### Breaking Changes

- Environment with **Unrealized PnL** reward function is now built using `env = gym.make('UnrealizedPnLEnv-v0')` rather than `env = gym.make('CryptoTrading-v0')`