### Memory Handling & Access
- **Pass by Const Reference (`const Type&`):** This is the "gold standard" for passing objects like Quaternions. It prevents a **copy** (saving CPU/RAM) while the `const` prevents the function from modifying the original data.
	- `Type &` and `Type&` same
- **References vs. Pointers:** * **References (`&`):** Used for mandatory data. They cannot be null and have cleaner syntax (`object.member`).
    - **Pointers (`*`):** Used only if the data is optional (`nullptr`) or needs to be reassigned to a different memory address later.
- **The `this` Pointer:** Used in **Lambdas** (`[this]`) to give a small, anonymous function permission to access the private variables and methods of the current class instance.
### The Constructor & Initialiser List
- **Member Initialiser List:** The syntax following the colon (`:`) in a constructor. It is **essential** because it calls the **constructors** for each private member variable before the body `{}` of the function even starts.
- **Initialisation vs. Assignment:** 
	- **Initialisation:** Happening in the list, the variable is "born" with its value. This is mandatory for `const` and `reference` members.
    - **Assignment:** Happening in the `{}` braces, the variable is created with a default value and then overwritten, which is less efficient.
- **Base Class Initialisation:** If your class inherits (e.g., `class Node : public rclcpp::Node`), you must pass the parent's required parameters in this list (e.g., `: rclcpp::Node("name")`)

```cpp
#include <rclcpp/rclcpp.hpp>
#include <memory>

// 1. A simple class to represent our math logic
class ControllerMath {
public:
    // Pass by Const Reference: No copying, but 'target' is safe from modification
    float compute_error(const float& current, const float& target) {
        return target - current;
    }
};

// 2. Our Node inheriting from rclcpp::Node
class ViperNode : public rclcpp::Node {
public:
    // The Constructor Signature
    ViperNode() 
    : rclcpp::Node("viper_node")     // Base Class Initialization (Mandatory)
    , _kp{1.5f}                       // Uniform Initialization (Safe)
    , _current_val{0.0f}              // Member Initializer List (Efficient)
    , _math_unit{}                    // Calling Default Constructor for an object
    {
        // Body of the constructor for "active" logic
        RCLCPP_INFO(this->get_logger(), "Viper Node Initialized with Kp: %.2f", _kp);
    }

	// Public method
    void run_control(float target) {
        // Using our member object and its reference-based method
        float error = _math_unit.compute_error(_current_val, target);
        float output = error * _kp;
    }

private:
    // Private Members
    const float _kp;           // Must be initialized in the list (cannot be assigned in {}) since const
    float _current_val;
    ControllerMath _math_unit; // Member object constructed via the list
};

int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<ViperNode>(); // Triggers the construction chain
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```
