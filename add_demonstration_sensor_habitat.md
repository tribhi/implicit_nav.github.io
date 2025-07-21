# Adding a Custom Task Sensor in Habitat 0.3.1

This document describes how to add a custom task sensor (DemonstrationSensor) to Habitat.

---

## 1. Add the sensor definition to `object_nav_task.py`

Insert the following code to define and register the sensor:

```python
from habitat.core.registry import registry
from habitat.core.simulator import Sensor
from gym import spaces
from habitat.tasks.nav.nav import EmbodiedTask
from habitat.core.simulator import Observations
from typing import Any, Dict

@registry.register_sensor(name="DemonstrationSensor")
class DemonstrationSensor(Sensor):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self._uuid = "demonstration"
        self._observation_space = spaces.Discrete(1)
        self.timestep = 0

    def _get_uuid(self, *args: Any, **kwargs: Any) -> str:
        return self._uuid

    def _get_observation(
        self, observations: Dict[str, Observations], episode, task: EmbodiedTask, **kwargs
    ):
        if self.episode_over:
            self.timestep = 0

        if hasattr(episode, "reference_replay") and self.timestep < len(episode.reference_replay):
            action = episode.reference_replay[self.timestep].action
        else:
            action = 0

        self.timestep += 1
        return action
```

---

## 2. Add the sensor config to `default/structured_configs.py`

Add the sensor config and register it:

```python
from dataclasses import dataclass
from habitat.config.default_structured_configs import LabSensorConfig
from hydra.core.config_store import ConfigStore

cs = ConfigStore.instance()

@dataclass
class DemonstrationSensorConfig(LabSensorConfig):
    type: str = "DemonstrationSensor"


cs.store(
    package="habitat.task.lab_sensors.demonstration_sensor",
    group="habitat/task/lab_sensors",
    name="demonstration_sensor",
    node=DemonstrationSensorConfig,
)
```

---

## 3. Add the sensor to the `config/habitat/task/objectnav.yaml` file

Update the YAML to include the new sensor:

```yaml
defaults:
  - task_config_base
  - actions:
    - stop
    - move_forward
    - turn_left
    - turn_right
    - look_up
    - look_down
  - measurements:
    - distance_to_goal
    - success
    - spl
    - soft_spl
    - distance_to_goal_reward
  - lab_sensors:
    - objectgoal_sensor
    - compass_sensor
    - gps_sensor
    - demonstration_sensor
  - _self_

actions:
  look_up:
    tilt_angle: 30
  look_down:
    tilt_angle: 30

type: ObjectNav-v1
end_on_success: True
reward_measure: "distance_to_goal_reward"
success_measure: "spl"


goal_sensor_uuid: objectgoal

measurements:
  distance_to_goal:
    distance_to: VIEW_POINTS
  success:
    success_distance: 0.1

```
