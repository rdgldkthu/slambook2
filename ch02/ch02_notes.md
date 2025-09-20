# 2.1 Sensors in Visual SLAM
## 2.1.1 Monocular Camera
- 구조가 간단하고, 저렴하다.
- 이미지 데이터 (사진): 어떤 Scene이 카메라의 이미지 센서에 투영된 것 
	- 3D 세상을 2D에 기록했기 때문에 하나의 Dimension이 줄어듦 (Depth) 
	- -> 한장의 사진만으로는 Scene에 있는 물체와 카메라 사이의 거리를 계산할 수 없다. (SLAM에서 이 거리값은 중요)
	- -> 한장의 사진만으로는 물체의 실제 크기를 특정할 수 없다. (크지만 먼 물체, 작지만 가까운 물체는 사진 속에서 비슷하다.)
	- 3D 구조를 나타내기 위해선 카메라 앵글이 변해야 한다.
	- -> 카메라의 이동이 있어야 Motion 추정이 가능해지고 Scene에 포함된 물체의 거리와 크기를 계산할 수 있다.
- 카메라로부터 물체의 거리에 따라 사진 속에서의 이동 속도가 달라지는 점(Disparity)을 이용하면 거리의 정량적 판단이 가능해진다. 
	- 하지만 이렇게 계산된 Depth는 상대적인 값이므로 실제와는 차이가 있다. (Scale Ambiguity)
## 2.1.2 Stereo Camera and RGB-D Camera
- 물체와 카메라 사이의 거리를 측정해 Monocular의 한계를 극복하기 위해 사용한다.
- 거리를 알면 Scene의 3D 구조를 한장의 사진만으로 Reconstruct하고 Scale Ambiguity를 제거할 수 있다.
- **Stereo Camera**
	- Stereo Camera는 두개의 Monocular Camera로 구성되어 있고 두 카메라 간 거리인 Baseline 길이를 통해 각 픽셀의 공간적 위치를 추정한다. (이는 사람의 눈이 물체의 원근감을 파악하는 방식과 비슷하다.)
	- 다른 센서에 의존하지 않기 때문에 실내/실외에서 모두 사용 가능하다.
	- 측정할 수 있는 거리는 Baseline이 길어질수록 길어진다.
	- 단점으로는 설치와 Calibration 과정이 복잡하고 계산량이 많아 GPU/FPGA 가속이 필요하다.
- **RGB-D Camera**
	- Infrared Structured Light 또는 Time-of-Flight 원리를 사용해 레이더와 비슷하게 직접적으로 물체와의 거리를 측정한다.
	- Stereo Camera처럼 많은 양의 계산을 처리할 필요가 없다.
	- 측정 범위가 좁고 노이즈가 크며 태양광의 영향을 받기 쉽고 투명한 재질의 물체를 측정할 수 없다는 등의 문제로 주로 실내에서 사용한다.
# 2.2 Classical Visual SLAM Structure
1. Obtaining Sensor Data : 
	- 카메라에서 이미지 데이터 읽기 및 전처리 (Encoder/IMU 등 센서 데이터 읽기 및 동기화)
2. Visual Odometry, VO (Front End) : 
	- 연결된 프레임에서 카메라의 운동을 추정하고 Local Map 작성
3. Optimization (Back End) :
	- VO에서 온 Camera Pose와 Loop Closure 데이터를 최적화해 Global Map과 궤적 작성
4. Loop Closure Detection :
	- 왔었던 곳에 다시 왔는지 판단
5. Mapping :
	- 추정한 궤적을 통해 지도 작성
이 Structure는 십몇년간의 연구 성과로서 정형화되어 사용되어왔다. **Static, Rigid한 환경에서 조명의 변화가 크지 않고 인위적인 간섭이 없다면 SLAM기술은 이미 상당히 성숙하다.**
# 2.3 Mathematical Representation of SLAM
- 확률적 상태공간 모델에 기반하며, 두 가지 주요 방정식으로 이루어져 있다.
	- **로봇의 자세가 시간에 따라 어떻게 변하는지**를 나타내는 **운동 모델** : $\mathbf{x}_k = f(\mathbf{x}_{k-1},\mathbf{u}_k,\mathbf{w}_k)$ 
	- **센서가 관측한 랜드마크 정보**를 설명하는 **관측 모델** : $\mathbf{z}_{k,j} = h(\mathbf{y}_{j},\mathbf{x}_k,\mathbf{v}_{k,j})$ 
		- 사진이 찍힌 시점들만을 다루므로, $k=1,\cdots,K$ (Continuous -> Discrete)
		- $\mathbf{x}$ : Camera Pose. (로봇이 이동한 경로 : $\mathbf{x}_1,\cdots,\mathbf{x}_K$ )
		- $\mathbf{y}$ : Landmark Point. ($N$개의 Landmark Point들로 이루어진 지도 : $\mathbf{y}_1,\cdots,\mathbf{y}_N$ )
		- $\mathbf{z}$ : Observation Data.
		- $\mathbf{u}$ : Motion Sensor Data.
		- $\mathbf{v}$ : Observation Noise.
		- $\mathbf{w}$ : Sensor Noise.
- SLAM의 목표는 센서 데이터가 주어졌을 때, **로봇의 경로**와 **랜드마크 맵**을 동시에 추정하는 것이다.
	- $\mathbf{u}$와 $\mathbf{z}$를 알고 있을 때, $\mathbf{x}$(**Localization**)와 $\mathbf{y}$(**Mapping**)를 추정하는 것.