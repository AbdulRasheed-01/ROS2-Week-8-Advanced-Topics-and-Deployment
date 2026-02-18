# ROS2-Week-8-Advanced-Topics-and-Deployment

🎯 Learning Objectives
By the end of this week, you will be able to:

✅ Understand micro-ROS for embedded systems

✅ Containerize ROS 2 applications with Docker

✅ Implement real-time and safety-critical systems

✅ Optimize performance with QoS tuning

✅ Deploy multi-robot systems in production

✅ Use ROS 2 for cloud robotics

✅ Implement CI/CD for ROS 2 projects

✅ Build complete production-grade robot applications

📚 Theory Content

Key Concepts:

Component	          |      Description	                         |     Use Case

Micro-ROS Agent	    |      Bridge between micro-ROS and ROS 2	   |     Runs on Linux, connects to MCU

XRCE Client	        |      XRCE-DDS protocol implementation	     |     Runs on MCU, communicates with agent

Memory Pool	        |      Pre-allocated memory for real-time	   |     Ensures deterministic behavior

Static Allocator	  |      No dynamic memory allocation	         |     Safety-critical systems

Supported Hardware:

STM32 (F4, H7, L4 series)

ESP32 (with FreeRTOS)

Arduino (Portenta, Nicla)

Teensy (4.0, 4.1)

Raspberry Pi Pico

Custom ARM Cortex-M board

8.2 Docker for ROS 2

Docker Layers for ROS 2:

    # Base layer (OS + ROS 2 core)
    FROM ubuntu:22.04
    RUN apt-get update && apt-get install -y ros-humble-desktop
    
    # Dependencies layer
    FROM base AS deps
    COPY dependencies.sh .
    RUN ./dependencies.sh
    
    # Application layer
    FROM deps AS app
    COPY src/ /workspace/src
    RUN colcon build
    
    # Runtime layer
    FROM app AS runtime
    CMD ["ros2", "launch", "my_robot", "system.launch.py"]
8.3 Real-Time ROS 2

Real-Time Requirements:

Application	    |    Deadline	      |      Jitter Tolerance	        Safety Level

Motor Control	  |    1-10 ms	      |      < 100 μs	                SIL 2/3

Sensor Fusion	  |    10-50 ms	      |      < 1 ms	                  SIL 1

Navigation	    |    100 ms	        |      < 10 ms	                QM

Vision Processing	 | 33 ms (30 Hz)	|      < 5 ms	                  QM

Real-Time Configuration:

    # Configure real-time kernel
    sudo apt-get install linux-image-rt-amd64
    sudo apt-get install linux-headers-rt-amd64
    
    # Set real-time scheduling
    sudo chrt -f 90 -p $(pgrep -f "robot_node")
    
    # Configure CPU isolation
    # Add to GRUB_CMDLINE_LINUX: isolcpus=1,2 nohz_full=1,2 rcu_nocbs=1,2
    
    # Set memory locking
    sudo ulimit -l unlimited
8.4 Performance Optimization

QoS Profiles Comparison:

Profile	         |   Reliability	 |       Durability	    |        History	 |       Use Case

System Default	 |   Reliable	     |       Volatile	    |        Keep Last	 |       General purpose

Sensor Data	     |   Best Effort	 |       Volatile	    |        Keep Last	 |       High-frequency sensors

Parameters	     |   Reliable	     |       Transient Local	|    Keep All	 |       System configuration

Services	     |   Reliable	     |       Volatile	        |    Keep Last	 |       Request-response


Multi-Robot Coordination Patterns:

Pattern	      |          Description        	 |       Use Case

Centralized	  |          Single master node	     |       Small fleets (<10 robots)

Decentralized |	         Peer-to-peer communication |	 Large fleets, resilient

Hierarchical  | 	     Zone leaders	        |        Warehouse automation

Market-based  | 	     Auction for tasks	      |      Heterogeneous robots


⚙️ Setup and Installation

Step 1: Install Micro-ROS

    # Create micro-ROS workspace
    mkdir -p ~/microros_ws/src
    cd ~/microros_ws
    
    # Clone micro-ROS repository
    git clone -b humble https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup
    
    # Install dependencies
    sudo apt update && rosdep update
    rosdep install --from-paths src --ignore-src -y
    
    # Build micro-ROS tools
    colcon build --packages-select micro_ros_setup
    source install/setup.bash
    
    # Create micro-ROS firmware for your target
    ros2 run micro_ros_setup create_firmware_ws.sh freertos
    
    # Build for specific hardware (example: STM32F4)
    ros2 run micro_ros_setup configure_firmware.sh stm32f4 -t <transport>
    ros2 run micro_ros_setup build_firmware.sh
    
    # Start micro-ROS agent
    ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888

Step 2: Install Docker

    # Install Docker Engine
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    
    # Add user to docker group
    sudo usermod -aG docker $USER
    newgrp docker
    
    # Test installation
    docker run hello-world
    
    # Install Docker Compose
    sudo apt-get install docker-compose-plugin
    
    # Install NVIDIA container toolkit (for GPU support)
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
    curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    sudo apt-get update
    sudo apt-get install -y nvidia-docker2
    sudo systemctl restart docker

Step 3: Create Advanced Project Package

    cd ~/ros2_ws/src
    ros2 pkg create advanced_robotics --build-type ament_python \
        --dependencies rclpy rclcpp micro_ros_msgs micro_ros_agent \
                     nav2_msgs moveit_msgs geometry_msgs \
        --description "Week 8: Advanced Topics and Deployment"
    
    cd advanced_robotics
    mkdir -p advanced_robotics/{micro_ros,docker,cloud,real_time}
    mkdir -p {docker,config,launch,scripts,benchmarks}
    mkdir -p docker/{base,deps,app,runtime}
    mkdir -p config/{micro_ros,dds,qos}
    mkdir -p benchmarks/{cpu,latency,throughput}

🔧 Practical Exercises

Exercise 1: Micro-ROS with ESP32

1.1 Setup ESP32 for Micro-ROS:

    # Install ESP32 toolchain
    sudo apt-get install git wget flex bison gperf python3 python3-pip \
        python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util
    
    # Create micro-ROS workspace for ESP32
    cd ~/microros_ws
    ros2 run micro_ros_setup create_firmware_ws.sh freertos
    
    # Configure for ESP32
    ros2 run micro_ros_setup configure_firmware.sh esp32 -t udp -i 192.168.1.100 -p 8888
    
    # Build firmware
    ros2 run micro_ros_setup build_firmware.sh
    
    # Flash to ESP32
    ros2 run micro_ros_setup flash_firmware.sh
    
1.2 Micro-ROS Publisher Node (ESP32 Arduino):

