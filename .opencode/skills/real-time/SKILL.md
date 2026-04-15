---
name: real-time
description: Implement real-time features — WebSockets, Server-Sent Events, presence detection, live collaboration, notifications, and optimistic UI patterns
---

## What I do

I implement real-time communication and live updates in web applications:

- **Transport selection** — WebSockets vs SSE vs polling — when to use each
- **WebSocket architecture** — Connection management, reconnection, heartbeat, rooms
- **Server-Sent Events** — Unidirectional real-time from server to client
- **Presence** — Who's online, typing indicators, cursor tracking
- **Live collaboration** — Operational transforms, conflict resolution, CRDTs
- **Notifications** — Real-time notifications, toast delivery, badge counts
- **Optimistic UI** — Show changes instantly, reconcile with server

## When to use me

Use this skill when:
- Adding real-time features (chat, notifications, live updates) to an app
- Choosing between WebSockets, SSE, and polling
- Implementing presence (online status, typing indicators)
- Building collaborative editing (documents, whiteboards, cursors)
- Designing notification systems (in-app, push, email triggers)
- Handling reconnection and offline behavior

## Transport selection

### Decision tree

```
Need bidirectional real-time?
├── Yes (chat, collaboration, multiplayer)
│   └── → WebSocket
├── No (only server → client updates)
│   ├── Updates are infrequent (< 1/min)?
│   │   └── → Polling (simple, no persistent connection)
│   ├── Updates need to reach all clients?
│   │   └── → Server-Sent Events
│   └── Need maximum browser compatibility?
│       └── → SSE (works over HTTP/1.1, no proxy issues)
└── Need both directions?
    └── → WebSocket
```

### Comparison

| Feature | WebSocket | SSE | Polling |
|---------|-----------|-----|---------|
| Direction | Bidirectional | Server → Client only | Client → Server (request) |
| Connection | Persistent TCP | Persistent HTTP | New HTTP per request |
| Latency | ~ms | ~ms | seconds (interval) |
| Browser support | All modern | All modern | All |
| Proxy/CDN | May need upgrade | Works over HTTP/1.1 | Works everywhere |
| Auto-reconnect | Manual | Built-in | N/A |
| Binary data | Yes | No (text only) | Yes |
| Complexity | Medium | Low | Very low |
| Scaling | Needs sticky sessions or pub/sub | Stateful but simpler | Stateless |
| Use for | Chat, collaboration, gaming | Live feeds, notifications, dashboards | Fallback, infrequent updates |

## WebSocket implementation

### Server (Node.js with ws)

```ts
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

const rooms = new Map<string, Set<WebSocket>>();

wss.on('connection', (ws, req) => {
  const userId = authenticate(req);

  ws.on('message', (data) => {
    const message = JSON.parse(data.toString());

    switch (message.type) {
      case 'join':
        joinRoom(ws, message.roomId, userId);
        break;
      case 'leave':
        leaveRoom(ws, message.roomId, userId);
        break;
      case 'chat':
        broadcastToRoom(message.roomId, {
          type: 'chat',
          userId,
          text: message.text,
          timestamp: Date.now(),
        });
        break;
      case 'typing':
        broadcastToRoom(message.roomId, {
          type: 'typing',
          userId,
          isTyping: message.isTyping,
        }, ws); // Exclude sender
        break;
    }
  });

  ws.on('close', () => {
    leaveAllRooms(ws, userId);
  });
});

function joinRoom(ws: WebSocket, roomId: string, userId: string) {
  if (!rooms.has(roomId)) rooms.set(roomId, new Set());
  rooms.get(roomId)!.add(ws);
  ws.roomId = roomId;

  broadcastToRoom(roomId, {
    type: 'presence',
    userId,
    status: 'online',
  });
}

function broadcastToRoom(roomId: string, message: object, exclude?: WebSocket) {
  const room = rooms.get(roomId);
  if (!room) return;

  const data = JSON.stringify(message);
  for (const client of room) {
    if (client !== exclude && client.readyState === WebSocket.OPEN) {
      client.send(data);
    }
  }
}
```

### Client (React hook)

