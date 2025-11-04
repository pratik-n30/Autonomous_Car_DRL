# **Training and Fine-Tuning a Donkey Car Agent with PPO**

## **Aim**

This project details the complete two-stage process of training an autonomous driving agent using Deep Reinforcement Learning in the Donkey Car simulator:

1. **Initial Training:** To train a foundational PPO agent from scratch on a basic, procedurally generated road environment (donkey-generated-roads-v0). The resulting model is saved as genroads.zip.  
2. **Transfer Learning:** To fine-tune the pre-trained genroads model on new, more complex environments, adapting its knowledge to new challenges. The final models can be saved with new names.

## **Requirements**

* gym  
* numpy  
* stable-baselines3  
* Donkey Car Simulator (Windows/Linux executable)

## **Algorithm: PPO (Proximal Policy Optimization)**

This project uses PPO from the stable-baselines3 library. PPO is a state-of-the-art, on-policy algorithm that is highly effective for tasks with continuous action spaces and high-dimensional (pixel) inputs. It has clipped objective function which avoids sudden fluctuations in the policy.

## **Core Environment Details**

Both training phases use the same underlying observation, action, and reward structure, differing only in the track generation.

### **State Space**

The agent's observation is the front-facing camera image (80x60x3 pixels). This raw pixel data is processed by a Convolutional Neural Network (CNN) within the PPO policy, allowing the agent to learn to drive directly from vision.

### **Action Space**

The action space is continuous, consisting of a 2-element array:

* steering: A value between \-1 (full left) and 1 (full right).  
* throttle: A value between 0 (no throttle) and 1 (full throttle).

### **Custom Reward Function**

To encourage "good driving," a custom reward function is implemented:

#### **Centering**

Provides a high reward (max 2.0) for staying in the center of the track (low Cross-Track Error).

#### **Speed**

Rewards higher speeds, but this reward is scaled by the centering reward. This prevents the agent from learning to go fast off-track.

#### **Smoothness**

Applies a small penalty based on the change in steering from the previous step to discourage jerky, erratic movements.

#### **Crash Penalty**

Applies a large negative penalty (-5.0) if the agent collides with an object.

## **The Learning Process**

### **Phase 1: Initial Training (Model: genroads.zip)**

The first phase involves training a model from a random policy to learn basic driving skills.

#### **Environment**

donkey-generated-roads-v0

#### **Goal**

Learn to stay on the road and manage throttle.

#### **Process**

1. A new PPO model with a CnnPolicy is created.  
2. The model is trained from scratch on the donkey-generated-roads-v0 environment for a significant number of timesteps \- 30,000.  
3. The final foundational model is saved as genroads.zip.

### **Phase 2: Fine-Tuning (e.g., Model: gentracks.zip)**

The second phase uses transfer learning to adapt the foundational model to new, unseen environments.

#### **Environments**

This can be donkey-generated-track-v0 or other built-in tracks, such as:

* donkey-warehouse-v0  
* donkey-avc-sparkfun-v0  
* donkey-roboracingleague-track-v0

#### **Goal**

Apply existing knowledge to a new environment and quickly adapt.

#### **Process**

1. The genroads.zip model from Phase 1 is loaded.  
2. The model's environment is set to the new target (e.g., donkey-generated-track-v0).  
3. Training is *continued* (not reset) for an additional number of timesteps (e.g., 20,000).  
4. The newly fine-tuned model is saved with a new name (e.g., gentracks.zip, genwarehouse.zip, etc.).

This transfer learning approach significantly speeds up training for the new track, as the agent does not need to re-learn basic concepts like "stay on the road."

## **Results**

