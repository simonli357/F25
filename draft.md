# Learning-based traversability mapping for navigation

## Proposed Approach
### main idea
- develop unified framework that combines classical mapping (elevation_mapping_cupy) and learning-based mapping to produce a better semantic + elevation map for navigation.
- use mem to produce a semantic + elevation layer in the elevation map using the sensor data.
- feed that into a learning-based network to output an improved semantic + elevation layer, which solves the issues mentioned in motivation.

## Background & Motivation
### existing works in traversability mapping/navigation
- 3 main types of approaches used for determining traversability of terrain for navigation:
1) geometric approaches: use geometric properties like elevation, slope, occupancy. typically use lidars or depth cameras.
2) semantic approaches: determine traversability based on terrain semantics (grass, road, sidewalk, bush, etc.), mostly vision-based (RGB cameras).
3) physical property approaches: estimate physical properties like friction, compliance, texture. mostly proprioception-based (IMU, force, vibration sensors). recent approaches also use vision (RGB cameras) to estimate physical properties. proprioception-based approaches require traversal of terrain to estimate properties, so not suitable for long-range planning. vision-based approaches can estimate physical properties from a distance, but may not be accurate.
- some approaches combine multiple types of info, e.g. geometric + semantic, semantic + physical properties, etc.
- 2 types of environments: urban and off-road. most works focus on one or the other, but none combine both.
- some approaches work on legged robots, some on wheeled robots. Most require specific sensor setups; few are sensor-agnostic.
- autonomous driving, some works use multiple cameras to build a bird's eye view (BEV) semantic map for navigation, but assume flat ground, so no elevation map.
- some works like bevformer use temporal info from multiple frames to generate 3d bounding boxes in BEV.
### Motivation
- develop framework that combines geometric + semantic for traversability mapping, to work in both urban and off-road environments.
- framework should be sensor-agnostic and work with different sensor setups (various combinations of cameras, lidar, depth camera).
- framework should work on both legged and wheeled robots.
- framework should be able to map environments with varying terrain types, weather & lighting conditions, and dynamic obstacles.
- tried elevation_mapping_cupy (MEM) to build a 3D semantic-elevation map from lidar and depth camera and semantic segmentation images
- works reasonably well for static environments, but has issues with dynamic obstacles (cars, pedestrians) and some artifacts in the elevation map. updates for moving objects lagging behind when using 'average' or 'bayesian' or 'exponential' fusion methods.
- if no fusion method and use latest, the elevation map is very noisy and has many outliers.
- also many areas occluded due to limited camera views.
- want to see if can use learning-based approach to improve the quality of the semantic-elevation map.

