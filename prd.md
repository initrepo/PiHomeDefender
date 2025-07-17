## PiHomeDefender: Product Requirements Document (PRD)

**1. Executive Summary**

*   **Product Vision:** PiHomeDefender is a comprehensive, self-hosted home security and automation solution built on a serverless architecture (Next.js, Convex, Vercel). It empowers users with complete control over their home's security and privacy, offering a cost-effective and feature-rich alternative to traditional security systems.

*   **Value Proposition:** PiHomeDefender offers:
    *   **Privacy:** All data is stored locally on the user's Raspberry Pi, eliminating reliance on cloud services and protecting sensitive information.
    *   **Control:** Users have complete control over their security system, including device management, data access, and system configuration.
    *   **Cost Savings:** Eliminates recurring monthly fees associated with traditional security systems.
    *   **Intelligence:** Leverages AI to provide intelligent alerts, anomaly detection, and proactive security recommendations.
    *   **Customization:** Highly customizable and extensible, allowing users to integrate various sensors, cameras, and smart home devices.
    *   **Ease of Use:** A user-friendly, mobile-first web application simplifies setup and daily management.

*   **Target User Segments and Use Cases:**
    *   **Tech-Savvy Homeowners:** Individuals who value privacy, control, and customization.
    *   **DIY Enthusiasts:** Users with experience setting up and managing Raspberry Pi devices.
    *   **Renters and Homeowners:** Users who desire a cost-effective and feature-rich home security solution.
    *   **Use Cases:**
        *   **Real-time Monitoring:** View live camera feeds, sensor status, and network activity.
        *   **Anomaly Detection:** Receive alerts for unusual events, such as unauthorized access attempts or suspicious network activity.
        *   **Remote Control:** Control smart home devices (lights, outlets) from anywhere.
        *   **Event History:** Review past security events and camera recordings.
        *   **Security Recommendations:** Receive AI-powered suggestions to improve security coverage.

*   **Success Criteria and KPIs:**
    *   **User Acquisition:**
        *   Number of registered users.
        *   Conversion rate from trial/free users to paid subscriptions.
        *   Website traffic and engagement (e.g., time on site, bounce rate).
    *   **User Engagement:**
        *   Monthly/Weekly Active Users (MAU/WAU).
        *   Average session duration.
        *   Feature usage (e.g., number of cameras connected, sensor integration).
    *   **Retention:**
        *   Churn rate (percentage of users who cancel subscriptions).
        *   Customer lifetime value (CLTV).
    *   **Technical Performance:**
        *   Page load time (under 3 seconds).
        *   Serverless function execution time.
        *   Database query performance.
        *   System uptime.
    *   **Customer Satisfaction:**
        *   Customer satisfaction score (CSAT).
        *   Net Promoter Score (NPS).
        *   Number of support tickets.
    *   **Financial:**
        *   Monthly Recurring Revenue (MRR).
        *   Customer Acquisition Cost (CAC).
        *   Profitability.

**2. User Experience Requirements**

*   **Core User Flows:**
    *   **Onboarding:**
        1.  User registers and logs in.
        2.  Guided setup for the Raspberry Pi and system configuration (e.g., network setup, device discovery).
        3.  User connects security sensors, cameras, and smart home devices.
        4.  User configures AI settings and notifications.
    *   **Dashboard:**
        1.  Real-time display of sensor status, camera feeds, and network activity.
        2.  Alerts and notifications displayed.
        3.  Quick actions (e.g., arm/disarm system, control lights).
    *   **Device Management:**
        1.  Add, edit, and remove devices (sensors, cameras, smart home devices).
        2.  Configure device settings (e.g., camera resolution, sensor sensitivity).
    *   **Event History:**
        1.  View a chronological list of security events.
        2.  Filter events by type, date, and device.
        3.  View associated camera recordings.
    *   **AI Co-Pilot:**
        1.  View AI-generated alerts and summaries.
        2.  Review AI-powered recommendations to improve security coverage.
    *   **Subscription Management:** (If applicable, see section 4)
        1.  View subscription details.
        2.  Upgrade/downgrade plans.
        3.  Update payment information.

*   **Mobile-Responsive Design Requirements (Next.js):**
    *   **Layout:** Use a mobile-first approach with responsive layouts using CSS-in-JS (e.g., styled-components, emotion) or a CSS framework (e.g., Tailwind CSS, Material UI).
    *   **Navigation:** Implement a responsive navigation menu (e.g., hamburger menu for smaller screens).
    *   **Typography:** Use relative units (e.g., rem, em) for font sizes to ensure scalability.
    *   **Images:** Optimize images for different screen sizes using the `next/image` component.
    *   **Touch Gestures:** Ensure touch-friendly interactions for mobile devices.
    *   **Testing:** Thoroughly test the application on various mobile devices and screen sizes.

