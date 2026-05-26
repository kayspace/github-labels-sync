# GitHub Labels Sync

A minimal, browser-only utility to copy GitHub issue labels from one repository to another using the GitHub REST API.

No backend.

No CLI.

No installation.

Everything runs directly inside the browser.

---

## Features

- Copy labels between GitHub repositories
- Preview labels before syncing
- Skip or overwrite existing labels
- Live activity logging
- Progress tracking
- Browser-only execution
- Supports:
  - Classic PAT
  - Fine-Grained PAT
- Handles pagination automatically
- Zero dependencies
- Open source

---

## Why This Exists

Managing labels across multiple repositories becomes repetitive very quickly.

GitHub does not provide a native label synchronization tool.

This project solves that problem using a lightweight browser-based interface that directly communicates with the GitHub REST API.

---

## How It Works

The application follows this flow:

### 1. Authentication

The user enters a GitHub Personal Access Token.

---

### 2. Repository Input

The user enters:

- source repository
- target repository

Format:

```text
owner/repository
```

Example:

```text
facebook/react
```

---

### 3. Source Labels Fetch

The app fetches labels from the source repository.

---

### 4. Target Labels Fetch

The app fetches existing labels from the target repository.

---

### 5. Conflict Detection

Labels are compared case-insensitively.

Example:

```text
bug
BUG
Bug
```

are treated as the same label.

---

### 6. Sync Strategy

Depending on the selected mode:

#### Skip Existing

Existing labels remain untouched.

#### Overwrite Existing

Existing labels are updated.

---

### 7. Label Creation / Update

The app creates or updates labels using the GitHub REST API.

---

### 8. Live Activity Stream

The UI streams:
- progress
- logs
- sync statistics
- API responses

in real time.

---

## Architecture

This project intentionally keeps the architecture extremely simple.

Everything exists inside a single:

```text
index.html
```

file.

---

## Stack

| Layer | Technology |
|---|---|
| Structure | HTML |
| Styling | CSS |
| Logic | Vanilla JavaScript |
| API | GitHub REST API |
| Runtime | Browser |

---

## Application Flow

```text
Browser
   │
   │ fetch()
   ▼
GitHub REST API
```

---

## GitHub API Endpoints Used

### Fetch Labels

```http
GET /repos/{owner}/{repo}/labels
```

Used for:
- source repository labels
- target repository labels

Pagination:

```http
?per_page=100&page=n
```

---

### Create Label

```http
POST /repos/{owner}/{repo}/labels
```

Used when a label does not exist.

---

### Update Label

```http
PATCH /repos/{owner}/{repo}/labels/{name}
```

Used in overwrite mode.

---

## Authentication

The application supports:

- Classic PAT
- Fine-Grained PAT

---

### Classic PAT

Required scope:

```text
repo
```

---

### Fine-Grained PAT

#### Source Repository Permissions

- Contents → Read

#### Target Repository Permissions

- Issues → Read and Write

---

## Conflict Strategies

### Skip Existing

If a label already exists:
- do nothing
- preserve existing values

Safest mode.

---

### Overwrite Existing

If a label already exists:
- update color
- update description

Useful for keeping repositories consistent.

---

## What Gets Synced

| Property | Synced |
|---|---|
| Label Name | Yes |
| Label Color | Yes |
| Label Description | Yes |
| Issues Using Label | No |
| Pull Requests | No |

---

## Developer Flow

This section explains the internal logic for contributors.

---

## Core Functions

### `fetchAllLabels(repo, token)`

Responsible for fetching all labels from a repository.

#### Responsibilities

- Handles pagination
- Fetches labels in batches of 100
- Combines all pages
- Handles API errors

---

#### Internal Flow

```text
Start
  ↓
Fetch page 1
  ↓
Append labels
  ↓
More pages?
  ├── Yes → Fetch next page
  └── No  → Return labels
```

---

### `previewLabels()`

Used for rendering source label previews.

#### Responsibilities

- Validate input
- Fetch source labels
- Render label chips dynamically

---

#### Rendering

Labels are rendered using:

```js
document.createElement('span')
```

Text contrast is calculated dynamically for readability.

---

### `runSync()`

Main synchronization engine.

Controls the complete sync lifecycle.

---

#### Responsibilities

- Input validation
- Source label fetching
- Target label fetching
- Conflict detection
- Label creation
- Label updating
- Progress updates
- Logging
- Statistics tracking

---

## Sync Algorithm

### Step 1 — Fetch Source Labels

```js
const srcLabels = await fetchAllLabels(src, token);
```

---

### Step 2 — Fetch Target Labels

```js
const dstLabels = await fetchAllLabels(dst, token);
```

---

### Step 3 — Build Lookup Map

```js
const dstMap = new Map(
  dstLabels.map(l => [l.name.toLowerCase(), l])
);
```

Purpose:
- fast conflict lookup
- avoids repeated array scans

Complexity:

```text
O(n)
```

instead of:

```text
O(n²)
```

---

### Step 4 — Process Labels

```js
for (let i = 0; i < total; i++)
```

For each label:

#### If Label Exists

Depending on mode:

##### Skip Mode

```text
Skip label
```

##### Overwrite Mode

```http
PATCH existing label
```

---

#### If Label Does Not Exist

```http
POST new label
```

---

### Step 5 — Rate Limit Protection

Write requests include a small delay:

```js
await new Promise(r => setTimeout(r, 80));
```

Purpose:
- reduce secondary rate limit triggers
- avoid abuse throttling

