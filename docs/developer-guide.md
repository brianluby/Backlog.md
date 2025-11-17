# Developer Guide

## Getting Started

### Prerequisites

- **Node.js**: Version 18 or higher
- **Bun**: Package manager and runtime
- **Git**: Version control system
- **TypeScript**: Version 5 or higher

### Development Setup

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd Backlog.md
   ```

2. **Install dependencies**:
   ```bash
   bun install
   ```

3. **Set up development environment**:
   ```bash
   bun run dev
   ```

### Project Structure

```
Backlog.md/
├── src/                    # Source code
│   ├── core/             # Core business logic
│   ├── commands/         # CLI commands
│   ├── mcp/              # MCP server
│   ├── ui/               # Terminal UI
│   ├── web/              # Web UI
│   ├── utils/            # Utilities
│   └── types/            # Type definitions
├── test/                 # Test files
├── docs/                 # Documentation
└── backlog/              # User data (git ignored)
```

## Development Workflow

### Code Style and Standards

#### TypeScript Configuration

- **Strict Mode**: Enabled for maximum type safety
- **Target**: ES2022 for modern JavaScript features
- **Module**: ESNext for modern module system
- **Declaration**: Generate declaration files

#### Code Formatting

- **Biome**: Primary formatter and linter
- **Indentation**: Tabs with 2-space width
- **Quotes**: Double quotes for strings
- **Semicolons**: Required at end of statements

#### Import Organization

```typescript
// Import order
import { afterEach, beforeEach, describe, expect, it } from "bun:test";
import { mkdir, rm } from "node:fs/promises";
import { join } from "node:path";
import { $ } from "bun";
import { Core } from "../core/backlog.ts";
import type { Task } from "../types/index.ts";
import { createUniqueTestDir, safeCleanup } from "./test-utils.ts";
```

### Testing

#### Test Structure

```typescript
// Standard test file structure
import { afterEach, beforeEach, describe, expect, it } from "bun:test";
import { mkdir, rm } from "node:fs/promises";
import { join } from "node:path";
import { $ } from "bun";
import { Core } from "../core/backlog.ts";
import { createUniqueTestDir, safeCleanup } from "./test-utils.ts";

let TEST_DIR: string;

describe("Feature name", () => {
  let core: Core;
  
  beforeEach(async () => {
    TEST_DIR = createUniqueTestDir("test-feature-name");
    await rm(TEST_DIR, { recursive: true, force: true }).catch(() => {});
    await mkdir(TEST_DIR, { recursive: true });
    
    await $`git init`.cwd(TEST_DIR).quiet();
    await $`git config user.name "Test User"`.cwd(TEST_DIR).quiet();
    await $`git config user.email test@example.com`.cwd(TEST_DIR).quiet();
    
    core = new Core(TEST_DIR);
    await core.initializeProject("Test Project Name");
  });
  
  afterEach(async () => {
    try {
      await safeCleanup(TEST_DIR);
    } catch {
      // Ignore cleanup errors
    }
  });
  
  it("should perform operation", async () => {
    // Test implementation
  });
});
```

#### Running Tests

```bash
# Run all tests
bun test

# Run specific test file
bun test test/core/backlog.test.ts

# Run tests with coverage
bun test --coverage

# Run tests in watch mode
bun test --watch
```

### Building and Linting

```bash
# Build the project
bun run build

# Run type checking
bun run check:types

# Run linting and formatting
bun run check

# Format code
bun run format
```

## Core Development

### Core Module Development

#### Core Class Structure

```typescript
// Core class pattern
export class Core {
  private projectPath: string;
  private config: ProjectConfig;
  private taskLoader: TaskLoader;
  private contentStore: ContentStore;
  
  constructor(projectPath: string) {
    this.projectPath = projectPath;
    this.taskLoader = new TaskLoader(projectPath);
    this.contentStore = new ContentStore(projectPath);
  }
  
  async initializeProject(name: string): Promise<void> {
    // Project initialization logic
  }
  
  async createTask(task: Task, autoCommit: boolean): Promise<string> {
    // Task creation logic
  }
  
  async updateTask(id: string, updates: Partial<Task>): Promise<void> {
    // Task update logic
  }
}
```

#### Task Management

```typescript
// Task management patterns
export class TaskManager {
  private core: Core;
  
  constructor(core: Core) {
    this.core = core;
  }
  
  async createTask(taskData: CreateTaskData): Promise<Task> {
    const task: Task = {
      id: generateTaskId(),
      title: taskData.title,
      status: taskData.status || "To Do",
      assignee: taskData.assignee || [],
      createdDate: new Date().toISOString().split('T')[0],
      labels: taskData.labels || [],
      dependencies: taskData.dependencies || [],
      body: taskData.body || "",
    };
    
    await this.core.createTask(task, taskData.autoCommit || false);
    return task;
  }
  
