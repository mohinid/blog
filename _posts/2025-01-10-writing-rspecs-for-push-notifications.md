---
layout: post
title: Testing Push Notifications
subtitle: RSpecs for End to End testing using Firebase push notification service as an example.
tags: [unit testing, rspecs, firebase, push notifications, ruby on rails]
comments: true  
thumbnail-img: /assets/img/testing.jpeg
---

Writing tests for a Rails application involves creating specs for models, controllers, and business logic. Let’s break this down with an example focused on implementing push notifications using Firebase as an example.

<p align="center" >
  <img src="/assets/img/push-notifications.gif" alt="push notification"/>
</p>

---

### **1. Model Specs**
Model specs to test validations, associations, and methods.

#### Example: Model `Notification`
```ruby
# app/models/notification.rb
class Notification < ApplicationRecord
  validates :title, presence: true
  validates :message, presence: true
  validates :user_id, presence: true

  belongs_to :user
end
```

#### RSpec for `Notification`
```ruby
# spec/models/notification_spec.rb
require 'rails_helper'

RSpec.describe Notification, type: :model do
  describe "validations" do
    it { should validate_presence_of(:title) }
    it { should validate_presence_of(:message) }
    it { should validate_presence_of(:user_id) }
  end

  describe "associations" do
    it { should belong_to(:user) }
  end
end
```

---

### **2. Controller Specs**
Controller specs to test request and response behavior.

#### Example: NotificationsController
```ruby
# app/controllers/notifications_controller.rb
class NotificationsController < ApplicationController
  def create
    notification = Notification.new(notification_params)

    if notification.save
      FirebaseService.new.send_notification(notification)
      render json: { success: true, message: "Notification sent." }, status: :created
    else
      render json: { success: false, errors: notification.errors.full_messages }, status: :unprocessable_entity
    end
  end

  private

  def notification_params
    params.require(:notification).permit(:title, :message, :user_id)
  end
end
```

#### RSpec for Controller
```ruby
# spec/controllers/notifications_controller_spec.rb
require 'rails_helper'

RSpec.describe NotificationsController, type: :controller do
  describe "POST #create" do
    let(:valid_params) { { notification: { title: "Test Title", message: "Test Message", user_id: 1 } } }
    let(:invalid_params) { { notification: { title: "", message: "", user_id: nil } } }

    it "creates a notification with valid params" do
      expect {
        post :create, params: valid_params
      }.to change(Notification, :count).by(1)
      
      expect(response).to have_http_status(:created)
      expect(JSON.parse(response.body)["success"]).to be true
    end

    it "does not create a notification with invalid params" do
      expect {
        post :create, params: invalid_params
      }.not_to change(Notification, :count)

      expect(response).to have_http_status(:unprocessable_entity)
      expect(JSON.parse(response.body)["success"]).to be false
    end
  end
end
```

---

### **3. Business Logic Specs**
For reusable logic, like sending notifications via Firebase, we use service objects.

#### Example: FirebaseService
```ruby
# app/services/firebase_service.rb
class FirebaseService
  def send_notification(notification)
    firebase_url = "https://fcm.googleapis.com/fcm/send"
    headers = {
      "Authorization" => "key=#{ENV['FIREBASE_SERVER_KEY']}",
      "Content-Type" => "application/json"
    }
    payload = {
      to: notification.user.device_token,
      notification: {
        title: notification.title,
        body: notification.message
      }
    }

    response = Faraday.post(firebase_url, payload.to_json, headers)
    response.success?
  end
end
```

#### RSpec for FirebaseService
For testing we will use mocking using stub_request which is a method provided by the WebMock gem. It allows stubbing (fake) HTTP requests.

```ruby
# spec/services/firebase_service_spec.rb
require 'rails_helper'

RSpec.describe FirebaseService do
  describe "#send_notification" do
    let(:user) { create(:user, device_token: "fake_device_token") }
    let(:notification) { create(:notification, title: "Test Title", message: "Test Message", user: user) }
    let(:firebase_service) { FirebaseService.new }
    let(:firebase_url) { "https://fcm.googleapis.com/fcm/send" }

    before do
      stub_request(:post, firebase_url)
        .with(
          headers: { "Authorization" => "key=#{ENV['FIREBASE_SERVER_KEY']}", "Content-Type" => "application/json" },
          body: hash_including(notification: { title: notification.title, body: notification.message })
        )
        .to_return(status: 200, body: "", headers: {})
    end

    it "sends the notification to Firebase" do
      expect(firebase_service.send_notification(notification)).to be true
    end

    it "handles unsuccessful responses gracefully" do
      stub_request(:post, firebase_url).to_return(status: 500)
      expect(firebase_service.send_notification(notification)).to be false
    end
  end
end
```

---

### **Key Points**
1. **Model Tests**:
   - Focus on validations and associations.
   - Use gems like `shoulda-matchers` for concise tests.

2. **Controller Tests**:
   - Test request/response and state changes (e.g., creating records).
   - Mock external dependencies like Firebase.

