# OI OS Integration Guide for OI-stackoverflow-read-write-mcp

This guide provides complete instructions for AI agents to install, configure, and use the OI-stackoverflow-read-write-mcp server in OI OS (Brain Trust 4).

## 🚀 Installation

### Prerequisites

| Requirement | Version        |
| ----------- | -------------- |
| **Node.js** | 18.x or higher |
| **npm**     | Latest         |
| **Git**     | Any            |

### Installation Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/OI-OS/OI-stackoverflow-read-write-mcp.git
   ```

2. **Navigate to the server directory:**
   ```bash
   cd MCP-servers/OI-stackoverflow-read-write-mcp
   ```

3. **Install dependencies:**
   ```bash
   npm install
   ```

4. **Build the project:**
   ```bash
   npm run build
   ```

5. **Connect the server to OI OS:**
   ```bash
   cd ../../ # Go back to the OI OS root directory
   ./brain-trust4 connect OI-stackoverflow-read-write-mcp node -- "$(pwd)/MCP-servers/OI-stackoverflow-read-write-mcp/build/index.js"
   ```

## 🔧 Configuration

### Stack Overflow API Authentication (Optional)

The server works without authentication but has rate limits. To increase rate limits and enable write operations:

1. **Get an API key from [Stack Apps](https://stackapps.com/apps/oauth/register)**
2. **For READ-ONLY usage:** Add the API key to your environment:
   ```bash
   export STACKOVERFLOW_API_KEY="your-api-key"
   ```

3. **For WRITE operations:** You MUST also obtain an OAuth `access_token` with the correct scopes from Stack Apps, then set both:
   ```bash
   export STACKOVERFLOW_API_KEY="your-api-key"
   export STACKOVERFLOW_ACCESS_TOKEN="your-access-token"
   ```

**Note:** Write operations (`post_question`, `post_solution`, `thumbs_up`, `comment_solution`) require both `STACKOVERFLOW_API_KEY` and `STACKOVERFLOW_ACCESS_TOKEN`.

## 📋 Creating Intent Mappings

Intent mappings connect natural language keywords to specific MCP server tools.

**SQL to create intent mappings:**

```sql
BEGIN TRANSACTION;

-- Intent mappings for OI-stackoverflow-read-write-mcp
INSERT OR REPLACE INTO intent_mappings (keyword, server_name, tool_name, priority) VALUES
-- Search tools
('search by error', 'OI-stackoverflow-read-write-mcp', 'search_by_error', 10),
('search stackoverflow error', 'OI-stackoverflow-read-write-mcp', 'search_by_error', 10),
('find error solution', 'OI-stackoverflow-read-write-mcp', 'search_by_error', 10),
('search by tags', 'OI-stackoverflow-read-write-mcp', 'search_by_tags', 10),
('search stackoverflow tags', 'OI-stackoverflow-read-write-mcp', 'search_by_tags', 10),
('find questions by tags', 'OI-stackoverflow-read-write-mcp', 'search_by_tags', 10),
('analyze stack trace', 'OI-stackoverflow-read-write-mcp', 'analyze_stack_trace', 10),
('stack trace analysis', 'OI-stackoverflow-read-write-mcp', 'analyze_stack_trace', 10),
('find stack trace solution', 'OI-stackoverflow-read-write-mcp', 'analyze_stack_trace', 10),
-- Write tools (STRICT)
('post question', 'OI-stackoverflow-read-write-mcp', 'post_question', 10),
('create question', 'OI-stackoverflow-read-write-mcp', 'post_question', 10),
('ask question', 'OI-stackoverflow-read-write-mcp', 'post_question', 10),
('post solution', 'OI-stackoverflow-read-write-mcp', 'post_solution', 10),
('post answer', 'OI-stackoverflow-read-write-mcp', 'post_solution', 10),
('answer question', 'OI-stackoverflow-read-write-mcp', 'post_solution', 10),
('thumbs up', 'OI-stackoverflow-read-write-mcp', 'thumbs_up', 10),
('upvote', 'OI-stackoverflow-read-write-mcp', 'thumbs_up', 10),
('vote up', 'OI-stackoverflow-read-write-mcp', 'thumbs_up', 10),
('comment solution', 'OI-stackoverflow-read-write-mcp', 'comment_solution', 10),
('add comment', 'OI-stackoverflow-read-write-mcp', 'comment_solution', 10),
('comment on question', 'OI-stackoverflow-read-write-mcp', 'comment_solution', 10);