## Methodology
### framework overview
- setup multiple rgb & depth cameras (e.g. front, left, right, back) and lidars on the robot.
- use real-time model to perform semantic segmentation on rgb images from multiple cameras.
- leverage MEM to build elevation map from sensor data. map contains 2 layers: elevation layer (from lidar + depth camera) and semantic layer (from semantic images from multiple cameras).
- feed the semantic + elevation layer from MEM into a learning-based network to output an improved semantic + elevation layer.
- generate traversability cost map from the improved semantic + elevation layer for planning.
- use a planner to plan paths using the traversability cost.
- use mpc to track the planned path. mpc outputs high-level velocity commands to the robot. so it works for both legged and wheeled robots.
### data collection and training pipeline
1) Leverage Carla Simulator. Although made for autonomous driving research, offers diverse environments (urban, suburban, rural) and dynamic objects (cars, pedestrians). Can simulate various weather and lighting conditions. Supports multiple sensors: RGB cameras, depth cameras, lidars. Carla provides ground truth semantic segmentation and depth images, useful for training.
2) Collect data using 4 cameras (front, left, right, back), depth camera in carla. carla provides ground truth semantic segmentation images, depth images, and lidar point clouds. Additionally mount a bev semantic camera and bev depth camera to get ground truth semantic and depth images in bird's eye view. use carla's built-in vehicle to drive around and collect data in urban environments.
3) use rgb images from 4 cameras to train a semantic segmentation network (e.g. unet, PIDNet, etc.) to perform real-time semantic segmentation on rgb images from 4 cameras. use carla's ground truth semantic images as labels. train on various weather and lighting conditions to improve robustness.
4) To produce the training data (inputs) to the learning-based network:
- use ground truth semantic images from 4 mounted cameras and ground truth depth images from depth camera collected from 1).
- use elevation_mapping_cupy (MEM) to build the elevation layer in the elevation map from lidar and depth camera data.
- use elevation_mapping_cupy (MEM) to build the semantic layer in the elevation map from ground truth semantic images from 4 mounted cameras. Elevation mapping already projects cells to camera frame and does raycasting to fill in cells, so can directly use the semantic images as color images to build the semantic layer in the elevation map.
- input to the network will be the semantic + elevation layer from the elevation map. data format/encoding not decided yet, discussed in network architecture section. 
- preferably could add temporal info by stacking multiple frames together.
4) To produce the ground truth to the network:
- generate pseudo ground truth elevation+semantic grid map using MEM also, but using semantic and depth cameras from better angles: 4 cameras mounted higher up and angled downwards to cover a larger area in front of the vehicle. this will give better coverage and less occlusions. use ground truth semantic images from these 4 cameras and ground truth depth images from bev depth camera to build the pseudo ground truth elevation+semantic grid map using MEM.
### network architecture design:
#### requirements:
1) lightweight and can run in real-time.
2) can incorporate temporal info.
3) want it to predict occluded regions as well as regions cameras can't see. so kind of like image/video inpainting or restoration.
4) want it to have filtering/denoising capabilities to improve quality of elevation map, so kind of like image/video denoising or super-resolution.
#### input data format/encoding:
- input is a grid map (elevation + semantic layer from mem), output is same.
- so input can be treated like an image with multiple channels, number of pixels = number of cells in elevation map (e.g. 4 cm resolution & 10m square region will be a 250x250 image).
- one-hot encoding for semantic classes + elevation channel. number of channels = number of semantic classes + 1 (for elevation). data in semantic channels could be binary or a probability distribution (softmax). Pros: makes it explicit that semantic data is categorical, which may be easier for the network to learn. Cons: more channels -> moderate increase in computational cost.
#### network architecture:
- U-Net with a Convolutional LSTM (ConvLSTM) bottleneck. This architecture treats the problem as a map-to-map translation task, taking a sequence of noisy, incomplete maps and producing a single, clean, and predictive map.
- U-Net for Spatial Reasoning (Denoising & Inpainting): The U-Net's encoder-decoder structure is essential for spatial reasoning, even though the input is already segmented. It is not used for re-segmenting.
- The encoder path (down-sampling) progressively increases the receptive field, allowing the network to capture large-scale spatial context. This is crucial for inpainting occluded regions by making educated predictions based on the surrounding environment (e.g., filling a gap between two patches of "road").
- The decoder path (up-sampling), combined with skip connections, reconstructs a high-resolution map. This allows the network to perform denoising and filtering (e.g., removing elevation outliers) while preserving sharp, fine-grained details like curbs and road boundaries.
- ConvLSTM for Temporal Reasoning (Handling Dynamic Objects): To handle dynamic objects and their lagging artifacts in the MEM, a ConvLSTM module is placed at the bottleneck of the U-Net. Instead of simply stacking frames, which teaches the network a fixed pattern, the ConvLSTM learns a flexible state transition model. It processes the sequence of input maps frame-by-frame, maintaining an internal memory (hidden state). Allows the network to understand motion and distinguish between static and dynamic elements. It can learn to "remember" the static background that was temporarily covered by a moving car or pedestrian, effectively removing the "ghost" artifacts and predicting the true, underlying traversable scene.
- Overall Data Flow: A sequence of T maps is fed into the model. The U-Net encoder extracts features for each map. The ConvLSTM processes this sequence of features to produce a single, temporally-fused feature map. The U-Net decoder then upsamples this map to produce the final, corrected output for the latest timestep.
- Real-time Performance: network is designed to be lightweight. complexity (number of channels, depth of the U-Net) can be easily adjusted to strike balance between predictive accuracy and real-time performance on target hardware.

