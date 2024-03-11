## 1. 프로젝트

- 프로젝트 명 : 터틀봇 네비게이션
- 시나리오
    - SLAM 으로 맵을 그려 해당 맵에서 시리얼 통신을 이용한다.
    - 키보드 입력을 받고 그 입력값에 따라 각각의 (x, y) 좌표로 터틀봇 이동한다.
    - 터틀봇에 USB 카메라를 장착하 OpenCV 라이브러리를 이용해 AR마커를 인식하고 마커에 저장되어 있는 x, y 좌표로 이동한다.

## 2. 작업순서

[간트차트](https://www.notion.so/1416181321a04eb3aafdf834ef693d48?pvs=21)

[칸반보드](https://www.notion.so/7e9d3aad44224ccdb04ac573c0c7cbfa?pvs=21)

## 3. 다이어그램

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/de458be3-9f2d-41fc-b9d0-2f0b628fafc9/d7221698-bf19-4cdd-bbf7-b15fa24d6a2a/Untitled.jpeg)

## 4. 소스코드 분석

- 코드 분석
- 전체 코드
    - getchar.py
        
        ```python
        import os, time, sys, termios, atexit, tty
        from select import select
          
        # class for checking keyboard input
        class GetChar:
            def __init__(self):
                # Save the terminal settings
                self.fd = sys.stdin.fileno()
                self.new_term = termios.tcgetattr(self.fd)
                self.old_term = termios.tcgetattr(self.fd)
          
                # New terminal setting unbuffered
                self.new_term[3] = (self.new_term[3] & ~termios.ICANON & ~termios.ECHO)
                termios.tcsetattr(self.fd, termios.TCSAFLUSH, self.new_term)
          
                # Support normal-terminal reset at exit
                atexit.register(self.set_normal_term)      
              
            def set_normal_term(self):
                termios.tcsetattr(self.fd, termios.TCSAFLUSH, self.old_term)
          
            def getch(self):        # get 1 byte from stdin
                """ Returns a keyboard character after getch() has been called """
                return sys.stdin.read(1)
          
            def chk_stdin(self):    # check keyboard input
                """ Returns True if keyboard character was hit, False otherwise. """
                dr, dw, de = select([sys.stdin], [], [], 0)
                return dr
        ```
        
    - pub_nav_msg.py
        
        ```python
        import rclpy
        from rclpy.node import Node
        
        from std_msgs.msg import String
        from arduino.getchar import Getchar
        
        class PubNAV_MSG(Node):
        
            def __init__(self):
                super().__init__('pub_nav_msg')
                self.pub_nav = self.create_publisher(String, 'nav_msg', 10)
                self.nav_msg = String()
                
            def pub_nav_msg(self, nav_msg):
                msg = String()
                msg.data = nav_msg
                self.pub_nav.publish(msg)
        
        def main(args=None):
            rclpy.init(args=args)
        
            node = PubNAV_MSG()
        
            #rclpy.spin(node)
            try:
                kb = Getchar()
                key =''
                while rclpy.ok():
                    key = kb.getch()
                    if key == '1':
                        node.pub_nav_msg('point1')
                    elif key == '2':
                        node.pub_nav_msg('point2')
                    elif key == '3':
                        node.pub_nav_msg('point3')
                    elif key == '4':
                        node.pub_nav_msg('point4')
                    else:
                        pass
            except KeyboardInterrupt:
            # Destroy the node explicitly
            # (optional - otherwise it will be done automatically
            # when the garbage collector destroys the node object)
            
                    node.destroy_node()
                    rclpy.shutdown()
        
        if __name__ == '__main__':
            main()
        
        ```
        
    - sub_n_follow.py
        
        ```python
        import rclpy
        from rclpy.node import Node
        from geometry_msgs.msg import PoseStamped
        from rclpy.action import ActionClient
        from action_msgs.msg import GoalStatus
        from nav2_msgs.action import FollowWaypoints
        # from rclpy.duration import Duration # Handles time for ROS 2
        
        from rclpy.qos import QoSProfile
        from std_msgs.msg import String
        
        class Sub_n_Follow(Node):
        
            def __init__(self):
                super().__init__('sub_n_follow')
                self._client = ActionClient(self, FollowWaypoints, '/FollowWaypoints')
                qos_profile = QoSProfile(depth=10)
                self.create_subscription( String, '/nav_msg', self.get_nav_msg, qos_profile )
                self.nav_msg = String()
            def send_points(self, points):
                msg = FollowWaypoints.Goal()
                msg.poses = points
                
                self._client.wait_for_server()
                self._send_goal_future = self._client.send_goal_async(msg, feedback_callback=self.feedback_callback)
                self._send_goal_future.add_done_callback(self.goal_response_callback)
        
            def goal_response_callback(self, future):
                goal_handle = future.result()
                if not goal_handle.accepted:
                    self.get_logger().info('Goal rejected')
                    return
        
                self.get_logger().info('Goal accepted')
        
                self._get_result_future = goal_handle.get_result_async()
                self._get_result_future.add_done_callback(self.get_result_callback)
        
            def get_result_callback(self, future):
                result = future.result().result
                self.get_logger().info('Result: {0}'.format(result.missed_waypoints))
        
            def feedback_callback(self, feedback_msg):
                feedback = feedback_msg.feedback
                self.get_logger().info('Received feedback: {0}'.format(feedback.current_waypoint))
                            
            def get_nav_msg(self, msg):
                self.nav_msg = msg
        
        def main(args=None):
            rclpy.init(args=args)
            node = Sub_n_Follow()
            rgoal = PoseStamped()
            try:   
                while rclpy.ok():
                    rclpy.spin_once(node, timeout_sec = 0.1)
                    
                    if node.nav_msg.data == "point1":
                        rgoal.header.frame_id = "map"
                        rgoal.header.stamp.sec = 0
                        rgoal.header.stamp.nanosec = 0
                        rgoal.pose.position.z = 0.0
                        rgoal.pose.position.x = 2.85
                        rgoal.pose.position.y = 2.64
                        rgoal.pose.orientation.w = 1.0
                        
                    elif node.nav_msg.data == "point2":
                        rgoal.header.frame_id = "map"
                        rgoal.header.stamp.sec = 0
                        rgoal.header.stamp.nanosec = 0
                        rgoal.pose.position.z = 0.0
                        rgoal.pose.position.x = 2.85
                        rgoal.pose.position.y = -0.9
                        rgoal.pose.orientation.w = 1.0
                        
                    elif node.nav_msg.data == "point3":
                        rgoal.header.frame_id = "map"
                        rgoal.header.stamp.sec = 0
                        rgoal.header.stamp.nanosec = 0
                        rgoal.pose.position.z = 0.0
                        rgoal.pose.position.x = 1.73
                        rgoal.pose.position.y = -0.9
                        rgoal.pose.orientation.w = 1.0
                        
                    elif node.nav_msg.data == "point4":
                        rgoal.header.frame_id = "map"
                        rgoal.header.stamp.sec = 0
                        rgoal.header.stamp.nanosec = 0
                        rgoal.pose.position.z = 0.0
                        rgoal.pose.position.x = 1.73
                        rgoal.pose.position.y = 2.6
                        rgoal.pose.orientation.w = 1.0
                    else:
                        pass
                    print(rgoal)
                    mgoal = [rgoal]
                    node.send_points(mgoal)
                    
                    
                    
            except KeyboardInterrupt:
            
            # Destroy the node explicitly
            # (optional - otherwise it will be done automatically
            # when the garbage collector destroys the node object)
            
                    node.destroy_node()
                    rclpy.shutdown()
        
        if __name__ == '__main__':
            main()
        
        ```
        
    - setup.py
        
        ```python
        from setuptools import find_packages
        from setuptools import setup
        
        package_name = 'nav_pkg'
        
        setup(
            name=package_name,
            version='0.0.0',
            packages=[package_name],
            data_files=[
                ('share/ament_index/resource_index/packages',
                    ['resource/' + package_name]),
                ('share/' + package_name, ['package.xml']),
            ],
            install_requires=['setuptools'],
            zip_safe=True,
            maintainer='gnd0',
            maintainer_email='greattoe@gmail.com',
            description='TODO: Package description',
            license='TODO: License declaration',
            tests_require=['pytest'],
            entry_points={
                'console_scripts': [
                        'pub_nav_msg = nav_pkg.pub_nav_msg:main',
                        'sub_n_follow = nav_pkg.sub_n_follow:main',
                ],
            },
        )
        
        ```
        

## 5. 프로젝트 성과 결과

- Gazebo 및 SLAM과 같은 도구를 사용하여 터틀봇을 제어하고, 이를 통해 ROS 관련 지식을 확보했다.
- 시리얼 통신을 활용하여 터틀봇을 제어했다.
- 네트워크와 관련된 SSH 연결로 PC에서 터틀봇에 장착된 라즈베리파이에 접속하여 파일을 조작했다.
- 파이썬 문법을 익혔다.
- Ubuntu 운영체제를 사용하여 프로젝트를 진행 했다.

## 6. 아쉬운점

1. ROS를 구동하기에는 컴퓨터 사양이 좋지 않아 자주 튕겼다.
2. AR마커를 인식하고 싶었지만 구현하는 데 어려움을 겪었다.

## 7. 참고

https://github.com/greattoe/ros2tutorial.git

## 8. 시연

[터틀봇_네비게이션](https://youtu.be/Mtbn5j1OkVw)
