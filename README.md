# CoCo Desktop Workshop

**Duration:** 4 hours (half-day)  
**Audience:** Grocery retail analytics teams, data engineers, and technical stakeholders  
**Prerequisites:** CoCo Desktop installed, active Snowflake connection configured with access to retail/grocery data tables  
**Scenario:** You are part of a grocery retailer's data team building analytics, dashboards, and automation to improve store operations, inventory management, and customer insights.

---

## Agenda Overview

| Time | Section | Duration |
|------|---------|----------|
| 10:00 - 10:40 | Section 1: Orientation & UI Fundamentals | 40 min |
| 10:40 - 11:20 | Section 2: Understanding Skills | 40 min |
| 11:20 - 12:00 | Section 3: Building an Application | 40 min |
| 12:00 - 12:30 | Lunch | 30 min |
| 12:30 - 1:10 | Section 4: Hooks and Rules | 40 min |
| 1:10 - 1:40 | Section 5: Task Board, Scheduled Tasks & Wrap-Up | 30 min |
| 1:40 - 2:10 | Optional: MCP (Model Context Protocol) | 30 min |

---

## Section 1: Orientation & UI Fundamentals (40 min)

### 1.1 Launching CoCo

- Open CoCo Desktop application
- Start a session: the app opens an interactive REPL (Read-Eval-Print-Loop)
- Review the interface layout: prompt area, output area, status bar

**Retail Context:** You've just joined the morning standup — the produce team needs a quick analysis of spoilage rates. Instead of opening multiple tools, you'll use CoCo as your single pane of glass.

### 1.2 The Two Tabs: Agent vs Editor

CoCo Desktop has two tabs in the top-right corner:

**Agent Tab:**
- The AI chat panel for conversational interaction
- Reads/writes files, runs commands, executes SQL autonomously
- All agentic capabilities: skills, subagents, MCP, Snowflake integration
- Use for: "Show me last week's shrinkage by department" or "Build me a promotion impact model"

**Editor Tab:**
- A standard code editor (VS Code-based) for manual file editing
- Syntax highlighting, direct edits, code navigation — no AI involvement
- Use for: reviewing SQL the agent wrote, tweaking a Streamlit layout, browsing project files

You can work in both simultaneously — edit a SQL file manually in Editor, then ask the Agent to run and validate it.

**Within the Agent tab — Modes:**

| Mode | Behavior |
|------|----------|
| Default (Confirm Actions) | Agent prompts for permission before changes |
| Plan Mode (`/plan`) | Read-only — agent proposes but cannot execute |
| Bypass (`/bypass`) | Agent auto-approves all actions |

**Hands-on Exercise:**
1. Click the **Editor** tab — use **File → Open Folder** to open your workshop project directory (e.g., `~/CortexCodeWorkshop`). This sets your **working directory** — all files the agent creates or reads will be relative to this folder. You'll see the folder tree appear in the left sidebar.
2. In the left sidebar, expand the **Snowflake catalog** and browse to the **RETAIL_DB** database. Explore the schemas and tables listed — this is your reference for the retail database schema used throughout the workshop.
3. Click the **Agent** tab — ask: "Within RETAIL_DB, what tables do we have related to produce inventory?"
4. Switch to plan mode: `/plan`
5. Ask: "Build a pipeline that loads daily POS files from a stage into RETAIL_DB.SALES.DAILY_ITEM_SALES, deduplicates on item_id + store_id + sale_date, and refreshes a downstream dynamic table for weekly shrinkage rollups. Prefix all objects created with XX_" — Replace **XX_** with your initials since we are in a shared working environment. observe it designs the full pipeline without creating any objects
6. Click **Edit Plan** on the plan card that appears — review the architecture diagram (mermaid flowchart showing stage → raw → dedup → dynamic table) and the specific SQL statements that will be executed for each step
7. Switch back: `/plan-off`

### 1.3 Setting Approval / Permission Modes

| Mode | Command | Grocery Use Case |
|------|---------|-----------------|
| Confirm Actions | (default) | Building a new promotion analysis — review each step |
| Plan Mode | `/plan` | Exploring a complex supply chain query before running it |
| Bypass | `/bypass` | Generating 20 department-level reports in a batch |

**Hands-on Exercise:**
1. In default mode, ask: "Create a SQL file that calculates weekly produce shrinkage by store" — observe the permission prompt before file creation
2. Switch to `/bypass` and ask: "Now create the same for dairy, bakery, and deli departments" — observe auto-approval
3. Return to safe mode: `/bypass-off`

### 1.4 The Context Window & When to Start a New Session

The **context window** is the total amount of information the agent can hold in memory during a single session. Think of it as the agent's working memory — it includes everything: your messages, the agent's responses, tool outputs (SQL results, file contents), and internal reasoning.

