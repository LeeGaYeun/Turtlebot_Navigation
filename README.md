# 터틀봇 네비게이션 프로젝트

## 프로젝트 개요

- **프로젝트 명**: 터틀봇 네비게이션
- **시나리오**
    - SLAM 으로 맵을 그리고 해당 맵에서 시리얼 통신을 이용합니다.
    - 키보드 입력을 받고 입력값에 따라 각각의 (x, y) 좌표로 터틀봇을 이동합니다.
    - 터틀봇에 USB 카메라를 장착하고 OpenCV 라이브러리를 이용하여 AR 마커를 인식하고 마커에 저장된 (x, y) 좌표로 이동합니다.

## 작업순서

- [간트차트](https://www.notion.so/1416181321a04eb3aafdf834ef693d48?pvs=21)
- [칸반보드](https://www.notion.so/7e9d3aad44224ccdb04ac573c0c7cbfa?pvs=21)

## 다이어그램

![시퀀스 다이어그램](https://github.com/LeeGaYeun/Turtlebot_Navigation/assets/149138767/b2db1ba8-0aaa-4e37-9102-6c10fc2e3930)

## 프로젝트 성과 결과

- Gazebo 및 SLAM과 같은 도구를 사용하여 터틀봇을 제어하고, 이를 통해 ROS 관련 지식을 확보했습니다.
- 시리얼 통신을 활용하여 터틀봇을 제어했습니다.
- 네트워크와 관련된 SSH 연결로 PC에서 터틀봇에 장착된 라즈베리파이에 접속하여 파일을 조작했습니다.
- 파이썬 문법을 익혔습니다.
- Ubuntu 운영체제를 사용하여 프로젝트를 진행했습니다.

## 아쉬운점

1. ROS를 구동하기에는 컴퓨터 사양이 좋지 않아 자주 튕겼습니다.
2. AR마커를 인식하고 싶었지만 구현하는 데 어려움을 겪었습니다.

## 참고

- [GitHub Repository](https://github.com/greattoe/ros2tutorial.git)

## 시연

[터틀봇_네비게이션 시연 영상](https://youtu.be/Mtbn5j1OkVw)
