# API Reference

## Core API

### Core Class

The `Core` class is the main entry point for all Backlog.md operations.

#### Constructor

```typescript
constructor(projectPath: string)
```

**Parameters:**
- `projectPath` (string): Path to the project directory

**Example:**
```typescript
const core = new Core("/path/to/project");
```

#### Methods

##### `initializeProject(name: string): Promise<void>`

Initialize a new Backlog.md project.

**Parameters:**
- `name` (string): Project name

**Example:**
```typescript
await core.initializeProject("My Project");
```

##### `createTask(task: Task, autoCommit: boolean): Promise<string>`

Create a new task.

**Parameters:**
- `task` (Task): Task object to create
- `autoCommit` (boolean): Whether to auto-commit changes

**Returns:**
- `Promise<string>`: ID of the created task

**Example:**
```typescript
const taskId = await core.createTask({
  id: "task-1",
  title: "New Task",
  status: "To Do",
  assignee: [],
  createdDate: "2025-01-01",
  labels: [],
  dependencies: [],
  body: "Task description"
}, true);
```

##### `updateTask(id: string, updates: Partial<Task>): Promise<void>`

Update an existing task.

**Parameters:**
- `id` (string): Task ID to update
- `updates` (Partial<Task>): Partial task object with updates

**Example:**
```typescript
await core.updateTask("task-1", {
  status: "In Progress",
  assignee: ["user@example.com"]
});
```

##### `deleteTask(id: string, autoCommit: boolean): Promise<void>`

Delete a task.

**Parameters:**
- `id` (string): Task ID to delete
- `autoCommit` (boolean): Whether to auto-commit changes

**Example:**
```typescript
await core.deleteTask("task-1", true);
```

##### `getTask(id: string): Promise<Task | null>`

Get a task by ID.

**Parameters:**
- `id` (string): Task ID to retrieve

**Returns:**
- `Promise<Task | null>`: Task object or null if not found

**Example:**
```typescript
const task = await core.getTask("task-1");
if (task) {
  console.log(task.title);
}
```

##### `listTasks(options?: ListTasksOptions): Promise<Task[]>`

List tasks with optional filtering.

**Parameters:**
- `options` (ListTasksOptions, optional): Filtering options

**Returns:**
- `Promise<Task[]>`: Array of task objects

**Example:**
```typescript
const tasks = await core.listTasks({
  status: "To Do",
  assignee: ["user@example.com"],
  labels: ["bug"]
});
```

##### `createDocument(document: Document, autoCommit: boolean): Promise<string>`

Create a new document.

**Parameters:**
- `document` (Document): Document object to create
- `autoCommit` (boolean): Whether to auto-commit changes

**Returns:**
- `Promise<string>`: ID of the created document

**Example:**
```typescript
const docId = await core.createDocument({
  id: "doc-1",
  title: "Architecture Guide",
  type: "guide",
  createdDate: "2025-01-01",
  content: "# Architecture Guide\n\n..."
}, true);
```

##### `updateDocument(id: string, updates: Partial<Document>): Promise<void>`

Update an existing document.

**Parameters:**
- `id` (string): Document ID to update
- `updates` (Partial<Document>): Partial document object with updates

**Example:**
```typescript
await core.updateDocument("doc-1", {
  content: "# Updated Architecture Guide\n\n..."
});
```

##### `deleteDocument(id: string, autoCommit: boolean): Promise<void>`

Delete a document.

**Parameters:**
- `id` (string): Document ID to delete
- `autoCommit` (boolean): Whether to auto-commit changes

**Example:**
```typescript
await core.deleteDocument("doc-1", true);
```

##### `getDocument(id: string): Promise<Document | null>`

Get a document by ID.

**Parameters:**
- `id` (string): Document ID to retrieve

**Returns:**
- `Promise<Document | null>`: Document object or null if not found

**Example:**
```typescript
const doc = await core.getDocument("doc-1");
if (doc) {
  console.log(doc.title);
}
```

##### `listDocuments(options?: ListDocumentsOptions): Promise<Document[]>`

List documents with optional filtering.

**Parameters:**
- `options` (ListDocumentsOptions, optional): Filtering options

**Returns:**
- `Promise<Document[]>`: Array of document objects

**Example:**
```typescript
const docs = await core.listDocuments({
  type: "guide",
  createdAfter: "2025-01-01"
});
```

