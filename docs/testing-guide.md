# Testing Guide

## Overview

This guide provides comprehensive patterns and best practices for testing the Backlog.md codebase. The project uses Bun's built-in test runner with a focus on maintainable, reliable tests.

## Testing Philosophy

### Principles

1. **Isolation**: Each test should be independent and isolated
2. **Determinism**: Tests should produce consistent results
3. **Clarity**: Tests should be easy to understand and maintain
4. **Coverage**: Tests should cover all critical functionality
5. **Performance**: Tests should run quickly and efficiently

### Test Organization

```
test/
├── core/                 # Core module tests
│   ├── backlog.test.ts
│   ├── task-loader.test.ts
│   └── content-store.test.ts
├── commands/             # CLI command tests
│   ├── mcp.test.ts
│   └── overview.test.ts
├── mcp/                  # MCP server tests
│   ├── server.test.ts
│   └── tools/
│       ├── tasks.test.ts
│       └── documents.test.ts
├── utils/                # Utility tests
│   ├── file-system.test.ts
│   └── validation.test.ts
└── integration/          # Integration tests
    ├── workflow.test.ts
    └── end-to-end.test.ts
```

## Test Structure

### Standard Test File Structure

```typescript
// Import organization
import { afterEach, beforeEach, describe, expect, it } from "bun:test";
import { mkdir, rm } from "node:fs/promises";
import { join } from "node:path";
import { $ } from "bun";
import { Core } from "../src/core/backlog.ts";
import type { Task } from "../src/types/index.ts";
import { createUniqueTestDir, safeCleanup } from "./test-utils.ts";

// Global test directory
let TEST_DIR: string;

// Main test suite
describe("Core functionality", () => {
  let core: Core;
  
  // Setup before each test
  beforeEach(async () => {
    // Create unique test directory
    TEST_DIR = createUniqueTestDir("test-core-functionality");
    
    // Clean up any existing directory
    await rm(TEST_DIR, { recursive: true, force: true }).catch(() => {});
    
    // Create test directory
    await mkdir(TEST_DIR, { recursive: true });
    
    // Initialize git repository if needed
    await $`git init`.cwd(TEST_DIR).quiet();
    await $`git config user.name "Test User"`.cwd(TEST_DIR).quiet();
    await $`git config user.email test@example.com`.cwd(TEST_DIR).quiet();
    
    // Initialize core
    core = new Core(TEST_DIR);
    await core.initializeProject("Test Project");
  });
  
  // Cleanup after each test
  afterEach(async () => {
    try {
      await safeCleanup(TEST_DIR);
    } catch {
      // Ignore cleanup errors - unique directories prevent conflicts
    }
  });
  
  // Test cases
  describe("task creation", () => {
    it("should create task with valid input", async () => {
      const taskData = {
        id: "task-1",
        title: "Test Task",
        status: "To Do",
        assignee: [],
        createdDate: "2025-01-01",
        labels: [],
        dependencies: [],
        body: "Test task description"
      };
      
      const taskId = await core.createTask(taskData, false);
      
      expect(taskId).toBe("task-1");
      
      // Verify task was created
      const task = await core.getTask("task-1");
      expect(task).toBeTruthy();
      expect(task!.title).toBe("Test Task");
    });
    
    it("should validate required fields", async () => {
      const invalidTask = {
        id: "task-2",
        title: "", // Invalid: empty title
        status: "To Do",
        assignee: [],
        createdDate: "2025-01-01",
        labels: [],
        dependencies: [],
        body: ""
      };
      
      await expect(core.createTask(invalidTask, false)).rejects.toThrow();
    });
  });
});
```

### Test Utilities

#### Test Directory Management

