# Backlog.md Architecture Documentation

## Overview

Backlog.md is a comprehensive task management system that combines CLI, TUI (Terminal User Interface), and web UI interfaces with AI agent integration through MCP (Model Context Protocol). The system manages tasks, documents, and decisions using structured markdown files with frontmatter.

## Core Architecture

### System Components

```
Backlog.md/
├── src/                    # Main source code
│   ├── core/             # Core business logic
│   ├── commands/         # CLI command implementations
│   ├── mcp/              # MCP server and tools
│   ├── ui/               # Terminal UI components
│   ├── web/              # Web UI components
│   ├── utils/            # Utility functions
│   └── types/            # TypeScript type definitions
├── backlog/              # User data directory
│   ├── tasks/            # Task files
│   ├── docs/             # Documentation files
│   ├── decisions/        # Decision records
│   └── archive/          # Archived content
└── docs/                 # Project documentation
```

### Core Modules

#### Core System (`src/core/`)

- **`backlog.ts`**: Main Core class that orchestrates all operations
- **`task-loader.ts`**: Handles loading and parsing task files
- **`content-store.ts`**: Manages content storage and retrieval
- **`sequences.ts`**: Implements task sequences and dependencies
- **`cross-branch-tasks.ts`**: Manages tasks across git branches
- **`remote-tasks.ts`**: Handles remote task operations
- **`search-service.ts`**: Provides search functionality
- **`reorder.ts`**: Handles task reordering operations

#### MCP Integration (`src/mcp/`)

- **`server.ts`**: MCP server implementation
- **`tools/`**: MCP tools for different operations
  - `tasks/`: Task-related MCP tools
  - `documents/`: Document-related MCP tools
  - `decisions/`: Decision-related MCP tools
- **`resources/`**: MCP resource definitions
- **`utils/`**: MCP utility functions
- **`validation/`**: Input validation for MCP operations

#### CLI Commands (`src/commands/`)

- **`mcp.ts`**: MCP server management commands
- **`overview.ts`**: Project overview commands
- **`completion.ts`**: Shell completion setup
- **`configure-advanced-settings.ts`**: Advanced configuration
- **`advanced-config-wizard.ts`**: Configuration wizard

### Data Models

#### Task Structure
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

#### Document Structure
```typescript
interface Document {
  id: string;
  title: string;
  type: string;
  createdDate: string;
  content: string;
}
```

#### Decision Structure
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

## Key Features

### 1. Multi-Interface Support

- **CLI**: Command-line interface for all operations
- **TUI**: Terminal-based interactive interface
- **Web UI**: Browser-based interface with React components

### 2. MCP Integration

- **AI Agent Support**: Full MCP server implementation
- **Tool-based Architecture**: Extensible tool system for AI agents
- **Resource Management**: Structured resource access for agents
- **Validation**: Comprehensive input validation

### 3. Task Management

- **Sequences**: Task sequences with dependencies
- **Cross-branch**: Tasks spanning multiple git branches
- **Remote Operations**: Synchronization with remote repositories
- **Search**: Full-text search across all content

### 4. Content Management

- **Structured Markdown**: All content uses markdown with frontmatter
- **File Organization**: Hierarchical organization with archives
- **Version Control**: Git integration for change tracking
- **Templates**: Reusable templates for common content types

## Workflow Integration

### Git Integration

- **Auto-commit**: Automatic commits for task operations
- **Branch Management**: Task tracking across branches
- **Remote Sync**: Synchronization with remote repositories
- **Conflict Resolution**: Handling merge conflicts

### AI Agent Workflow

1. **Agent Connection**: AI agents connect via MCP protocol
2. **Tool Access**: Agents access structured tools for operations
3. **Resource Query**: Agents query project resources
4. **Operation Execution**: Agents execute operations through tools
5. **Result Feedback**: Agents receive structured results

### Development Workflow

1. **Project Setup**: Initialize with `backlog init`
2. **Task Creation**: Create tasks with `backlog task create`
3. **Development**: Work on tasks with AI agent assistance
4. **Documentation**: Document decisions and processes
5. **Review**: Use board views for progress tracking
6. **Completion**: Mark tasks done and archive

## Configuration

### Project Configuration (`config.yml`)

```yaml
project_name: "Project Name"
default_assignee: "user@example.com"
default_status: "To Do"
statuses: ["To Do", "In Progress", "Done"]
labels: ["bug", "feature", "documentation"]
milestones: ["v1.0", "v1.1", "v2.0"]
date_format: "yyyy-mm-dd"
```

### Advanced Configuration

- **MCP Settings**: Server configuration and tool enablement
- **UI Settings**: Interface customization options
- **Git Settings**: Repository and branch configuration
- **Integration Settings**: External tool integrations

## Performance Considerations

### File Operations

- **Lazy Loading**: Content loaded on demand
- **Caching**: Intelligent caching for frequently accessed content
- **Batch Operations**: Efficient batch processing for multiple operations
- **Indexing**: Search indexing for fast content retrieval

### Memory Management

- **Stream Processing**: Large files processed in streams
- **Garbage Collection**: Proper cleanup of temporary resources
- **Memory Limits**: Configurable memory limits for operations

## Security

### Data Protection

- **Local Storage**: All data stored locally by default
- **Access Control**: File system permissions for access control
- **Input Validation**: Comprehensive validation for all inputs
- **Safe Operations**: Atomic operations for data integrity

### MCP Security

- **Connection Validation**: Secure connection establishment
- **Tool Authorization**: Authorization checks for tool access
- **Resource Protection**: Protected resource access
- **Audit Logging**: Operation logging for security auditing

## Extensibility

### Plugin Architecture

- **Tool Plugins**: Custom MCP tools
- **UI Plugins**: Custom UI components
- **Integration Plugins**: External system integrations
- **Format Plugins**: Custom content format support

### Customization Points

- **Templates**: Custom templates for content creation
- **Workflows**: Custom workflow definitions
- **Validators**: Custom validation rules
- **Formatters**: Custom output formatting

## Testing Strategy

### Test Organization

- **Unit Tests**: Individual component testing
- **Integration Tests**: Component interaction testing
- **Acceptance Tests**: End-to-end workflow testing
- **Performance Tests**: Performance and load testing

### Test Patterns

- **Unique Directories**: Isolated test environments
- **Consistent Setup**: Standardized test setup patterns
- **Cleanup Management**: Proper cleanup and resource management
- **Mock Data**: Realistic test data generation

## Deployment

### Distribution

- **NPM Package**: Distributed as npm package
- **Binary Distribution**: Standalone binary options
- **Docker Support**: Containerized deployment options
- **Installation Scripts**: Automated installation and setup

### Version Management

- **Semantic Versioning**: Follows semantic versioning
- **Backward Compatibility**: Maintains backward compatibility
- **Migration Guides**: Comprehensive migration documentation
- **Deprecation Policy**: Clear deprecation timelines

## Future Roadmap

### Planned Features

- **Enhanced AI Integration**: More sophisticated AI agent capabilities
- **Real-time Collaboration**: Multi-user collaboration features
- **Advanced Analytics**: Project analytics and reporting
- **Mobile Support**: Mobile application development

### Architecture Evolution

- **Microservices**: Potential microservices architecture
- **Cloud Integration**: Cloud storage and synchronization
- **API First**: RESTful API development
- **Plugin Marketplace**: Plugin distribution platform