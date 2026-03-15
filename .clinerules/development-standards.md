# Tauri Note App - Development Standards

## Project Overview
This is a Tauri v2 note-taking application built with React, TypeScript, and Rust. The project serves as a learning resource for Tauri development.

## Technology Stack
- **Frontend**: React 18+ with TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **Backend**: Rust (Tauri v2)
- **Storage**: JSON files in app data directory
- **Package Manager**: pnpm (preferred) or npm

## Naming Conventions

### Files and Directories
- React components: `PascalCase.tsx`
- TypeScript utilities: `camelCase.ts`
- Rust modules: `snake_case.rs`
- CSS/Style files: `kebab-case.css`

### Variables and Functions
- TypeScript: `camelCase`
- Rust: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Types/Interfaces: `PascalCase`

### Tauri Commands
- Rust function: `snake_case` (e.g., `get_all_notes`)
- Invoke call: `snake_case` string (e.g., `invoke('get_all_notes')`)

## Git Commit Standards

### Commit Message Format
```
<type>(<scope>): <description>

[optional body]
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples
```
feat(notes): add create note functionality
fix(storage): handle file not found error
docs(readme): update installation instructions
refactor(commands): extract note validation logic
```

## Testing Guidelines

### Test-Driven Development (TDD) Approach
This project follows TDD methodology:
1. **Red**: Write a failing test first
2. **Green**: Write minimal code to make the test pass
3. **Refactor**: Improve the code while keeping tests green

### TDD Workflow with Cline
- Before implementing any feature, write tests first
- When Cline needs to update or modify test cases, control will be returned to the user with an explanation of:
  - What test changes are proposed
  - Why the changes are necessary
  - The expected behavior being tested
- User approval is required before modifying any existing test cases

### Frontend Testing
- Test components with React Testing Library
- Test hooks in isolation
- Mock Tauri invoke calls in tests
- Write tests before implementing components

### Backend Testing (Rust)
- Unit test business logic
- Integration test Tauri commands
- Test file operations with temp directories
- Write tests before implementing commands