  async updateTask(id: string, updates: Partial<Task>): Promise<void> {
    await this.core.updateTask(id, updates);
  }
}
```

### MCP Tool Development

#### Tool Creation Pattern

```typescript
// MCP tool development pattern
export const createTaskTool = {
  name: "create-task",
  description: "Create a new task",
  inputSchema: {
    type: "object",
    properties: {
      title: {
        type: "string",
        description: "Task title",
        minLength: 1,
      },
      status: {
        type: "string",
        enum: ["To Do", "In Progress", "Done"],
        default: "To Do",
      },
      assignee: {
        type: "array",
        items: { type: "string" },
        default: [],
      },
      labels: {
        type: "array",
        items: { type: "string" },
        default: [],
      },
      body: {
        type: "string",
        default: "",
      },
    },
    required: ["title"],
  },
  handler: async (args: unknown) => {
    try {
      const validated = validateCreateTaskArgs(args);
      const core = new Core(process.cwd());
      const taskId = await core.createTask(validated, false);
      
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({ taskId, success: true }),
          },
        ],
      };
    } catch (error) {
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({
              error: "Failed to create task",
              message: error.message,
            }),
          },
        ],
        isError: true,
      };
    }
  },
};
```

#### Validation Patterns

```typescript
// Input validation patterns
import { z } from "zod";

const CreateTaskArgsSchema = z.object({
  title: z.string().min(1, "Title is required"),
  status: z.enum(["To Do", "In Progress", "Done"]).optional(),
  assignee: z.array(z.string()).optional(),
  labels: z.array(z.string()).optional(),
  body: z.string().optional(),
});

type CreateTaskArgs = z.infer<typeof CreateTaskArgsSchema>;

function validateCreateTaskArgs(args: unknown): CreateTaskArgs {
  return CreateTaskArgsSchema.parse(args);
}
```

### CLI Command Development

#### Command Structure

```typescript
// CLI command pattern
import { Command } from "commander";
import { Core } from "../core/backlog.ts";

export const createTaskCommand = new Command("create")
  .description("Create a new task")
  .argument("<title>", "Task title")
  .option("-s, --status <status>", "Task status", "To Do")
  .option("-a, --assignee <assignee...>", "Task assignees")
  .option("-l, --label <label...>", "Task labels")
  .option("-b, --body <body>", "Task body")
  .option("--no-auto-commit", "Disable auto-commit")
  .action(async (title, options) => {
    try {
      const core = new Core(process.cwd());
      const taskId = await core.createTask({
        title,
        status: options.status,
        assignee: options.assignee || [],
        labels: options.label || [],
        body: options.body || "",
      }, options.autoCommit);
      
      console.log(`Task created: ${taskId}`);
    } catch (error) {
      console.error("Error creating task:", error.message);
      process.exit(1);
    }
  });
```

### UI Component Development

#### Terminal UI Components

```typescript
// Terminal UI component pattern
import { Box, Text, useInput } from "ink";
import React, { useState, useEffect } from "react";

interface TaskListProps {
  tasks: Task[];
  onSelect: (task: Task) => void;
}

export const TaskList: React.FC<TaskListProps> = ({ tasks, onSelect }) => {
  const [selectedIndex, setSelectedIndex] = useState(0);
  
  useInput((input, key) => {
    if (key.upArrow) {
      setSelectedIndex(Math.max(0, selectedIndex - 1));
    } else if (key.downArrow) {
      setSelectedIndex(Math.min(tasks.length - 1, selectedIndex + 1));
    } else if (key.return) {
      onSelect(tasks[selectedIndex]);
    }
  });
  
  return (
    <Box flexDirection="column" padding={1}>
      {tasks.map((task, index) => (
        <Box key={task.id}>
          <Text color={index === selectedIndex ? "green" : "white"}>
            {index === selectedIndex ? "> " : "  "}
            {task.title} ({task.status})
          </Text>
        </Box>
      ))}
    </Box>
  );
};
```

#### Web UI Components

```typescript
// Web UI component pattern
import React, { useState, useEffect } from "react";
import { Task } from "../../types/index";

interface TaskCardProps {
  task: Task;
  onUpdate: (task: Task) => void;
  onDelete: (id: string) => void;
}

