## PiHomeDefender: Technical Architecture Document

This document outlines the technical architecture for PiHomeDefender, a serverless home security and automation suite built with Next.js, Convex, and Vercel. It addresses all requirements, including the complex conditional logic for authentication, payments, and subscription management using Clerk.

### 1. Frontend Architecture (Next.js 14+)

#### 1.1. App Router Structure and Page Organization

The application will leverage the Next.js App Router for a modern, performance-focused structure.

```
pihomedefender/
├── app/
│   ├── (auth)/                    # Authentication-related routes (protected by Clerk)
│   │   ├── login/                 # Login page
│   │   ├── signup/                # Signup page
│   │   └── layout.tsx             # Layout for all auth routes
│   ├── (dashboard)/               # Dashboard routes (protected by Clerk)
│   │   ├── layout.tsx             # Dashboard layout (sidebar, header)
│   │   ├── page.tsx               # Main dashboard page
│   │   ├── sensors/               # Sensor management
│   │   │   ├── page.tsx
│   │   │   └── [sensorId]/        # Individual sensor detail
│   │   │       └── page.tsx
│   │   ├── cameras/               # Camera management
│   │   │   ├── page.tsx
│   │   │   └── [cameraId]/        # Individual camera detail
│   │   │       └── page.tsx
│   │   ├── smart-home/            # Smart home device control
│   │   │   ├── page.tsx
│   │   │   └── [deviceId]/        # Individual device detail
│   │   │       └── page.tsx
│   │   ├── network-shield/        # Network Shield configuration
│   │   │   ├── page.tsx
│   │   │   └── logs/
│   │   │       └── page.tsx
│   │   ├── ai-copilot/            # AI Co-Pilot dashboard
│   │   │   ├── page.tsx
│   │   │   └── events/
│   │   │       └── page.tsx
│   │   ├── settings/              # User settings and admin settings
│   │   │   ├── page.tsx
│   │   │   └── admin/            # Admin settings
│   │   │       └── page.tsx
│   │   └── billing/             # Billing portal
│   │       └── page.tsx
│   ├── (blog)/                    # Blog routes (public)
│   │   ├── page.tsx               # Blog listing page
│   │   ├── [slug]/                # Blog post detail page (SSG)
│   │   └── layout.tsx             # Blog layout
│   ├── (public)/                  # Public routes (landing, about, etc.)
│   │   ├── page.tsx               # Landing page
│   │   ├── about/                 # About us page
│   │   └── contact/               # Contact page
│   ├── layout.tsx                 # Root layout (global styles, Clerk provider, etc.)
│   ├── globals.css                # Global styles
│   ├── providers.tsx              # Context providers
│   └── api/
│       └── convex/                # Convex API routes (optional)
│           └── route.ts
├── components/                    # Reusable UI components
│   ├── ui/                        # UI primitives (buttons, inputs, etc.)
│   ├── layout/                    # Layout-related components (sidebar, header)
│   ├── dashboard/                 # Dashboard-specific components (charts, alerts)
│   ├── sensors/                   # Sensor management components
│   ├── cameras/                   # Camera management components
│   ├── smart-home/                # Smart home components
│   ├── network-shield/            # Network Shield components
│   ├── ai-copilot/                # AI Co-Pilot components
│   ├── blog/                      # Blog components
│   ├── notifications/             # Notification components
│   └── ...
├── lib/                           # Utility functions
│   ├── utils.ts                   # Helper functions
│   ├── api.ts                     # API interaction functions
│   ├── auth.ts                    # Authentication-related functions
│   ├── convex.ts                  # Convex-related functions
│   └── ...
├── styles/                      # Stylesheets (optional)
│   └── ...
├── types/                       # TypeScript types and interfaces
│   ├── sensor.ts
│   ├── camera.ts
│   ├── smart-home.ts
│   ├── network-shield.ts
│   ├── ai-copilot.ts
│   ├── blog.ts
│   ├── user.ts
│   └── ...
├── next.config.js                # Next.js configuration
├── postcss.config.js             # PostCSS configuration
├── tailwind.config.js            # Tailwind CSS configuration
├── tsconfig.json                 # TypeScript configuration
└── ...
```

#### 1.2. Server Components vs. Client Components

*   **Server Components:** Used for rendering content that doesn't require client-side interactivity or state. Data fetching, data transformation, and initial rendering will primarily occur on the server.  This improves SEO, performance, and initial load times. Examples: Dashboard layout, blog post listing, sensor data display.
*   **Client Components:** Used for interactive elements, state management, and components that need to respond to user interactions. Examples:  Sensor control buttons, camera stream, AI Co-Pilot interactive elements, form submissions.

#### 1.3. State Management

