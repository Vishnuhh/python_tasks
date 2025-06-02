# Task: Develop an Agentic Personal Assistant API 

## 1. Introduction  
You are to implement a FastAPI-based “Agentic Personal Assistant” service that can interpret a single natural-language instruction (in Tamil, English, or a mix), dynamically plan and invoke multiple Indian-focused services (restaurant search, reservation, messaging, reminders), handle failures and edge cases, and return a structured JSON summary of outcomes. This document outlines all functional and non-functional requirements, architectural guidelines, and deliverables needed for you to build, test, and deploy the project.

---

## 2. Objective & Scope  

- **Primary Objective**:  
  Build a RESTful FastAPI service exposing one endpoint (`POST /assistant`) which:  
  1. Receives a JSON payload with `user_id` and `instruction` (e.g., “Book a table for four at an Indian restaurant in Bandra, Mumbai, next Saturday at 8 PM, then send a WhatsApp confirmation, and remind me an hour before.”).  
  2. Uses a LangChain agent to decide, in real time, which “tools” to call (search_restaurant → reserve_table → send_message → schedule_reminder).  
  3. Coordinates the tool calls in sequence, re‐planning or prompting the user if a tool fails (e.g., “No availability—would you like 7 PM instead?”).  
  4. Returns a final JSON object summarising “restaurant,” “reservation,” “notification,” and “reminder” details.  

- **Key Challenges / Complexities**:  
  1. **Dynamic Planning**: The agent must select and invoke tools based on the user’s natural-language instruction (zero-shot or few-shot reasoning).  
  2. **Error Handling & Fallbacks**: If a tool returns “no results” or “no availability,” the agent should automatically propose alternatives or ask the user for clarification.  
  3. **Multi-turn Clarifications**: Support follow-up prompts (e.g., user changes date/time mid-conversation).  
  4. **Concurrency & State**: Multiple users may call `/assistant` concurrently; ensure isolation of each user’s session and agent memory.  
  5. **Indian Locale Considerations**:  
     - Date formats: ISO “YYYY-MM-DD” combined with 24-hour “HH:MM.”  
     - Timezone: All timestamps assume “Asia/Kolkata.”  
     - Localization: Accept instructions in Hindi+English (“Hinglish”) and ensure the agent handles them.  
  6. **Security & API Key Management**: Sensitive keys (LLM API key, external-API credentials) must be stored securely (e.g., via environment variables or a secrets manager).  

- **Out-of-Scope**:  
  - No front-end UI is required.  
  - Actual integration with a real LLM model (you can assume `OPENAI_API_KEY` is provided).  
  - Persisting long-term conversation history (beyond a single API invocation) unless explicitly flagged.  
  - Detailed CI/CD pipeline scripts (though deployment guidelines should be provided).  

---

## 3. Functional Requirements  

### 3.1. Overall Flow  
1. **Client Request**  
   - Endpoint: `POST /assistant`  
   - Payload:  
     ```json
     {
       "user_id": "string",
       "instruction": "string"
     }
     ```  
   - Response:  
     - **200** with a JSON body containing `restaurant`, `reservation`, `notification`, and (if requested) `reminder` objects (see Section 7).  
     - **4XX/5XX** with an error message if something goes wrong (e.g., LLM failure, invalid input).  

2. **Agent Invocation**  
   - On receiving a request, the FastAPI handler:  
     1. Validates `user_id` and `instruction` (neither may be empty; `user_id` length ≤ 64, `instruction` length ≤ 1000 characters).  
     2. Loads or generates the system prompt (see Section 5).  
     3. Instantiates a new LangChain agent (or retrieves an existing memory‐enabled agent for that `user_id`, if multi-turn is enabled).  
     4. Passes `instruction` to the agent’s `.run(...)` method.  
     5. The agent outputs a sequence of **Thought → Action → Observation** cycles:  
        - **Thought**: Natural-language reasoning by the LLM (not returned to client).  
        - **Action**: A JSON specifying which tool to call and with what inputs.  
        - **Observation**: The response from the tool (returned by FastAPI to the agent).  
     6. The agent continues until it returns a final JSON summarising all steps.  
     7. The FastAPI handler parses that final JSON, validates it against the schema, and responds to the client.