**Why it matters:**
- Every message, query result, and file read consumes context space
- When the context window fills up, older exchanges are summarized or dropped — the agent loses detail on earlier work
- A bloated context leads to slower responses, missed instructions, and the agent "forgetting" things you told it earlier in the session

**Context window indicators:**
- A usage indicator appears in the status area when context is ≤30% remaining
- Enable **Settings → Show Context Usage** (`alwaysShowContextUsage`) to always see it
- Use `/context` to view a detailed breakdown of what's consuming your context

**Best practices — when to start a new session:**

| Signal | Action |
|--------|--------|
| Switching to an unrelated topic | Start a new session — don't mix "build a promo dashboard" with "debug a data pipeline" |
| Context indicator shows <20% remaining | Start fresh — the agent is about to lose earlier details |
| The agent repeats questions you already answered | Context has been summarized — key details were lost |
| A long exploratory session produced a final plan | Start a new session to execute the plan cleanly |
| You finished a task and want to begin another | New session keeps things focused and fast |
| The agent starts making errors on things it got right earlier | Context pressure is degrading quality |

**Strategies to extend a session when needed:**
- **Compact** (`/compact` or session menu → Compact): Summarizes the conversation history, freeing space while retaining key context. Add instructions like `/compact keep the SQL schema details` to preserve specific information.
- **Be specific in prompts**: "Query RETAIL_DB.INVENTORY.SHRINKAGE for Store 305" consumes less context than vague requests that trigger exploratory searches
- **Use files as external memory**: Ask the agent to save intermediate results to files (e.g., `save this analysis to reports/findings.md`) — you can `@`-reference them later without re-querying

**Hands-on Exercise:**
1. Hover over the **context window indicator** in the bottom-right status area of the Agent chat — note the percentage consumed and the token count (e.g., "10.7% · 106.5K out of 1M tokens used")

### 1.5 Essential Shortcuts & Syntax

| Shortcut | Action | Retail Example |
|----------|--------|----------------|
| `@` | File completion | `@sql/promo_lift.sql` |
| `/` | Skill invocation | `/sql-author write a shrinkage query` |
| `!` | Run bash inline | `!wc -l sql/*.sql` |
| `/sql` | Execute SQL inline | `/sql SELECT store_id, SUM(waste_lbs) FROM produce_waste GROUP BY 1` |
| `/sql-author` | Write & run SQL from natural language | `/sql-author find top 10 stores by shrinkage rate` |

---

## Section 2: Understanding Skills (40 min)

### 2.1 What Are Skills?

- Reusable instruction sets that encode domain expertise
- Invoked with `/skill-name` from the Agent chat
- Browse and discover available skills in the **Skills panel** in the sidebar
- Priority: Project > Global > Remote > Bundled

**Retail Example:** A `/shrinkage-analysis` skill knows how to query your inventory tables, calculate shrinkage rates, compare to benchmarks, and output formatted reports — all from a single prompt.

### 2.2 Reviewing Packaged (Bundled) Skills

Bundled skills ship with CoCo and are accessible directly from the **Skills panel** in the sidebar — no file browsing required.

**Hands-on Exercise:**
1. Open the **Skills panel** from the sidebar to view all available skills
2. Find `sql-author` — click it to see its description, then invoke it from the Agent chat: `/sql-author Write a query to find the top 10 items by unit sales last week across all stores`
3. Find `developing-with-streamlit` in the Skills panel — we'll use this later for dashboards
4. Try from the Agent chat: `/cortex-code-guide What shortcuts help me work with Snowflake tables?`

### 2.3 Adding a Skill from GitHub

Remote skills from GitHub can be added via the **Skills panel**. Open the Skills panel (lightning bolt icon in the sidebar), then click the **`+`** button beside the **GITHUB** section. Provide the GitHub repo URL pointing to the skill folder and CoCo will fetch and register it automatically.

**Hands-on Exercise:**
1. Open the **Skills panel** (lightning bolt icon) and click the **`+`** beside the **GITHUB** section
2. Enter the following public repo skill URL: `https://github.com/Snowflake-Labs/coco-skills/tree/main/skills/snowflake-docs`
3. CoCo will clone the repo and register the skill
4. Once installed, verify the new skill appears in the GITHUB section of the Skills panel
5. Test the skill from the Agent chat: `/snowflake-docs How do I create a dynamic table?` — observe how the skill fetches relevant Snowflake documentation and provides a detailed answer with syntax examples

**Troubleshooting — "Host key verification failed" error:**

If you see an error like:
> Failed to add skill: Failed to download repository: Cloning into '...'... Host key verification failed. fatal: Could not read from remote repository.

This means Git is trying to use SSH to clone the repo but your machine isn't configured for SSH access to GitHub. Since the skills repo is public, the simplest fix is to tell Git to use HTTPS instead. Ask CoCo in the Agent chat:

```
Please run: git config --global url."https://github.com/".insteadOf "git@github.com:"
```