```typescript
// test-utils.ts
import { mkdir, rm } from "node:fs/promises";
import { join } from "node:path";

export function createUniqueTestDir(baseName: string): string {
  const timestamp = Date.now();
  const random = Math.random().toString(36).substring(7);
  return join(process.cwd(), "test-temp", `${baseName}-${timestamp}-${random}`);
}

export async function safeCleanup(dirPath: string): Promise<void> {
  try {
    await rm(dirPath, { recursive: true, force: true });
  } catch (error) {
    // Log warning but don't fail the test
    console.warn(`Failed to cleanup test directory: ${dirPath}`, error);
  }
}

export async function setupTestDirectory(baseName: string): Promise<string> {
  const testDir = createUniqueTestDir(baseName);
  
  // Clean up any existing directory
  await rm(testDir, { recursive: true, force: true }).catch(() => {});
  
  // Create test directory
  await mkdir(testDir, { recursive: true });
  
  return testDir;
}
```

#### Test Data Generation

```typescript
// test-data.ts
import { Task, Document, Decision } from "../src/types/index.ts";

export class TestDataFactory {
  static createTask(overrides: Partial<Task> = {}): Task {
    return {
      id: `task-${Date.now()}`,
      title: "Test Task",
      status: "To Do",
      assignee: [],
      createdDate: "2025-01-01",
      labels: ["test"],
      dependencies: [],
      body: "This is a test task",
      ...overrides,
    };
  }
  
  static createDocument(overrides: Partial<Document> = {}): Document {
    return {
      id: `doc-${Date.now()}`,
      title: "Test Document",
      type: "guide",
      createdDate: "2025-01-01",
      content: "# Test Document\n\nThis is a test document.",
      ...overrides,
    };
  }
  
  static createDecision(overrides: Partial<Decision> = {}): Decision {
    return {
      id: `decision-${Date.now()}`,
      title: "Test Decision",
      status: "accepted",
      createdDate: "2025-01-01",
      content: "# Test Decision\n\nThis is a test decision.",
      ...overrides,
    };
  }
  
  static createTaskList(count: number): Task[] {
    return Array.from({ length: count }, (_, index) => 
      this.createTask({
        id: `task-${index + 1}`,
        title: `Test Task ${index + 1}`,
        status: index % 3 === 0 ? "Done" : "To Do",
      })
    );
  }
}
```

#### Mock Helpers

```typescript
// test-mocks.ts
import { vi } from "vitest";

export function createMockCore() {
  return {
    createTask: vi.fn(),
    updateTask: vi.fn(),
    deleteTask: vi.fn(),
    getTask: vi.fn(),
    listTasks: vi.fn(),
    createDocument: vi.fn(),
    updateDocument: vi.fn(),
    deleteDocument: vi.fn(),
    getDocument: vi.fn(),
    listDocuments: vi.fn(),
  };
}

export function createMockFileSystem() {
  return {
    readFile: vi.fn(),
    writeFile: vi.fn(),
    mkdir: vi.fn(),
    rm: vi.fn(),
    exists: vi.fn(),
  };
}

export function createMockGit() {
  return {
    init: vi.fn(),
    add: vi.fn(),
    commit: vi.fn(),
    status: vi.fn(),
    branch: vi.fn(),
  };
}
```

## Testing Patterns

### Unit Testing

#### Core Functionality

```typescript
// core/backlog.test.ts
describe("Core", () => {
  let core: Core;
  let testDir: string;
  
  beforeEach(async () => {
    testDir = await setupTestDirectory("test-core");
    core = new Core(testDir);
    await core.initializeProject("Test Project");
  });
  
  afterEach(async () => {
    await safeCleanup(testDir);
  });
  
  describe("createTask", () => {
    it("should create task with valid data", async () => {
      const task = TestDataFactory.createTask();
      
      const taskId = await core.createTask(task, false);
      
      expect(taskId).toBe(task.id);
      
      const createdTask = await core.getTask(task.id);
      expect(createdTask).toEqual(task);
    });
    
    it("should throw error for invalid task data", async () => {
      const invalidTask = TestDataFactory.createTask({ title: "" });
      
      await expect(core.createTask(invalidTask, false)).rejects.toThrow();
    });
    
    it("should auto-commit when enabled", async () => {
      const task = TestDataFactory.createTask();
      
      await core.createTask(task, true);
      
      // Verify commit was made
      const result = await $`git log --oneline`.cwd(testDir).quiet();
      expect(result.text().trim()).toContain("Create task");
    });
  });
});
```