### 3.2. Tools & External Services (Wrappers)  
Implement four “tool wrappers” as Python functions. Each function:  
- Accepts a validated Python `dict` of inputs.  
- Calls an external Indian-focused API (or a local mock).  
- Translates the external API’s HTTP response into a consistent Python `dict` for the agent’s “Observation.”  
- If an external API call fails (4XX/5XX), returns a structured `{"error": "...", "details": "..."}` object.  

#### 3.2.1. Tool A: `search_restaurant`  
- **Purpose**: Given `city`, `locality`, `cuisine`, `date`, `time`, return up to **5** top restaurants that match.  
- **Input Fields (all required)**:  
  - `city` (string, e.g., `"Mumbai"`)  
  - `locality` (string, e.g., `"Bandra West"`)  
  - `cuisine` (string, any Indian cuisine or common variants, e.g., `"North Indian"`, `"Bengali"`)  
  - `date` (string in ISO format, `"YYYY-MM-DD"`, must be ≥ today’s date)  
  - `time` (string in 24-hour format, `"HH:MM"`)  

- **Output Fields**:  
  - `restaurants`: Array of 0–5 restaurant objects, each containing:  
    - `id` (string, unique, e.g., `"res_12345"`)  
    - `name` (string, e.g., `"Swaad South Indian Kitchen"`)  
    - `address` (string, e.g., `"Colaba Causeway, Mumbai"`)  
    - `rating` (float between 0.0 and 5.0)  
    - `cuisine` (string)  
    - `available_slots` (optional array of alternate time strings if fully booked, e.g., `["19:00", "21:00"]`)  

- **Error Cases**:  
  - **`400 Bad Request`**: Missing or invalid fields (`city`/`locality` blank, `date` in invalid format, `time` invalid).  
  - **`404 Not Found`**: No restaurants match the criteria (return `{"restaurants": []}`).  
  - **`429 Too Many Requests`**: Rate limit exceeded—agent should back off or ask user to retry.  
  - **`500 Server Error`**: Unexpected error—agent should retry once or inform the user.

#### 3.2.2. Tool B: `reserve_table`  
- **Purpose**: Given `restaurant_id`, `date`, `time`, `guests`, `user_name`, and `user_phone`, attempt to reserve a table.  
- **Input Fields (all required)**:  
  - `restaurant_id` (string, returned from `search_restaurant`)  
  - `date` (`"YYYY-MM-DD"`)  
  - `time` (`"HH:MM"`)  
  - `guests` (integer, 1 to 20)  
  - `user_name` (string, max 100 chars)  
  - `user_phone` (string, E.164 format, `"+91XXXXXXXXXX"`)  

- **Output Fields**:  
  - On **Success** (`status: "confirmed"`):  
    - `reservation_id` (string, e.g., `"rev_67890"`)  
    - `table_number` (string or integer, e.g., `"5"`)  
    - `status` (string, `"confirmed"`)  
    - `instructions` (string, e.g., `"Arrive by 7:50 PM."`)  
    - `alternate_slots` (optional array of available time strings if requested slot unavailable)  
  - On **Failure**:  
    - `status` (string, e.g., `"no_availability"` or `"invalid_restaurant"`)  
    - `error_message` (string, e.g., `"No tables available at 20:00. Next available: 19:00, 21:00."`)  

- **Error Cases**:  
  - **`400 Bad Request`**: Missing/invalid fields or malformed phone.  
  - **`404 Not Found`**: `restaurant_id` not recognized.  
  - **`409 Conflict`**: Requested slot fully booked (return `alternate_slots`). Agent must handle fallback.  
  - **`500 Server Error`**: Unexpected server fault—agent should retry once then escalate.

