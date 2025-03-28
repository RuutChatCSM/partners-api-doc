# Ruut API Documentation

## Authentication and Access

### Obtaining API Access Token and Account ID

Follow these steps to get your API access token from the Ruut dashboard:

## 1. Login to Ruut
- Open Ruut on your browser (e.g., `https://app.ruut.chat`).
- Enter your credentials and log in.

## 2. Navigate to Profile Settings
- Click on your **Profile Avatar** at the bottom left corner.
- Select **Profile Settings**.

## 3. Copy Your Access Token
- Scroll down to the **Access Token** section.
- Copy the token displayed there.

#### Locating Your Account ID via Dashboard
1. In the Ruut admin dashboard
2. Go to Settings > Account Information
3. Your Account ID will be displayed in the top section
4. Alternatively, make an API call to `/api/v1/accounts` to retrieve account details

### API Authentication Methods

#### Header-Based Authentication
```http
api_access_token: YOUR_API_ACCESS_TOKEN
```

<!-- #### Query Parameter Authentication (Not Recommended)
```http
GET /api/v1/conversations?api_access_token=YOUR_TOKEN
``` -->

## Conversations API in Depth

### Conversation Statuses and Workflow

#### Status Types
1. **Open**
   - Default status for new conversations
   - Indicates an active, unresolved interaction
   - Visible to all agents in the inbox
   - Triggers notifications for agent assignment

2. **Pending**
   - Conversation awaiting agent response
   - Often used when initial customer inquiry requires investigation
   - May have specific routing rules
   - Agents can manually set or automatically triggered by system rules

3. **Resolved**
   - Conversation considered completed
   - No further immediate action required
   - Can be reopened if customer provides additional context
   - Typically moves conversation to a resolved conversations archive

4. **Snoozed**
   - Temporarily paused conversation
   - Used when:
     * Waiting for customer information
     * Requiring internal team consultation
     * Scheduled for follow-up at a specific time
   - Automatically or manually set
   - Doesn't close the conversation permanently

5. **Scheduling**
   - Conversation marked for future follow-up
   - Can be associated with specific time/date
   - Useful for sales pipelines or complex support scenarios

### Status Change Behaviors

#### Open → Pending
- Triggers notification to assigned agent
- Pauses automatic resolution timers
- Maintains conversation in active queue

#### Pending → Resolved
- Closes active conversation thread
- Stops further automatic notifications
- Archives conversation for future reference

#### Resolved → Reopened
- Moves conversation back to active status
- Resets some automatic workflow triggers
- Notifies original or new agent about reopening

### Conversation Status Management API

#### Change Conversation Status
```http
PATCH /api/v1/conversations/{conversation_id}/status
```

##### Request Body Examples
```json
{
  "status": "resolved",
  "reason": "Customer issue addressed",
  "agent_id": "current_agent_identifier"
}

{
  "status": "snoozed",
  "snooze_time": "2024-04-15T14:30:00Z"
}
```

### Agent Bot Interaction with Conversation Statuses

#### Bot Access and Status Workflow
- Agent bots can interact with conversations based on their configured permissions
- Status changes can trigger bot workflows
- Example bot logic:
  ```javascript
  if (conversation.status === 'open') {
    // Initial response handling
    sendWelcomeMessage();
    transferToSuitableAgent();
  }

  if (conversation.status === 'pending') {
    // Escalation or additional context gathering
    requestMoreInformation();
  }
  ```

### Conversation Metadata

#### Additional Conversation Attributes
```json
{
  "id": "unique_conversation_id",
  "status": "open",
  "channel": "web",
  "inbox_id": "support_inbox",
  "contact": {
    "id": "customer_contact_id",
    "name": "Customer Name",
    "email": "customer@example.com"
  },
  "assignee": {
    "id": "agent_id",
    "name": "Agent Name"
  },
  "created_at": "2024-03-28T10:15:30Z",
  "updated_at": "2024-03-28T10:20:45Z"
}
```

### Best Practices for Conversation Management
1. Always include context when changing conversation status
2. Use appropriate status for accurate workflow routing
3. Implement timeout mechanisms for pending conversations
4. Allow manual override of automated status changes
5. Log status change reasons for audit trails

### Error Handling
```json
{
  "error": {
    "code": "STATUS_CHANGE_FORBIDDEN",
    "message": "Cannot change status due to current conversation state",
    "details": "Conversation is locked or requires specific permissions"
  }
}
```

## Security Considerations
- Rotate API access tokens periodically
- Use the least privilege principle when generating tokens
- Monitor and audit API access logs
- Implement IP whitelisting for API access

## Troubleshooting
- Verify API access token validity
- Check account permissions
- Ensure correct account ID is used
- Validate webhook configurations