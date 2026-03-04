# StrictMode Safety for External Resources — Design Lesson

**Date**: 2026-03-02
**Source**: Typing indicator feature — ActionCable subscription race condition
**Project**: Harmoniq Frontend (harmoniq-frontend)
**Commits**: f978612 (fix), 9d3a268 (scroll follow-up)

---

## The Bug

React StrictMode double-mounts components: mount → cleanup → re-mount. When a component subscribes to an ActionCable channel in useEffect and unsubscribes in cleanup, this produces:

```
subscribe(channel) → unsubscribe(channel) → subscribe(channel)
```

ActionCable processes these asynchronously. The server sometimes processed the unsubscribe *after* the second subscribe, removing the valid subscription. All real-time events (messages, typing indicators, reactions, processing status) silently stopped arriving.

The bug was **flaky** — it depended on server-side processing order. Different tests failed on different runs. We initially misdiagnosed it as a directional problem (one user's subscription failing) when it was actually a race condition affecting any user unpredictably.

## The Fix

Deferred unsubscribe pattern — a well-documented approach for external resources in StrictMode:

```typescript
// unsubscribe() no longer executes immediately
unsubscribe(): void {
  this.pendingUnsubscribeConversationId = this.currentConversationId;
  this.pendingUnsubscribe = setTimeout(() => {
    this.executeUnsubscribe(); // actually unsubscribe after 100ms
  }, 100);
}

// subscribe() cancels the pending unsubscribe if it's for the same channel
subscribe(conversationId): void {
  if (this.pendingUnsubscribe && this.pendingUnsubscribeConversationId === conversationId) {
    clearTimeout(this.pendingUnsubscribe); // StrictMode re-mount — cancel cleanup
    return; // subscription is still alive on the wire
  }
  // ... normal subscribe logic
}
```

## Lessons

### 1. Every external resource subscription needs StrictMode protection

This isn't specific to ActionCable. Any imperative resource managed through useEffect (WebSocket connections, event listeners on global objects, timer-based polling, SSE connections) needs the same treatment. The pattern:

- **Cleanup defers** the actual teardown (setTimeout ~100ms)
- **Re-mount cancels** the deferred teardown if same resource
- **Real unmount** lets the timer fire and tear down normally
- **Different resource** on re-mount executes deferred teardown immediately, then subscribes to new resource

### 2. StrictMode bugs are production bugs hiding in plain sight

StrictMode double-mount exposes bugs that also occur in production — just less frequently. In production, rapid navigation (click channel A, immediately click channel B) produces the same subscribe → unsubscribe → subscribe race. StrictMode makes it happen on *every* mount, turning a rare production bug into a consistent dev bug. We should treat StrictMode as a gift, not an annoyance.

### 3. The bug was latent — typing indicator just exposed it

ConversationChannelService has been in production since the channel system was built. The race condition affected ALL event types (new_message, reactions, processing_status), not just typing indicators. We never noticed because:

- Message delivery has an HTTP fallback (initial load fetches messages via REST)
- Reactions are low-frequency (a missed update isn't noticed)
- Processing status is transient (expires naturally)

Typing indicators have no fallback and are time-sensitive — they made the bug visible. **Lesson**: Just because a service "works" doesn't mean its subscription lifecycle is correct. Silent failures accumulate.

### 4. Scroll impact is a checklist item for dynamic content

We shipped the typing indicator, then had to immediately follow up with a scroll fix. Any content that appears dynamically at the bottom of a scrollable container needs scroll consideration. The existing scroll mechanisms (ResizeObserver on the container, new-message effect on `messages.length`) don't catch non-message content because:

- ResizeObserver fires on element dimension changes — a fixed-size flex container doesn't change dimensions when its scrollHeight increases
- The message-count effect only fires on `messages.length` changes

**Checklist item**: "If I'm adding content to a scrollable list that isn't a new item in the tracked array, does the scroll system know about it?"

---

## Proposed Defensive Pattern

For any future external resource integration in React:

```
1. Does this resource have a subscribe/connect lifecycle?
   → Add deferred cleanup (100ms timer in cleanup, cancel in mount)

2. Does this resource produce events that update UI?
   → Verify events still arrive after StrictMode double-mount
   → Test with rapid navigation (switch away and back quickly)

3. Does this resource's UI appear in a scrollable container?
   → Verify scroll position accounts for the new content
   → Don't assume existing ResizeObserver/scroll effects catch it
```

## Relationship to Existing Knowledge

This lesson extends the "Singleton-React Seam" bug family documented in `architecture-knowledge-base.md`. The specific sub-pattern is: **imperative subscription lifecycle + React declarative cleanup = race condition**. The deferred unsubscribe pattern is the canonical fix.