export const TaskCard: React.FC<TaskCardProps> = ({ task, onUpdate, onDelete }) => {
  const [isEditing, setIsEditing] = useState(false);
  const [editedTask, setEditedTask] = useState(task);
  
  const handleSave = () => {
    onUpdate(editedTask);
    setIsEditing(false);
  };
  
  if (isEditing) {
    return (
      <div className="task-card editing">
        <input
          type="text"
          value={editedTask.title}
          onChange={(e) => setEditedTask({ ...editedTask, title: e.target.value })}
        />
        <select
          value={editedTask.status}
          onChange={(e) => setEditedTask({ ...editedTask, status: e.target.value })}
        >
          <option value="To Do">To Do</option>
          <option value="In Progress">In Progress</option>
          <option value="Done">Done</option>
        </select>
        <button onClick={handleSave}>Save</button>
        <button onClick={() => setIsEditing(false)}>Cancel</button>
      </div>
    );
  }
  
  return (
    <div className="task-card">
      <h3>{task.title}</h3>
      <p>Status: {task.status}</p>
      <button onClick={() => setIsEditing(true)}>Edit</button>
      <button onClick={() => onDelete(task.id)}>Delete</button>
    </div>
  );
};
```

## Utility Development

### File System Utilities

```typescript
// File system utility patterns
import { mkdir, readFile, writeFile, rm } from "node:fs/promises";
import { join } from "node:path";

export class FileSystemUtils {
  static async ensureDirectory(dirPath: string): Promise<void> {
    try {
      await mkdir(dirPath, { recursive: true });
    } catch (error) {
      if (error.code !== "EEXIST") {
        throw error;
      }
    }
  }
  
  static async safeWriteFile(filePath: string, content: string): Promise<void> {
    const dirPath = join(filePath, "..");
    await this.ensureDirectory(dirPath);
    await writeFile(filePath, content, "utf-8");
  }
  
  static async safeReadFile(filePath: string): Promise<string> {
    try {
      return await readFile(filePath, "utf-8");
    } catch (error) {
      if (error.code === "ENOENT") {
        return "";
      }
      throw error;
    }
  }
  
  static async safeRemove(filePath: string): Promise<void> {
    try {
      await rm(filePath, { recursive: true, force: true });
    } catch (error) {
      if (error.code !== "ENOENT") {
        throw error;
      }
    }
  }
}
```

### Git Utilities

```typescript
// Git utility patterns
import { $ } from "bun";

export class GitUtils {
  static async isGitRepo(dirPath: string): Promise<boolean> {
    try {
      await $`git rev-parse --is-inside-work-tree`.cwd(dirPath).quiet();
      return true;
    } catch {
      return false;
    }
  }
  
  static async getCurrentBranch(dirPath: string): Promise<string> {
    const result = await $`git branch --show-current`.cwd(dirPath).quiet();
    return result.text().trim();
  }
  
  static async commitChanges(dirPath: string, message: string): Promise<void> {
    await $`git add .`.cwd(dirPath).quiet();
    await $`git commit -m ${message}`.cwd(dirPath).quiet();
  }
  
  static async hasUncommittedChanges(dirPath: string): Promise<boolean> {
    try {
      const result = await $`git status --porcelain`.cwd(dirPath).quiet();
      return result.text().trim().length > 0;
    } catch {
      return false;
    }
  }
}
```

### Validation Utilities

```typescript
// Validation utility patterns
import { z } from "zod";

export class ValidationUtils {
  static validateTaskId(id: string): boolean {
    return /^task-\d+$/.test(id);
  }
  
  static validateDocumentId(id: string): boolean {
    return /^doc-\d+$/.test(id);
  }
  
  static validateDecisionId(id: string): boolean {
    return /^decision-\d+$/.test(id);
  }
  
  static validateEmail(email: string): boolean {
    const emailSchema = z.string().email();
    return emailSchema.safeParse(email).success;
  }
  
  static validateDate(date: string): boolean {
    const dateSchema = z.string().regex(/^\d{4}-\d{2}-\d{2}$/);
    return dateSchema.safeParse(date).success;
  }
}
```

## Testing Utilities

### Test Data Generation

```typescript
// Test data generation patterns
import { Task, Document, Decision } from "../types/index";

export class TestDataGenerator {
  static generateTask(overrides: Partial<Task> = {}): Task {
    return {
      id: `task-${Date.now()}`,
      title: "Test Task",
      status: "To Do",
      assignee: [],
      createdDate: new Date().toISOString().split('T')[0],
      labels: ["test"],
      dependencies: [],
      body: "This is a test task",
      ...overrides,
    };
  }
  
  static generateDocument(overrides: Partial<Document> = {}): Document {
    return {
      id: `doc-${Date.now()}`,
      title: "Test Document",
      type: "guide",
      createdDate: new Date().toISOString().split('T')[0],
      content: "# Test Document\n\nThis is a test document.",
      ...overrides,
    };
  }
  
  static generateDecision(overrides: Partial<Decision> = {}): Decision {
    return {
      id: `decision-${Date.now()}`,
      title: "Test Decision",
      status: "accepted",
      createdDate: new Date().toISOString().split('T')[0],
      content: "# Test Decision\n\nThis is a test decision.",
      ...overrides,
    };
  }
}
```

### Test Environment Setup

```typescript
// Test environment setup patterns
import { mkdir, rm } from "node:fs/promises";
import { join } from "node:path";
import { $ } from "bun";