#### Utility Functions

```typescript
// utils/validation.test.ts
import { validateTaskId, validateEmail, validateDate } from "../src/utils/validation";

describe("Validation Utils", () => {
  describe("validateTaskId", () => {
    it("should return true for valid task IDs", () => {
      expect(validateTaskId("task-1")).toBe(true);
      expect(validateTaskId("task-123")).toBe(true);
      expect(validateTaskId("task-999")).toBe(true);
    });
    
    it("should return false for invalid task IDs", () => {
      expect(validateTaskId("task")).toBe(false);
      expect(validateTaskId("task-")).toBe(false);
      expect(validateTaskId("doc-1")).toBe(false);
      expect(validateTaskId("")).toBe(false);
    });
  });
  
  describe("validateEmail", () => {
    it("should return true for valid email addresses", () => {
      expect(validateEmail("user@example.com")).toBe(true);
      expect(validateEmail("test.user+tag@example.co.uk")).toBe(true);
    });
    
    it("should return false for invalid email addresses", () => {
      expect(validateEmail("invalid-email")).toBe(false);
      expect(validateEmail("@example.com")).toBe(false);
      expect(validateEmail("user@")).toBe(false);
    });
  });
  
  describe("validateDate", () => {
    it("should return true for valid dates", () => {
      expect(validateDate("2025-01-01")).toBe(true);
      expect(validateDate("2025-12-31")).toBe(true);
    });
    
    it("should return false for invalid dates", () => {
      expect(validateDate("2025-13-01")).toBe(false);
      expect(validateDate("2025-01-32")).toBe(false);
      expect(validateDate("01-01-2025")).toBe(false);
    });
  });
});
```

### Integration Testing

#### MCP Server Integration

```typescript
// mcp/server.test.ts
import { MCPServer } from "../src/mcp/server";
import { Core } from "../src/core/backlog";

describe("MCP Server", () => {
  let server: MCPServer;
  let core: Core;
  let testDir: string;
  
  beforeEach(async () => {
    testDir = await setupTestDirectory("test-mcp-server");
    core = new Core(testDir);
    await core.initializeProject("Test Project");
    server = new MCPServer(core);
  });
  
  afterEach(async () => {
    await safeCleanup(testDir);
  });
  
  describe("tool handling", () => {
    it("should handle create-task tool", async () => {
      const toolArgs = {
        title: "Test Task",
        status: "To Do",
        assignee: [],
        labels: [],
        body: "Test task description"
      };
      
      const result = await server.handleToolCall("create-task", toolArgs);
      
      expect(result.isError).toBe(false);
      expect(result.content[0].text).toContain("taskId");
    });
    
    it("should validate tool arguments", async () => {
      const invalidArgs = {
        title: "", // Invalid: empty title
        status: "To Do"
      };
      
      const result = await server.handleToolCall("create-task", invalidArgs);
      
      expect(result.isError).toBe(true);
      expect(result.content[0].text).toContain("Validation error");
    });
  });
  
  describe("resource handling", () => {
    it("should handle tasks://list resource", async () => {
      // Create some test tasks
      await core.createTask(TestDataFactory.createTask(), false);
      await core.createTask(TestDataFactory.createTask(), false);
      
      const result = await server.handleResourceRequest("tasks://list");
      
      expect(result.contents).toHaveLength(1);
      const data = JSON.parse(result.contents[0].text);
      expect(data.tasks).toHaveLength(2);
    });
  });
});
```

#### CLI Integration