##### `createDecision(decision: Decision, autoCommit: boolean): Promise<string>`

Create a new decision record.

**Parameters:**
- `decision` (Decision): Decision object to create
- `autoCommit` (boolean): Whether to auto-commit changes

**Returns:**
- `Promise<string>`: ID of the created decision

**Example:**
```typescript
const decisionId = await core.createDecision({
  id: "decision-1",
  title: "Use TypeScript",
  status: "accepted",
  createdDate: "2025-01-01",
  content: "# Decision: Use TypeScript\n\n..."
}, true);
```

##### `updateDecision(id: string, updates: Partial<Decision>): Promise<void>`

Update an existing decision.

**Parameters:**
- `id` (string): Decision ID to update
- `updates` (Partial<Decision>): Partial decision object with updates

**Example:**
```typescript
await core.updateDecision("decision-1", {
  status: "accepted",
  impact: "Improved type safety"
});
```

##### `deleteDecision(id: string, autoCommit: boolean): Promise<void>`

Delete a decision.

**Parameters:**
- `id` (string): Decision ID to delete
- `autoCommit` (boolean): Whether to auto-commit changes

**Example:**
```typescript
await core.deleteDecision("decision-1", true);
```

##### `getDecision(id: string): Promise<Decision | null>`

Get a decision by ID.

**Parameters:**
- `id` (string): Decision ID to retrieve

**Returns:**
- `Promise<Decision | null>`: Decision object or null if not found

**Example:**
```typescript
const decision = await core.getDecision("decision-1");
if (decision) {
  console.log(decision.title);
}
```

##### `listDecisions(options?: ListDecisionsOptions): Promise<Decision[]>`

List decisions with optional filtering.

**Parameters:**
- `options` (ListDecisionsOptions, optional): Filtering options

**Returns:**
- `Promise<Decision[]>`: Array of decision objects

**Example:**
```typescript
const decisions = await core.listDecisions({
  status: "accepted",
  createdAfter: "2025-01-01"
});
```

## Types

### Task

```typescript
interface Task {
  id: string;
  title: string;
  status: string;
  assignee: string[];
  createdDate: string;
  labels: string[];
  dependencies: string[];
  body: string;
  acceptanceCriteria?: string[];
  implementationNotes?: string;
  priority?: 'high' | 'medium' | 'low';
}
```

**Properties:**
- `id` (string): Unique task identifier
- `title` (string): Task title
- `status` (string): Task status (e.g., "To Do", "In Progress", "Done")
- `assignee` (string[]): Array of assignee email addresses
- `createdDate` (string): Creation date in YYYY-MM-DD format
- `labels` (string[]): Array of task labels
- `dependencies` (string[]): Array of task IDs this task depends on
- `body` (string): Task description
- `acceptanceCriteria` (string[], optional): Array of acceptance criteria
- `implementationNotes` (string, optional): Implementation notes
- `priority` ('high' | 'medium' | 'low', optional): Task priority

### Document

```typescript
interface Document {
  id: string;
  title: string;
  type: string;
  createdDate: string;
  content: string;
}
```

**Properties:**
- `id` (string): Unique document identifier
- `title` (string): Document title
- `type` (string): Document type (e.g., "guide", "spec", "reference")
- `createdDate` (string): Creation date in YYYY-MM-DD format
- `content` (string): Document content in markdown format

### Decision

```typescript
interface Decision {
  id: string;
  title: string;
  status: string;
  createdDate: string;
  content: string;
  impact?: string;
  alternatives?: string[];
}
```

**Properties:**
- `id` (string): Unique decision identifier
- `title` (string): Decision title
- `status` (string): Decision status (e.g., "proposed", "accepted", "rejected")
- `createdDate` (string): Creation date in YYYY-MM-DD format
- `content` (string): Decision content in markdown format
- `impact` (string, optional): Impact of the decision
- `alternatives` (string[], optional): Array of considered alternatives

### ListTasksOptions

```typescript
interface ListTasksOptions {
  status?: string;
  assignee?: string[];
  labels?: string[];
  createdAfter?: string;
  createdBefore?: string;
  priority?: 'high' | 'medium' | 'low';
}
```