This configures Git to automatically use HTTPS for all GitHub repos, which requires no SSH keys or authentication for public repositories. After this, retry adding the skill from the `+` button and it will succeed.

### 2.4 Creating a New Skill from Scratch

**Scenario:** Your category management team frequently asks for promotion effectiveness analysis. Instead of manually writing a skill file, we'll use CoCo's built-in `skill-development` skill to create a properly structured skill that follows best practices.

**How it works:** When you ask CoCo to create a skill, it automatically invokes the `skill-development` bundled skill. This ensures the generated skill follows best practices — proper frontmatter, stopping points, clear workflow steps, and concise structure.

**Hands-on Exercise:**

1. In the **Agent tab**, copy and paste the following prompt:

   ```
   Create a new skill as follows:
   name: promo-effectiveness
   description: Analyze promotion lift and ROI. Use when: promo analysis, promotion impact, lift calculation, ad effectiveness. Triggers: promo, promotion, lift, TPR, ad spend.
   Source data from the following: - Promotions table: RETAIL_DB.MARKETING.PROMOTIONS (promo_id, item_id, start_date, end_date, discount_pct, promo_type)
   - Sales table: RETAIL_DB.SALES.DAILY_ITEM_SALES (store_id, item_id, sale_date, units_sold, revenue)

   Perform the following analysis:
   1. Identify the promotion period and the baseline period (4 weeks prior)
   2. Calculate baseline average daily units
   3. Calculate promo-period average daily units
   4. Compute lift percentage: ((promo_avg - baseline_avg) / baseline_avg) * 100
   5. Estimate incremental units and revenue
   6. If discount cost data available, calculate ROI

   Present results as a summary table with: Item, Baseline Units/Day, Promo Units/Day, Lift %, Incremental Revenue, ROI

   Always confrim data range and items with user
   ```

2. When prompted for the skill location, select **Global** (`~/.snowflake/cortex/skills/`) so it's available across all projects

3. Observe how CoCo invokes the `skill-development` skill and generates a well-structured `SKILL.md` with proper frontmatter, workflow steps, stopping points, and SQL templates

4. **Refresh the skill list:** Open the **Skills panel** (lightning bolt icon in the sidebar) and click the **refresh icon** to reload available skills — verify `promo-effectiveness` now appears in the list

5. **View the skill:** Click on `promo-effectiveness` in the Skills panel to view the generated SKILL.md — note the structured workflow, mandatory stopping points, and SQL templates

6. **Test the skill** — try these example prompts from the Agent chat:

   - `/promo-effectiveness What was the lift on the BOGO promo?`
   - `/promo-effectiveness Compare ROI across all TPR promotions in Q1 vs Q2`

7. Observe how the skill pauses to confirm scope (items, date range, granularity) before running queries — this is the stopping point you specified in the requirements. Note that if you prompted 'What was the lift on the BOGO strawberries promo in April?' then it would have assumed those answers and not stopped to ask.

---

## Section 3: Building an Application (40 min)

### 3.1 Building & Deploying a Streamlit App to Snowflake

**Scenario:** The store operations team needs a real-time dashboard showing inventory levels, out-of-stock items, and spoilage alerts by department. You'll build it and deploy it directly to Snowflake so all 200 store managers can access it via Snowsight — no local Python environment required.

**How it works:** CoCo uses the `/build-app` skill to scaffold Streamlit apps. It asks what you want to build, recommends a framework (Streamlit-in-Snowflake for dashboards, Snowflake App Runtime for full-stack web apps), confirms your choice, then generates a working app and deploys it directly to Snowflake as a Streamlit-in-Snowflake (SiS) object.

**Hands-on Exercise:**

1. In the Agent tab, type: `/build-app create an app for a store dashboard and deploy it to Snowflake`

2. CoCo will ask what the app should do. Provide a detailed description that includes:
   - **What it does** — the specific visualizations and KPIs you want
   - **Who uses it** — the audience (e.g., store managers)
   - **What data it touches** — name the schemas explicitly so CoCo queries real tables from the start
   - **Whether users view data or take actions** — this determines the framework choice
   - **App name with your initials** — since multiple participants are deploying to the same account

   Example prompt:
   > "This should be for store managers to use. It should include a store selector and date range picker. It should show KPIs for total revenue, shrinkage rate, out-of-stock count, avg basket size. I would like to see a bar chart of revenue by department and a table of the top 10 items approaching expiration date. Use data from RETAIL_DB.SALES and RETAIL_DB.INVENTORY schemas. Deploy it to Snowflake as Streamlit in Snowflake. Name the app '[YOUR_INITIALS]_Store Dashboard'. Use Snowpark get_active_session() for data access."

   Replace `[YOUR_INITIALS]` with your actual initials (e.g. `CL_Store Dashboard`).

