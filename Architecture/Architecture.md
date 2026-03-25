# Architecture Description
- The Arduino Portenta X8 has 2 main chips, the i.MX 8 Mini that has the embedded Linux, and the STM32H747 that is the real time core
- Main issues are that the IMU is connected to the i.MX, which is *not* real time. 
- Additionally, CAN bus connects via the STM32
- Arduino's distribution of Yocto Linux already contains a driver to bridge CAN-utils used in embedded Linux with the actual CAN bus on the STM32
- IMU data and PS3 controller input is streamed via ROS 2 topics
- Plan now is to use the embedded Linux as a forwarder of packets to the STM32, then use the 107-Arduino-Cyphal wrapper library to send Cyphal packets out over CAN (may need some sort of CAN driver)

```mermaid
graph TD
    subgraph "`Application Core - i.MX8`"
        A[IMU Sensor / ROS 2 Node] -->|Topic: /imu/data| B(can-utils: cansend/candump)
        B -->|SocketCAN| C[Linux Network Stack]
        C -->|IOCTL/Data| D[x8h7_can Driver]
    end

    subgraph "Inter-Processor Communication (IPC)"
        D -->|OpenAMP / RPMSG| E{Shared Memory Bridge}
    end

    subgraph "Real-Time Core - STM32H7"
        E -->|Interrupt/Callback| F[M4/M7 CAN Proxy Firmware]
        F -->|HAL_FDCAN_Transmit| G[FDCAN Peripheral]
    end

    subgraph "Physical Layer"
        G -->|TTL 3.3V TX/RX| H[External CAN Transceiver]
        H -->|Differential High/Low| I[Physical CAN Bus]
    end

    %%% style A fill:#f9f,stroke:#333,stroke-width:2px
    %% style E fill:#fff4dd,stroke:#d4a017,stroke-dasharray: 5 5
    %% style G fill:#bbf,stroke:#333,stroke-width:2px
    %% style I fill:#dfd,stroke:#333,stroke-width:2px
```