**Properties:**
- `status` (string, optional): Filter by task status
- `assignee` (string[], optional): Filter by assignee
- `labels` (string[], optional): Filter by labels
- `createdAfter` (string, optional): Filter tasks created after this date
- `createdBefore` (string, optional): Filter tasks created before this date
- `priority` ('high' | 'medium' | 'low', optional): Filter by priority

### ListDocumentsOptions

```typescript
interface ListDocumentsOptions {
  type?: string;
  createdAfter?: string;
  createdBefore?: string;
}
```

**Properties:**
- `type` (string, optional): Filter by document type
- `createdAfter` (string, optional): Filter documents created after this date
- `createdBefore` (string, optional): Filter documents created before this date

### ListDecisionsOptions

```typescript
interface ListDecisionsOptions {
  status?: string;
  createdAfter?: string;
  createdBefore?: string;
}
```

**Properties:**
- `status` (string, optional): Filter by decision status
- `createdAfter` (string, optional): Filter decisions created after this date
- `createdBefore` (string, optional): Filter decisions created before this date

## MCP API

### MCP Tools

#### Task Tools

##### `create-task`

Create a new task.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    title: {
      type: "string",
      description: "Task title",
      minLength: 1
    },
    status: {
      type: "string",
      enum: ["To Do", "In Progress", "Done"],
      default: "To Do"
    },
    assignee: {
      type: "array",
      items: { type: "string" },
      default: []
    },
    labels: {
      type: "array",
      items: { type: "string" },
      default: []
    },
    body: {
      type: "string",
      default: ""
    },
    priority: {
      type: "string",
      enum: ["high", "medium", "low"],
      default: "medium"
    }
  },
  required: ["title"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        taskId: string,
        success: boolean
      })
    }
  ]
}
```

##### `update-task`

Update an existing task.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    id: {
      type: "string",
      description: "Task ID to update",
      minLength: 1
    },
    title: {
      type: "string",
      description: "New task title"
    },
    status: {
      type: "string",
      enum: ["To Do", "In Progress", "Done"]
    },
    assignee: {
      type: "array",
      items: { type: "string" }
    },
    labels: {
      type: "array",
      items: { type: "string" }
    },
    body: {
      type: "string"
    },
    priority: {
      type: "string",
      enum: ["high", "medium", "low"]
    }
  },
  required: ["id"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        success: boolean,
        message: string
      })
    }
  ]
}
```

##### `delete-task`

Delete a task.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    id: {
      type: "string",
      description: "Task ID to delete",
      minLength: 1
    }
  },
  required: ["id"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        success: boolean,
        message: string
      })
    }
  ]
}
```

##### `list-tasks`

List tasks with optional filtering.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    status: {
      type: "string",
      description: "Filter by task status"
    },
    assignee: {
      type: "array",
      items: { type: "string" },
      description: "Filter by assignee"
    },
    labels: {
      type: "array",
      items: { type: "string" },
      description: "Filter by labels"
    },
    createdAfter: {
      type: "string",
      description: "Filter tasks created after this date (YYYY-MM-DD)"
    },
    createdBefore: {
      type: "string",
      description: "Filter tasks created before this date (YYYY-MM-DD)"
    },
    priority: {
      type: "string",
      enum: ["high", "medium", "low"],
      description: "Filter by priority"
    }
  }
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        tasks: Task[],
        count: number
      })
    }
  ]
}
```

#### Document Tools

##### `create-document`

Create a new document.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    title: {
      type: "string",
      description: "Document title",
      minLength: 1
    },
    type: {
      type: "string",
      description: "Document type",
      default: "guide"
    },
    content: {
      type: "string",
      description: "Document content in markdown format",
      default: ""
    }
  },
  required: ["title"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        documentId: string,
        success: boolean
      })
    }
  ]
}
```

##### `update-document`

Update an existing document.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    id: {
      type: "string",
      description: "Document ID to update",
      minLength: 1
    },
    title: {
      type: "string",
      description: "New document title"
    },
    type: {
      type: "string",
      description: "New document type"
    },
    content: {
      type: "string",
      description: "New document content"
    }
  },
  required: ["id"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        success: boolean,
        message: string
      })
    }
  ]
}
```

##### `delete-document`