COMMIT;
```

## 📝 Creating Parameter Rules

Parameter rules define which fields are required and how to extract them from natural language queries.

**SQL to create parameter rules:**

```sql
BEGIN TRANSACTION;

-- Parameter rules for OI-stackoverflow-read-write-mcp
INSERT OR REPLACE INTO parameter_rules (server_name, tool_name, tool_signature, required_fields, field_generators, patterns) VALUES
-- Search tools
('OI-stackoverflow-read-write-mcp', 'search_by_error', 'OI-stackoverflow-read-write-mcp::search_by_error', '["errorMessage"]',
 '{"errorMessage": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_error.errorMessage"}, "language": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_error.language"}, "technologies": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_error.technologies"}, "minScore": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_error.minScore"}, "includeComments": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_error.includeComments"}, "responseFormat": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_error.responseFormat"}, "limit": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_error.limit"}}', '[]'),

('OI-stackoverflow-read-write-mcp', 'search_by_tags', 'OI-stackoverflow-read-write-mcp::search_by_tags', '["tags"]',
 '{"tags": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_tags.tags"}, "minScore": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_tags.minScore"}, "includeComments": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_tags.includeComments"}, "responseFormat": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_tags.responseFormat"}, "limit": {"FromQuery": "OI-stackoverflow-read-write-mcp::search_by_tags.limit"}}', '[]'),

('OI-stackoverflow-read-write-mcp', 'analyze_stack_trace', 'OI-stackoverflow-read-write-mcp::analyze_stack_trace', '["stackTrace", "language"]',
 '{"stackTrace": {"FromQuery": "OI-stackoverflow-read-write-mcp::analyze_stack_trace.stackTrace"}, "language": {"FromQuery": "OI-stackoverflow-read-write-mcp::analyze_stack_trace.language"}, "includeComments": {"FromQuery": "OI-stackoverflow-read-write-mcp::analyze_stack_trace.includeComments"}, "responseFormat": {"FromQuery": "OI-stackoverflow-read-write-mcp::analyze_stack_trace.responseFormat"}, "limit": {"FromQuery": "OI-stackoverflow-read-write-mcp::analyze_stack_trace.limit"}}', '[]'),

-- Write tools (STRICT - require API key and access token)
('OI-stackoverflow-read-write-mcp', 'post_question', 'OI-stackoverflow-read-write-mcp::post_question', '["title", "body", "tags", "errorSignature", "triedApproaches"]',
 '{"title": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_question.title"}, "body": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_question.body"}, "tags": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_question.tags"}, "errorSignature": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_question.errorSignature"}, "triedApproaches": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_question.triedApproaches"}}', '[]'),

('OI-stackoverflow-read-write-mcp', 'post_solution', 'OI-stackoverflow-read-write-mcp::post_solution', '["questionId", "body", "confirmedResolved", "evidence"]',
 '{"questionId": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_solution.questionId"}, "body": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_solution.body"}, "confirmedResolved": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_solution.confirmedResolved"}, "evidence": {"FromQuery": "OI-stackoverflow-read-write-mcp::post_solution.evidence"}}', '[]'),

('OI-stackoverflow-read-write-mcp', 'thumbs_up', 'OI-stackoverflow-read-write-mcp::thumbs_up', '["postId", "confirmedFixed"]',
 '{"postId": {"FromQuery": "OI-stackoverflow-read-write-mcp::thumbs_up.postId"}, "confirmedFixed": {"FromQuery": "OI-stackoverflow-read-write-mcp::thumbs_up.confirmedFixed"}}', '[]'),

