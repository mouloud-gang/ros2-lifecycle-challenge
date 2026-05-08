source /opt/ros/jazzy/setup.bash

source ~/ros2_ws/install/setup.bash

cd ~/ros2_ws

ros2 pkg create --build-type ament_python lifecycle_pkg

cd lifecycle_pkg/lifecycle_pkg

nano lifecycle_node.py

----paste:

import rclpy

from rclpy.lifecycle import Node

from rclpy.lifecycle import State

from rclpy.lifecycle import TransitionCallbackReturn

from std_msgs.msg import String


class MonLifecycle(Node):

    def __init__(self):
        super().__init__('mon_lifecycle')

        self._pub = None
        self._timer = None

    def on_configure(self, state: State):
        self.get_logger().info('Configured')
        return TransitionCallbackReturn.SUCCESS

    def on_activate(self, state: State):
        self.get_logger().info('Activated')

        self._pub = self.create_publisher(String, '/actif', 10)

        self._timer = self.create_timer(
            1.0,
            self._publish
        )

        return TransitionCallbackReturn.SUCCESS

    def on_deactivate(self, state: State):
        self.get_logger().info('Deactivated')

        self.destroy_timer(self._timer)
        self.destroy_publisher(self._pub)

        self._timer = None
        self._pub = None

        return TransitionCallbackReturn.SUCCESS

    def on_cleanup(self, state: State):
        self.get_logger().info('Cleanup')
        return TransitionCallbackReturn.SUCCESS

    def _publish(self):
        msg = String()
        msg.data = 'Je suis actif!'

        self._pub.publish(msg)

        self.get_logger().info(msg.data)


def main(args=None):

    rclpy.init(args=args)

    node = MonLifecycle()

    rclpy.spin(node)

    rclpy.shutdown() 
    if __name__ == '__main__':
    main()
    ------------------------------------
chmod +x lifecycle_node.py

cd ~/ros2_ws/src/lifecycle_pkg

nano setup.py

------remplace avec:

    entry_points={
   
    'console_scripts': [
      'lifecycle_node = lifecycle_pkg.lifecycle_node:main',
    ],
    },
-------------------------------------
nano package.xml

ajoute:

 `<depend>rclpy</depend>`

 `<depend>std_msgs</depend>`

apres

 `<license> TODO: License declaration</license>`
 
-------------------------------------
cd ~/ros2_ws

colcon build

source install/setup.bash

ros2 run lifecycle_pkg lifecycle_node

-----dans le 2eme terminal:

source /opt/ros/jazzy/setup.bash

source ~/ros2_ws/install/setup.bash

ros2 lifecycle get /mon_lifecycle

----on a le message:

unconfigured [1]

ros2 lifecycle set /mon_lifecycle configure

----on a le message:

Transitioning successful

ros2 lifecycle get /mon_lifecycle


----on a le message:

inactive [2]

ros2 lifecycle set /mon_lifecycle activate

----on a le message:

Transitioning successful

ros2 topic echo /actif

----on a le message:

data: Je suis actif!
---
data: Je suis actif!
---
data: Je suis actif!
---
data: Je suis actif!
---
data: Je suis actif!
---
data: Je suis actif!
ros2 lifecycle set /mon_lifecycle deactivate


    


if __name__ == '__main__':
    main()