3. CoCo recommends **Streamlit-in-Snowflake** as the framework (since this is a pure view/explore dashboard with no user actions). Confirm the selection.

4. CoCo discovers the tables in your schemas, inspects their columns, scaffolds the app using Snowpark's `get_active_session()` for data access, creates a stage, uploads the files, creates the Streamlit object in Snowflake, and opens the deployed app in the browser.

5. The initial deployment may have errors (wrong table names, incompatible Streamlit features, etc.). Ask CoCo to fix them:
   > "Test and validate the streamlit app works without errors and that selecting a store or date range updates the data."

   CoCo will open the deployed app in the browser, identify any runtime errors, fix the code (correcting table/column references, Streamlit version compatibility issues, Snowpark API usage), re-upload to the stage, and verify the app renders correctly with live data.

6. Navigate to the app yourself in Snowsight: **Projects** → **Streamlit** → find your named app (e.g. "CL_Store Dashboard"). Test the store selector and date range picker — verify the KPIs, bar chart, and expiration table all update with different values for each store.

7. **Iterate on the app with natural language.** Building an app with CoCo is iterative — once the base dashboard works, you can keep adding features by describing what you want. CoCo will modify the code, re-upload to the stage, and the deployed app updates automatically.

   Try adding a new tab:
   > "Modify the streamlit app to add a tab that shows a heatmap of out-of-stock events by hour and department. Include a filter for organic vs conventional produce."

   CoCo will:
   - Add a tabbed layout (Overview + Out-of-Stock Heatmap)
   - Write a SQL query joining `DAILY_STOCK_LEVELS` with `PRODUCTS` to get OOS events by department and the `IS_ORGANIC` flag
   - Generate a simulated hourly distribution (since stock data is daily) with realistic peak-hour weighting
   - Render an Altair heatmap with a radio button filter for All / Organic / Conventional
   - Re-upload the updated files to the Snowflake stage so the deployed app reflects the changes

   Other examples of iterative prompts that work well:
   - "Add a sparkline showing daily revenue trend for the past 7 days next to the Total Revenue KPI"
   - "Add a sidebar with a department multi-select filter that applies to all charts"
   - "Replace the expiration table with a color-coded version where items expiring within 2 days are highlighted red"