('OI-stackoverflow-read-write-mcp', 'comment_solution', 'OI-stackoverflow-read-write-mcp::comment_solution', '["questionId", "body"]',
 '{"questionId": {"FromQuery": "OI-stackoverflow-read-write-mcp::comment_solution.questionId"}, "body": {"FromQuery": "OI-stackoverflow-read-write-mcp::comment_solution.body"}}', '[]');

COMMIT;
```

## 🔍 Parameter Extractors

Add these patterns to `parameter_extractors.toml.default`:

```toml
# ============================================================================
# OI-stackoverflow-read-write-mcp
# ============================================================================

# Search tools
"OI-stackoverflow-read-write-mcp::search_by_error.errorMessage" = "remove:search,by,error,for,find,solution"
"OI-stackoverflow-read-write-mcp::search_by_error.language" = "regex:(?:language|lang|programming[\\s_-]?language)[\\s:]+([a-zA-Z]+)"
"OI-stackoverflow-read-write-mcp::search_by_error.technologies" = "regex:(?:technologies?|tech|frameworks?|libraries?)[\\s:]+([^\\s]+(?:\\s*,\\s*[^\\s]+)*)"
"OI-stackoverflow-read-write-mcp::search_by_error.minScore" = "regex:(?:min[\\s_-]?score|minimum[\\s_-]?score|score)[\\s:]+(\\d+)"
"OI-stackoverflow-read-write-mcp::search_by_error.includeComments" = "regex:(?:include[\\s_-]?comments?|with[\\s_-]?comments?)[\\s:]+(true|false|yes|no)"
"OI-stackoverflow-read-write-mcp::search_by_error.responseFormat" = "regex:(?:format|output[\\s_-]?format)[\\s:]+(json|markdown)"
"OI-stackoverflow-read-write-mcp::search_by_error.limit" = "regex:(?:limit|max|maximum|results?)[\\s:]+(\\d+)"

"OI-stackoverflow-read-write-mcp::search_by_tags.tags" = "remove:search,by,tags,for,find,questions"
"OI-stackoverflow-read-write-mcp::search_by_tags.minScore" = "regex:(?:min[\\s_-]?score|minimum[\\s_-]?score|score)[\\s:]+(\\d+)"
"OI-stackoverflow-read-write-mcp::search_by_tags.includeComments" = "regex:(?:include[\\s_-]?comments?|with[\\s_-]?comments?)[\\s:]+(true|false|yes|no)"
"OI-stackoverflow-read-write-mcp::search_by_tags.responseFormat" = "regex:(?:format|output[\\s_-]?format)[\\s:]+(json|markdown)"
"OI-stackoverflow-read-write-mcp::search_by_tags.limit" = "regex:(?:limit|max|maximum|results?)[\\s:]+(\\d+)"

"OI-stackoverflow-read-write-mcp::analyze_stack_trace.stackTrace" = "remove:analyze,stack,trace,find,solution"
"OI-stackoverflow-read-write-mcp::analyze_stack_trace.language" = "regex:(?:language|lang|programming[\\s_-]?language)[\\s:]+([a-zA-Z]+)"
"OI-stackoverflow-read-write-mcp::analyze_stack_trace.includeComments" = "regex:(?:include[\\s_-]?comments?|with[\\s_-]?comments?)[\\s:]+(true|false|yes|no)"
"OI-stackoverflow-read-write-mcp::analyze_stack_trace.responseFormat" = "regex:(?:format|output[\\s_-]?format)[\\s:]+(json|markdown)"
"OI-stackoverflow-read-write-mcp::analyze_stack_trace.limit" = "regex:(?:limit|max|maximum|results?)[\\s:]+(\\d+)"

# Write tools (STRICT - require API key and access token)
"OI-stackoverflow-read-write-mcp::post_question.title" = "regex:(?:title)[\\s:]+([^\\n]+)"
"OI-stackoverflow-read-write-mcp::post_question.body" = "regex:(?:body|content|question[\\s_-]?body)[\\s:]+([^\\n]+(?:\\n[^\\n]+)*)"
"OI-stackoverflow-read-write-mcp::post_question.tags" = "regex:(?:tags?)[\\s:]+([^\\s]+(?:\\s*,\\s*[^\\s]+)*)"
"OI-stackoverflow-read-write-mcp::post_question.errorSignature" = "regex:(?:error[\\s_-]?signature|error[\\s_-]?summary)[\\s:]+([^\\n]+)"
"OI-stackoverflow-read-write-mcp::post_question.triedApproaches" = "regex:(?:tried[\\s_-]?approaches?|attempted[\\s_-]?fixes?)[\\s:]+([^\\n]+(?:\\n[^\\n]+)*)"

