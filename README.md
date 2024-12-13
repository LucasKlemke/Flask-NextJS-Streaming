# Streaming Data with Flask and Next.js

This document explains how the Flask backend streams data to a Next.js frontend using **Server-Sent Events (SSE)**. The example demonstrates:

1. How the Flask backend sends real-time data.
2. How the Next.js frontend receives and displays the streamed data.

---

## Backend Implementation (Flask)

### Code Explanation
```python
@app.flask_app.route("/api/v1/stream", methods=["GET"])
def stream():
    def generate_data():
        for i in range(10):
            yield f"data: {i}\n\n"  # SSE format: "data: <message>\n\n"
            time.sleep(1)  # Simulate streaming with 1-second intervals

    return Response(generate_data(), mimetype="text/event-stream")
```

### Key Components
1. **Route Definition:**
   The `/api/v1/stream` endpoint handles **GET** requests to provide the SSE data stream.

2. **`generate_data` Function:**
   - Uses Python's `yield` to send chunks of data iteratively.
   - Messages are sent in the SSE format: `data: <message>\n\n`.
   - A 1-second delay (`time.sleep(1)`) simulates real-time data generation.

3. **Response:**
   - The `Response` object is configured with `mimetype="text/event-stream"` to indicate SSE.

### SSE Format
Each message follows the structure:
```
data: <message>\n\n
```
Example of multiple messages:
```
data: 0

data: 1

data: 2

```

---

## Frontend Implementation (Next.js)

### Code Explanation
```javascript
'use client';
import { Button } from '@/components/ui/button';
import React, { useEffect, useState } from 'react';

const Page = () => {
  const [messages, setMessages] = useState([]);
  const [eventSource, setEventSource] = useState(null);

  const handleClick = () => {
    // Avoid multiple connections
    if (eventSource) {
      eventSource.close();
      setEventSource(null);
    }

    const newEventSource = new EventSource('http://localhost:8084/api/v1/stream');

    newEventSource.onmessage = (event) => {
      setMessages((prevMessages) => [...prevMessages, event.data]);
    };

    newEventSource.onerror = () => {
      newEventSource.close(); // Close connection on error
      setEventSource(null);
    };

    setEventSource(newEventSource);
  };

  useEffect(() => {
    // Cleanup on component unmount
    return () => {
      if (eventSource) {
        eventSource.close();
      }
    };
  }, [eventSource]);

  return (
    <div>
      <Button onClick={handleClick}>Generate Text</Button>
      <ul>
        {messages.map((msg, idx) => (
          <li key={idx}>{msg}</li>
        ))}
      </ul>
    </div>
  );
};

export default Page;
```

### Key Components
1. **State Management:**
   - `messages`: Stores the streamed data.
   - `eventSource`: Tracks the active SSE connection to prevent duplicates.

2. **`handleClick` Function:**
   - Creates a new `EventSource` connected to the `/api/v1/stream` endpoint.
   - Updates the `messages` state with incoming data.
   - Handles errors by closing the connection.

3. **`useEffect` Hook:**
   - Cleans up the `EventSource` connection when the component unmounts or the connection state changes.

4. **Rendering Messages:**
   - The streamed messages are displayed in a list (`<ul>`).

### SSE in JavaScript
- **`EventSource` API:**
  - Opens a persistent connection to the server.
  - Automatically reconnects if the connection drops.

---

## How It Works

1. **Backend Process:**
   - The Flask server starts streaming messages when the `/api/v1/stream` endpoint is accessed.
   - Messages are sent periodically in SSE format (`data: <message>\n\n`).

2. **Frontend Process:**
   - The Next.js page establishes an SSE connection using `EventSource`.
   - Each message sent by the server triggers the `onmessage` handler, updating the `messages` state.
   - The messages are rendered dynamically in the UI.

---