```tsx
import { useEffect, useRef, useCallback } from 'react';

type MessageHandler = (data: any) => void;

function useWebSocket(url: string) {
  const wsRef = useRef<WebSocket | null>(null);
  const handlersRef = useRef<Map<string, Set<MessageHandler>>>(new Map());
  const reconnectTimeoutRef = useRef<number>();

  const connect = useCallback(() => {
    const ws = new WebSocket(url);

    ws.onopen = () => {
      console.log('WebSocket connected');
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
      }
    };

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      const handlers = handlersRef.current.get(message.type);
      if (handlers) {
        handlers.forEach(handler => handler(message));
      }
    };

    ws.onclose = () => {
      console.log('WebSocket disconnected, reconnecting in 3s...');
      reconnectTimeoutRef.current = window.setTimeout(connect, 3000);
    };

    wsRef.current = ws;
  }, [url]);

  useEffect(() => {
    connect();
    return () => {
      if (reconnectTimeoutRef.current) clearTimeout(reconnectTimeoutRef.current);
      wsRef.current?.close();
    };
  }, [connect]);

  const send = useCallback((type: string, data: any) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify({ type, ...data }));
    }
  }, []);

  const subscribe = useCallback((type: string, handler: MessageHandler) => {
    if (!handlersRef.current.has(type)) handlersRef.current.set(type, new Set());
    handlersRef.current.get(type)!.add(handler);
    return () => handlersRef.current.get(type)?.delete(handler);
  }, []);

  return { send, subscribe };
}
```

### Usage

```tsx
function ChatRoom({ roomId, userId }: { roomId: string; userId: string }) {
  const [messages, setMessages] = useState<Message[]>([]);
  const { send, subscribe } = useWebSocket(`wss://api.example.com/ws`);

  useEffect(() => {
    send('join', { roomId });

    const unsubChat = subscribe('chat', (msg) => {
      setMessages(prev => [...prev, msg]);
    });

    const unsubPresence = subscribe('presence', (msg) => {
      // Handle online/offline status
    });

    return () => {
      send('leave', { roomId });
      unsubChat();
      unsubPresence();
    };
  }, [roomId]);

  return (
    <div>
      {messages.map(msg => <ChatMessage key={msg.id} message={msg} />)}
      <ChatInput onSend={(text) => send('chat', { roomId, text })} />
    </div>
  );
}
```

## Server-Sent Events (SSE)

### When SSE beats WebSocket

- Only need server → client (notifications, live feeds, dashboards)
- Want simpler infrastructure (no sticky sessions, no special proxy config)
- Need auto-reconnect (built into SSE)
- Need to work through corporate proxies (SSE uses standard HTTP)

### Server (Node.js)

```ts
import { Router } from 'express';

const router = Router();

router.get('/api/events', (req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
  });

  const userId = req.user.id;

  // Send initial connection event
  res.write(`data: ${JSON.stringify({ type: 'connected', userId })}\n\n`);

  // Subscribe to user-specific events
  const unsubscribe = eventBus.subscribe(`user:${userId}`, (event) => {
    res.write(`data: ${JSON.stringify(event)}\n\n`);
  });

  // Heartbeat to keep connection alive
  const heartbeat = setInterval(() => {
    res.write(`:heartbeat\n\n`);
  }, 30000);

  req.on('close', () => {
    unsubscribe();
    clearInterval(heartbeat);
  });
});
```

### Client

```ts
function useSSE(url: string) {
  const [events, setEvents] = useState<any[]>([]);

  useEffect(() => {
    const source = new EventSource(url);

    source.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setEvents(prev => [...prev.slice(-99), data]); // Keep last 100
    };

    source.onerror = () => {
      // SSE automatically reconnects
      console.log('SSE connection error, will auto-reconnect');
    };

    return () => source.close();
  }, [url]);

  return events;
}
```

## Presence (who's online)

### Architecture

```
1. User opens app → send "online" heartbeat every 30s
2. Server stores presence in Redis: SET online_users with TTL
3. User closes tab → send "offline" via beforeunload + Beacon API
4. Server removes user from presence set
5. Other users subscribe to presence stream → see updates in real-time
```

### Implementation

```ts
// Server-side presence with Redis
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function setOnline(userId: string) {
  await redis.sadd('online_users', userId);
  await redis.expire(`user_presence:${userId}`, 60); // TTL 60s
}

async function setOffline(userId: string) {
  await redis.srem('online_users', userId);
  await redis.del(`user_presence:${userId}`);
}

async function getOnlineUsers(): Promise<string[]> {
  return redis.smembers('online_users');
}

// Heartbeat endpoint (called every 30s)
app.post('/api/presence/heartbeat', async (req, res) => {
  await setOnline(req.user.id);
  res.json({ ok: true });
});