"OI-stackoverflow-read-write-mcp::post_solution.questionId" = "regex:(?:question[\\s_-]?id|question[\\s_-]?number)[\\s:]+(\\d+)"
"OI-stackoverflow-read-write-mcp::post_solution.body" = "regex:(?:body|content|answer[\\s_-]?body)[\\s:]+([^\\n]+(?:\\n[^\\n]+)*)"
"OI-stackoverflow-read-write-mcp::post_solution.confirmedResolved" = "regex:(?:confirmed[\\s_-]?resolved|resolved|fixed)[\\s:]+(true|false|yes|no)"
"OI-stackoverflow-read-write-mcp::post_solution.evidence" = "regex:(?:evidence|proof|tests?|logs?)[\\s:]+([^\\n]+(?:\\n[^\\n]+)*)"

"OI-stackoverflow-read-write-mcp::thumbs_up.postId" = "regex:(?:post[\\s_-]?id|post[\\s_-]?number|answer[\\s_-]?id|question[\\s_-]?id)[\\s:]+(\\d+)"
"OI-stackoverflow-read-write-mcp::thumbs_up.confirmedFixed" = "regex:(?:confirmed[\\s_-]?fixed|fixed|resolved)[\\s:]+(true|false|yes|no)"

"OI-stackoverflow-read-write-mcp::comment_solution.questionId" = "regex:(?:question[\\s_-]?id|question[\\s_-]?number)[\\s:]+(\\d+)"
"OI-stackoverflow-read-write-mcp::comment_solution.body" = "regex:(?:body|content|comment[\\s_-]?body)[\\s:]+([^\\n]+(?:\\n[^\\n]+)*)"
```

## 🛠️ Available Tools (7 total)

### Search Tools (3 tools)

1. **`search_by_error`** - Search Stack Overflow for error-related questions
   - **Parameters:**
     - `errorMessage` (required, string): Error message to search for
     - `language` (optional, string): Programming language
     - `technologies` (optional, array): Related technologies
     - `minScore` (optional, number): Minimum score threshold
     - `includeComments` (optional, boolean): Include comments in results
     - `responseFormat` (optional, enum): "json" or "markdown"
     - `limit` (optional, number): Maximum number of results

2. **`search_by_tags`** - Search Stack Overflow questions by tags
   - **Parameters:**
     - `tags` (required, array): Tags to search for
     - `minScore` (optional, number): Minimum score threshold
     - `includeComments` (optional, boolean): Include comments in results
     - `responseFormat` (optional, enum): "json" or "markdown"
     - `limit` (optional, number): Maximum number of results

3. **`analyze_stack_trace`** - Analyze stack trace and find relevant solutions
   - **Parameters:**
     - `stackTrace` (required, string): Stack trace to analyze
     - `language` (required, string): Programming language
     - `includeComments` (optional, boolean): Include comments in results
     - `responseFormat` (optional, enum): "json" or "markdown"
     - `limit` (optional, number): Maximum number of results

### Write Tools (4 tools) - STRICT Policies

**⚠️ IMPORTANT:** All write tools require both `STACKOVERFLOW_API_KEY` and `STACKOVERFLOW_ACCESS_TOKEN` environment variables.

4. **`post_question`** - Create a new Stack Overflow question (STRICT)
   - **Policy:** ONLY if no remotely similar error exists AND ONLY after at least 3 distinct attempted fixes
   - **Parameters:**
     - `title` (required, string): Concise question title
     - `body` (required, string): Full markdown body including problem context
     - `tags` (required, array): Up to 5 tags relevant to the question
     - `errorSignature` (required, string): Short error signature used to check for duplicates
     - `triedApproaches` (required, array): At least 3 distinct approaches already attempted

5. **`post_solution`** - Post an answer (STRICT)
   - **Policy:** ONLY if no similar solution exists, the issue is confirmed resolved, AND concrete evidence is provided
   - **Parameters:**
     - `questionId` (required, number): Target question ID
     - `body` (required, string): Markdown answer including steps and rationale
     - `confirmedResolved` (required, boolean): Must be true only if this solution fixed the issue
     - `evidence` (required, array): Evidence references (tests, logs, repo links, repro cases)

6. **`thumbs_up`** - Upvote a post (STRICT)
   - **Policy:** ONLY when a solution demonstrably fixed the issue
   - **Parameters:**
     - `postId` (required, number): ID of the answer or question to upvote
     - `confirmedFixed` (required, boolean): Must be true only if the solution actually fixed the issue

7. **`comment_solution`** - Add a comment (STRICT)
   - **Policy:** ONLY on a question that currently has no accepted solution
   - **Parameters:**
     - `questionId` (required, number): Question ID to comment on
     - `body` (required, string): Concise, constructive comment with additional context

## 💡 Usage Examples

### Natural Language Queries

```bash
# Search by error
./oi "search by error TypeError Cannot read property length of undefined language javascript"
./oi "find error solution for connection refused in python"