```typescript
// commands/mcp.test.ts
import { $ } from "bun";
import { setupTestDirectory, safeCleanup } from "../test-utils";

describe("MCP Command", () => {
  let testDir: string;
  
  beforeEach(async () => {
    testDir = await setupTestDirectory("test-mcp-command");
    process.chdir(testDir);
  });
  
  afterEach(async () => {
    await safeCleanup(testDir);
    process.chdir(process.cwd());
  });
  
  it("should start MCP server", async () => {
    const result = await $`bun run src/index.ts mcp server`.timeout(5000).quiet();
    
    // Server should start without errors
    expect(result.exitCode).toBe(0);
  });
  
  it("should list available tools", async () => {
    const result = await $`bun run src/index.ts mcp list-tools`.quiet();
    
    expect(result.exitCode).toBe(0);
    expect(result.text()).toContain("create-task");
    expect(result.text()).toContain("update-task");
    expect(result.text()).toContain("delete-task");
  });
});
```

### End-to-End Testing

#### Complete Workflow

```typescript
// integration/workflow.test.ts
import { $ } from "bun";
import { setupTestDirectory, safeCleanup } from "../test-utils";

describe("End-to-End Workflow", () => {
  let testDir: string;
  
  beforeEach(async () => {
    testDir = await setupTestDirectory("test-e2e-workflow");
    process.chdir(testDir);
    
    // Initialize project
    await $`bun run src/index.ts init "Test Project"`.quiet();
  });
  
  afterEach(async () => {
    await safeCleanup(testDir);
    process.chdir(process.cwd());
  });
  
  it("should complete full task workflow", async () => {
    // Create task
    await $`bun run src/index.ts task create "Implement feature" --status "To Do" --label "feature"`.quiet();
    
    // List tasks
    const listResult = await $`bun run src/index.ts task list`.quiet();
    expect(listResult.text()).toContain("Implement feature");
    
    // Update task
    await $`bun run src/index.ts task update task-1 --status "In Progress" --assignee "dev@example.com"`.quiet();
    
    // Verify update
    const taskResult = await $`bun run src/index.ts task show task-1`.quiet();
    expect(taskResult.text()).toContain("In Progress");
    expect(taskResult.text()).toContain("dev@example.com");
    
    // Complete task
    await $`bun run src/index.ts task update task-1 --status "Done"`.quiet();
    
    // Verify completion
    const finalResult = await $`bun run src/index.ts task show task-1`.quiet();
    expect(finalResult.text()).toContain("Done");
  });
  
  it("should handle document workflow", async () => {
    // Create document
    await $`bun run src/index.ts doc create "Architecture Guide" --type "guide"`.quiet();
    
    // List documents
    const listResult = await $`bun run src/index.ts doc list`.quiet();
    expect(listResult.text()).toContain("Architecture Guide");
    
    // Update document
    await $`bun run src/index.ts doc update doc-1 --content "# Updated Architecture Guide\n\nThis is updated."`.quiet();
    
    // Verify update
    const docResult = await $`bun run src/index.ts doc show doc-1`.quiet();
    expect(docResult.text()).toContain("Updated Architecture Guide");
  });
});
```

## Testing Best Practices

### Test Naming

#### Use Clear, Descriptive Names

```typescript
// Good
it("should create task with valid input", async () => {
  // Test implementation
});

it("should throw error when task title is empty", async () => {
  // Test implementation
});

it("should auto-commit changes when autoCommit is true", async () => {
  // Test implementation
});

// Bad
it("test task creation", async () => {
  // Test implementation
});

it("error handling", async () => {
  // Test implementation
});

it("commit functionality", async () => {
  // Test implementation
});
```

### Test Organization

#### Group Related Tests

```typescript
describe("Task Management", () => {
  describe("createTask", () => {
    it("should create task with valid input", async () => {
      // Test implementation
    });
    
    it("should validate required fields", async () => {
      // Test implementation
    });
    
    it("should generate ID when not provided", async () => {
      // Test implementation
    });
  });
  
  describe("updateTask", () => {
    it("should update existing task", async () => {
      // Test implementation
    });
    
    it("should throw error when task not found", async () => {
      // Test implementation
    });
  });
});
```

### Test Data Management

#### Use Factories for Test Data

