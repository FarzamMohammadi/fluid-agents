# Architecture https://docs.ag-ui.com/concepts/architecture
- Event-driven
- Bidirectional communication


# Events https://docs.ag-ui.com/concepts/events
Types:
  - Lifecycle -> monitor the progression of agent runs
  - Text Message -> Handle streaming textual content
  - Tool Call -> Manage tool execution by agent
  - State Management -> Sync state between agent and UI
  - Activity -> Represent ongoing activity progress
  - Special -> Support custom functionality
  - Draft -> Proposed events under development

Base event properties:
  - `type`: event type ID
  - `timestamp`: optional timestamp of when event was created
  - `metadata`: optional field containing original event data if transformed


## Lifecycle Events (5): Enables progress tracking and managing UI states, including error handling.
  - `RunsStarted` (mandatory): signals the start of an agent run and establishes a new execution context with a unique `runId`
  - `StepStarted` / `StepFinished` (optional): indicate the start and completion of steps within an agent run, providing granular visibility into the agent's process - identified by `stepName` (seemingly no `runId`)
  - `RunFinished` (mandatory): signals the successful completion of an agent run - identified by `runId`
  - `RunError` (mandatory): signals an error during an agent run, providing details - identified by `runId`

RunStarted Event:
  - Establishes new execution context identified by `runId` Can use this for initializing UI state tracking

RunFinished Event:
  - Frontend should finalize UI state tracking
  - Optional `result` field may contain any data produced by the agent

RunError Event:
  - Provides details on what went wrong, allowing the frontend to display error messages or take corrective actions
  - After this event, no further processing will occur

StepStarted Event:
  - Could indicate the start of a subtask or a phase of its processing
  - Provides granular visibility into the agent's process - enabling more processing tracking and UI feedback
  - While optional, they're recommended for complex operations that benefit from being broken down into observable stages
  - `stepName` is used to output the name of a node or a function that's being executed

StepFinished Event:
  - Indicates a subtask or phase has finished
  - When coupled with `StepStarted`, it allows for detailed tracking of the agent's subprocesses
  - `stepName` is used to output the name of a node or a function that's being executed - which matches the `stepName` in the corresponding `StepStarted` event


## Text Message Events (4): Represent the lifecycle of a text message in a conversation. Follows a streaming pattern, where content is delivered incrementally.
  - `TextMessageStarted` (mandatory): signals the start of a text message with unique `messageId` and `role`
  - `TextMessageContent` (mandatory): delivers chunks of text in `delta` which should be concatenated to form the complete message
  - `TextMessageEnd` (mandatory): signals the conclusion of a text message with the corresponding `messageId`
  - `TextMessageChunk` (not explicitly defined but seems mandatory) convenience event that expands to Start → Content → End automatically.

TextMessageStarted Event:
  - Established unique `messageId` that'll be references by subsequent `TextMessageContent` and `TextMessageEnd` events
  - `role` property identifies who the message is from
    - Role of the message sender (“developer”, “system”, “assistant”, “user”, “tool”)

TextMessageContent Event:
  - Represents a chunk of content in a streaming text message
  - Delivers incremental parts as they become available 
  - Chunk is delivered via `delta` property - which should be appended to previously received chunks

TextMessageEnd Event:
    - `messageId` is used to identify which text message is ending

TextMessageChunk Event:
  - Not documented well
  - This lets us omit explicit TextMessageStart and TextMessageEnd events:
    - First chunk for a message must include messageId and will emit TextMessageStart (role defaults to assistant when not provided)
    - Each chunk with a delta emits a TextMessageContent for the current messageId
    - TextMessageEnd is emitted automatically when the stream switches to a new message ID or when the stream completes


## Tool Call Events (4): Events representing the lifecycle of tool calls made by agents.
  - `ToolCallStart` (mandatory): signals the start of a tool call, establishing a new tool call context with unique `toolCallId`
  - `ToolCallArgs` (mandatory): streaming arguments being passed to the tool via `delta` property, which should be concatenated to form the complete arguments. `toolCallId` is used to associate the arguments with the correct tool call
  - `ToolCallEnd` (mandatory): signals the conclusion of a tool call. Where agent has finished specifying the tool and its arguments and **is now waiting for OR has received the results**
  - `ToolCallResult` (mandatory): delivers the results of a tool call - identified by `toolCallId` - outputting results via `content` property
  - `ToolCallChunk` (not explicitly defined but seems mandatory) Convenience event that expands to Start → Args → End automatically

ToolCallStart Event:
  - Establishes a new tool call context identified by `toolCallId`
  - Optional `parentMessageId` allows linking the tool call to a specific message in conversation - providing context on why the tool is being used

ToolCallArgs Event:
 - Delivers streaming incremental parts of tool's arguments
 - Each event contains a segment of the argument data in `delta` property
   - Deltas are often JSON fragments that need to be concatenated
 - Frontend can reveal arguments to provide insights into what parameters are being passed to the tool

ToolCallEnd Event:
  - Marks the completion of a tool -call- identified by `toolCallId`
  - This event indicates the agent completed specifying the tool and its arguments **and is now waiting for OR has received the results**

ToolCallResult Event:
  - Always sent AFTER `ToolCallEnd` and after the system has executed the tool and obtained results
  - Does not follow a streaming pattern - the complete result is delivered in a single unit via `content` property
  - Frontend can use this to display results, append to conversation, or trigger additional actions

ToolCallChunk Event:
  - Similar to TextMessageChunk, not well documented
  - Similar to TextMessageChunk, this lets us omit explicit ToolCallStart and ToolCallEnd events