Create firmware/esp32_publisher.ino:

    #include <micro_ros_arduino.h>
    #include <stdio.h>
    #include <rcl/rcl.h>
    #include <rcl/error_handling.h>
    #include <rclc/rclc.h>
    #include <rclc/executor.h>
    #include <std_msgs/msg/int32.h>
    
    // ROS 2 entities
    rcl_publisher_t publisher;
    std_msgs__msg__Int32 msg;
    rclc_support_t support;
    rcl_allocator_t allocator;
    rcl_node_t node;
    rcl_timer_t timer;
    rclc_executor_t executor;
    
    // Error handling
    #define RCCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){error_loop();}}
    #define RCSOFTCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){}}
    
    void error_loop() {
      while(1) {
        delay(100);
      }
    }
    
    void timer_callback(rcl_timer_t * timer, int64_t last_call_time) {
      RCLC_UNUSED(last_call_time);
      static int counter = 0;
      
      if (timer != NULL) {
        msg.data = counter++;
        RCSOFTCHECK(rcl_publish(&publisher, &msg, NULL));
        Serial.printf("Published: %d\n", msg.data);
      }
    }
    
    void setup() {
      Serial.begin(115200);
      delay(2000);
      
      // Set up micro-ROS transport (WiFi or Serial)
      set_microros_wifi_transports("YOUR_SSID", "YOUR_PASSWORD", "192.168.1.100", 8888);
      // OR for serial: set_microros_serial_transports(Serial);
      
      delay(2000);
      
      allocator = rcl_get_default_allocator();
      
      // Create init_options
      RCCHECK(rclc_support_init(&support, 0, NULL, &allocator));
      
      // Create node
      RCCHECK(rclc_node_init_default(&node, "esp32_node", "", &support));
      
      // Create publisher
      RCCHECK(rclc_publisher_init_default(
        &publisher,
        &node,
        ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32),
        "esp32_counter"));
      
      // Create timer
      const unsigned int timer_timeout = 1000;
      RCCHECK(rclc_timer_init_default(
        &timer,
        &support,
        RCL_MS_TO_NS(timer_timeout),
        timer_callback));
      
      // Create executor
      executor = rclc_executor_get_zero_initialized_executor();
      RCCHECK(rclc_executor_init(&executor, &support.context, 1, &allocator));
      RCCHECK(rclc_executor_add_timer(&executor, &timer));
      
      Serial.println("Micro-ROS node started");
    }
    
    void loop() {
      delay(100);
      RCSOFTCHECK(rclc_executor_spin_some(&executor, RCL_MS_TO_NS(100)));
    }
1.3 Micro-ROS Subscriber in ROS 2:

Create advanced_robotics/micro_ros/microros_subscriber.py:

    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from std_msgs.msg import Int32
    
    class MicroROSSubscriber(Node):
        def __init__(self):
            super().__init__('microros_subscriber')
            
            # Subscribe to ESP32 counter
            self.subscription = self.create_subscription(
                Int32,
                'esp32_counter',
                self.listener_callback,
                10)
            
            # Publisher for processed data
            self.processed_pub = self.create_publisher(
                Int32, 'processed_counter', 10)
            
            self.get_logger().info("Micro-ROS Subscriber started")
        
        def listener_callback(self, msg):
            self.get_logger().info(f'Received from ESP32: {msg.data}')
            
            # Process data
            processed = Int32()
            processed.data = msg.data * 2
            
            # Publish processed data
            self.processed_pub.publish(processed)
            self.get_logger().info(f'Published processed: {processed.data}')
    
    def main(args=None):
        rclpy.init(args=args)
        node = MicroROSSubscriber()
        
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass
        
        node.destroy_node()
        rclpy.shutdown()
    
    if __name__ == '__main__':
        main()
1.4 Run Micro-ROS System:

    # Terminal 1: Start micro-ROS agent
    ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
    
    # Terminal 2: Run ROS 2 subscriber
    ros2 run advanced_robotics microros_subscriber
    
    # Terminal 3: Monitor topics
    ros2 topic list
    ros2 topic echo /esp32_counter

Exercise 2: Dockerizing ROS 2 Applications

2.1 Base Dockerfile:

Create docker/base/Dockerfile:

    # Base ROS 2 Humble image
    FROM ubuntu:22.04
    
    # Set environment variables
    ENV DEBIAN_FRONTEND=noninteractive
    ENV LANG=en_US.UTF-8
    ENV ROS_DISTRO=humble
    
    # Install basic dependencies
    RUN apt-get update && apt-get install -y \
        locales \
        software-properties-common \
        curl \
        gnupg2 \
        lsb-release \
        && rm -rf /var/lib/apt/lists/* \
        && locale-gen en_US en_US.UTF-8
    
    # Add ROS 2 repository
    RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg \
        && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null
    
    # Install ROS 2 Humble (minimal)
    RUN apt-get update && apt-get install -y \
        ros-humble-ros-base \
        python3-colcon-common-extensions \
        && rm -rf /var/lib/apt/lists/*
    
    # Install development tools
    RUN apt-get update && apt-get install -y \
        python3-pip \
        python3-vcstool \
        git \
        nano \
        && rm -rf /var/lib/apt/lists/*
    
    # Source ROS 2
    RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc
    
    # Set working directory
    WORKDIR /workspace
    
    # Default command
    CMD ["/bin/bash"]
2.2 Development Dockerfile:

Create docker/dev/Dockerfile:

    # Development stage with full desktop and debugging tools
    FROM advanced_robotics_base:latest as dev
    
    # Install full desktop (for visualization)
    RUN apt-get update && apt-get install -y \
        ros-humble-desktop \
        ros-humble-rviz2 \
        ros-humble-gazebo-ros-pkgs \
        && rm -rf /var/lib/apt/lists/*
    
    # Install debugging tools
    RUN apt-get update && apt-get install -y \
        gdb \
        valgrind \
        strace \
        htop \
        tmux \
        && rm -rf /var/lib/apt/lists/*
    
    # Install Python debugging tools
    RUN pip3 install \
        ipython \
        jupyter \
        matplotlib \
        numpy \
        opencv-python
    
    # Set up display for GUI
    ENV DISPLAY=:0
    ENV QT_X11_NO_MITSHM=1
    ENV LIBGL_ALWAYS_SOFTWARE=1
    
    # Create workspace
    WORKDIR /workspace
    
    # Copy source code (mounted in development)
    CMD ["/bin/bash"]

2.3 Production Dockerfile:

Create docker/prod/Dockerfile:

    # Multi-stage build for production
    FROM advanced_robotics_base:latest as builder
    
    # Copy source code
    COPY src/ /workspace/src/
    
    # Build workspace
    WORKDIR /workspace
    RUN . /opt/ros/humble/setup.sh && \
        colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
    
    # Production image (minimal)
    FROM ubuntu:22.04 as runtime
    
    # Install runtime dependencies only
    RUN apt-get update && apt-get install -y \
        python3 \
        python3-yaml \
        python3-numpy \
        && rm -rf /var/lib/apt/lists/*
    
    # Copy built artifacts from builder
    COPY --from=builder /workspace/install /workspace/install
    COPY --from=builder /opt/ros/humble /opt/ros/humble
    
    # Set environment
    ENV ROS_DISTRO=humble
    ENV AMENT_PREFIX_PATH=/workspace/install
    ENV PYTHONPATH=/workspace/install/lib/python3.10/site-packages:$PYTHONPATH
    
    # Source ROS 2 and workspace
    RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc && \
        echo "source /workspace/install/setup.bash" >> /root/.bashrc
    
    WORKDIR /workspace
    
    # Default command
    CMD ["ros2", "launch", "my_robot", "system.launch.py"]

2.4 Docker Compose for Multi-Container Setup:

Create docker/docker-compose.yml:

    version: '3.8'
    
    services:
      # Micro-ROS Agent
      microros_agent:
        image: microros/micro-ros-agent:humble
        network_mode: host
        command: udp4 --port 8888
        restart: unless-stopped
        
      # Robot Core (Navigation + Control)
      robot_core:
        build:
          context: ..
          dockerfile: docker/prod/Dockerfile
        network_mode: host
        ipc: host
        pid: host
        privileged: true
        environment:
          - ROS_DOMAIN_ID=42
          - RMW_IMPLEMENTATION=rmw_fastrtps_cpp
        volumes:
          - /tmp/.X11-unix:/tmp/.X11-unix:rw
          - /dev:/dev:ro
        devices:
          - /dev/ttyUSB0:/dev/ttyUSB0
        command: ros2 launch robot_navigation navigation_stack.launch.py
        restart: unless-stopped
        
      # Perception (Vision + ML)
      perception:
        image: nvidia/cuda:11.8-runtime-ubuntu22.04
        runtime: nvidia
        network_mode: host
        environment:
          - NVIDIA_VISIBLE_DEVICES=all
          - ROS_DOMAIN_ID=42
        volumes:
          - ./models:/models
        command: ros2 run robot_perception perception_node
        
      # Fleet Manager
      fleet_manager:
        build:
          context: ..
          dockerfile: docker/fleet/Dockerfile
        network_mode: host
        environment:
          - ROS_DOMAIN_ID=42
        command: ros2 run fleet_manager fleet_manager_node
        
      # Database (for logging)
      influxdb:
        image: influxdb:2.7
        ports:
          - "8086:8086"
        volumes:
          - influxdb_data:/var/lib/influxdb2
          
      # Visualization
      viz:
        build:
          context: ..
          dockerfile: docker/dev/Dockerfile
        network_mode: host
        environment:
          - DISPLAY=${DISPLAY}
          - ROS_DOMAIN_ID=42
        volumes:
          - /tmp/.X11-unix:/tmp/.X11-unix:rw
        command: rviz2 -d /workspace/config/navigation.rviz
        
    volumes:
      influxdb_data:
2.5 Build and Run Docker Containers:

    # Build base image
    cd ~/ros2_ws/src/advanced_robotics/docker
    docker build -t advanced_robotics_base:latest -f base/Dockerfile .
    
    # Build development image
    docker build -t advanced_robotics_dev:latest -f dev/Dockerfile .
    
    # Build production image
    docker build -t advanced_robotics_prod:latest -f prod/Dockerfile .
    
    # Run with Docker Compose
    docker-compose up -d
    
    # Check logs
    docker-compose logs -f robot_core
    
    # Execute commands in running container
    docker exec -it advanced-robotics_robot_core_1 bash
    
    # Stop all containers
    docker-compose down
    
    # Push to registry
    docker tag advanced_robotics_prod:latest myregistry.com/advanced_robotics:latest
    docker push myregistry.com/advanced_robotics:latest

Exercise 3: Real-Time ROS 2

3.1 Real-Time Node with RT Priority:

Create advanced_robotics/real_time/rt_node.py:

    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from std_msgs.msg import Float64
    import os
    import threading
    import psutil
    import time
    
    class RealTimeNode(Node):
        def __init__(self):
            super().__init__('real_time_node')
            
            # Set real-time priority
            self.set_realtime_priority(priority=95)
            
            # Pin to isolated CPU core
            self.pin_to_cpu(core=1)
            
            # Lock memory to prevent swapping
            self.lock_memory()
            
            # Publishers
            self.rt_pub = self.create_publisher(
                Float64, '/real_time/data', 10)
            
            # Timer with high frequency
            timer_period = 0.001  # 1 kHz
            self.timer = self.create_timer(timer_period, self.timer_callback)
            
            # Performance monitoring
            self.latencies = []
            self.create_timer(1.0, self.report_performance)
            
            self.get_logger().info("Real-Time Node started")
        
        def set_realtime_priority(self, priority):
            """Set SCHED_FIFO real-time priority"""
            try:
                # Get current thread ID
                tid = threading.get_ident()
                
                # Set scheduling policy and priority
                param = os.sched_param(priority)
                os.sched_setscheduler(0, os.SCHED_FIFO, param)
                
                self.get_logger().info(f"Set RT priority to {priority}")
            except PermissionError:
                self.get_logger().error("Need root privileges for RT scheduling")
                self.get_logger().info("Run with: sudo -E python3 rt_node.py")
        
        def pin_to_cpu(self, core):
            """Pin process to specific CPU core"""
            try:
                p = psutil.Process()
                p.cpu_affinity([core])
                self.get_logger().info(f"Pinned to CPU core {core}")
            except Exception as e:
                self.get_logger().error(f"Failed to pin to CPU: {e}")
        
        def lock_memory(self):
            """Lock memory to prevent swapping"""
            try:
                import resource
                resource.setrlimit(resource.RLIMIT_MEMLOCK, 
                                  (resource.RLIM_INFINITY, resource.RLIM_INFINITY))
                self.get_logger().info("Memory locked")
            except Exception as e:
                self.get_logger().error(f"Failed to lock memory: {e}")
        
        def timer_callback(self):
            """High-frequency callback"""
            start_time = time.perf_counter()
            
            # Real-time task
            msg = Float64()
            msg.data = time.time()
            self.rt_pub.publish(msg)
            
            # Measure latency
            latency = (time.perf_counter() - start_time) * 1e6  # microseconds
            self.latencies.append(latency)
        
        def report_performance(self):
            """Report timing performance"""
            if self.latencies:
                avg_latency = sum(self.latencies) / len(self.latencies)
                max_latency = max(self.latencies)
                min_latency = min(self.latencies)
                
                self.get_logger().info(
                    f"Latency (μs) - Avg: {avg_latency:.1f}, "
                    f"Min: {min_latency:.1f}, Max: {max_latency:.1f}"
                )
                
                self.latencies.clear()
    
    def main(args=None):
        rclpy.init(args=args)
        node = RealTimeNode()
        
        # Use single-threaded executor for deterministic behavior
        executor = rclpy.executors.SingleThreadedExecutor()
        executor.add_node(node)
        
        try:
            executor.spin()
        except KeyboardInterrupt:
            pass
        
        node.destroy_node()
        rclpy.shutdown()
    
    if __name__ == '__main__':
        main()

3.2 Run with Real-Time Privileges:

    # Check current scheduling policy
    chrt -p $$
    
    # Run with real-time priority
    sudo -E python3 rt_node.py
    
    # Verify scheduling policy
    ps -eo pid,cls,rtprio,ni,pri,cmd | grep rt_node
    
    # Monitor with perf
    sudo perf stat -e context-switches,cpu-migrations,page-faults python3 rt_node.py
3.3 Real-Time Configuration Script:

Create scripts/setup_realtime.sh:

    #!/bin/bash
    
    # Real-time system setup for ROS 2
    
    # Check if running as root
    if [ "$EUID" -ne 0 ]; then 
        echo "Please run as root"
        exit 1
    fi
    
    # Configure CPU isolation
    echo "isolcpus=1,2 nohz_full=1,2 rcu_nocbs=1,2" >> /etc/default/grub
    update-grub
    
    # Set real-time limits
    echo "@realtime - rtprio 95" >> /etc/security/limits.conf
    echo "@realtime - memlock unlimited" >> /etc/security/limits.conf
    
    # Create real-time user group
    groupadd realtime
    usermod -a -G realtime $SUDO_USER
    
    # Configure kernel for real-time
    echo "kernel.sched_rt_runtime_us = 1000000" >> /etc/sysctl.conf
    echo "kernel.sched_rt_period_us = 1000000" >> /etc/sysctl.conf
    
    # Disable CPU frequency scaling
    echo "performance" > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
    
    # Disable hyper-threading (optional)
    echo off > /sys/devices/system/cpu/smt/control
    
    # Set IRQ affinity
    echo 1 > /proc/irq/default_smp_affinity
    
    echo "Real-time configuration complete. Reboot required."

Exercise 4: QoS Tuning and Performance Optimization

4.1 QoS Benchmark Node:

Create advanced_robotics/benchmarks/qos_benchmark.py:
    
    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from rclpy.qos import QoSProfile, ReliabilityPolicy, DurabilityPolicy, HistoryPolicy
    from std_msgs.msg import String
    import time
    import numpy as np
    import csv
    from datetime import datetime
    
    class QoSBenchmark(Node):
        def __init__(self):
            super().__init__('qos_benchmark')
            
            # Test configurations
            self.qos_configs = [
                {
                    'name': 'Reliable Low Latency',
                    'qos': QoSProfile(
                        depth=5,
                        reliability=ReliabilityPolicy.RELIABLE,
                        durability=DurabilityPolicy.VOLATILE,
                        history=HistoryPolicy.KEEP_LAST
                    )
                },
                {
                    'name': 'Best Effort',
                    'qos': QoSProfile(
                        depth=1,
                        reliability=ReliabilityPolicy.BEST_EFFORT,
                        durability=DurabilityPolicy.VOLATILE,
                        history=HistoryPolicy.KEEP_LAST
                    )
                },
                {
                    'name': 'Transient Local',
                    'qos': QoSProfile(
                        depth=10,
                        reliability=ReliabilityPolicy.RELIABLE,
                        durability=DurabilityPolicy.TRANSIENT_LOCAL,
                        history=HistoryPolicy.KEEP_LAST
                    )
                },
                {
                    'name': 'Keep All',
                    'qos': QoSProfile(
                        depth=100,
                        reliability=ReliabilityPolicy.RELIABLE,
                        durability=DurabilityPolicy.VOLATILE,
                        history=HistoryPolicy.KEEP_ALL
                    )
                }
            ]
            
            self.results = {}
            
            # Run benchmarks
            self.run_all_benchmarks()
            
            # Save results
            self.save_results()
            
            self.get_logger().info("QoS Benchmark completed")
        
        def run_all_benchmarks(self):
            """Test all QoS configurations"""
            for config in self.qos_configs:
                self.get_logger().info(f"\nTesting: {config['name']}")
                
                # Create publisher and subscriber
                pub = self.create_publisher(String, 'test_topic', config['qos'])
                sub = self.create_subscription(
                    String, 'test_topic', self.callback, config['qos'])
                
                # Test parameters
                message_count = 1000
                message_size = 1024  # bytes
                latencies = []
                throughput_start = time.time()
                
                # Send messages
                for i in range(message_count):
                    msg = String()
                    msg.data = 'X' * message_size
                    
                    start_time = time.perf_counter()
                    pub.publish(msg)
                    
                    # Measure round trip
                    rclpy.spin_once(self, timeout_sec=0.001)
                    
                    if hasattr(self, 'last_receive_time'):
                        latency = (self.last_receive_time - start_time) * 1e6
                        latencies.append(latency)
                    
                    self.last_send_time = start_time
                
                throughput_end = time.time()
                
                # Calculate metrics
                throughput = message_count / (throughput_end - throughput_start)
                
                # Store results
                self.results[config['name']] = {
                    'latency_us': {
                        'mean': np.mean(latencies),
                        'std': np.std(latencies),
                        'min': np.min(latencies),
                        'max': np.max(latencies),
                        'p99': np.percentile(latencies, 99)
                    },
                    'throughput_hz': throughput,
                    'loss_rate': self.calculate_loss_rate(message_count)
                }
                
                # Log results
                self.get_logger().info(
                    f"  Mean Latency: {self.results[config['name']]['latency_us']['mean']:.2f} μs\n"
                    f"  Throughput: {throughput:.2f} Hz\n"
                    f"  P99 Latency: {self.results[config['name']]['latency_us']['p99']:.2f} μs"
                )
        
        def callback(self, msg):
            self.last_receive_time = time.perf_counter()
            self.received_count += 1
        
        def calculate_loss_rate(self, expected):
            return (expected - self.received_count) / expected * 100
        
        def save_results(self):
            """Save benchmark results to CSV"""
            filename = f"qos_benchmark_{datetime.now().strftime('%Y%m%d_%H%M%S')}.csv"
            
            with open(filename, 'w', newline='') as csvfile:
                fieldnames = ['Config', 'Mean Latency (μs)', 'Std Dev', 'Min', 'Max', 'P99', 'Throughput (Hz)', 'Loss Rate (%)']
                writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
                
                writer.writeheader()
                for config, metrics in self.results.items():
                    writer.writerow({
                        'Config': config,
                        'Mean Latency (μs)': f"{metrics['latency_us']['mean']:.2f}",
                        'Std Dev': f"{metrics['latency_us']['std']:.2f}",
                        'Min': f"{metrics['latency_us']['min']:.2f}",
                        'Max': f"{metrics['latency_us']['max']:.2f}",
                        'P99': f"{metrics['latency_us']['p99']:.2f}",
                        'Throughput (Hz)': f"{metrics['throughput_hz']:.2f}",
                        'Loss Rate (%)': f"{metrics['loss_rate']:.2f}"
                    })
            
            self.get_logger().info(f"Results saved to {filename}")
    
    def main(args=None):
        rclpy.init(args=args)
        node = QoSBenchmark()
        
        rclpy.spin(node)
        node.destroy_node()
        rclpy.shutdown()
    
    if __name__ == '__main__':
        main()

4.2 DDS Tuning Configuration:

Create config/dds/fastdds.xml:

    <?xml version="1.0" encoding="UTF-8" ?>
    <dds xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
        <profiles>
            <!-- Transport configuration -->
            <transport_descriptors>
                <transport_descriptor>
                    <transport_id>udp_transport</transport_id>
                    <type>UDPv4</type>
                    <sendBufferSize>65536</sendBufferSize>
                    <receiveBufferSize>65536</receiveBufferSize>
                    <TTL>1</TTL>
                    <non_blocking_send>false</non_blocking_send>
                </transport_descriptor>
                
                <transport_descriptor>
                    <transport_id>shm_transport</transport_id>
                    <type>SHM</type>
                    <segment_size>104857600</segment_size>
                    <port_queue_capacity>100</port_queue_capacity>
                    <healthy_check_timeout_ms>1000</healthy_check_timeout_ms>
                    <rtps_dump_file>shm_dump.log</rtps_dump_file>
                </transport_descriptor>
            </transport_descriptors>
            
            <!-- Participant configuration -->
            <participant profile_name="default_participant" is_default_profile="true">
                <rtps>
                    <name>DefaultParticipant</name>
                    
                    <!-- Discovery settings -->
                    <builtin>
                        <discovery_config>
                            <discoveryProtocol>SIMPLE</discoveryProtocol>
                            <ignoreParticipantFlags>NO_FILTER</ignoreParticipantFlags>
                            <leaseDuration>20</leaseDuration>
                            <leaseAnnouncement>5</leaseAnnouncement>
                            <initialAnnouncements>
                                <count>5</count>
                                <period>
                                    <sec>1</sec>
                                    <nanosec>0</nanosec>
                                </period>
                            </initialAnnouncements>
                        </discovery_config>
                        
                        <metatrafficUnicastLocatorList>
                            <locator>
                                <udpv4>
                                    <address>239.255.0.1</address>
                                    <port>7400</port>
                                </udpv4>
                            </locator>
                        </metatrafficUnicastLocatorList>
                    </builtin>
                    
                    <!-- Transport selection -->
                    <userTransports>
                        <transport_id>shm_transport</transport_id>
                        <transport_id>udp_transport</transport_id>
                    </userTransports>
                    <useBuiltinTransports>false</useBuiltinTransports>
                    
                    <!-- Thread settings -->
                    <publishMode>ASYNCHRONOUS</publishMode>
                    <sendSocketBufferSize>65536</sendSocketBufferSize>
                    <listenSocketBufferSize>65536</listenSocketBufferSize>
                </rtps>
            </participant>
            
            <!-- Publisher configuration -->
            <publisher profile_name="reliable_publisher" is_default_profile="true">
                <historyMemoryPolicy>PREALLOCATED_WITH_REALLOC</historyMemoryPolicy>
                <qos>
                    <reliability>
                        <kind>RELIABLE_RELIABILITY_QOS</kind>
                        <max_blocking_time>
                            <sec>0</sec>
                            <nanosec>100000000</nanosec>
                        </max_blocking_time>
                    </reliability>
                    <durability>
                        <kind>VOLATILE_DURABILITY_QOS</kind>
                    </durability>
                    <publishMode>
                        <kind>ASYNCHRONOUS_PUBLISH_MODE</kind>
                    </publishMode>
                    <history>
                        <kind>KEEP_LAST_HISTORY_QOS</kind>
                        <depth>10</depth>
                    </history>
                </qos>
                <times>
                    <initialHeartbeatDelay>
                        <sec>0</sec>
                        <nanosec>10000000</nanosec>
                    </initialHeartbeatDelay>
                    <heartbeatPeriod>
                        <sec>0</sec>
                        <nanosec>100000000</nanosec>
                    </heartbeatPeriod>
                </times>
            </publisher>
            
            <!-- Subscriber configuration -->
            <subscriber profile_name="reliable_subscriber" is_default_profile="true">
                <historyMemoryPolicy>PREALLOCATED_WITH_REALLOC</historyMemoryPolicy>
                <qos>
                    <reliability>
                        <kind>RELIABLE_RELIABILITY_QOS</kind>
                    </reliability>
                    <durability>
                        <kind>VOLATILE_DURABILITY_QOS</kind>
                    </durability>
                </qos>
            </subscriber>
        </profiles>
    </dds>

4.3 Apply DDS Configuration:

    # Set Fast DDS configuration
    export FASTRTPS_DEFAULT_PROFILES_FILE=/path/to/fastdds.xml
    
    # Set RMW implementation
    export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
    
    # Alternative: Use Cyclone DDS
    export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
    
    # Set domain ID for network isolation
    export ROS_DOMAIN_ID=42
    
    # Network interface binding
    export ROS_LOCALHOST_ONLY=0  # 1 to restrict to localhost
    
    # Run with custom config
    ros2 run my_package my_node --ros-args --params-file config/optimized_params.yaml

Exercise 5: Multi-Robot Fleet Management

5.1 Fleet Manager Node:

Create advanced_robotics/cloud/fleet_manager.py:

    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from std_msgs.msg import String, Float32
    from geometry_msgs.msg import Pose, PoseStamped
    from nav2_msgs.action import NavigateToPose
    from visualization_msgs.msg import Marker, MarkerArray
    import json
    import aiohttp
    import asyncio
    import threading
    from datetime import datetime
    import sqlite3
    import hashlib
    
    class FleetManager(Node):
        def __init__(self):
            super().__init__('fleet_manager')
            
            # Fleet database
            self.db = sqlite3.connect('fleet.db')
            self.init_database()
            
            # Robot registry
            self.robots = {}
            self.tasks = {}
            self.task_queue = []
            
            # Publishers
            self.task_pub = self.create_publisher(String, '/fleet/task', 10)
            self.status_pub = self.create_publisher(String, '/fleet/status', 10)
            self.marker_pub = self.create_publisher(MarkerArray, '/fleet/markers', 10)
            
            # Subscribers
            self.create_subscription(String, '/fleet/robot_discovery', 
                                    self.robot_discovery_callback, 10)
            self.create_subscription(String, '/fleet/task_status',
                                    self.task_status_callback, 10)
            self.create_subscription(PoseStamped, '/fleet/robot_pose',
                                    self.robot_pose_callback, 10)
            
            # HTTP server for cloud integration
            self.start_http_server()
            
            # Timers
            self.create_timer(1.0, self.publish_status)
            self.create_timer(0.5, self.update_visualization)
            self.create_timer(5.0, self.save_to_database)
            self.create_timer(2.0, self.check_task_timeouts)
            
            self.get_logger().info("Fleet Manager started")
        
        def init_database(self):
            """Initialize SQLite database"""
            cursor = self.db.cursor()
            
            # Robots table
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS robots (
                    id TEXT PRIMARY KEY,
                    type TEXT,
                    status TEXT,
                    battery REAL,
                    last_seen TIMESTAMP,
                    current_task TEXT,
                    total_uptime INTEGER,
                    total_tasks INTEGER
                )
            ''')
            
            # Tasks table
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS tasks (
                    id TEXT PRIMARY KEY,
                    type TEXT,
                    robot_id TEXT,
                    status TEXT,
                    created TIMESTAMP,
                    started TIMESTAMP,
                    completed TIMESTAMP,
                    parameters TEXT,
                    result TEXT,
                    FOREIGN KEY (robot_id) REFERENCES robots(id)
                )
            ''')
            
            # Events table
            cursor.execute('''
                CREATE TABLE IF NOT EXISTS events (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    timestamp TIMESTAMP,
                    type TEXT,
                    robot_id TEXT,
                    description TEXT,
                    severity TEXT
                )
            ''')
            
            self.db.commit()
        
        def robot_discovery_callback(self, msg):
            """Register new robots"""
            try:
                data = json.loads(msg.data)
                robot_id = data['id']
                
                if robot_id not in self.robots:
                    self.robots[robot_id] = {
                        'type': data['type'],
                        'status': 'IDLE',
                        'battery': 100.0,
                        'pose': None,
                        'current_task': None,
                        'last_seen': self.get_clock().now(),
                        'capabilities': data.get('capabilities', []),
                        'tasks_completed': 0,
                        'uptime': 0
                    }
                    
                    # Add to database
                    cursor = self.db.cursor()
                    cursor.execute('''
                        INSERT OR REPLACE INTO robots 
                        (id, type, status, last_seen, total_tasks)
                        VALUES (?, ?, ?, ?, ?)
                    ''', (robot_id, data['type'], 'IDLE', 
                         datetime.now(), 0))
                    self.db.commit()
                    
                    self.get_logger().info(f"Robot registered: {robot_id}")
                    
                    # Log event
                    self.log_event('ROBOT_REGISTERED', robot_id, 
                                  f"Robot {robot_id} registered")
            
            except Exception as e:
                self.get_logger().error(f"Robot discovery error: {e}")
        
        def robot_pose_callback(self, msg):
            """Update robot pose"""
            robot_id = msg.header.frame_id
            if robot_id in self.robots:
                self.robots[robot_id]['pose'] = msg.pose
                self.robots[robot_id]['last_seen'] = self.get_clock().now()
        
        def task_status_callback(self, msg):
            """Update task status"""
            try:
                data = json.loads(msg.data)
                task_id = data['task_id']
                robot_id = data['robot_id']
                status = data['status']
                
                if task_id in self.tasks:
                    self.tasks[task_id]['status'] = status
                    
                    if status == 'COMPLETED':
                        self.robots[robot_id]['tasks_completed'] += 1
                        self.robots[robot_id]['status'] = 'IDLE'
                        self.tasks[task_id]['completed'] = datetime.now()
                        
                    elif status == 'FAILED':
                        self.robots[robot_id]['status'] = 'ERROR'
                        self.retry_task(task_id)
                    
                    # Update database
                    cursor = self.db.cursor()
                    cursor.execute('''
                        UPDATE tasks SET status=?, completed=? WHERE id=?
                    ''', (status, datetime.now() if status == 'COMPLETED' else None, 
                         task_id))
                    self.db.commit()
                    
                    self.get_logger().info(f"Task {task_id} status: {status}")
            
            except Exception as e:
                self.get_logger().error(f"Task status error: {e}")
        
        def assign_task(self, task_type, parameters=None):
            """Assign task to best available robot"""
            # Find best robot
            best_robot = None
            best_score = -1
            
            for robot_id, info in self.robots.items():
                if info['status'] == 'IDLE' and info['battery'] > 20.0:
                    # Score based on battery, proximity, capabilities
                    score = info['battery'] / 100.0
                    
                    if task_type in info.get('capabilities', []):
                        score += 0.5
                    
                    if score > best_score:
                        best_score = score
                        best_robot = robot_id
            
            if best_robot:
                # Create task
                task_id = hashlib.md5(f"{task_type}_{datetime.now()}".encode()).hexdigest()[:8]
                
                task = {
                    'id': task_id,
                    'type': task_type,
                    'robot_id': best_robot,
                    'status': 'ASSIGNED',
                    'created': datetime.now(),
                    'parameters': parameters or {}
                }
                
                self.tasks[task_id] = task
                self.robots[best_robot]['status'] = 'BUSY'
                self.robots[best_robot]['current_task'] = task_id
                
                # Save to database
                cursor = self.db.cursor()
                cursor.execute('''
                    INSERT INTO tasks (id, type, robot_id, status, created, parameters)
                    VALUES (?, ?, ?, ?, ?, ?)
                ''', (task_id, task_type, best_robot, 'ASSIGNED', 
                     datetime.now(), json.dumps(parameters)))
                self.db.commit()
                
                # Send task to robot
                task_msg = String()
                task_msg.data = json.dumps(task)
                self.task_pub.publish(task_msg)
                
                self.get_logger().info(f"Assigned {task_type} to {best_robot}")
                return task_id
            else:
                # Queue task
                self.task_queue.append({
                    'type': task_type,
                    'parameters': parameters,
                    'created': datetime.now()
                })
                self.get_logger().info(f"Task queued: {task_type}")
                return None
        
        def retry_task(self, task_id):
            """Retry failed task"""
            if task_id in self.tasks:
                task = self.tasks[task_id]
                self.get_logger().info(f"Retrying task {task_id}")
                self.assign_task(task['type'], task['parameters'])
        
        def check_task_timeouts(self):
            """Check for timed out tasks"""
            now = datetime.now()
            for task_id, task in list(self.tasks.items()):
                if task['status'] == 'ASSIGNED':
                    elapsed = (now - task['created']).total_seconds()
                    if elapsed > 60:  # 1 minute timeout
                        self.get_logger().warn(f"Task {task_id} timed out")
                        task['status'] = 'TIMEOUT'
                        self.retry_task(task_id)
        
        def publish_status(self):
            """Publish fleet status"""
            status = {
                'timestamp': datetime.now().isoformat(),
                'total_robots': len(self.robots),
                'idle_robots': sum(1 for r in self.robots.values() if r['status'] == 'IDLE'),
                'busy_robots': sum(1 for r in self.robots.values() if r['status'] == 'BUSY'),
                'charging_robots': sum(1 for r in self.robots.values() if r['status'] == 'CHARGING'),
                'error_robots': sum(1 for r in self.robots.values() if r['status'] == 'ERROR'),
                'queue_size': len(self.task_queue),
                'total_tasks_completed': sum(r['tasks_completed'] for r in self.robots.values())
            }
            
            msg = String()
            msg.data = json.dumps(status)
            self.status_pub.publish(msg)
            
            self.get_logger().info(
                f"Fleet Status - Robots: {status['total_robots']}, "
                f"Idle: {status['idle_robots']}, Busy: {status['busy_robots']}, "
                f"Queue: {status['queue_size']}",
                throttle_duration_sec=5.0
            )
        
        def update_visualization(self):
            """Update RViz markers for fleet visualization"""
            marker_array = MarkerArray()
            
            for i, (robot_id, info) in enumerate(self.robots.items()):
                if not info['pose']:
                    continue
                
                # Robot marker
                marker = Marker()
                marker.header.frame_id = 'map'
                marker.header.stamp = self.get_clock().now().to_msg()
                marker.ns = 'robots'
                marker.id = i
                marker.type = Marker.CUBE
                marker.action = Marker.ADD
                
                marker.pose = info['pose']
                marker.scale.x = 0.4
                marker.scale.y = 0.4
                marker.scale.z = 0.4
                
                # Color based on status
                if info['status'] == 'IDLE':
                    marker.color.g = 1.0
                elif info['status'] == 'BUSY':
                    marker.color.b = 1.0
                elif info['status'] == 'CHARGING':
                    marker.color.r = 1.0
                    marker.color.g = 1.0
                elif info['status'] == 'ERROR':
                    marker.color.r = 1.0
                
                marker.color.a = 0.8
                
                marker_array.markers.append(marker)
                
                # Text label
                text_marker = Marker()
                text_marker.header.frame_id = 'map'
                text_marker.header.stamp = self.get_clock().now().to_msg()
                text_marker.ns = 'robot_labels'
                text_marker.id = i + 100
                text_marker.type = Marker.TEXT_VIEW_FACING
                text_marker.action = Marker.ADD
                
                text_marker.pose = info['pose']
                text_marker.pose.position.z += 0.5
                text_marker.scale.z = 0.2
                text_marker.color.a = 1.0
                
                text_marker.text = f"{robot_id}\n{info['status']}\n{info['battery']:.0f}%"
                
                marker_array.markers.append(text_marker)
            
            self.marker_pub.publish(marker_array)
        
        def save_to_database(self):
            """Save current state to database"""
            cursor = self.db.cursor()
            for robot_id, info in self.robots.items():
                cursor.execute('''
                    UPDATE robots SET 
                        status=?, battery=?, last_seen=?, current_task=?,
                        total_tasks=?
                    WHERE id=?
                ''', (info['status'], info['battery'], datetime.now(),
                     info['current_task'], info['tasks_completed'], robot_id))
            self.db.commit()
        
        def log_event(self, event_type, robot_id, description, severity='INFO'):
            """Log system event"""
            cursor = self.db.cursor()
            cursor.execute('''
                INSERT INTO events (timestamp, type, robot_id, description, severity)
                VALUES (?, ?, ?, ?, ?)
            ''', (datetime.now(), event_type, robot_id, description, severity))
            self.db.commit()
        
        def start_http_server(self):
            """Start HTTP server for cloud integration"""
            from aiohttp import web
            import threading
            
            async def handle_get_robots(request):
                return web.json_response(self.robots)
            
            async def handle_get_tasks(request):
                return web.json_response(self.tasks)
            
            async def handle_post_task(request):
                data = await request.json()
                task_id = self.assign_task(data['type'], data.get('parameters'))
                return web.json_response({'task_id': task_id})
            
            async def handle_get_status(request):
                status = {
                    'robots': len(self.robots),
                    'tasks': len(self.tasks),
                    'queue': len(self.task_queue)
                }
                return web.json_response(status)
            
            app = web.Application()
            app.router.add_get('/api/robots', handle_get_robots)
            app.router.add_get('/api/tasks', handle_get_tasks)
            app.router.add_post('/api/tasks', handle_post_task)
            app.router.add_get('/api/status', handle_get_status)
            
            def run_server():
                web.run_app(app, port=8080)
            
            thread = threading.Thread(target=run_server, daemon=True)
            thread.start()
            self.get_logger().info("HTTP server started on port 8080")
    
    def main(args=None):
        rclpy.init(args=args)
        node = FleetManager()
        
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass
        
        node.db.close()
        node.destroy_node()
        rclpy.shutdown()
    
    if __name__ == '__main__':
        main()
5.2 Robot Client Node:

Create advanced_robotics/cloud/robot_client.py:

    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from std_msgs.msg import String, Float32
    from geometry_msgs.msg import PoseStamped
    from nav2_msgs.action import NavigateToPose
    import json
    import socket
    import requests
    import threading
    
    class RobotClient(Node):
        def __init__(self, robot_id):
            super().__init__(f'robot_client_{robot_id}')
            
            self.robot_id = robot_id
            self.fleet_manager_ip = '192.168.1.100'
            self.fleet_manager_port = 8080
            
            # Robot state
            self.status = 'IDLE'
            self.battery = 100.0
            self.current_task = None
            
            # Publishers
            self.discovery_pub = self.create_publisher(
                String, '/fleet/robot_discovery', 10)
            self.status_pub = self.create_publisher(
                String, '/fleet/task_status', 10)
            self.pose_pub = self.create_publisher(
                PoseStamped, '/fleet/robot_pose', 10)
            
            # Subscribers
            self.task_sub = self.create_subscription(
                String, '/fleet/task', self.task_callback, 10)
            
            # Timers
            self.create_timer(5.0, self.send_heartbeat)
            self.create_timer(1.0, self.publish_pose)
            self.create_timer(10.0, self.update_battery)
            
            # Register with fleet manager
            self.register_robot()
            
            # Start cloud sync thread
            self.start_cloud_sync()
            
            self.get_logger().info(f"Robot Client {robot_id} started")
        
        def register_robot(self):
            """Send registration to fleet manager"""
            discovery_msg = {
                'id': self.robot_id,
                'type': 'AGV',
                'capabilities': ['navigation', 'pickup', 'dropoff'],
                'ip': self.get_local_ip()
            }
            
            msg = String()
            msg.data = json.dumps(discovery_msg)
            self.discovery_pub.publish(msg)
            
            self.get_logger().info(f"Registered with fleet manager")
        
        def task_callback(self, msg):
            """Process incoming task"""
            try:
                task = json.loads(msg.data)
                
                if task['robot_id'] == self.robot_id:
                    self.get_logger().info(f"Received task: {task['type']}")
                    self.current_task = task
                    
                    # Execute task based on type
                    if task['type'] == 'NAVIGATE':
                        self.execute_navigation(task['parameters'])
                    elif task['type'] == 'PICKUP':
                        self.execute_pickup(task['parameters'])
                    elif task['type'] == 'DROPOFF':
                        self.execute_dropoff(task['parameters'])
                    
            except Exception as e:
                self.get_logger().error(f"Task processing error: {e}")
        
        def execute_navigation(self, params):
            """Execute navigation task"""
            self.status = 'BUSY'
            
            # Send task started status
            self.send_task_status('STARTED')
            
            # Simulate navigation
            self.get_logger().info(f"Navigating to {params.get('x', 0)}, {params.get('y', 0)}")
            
            # Here you would call Nav2 action client
            
            # Simulate completion
            self.create_timer(5.0, lambda: self.complete_task('SUCCESS'))
        
        def execute_pickup(self, params):
            """Execute pickup task"""
            self.get_logger().info(f"Picking up object at {params.get('location', 'unknown')}")
            self.create_timer(3.0, lambda: self.complete_task('SUCCESS'))
        
        def execute_dropoff(self, params):
            """Execute dropoff task"""
            self.get_logger().info(f"Dropping off at {params.get('location', 'unknown')}")
            self.create_timer(2.0, lambda: self.complete_task('SUCCESS'))
        
        def complete_task(self, result):
            """Mark task as completed"""
            self.status = 'IDLE'
            self.send_task_status('COMPLETED', result)
            self.current_task = None
        
        def send_task_status(self, status, result=None):
            """Send task status to fleet manager"""
            status_msg = {
                'task_id': self.current_task['id'] if self.current_task else 'unknown',
                'robot_id': self.robot_id,
                'status': status,
                'result': result,
                'timestamp': self.get_clock().now().nanoseconds
            }
            
            msg = String()
            msg.data = json.dumps(status_msg)
            self.status_pub.publish(msg)
        
        def publish_pose(self):
            """Publish robot pose"""
            # In real implementation, get from localization
            pose_msg = PoseStamped()
            pose_msg.header.frame_id = self.robot_id
            pose_msg.header.stamp = self.get_clock().now().to_msg()
            
            # Simulated pose
            pose_msg.pose.position.x = 0.0
            pose_msg.pose.position.y = 0.0
            pose_msg.pose.orientation.w = 1.0
            
            self.pose_pub.publish(pose_msg)
        
        def update_battery(self):
            """Simulate battery drain"""
            self.battery = max(0, self.battery - 1.0)
            
            if self.battery < 20.0 and self.status == 'IDLE':
                self.request_charging()
        
        def request_charging(self):
            """Request charging task"""
            self.get_logger().warn(f"Battery low ({self.battery}%), requesting charge")
            
            # Send charging request via HTTP
            try:
                response = requests.post(
                    f"http://{self.fleet_manager_ip}:{self.fleet_manager_port}/api/tasks",
                    json={'type': 'CHARGE', 'parameters': {'robot_id': self.robot_id}}
                )
            except:
                pass
        
        def send_heartbeat(self):
            """Send heartbeat to fleet manager"""
            try:
                response = requests.get(
                    f"http://{self.fleet_manager_ip}:{self.fleet_manager_port}/api/status",
                    timeout=2
                )
                if response.status_code == 200:
                    self.get_logger().debug("Heartbeat OK", throttle_duration_sec=10.0)
            except:
                self.get_logger().warn("Fleet manager unreachable")
        
        def get_local_ip(self):
            """Get local IP address"""
            s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            try:
                s.connect(('8.8.8.8', 1))
                ip = s.getsockname()[0]
            except Exception:
                ip = '127.0.0.1'
            finally:
                s.close()
            return ip
        
        def start_cloud_sync(self):
            """Start cloud synchronization thread"""
            def sync_loop():
                while rclpy.ok():
                    try:
                        # Sync with cloud
                        response = requests.get(
                            f"http://{self.fleet_manager_ip}:{self.fleet_manager_port}/api/tasks",
                            timeout=5
                        )
                        # Process cloud commands
                        time.sleep(10)
                    except:
                        time.sleep(5)
            
            thread = threading.Thread(target=sync_loop, daemon=True)
            thread.start()
    
    def main(args=None):
        rclpy.init(args=args)
        
        import sys
        robot_id = sys.argv[1] if len(sys.argv) > 1 else 'robot_001'
        
        node = RobotClient(robot_id)
        
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass
        
        node.destroy_node()
        rclpy.shutdown()
    
    if __name__ == '__main__':
        main()

Exercise 6: CI/CD for ROS 2

6.1 GitHub Actions Workflow:

Create .github/workflows/ci.yml:

    name: ROS 2 CI/CD Pipeline
    
    on:
      push:
        branches: [ main, develop ]
      pull_request:
        branches: [ main ]
      release:
        types: [ published ]
    
    env:
      ROS_DISTRO: humble
      ROS2_WORKSPACE: ros2_ws
    
    jobs:
      build-and-test:
        runs-on: ubuntu-22.04
        
        strategy:
          matrix:
            include:
              - container: ubuntu:22.04
                ros_distro: humble
        
        container:
          image: ${{ matrix.container }}
        
        steps:
        - name: Checkout code
          uses: actions/checkout@v3
          with:
            fetch-depth: 0
        
        - name: Setup ROS 2 environment
          run: |
            apt-get update
            apt-get install -y locales
            locale-gen en_US en_US.UTF-8
            update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
            export LANG=en_US.UTF-8
            
            # Add ROS 2 repository
            apt-get install -y curl gnupg2 lsb-release
            curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
            echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null
            
            # Install ROS 2
            apt-get update
            apt-get install -y ros-${{ env.ROS_DISTRO }}-desktop \
                              python3-colcon-common-extensions \
                              python3-rosdep \
                              python3-vcstool
            
            # Initialize rosdep
            rosdep init
            rosdep update
        
        - name: Create workspace
          run: |
            mkdir -p ${{ env.ROS2_WORKSPACE }}/src
            cp -r $GITHUB_WORKSPACE/* ${{ env.ROS2_WORKSPACE }}/src/
        
        - name: Install dependencies
          run: |
            cd ${{ env.ROS2_WORKSPACE }}
            rosdep install --from-paths src --ignore-src -r -y
        
        - name: Build workspace
          run: |
            cd ${{ env.ROS2_WORKSPACE }}
            source /opt/ros/${{ env.ROS_DISTRO }}/setup.bash
            colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
        
        - name: Run tests
          run: |
            cd ${{ env.ROS2_WORKSPACE }}
            source /opt/ros/${{ env.ROS_DISTRO }}/setup.bash
            source install/setup.bash
            colcon test --packages-select advanced_robotics
            colcon test-result --verbose
        
        - name: Upload test results
          uses: actions/upload-artifact@v3
          if: always()
          with:
            name: test-results
            path: ${{ env.ROS2_WORKSPACE }}/log/
        
        - name: Run linters
          run: |
            cd ${{ env.ROS2_WORKSPACE }}
            source /opt/ros/${{ env.ROS_DISTRO }}/setup.bash
            
            # Python linting
            pip3 install flake8 pytest
            colcon build --packages-select advanced_robotics
            colcon test --packages-select advanced_robotics --pytest-args --flake8
            
            # C++ linting
            apt-get install -y clang-tidy
            find src -name "*.cpp" -exec clang-tidy {} \;
    
      docker-build:
        needs: build-and-test
        runs-on: ubuntu-22.04
        
        steps:
        - name: Checkout code
          uses: actions/checkout@v3
        
        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v2
        
        - name: Login to DockerHub
          uses: docker/login-action@v2
          with:
            username: ${{ secrets.DOCKER_USERNAME }}
            password: ${{ secrets.DOCKER_PASSWORD }}
        
        - name: Build and push base image
          uses: docker/build-push-action@v4
          with:
            context: .
            file: docker/base/Dockerfile
            push: ${{ github.event_name != 'pull_request' }}
            tags: |
              ${{ secrets.DOCKER_USERNAME }}/advanced_robotics:latest
              ${{ secrets.DOCKER_USERNAME }}/advanced_robotics:${{ github.sha }}
        
        - name: Build and push production image
          uses: docker/build-push-action@v4
          with:
            context: .
            file: docker/prod/Dockerfile
            push: ${{ github.event_name != 'pull_request' }}
            tags: |
              ${{ secrets.DOCKER_USERNAME }}/advanced_robotics-prod:latest
              ${{ secrets.DOCKER_USERNAME }}/advanced_robotics-prod:${{ github.sha }}
      
      deploy:
        needs: docker-build
        if: github.event_name == 'release' && github.event.action == 'published'
        runs-on: ubuntu-22.04
        
        steps:
        - name: Deploy to production
          run: |
            echo "Deploying version ${{ github.event.release.tag_name }}"
            # Add deployment commands here (kubectl, ansible, etc.)
        
        - name: Notify deployment
          uses: actions/github-script@v6
          with:
            script: |
              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: '✅ Deployment completed successfully!'
              })
6.2 Pre-commit Hooks:

Create .pre-commit-config.yaml:

    repos:
      - repo: https://github.com/pre-commit/pre-commit-hooks
        rev: v4.4.0
        hooks:
          - id: trailing-whitespace
          - id: end-of-file-fixer
          - id: check-yaml
          - id: check-added-large-files
          - id: check-merge-conflict
          - id: detect-private-key
      
      - repo: https://github.com/psf/black
        rev: 23.3.0
        hooks:
          - id: black
            language_version: python3
      
      - repo: https://github.com/PyCQA/flake8
        rev: 6.0.0
        hooks:
          - id: flake8
            args: [--max-line-length=120]
      
      - repo: https://github.com/pre-commit/mirrors-clang-format
        rev: v16.0.0
        hooks:
          - id: clang-format
            args: [-i]
      
      - repo: https://github.com/cpplint/cpplint
        rev: 1.6.1
        hooks:
          - id: cpplint
            args: [--filter=-build/include_subdir,-runtime/references]
      
      - repo: https://github.com/ament/ament_lint
        rev: humble
        hooks:
          - id: ament_copyright
          - id: ament_xmllint

🔍 Troubleshooting Production Issues

Issue 1: Micro-ROS Connection Lost
    
    # Check agent logs
    docker logs production_microros_agent_1
    
    # Verify network connectivity
    ping 192.168.1.100
    
    # Restart agent
    docker-compose restart microros_agent

Issue 2: Container Resource Exhaustion

    # Monitor resource usage
    docker stats
    
    # Set resource limits in docker-compose
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    
    # Check OOM kills
    dmesg | grep -i "out of memory"

Issue 3: DDS Discovery Problems

    # Check ROS domain ID
    echo $ROS_DOMAIN_ID
    
    # Verify network interface
    ros2 multicast receive
    
    # Check firewall
    sudo ufw status
    sudo ufw allow 7400/udp

Issue 4: Database Corruption

    # Backup database
    docker exec production_influxdb_1 influx backup /backup
    
    # Restore from backup
    docker exec production_influxdb_1 influx restore /backup
    
    # Check database logs
    docker logs production_influxdb_1 --tail 100

📚 Additional Resources
Professional Certifications
ROS 2 Developer Certification

AWS RoboMaker Certification

NVIDIA Isaac ROS Certification

Advanced Topics for Further Study
Formal Verification of robot behaviors

Digital Twins for simulation and monitoring

5G Integration for low-latency control

Swarm Robotics algorithms

Human-Robot Interaction safety

Reinforcement Learning for robot control

Edge AI for on-robot inference

Blockchain for robot coordination

Production Tools
Kubernetes for container orchestration

Prometheus for metrics collection

ELK Stack for log aggregation

GitOps with ArgoCD

Service Mesh (Istio) for microservices

Career Paths
Robotics Software Engineer

Autonomous Systems Architect

ROS 2 Consultant

Robotics DevOps Engineer

AI Robotics Specialist

Fleet Operations Manager

🎓 Congratulations, Graduate!
You've successfully completed the 8-Week ROS 2 Humble Course from zero to hero!

What You've Achieved:
✅ Mastered ROS 2 fundamentals and architecture

✅ Built complex robotic systems with navigation and manipulation

✅ Integrated perception and real-time control

✅ Deployed production-ready robot applications

✅ Learned cloud robotics and fleet management

✅ Implemented CI/CD for robotic systems

Next Steps:
Build your portfolio with the projects from each week

Contribute to open-source ROS 2 packages

Join the ROS community at discourse.ros.org

Apply for robotics roles with your new skills

Keep learning with advanced topics in AI, control, and hardware

Final Project Ideas:
Autonomous delivery robot with navigation and manipulation

Multi-robot warehouse system with fleet management

Agricultural robot for crop monitoring

Search and rescue robot with autonomy

Telepresence robot with web interface

💡 Remember: Robotics is a journey, not a destination. Keep building, keep learning, and never stop innovating!