```typescript
// Good: Use factory pattern
const task = TestDataFactory.createTask({
  title: "Specific Test Task",
  status: "In Progress"
});

// Bad: Hardcoded data
const task = {
  id: "task-1",
  title: "Specific Test Task",
  status: "In Progress",
  assignee: [],
  createdDate: "2025-01-01",
  labels: [],
  dependencies: [],
  body: ""
};
```

#### Isolate Test Data

```typescript
// Good: Each test has its own data
it("should handle task with dependencies", async () => {
  const task1 = TestDataFactory.createTask({ id: "task-1" });
  const task2 = TestDataFactory.createTask({ 
    id: "task-2",
    dependencies: ["task-1"]
  });
  
  await core.createTask(task1, false);
  await core.createTask(task2, false);
  
  // Test implementation
});

// Bad: Shared data between tests
let sharedTask: Task;

beforeEach(async () => {
  sharedTask = TestDataFactory.createTask();
});

it("should handle task dependencies", async () => {
  // Uses sharedTask - can cause test interference
});
```

### Error Handling

#### Test Error Scenarios

```typescript
describe("error handling", () => {
  it("should throw ValidationError for empty title", async () => {
    const invalidTask = TestDataFactory.createTask({ title: "" });
    
    await expect(core.createTask(invalidTask, false))
      .rejects
      .toThrow(ValidationError);
  });
  
  it("should throw NotFoundError for non-existent task", async () => {
    await expect(core.getTask("non-existent-task"))
      .rejects
      .toThrow(NotFoundError);
  });
  
  it("should throw FileSystemError for invalid directory", async () => {
    const invalidCore = new Core("/invalid/directory");
    
    await expect(invalidCore.initializeProject("Test"))
      .rejects
      .toThrow(FileSystemError);
  });
});
```

### Async Testing

#### Handle Async Operations Properly

```typescript
// Good: Proper async handling
it("should create task asynchronously", async () => {
  const task = TestDataFactory.createTask();
  
  const taskId = await core.createTask(task, false);
  
  expect(taskId).toBe(task.id);
  
  const createdTask = await core.getTask(task.id);
  expect(createdTask).toEqual(task);
});

// Bad: Missing await
it("should create task asynchronously", () => {
  const task = TestDataFactory.createTask();
  
  core.createTask(task, false); // Missing await
  
  expect(true).toBe(true); // Test might pass before task is created
});
```

### Mocking and Stubbing

#### Use Mocks for External Dependencies

```typescript
// Good: Mock external dependencies
import { vi } from "vitest";

describe("with mocked file system", () => {
  let mockFs: ReturnType<typeof createMockFileSystem>;
  
  beforeEach(() => {
    mockFs = createMockFileSystem();
    vi.mock("node:fs/promises", () => ({
      readFile: mockFs.readFile,
      writeFile: mockFs.writeFile,
      mkdir: mockFs.mkdir,
      rm: mockFs.rm,
    }));
  });
  
  it("should handle file read errors", async () => {
    mockFs.readFile.mockRejectedValue(new Error("File not found"));
    
    await expect(core.loadTask("task-1"))
      .rejects
      .toThrow("File not found");
  });
});
```

### Performance Testing

#### Test Performance-Critical Operations

```typescript
describe("performance", () => {
  it("should handle large task lists efficiently", async () => {
    // Create many tasks
    const tasks = TestDataFactory.createTaskList(1000);
    
    const startTime = Date.now();
    
    for (const task of tasks) {
      await core.createTask(task, false);
    }
    
    const endTime = Date.now();
    const duration = endTime - startTime;
    
    expect(duration).toBeLessThan(5000); // Should complete in under 5 seconds
    
    // Test listing performance
    const listStart = Date.now();
    const taskList = await core.listTasks();
    const listEnd = Date.now();
    
    expect(taskList).toHaveLength(1000);
    expect(listEnd - listStart).toBeLessThan(1000); // Should list in under 1 second
  });
});
```

## Running Tests