---

## Logging System

Logs are streamed dynamically into the activity panel.

---

### Log Types

- info
- ok
- skip
- overwrite
- error

---

### Example

```text
12:10:33 info       Fetching source labels
12:10:35 ok         bug
12:10:35 overwrite  enhancement
12:10:36 skip       documentation
```

---

## Progress System

Progress updates continuously during sync.

Formula:

```js
5 + Math.round(((i + 1) / total) * 95)
```

---

### Why This Formula Exists

- first 5% reserved for initialization
- remaining 95% reserved for label processing

---

## UI Design Principles

The interface intentionally avoids unnecessary complexity.

---

### Design Goals

- low visual noise
- fast interaction
- readable logs
- zero framework dependency
- minimal maintenance surface

---

### Intentionally Avoided

- React
- Vue
- Node.js
- npm
- bundlers
- transpilers
- build systems

---

## Security Model

Important behavior:

- tokens never leave the browser except to GitHub
- no backend exists
- no analytics exist
- no database exists
- no cookies exist
- no persistent token storage exists

All API traffic flows directly:

```text
Browser → GitHub
```

---

## Local Development

This project is designed to be:
- dependency-free
- browser-native
- zero setup

---

### 1. Clone Repository

```bash
git clone git@github.com:kayspace/github-labels-sync.git
```

---

### 2. Enter Project Directory

```bash
cd github-labels-sync
```

---

### 3. Open Application

Open:

```text
index.html
```

inside your browser.

That is enough to run the project.

---

## Quick Start Options

### Option A — Direct Browser Open

Double click:

```text
index.html
```

The application launches immediately.

---

### Option B — VS Code Live Server (Recommended)

#### Install Extension

Search:

```text
Live Server
```

Author:

```text
Ritwick Dey
```

---

#### Launch

1. Open project folder in VS Code
2. Right click:

```text
index.html
```

3. Click:

```text
Open with Live Server
```

---

## Development Requirements

Required:

```text
Nothing
```

---

### Not Required

This project intentionally avoids:

- Node.js
- npm
- yarn
- vite
- webpack
- frameworks
- transpilers

---

## Browser Compatibility

Tested on:

- Chrome
- Edge
- Firefox
- Brave
- Arc

Safari should also work.

---

## Project Structure

```text
github-labels-sync/
│
├── index.html
├── README.md
├── CONTRIBUTING
├── CODE_OF_CONDUCT
└── LICENSE
```

---

## Making Changes

All logic exists inside:

```text
index.html
```

Including:
- UI
- styling
- GitHub API logic
- sync engine
- logging system
- version fetching

---

## Development Workflow

### Edit File

Modify:

```text
index.html
```

---

### Refresh Browser

Workflow:

```text
Save → Refresh Browser
```

No build step exists.

---

## Running a Test Sync

### 1. Generate GitHub Token

Open:

```text
https://github.com/settings/tokens/new
```

---

## Authentication Options

### Classic PAT

Required scope:

```text
repo
```

---

### Fine-Grained PAT

#### Source Repository

- Contents → Read

#### Target Repository

- Issues → Read and Write

---

## Repository Input Format

```text
owner/repository
```

Example:

```text
facebook/react
```

---

## Sync Steps

### 1. Enter Token

Paste GitHub Personal Access Token.

---

### 2. Enter Source Repository

Example:

```text
facebook/react
```

---

### 3. Enter Target Repository

Example:

```text
kayspace/test-repo
```

---

### 4. Preview Labels

Click:

```text
preview source labels
```

Verify:
- labels load correctly
- colors render correctly
- descriptions render correctly

---

### 5. Run Sync

Click:

```text
Run sync
```

---

## Monitor Activity

The UI provides:
- live logs
- progress tracking
- sync statistics
- overwrite visibility
- error reporting

---

## Recommended Git Workflow

### Create Feature Branch

```bash
git checkout -b feature/my-feature
```

---

### Stage Changes

```bash
git add .
```

---

### Commit Changes

```bash
git commit -m "add feature"
```

---

### Push Changes

```bash
git push origin feature/my-feature
```

---

## Versioning

This project follows:

```text
Semantic Versioning
```

Format:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
v1.0.0
```

---

### Version Meaning

| Type | Purpose |
|---|---|
| MAJOR | Breaking changes |
| MINOR | New features |
| PATCH | Bug fixes |

---

## Release Workflow

Releases should only be created for:
- stable checkpoints
- meaningful updates
- public versions

Not for every commit.

---

## Creating a Release

### 1. Commit Final Changes

```bash
git add .
git commit -m "release v1.0.0"
```

---

### 2. Push Changes

```bash
git push
```

---

### 3. Create Version Tag

```bash
git tag -a v1.0.0 -m "Initial stable release"
```

---

### 4. Push Tag

```bash
git push origin v1.0.0
```

---

### 5. Publish GitHub Release

Open:

```text
GitHub Repository → Releases → Create New Release
```

Select:

```text
Tag: v1.0.0
```

Then publish the release.

---

## Dynamic Version System

If enabled, the header version automatically syncs using:

```text
/releases/latest
```

Example:

```text
v1.0.0
→
v1.1.0
→
v1.2.0
```

No manual version updates required.


---

## Contribution Guidelines

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for detailed information on how to get started.

---

## Contact

For inquiries or collaborations about using the content or the code, reach out to me at kayzspace@outlook.com

## License

Licensed under the: [MIT License](LICENSE)