export class TestEnvironment {
  static async createTestDir(baseName: string): Promise<string> {
    const timestamp = Date.now();
    const random = Math.random().toString(36).substring(7);
    const testDir = join(process.cwd(), "test-temp", `${baseName}-${timestamp}-${random}`);
    
    await rm(testDir, { recursive: true, force: true }).catch(() => {});
    await mkdir(testDir, { recursive: true });
    
    return testDir;
  }
  
  static async setupGitRepo(dirPath: string): Promise<void> {
    await $`git init`.cwd(dirPath).quiet();
    await $`git config user.name "Test User"`.cwd(dirPath).quiet();
    await $`git config user.email test@example.com`.cwd(dirPath).quiet();
  }
  
  static async cleanupTestDir(dirPath: string): Promise<void> {
    try {
      await rm(dirPath, { recursive: true, force: true });
    } catch (error) {
      console.warn(`Failed to cleanup test directory: ${dirPath}`, error);
    }
  }
}
```

## Debugging

### Debug Mode

Enable debug mode for detailed logging:

```bash
# Enable debug mode
DEBUG=backlog:* bun run dev

# Enable specific debug namespaces
DEBUG=backlog:core,backlog:mcp bun run dev
```

### Common Debugging Scenarios

#### Task Loading Issues

```typescript
// Debug task loading
const core = new Core(projectPath);
console.log("Project path:", projectPath);
console.log("Config exists:", await core.configExists());
console.log("Tasks directory:", core.getTasksDirectory());
```

#### MCP Server Issues

```typescript
// Debug MCP server
const server = new MCPServer();
console.log("Server capabilities:", server.getCapabilities());
console.log("Available tools:", server.getTools());
console.log("Available resources:", server.getResources());
```

#### Git Integration Issues

```typescript
// Debug git integration
const gitUtils = new GitUtils();
console.log("Is git repo:", await gitUtils.isGitRepo(projectPath));
console.log("Current branch:", await gitUtils.getCurrentBranch(projectPath));
console.log("Has uncommitted changes:", await gitUtils.hasUncommittedChanges(projectPath));
```

## Performance Optimization

### File Operations

```typescript
// Optimized file operations
export class OptimizedFileOperations {
  private static cache = new Map<string, string>();
  
  static async readFileWithCache(filePath: string): Promise<string> {
    if (this.cache.has(filePath)) {
      return this.cache.get(filePath)!;
    }
    
    const content = await readFile(filePath, "utf-8");
    this.cache.set(filePath, content);
    return content;
  }
  
  static clearCache(): void {
    this.cache.clear();
  }
  
  static async batchReadFiles(filePaths: string[]): Promise<Map<string, string>> {
    const results = new Map<string, string>();
    
    await Promise.all(
      filePaths.map(async (filePath) => {
        const content = await this.readFileWithCache(filePath);
        results.set(filePath, content);
      })
    );
    
    return results;
  }
}
```

### Memory Management

```typescript
// Memory management patterns
export class MemoryManager {
  private static instances = new Map<string, any>();
  
  static getInstance<T>(key: string, factory: () => T): T {
    if (!this.instances.has(key)) {
      this.instances.set(key, factory());
    }
    return this.instances.get(key) as T;
  }
  
  static clearInstance(key: string): void {
    this.instances.delete(key);
  }
  
  static clearAllInstances(): void {
    this.instances.clear();
  }
}
```

## Contributing

### Pull Request Process

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/new-feature`
3. **Make changes and commit**: `git commit -m "Add new feature"`
4. **Push to branch**: `git push origin feature/new-feature`
5. **Create pull request**

### Code Review Guidelines

- **Code Quality**: Ensure code follows project standards
- **Test Coverage**: Maintain or improve test coverage
- **Documentation**: Update documentation as needed
- **Performance**: Consider performance implications
- **Security**: Review security implications

### Release Process

1. **Update version**: `npm version patch/minor/major`
2. **Build project**: `npm run build`
3. **Run tests**: `npm test`
4. **Publish**: `npm publish`
5. **Create release**: Create GitHub release with changelog

## Resources

### Documentation

- [Architecture Documentation](./architecture.md)
- [MCP Integration](./mcp-integration.md)
- [API Reference](./api-reference.md)
- [Testing Guide](./testing-guide.md)

### Tools and Libraries

- **Bun**: Package manager and runtime
- **TypeScript**: Type-safe JavaScript
- **Commander.js**: CLI framework
- **Ink**: Terminal UI framework
- **React**: Web UI framework
- **Zod**: Schema validation
- **Biome**: Formatter and linter

### Community

- **GitHub Issues**: Report bugs and request features
- **Discussions**: Ask questions and share ideas
- **Contributing Guide**: Learn how to contribute
- **Code of Conduct**: Community guidelines