Delete a document.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    id: {
      type: "string",
      description: "Document ID to delete",
      minLength: 1
    }
  },
  required: ["id"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        success: boolean,
        message: string
      })
    }
  ]
}
```

##### `list-documents`

List documents with optional filtering.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    type: {
      type: "string",
      description: "Filter by document type"
    },
    createdAfter: {
      type: "string",
      description: "Filter documents created after this date (YYYY-MM-DD)"
    },
    createdBefore: {
      type: "string",
      description: "Filter documents created before this date (YYYY-MM-DD)"
    }
  }
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        documents: Document[],
        count: number
      })
    }
  ]
}
```

#### Decision Tools

##### `create-decision`

Create a new decision record.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    title: {
      type: "string",
      description: "Decision title",
      minLength: 1
    },
    status: {
      type: "string",
      enum: ["proposed", "accepted", "rejected"],
      default: "proposed"
    },
    content: {
      type: "string",
      description: "Decision content in markdown format",
      default: ""
    },
    impact: {
      type: "string",
      description: "Impact of the decision"
    },
    alternatives: {
      type: "array",
      items: { type: "string" },
      description: "Considered alternatives"
    }
  },
  required: ["title"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        decisionId: string,
        success: boolean
      })
    }
  ]
}
```

##### `update-decision`

Update an existing decision.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    id: {
      type: "string",
      description: "Decision ID to update",
      minLength: 1
    },
    title: {
      type: "string",
      description: "New decision title"
    },
    status: {
      type: "string",
      enum: ["proposed", "accepted", "rejected"]
    },
    content: {
      type: "string",
      description: "New decision content"
    },
    impact: {
      type: "string",
      description: "New impact description"
    },
    alternatives: {
      type: "array",
      items: { type: "string" },
      description: "New alternatives"
    }
  },
  required: ["id"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        success: boolean,
        message: string
      })
    }
  ]
}
```

##### `delete-decision`

Delete a decision.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    id: {
      type: "string",
      description: "Decision ID to delete",
      minLength: 1
    }
  },
  required: ["id"]
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        success: boolean,
        message: string
      })
    }
  ]
}
```

##### `list-decisions`

List decisions with optional filtering.

**Input Schema:**
```typescript
{
  type: "object",
  properties: {
    status: {
      type: "string",
      description: "Filter by decision status"
    },
    createdAfter: {
      type: "string",
      description: "Filter decisions created after this date (YYYY-MM-DD)"
    },
    createdBefore: {
      type: "string",
      description: "Filter decisions created before this date (YYYY-MM-DD)"
    }
  }
}
```

**Response:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        decisions: Decision[],
        count: number
      })
    }
  ]
}
```

### MCP Resources

#### Task Resources

##### `tasks://list`

List all tasks.

**Response:**
```typescript
{
  contents: [
    {
      uri: "tasks://list",
      text: JSON.stringify({
        tasks: Task[],
        count: number
      })
    }
  ]
}
```

##### `tasks://active`

List active tasks (not "Done").

**Response:**
```typescript
{
  contents: [
    {
      uri: "tasks://active",
      text: JSON.stringify({
        tasks: Task[],
        count: number
      })
    }
  ]
}
```

##### `tasks://completed`

List completed tasks ("Done" status).

**Response:**
```typescript
{
  contents: [
    {
      uri: "tasks://completed",
      text: JSON.stringify({
        tasks: Task[],
        count: number
      })
    }
  ]
}
```

##### `tasks://assigned/{user}`

List tasks assigned to a specific user.

**Parameters:**
- `user` (string): User email address

**Response:**
```typescript
{
  contents: [
    {
      uri: "tasks://assigned/{user}",
      text: JSON.stringify({
        tasks: Task[],
        count: number
      })
    }
  ]
}
```

##### `tasks://status/{status}`

List tasks with a specific status.

**Parameters:**
- `status` (string): Task status

**Response:**
```typescript
{
  contents: [
    {
      uri: "tasks://status/{status}",
      text: JSON.stringify({
        tasks: Task[],
        count: number
      })
    }
  ]
}
```

#### Document Resources

##### `documents://list`

List all documents.

**Response:**
```typescript
{
  contents: [
    {
      uri: "documents://list",
      text: JSON.stringify({
        documents: Document[],
        count: number
      })
    }
  ]
}
```

##### `documents://type/{type}`

List documents of a specific type.

**Parameters:**
- `type` (string): Document type

**Response:**
```typescript
{
  contents: [
    {
      uri: "documents://type/{type}",
      text: JSON.stringify({
        documents: Document[],
        count: number
      })
    }
  ]
}
```