*   **Progressive Web App (PWA) Considerations:**
    *   **Manifest File:** Create a `manifest.json` file to define the app's name, icons, and other metadata for installation on the user's device.
    *   **Service Worker:** Implement a service worker to enable offline functionality, caching, and background sync.
    *   **HTTPS:** Ensure the application is served over HTTPS for security and PWA compatibility.
    *   **Install Prompt:** Implement a prompt to encourage users to install the app on their home screen.
    *   **Offline Support:** Design the app to provide a basic offline experience (e.g., access to cached data, ability to view event history).

**3. Functional Requirements by Feature**

*   **Feature: Real-time Data Synchronization**

    *   **User Story:** As a user, I want the dashboard to update in real-time so I can monitor the security of my home instantly.
    *   **Acceptance Criteria:**
        *   Sensor status (e.g., door open/closed, motion detected) updates within 1 second.
        *   Camera feeds display live video with minimal latency.
        *   Notifications appear immediately after an event is triggered.
    *   **Next.js Routing and Page Requirements:**
        *   Dashboard page (`/dashboard`).
        *   Use `useEffect` and `useState` hooks to manage real-time data updates.
    *   **Convex Schema and Function Specifications:**
        *   **Schema:**
            ```typescript
            // convex/schema.ts
            import { defineSchema, defineTable } from "convex/server";
            import { v } from "convex/values";

            export default defineSchema({
                sensors: defineTable({
                    deviceId: v.string(),
                    sensorType: v.string(), // "door", "window", "motion", etc.
                    status: v.string(), // "open", "closed", "active", "inactive"
                    lastUpdated: v.number(),
                }).index("by_deviceId", ["deviceId"]),
                cameras: defineTable({
                    deviceId: v.string(),
                    cameraName: v.string(),
                    streamUrl: v.string(), // URL for RTSP stream
                    lastUpdated: v.number(),
                }).index("by_deviceId", ["deviceId"]),
                networkActivity: defineTable({
                  timestamp: v.number(),
                  type: v.string(), // "intrusion", "device_connected", "malicious_site_blocked"
                  description: v.string(),
                  details: v.optional(v.any()),
                }),
                alerts: defineTable({
                  timestamp: v.number(),
                  type: v.string(), // "intrusion", "device_connected", "malicious_site_blocked"
                  description: v.string(),
                  details: v.optional(v.any()),
                  read: v.boolean(),
                }),
            });
            ```
        *   **Functions:**
            *   **`convex/sensors/updateSensorStatus.ts`:** (Mutation)
                *   Updates the `sensors` table with new sensor status data.
                *   Triggered by sensor events from the Raspberry Pi.
            *   **`convex/cameras/updateCameraStream.ts`:** (Mutation)
                *   Updates the `cameras` table with new camera stream data.
                *   Triggered by camera events from the Raspberry Pi.
            *   **`convex/networkActivity/logNetworkEvent.ts`:** (Mutation)
                *   Logs network events such as intrusions, device connections, and malicious site blocks.
            *   **`convex/alerts/createAlert.ts`:** (Mutation)
                *   Creates a new alert.
            *   **`convex/alerts/markAlertAsRead.ts`:** (Mutation)
                *   Marks an alert as read.
            *   **`convex/alerts/getUnreadAlerts.ts`:** (Query)
                *   Retrieves unread alerts.
        *   **Real-time Updates:** Use Convex's `useLiveQuery` hook in the Next.js dashboard component to subscribe to real-time updates from the `sensors`, `cameras`, and `alerts` tables.

*   **Feature: Device Management Portal**

    *   **User Story:** As a user, I want to be able to add, configure, and manage my security sensors, cameras, and smart home devices through a user-friendly interface.
    *   **Acceptance Criteria:**
        *   Users can add new devices by specifying device type, ID, and configuration settings.
        *   Users can edit device settings (e.g., sensor sensitivity, camera resolution).
        *   Users can view a list of connected devices and their status.
        *   The portal provides clear instructions and error messages.
    *   **Next.js Routing and Page Requirements:**
        *   Device Management page (`/devices`).
        *   Pages for adding/editing individual devices (e.g., `/devices/add`, `/devices/[deviceId]/edit`).
        *   Use forms for data input and validation.
    *   **Convex Schema and Function Specifications:**
        *   **Schema:** (Extend the schema above)
            ```typescript
            // convex/schema.ts
            import { defineSchema, defineTable } from "convex/server";
            import { v } from "convex/values";

            export default defineSchema({
                // ... (previous schema)
                devices: defineTable({
                    deviceId: v.string(),
                    deviceType: v.string(), // "sensor", "camera", "smartplug"
                    deviceName: v.string(),
                    configuration: v.object({
                        // Device-specific configuration (e.g., camera resolution, sensor sensitivity)
                    }),
                }).index("by_deviceType", ["deviceType"]),
            });
            ```
        *   **Functions:**
            *   **`convex/devices/addDevice.ts`:** (Mutation)
                *   Adds a new device to the `devices` table.
            *   **`convex/devices/getDevices.ts`:** (Query)
                *   Retrieves a list of all devices.
            *   **`convex/devices/getDevice.ts`:** (Query)
                *   Retrieves a specific device by its ID.
            *   **`convex/devices/updateDevice.ts`:** (Mutation)
                *   Updates the configuration of an existing device.
            *   **`convex/devices/deleteDevice.ts`:** (Mutation)
                *   Deletes a device.
        *   **Real-time Updates:** Use `useLiveQuery` to update the device list in real-time when devices are added, updated, or deleted.