// Client-side heartbeat
function usePresence() {
  useEffect(() => {
    const interval = setInterval(() => {
      fetch('/api/presence/heartbeat', { method: 'POST' });
    }, 30000);

    const handleUnload = () => {
      navigator.sendBeacon('/api/presence/offline');
    };
    window.addEventListener('beforeunload', handleUnload);

    return () => {
      clearInterval(interval);
      window.removeEventListener('beforeunload', handleUnload);
      navigator.sendBeacon('/api/presence/offline');
    };
  }, []);
}
```

### Typing indicators

```ts
// Client: send typing event (debounced)
function useTypingIndicator(roomId: string, ws: WebSocket) {
  const sendTyping = useDebouncedCallback((isTyping: boolean) => {
    ws.send(JSON.stringify({ type: 'typing', roomId, isTyping }));
  }, 500);

  return sendTyping;
}

// Server: broadcast typing with auto-expiry (3s)
// Don't store typing state — it's ephemeral
// Client: track who's typing locally, clear after 3s of no update
function useTypingUsers(roomId: string) {
  const [typingUsers, setTypingUsers] = useState<Map<string, number>>(new Map());

  useEffect(() => {
    const interval = setInterval(() => {
      const now = Date.now();
      setTypingUsers(prev => {
        const next = new Map();
        for (const [userId, timestamp] of prev) {
          if (now - timestamp < 3000) next.set(userId, timestamp);
        }
        return next;
      });
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  const onTyping = (userId: string) => {
    setTypingUsers(prev => new Map(prev).set(userId, Date.now()));
  };

  return { typingUsers, onTyping };
}
```

## Live collaboration (CRDTs)

### Approach: Yjs for document collaboration

```ts
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

function useCollaborativeDocument(docId: string, userId: string) {
  const ydoc = useRef(new Y.Doc());

  useEffect(() => {
    const provider = new WebsocketProvider(
      'wss://collab.example.com',
      docId,
      ydoc.current,
      { params: { userId } }
    );

    return () => {
      provider.disconnect();
      ydoc.current.destroy();
    };
  }, [docId]);

  // Shared text (for editors)
  const ytext = ydoc.current.getText('content');

  // Shared map (for structured data)
  const ymap = ydoc.current.getMap('metadata');

  return { ydoc: ydoc.current, ytext, ymap };
}
```

### Conflict resolution principles

```
CRDT (Conflict-free Replicated Data Types):
  - Every operation has a unique ID and timestamp
  - Operations commute (order doesn't matter)
  - Operations are idempotent (applying twice = applying once)
  - No central authority needed for conflict resolution
  - Yjs handles this automatically

For custom collaboration:
  - Last-Write-Wins: simplest, good for metadata
  - Operational Transform: for text editing (complex, use a library)
  - CRDT: for structured data (use Yjs/Automerge)
```

## Notifications

### Architecture

```
Event triggers:
  - User action (mention, assignment, reply)
  - System event (task overdue, payment processed)
  - Scheduled event (daily digest, weekly report)

Processing:
  1. Event occurs → Create notification record in database
  2. Fan out to delivery channels:
     a. In-app (real-time via WebSocket/SSE)
     b. Push (browser notification via service worker)
     c. Email (transactional email for important events)
     d. SMS (critical alerts only)

Each user has notification preferences:
  - Channel preferences (in-app, email, push) per notification type
  - Digest preferences (immediate, daily, weekly)
  - Quiet hours (no push/email between 10pm-8am)
```

### Notification model

```ts
model Notification {
  id          String   @id @default(cuid())
  userId      String   // Recipient
  type        String   // 'mention', 'assignment', 'reply', 'system'
  title       String
  message     String
  link        String?  // Deep link to the relevant resource
  read        Boolean  @default(false)
  createdAt   DateTime @default(now())

  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId, read])
  @@index([userId, createdAt])
}
```

### Notification delivery

```ts
async function sendNotification(userId: string, notification: NotificationInput) {
  // 1. Persist to database
  const record = await db.notification.create({
    data: {
      userId,
      type: notification.type,
      title: notification.title,
      message: notification.message,
      link: notification.link,
    },
  });

  // 2. Push via real-time channel (WebSocket or SSE)
  websocket.broadcastToUser(userId, {
    type: 'notification',
    data: record,
  });

  // 3. Check user preferences for email/push
  const preferences = await getNotificationPreferences(userId);

  if (preferences.email.includes(notification.type)) {
    await sendEmail(userId, notification);
  }

  if (preferences.push.includes(notification.type)) {
    await sendPushNotification(userId, notification);
  }

  return record;
}
```

### Notification badge count

```ts
// API: GET /api/notifications/unread-count
app.get('/api/notifications/unread-count', async (req, res) => {
  const count = await db.notification.count({
    where: { userId: req.user.id, read: false },
  });
  res.json({ count });
});

// Client: subscribe to real-time count
function useUnreadCount() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    // Initial fetch
    fetch('/api/notifications/unread-count')
      .then(r => r.json())
      .then(data => setCount(data.count));

    // Real-time updates
    const unsub = subscribe('notification', () => {
      setCount(prev => prev + 1);
    });
    const unsubRead = subscribe('notification:read', () => {
      setCount(prev => Math.max(0, prev - 1));
    });

    return () => { unsub(); unsubRead(); };
  }, []);

  return count;
}
```

### Notification list UI

```tsx
function NotificationDropdown() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const unreadCount = useUnreadCount();

  const markAsRead = async (id: string) => {
    await fetch(`/api/notifications/${id}/read`, { method: 'PATCH' });
    setNotifications(prev => prev.map(n => n.id === id ? { ...n, read: true } : n));
  };

  const markAllAsRead = async () => {
    await fetch('/api/notifications/read-all', { method: 'PATCH' });
    setNotifications(prev => prev.map(n => ({ ...n, read: true })));
  };

  return (
    <DropdownMenu>
      <DropdownMenuTrigger>
        <BellIcon />
        {unreadCount > 0 && <Badge>{unreadCount > 99 ? '99+' : unreadCount}</Badge>}
      </DropdownMenuTrigger>
      <DropdownMenuContent>
        <div className="flex justify-between p-2">
          <span className="font-semibold">Notifications</span>
          {unreadCount > 0 && (
            <button onClick={markAllAsRead}>Mark all read</button>
          )}
        </div>
        <DropdownMenuSeparator />
        {notifications.slice(0, 10).map(n => (
          <NotificationItem key={n.id} notification={n} onRead={markAsRead} />
        ))}
      </DropdownMenuContent>
    </DropdownMenu>
  );
}
```

## Reconnection and offline

### Reconnection strategy

```ts
function useReconnect(socket: WebSocket) {
  const reconnectAttempts = useRef(0);
  const maxAttempts = 5;
  const baseDelay = 1000;

  const connect = useCallback(() => {
    socket.onclose = () => {
      if (reconnectAttempts.current < maxAttempts) {
        const delay = baseDelay * Math.pow(2, reconnectAttempts.current);
        reconnectAttempts.current += 1;
        setTimeout(connect, delay); // Exponential backoff: 1s, 2s, 4s, 8s, 16s
      }
    };

    socket.onopen = () => {
      reconnectAttempts.current = 0; // Reset on successful connection
    };
  }, []);
}
```

### Offline queue

```ts
// Queue messages while offline, send when reconnected
class OfflineQueue {
  private queue: Array<{ type: string; data: any }> = [];

