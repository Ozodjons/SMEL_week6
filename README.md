# Kunishev Ozodjon | SMEL_week6

<br />

## Step 1: Writing a simple service and client

<br />

### 1.1. Create a package

<br />

First of all, we will source our ROS2 so that its commands will work:
```
source /opt/ros/eloquent/setup.bash
```

Then, we will navigate to the ```dev_ws/src``` floder and create a package:
```
ros2 pkg create --build-type ament_cmake cpp_srvcli --dependencies rclcpp example_interfaces
```

<br/>

Then, we will open ```package.xml``` file,
```
gedit package.xml
```
and add description, maintainer email and name and license information:
```
<description>C++ client server tutorial</description>
<maintainer email="ozodjonkunishev@email.com">Ozodjon</maintainer>
<license>Apache License 2.0</license>
```

<br />

### 1.2. Write the service node

<br />

After that, we will anvigate to the ```dev_ws/src/cpp_srvcli/src``` folder, create a new file with the name of ```add_two_ints_server.cpp``` :
```
gedit add_two_ints_server.cpp
```
and write the following code inside the file:
```
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

#include <memory>

void add(const std::shared_ptr<example_interfaces::srv::AddTwoInts::Request> request,
          std::shared_ptr<example_interfaces::srv::AddTwoInts::Response>      response)
{
  response->sum = request->a + request->b;
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Incoming request\na: %ld" " b: %ld",
                request->a, request->b);
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "sending back response: [%ld]", (long int)response->sum);
}

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);

  std::shared_ptr<rclcpp::Node> node = rclcpp::Node::make_shared("add_two_ints_server");

  rclcpp::Service<example_interfaces::srv::AddTwoInts>::SharedPtr service =
    node->create_service<example_interfaces::srv::AddTwoInts>("add_two_ints", &add);

  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Ready to add two ints.");

  rclcpp::spin(node);
  rclcpp::shutdown();
}
```

<br />

Now, we will create an executable named ```server``` by adding the following code to ```CMakeLists.txt``` file:
```
add_executable(server src/add_two_ints_server.cpp)
ament_target_dependencies(server
rclcpp example_interfaces)
```
At the end of the file, we will add the following lines of code so that ```ros2 run``` can find the executable:
```
install(TARGETS
  server
  DESTINATION lib/${PROJECT_NAME})
```

<br />

### 1.3. Write the client node

<br />

First, we will navigate to the ```dev_ws/src/cpp_srvcli/src``` directory, create a new file called ```add_two_ints_client.cpp``` and write the code below inside the file:
```
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

#include <chrono>
#include <cstdlib>
#include <memory>

using namespace std::chrono_literals;

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);

  if (argc != 3) {
      RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "usage: add_two_ints_client X Y");
      return 1;
  }

  std::shared_ptr<rclcpp::Node> node = rclcpp::Node::make_shared("add_two_ints_client");
  rclcpp::Client<example_interfaces::srv::AddTwoInts>::SharedPtr client =
    node->create_client<example_interfaces::srv::AddTwoInts>("add_two_ints");

  auto request = std::make_shared<example_interfaces::srv::AddTwoInts::Request>();
  request->a = atoll(argv[1]);
  request->b = atoll(argv[2]);

  while (!client->wait_for_service(1s)) {
    if (!rclcpp::ok()) {
      RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Interrupted while waiting for the service. Exiting.");
      return 0;
    }
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "service not available, waiting again...");
  }

  auto result = client->async_send_request(request);
  // Wait for the result.
  if (rclcpp::spin_until_future_complete(node, result) ==
    rclcpp::executor::FutureReturnCode::SUCCESS)
  {
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Sum: %ld", result.get()->sum);
  } else {
    RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Failed to call service add_two_ints");
  }

  rclcpp::shutdown();
  return 0;
}
```

<br />

After that, we will add executables into the CMakeList.txt for the new node:
```
cmake_minimum_required(VERSION 3.5)
project(cpp_srvcli)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(example_interfaces REQUIRED)

add_executable(server src/add_two_ints_server.cpp)
ament_target_dependencies(server
  rclcpp example_interfaces)

add_executable(client src/add_two_ints_client.cpp)
ament_target_dependencies(client
  rclcpp example_interfaces)

install(TARGETS
  server
  client
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

<br />

### 1.4. Build and run

<br/>

Before building, we will check for missing dependencies:
```
rosdep install -i --from-path src --rosdistro eloquent -y
```
After making sure that all dependencies are installed, we navigate to the root of our workspace ```dev_ws``` and build our new package:
```
colcon build --packages-select cpp_srvcli
```

<br/>

Then, we will open  a new terminal and again navigate to our workspace directory ```dev_ws``` and source the setup files:
```
. install/setup.bash
```
After that, we run the service node:
```
ros2 run cpp_srvcli server
```
The terminal will wait until we start a client node following with any two integers in a new terminal:
```
ros2 run cpp_srvcli client 7 10
```
Then, the client will receive a response as below:
![output1](https://user-images.githubusercontent.com/90167023/193986591-d021e24b-5ed2-4918-8e01-71133e99b518.png)
>In my case, I chose 7 and 10, so the addition of 7 and 10 equals to 17.

<br/>

If we return back to the terminal whre our service node is running, we will see a message as below:
![output2](https://user-images.githubusercontent.com/90167023/193986766-dd3dacb0-a7a6-4f7c-94ce-457b176cd195.png)

We can close terminals by pressing ```ctrl+C```.