#### 3.2.3. Tool C: `send_message`  
- **Purpose**: Send a WhatsApp or SMS confirmation to the user’s phone.  
- **Input Fields (all required)**:  
  - `phone_number` (string, E.164 format, `"+91XXXXXXXXXX"`)  
  - `message_text` (string, up to 500 characters; must include reservation details)  

- **Output Fields**:  
  - On **Success**:  
    - `message_id` (string, e.g., `"msg_24680"`)  
    - `status` (string, `"sent"`)  
  - On **Failure**:  
    - `status` (string, `"failed"`)  
    - `error_message` (string, e.g., `"Invalid phone number format."`)  

- **Error Cases**:  
  - **`400 Bad Request`**: Invalid phone, message too long.  
  - **`401 Unauthorized`**: Missing/invalid API key.  
  - **`503 Service Unavailable`**: SMS gateway down—agent can retry once.  
  - **`500 Server Error`**: General failure—agent to escalate to user.

#### 3.2.4. Tool D: `schedule_reminder`  
- **Purpose**: Schedule a reminder for the user in the specified timezone (Asia/Kolkata).  
- **Input Fields (all required)**:  
  - `user_id` (string, e.g., `"user_ind001"`)  
  - `reminder_time` (string, ISO-8601 in “YYYY-MM-DDTHH:MM” format, e.g., `"2025-06-14T19:00"`)  
  - `message_text` (string, up to 300 characters, e.g., `"Your reservation at Swaad South Indian Kitchen is at 8 PM."`)  

- **Output Fields**:  
  - On **Success**:  
    - `reminder_id` (string, e.g., `"rem_13579"`)  
    - `status` (string, `"scheduled"`)  
    - `scheduled_for` (string, same as `reminder_time`)  
  - On **Failure**:  
    - `status` (string, e.g., `"failed"`)  
    - `error_message` (string, e.g., `"Reminder time is in the past."`)  

- **Error Cases**:  
  - **`400 Bad Request`**: Invalid fields, `reminder_time` earlier than current time.  
  - **`401 Unauthorized`**: Missing/invalid credentials for scheduling service.  
  - **`500 Server Error`**: Unexpected fault—agent should notify user to retry.

---

## 4. Non-Functional Requirements  

1. **Performance & Concurrency**  
   - The `/assistant` endpoint must handle at least **20 concurrent requests** with an average response latency (end-to-end, including LLM and external API calls) under **8 seconds**.  
   - Use asynchronous HTTP clients (`httpx.AsyncClient`) for external API calls.  
   - Limit LLM streaming to non-blocking calls; avoid blocking the event loop.  

2. **Scalability**  
   - Code should be container-friendly (stateless for the agent if not using memory).  
   - Configure thread pools / worker pools for CPU-bound tasks (if any future expansion).  

3. **Security**  
   - Store all API keys (LLM, restaurant APIs, messaging, scheduler) in environment variables.  
   - Validate all incoming JSON payloads strictly with Pydantic.  
   - Do not log or expose user phone numbers, reservation IDs, or reminder IDs in public logs.  
   - Use HTTPS in production; local dev can use HTTP.  

4. **Internationalization (i18n)**  
   - Accept instructions containing Hindi keywords (e.g., “मुझे कल शाम 8 बजे का टेबल बुक करना है।” or “Book a table tomorrow at 7 pm Hyderabad”).  
   - The agent’s prompt should mention that Hindi or Hinglish phrases may appear; instruct the LLM to handle them appropriately.  
   - Response JSON does NOT need translation; it must use standardized English field names and values.  

5. **Logging & Monitoring**  
   - Use a structured logger (e.g., Python’s `logging` with JSON formatting).  
   - Log at least:  
     - Incoming `instruction` and `user_id` at **INFO** level.  
     - Each tool-call “Action” JSON at **DEBUG** level.  
     - Each “Observation” (tool response) at **DEBUG** level.  
     - Agent’s final JSON output at **INFO** level.  
     - Any errors (HTTP errors, exceptions) at **ERROR** level (including stack trace).  
   - Expose a simple `/health` endpoint returning `{"status": "ok"}` for uptime checks.  