# Search by tags
./oi "search by tags python pandas dataframe"
./oi "find questions by tags react typescript"

# Analyze stack trace
./oi "analyze stack trace Error: ENOENT: no such file or directory language javascript"
```

### Direct Calls

```bash
# Search by error
./brain-trust4 call OI-stackoverflow-read-write-mcp search_by_error '{
  "errorMessage": "TypeError: Cannot read property '\''length'\'' of undefined",
  "language": "javascript",
  "technologies": ["react"],
  "minScore": 5,
  "includeComments": true,
  "responseFormat": "markdown",
  "limit": 3
}'

# Search by tags
./brain-trust4 call OI-stackoverflow-read-write-mcp search_by_tags '{
  "tags": ["python", "pandas", "dataframe"],
  "minScore": 10,
  "includeComments": true,
  "responseFormat": "json",
  "limit": 5
}'

# Analyze stack trace
./brain-trust4 call OI-stackoverflow-read-write-mcp analyze_stack_trace '{
  "stackTrace": "Error: ENOENT: no such file or directory, open '\''config.json'\''\n    at Object.openSync (fs.js:476:3)",
  "language": "javascript",
  "includeComments": true,
  "responseFormat": "markdown",
  "limit": 3
}'
```

## 🚨 Important Notes

### Rate Limiting

- The server implements rate limiting (30 requests per minute)
- Without API key: Subject to Stack Overflow's default rate limits
- With API key: Higher rate limits (300 requests per day)

### Write Operations

All write operations (`post_question`, `post_solution`, `thumbs_up`, `comment_solution`) have **STRICT policies** enforced:

1. **`post_question`**: 
   - Checks for similar questions before posting
   - Requires at least 3 attempted fixes
   - Prevents duplicate questions

2. **`post_solution`**:
   - Only posts if question has no answers
   - Requires `confirmedResolved: true`
   - Requires evidence array

3. **`thumbs_up`**:
   - Only upvotes if `confirmedFixed: true`
   - Prevents spam upvoting

4. **`comment_solution`**:
   - Only comments on questions without accepted answers
   - Prevents unnecessary comments

### Response Formats

- **JSON**: Structured data with all question/answer details
- **Markdown**: Formatted view with question title, body, answers, and comments

## 📖 Additional Resources

- **GitHub Repository:** https://github.com/OI-OS/OI-stackoverflow-read-write-mcp
- **Original Repository:** https://github.com/omerhochman/stackoverflow-read-write-mcp
- **Stack Overflow API:** https://api.stackexchange.com/docs
- **Stack Apps (API Registration):** https://stackapps.com/apps/oauth/register

## ✅ Verification

After installation, verify the server is working:

```bash
# List tools
./brain-trust4 tools OI-stackoverflow-read-write-mcp

# Test search
./oi "search by error TypeError language javascript"
```

If all commands work, the server is successfully installed and configured!