3. **Business Logic Tests**:
   - Test services or jobs in isolation.
   - Use `WebMock` or `VCR` to mock API calls.

4. **Environment Variables**:
   - Use tools like `dotenv-rails` or `figaro` to manage sensitive keys like `FIREBASE_SERVER_KEY`.

With these structured tests, we'll have confidence in the reliability of our push notification system implementation! But for End to End logic(Integration) we can run tests with actual device token and live api end points as explained below.

<p align="center" >
  <img src="/assets/img/push-notifications.png" alt="push notification"/>
</p>

## Integration testing with actual services ensures our application works as expected with real external APIs. Unlike unit tests with stubs, integration tests perform real HTTP requests to the external service, verifying the end-to-end functionality.

Here’s how to approach integration testing with real external services effectively:

---

### **1. When to Use Integration Testing?**
- To ensure your application integrates correctly with third-party APIs or services like Firebase, Stripe, or Twilio.
- To validate that the actual API behaves as documented.
- To test configurations (e.g., authentication tokens, headers, or network settings).

---

### **2. Challenges in Integration Testing**
- **External dependency availability**: The service must be online.
- **Rate limits**: Frequent API calls may exceed the rate limits.
- **Cost**: Real calls may incur charges.
- **Latency**: Network delays might slow down test execution.

---

### **3. Strategies for Integration Testing**

#### a) **Setup a Separate Test Environment**
- Use API keys or credentials dedicated to testing to avoid interfering with production data.
- Many services (e.g., Firebase, Stripe) provide sandbox environments for safe testing.

#### b) **Configure RSpec or Test Suite**
Ensure your tests are organized, and only integration tests hit real APIs. Tag them appropriately:
```ruby
RSpec.describe "Firebase Notifications", :integration do
  it "sends a push notification successfully" do
    response = FirebaseService.new.send_notification(
      title: "Test Notification",
      body: "This is a test.",
      token: "valid_device_token"
    )

    expect(response.status).to eq(200)
    expect(response.body).to include("success")
  end
end
```

Run these tests separately:
```bash
rspec --tag integration
```

#### c) **Use Live API Endpoints**
Perform real API requests with minimal data:
```ruby
require 'net/http'
require 'json'

RSpec.describe "Firebase Integration", :integration do
  it "sends a notification using Firebase" do
    uri = URI("https://fcm.googleapis.com/fcm/send")
    request = Net::HTTP::Post.new(uri)
    request["Authorization"] = "key=YOUR_SERVER_KEY"
    request["Content-Type"] = "application/json"
    request.body = {
      to: "valid_device_token",
      notification: {
        title: "Integration Test",
        body: "Testing Firebase notification delivery"
      }
    }.to_json

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
      http.request(request)
    end

    expect(response.code).to eq("200")
    expect(JSON.parse(response.body)["success"]).to eq(1)
  end
end
```

---

### **4. Best Practices for Integration Testing**

1. **Keep Tests Minimal**
   - Test small, specific scenarios to avoid excessive API usage.
   - Validate success, failure, and edge cases without duplicating unit tests.

2. **Rate-Limit Handling**
   - Introduce delays between requests if necessary (e.g., `sleep 1`).
   - Use exponential backoff to retry failed tests.

3. **Environment Variables**
   - Store API keys and sensitive data in environment variables to keep them secure.
   ```ruby
   ENV["FIREBASE_SERVER_KEY"]
   ```

4. **Avoid Running Integration Tests Always**
   - Use tags (`:integration`) or a specific suite to run these tests separately:
     ```bash
     rspec --tag integration
     ```

5. **Mock Non-Critical Tests**
   - Mock non-critical API interactions in regular test runs to save time and resources.
   - Reserve real API tests for nightly or CI/CD pipelines.

6. **Error Handling**
   - Verify how your app behaves under errors like timeouts, bad requests, or unauthorized access.

---

### **5. Combine Integration with Unit Tests**
- Use **unit tests with stubs** for fast, isolated testing of business logic.
- Complement them with **integration tests** to validate real-world behavior.

---

### **6. Tools to Enhance Integration Testing**

- **VCR (optional)**: Record real API responses and replay them in subsequent runs to reduce API calls:
  ```ruby
  require 'vcr'

  VCR.configure do |config|
    config.cassette_library_dir = "spec/vcr"
    config.hook_into :webmock
  end

  RSpec.describe "Firebase Integration", :integration do
    it "sends a notification" do
      VCR.use_cassette("firebase_notification") do
        response = FirebaseService.new.send_notification(
          title: "Integration Test",
          body: "Hello, World!",
          token: "valid_device_token"
        )

        expect(response.status).to eq(200)
      end
    end
  end
  ```
- **Postman/Newman**: Use Postman for manual API testing and Newman for automated tests.

---

### **7. CI/CD Integration**
- Run integration tests in a separate stage of your CI/CD pipeline to avoid slowing down unit tests.
- Mock or disable integration tests in local development.

---

## Final Note
Testing is crucial in any system. With these above mentioned techniques we can thoroughly test our implementation end to end, making our system robust and reliable for all scenarios!
Happy testing :)