6. **Testing & Validation**  
   - Write **unit tests** for each tool wrapper:  
     - Mock external API responses (success, 400, 404, 409, 500) and assert the wrapper returns correct Python dicts.  
   - Write **integration tests** for the agent logic:  
     - Simulate “happy path” (search → reserve → send message → schedule reminder).  
     - Simulate fallback path (search returns empty → agent asks for clarification).  
     - Simulate reservation conflict (agent sees `409 No availability` → chooses alternate slot automatically).  
     - Simulate reminder scheduling failure (e.g., past time) and agent’s recovery.  
   - Use a test framework (e.g., `pytest`) and a test client (`httpx.AsyncClient` or FastAPI’s `TestClient`).  
   - Achieve at least **80% code coverage** across all modules.  

7. **Documentation & Code Quality**  
   - Each Python function and class must have a docstring (Google style or NumPy style).  
   - Provide a top‐level `README.md` summarizing:  
     - Project overview, prerequisites, how to run locally, how to run tests, how to deploy.  
   - Use **flake8** or **pylint** to enforce consistent code style; address all warnings.  
   - Write inline comments for complex logic (agent re‐planning, fallback handling).  

8. **Deployment & DevOps**  
   - Provide a `Dockerfile` that:  
     - Uses a slim Python base image (e.g., `python:3.10-slim`).  
     - Installs dependencies (`pip install -r requirements.txt`).  
     - Copies application code.  
     - Exposes port **8000** and runs `uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4`.  
   - Provide a sample `docker-compose.yml` (optional) to spin up the API plus a local mock server for tools (e.g., using “mockoon” or simple Flask mocks).  
   - Suggest basic logging and metrics:  
     - Expose a `/metrics` endpoint (Prometheus format) with request counts, latencies (optional).  
     - Provide instructions for deploying on AWS ECS (Mumbai region) or DigitalOcean App Platform (Bangalore region).  

---

## 5. Agent Prompt & Behavior Specification  

### 5.1. System Prompt Requirements  
The agent’s system prompt (passed to the LLM as “system” role) must include:

1. **Role Definition**  
   - “You are an AI assistant specialized in Indian restaurant reservations, notifications, and reminders. Your job is to orchestrate multiple tools in sequence and handle errors gracefully.”  

2. **Tool List & Schemas**  
   - For each tool—`search_restaurant`, `reserve_table`, `send_message`, `schedule_reminder`—list:  
     - **Tool Name**  
     - **Purpose** (one sentence)  
     - **Required Input Fields** (with types and examples)  
     - **Expected Output Fields** (with types and examples)  
     - **Common Error Cases**  

   - Example snippet for `search_restaurant`:  
     ```
     Tool Name: search_restaurant  
     Purpose: Given city, locality, cuisine, date, and time, return up to 5 top Indian restaurants in that area.  
     Input:  
       - city (string, e.g., "Mumbai")  
       - locality (string, e.g., "Bandra West")  
       - cuisine (string, e.g., "North Indian")  
       - date (string, YYYY-MM-DD, e.g., "2025-06-14")  
       - time (string, HH:MM, e.g., "20:00")  
     Output:  
       - restaurants: [  
           {  
             id (string, e.g., "res_12345"),  
             name (string, e.g., "The Bombay Canteen"),  
             address (string, e.g., "Lower Parel, Mumbai"),  
             rating (float, 0.0–5.0),  
             cuisine (string),  
             available_slots (optional [strings], e.g., ["19:00", "21:00"])  
           }, … up to 5 ]  
     Errors:  
       - 404: No restaurants found (return {"restaurants": []})  
       - 429: Rate limit exceeded (return {"error": "rate_limit", "details": "Try again in 1 minute."})  
       - 500: Server error (return {"error": "server_error", "details": "Internal server error."})  
     ```

   - Example snippet for `schedule_reminder`:  
     ```
     Tool Name: schedule_reminder  
     Purpose: Schedule a reminder for a user in Asia/Kolkata timezone.  
     Input:  
       - user_id (string, e.g., "user_ind001")  
       - reminder_time (string, ISO-8601 "YYYY-MM-DDTHH:MM", e.g., "2025-06-14T19:00")  
       - message_text (string, up to 300 chars, e.g., "Your reservation at Swaad Kitchen is at 8 PM.")  
     Output:  
       - reminder_id (string, e.g., "rem_13579")  
       - status (string, "scheduled")  
       - scheduled_for (string, same as reminder_time)  
     Errors:  
       - 400: Invalid fields or time in the past (return {"error": "invalid_time", "details": "Cannot schedule in the past."})  
       - 500: Server error (return {"error": "server_error", "details": "Internal server error."})  
     ```