*   **React Hooks:**  `useState`, `useEffect`, `useContext` for component-level state and side effects.
*   **Zustand:** A simple state management solution for global state management, especially for managing data from Convex. It is lightweight and easy to integrate.

#### 1.4. Real-time Data Handling with Convex React Hooks

*   **`useQuery`:**  Fetches data from Convex queries.  Used for real-time updates by re-fetching data on changes.
*   **`useMutation`:** Executes Convex mutations.  Used for updating data in the database.
*   **`useConvexAuth`:** Handles the authentication state from Clerk in the Convex environment.
*   **`useSubscription`:**  Used for real-time updates from Convex subscriptions.  This will be used to update the dashboard in real-time.

#### 1.5. Responsive Design and Component Library

*   **Tailwind CSS:**  A utility-first CSS framework for rapid UI development.  Provides a consistent design system and responsive design capabilities.
*   **UI Primitives:**  Create a set of reusable UI components (buttons, input fields, cards, etc.)  These will be built with Tailwind CSS.

### 2. Backend Architecture (Convex)

#### 2.1. Database Schema Design

```typescript
// convex/schema.ts
import { defineSchema, defineTable, Table } from "convex/server";
import { v } from "convex/values";

export const user = defineTable({
  clerkId: v.string(),
  email: v.string(),
  firstName: v.optional(v.string()),
  lastName: v.optional(v.string()),
  profileImageUrl: v.optional(v.string()),
  subscriptionId: v.optional(v.string()), // Clerk Subscription ID
  subscriptionStatus: v.optional(v.string()), // Clerk Subscription Status
  role: v.optional(v.union(v.literal("admin"), v.literal("user"))),
}).index("by_clerkId", ["clerkId"]);

export const sensor = defineTable({
  userId: v.id("user"),
  name: v.string(),
  type: v.union(
    v.literal("door"),
    v.literal("window"),
    v.literal("flood"),
    v.literal("motion"),
    v.literal("temperature"),
  ),
  location: v.string(),
  deviceId: v.string(), // Unique ID from the sensor device
  isActive: v.boolean(),
  lastReportedValue: v.optional(v.string()), // Last reported sensor value
  lastReportedAt: v.optional(v.number()), // Timestamp of last report
}).index("by_userId", ["userId"]);

export const camera = defineTable({
  userId: v.id("user"),
  name: v.string(),
  location: v.string(),
  rtspUrl: v.string(), // RTSP stream URL
  deviceId: v.string(), // Unique ID from the camera device
  isActive: v.boolean(),
  lastSeen: v.optional(v.number()), // Timestamp of last seen
}).index("by_userId", ["userId"]);

export const smartHomeDevice = defineTable({
  userId: v.id("user"),
  name: v.string(),
  type: v.union(v.literal("light"), v.literal("outlet")),
  location: v.string(),
  deviceId: v.string(), // Unique ID from the device
  isOn: v.boolean(),
  isActive: v.boolean(),
}).index("by_userId", ["userId"]);

export const networkDevice = defineTable({
    userId: v.id("user"),
    macAddress: v.string(),
    ipAddress: v.string(),
    hostname: v.string(),
    isTrusted: v.boolean(), // Manually approved devices
    firstSeen: v.number(),
    lastSeen: v.number(),
    deviceId: v.string(), // Unique ID from the device
}).index("by_userId", ["userId"]);

export const event = defineTable({
  userId: v.id("user"),
  timestamp: v.number(),
  type: v.union(
    v.literal("sensor_triggered"),
    v.literal("camera_detected"),
    v.literal("network_device_connected"),
    v.literal("network_device_blocked"),
    v.literal("smart_home_device_updated"),
    v.literal("ai_anomaly_detected"),
    v.literal("user_login"),
    v.literal("user_logout"),
  ),
  sensorId: v.optional(v.id("sensor")),
  cameraId: v.optional(v.id("camera")),
  smartHomeDeviceId: v.optional(v.id("smartHomeDevice")),
  networkDeviceId: v.optional(v.id("networkDevice")),
  description: v.string(),
  data: v.optional(v.any()), // Store additional data as needed (e.g., image URLs, alert details)
}).index("by_userId", ["userId", "timestamp"]);

export const alert = defineTable({
  userId: v.id("user"),
  timestamp: v.number(),
  summary: v.string(),
  actionPlan: v.optional(v.string()),
  isRead: v.boolean(),
  eventIds: v.array(v.id("event")),
}).index("by_userId", ["userId", "timestamp"]);


export const blogPost = defineTable({
  userId: v.id("user"),
  title: v.string(),
  slug: v.string(),
  content: v.string(),
  publishedAt: v.optional(v.number()),
  imageUrl: v.optional(v.string()),
}).index("by_slug", ["slug"]).index("by_userId", ["userId"]);

export const comment = defineTable({
  blogPostId: v.id("blogPost"),
  userId: v.id("user"),
  content: v.string(),
  createdAt: v.number(),
}).index("by_blogPostId", ["blogPostId"]);

export default defineSchema({
  user,
  sensor,
  camera,
  smartHomeDevice,
  networkDevice,
  event,
  alert,
  blogPost,
  comment,
});
```