## Why this approach?
- MEM combines semantic + geometric info and builds 2.5D BEV grid map, which is useful for navigation. Good starting point. 
- Problems with MEM:
1) filters and fusion techniques in MEM works well for static environments, but has issues when dynamic objects are present, as explained. updates for moving objects lagging behind when using 'average' or 'bayesian' or 'exponential' fusion methods. if no fusion method and use latest, the elevation map is very noisy and has many outliers. 
2) many areas occluded due to limited camera views.
3) relies on classical filtering and fusion techniques, which may not be optimal. needs to combine multiple filtering and fusion techniques to get best results.
- hoping that the learning-based network can solve the above issues:
1) learn filtering technique that works well for environments with both static and dynamic objects. can learn to identify dynamic objects and update their elevation and semantics more quickly. can also learn to filter out noise and outliers in the elevation map.
2) learn to fill in missing data and occlusions in the semantic and elevation layers, using context from surrounding areas and temporal info from multiple frames. for example if there's a car or an object, using MEM will predict a thin elevation and semantic layer for the car or object, but cannot determine how thick the car or object is. hope the network can learn this kind of context from the ground truth data (e.g. how thick a car is compared to a pedestrian, etc.). and output a more complete elevation and semantic layer. MEM's raycasting and projection from cell to image works fine for moderate elevation, but in higher elevation areas, occlusions and distortions can cause issues.
- System is multi-modal and sensor agnostic. for example if only have 3 cameras and depth camera, or only have 4 cameras and lidar, or only have 4 cameras, lidar and depth camera, and any of these but with the sensors placed at different positions, the system can still work. the network can learn to be robust to different sensor configurations and noise in the input data. during training, can modify sensor configurations to improve robustness. also can use different resolutions for input and gt. The proposal is largely to use the output of the mapping algorithm (MEM) as the network input. This has the benefit of sensor-agnosticism, as long as the mapping algorithm can ingest various sensor setups and output a consistent grid, the network doesn’t need to change.
- also this approach still allows it to be used on both mobile and legged robots with various sensor setups.
- mem already sends the data to gpu, so can directly run the model as a plugin in mem, and output the improved elevation+semantic layer to the elevation map. so can still do real-time mapping and planning.
- no need to worry about sim2real gap since using lidar and depth camera to build elevation map, and semantic segmentation network can be trained on real data (cityscapes, etc.).

## Experiments & Evaluation
### experiment 1: against baseline MEM
- compare semantic & rgb layer output from network vs from raw MEM & MEM with elevation inpainting plugin.
- metrics:
1) qualitative: visualize the semantic + elevation layer from network vs from MEM. see if network can fill in occluded areas better, remove dynamic object artifacts, and produce cleaner elevation maps.
2) quantitative:
- for semantic layer: use metrics like mean IoU, pixel accuracy, etc.
- for elevation layer: use metrics like RMSE, MAE, etc. compared to pseudo ground truth elevation map.
### experiment 2: ablation study
- ablate different components of the network to see their impact on performance.
1) remove ConvLSTM and just stack frames as input. see if temporal reasoning helps.
2) vary number of input frames (e.g. 1, 3, 5, 7) to see how much temporal context is needed.
3) vary U-Net complexity (number of channels, depth) to see trade-off between accuracy and real-time performance.
### experiment 3: generalization to unseen environments
- test the trained network on unseen environments in Carla (different towns, weather, lighting) to see how well it generalizes.
- test on grandtour dataset from Anybotics
- also test on real-world data 
- metrics: no ground truth available, so only qualitative evaluation by visualizing the output semantic + elevation layer and seeing if it looks reasonable.
### experiment 4: navigation performance
- test full pipeline on grandtour dataset, compare against other navigation/mapping approaches (e.g. wvn, terp, etc.)
- metrics: success rate, time to reach goal, path length, etc.
- qualitative: demonstrate successful navigation in challenging environments with dynamic obstacles, show preference for traversable terrain (e.g. sidewalks over road) compared to other methods.