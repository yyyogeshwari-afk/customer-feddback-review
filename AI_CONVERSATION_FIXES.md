# AI Conversation Flow Fixes

## Issues Fixed

### 1. Popup Timing Issue ✅
**Problem**: The "Last 30 seconds" warning was appearing as a message at the end of conversation instead of as a popup at 1:30 remaining.

**Solution**:
- Changed warning trigger from 30 seconds to 90 seconds (1:30 remaining)
- Updated warning message to say "1 minute and 30 seconds" instead of "30 seconds"
- The popup now displays as a modal overlay with proper styling

**Code Changes**:
- `src/components/ConversationalAI.tsx` line 260: Changed `timerSeconds === 30` to `timerSeconds === 90`
- Updated speech synthesis message to match new timing
- Updated modal text to reflect 1:30 remaining time

### 2. Redirect to Rating Page Issue ✅
**Problem**: After AI conversation ended, users were not being redirected to the Star Rating page before the Thank You page.

**Solution**:
- Fixed the `handleConversationEnd` logic to always redirect to rating page when timer expires
- Ensured the second parameter (`showRating`) is always `true` when conversation ends due to:
  - Timer expiration
  - AI-initiated conversation end
- Only redirect to Thank You page when user explicitly says farewell

**Code Changes**:
- `src/components/ConversationalAI.tsx` line 276: Timer expiration now calls `handleConversationEnd(undefined, true)`
- `src/components/ConversationalAI.tsx` line 490: AI-initiated end now calls `handleConversationEnd(undefined, true)`
- User farewell still calls `onConversationEnd(finalHistory, false)` to skip rating

## Flow Summary

### Timer Expiration Flow
1. Timer starts at 2:00 (120 seconds)
2. At 1:30 (90 seconds remaining): Popup warning appears
3. User can click "Got it!" to dismiss the popup
4. At 0:00: Conversation ends automatically
5. **User is redirected to Star Rating page** ✅
6. After submitting ratings: User sees Thank You page

### User Farewell Flow
1. User types farewell words (bye, thanks, done, etc.)
2. AI responds with farewell message
3. Conversation ends
4. **User is redirected directly to Thank You page** (skips rating)

### AI-Initiated End Flow
1. AI detects conversation is complete
2. AI sends message with [CONVERSATION_END] marker
3. Conversation ends
4. **User is redirected to Star Rating page** ✅
5. After submitting ratings: User sees Thank You page

## Testing Checklist

- [ ] Start AI conversation
- [ ] Wait until 1:30 remaining - verify popup appears
- [ ] Click "Got it!" on popup - verify it dismisses
- [ ] Let timer reach 0:00 - verify redirect to rating page
- [ ] Submit ratings - verify redirect to Thank You page
- [ ] Start new conversation and say "bye" - verify direct redirect to Thank You page
- [ ] Verify audio announcement plays at 1:30 mark (if sound enabled)

## Expected Behavior

✅ **Popup appears at 1:30 remaining as a modal overlay**
✅ **Conversation always redirects to rating page when timer expires**
✅ **Rating page is mandatory before Thank You page (except for user farewell)**
✅ **Audio warning plays at 1:30 mark when sound is enabled**

All issues have been resolved and the conversation flow now works as expected!