3. **Reasoning Format (ReAct Style)**  
   - Specify that the agent must think “step by step” and output each reasoning step using:  
     ```
     Thought: <NL reasoning about which tool to call or how to handle an error>  
     Action:  
     {  
       "tool": "<tool_name>",  
       "input": { … }  
     }  
     ```  
   - After calling the tool, the FastAPI service injects an **Observation** (invisible to the user but used by the agent). Then the agent continues until final.

4. **Error Handling Instructions**  
   - If a tool returns an error object with `"error"` or `"status" != "confirmed"/"sent"/"scheduled"`, the agent must:  
     1. Output a `Thought` acknowledging the specific error.  
     2. Attempt a fallback:
        - For `search_restaurant` → suggest a different locality or date.
        - For `reserve_table` → choose earliest `alternate_slots` automatically.
        - For `send_message` → retry once or notify user to recheck phone number.
        - For `schedule_reminder` → ask user for a valid future time.
     3. If the error is unrecoverable (e.g., invalid phone number), instruct the user to re-specify.

5. **Multi-Turn Clarifications**  
   - If the agent cannot proceed (e.g., “Which cuisine do you prefer?”), it should output a JSON with a special field `"ask_user": "<message>"`. The FastAPI handler must return a **200** containing:  
     ```json
     {
       "ask_user": "Please specify whether you want North Indian or South Indian cuisine."
     }
     ```  
   - The client (or tester) will then call `/assistant` again with the original `instruction` plus any new clarification; the agent should continue from where it left off (use memory).

6. **Final Output Specification**  
   - When all tasks succeed, the agent must produce exactly one JSON object (no extra text) with keys:  
     - `restaurant`: `{ name, address, rating, id }`  
     - `reservation`: `{ reservation_id, date, time, table_number, status }`  
     - `notification`: `{ message_id, status }`  
     - `reminder`: `{ reminder_id, scheduled_for, status }` (if a reminder was requested)  
   - Example:  
     ```json
     {
       "restaurant": {
         "id": "res_89012",
         "name": "Swaad South Indian Kitchen",
         "address": "Colaba Causeway, Mumbai",
         "rating": 4.7
       },
       "reservation": {
         "reservation_id": "rev_33445",
         "date": "2025-06-15",
         "time": "20:00",
         "table_number": "5",
         "status": "confirmed"
       },
       "notification": {
         "message_id": "msg_78901",
         "status": "sent"
       },
       "reminder": {
         "reminder_id": "rem_13579",
         "scheduled_for": "2025-06-15T19:00",
         "status": "scheduled"
       }
     }
     ```
   - If any step ultimately fails, return a JSON:  
     ```json
     {
       "error": "<high-level error summary>",
       "details": "<optional technical details>"
     }
     ```

---

## 6. API Endpoint Specification  

### 6.1. Endpoint: `POST /assistant`  
- **Request Headers**  
  - `Content-Type: application/json`  
  - (Optional) `X-User-Locale: "hi-IN"` or `"en-IN"` to hint at language preference.  

- **Request Body Schema** (`AssistantRequest`)  
  ```json
  {
    "user_id": "string (required, max 64 chars)",
    "instruction": "string (required, max 1000 chars)"
  }