##### `documents://recent`

List recent documents (created in the last 30 days).

**Response:**
```typescript
{
  contents: [
    {
      uri: "documents://recent",
      text: JSON.stringify({
        documents: Document[],
        count: number
      })
    }
  ]
}
```

#### Decision Resources

##### `decisions://list`

List all decisions.

**Response:**
```typescript
{
  contents: [
    {
      uri: "decisions://list",
      text: JSON.stringify({
        decisions: Decision[],
        count: number
      })
    }
  ]
}
```

##### `decisions://status/{status}`

List decisions with a specific status.

**Parameters:**
- `status` (string): Decision status

**Response:**
```typescript
{
  contents: [
    {
      uri: "decisions://status/{status}",
      text: JSON.stringify({
        decisions: Decision[],
        count: number
      })
    }
  ]
}
```

##### `decisions://recent`

List recent decisions (created in the last 30 days).

**Response:**
```typescript
{
  contents: [
    {
      uri: "decisions://recent",
      text: JSON.stringify({
        decisions: Decision[],
        count: number
      })
    }
  ]
}
```

#### Project Resources

##### `project://config`

Get project configuration.

**Response:**
```typescript
{
  contents: [
    {
      uri: "project://config",
      text: JSON.stringify({
        project_name: string,
        default_assignee?: string,
        default_status: string,
        statuses: string[],
        labels: string[],
        milestones: string[],
        date_format: string
      })
    }
  ]
}
```

##### `project://stats`

Get project statistics.

**Response:**
```typescript
{
  contents: [
    {
      uri: "project://stats",
      text: JSON.stringify({
        totalTasks: number,
        activeTasks: number,
        completedTasks: number,
        totalDocuments: number,
        totalDecisions: number,
        recentActivity: Activity[]
      })
    }
  ]
}
```

##### `project://milestones`

Get project milestones.

**Response:**
```typescript
{
  contents: [
    {
      uri: "project://milestones",
      text: JSON.stringify({
        milestones: Milestone[]
      })
    }
  ]
}
```

## Error Handling

### Error Types

#### `ValidationError`

Thrown when input validation fails.

**Properties:**
- `message` (string): Error message
- `errors` (ValidationError[]): Array of validation errors

**Example:**
```typescript
try {
  await core.createTask({
    title: "", // Invalid: empty title
    status: "To Do",
    assignee: [],
    createdDate: "2025-01-01",
    labels: [],
    dependencies: [],
    body: ""
  }, false);
} catch (error) {
  if (error instanceof ValidationError) {
    console.error("Validation failed:", error.errors);
  }
}
```

#### `NotFoundError`

Thrown when a requested resource is not found.

**Properties:**
- `message` (string): Error message
- `resourceType` (string): Type of resource that was not found
- `resourceId` (string): ID of the resource that was not found

**Example:**
```typescript
try {
  const task = await core.getTask("non-existent-task");
} catch (error) {
  if (error instanceof NotFoundError) {
    console.error(`Task not found: ${error.resourceId}`);
  }
}
```

#### `FileSystemError`

Thrown when file system operations fail.

**Properties:**
- `message` (string): Error message
- `code` (string): File system error code
- `path` (string): Path that caused the error

**Example:**
```typescript
try {
  await core.initializeProject("/invalid/path");
} catch (error) {
  if (error instanceof FileSystemError) {
    console.error(`File system error: ${error.code} at ${error.path}`);
  }
}
```

### Error Responses

#### MCP Error Responses

When MCP tools encounter errors, they return structured error responses.

**Format:**
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        error: string,
        message: string,
        details?: any
      })
    }
  ],
  isError: true
}
```

**Examples:**

Validation Error:
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        error: "Validation error",
        message: "Invalid input data",
        details: [
          {
            field: "title",
            message: "Title is required"
          }
        ]
      })
    }
  ],
  isError: true
}
```

Not Found Error:
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        error: "Not found",
        message: "Task not found",
        details: {
          resourceType: "task",
          resourceId: "task-999"
        }
      })
    }
  ],
  isError: true
}
```

File System Error:
```typescript
{
  content: [
    {
      type: "text",
      text: JSON.stringify({
        error: "File system error",
        message: "Failed to write file",
        details: {
          code: "EACCES",
          path: "/path/to/task.md"
        }
      })
    }
  ],
  isError: true
}
```