#### 2.2. Query and Mutation Function Organization

*   **Queries:**  Use `query` functions to fetch data from the database.  Examples:  `getSensors`, `getCamera`, `getEventsByTimeRange`.
*   **Mutations:** Use `mutation` functions to modify data in the database. Examples: `addSensor`, `updateSensor`, `toggleSmartHomeDevice`, `createEvent`.
*   **Actions:** Use `action` functions for external API integrations or more complex operations.  Examples:  Calling the AI Co-Pilot API, processing camera feed data.

```typescript
// convex/sensor.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";
import { Id } from "./_generated/dataModel";

export const addSensor = mutation({
  args: {
    userId: v.id("user"),
    name: v.string(),
    type: v.union(
      v.literal("door"),
      v.literal("window"),
      v.literal("flood"),
      v.literal("motion"),
      v.literal("temperature"),
    ),
    location: v.string(),
    deviceId: v.string(),
    isActive: v.boolean(),
  },
  handler: async (ctx, args) => {
    const sensorId = await ctx.db.insert("sensor", {
      userId: args.userId,
      name: args.name,
      type: args.type,
      location: args.location,
      deviceId: args.deviceId,
      isActive: args.isActive,
      lastReportedValue: null,
      lastReportedAt: null,
    });
    return sensorId;
  },
});

export const getSensors = query({
  args: { userId: v.id("user") },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("sensor")
      .withIndex("by_userId", (q) => q.eq("userId", args.userId))
      .collect();
  },
});

export const updateSensor = mutation({
  args: {
    sensorId: v.id("sensor"),
    name: v.optional(v.string()),
    location: v.optional(v.string()),
    isActive: v.optional(v.boolean()),
  },
  handler: async (ctx, args) => {
    await ctx.db.patch(args.sensorId, {
      name: args.name,
      location: args.location,
      isActive: args.isActive,
    });
  },
});
```

#### 2.3. Action Functions for External API Integrations

Use actions to interact with external services, such as the AI Co-Pilot API or a facial recognition service.

```typescript
// convex/aiCopilot.ts
import { action } from "./_generated/server";
import { v } from "convex/values";

export const analyzeEvent = action({
  args: { eventId: v.id("event") },
  handler: async (ctx, args) => {
    const event = await ctx.db.get(args.eventId);
    if (!event) {
      console.error("Event not found");
      return;
    }

    // 1. Call the AI Co-Pilot API
    const aiResponse = await fetch("YOUR_AI_COPILOT_API_ENDPOINT", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ event }),
    }).then((res) => res.json());

    // 2. Create an alert in Convex
    await ctx.db.insert("alert", {
      userId: event.userId,
      timestamp: Date.now(),
      summary: aiResponse.summary,
      actionPlan: aiResponse.actionPlan,
      isRead: false,
      eventIds: [args.eventId],
    });
  },
});
```

#### 2.4. Real-time Subscription Patterns

*   **Convex Subscriptions:**  Use Convex subscriptions to send real-time updates to the frontend. Subscribe to changes in the `event` and `alert` tables.
*   **Optimistic UI Updates:**  Apply changes to the UI immediately after a mutation is called, and then update the UI with the actual data from the Convex subscription.

```typescript
// Example: Frontend (Next.js Client Component)
import { useQuery, useMutation, useSubscription } from 'convex/react';
import { api } from '../../convex/_generated/api'; // Import your Convex API
import { useEffect, useState } from 'react';

function Dashboard() {
  const [alerts, setAlerts] = useState([]);
  const [events, setEvents] = useState([]);
  const user = useAuth().user;

  const userId = user?.id; // Assuming Clerk user ID is available

  // Fetch alerts and events
  const fetchedAlerts = useQuery(api.alerts.getAlerts, { userId: userId });
  const fetchedEvents = useQuery(api.events.getEvents, { userId: userId });

  useEffect(() => {
    if (fetchedAlerts) {
      setAlerts(fetchedAlerts);
    }
  }, [fetchedAlerts]);

  useEffect(() => {
    if (fetchedEvents) {
      setEvents(fetchedEvents);
    }
  }, [fetchedEvents]);

  // Real-time update with subscription
  useSubscription(api.alerts.alertsSubscription, { userId: userId }, (newAlerts) => {
    setAlerts(newAlerts);
  });
  useSubscription(api.events.eventsSubscription, { userId: userId }, (newEvents) => {
    setEvents(newEvents);
  });

  if (!user) {
      return <div>Loading...</div>;
