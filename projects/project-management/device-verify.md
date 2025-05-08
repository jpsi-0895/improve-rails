If you want to verify **mobile device details** and restrict access to certain users based on their device, you can integrate device fingerprinting or device verification into your authentication process.

Here's how you can do that step by step, including tracking and verifying **mobile device details** during user login and only allowing selected users based on the device's properties.

### **Steps to Implement Mobile Device Verification:**

1. **Identify the Mobile Device**

   * To track mobile device details, you can use device-specific attributes like **device ID**, **user-agent**, **IP address**, **device model**, **OS version**, etc.
   * You can either use a JavaScript-based solution on the frontend (for a web app) or rely on device SDKs for mobile apps (iOS/Android).
   * Libraries like **FingerprintJS** or **device\_detector** (for Ruby) can help you identify a user's device.

2. **Store Device Information for Each User**

   * You will need a model to store the user's device details (e.g., device type, device ID, OS version, etc.).

3. **Verify Device During Login**

   * After login, check if the current device matches the previously stored device. If it matches, allow the user to proceed; otherwise, prevent access.

4. **Restrict Access Based on Device (Allow Only Selected Users)**

   * You can define a policy to only allow certain users to log in from specific devices.

---

### **1. Add Device Tracking Model**

To track device information, you can create a new model in your Rails app to store device details for each user.

#### **Generate Device Model**

```bash
rails generate model UserDevice user:references device_id:string device_type:string os_version:string
rails db:migrate
```

#### **UserDevice Model**

The `UserDevice` model will store information like the device's `device_id`, `device_type`, and `os_version`.

```ruby
class UserDevice < ApplicationRecord
  belongs_to :user
  validates :device_id, uniqueness: true  # Ensure only one device per user
end
```

---

### **2. Install Device Detection Gem (Optional)**

If you're building a web-based solution, you can use a Ruby gem like **device\_detector** to detect the device type.

#### **Add device\_detector Gem to Gemfile**

```ruby
gem 'device_detector'
```

Run `bundle install`.

---

### **3. Add Device Info to User Login Flow**

After user login via **Devise** and **JWT**, you need to collect and save the device details.

#### **In SessionsController (login flow)**

In the `SessionsController`, you will capture the device details on user login and check whether the user is allowed to log in from that device.

```ruby
class SessionsController < ApplicationController
  skip_before_action :authenticate_user!, only: [:create]

  def create
    user = User.find_by(email: params[:email])

    if user && user.valid_password?(params[:password])
      # Capture device details
      device_info = request.user_agent # For web-based user agent string
      device_detector = DeviceDetector.new(device_info)
      device = device_detector.device_type  # Device type (mobile, desktop, etc.)
      os = device_detector.os_name  # OS name (iOS, Android, etc.)
      device_id = request.env["HTTP_X_DEVICE_ID"] || SecureRandom.hex(16)  # You can pass custom device ID from frontend (optional)

      # Store device information for future verification
      user_device = UserDevice.find_or_initialize_by(user: user, device_id: device_id)
      user_device.device_type = device
      user_device.os_version = os
      user_device.save

      # JWT token generation
      token = AuthenticationService.encode(user.id)
      render json: { token: token, user: user }
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end
end
```

### **Explanation:**

* **Device Detection:** Using the **DeviceDetector** gem, we capture the `device_type` and `os_version` from the user-agent string sent by the browser.
* **Device ID:** We can send a **device\_id** from the frontend (a unique identifier per device) via the `X-DEVICE-ID` header. If you don't have a specific device ID, you can generate a random one using `SecureRandom.hex`.
* **Saving Device Info:** The captured device details are stored in the `UserDevice` model, where we associate the device with the user. This lets us track the user's devices and ensure only selected devices are allowed.

---

### **4. Verify Device During Login**

On subsequent logins, you will check if the user is trying to access the system from an authorized device. If the device ID doesn’t match the previously stored one, you can block access.

#### **Check Device on Subsequent Login**

In the `SessionsController`, after user authentication, check if the device matches any stored device for that user:

```ruby
class SessionsController < ApplicationController
  skip_before_action :authenticate_user!, only: [:create]

  def create
    user = User.find_by(email: params[:email])

    if user && user.valid_password?(params[:password])
      # Capture device details again
      device_info = request.user_agent
      device_detector = DeviceDetector.new(device_info)
      device = device_detector.device_type
      os = device_detector.os_name
      device_id = request.env["HTTP_X_DEVICE_ID"] || SecureRandom.hex(16)

      # Check if device matches
      user_device = user.user_devices.find_by(device_id: device_id)
      if user_device.nil?
        render json: { error: 'Device not authorized' }, status: :unauthorized
        return
      end

      # Generate and return JWT token
      token = AuthenticationService.encode(user.id)
      render json: { token: token, user: user }
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end
end
```

### **Explanation:**

* During login, the system checks if the device ID sent in the request (`X-DEVICE-ID`) matches any stored device ID for that user.
* If the device is not authorized (i.e., not found in `user.user_devices`), the login will be rejected with an error message (`Device not authorized`).

---

### **5. Restrict Access to Selected Users (Optional)**

If you only want to allow **selected users** (e.g., specific admin users) to access from certain devices, you can add a custom check for that:

```ruby
# In the authentication process, after device verification
if user.role == "selected" && user_device.device_type != "mobile"
  render json: { error: "Access restricted to mobile devices" }, status: :unauthorized
  return
end
```

This ensures that **only selected users** (e.g., users with the "selected" role) can log in from specific device types.

---

### **6. Frontend Changes (Optional)**

If you're building a frontend (e.g., a mobile app or web app), you will need to pass the **device ID** to the backend. Here's how you could handle it:

* For **Web App**: Use JavaScript libraries like **FingerprintJS** to generate a unique device ID.
* For **Mobile App**: Use native device libraries (e.g., Android's `Settings.Secure.ANDROID_ID`, iOS's `UIDevice.identifierForVendor`) to generate a device ID.

**Example:**
In a web app, you could use **FingerprintJS** to create a unique ID:

```js
import FingerprintJS from '@fingerprintjs/fingerprintjs';

async function getDeviceID() {
  const fp = await FingerprintJS.load();
  const result = await fp.get();
  return result.visitorId;  // This is the unique device ID
}

// Add this device ID to the Authorization header
fetch('http://localhost:3000/sessions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${userToken}`,
    'X-DEVICE-ID': await getDeviceID(),
  },
  body: JSON.stringify({ email: 'user@example.com', password: 'password' })
})
.then(response => response.json())
.then(data => console.log(data));
```

---

### **Conclusion**

By following this approach:

1. **Track devices** using information like device ID, OS version, and device type.
2. **Store device details** in the database for each user.
3. **Verify device details** during each login to ensure that only **authorized devices** can access the account.
4. Optionally, you can **restrict certain users** to specific devices.

This method ensures that only **selected users** can access your app from their authorized devices. You can extend the logic to include additional checks, such as geolocation, IP verification, or more detailed device attributes. Let me know if you need further help!