  add(message: { type: string; data: any }) {
    this.queue.push(message);
  }

  flush(send: (message: any) => void) {
    while (this.queue.length > 0) {
      send(this.queue.shift()!);
    }
  }
}

// Usage in WebSocket hook
const queue = useRef(new OfflineQueue());

ws.onopen = () => {
  queue.current.flush((msg) => ws.send(JSON.stringify(msg)));
};

const send = (type: string, data: any) => {
  if (ws.readyState === WebSocket.OPEN) {
    ws.send(JSON.stringify({ type, ...data }));
  } else {
    queue.current.add({ type, data }); // Queue for later
  }
};
```

## Quality checklist

- [ ] Transport chosen appropriately (WebSocket for bidirectional, SSE for server-push)
- [ ] Reconnection with exponential backoff (1s, 2s, 4s, 8s, max 16s)
- [ ] Heartbeat/keepalive to detect dead connections (30s interval)
- [ ] Offline message queue that flushes on reconnection
- [ ] Presence indicators with 30s heartbeat and 60s TTL
- [ ] Typing indicators with 3s auto-expiry
- [ ] Notifications persisted in database before real-time delivery
- [ ] Unread badge count synced between real-time and REST
- [ ] Mark-as-read and mark-all-as-read functionality
- [ ] WebSocket rooms/channels for targeted broadcasting
- [ ] Authentication on WebSocket upgrade (verify token on connect)
- [ ] Rate limiting on WebSocket messages (prevent flooding)
- [ ] Connection status indicator (connected/reconnecting/offline)

## Anti-patterns I avoid

- Using WebSocket for server-only updates (use SSE — simpler)
- Polling when real-time is needed (use SSE or WebSocket)
- Storing typing indicators in the database (ephemeral — track locally + broadcast)
- Sending raw database records over WebSocket (transform to API response format)
- Not handling disconnection (users will disconnect — plan for it)
- Unbounded WebSocket messages without rate limiting (vulnerability)
- Missing authentication on WebSocket connections (anyone could connect)
- Broadcasting all messages to all users (use rooms/channels)
- Optimistic updates without rollback on failure (users see ghost data)
- Not cleaning up WebSocket listeners on component unmount (memory leak)