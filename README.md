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

