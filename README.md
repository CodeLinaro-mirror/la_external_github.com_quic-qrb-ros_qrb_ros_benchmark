<div align="center">
  <h1>QRB ROS Benchmark</h1>
  <p align="center">
   <img src="./docs/assets/qrb_ros_benchmark.png" width="300">
  </p>
  <p>ROS2 package for evaluating performance of ROS components on Qualcomm robotics platforms</p>
  <a href="https://ubuntu.com/download/qualcomm-iot" target="_blank"><img src="https://img.shields.io/badge/Qualcomm%20Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Qualcomm Ubuntu"></a>
  <a href="https://docs.ros.org/en/jazzy/" target="_blank"><img src="https://img.shields.io/badge/ROS%20Jazzy-1c428a?style=for-the-badge&logo=ros&logoColor=white" alt="Jazzy"></a>
</div>

---

## 👋 Overview
**QRB ROS Benchmark** is a benchmarking tool designed for evaluating performance of ROS components on Qualcomm robotics platforms. It provides reusable components for benchmarking various message types and ROS nodes, with a focus on zero-copy transport mechanisms.

<div align="center">
  <img src="./docs/assets/architecture.png" alt="architecture">
</div>

<br>

**QRB ROS Benchmark** builds on [ros2_benchmark](https://github.com/NVIDIA-ISAAC-ROS/ros2_benchmark) and extends it with specialized components for benchmarking QRB ROS transport types, including:

- QRB transport types (Image, IMU, PointCloud2)
- DMABuf transport types (Image, PointCloud2)
- Standard ROS message types

---

## 🔎 Table of Contents
  * [APIs](#-apis)
  * [Usage](#-usage)
  * [Build from Source](#-build-from-source)
  * [Contributing](#-contributing)
  * [License](#-license)

---

## ⚓ APIs
The basic usage is same to [ros2_benchmark](https://github.com/NVIDIA-ISAAC-ROS/ros2_benchmark) and following are the exclusive message tyeps supported by QRB ROS Benchmark:

**QRB Transport Types:**
- `qrb_ros/transport/type/Image`
- `qrb_ros/transport/type/Imu`
- `qrb_ros/transport/type/PointCloud2`
- `qrb_ros_tensor_list_msgs::msg::TensorList`

**DMABuf Transport Types:**
- `dmabuf_transport/type/Image`
- `dmabuf_transport/type/PointCloud2`

---

## 🚀 Usage
Here comes the steps for evaluating [qrb_ros/transport/type/Image](https://github.com/qualcomm-qrb-ros/qrb_ros_transport/tree/main/qrb_ros_transport_image_type):

Prepare the benchmark script, benchmark-transport-image.py:

```python
from launch_ros.actions import ComposableNodeContainer
from launch_ros.descriptions import ComposableNode
from ros2_benchmark import ImageResolution
from ros2_benchmark import ROS2BenchmarkConfig, ROS2BenchmarkTest

def launch_setup(container_prefix, container_sigterm_timeout):
    data_loader_node = ComposableNode(
        name='DataLoaderNode',
        namespace=TestQrbNode.generate_namespace(),
        package='ros2_benchmark',
        plugin='ros2_benchmark::DataLoaderNode',
    )

    # QrbPlaybackNode
    playback_node = ComposableNode(
        name='QrbPlaybackNode',
        namespace=TestQrbNode.generate_namespace(),
        package='qrb_ros_benchmark',
        plugin='qrb_ros::benchmark::QrbPlaybackNode',
        parameters=[{
            'data_formats': [
                'qrb_ros/transport/type/Image'
            ],
        }],
        remappings=[
            ('buffer/input0', '/data_loader_image'),
            ('input0', '/playback_image')
        ]
    )

    # Insert QRB ROS nodes

    # QrbMonitorNode
    monitor_node = ComposableNode(
        name='QrbMonitorNode',
        namespace=TestQrbNode.generate_namespace(),
        package='qrb_ros_benchmark',
        plugin='qrb_ros::benchmark::QrbMonitorNode',
        parameters=[{
            'monitor_data_format': 'qrb_ros/transport/type/Image',
        }]
    )

    composable_node_container = ComposableNodeContainer(
        name='container',
        namespace=TestQrbNode.generate_namespace(),
        package='rclcpp_components',
        executable='component_container_mt',
        prefix=container_prefix,
        sigterm_timeout=container_sigterm_timeout,
        composable_node_descriptions=[
            data_loader_node,
            playback_node,
            monitor_node,
            # Insert QRB ROS nodes
        ],
        output='screen'
    )

    return [composable_node_container]

def generate_test_description():
    return TestQrbNode.generate_test_description_with_nsys(launch_setup)

class TestQrbNode(ROS2BenchmarkTest):
    config = ROS2BenchmarkConfig(
        benchmark_name='Qrb_ros_transport image Test Benchmark',
        input_data_path='/path/to/rosbag',
        publisher_upper_frequency=100.0,
        publisher_lower_frequency=10.0,
        playback_message_buffer_size=10
    )

    def test_benchmark(self):
        self.run_benchmark()
```

Run the benchmark script:

```
source /opt/ros/jazzy/setup.bash
launch_test benchmark-transport-image.py
```

## 👨‍💻 Build from Source

Source is located at sources/quic-qrb-ros/qrb_ros_benchmark in the workspace.

```
cd build-utils/ubuntu/
python3 build.py --gen-debians --package ros-jazzy-qrb-ros-benchmark
```

Built .deb files are output to:

```
<workspace>/debian_packages/oss/ros-jazzy-qrb-ros-benchmark
```

---

## 🤝 Contributing

We love community contributions! Get started by reading our [CONTRIBUTING.md](CONTRIBUTING.md).
Feel free to create an issue for bug report, feature requests or any discussion.

---

## 📜 License

Project is licensed under the [BSD-3-Clause](https://spdx.org/licenses/BSD-3-Clause.html) License. See [LICENSE](./LICENSE) for the full license text.