*   **Feature: Local Network DVR Functionality**

    *   **User Story:** As a user, I want the ability to record video from my cameras and store it locally on an external drive, ensuring my privacy.
    *   **Acceptance Criteria:**
        *   Users can configure recording schedules and storage settings.
        *   Video recordings are stored on a user-provided external USB drive.
        *   Users can view and download recorded video footage.
        *   The system supports H.264 or H.265 video codecs for efficient storage.
    *   **Next.js Routing and Page Requirements:**
        *   DVR settings page (`/dvr`).
        *   Page for viewing and browsing video recordings (`/recordings`).
        *   Implement video player component (e.g., using HTML5 `<video>` tag).
    *   **Convex Schema and Function Specifications:**
        *   **Schema:** (Extend the schema)
            ```typescript
            // convex/schema.ts
            import { defineSchema, defineTable } from "convex/server";
            import { v } from "convex/values";

            export default defineSchema({
                // ... (previous schema)
                recordings: defineTable({
                    cameraId: v.string(),
                    startTime: v.number(), // Unix timestamp
                    endTime: v.number(), // Unix timestamp
                    videoUrl: v.string(), // URL of the video file on the external drive
                    thumbnailUrl: v.string(),
                    description: v.optional(v.string()),
                }).index("by_cameraId", ["cameraId"]),
                dvrSettings: defineTable({
                    storagePath: v.string(),
                    recordingSchedule: v.object({
                      // Configure recording schedule
                    }),
                    // ... other settings
                }),
            });
            ```
        *   **Functions:**
            *   **`convex/dvr/updateDvrSettings.ts`:** (Mutation)
                *   Updates the DVR settings.
            *   **`convex/recordings/getRecordings.ts`:** (Query)
                *   Retrieves a list of video recordings, filtered by camera and time range.
            *   **`convex/recordings/createRecording.ts`:** (Mutation)
                *   Creates a new recording entry after the recording is done.
            *   **`convex/recordings/deleteRecording.ts`:** (Mutation)
                *   Deletes a recording.
            *   **`convex/recordings/getRecording.ts`:** (Query)
                *   Gets one recording.
        *   **Real-time Updates:** Use `useLiveQuery` to update the recordings list when new recordings are available.
        *   **Backend Processing:**
            *   Develop a service on the Raspberry Pi (outside of Convex) to:
                *   Capture video streams from cameras.
                *   Save recordings to the external USB drive.
                *   Generate thumbnails for recordings.
                *   Upload recording metadata to Convex using the `createRecording` mutation.
            *   (Optional) Use a serverless function to handle thumbnail generation if needed.

*   **Feature: Real-time Anomaly Detection**

    *   **User Story:** As a user, I want the system to automatically detect anomalies in sensor data, network activity, and camera feeds to alert me to potential security threats.
    *   **Acceptance Criteria:**
        *   The system detects unusual events (e.g., unexpected motion, network intrusion attempts).
        *   Alerts are generated and displayed in the dashboard.
        *   Alerts include relevant details (e.g., time, location, event type).
    *   **Next.js Routing and Page Requirements:**
        *   Alerts are displayed in the dashboard.
    *   **Convex Schema and Function Specifications:** (See Real-time Data Synchronization)
        *   **Schema:** (See Real-time Data Synchronization)
        *   **Functions:**
            *   **`convex/alerts/createAlert.ts`:** (Mutation) - Called when an anomaly is detected.
            *   **Background Processing:**
                *   Implement anomaly detection logic as a service on the Raspberry Pi.
                *   The service should analyze sensor data, network activity logs, and camera feeds in real-time.
                *   When an anomaly is detected, the service calls the `createAlert` mutation to generate an alert.
        *   **Real-time Updates:** Use `useLiveQuery` to display alerts in real time.

*   **Feature: AI-Powered Alert Summarization and Action Plan Generation**

    *   **User Story:** As a user, I want the system to provide concise summaries of security events and suggest actions I can take to address potential threats.
    *   **Acceptance Criteria:**
        *   Alerts are summarized in a clear and concise manner.
        *   The system suggests relevant actions (e.g., "Check camera feed," "Contact law enforcement").
        *   The AI generates the alert summary within 2 seconds.
    *   **Next.js Routing and Page Requirements:**
        *   Alerts are displayed in the dashboard.
    *   **Convex Schema and Function Specifications:** (See Real-time Data Synchronization)
        *   **Schema:** (See Real-time Data Synchronization)
        *   **Functions:**
            *   **`convex/alerts/createAlert.ts`:** (Mutation) - Called when an anomaly is detected.
            *   **`convex/alerts/getAlertSummary.ts`:** (Query)
                *   Retrieves an alert summary (e.g., using OpenAI API).
            *   **Background Processing:**
                *   When an alert is created, trigger a background function (e.g., using a queue, or on-device processing) to:
                    *   Analyze the alert data.