**What to include in your prompt for best results:**
- Name the **specific KPIs and visualizations** you want (don't be vague)
- Specify the **audience** (store managers, analysts, executives) — this helps CoCo pick the right level of detail
- State whether users **only view data or also take actions** — this is the key signal for framework selection
- Name the **schemas or tables** your data lives in — CoCo will discover columns automatically
- Mention **filters** you want (store selector, date picker, department filter)
- Include your **initials prefix** in the app name for shared environments

**Key points:**
- `/build-app` handles the full flow: framework recommendation → scaffold → deploy to Snowflake
- The app deploys directly to Snowflake as a Streamlit-in-Snowflake object — no local server needed
- Data access uses Snowpark's `get_active_session()` — the app runs natively inside Snowflake
- Iteration is fast: CoCo edits the code and re-uploads to the stage; the deployed app updates on refresh
- Packages are limited to the Snowflake Anaconda channel (Streamlit ≤1.52)
- Access is controlled via Snowflake roles — grant access to any role for your store managers
- Authentication is handled by Snowflake session (SSO) — no separate credentials needed

| Aspect | Streamlit in Snowflake (SiS) |
|--------|-------------------------------|
| Runtime | Snowflake infrastructure |
| Packages | Snowflake Anaconda channel (max Streamlit 1.52) |
| Auth | Snowflake session (SSO) |
| Access | Any role you grant |
| Data | Native Snowpark `get_active_session()` |
| Iteration | Re-upload files to stage |

---

## Section 4: Hooks and Rules (40 min)

### 4.1 Hooks

**What are hooks?** Hooks are shell scripts that fire automatically at specific moments in the CoCo workflow — before a tool runs (`PreToolUse`), after it completes (`PostToolUse`), or at session events. Your script receives the full tool call as JSON on stdin. **Exit 0 = allow. Exit 2 = block.** When you block, CoCo receives your reason and adapts — it finds a safer way rather than giving up.

This gives you guardrails that CoCo can't enforce natively: protected schemas, blocked destructive operations, and audit trails — with zero friction to the engineer.

**Configured in one file:** `~/.snowflake/cortex/hooks.json`

---

**Hands-on Exercise — see the difference hooks make**

**Part 1: Without a hook (the problem)**

1. In the Agent chat, ask CoCo to create a view in the retail database:
   > "Create a view called `[YOUR_INITIALS]_daily_revenue` in `RETAIL_DB.SALES` that shows total revenue per store per day from DAILY_TRANSACTIONS"

   CoCo will generate and execute something like:
   ```sql
   CREATE OR REPLACE VIEW RETAIL_DB.SALES.[YOUR_INITIALS]_DAILY_REVENUE AS
   SELECT
       STORE_ID,
       TRANSACTION_DATE,
       TOTAL_REVENUE
   FROM RETAIL_DB.SALES.DAILY_TRANSACTIONS;
   ```

   CoCo creates it. ✓

2. Now ask CoCo to drop it:
   > "Drop the view `[YOUR_INITIALS]_daily_revenue` from `RETAIL_DB.SALES`"

   CoCo will execute:
   ```sql
   DROP VIEW RETAIL_DB.SALES.[YOUR_INITIALS]_DAILY_REVENUE;
   ```

   CoCo drops it without hesitation. No guardrail — a destructive DDL against a shared database ran silently.

---

**Part 2: Add the hook**

3. In the **Editor tab**, create a new file in your workshop folder called `block-drop-retail.sh` (**File → New File**, paste the contents below, then **Save As** → `block-drop-retail.sh` in your working directory):

```bash
#!/bin/bash
INPUT=$(cat)
TOOL=$(echo "$INPUT" | python3 -c "import sys,json; print(json.load(sys.stdin).get('tool_name',''))" 2>/dev/null)

if [ "$TOOL" = "snowflake_sql_execute" ]; then
    SQL=$(echo "$INPUT" | python3 -c "import sys,json; print(json.load(sys.stdin).get('tool_input',{}).get('sql','').upper())" 2>/dev/null)

    if echo "$SQL" | grep -qE "(DROP|DELETE|TRUNCATE).*RETAIL_DB"; then
        echo "BLOCKED: DROP, DELETE, and TRUNCATE operations against RETAIL_DB are not permitted from CoCo. Use Snowsight for destructive operations on shared databases." >&2
        exit 2
    fi
fi

exit 0
```

4. Ask CoCo to move it into place and make it executable:
   > "Move block-drop-retail.sh to ~/.snowflake/cortex/hooks/ and make it executable"

   CoCo will create the hooks directory if needed, move the file, and run `chmod +x`.

5. Ask CoCo to register the hook in `hooks.json`:
   > "Update ~/.snowflake/cortex/hooks.json to add a PreToolUse hook that runs the block-drop-retail.sh script in the hooks folder whenever snowflake_sql_execute is called, with a 5 second timeout. Use the full absolute path to the script, not ~"

   CoCo will create or update the JSON file with the correct structure.

   > **Important:** The command path in `hooks.json` must be an **absolute path** (e.g. `/Users/yourname/.snowflake/cortex/hooks/block-drop-retail.sh`). The `~` shorthand does not expand in JSON config files.

6. **Restart CoCo** (the hook file is read at startup)

---

**Part 3: With the hook in place (the guardrail)**

7. Re-create the view:
   > "Create a view called `[YOUR_INITIALS]_daily_revenue` in `RETAIL_DB.SALES` that shows total revenue per store per day from DAILY_TRANSACTIONS"

   CoCo creates it. ✓

8. Now try to drop it again with the same prompt as before:
   > "Drop the view `[YOUR_INITIALS]_daily_revenue` from `RETAIL_DB.SALES`"

   **Blocked.** CoCo receives the message: *"DROP, DELETE, and TRUNCATE operations against RETAIL_DB are not permitted..."* and tells you it cannot proceed. The view is still there.

9. Confirm the hook doesn't over-block — try a safe read:
   > "Show me the top 5 stores by revenue from RETAIL_DB.SALES.DAILY_TRANSACTIONS"

   This succeeds normally — the hook only fires on destructive DDL against RETAIL_DB.

**What you just saw:** The same prompt that silently dropped an object in Part 1 is blocked in Part 3. CoCo doesn't know which databases are production — hooks encode that policy and enforce it automatically, without changing how the engineer prompts.

**Key points:**
- `PreToolUse` fires before every tool execution — the most powerful event type
- `Exit 2` blocks the operation and sends your message to the agent as context
- The agent adapts (it doesn't retry the blocked command — it tells you it was stopped)
- Hooks work alongside CoCo's permission system — they add domain-specific guardrails that the AI can't derive from context alone
- Share `hooks.json` and the script directory via git to standardize guardrails across your team

### 4.2 Rules (AGENTS.md / Project Instructions)

**Scenario:** Your retail analytics project has standards — always filter by `is_active = TRUE` for stores, always use fiscal calendar dates, never expose PII from loyalty data.

**Hands-on Exercise:**
1. Create `AGENTS.md` in your project root (the workshop working directory you opened in Section 1.2). In the **Editor tab**, use **File → New File**, paste the contents below, then **Save As** → `AGENTS.md` in your workshop folder:
   ```markdown
   # Grocery Analytics Project Rules

   ## Data Access
   - Always filter stores by `is_active = TRUE` unless explicitly analyzing closed stores
   - Use fiscal calendar (fiscal year starts Feb 1) for all date-based reporting
   - Never include loyalty_member_id, email, phone, or address in query outputs
   - Always include store_id and department in GROUP BY for store-level queries

   ## Code Standards  
   - All SQL files go in the `sql/` directory
   - Python scripts go in `scripts/`
   - Streamlit apps go in `apps/`
   - Always add a comment header with author, date, and purpose

   ## Testing
   - Run `python -m pytest tests/` after any Python changes
   - Validate SQL with `/sql EXPLAIN <query>` before committing
   ```
2. Start a new session and ask: "Query the loyalty table and show me member emails with their purchase history"
3. Observe that the agent refuses to expose PII per the rules
4. Ask: "Show me total spend by store for closed locations" — observe it asks for confirmation since it must override the `is_active` filter rule

---

## Section 5: Task Board, Scheduled Tasks & Wrap-Up (30 min)

### 5.1 Scheduled Tasks with CoCo

**Scenario:** Every Monday morning, your team needs a fresh shrinkage report. Every night, inventory anomalies should be flagged.

CoCo Desktop supports scheduled tasks via **Settings → Scheduled Tasks**:
- Click **Add Scheduled Task**
- Enter a prompt (what you want CoCo to do), a schedule (cron expression or preset like "Every Monday at 6 AM"), and a working directory
- Tasks run automatically and save output to the configured location

**Automation patterns for grocery retail:**

| Task | Schedule | Prompt |
|------|----------|--------|
| Weekly shrinkage report | Every Monday at 6 AM | "Generate weekly shrinkage report for all stores, compare to last week, flag any store above 3% threshold. Save to /reports/weekly_shrinkage.md" |
| Nightly anomaly detection | Every day at 11 PM | "Check for inventory count anomalies in today's data. Flag items with variance >20% from expected. Save alerts to /alerts/inventory_anomalies.md" |
| Friday promo summary | Every Friday at 5 PM | "Summarize this week's active promotions: lift %, incremental revenue, and any underperforming promos. Save to /reports/promo_weekly.md" |

### 5.2 The Task Board — Project Task Tracking

When you give CoCo a complex, multi-step request, it automatically creates an internal **task board** (visible via `Ctrl+D` or `/tasks`) to plan, track, and show progress. This is CoCo's built-in project management — a visual checklist that updates in real-time as work completes.

**How it works:**
- CoCo breaks complex prompts into discrete tasks with statuses: `pending`, `in_progress`, `completed`
- Only one task is `in_progress` at a time
- The board updates live as each step finishes
- Open the fullscreen task view with `Ctrl+D` or click the **Task Board icon** in the toolbar

**Scenario:** You ask CoCo to build an end-to-end inventory health report for the produce department. Watch how it creates and progresses through a task board:

**Hands-on Exercise:**
1. Give CoCo a complex multi-step request:
   > "Build a complete produce department health report for Store 305. Include:
   > - Current stock levels vs par levels
   > - This week's shrinkage rate and trend vs last 4 weeks
   > - Top 10 items by waste (lbs)
   > - Items within 2 days of expiration
   > - A summary recommendation paragraph
   > Save the report as `reports/produce_health_store305.md`"

2. Watch CoCo create its task board — you'll see something like:
   ```
   ☐ Query current stock levels vs par levels for Store 305 produce
   ☐ Calculate weekly shrinkage rate and 4-week trend
   ☐ Identify top 10 items by waste weight
   ☐ Query items approaching expiration (within 2 days)
   ☐ Generate summary recommendation
   ☐ Compile and save final report
   ```

3. Press `Ctrl+D` or click the **Task Board icon** in the toolbar to open the fullscreen task view — observe tasks moving from pending → in_progress → completed

4. Try interrupting mid-task: press `Escape` to stop streaming, then ask "What's left to do?" — CoCo reports remaining tasks

5. Give another complex request to see the board reset:
   > "Now do the same report for dairy, bakery, and deli departments — one file each"
   
   Observe how CoCo creates a task per department.

**Key points:**
- The task board is automatic for complex requests (3+ steps)
- You don't need to create tasks manually — CoCo infers them from your prompt
- Tasks provide visibility into long-running work
- If a session is interrupted, use **File → Resume Session** to restore context — the task state is preserved
- The board helps CoCo stay on track and not skip steps

**When the task board activates:**
| Request Type | Task Board? |
|-------------|-------------|
| "What's the shrinkage rate?" | No — single query |
| "Create a file with this SQL" | No — single action |
| "Build a full report with 5 sections" | Yes — multi-step |
| "Refactor all department queries to use parameters" | Yes — multi-file |
| "Set up a new Streamlit app with 3 pages" | Yes — multi-step |

---

### 5.3 Scheduled Tasks & Automation

**Scenario:** Every Monday morning, your team needs a fresh shrinkage report. Every night, inventory anomalies should be flagged.

Scheduled tasks are configured in **Settings → Scheduled Tasks**. Each task needs:
- A **prompt** describing what CoCo should do
- A **schedule** (choose a preset or enter a cron expression)
- A **working directory** pointing to your project folder

**Automation patterns for grocery retail:**

| Task | Schedule | Prompt |
|------|----------|--------|
| Weekly shrinkage report | Every Monday at 6 AM | "Generate weekly shrinkage report for all stores, compare to last week, flag any store above 3% threshold. Save to /reports/weekly_shrinkage.md" |
| Nightly anomaly detection | Every day at 11 PM | "Check for inventory count anomalies in today's data. Flag items with variance >20% from expected. Save alerts to /alerts/inventory_anomalies.md" |
| Friday promo summary | Every Friday at 5 PM | "Summarize this week's active promotions: lift %, incremental revenue, and any underperforming promos. Save to /reports/promo_weekly.md" |

**Hands-on Exercise:**
1. Open **Settings → Scheduled Tasks** and click **Add Scheduled Task**
2. Configure a task: prompt "List the top 5 departments by shrinkage rate this month", schedule "Run now (one-time)"
3. Run it and observe the output
4. Discuss: What recurring reports would you automate for your stores?

### 5.4 Session Management for Continuity

**Retail scenario:** You're building a complex demand forecasting model across multiple sessions.

| Action | How to Access | Use Case |
|--------|--------------|----------|
| Rename session | Click the session name in the header and type a new name | Name your session for the forecasting project |
| Resume last session | **File → Resume Session** or select from session history panel | Resume where you left off tomorrow |
| Fork session | Click **Fork** in the session menu | Try a different forecasting algorithm without losing current work |
| Rewind | Click **Rewind** in the session menu | The agent went down a wrong path with the model — undo it |
| Compact | Click **Compact** in the session menu | Session is getting long after hours of iteration — summarize and continue |

### 5.5 Wrap-Up & Q&A

**Key Takeaways for Grocery Retail Teams:**
1. **Agent vs Editor** — Use Agent for "build me a shrinkage dashboard," Editor for tweaking SQL manually
2. **Profiles** — Role-based capabilities mean analysts and engineers get tailored experiences
3. **Skills** — Encode your team's domain expertise (promo analysis, inventory audits) as reusable skills
4. **MCP** (Optional) — Connect supplier APIs, weather data, POS systems directly into your workflow
5. **Streamlit** — Build and deploy dashboards directly to Snowflake for all 200 store managers
6. **Hooks** — Protect production data, get notified when long analytics jobs finish
7. **Scheduled tasks** — Automate Monday reports, nightly anomaly detection, weekly promo summaries

**Next Steps:**
- Build a team skill library in a shared GitHub repo
- Set up hooks to protect production tables
- Create an `AGENTS.md` for your analytics project standards
- Schedule your first automated report via **Settings → Scheduled Tasks**

**Resources:**
- In-app help: **Help menu** or ask "What can you do?"
- Skill guide: `/coco-guide`
- Docs: https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli
- Configuration: **Settings** panel in the app

---

## Optional: MCP — Model Context Protocol (30 min)

> **Note:** This section is optional and can be covered if time permits or explored independently after the workshop.

### MCP.1 What is MCP?

- MCP connects external services as tools the agent can call
- Tools follow the pattern: `mcp__<server>__<tool>`
- Configuration: `~/.snowflake/cortex/mcp.json`

**Retail Use Cases:**
- Connect to your **inventory management API** to check real-time stock levels
- Connect to **weather services** to correlate weather with produce demand
- Connect to **POS systems** for real-time transaction data
- Connect to **supplier portals** for delivery schedule lookups

### MCP.2 Adding and Managing MCP Servers

MCP servers are managed via **Settings → MCP** in the CoCo Desktop UI:
- **Add a server:** Click **Add MCP Server**, enter a name, command, arguments, and any required environment variables
- **Remove a server:** Select the server and click **Remove**
- **View active servers:** The MCP panel lists all configured servers and their status


### MCP.3 Using MCP Tools in Practice

**Hands-on Exercise — connect the Open Food Facts MCP server (no API keys required):**

This server connects you to the world's largest open food product database — over 4 million products including Canadian grocery items from Loblaws, Sobeys, Metro, President's Choice, and more. You can look up products by barcode, search by brand/category, and retrieve nutrition data.

1. Find your npx path — in the Agent chat:
   ```
   ! which npx
   ```
   Copy the result (e.g. `/opt/homebrew/bin/npx`).

2. Open **Settings → MCP** and click **Add MCP Server**

3. Fill in the form using the **full npx path** from step 1:
   - **Name**: `openfoodfacts`
   - **Command**: `/opt/homebrew/bin/npx` *(your full path)*
   - **Arguments**: `-y openfoodfacts-mcp`
   - **Environment Variables**: Add one entry:
     - Key: `OFF_USER_AGENT`
     - Value: `cortex-code-workshop/1.0 (your.email@snowflake.com)`

4. Click **Save** — the server should show a green status indicator

5. Test it in the Agent chat:
   > "Search Open Food Facts for Compliments chocolate chip cookies and show me the nutrition info"

   (Compliments is Sobeys' store brand — this should return products with full nutrition data and Nutri-Score ratings.)

6. Try more grocery-relevant queries:
   > "What breakfast cereals in the database have a Nutri-Score of A?"

   > "Look up the product with barcode 0055742528381 and tell me its ingredients and allergens"

**Combine with Snowflake data:**
> "Search Open Food Facts for all organic maple syrup products, then compare the brands found against our supplier list in `RETAIL_DB.INVENTORY.DAILY_STOCK_LEVELS` to see which ones we currently carry."

This demonstrates how an external product database can enrich your internal retail data — useful for verifying nutrition claims, checking allergen info, or identifying new products to stock.

---

**Optional Hands-on Exercise — connect a weather MCP server (no API keys required):**

> **Note:** This weather MCP server uses the US National Weather Service API and only supports **US locations**. If your stores are exclusively in Canada, you may skip this exercise — the Open Food Facts exercise above is the primary MCP demo.

1. Find your npx path — in the Agent chat:
   ```
   ! which npx
   ```
   Copy the result (e.g. `/opt/homebrew/bin/npx`).

2. Open **Settings → MCP** and click **Add MCP Server**

3. Fill in using the **full npx path** from step 1:
   - **Name**: `weather`
   - **Command**: `/opt/homebrew/bin/npx` *(your full path)*
   - **Arguments**: `-y @h1deya/mcp-server-weather`

4. Click **Save** — the server should show a green status indicator

5. Test it in the Agent chat:
   > "Use the weather MCP to get the 3-day forecast for Atlanta, GA"

**Combine with Snowflake data:**
> "Check the weather forecast for Atlanta and Chicago for the next 3 days, then look at `RETAIL_DB.STORES.STORE_DIRECTORY` to find which of our stores are in those cities. Suggest which produce items we should stock more of based on the forecast."

This demonstrates the core MCP value: the agent seamlessly combines a live external data source with your internal Snowflake tables in a single prompt — no custom integration code required.

**Troubleshooting — server fails to start (exit code 1 or "command not found"):**
CoCo Desktop does not inherit your shell PATH, so bare `npx` may not resolve. Use the full absolute path as the **Command** — run `! which npx` in the Agent chat to find it (e.g. `/opt/homebrew/bin/npx`).

Update both MCP servers to use the full path. The `@modelcontextprotocol/server-fetch` npm package has been removed from the registry — use the Python version (`mcp-server-fetch`) instead.

---

## Appendix: Quick Reference Card

```
MODES:       Mode selector in prompt toolbar (Confirm / Plan / Bypass)
FILES:       @path  |  @path$lines  (type @ in chat for file picker)
SKILLS:      /skill-name  (browse via Skills panel in sidebar)
TABLES:      #DB.SCHEMA.TABLE
BASH:        !command
TASK BOARD:  Ctrl+D  |  Task Board icon in toolbar
AGENTS:      Agents panel in sidebar
MCP:         Settings → MCP
CONNECTIONS: Click connection name in status bar
SCHEDULE:    Settings → Scheduled Tasks
SESSION:     Fork / Rewind / Compact / Resume via session menu
CONFIG:      Settings panel in the app
```

## Appendix: Sample Retail Tables Referenced

| Table | Description |
|-------|-------------|
| `RETAIL_DB.SALES.DAILY_TRANSACTIONS` | Store-level daily transaction summaries |
| `RETAIL_DB.SALES.DAILY_ITEM_SALES` | Item-level daily sales (units, revenue) |
| `RETAIL_DB.INVENTORY.DAILY_STOCK_LEVELS` | End-of-day inventory counts by item/store |
| `RETAIL_DB.INVENTORY.SHRINKAGE` | Recorded waste, damage, and theft by department |
| `RETAIL_DB.INVENTORY.EXPIRATION_TRACKING` | Items approaching best-by dates |
| `RETAIL_DB.MARKETING.PROMOTIONS` | Active and historical promotion definitions |
| `RETAIL_DB.STORES.STORE_DIRECTORY` | Store metadata (location, size, format, is_active) |
| `RETAIL_DB.LOYALTY.MEMBER_TRANSACTIONS` | Loyalty program purchase history (PII — protected) |
---

## Workshop Prerequisites

See **[Workshop_Prerequisites.md](Workshop_Prerequisites.md)** for full setup instructions, split into:
- **Snowflake Team Pre-Setup** — environment access (EDO_SANDBOX), AD group provisioning, and sample database setup
- **Attendee Laptop Setup** — CoCo Desktop install and connection config, Node.js, and Python