### Basic Test Execution

```bash
# Run all tests
bun test

# Run specific test file
bun test test/core/backlog.test.ts

# Run tests with coverage
bun test --coverage

# Run tests in watch mode
bun test --watch

# Run tests with verbose output
bun test --verbose
```

### Filtering Tests

```bash
# Run tests matching a pattern
bun test --test-name-pattern "should create task"

# Run tests in specific directory
bun test test/core/

# Run tests with specific tag
bun test --test-grep "@integration"
```

### Debug Mode

```bash
# Run tests with debug output
DEBUG=backlog:* bun test

# Run tests with specific debug namespace
DEBUG=backlog:core bun test

# Run tests with verbose debugging
bun test --debug
```

## Test Coverage

### Coverage Goals

- **Line Coverage**: 90% minimum
- **Branch Coverage**: 85% minimum
- **Function Coverage**: 95% minimum

### Coverage Reports

```bash
# Generate coverage report
bun test --coverage

# Generate HTML coverage report
bun test --coverage --coverage-reporter=html

# Generate coverage report with thresholds
bun test --coverage --coverage-thresholds '{"lines":90,"branches":85,"functions":95}'
```

### Coverage Exclusions

```typescript
// Exclude from coverage
/* istanbul ignore next */
function debugFunction() {
  // Debug code that doesn't need coverage
}

/* istanbul ignore if */
if (process.env.NODE_ENV === 'development') {
  // Development-only code
}
```

## Continuous Integration

### CI Configuration

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Bun
      uses: oven-sh/setup-bun@v1
    
    - name: Install dependencies
      run: bun install
    
    - name: Run tests
      run: bun test --coverage
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

### Test Matrix

```yaml
# Test multiple Node.js versions
strategy:
  matrix:
    node-version: [18, 20, 22]
    
steps:
- name: Use Node.js ${{ matrix.node-version }}
  uses: actions/setup-node@v3
  with:
    node-version: ${{ matrix.node-version }}
```

## Troubleshooting

### Common Issues

#### Test Timeout

```typescript
// Increase timeout for slow tests
it("should handle large file operations", async () => {
  // Test implementation
}, 10000); // 10 second timeout
```

#### Test Isolation

```typescript
// Ensure proper cleanup
afterEach(async () => {
  try {
    await safeCleanup(TEST_DIR);
  } catch (error) {
    console.warn("Cleanup failed:", error);
  }
});
```

#### Flaky Tests

```typescript
// Add retry for flaky tests
it("should handle race conditions", async () => {
  // Test implementation
}, {
  retry: 3,
  timeout: 5000
});
```

### Debugging Tests

```bash
# Run tests with debug output
DEBUG=backlog:* bun test --verbose

# Run specific test with debug
bun test test/core/backlog.test.ts --test-name-pattern "should create task" --debug

# Run tests with inspector
bun --inspect test/core/backlog.test.ts
```

## Test Documentation

### Document Test Cases

```typescript
/**
 * Test: Task Creation
 * 
 * Description: Verify that tasks can be created with valid input data
 * 
 * Steps:
 * 1. Create test task with valid data
 * 2. Call createTask method
 * 3. Verify task was created successfully
 * 4. Verify task data is correct
 * 
 * Expected: Task should be created with correct data
 */
it("should create task with valid input", async () => {
  // Test implementation
});
```

### Test Matrix

| Test Case | Description | Input | Expected Output |
|-----------|-------------|-------|-----------------|
| Valid Task | Create task with valid data | Valid task object | Task created successfully |
| Empty Title | Create task with empty title | Task with empty title | ValidationError thrown |
| Invalid Status | Create task with invalid status | Task with invalid status | ValidationError thrown |
| Auto-commit | Create task with auto-commit enabled | Task with autoCommit=true | Task created and committed |

This comprehensive testing guide provides patterns, best practices, and examples for testing all aspects of the Backlog.md codebase. Following these guidelines will ensure maintainable, reliable tests that provide good coverage of the application functionality.