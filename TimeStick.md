<h3>TimeStick protocol specification (Draft 0.1)</h3><p></p><h2>1. Overview</h2><p></p><p><strong>Protocol name:</strong> TimeStick<br><strong>Tagline:</strong> “Stuck in time!”<br><strong>URI scheme:</strong> <code>timestick://</code> <br><strong>Status:</strong> Experimental, draft</p><p></p><p>TimeStick is a <strong>session-based, real-time, bidirectional messaging protocol</strong> that runs over HTTP or HTTPS. It is designed to let a <strong>caller</strong> and a <strong>receiver</strong> enter a persistent “TimeStick session,” where they can exchange messages without losing state between requests. Unlike HTTP, <strong>responses are optional</strong> for each request.</p><p></p><p>TimeStick is intended to:</p><p></p><ul><li><p><strong>Maintain a stable session</strong> between caller and receiver.</p><p></p></li><li><p><strong>Support real-time messaging</strong> in both directions.</p><p></p></li><li><p><strong>Allow optional replies</strong> to messages.</p><p></p></li><li><p><strong>Provide clear start and stop semantics</strong>, including emergency shutdown.</p><p></p></li></ul><p></p><h2>2. Terminology</h2><p></p><ul><li><p><strong>TimeStick session (session):</strong> <br>A logical, persistent connection between a caller and a receiver, identified by a session ID.</p><p></p></li><li><p><strong>Caller:</strong> <br>The entity that initiates the TimeStick session using a START request.</p><p></p></li><li><p><strong>Receiver:</strong> <br>The entity that accepts the TimeStick session and communicates with the caller.</p><p></p></li><li><p><strong>Control message:</strong> <br>A special TimeStick message related to session control (e.g. START, STOP, KILL).</p><p></p></li><li><p><strong>Application message:</strong> <br>A normal data/message used by the application (e.g. “send chat text”, “update state”).</p><p></p></li><li><p><strong>Transport:</strong> <br>The underlying protocol: HTTP or HTTPS (HTTP over TLS).</p><p></p></li></ul><p></p><h2>3. URI scheme</h2><p></p><p><strong>Scheme:</strong> <code>timestick://</code></p><p></p><p>TimeStick URIs identify <strong>targets</strong> for TimeStick sessions.</p><p></p><p><strong>Syntax (informal):</strong></p><p></p><ul><li><p><code>timestick://host</code></p><p></p></li><li><p><code>timestick://host:port</code></p><p></p></li><li><p><code>timestick://host/path</code></p><p></p></li><li><p><code>timestick://host:port/path</code></p><p></p></li></ul><p></p><p><strong>Examples:</strong></p><p></p><ul><li><p><code>timestick://example.com</code></p><p></p></li><li><p><code>timestick://api.example.com/chat</code></p><p></p></li><li><p><code>timestick://example.net:8443/room/123</code></p><p></p></li></ul><p></p><p>The mapping from <code>timestick://</code> to actual HTTP(S) endpoints is implementation-specific, but a common rule could be:</p><p></p><ul><li><p><strong>timestick://example.com/path</strong> → <strong>https://example.com/timestick/path</strong> <br>(if using HTTPS)</p><p></p></li><li><p><strong>timestick://example.com/path</strong> → <strong>http://example.com/timestick/path</strong> <br>(if using HTTP)</p><p></p></li></ul><p></p><h2>4. Connection and transport model</h2><p></p><h3>4.1 Transports</h3><p></p><p>TimeStick uses <strong>HTTP or HTTPS</strong> as its transport:</p><p></p><ul><li><p><strong>HTTP:</strong> <br>Clear-text, no built-in encryption.</p><p></p></li><li><p><strong>HTTPS (HTTP over TLS):</strong> <br>Encrypted and authenticated by TLS.</p><p></p></li></ul><p></p><p>The <strong>receiver chooses</strong> whether to use HTTP or HTTPS, but HTTPS is <strong>strongly recommended</strong> for real-world use.</p><p></p><h3>4.2 Session concept</h3><p></p><p>A TimeStick session:</p><p></p><ul><li><p>Is identified by a <strong>Session ID</strong> (e.g. a UUID).</p><p></p></li><li><p>Starts after a successful START handshake.</p><p></p></li><li><p>Exists until:</p><p></p><ul><li><p>Caller stops the session, or</p><p></p></li><li><p>Receiver stops the session, or</p><p></p></li><li><p>A fatal error occurs.</p><p></p></li></ul><p></p></li></ul><p></p><p>Messages in a session are <strong>logically associated</strong> with that session ID.</p><p></p><h2>5. Handshake: starting a TimeStick session</h2><p></p><h3>5.1 START request</h3><p></p><p>To begin a session, the caller sends a <strong>START request</strong> to the receiver’s TimeStick API URL.</p><p></p><p><strong>Example (HTTP POST):</strong></p><p></p><ul><li><p><strong>Method:</strong> <code>POST</code></p><p></p></li><li><p><strong>Path:</strong> <code>/timestick/start</code> (example; implementation-defined)</p><p></p></li><li><p><strong>Headers:</strong></p><p></p><ul><li><p><code>Content-Type: application/json</code></p><p></p></li></ul><p></p></li><li><p><strong>Body (example JSON):</strong></p><p></p></li></ul><p></p><p>json</p><pre><code>{
  "action": "START",
  "protocol": "TimeStick",
  "version": "0.1",
  "caller_id": "client-123",
  "requested_uri": "timestick://api.example.com/chat",
  "capabilities": {
    "max_message_size": 65535,
    "supports_binary": false,
    "supports_notifications": true
  }
}
</code></pre><h3>5.2 START response</h3><p></p><p>If the receiver accepts the session, it responds with:</p><p></p><ul><li><p><strong>Status:</strong> 200 (OK)</p><p></p></li><li><p><strong>Body (example JSON):</strong></p><p></p></li></ul><p></p><p>json</p><pre><code>{
  "action": "START-OK",
  "protocol": "TimeStick",
  "version": "0.1",
  "session_id": "abcd-ef01-2345-6789",
  "transport": "https",
  "endpoint": "/timestick/session/abcd-ef01-2345-6789",
  "capabilities": {
    "max_message_size": 65535,
    "supports_binary": false,
    "supports_notifications": true,
    "server_can_stop_anytime": true
  }
}
</code></pre><p>If the receiver rejects the session:</p><p></p><ul><li><p><strong>Status:</strong> 4xx or 5xx (e.g. 400, 403, 503)</p><p></p></li><li><p><strong>Body (example JSON):</strong></p><p></p></li></ul><p></p><p>json</p><pre><code>{
  "action": "START-ERROR",
  "reason": "Unsupported version"
}
</code></pre><h3>5.3 Session endpoint</h3><p></p><p>The response defines an <strong>endpoint</strong> for the session, such as:</p><p></p><ul><li><p><code>/timestick/session/abcd-ef01-2345-6789</code></p><p></p></li></ul><p></p><p>Subsequent TimeStick messages for this session are sent to this endpoint.</p><p></p><h2>6. Messaging model</h2><p></p><h3>6.1 Message types</h3><p></p><p>TimeStick defines three high-level categories of messages:</p><p></p><ul><li><p><strong>Control:</strong></p><p></p><ul><li><p><code>START</code>, <code>START-OK</code>, <code>START-ERROR</code></p><p></p></li><li><p><code>STOP</code></p><p></p></li><li><p><code>KILL</code></p><p></p></li></ul><p></p></li><li><p><strong>Request:</strong> <br>A message that may or may not expect a response.</p><p></p></li><li><p><strong>Response:</strong> <br>A message sent in reply to a specific request.</p><p></p></li></ul><p></p><p>All messages are sent as HTTP requests (often POST) with a JSON body by default.</p><p></p><h3>6.2 Basic message format (JSON)</h3><p></p><p><strong>Common fields:</strong></p><p></p><p>json</p><pre><code>{
  "type": "request|response|control",
  "session_id": "abcd-ef01-2345-6789",
  "message_id": "unique-message-id",
  "correlation_id": "id-of-message-being-answered-or-null",
  "payload": { }
}
</code></pre><ul><li><p><strong>type:</strong> <br><code>control</code>, <code>request</code>, or <code>response</code>.</p><p></p></li><li><p><strong>session_id:</strong> <br>Associates the message with a TimeStick session.</p><p></p></li><li><p><strong>message_id:</strong> <br>Unique ID for this message.</p><p></p></li><li><p><strong>correlation_id:</strong> <br>For responses, this references the original request’s <code>message_id</code>. For messages not answering anything, it can be <code>null</code> or omitted.</p><p></p></li><li><p><strong>payload:</strong> <br>Application-specific or control-specific content.</p><p></p></li></ul><p></p><h3>6.3 Sending messages</h3><p></p><p>After START-OK, the caller and receiver can both send messages to the <strong>session endpoint</strong>.</p><p></p><p><strong>Example request (no guaranteed response):</strong></p><p></p><p>json</p><pre><code>{
  "type": "request",
  "session_id": "abcd-ef01-2345-6789",
  "message_id": "msg-001",
  "correlation_id": null,
  "payload": {
    "kind": "chat.message",
    "text": "Hello from caller!"
  }
}
</code></pre><p>The receiver can choose to:</p><p></p><ul><li><p><strong>Respond</strong> (with a message that has <code>type: "response"</code> and <code>correlation_id: "msg-001"</code>), or</p><p></p></li><li><p><strong>Not respond at all</strong>.</p><p></p></li></ul><p></p><p>This satisfies your rule:<br><strong>Each request does not mandate a response.</strong></p><p></p><h3>6.4 Real-time behavior</h3><p></p><p>To achieve “real-time” behavior, implementations may:</p><p></p><ul><li><p>Use <strong>long-polling</strong> (client holds an HTTP request open to receive messages).</p><p></p></li><li><p>Use <strong>HTTP/2</strong> streams or <strong>HTTP/3/QUIC</strong>.</p><p></p></li><li><p>Use <strong>WebSocket internally</strong>, while still respecting TimeStick’s message structure.</p><p></p></li></ul><p></p><p>The spec only requires:</p><p></p><ul><li><p><strong>Bidirectional sending of messages</strong></p><p></p></li><li><p><strong>Preservation of session state</strong> across multiple HTTP requests</p><p></p></li></ul><p></p><p>The exact mechanism is implementation-defined.</p><p></p><h2>7. Stopping a TimeStick session</h2><p></p><h3>7.1 Caller-initiated stop</h3><p></p><p>The caller can stop the session in two ways:</p><p></p><h4>7.1.1 Normal stop</h4><p></p><p>Caller sends a <strong>STOP</strong> control message over the session:</p><p></p><p>json</p><pre><code>{
  "type": "control",
  "session_id": "abcd-ef01-2345-6789",
  "message_id": "msg-100",
  "correlation_id": null,
  "payload": {
    "action": "STOP",
    "reason": "normal-termination"
  }
}
</code></pre><p>The receiver should:</p><p></p><ul><li><p>Clean up session resources.</p><p></p></li><li><p>Optionally acknowledge with a response or a final control message.</p><p></p></li><li><p>Consider the session closed.</p><p></p></li></ul><p></p><h4>7.1.2 Last-resort stop with kill level</h4><p></p><p>Caller can send a <strong>KILL</strong> control message:</p><p></p><p>json</p><pre><code>{
  "type": "control",
  "session_id": "abcd-ef01-2345-6789",
  "message_id": "msg-101",
  "correlation_id": null,
  "payload": {
    "action": "KILL",
    "kill_level": 3,
    "reason": "emergency-shutdown"
  }
}
</code></pre><ul><li><p><strong>kill_level:</strong> <br>A non-negative integer. Higher values may indicate more urgent or aggressive termination. The exact meaning is implementation-defined but <strong>must be documented</strong> by the receiver implementation.</p><p></p></li></ul><p></p><p>The receiver should:</p><p></p><ul><li><p>Treat <code>KILL</code> as a higher-priority termination signal than <code>STOP</code>.</p><p></p></li><li><p>Immediately terminate or downgrade the session based on the kill level.</p><p></p></li></ul><p></p><h3>7.2 Receiver-initiated stop</h3><p></p><p>The receiver may stop the session at any time, for any reason.</p><p></p><p>The receiver should send a control message to the caller:</p><p></p><p>json</p><pre><code>{
  "type": "control",
  "session_id": "abcd-ef01-2345-6789",
  "message_id": "msg-200",
  "correlation_id": null,
  "payload": {
    "action": "STOP",
    "reason": "server-initiated",
    "details": "maintenance"
  }
}
</code></pre><p>After sending this, the receiver:</p><p></p><ul><li><p>Cleans up resources.</p><p></p></li><li><p>Rejects further messages for this session ID.</p><p></p></li></ul><p></p><p>Callers that receive such a message must:</p><p></p><ul><li><p>Treat the session as closed.</p><p></p></li><li><p>Avoid sending more messages for that session.</p><p></p></li></ul><p></p><h2>8. Error handling</h2><p></p><h3>8.1 Message-level errors</h3><p></p><p>If a message is malformed or invalid, the receiver may:</p><p></p><ul><li><p>Respond with a <strong>response</strong> message indicating an error, or</p><p></p></li><li><p>Ignore the message.</p><p></p></li></ul><p></p><p><strong>Example error response:</strong></p><p></p><p>json</p><pre><code>{
  "type": "response",
  "session_id": "abcd-ef01-2345-6789",
  "message_id": "msg-300",
  "correlation_id": "msg-001",
  "payload": {
    "status": "error",
    "error_code": "INVALID_MESSAGE",
    "error_message": "Missing required field 'payload.kind'"
  }
}
</code></pre><h3>8.2 Session-level errors</h3><p></p><p>If a severe problem happens, the receiver may:</p><p></p><ul><li><p>Send a control message with <code>action: "STOP"</code> or <code>action: "KILL"</code>.</p><p></p></li><li><p>Immediately close any underlying connections.</p><p></p></li></ul><p></p><h2>9. Security considerations</h2><p></p><ul><li><p><strong>Transport security:</strong> <br>Implementations should prefer <strong>HTTPS</strong> for TimeStick sessions, so that messages are protected by TLS.</p><p></p></li><li><p><strong>Authentication:</strong> <br>TimeStick itself does not define how callers or receivers authenticate. Implementations should use existing methods such as:</p><p></p><ul><li><p>OAuth tokens</p><p></p></li><li><p>API keys</p><p></p></li><li><p>TLS client certificates</p><p></p></li></ul><p></p></li><li><p><strong>Authorization:</strong> <br>Receivers must ensure that only authorized callers can:</p><p></p><ul><li><p>Start sessions</p><p></p></li><li><p>Send messages</p><p></p></li><li><p>Issue <code>STOP</code> or <code>KILL</code> actions</p><p></p></li></ul><p></p></li><li><p><strong>Replay protection:</strong> <br>TimeStick messages should use:</p><p></p><ul><li><p>Unique <code>message_id</code> values</p><p></p></li><li><p>Session-bound semantics<br>so that old messages are not blindly re-accepted.</p><p></p></li></ul><p></p></li><li><p><strong>Resource limits:</strong> <br>Receivers should enforce limits on:</p><p></p><ul><li><p>Number of active sessions per caller</p><p></p></li><li><p>Maximum message size</p><p></p></li><li><p>Maximum rate of messages<br>to prevent abuse.</p><p></p></li></ul><p></p></li></ul><p></p><h2>10. Versioning</h2><p></p><ul><li><p><strong>Field:</strong> <code>version</code></p><p></p></li><li><p><strong>Format:</strong> String (e.g. <code>"0.1"</code>, <code>"1.0"</code>)</p><p></p></li></ul><p></p><p>Both caller and receiver:</p><p></p><ul><li><p><strong>Must include</strong> protocol version in the START handshake.</p><p></p></li><li><p><strong>May reject</strong> a session if the version is unsupported.</p><p></p></li></ul><p></p><p>Future versions of TimeStick should:</p><p></p><ul><li><p>Try to maintain backward compatibility, or</p><p></p></li><li><p>Use new version numbers and capabilities fields.</p><p></p></li></ul><p></p><h2>11. Extensibility</h2><p></p><ul><li><p><strong>Unknown fields:</strong> <br>Fields not recognized by an implementation <strong>must be ignored</strong>, unless the spec later says otherwise. This allows adding new fields safely.</p><p></p></li><li><p><strong>Custom payloads:</strong> <br>The <code>payload</code> object is application-defined and can contain any structure, as long as both sides understand it.</p><p></p></li></ul><p></p><h2>12. Minimal example flow</h2><p></p><ol><li><p><strong>Caller:</strong> sends <code>START</code> to <code>/timestick/start</code>.</p><p></p></li><li><p><strong>Receiver:</strong> replies <code>START-OK</code> with <code>session_id</code>.</p><p></p></li><li><p><strong>Caller:</strong> sends TimeStick messages to <code>/timestick/session/&lt;session_id&gt;</code>.</p><p></p></li><li><p><strong>Receiver:</strong> sends messages back over the same session.</p><p></p></li><li><p><strong>Caller:</strong> eventually sends <code>STOP</code> (or <code>KILL</code> if urgent).</p><p></p></li><li><p><strong>Receiver:</strong> closes session and stops accepting messages.</p><p></p></li></ol><hr><hr><hr><hr><hr><p></p>