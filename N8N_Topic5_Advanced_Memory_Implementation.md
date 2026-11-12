# Implementation Plan: N8N AI Agent Advanced Hands-On Course

## Topic 5: Managing Agent State with Advanced Memory Systems

Build a session-isolated AI chat agent, test temporary memory, and upgrade to PostgreSQL-backed persistent memory.

## Step 1: Set Up Chat Canvas and Parent Agent

1. Create a new n8n workflow and add the When Chat Message is Received trigger.
2. Connect an AI Agent node and attach an OpenAI Chat Model (or another supported LLM). Set temperature to 0.0.
3. Set the AI Agent system prompt to: You are a profile manager assistant. Your job is to remember user preferences (such as names, favorite colors, or hobbies) and refer to them in conversation.

## Step 2: Implement Temporary Session Memory

1. Connect the Simple Memory node (called Window Buffer Memory in some versions) to the AI Agent Memory port.
2. Set Session ID to the chat trigger session identifier, using the expression {{ $json.sessionId }} when available in the node context. If the node is a sub-node, reference the trigger explicitly.
3. Set the context window length to 3. This is temporary memory, not guaranteed to survive service restarts.

## Step 3: Test Session Separation

1. Use session ID user_alice_101 and send: My favorite color is emerald green.
2. Change to session ID user_bob_202 and ask: What is my favorite color? The agent should say it does not know.
3. Return to user_alice_101 and ask: What is my favorite color? The agent should answer emerald green.
4. Ensure each test really sends the intended sessionId. The built-in chat preview may generate its own session ID; use a controlled chat client or test payload if the UI does not permit changing it.

## Step 4: Upgrade to PostgreSQL Chat Memory

1. Remove the temporary memory node and connect Postgres Chat Memory to the AI Agent Memory port.
2. Create PostgreSQL credentials using the actual Host, Database, User, Password and Port from your database provider. For Supabase, use the appropriate connection pooler and SSL settings.
3. Set Session ID to {{ $json.sessionId }} or a direct chat-trigger reference if required. Set Table Name to n8n_chat_history.
4. Execute a chat turn after the database connection succeeds; the node can create its history table automatically when permitted.

## Step 5: Verify Persistent Writes

1. Use session ID user_compliance_99 and send: System test: Requesting connection confirmation.
2. In Supabase SQL Editor or another PostgreSQL client, run the query below.
3. Confirm the returned history belongs to user_compliance_99 and includes the user request and agent reply. Standard n8n Postgres Chat Memory implementations may store both messages in one row as JSON rather than exactly two rows.
4. Restart or reopen n8n, reuse the same session ID and verify that the agent can recall previously stored information.

```sql
SELECT * FROM n8n_chat_history
WHERE session_id = 'user_compliance_99'
ORDER BY id ASC;
```

## Troubleshooting and Completion Checklist

1. If n8n reports DNS errors, verify the exact hostname and whether the database project is active. A saved credential does not mean a successful connection.
2. If authentication fails, check the pooler username, database password, port and SSL mode.
3. Collect evidence: workflow canvas, Alice memory test, Bob isolation test, PostgreSQL credential connection success (hide secrets), SQL query results and restart persistence test.

