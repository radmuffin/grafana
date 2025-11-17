# Documentation and Bug Triage Contribution Guide

This guide covers two ways to contribute to Grafana without writing code: **documentation improvements** and **bug triage/reproduction**.

---

## 📚 Part 1: Documentation Contributions

### Overview

Documentation is critical for Grafana's success. It helps users understand features, troubleshoot issues, and get the most from the platform.

### Documentation Structure

Grafana uses the **Writers' Toolkit** for documentation management:

- **Repository:** https://github.com/grafana/writers-toolkit
- **Main Docs:** https://grafana.com/docs/writers-toolkit/
- **Contributing Guide:** Available in the writers-toolkit repo

Key resources in this repo:

- Contribution guidelines
- Structure guidelines
- Writing and style guide
- Build and review procedures

### Types of Documentation Work

#### 1. **Feature Documentation** (Most Valuable)

When a new feature is released or a feature has incomplete docs:

- Write step-by-step guides
- Add examples and screenshots
- Document configuration options
- Create troubleshooting sections

**Finding opportunities:**

- Check recent GitHub releases and changelogs
- Search issues for `type/docs` label
- Look for new areas marked as "needs documentation"

#### 2. **API Documentation**

- Document REST API endpoints
- Add parameter descriptions
- Include request/response examples
- Note rate limits and authentication

#### 3. **Plugin/Datasource Documentation**

- Setup guides for integrations
- Configuration walkthroughs
- Query language examples
- Common troubleshooting scenarios

#### 4. **Style Consistency**

- Update existing docs to match style guide
- Fix outdated examples
- Improve clarity of existing content
- Add missing cross-references

### Documentation Workflow

1. **Find an opportunity:**
   - Check `https://github.com/grafana/grafana/issues` for `type/docs` labels
   - Look for issues marked as "help wanted"
   - Check the Writers' Toolkit repo for open issues

2. **Understand the scope:**
   - Read the related issue thoroughly
   - Review existing related documentation
   - Identify what's missing or unclear

3. **Follow the style guide:**
   - Use the Writers' Toolkit style guide (not just the AGENTS.md)
   - Stick to present simple tense
   - Use active voice
   - Use second person ("you")
   - Use sentence case for headings

4. **Create the documentation:**
   - Write in Markdown
   - Use descriptive headings
   - Include code examples where relevant
   - Add internal links to related content
   - Include prerequisites if needed

5. **Submit for review:**
   - Create a PR against the docs repository
   - Reference the related GitHub issue
   - Request review from maintainers

### Good Documentation Issue Examples

Look for issues that:

- Have clear descriptions of what's missing
- Explain the impact on users
- Provide context on the feature
- Mark users' pain points

---

## 🐛 Part 2: Bug Triage and Reproduction

### What is Bug Triage?

Bug triage is the process of:

1. **Categorizing** issues (type, area, severity)
2. **Reproducing** reported bugs to confirm they exist
3. **Clarifying** issue details for developers
4. **Prioritizing** which bugs should be fixed first
5. **Filtering** duplicate issues

While Grafana has automation for triage, **human verification is still valuable** because:

- Automation can't always understand user context
- Some bugs need manual reproduction to confirm
- Unclear reports need clarification
- Edge cases might be missed

### Key Triage Labels (Reference)

From `contribute/ISSUE_TRIAGE.md`:

| Label                                  | Meaning                                   |
| -------------------------------------- | ----------------------------------------- |
| `type/bug`                             | Confirmed bug (after reproduction)        |
| `needs investigation`                  | Unconfirmed - needs manual testing        |
| `type/works-as-intended`               | Bug works by design                       |
| `area/*`                               | Functional area (explore, alerting, etc.) |
| `datasource/*`                         | Related to specific datasource            |
| `help wanted`                          | Community help appreciated                |
| `prio/high`, `prio/medium`, `prio/low` | Priority level                            |
| `effort/small`, `effort/medium`        | Estimated fix complexity                  |

### How to Reproduce a Bug

#### Step 1: Gather Information

Check the issue for:

- **Grafana version** - Are you using same version?
- **Operating system** - Windows, Linux, Mac?
- **Browser** - Chrome, Firefox, Safari?
- **Data source type** - Prometheus, Loki, Tempo, etc.?
- **Reproduction steps** - Are they clear and complete?
- **Expected vs actual behavior** - Is it well-defined?

#### Step 2: Set Up Your Environment

Based on the bug, you might need:

- Same Grafana version as reported
- Same datasource configured
- Test data or sample configuration
- Specific browser or OS

**Common setups:**

- Docker Compose (provided in devenv/)
- Local development build (via `make run`)
- Grafana Cloud (for cloud-specific issues)
- Play.grafana.com (public demo)

#### Step 3: Follow Reproduction Steps Exactly

- Don't skip steps
- Use exact values mentioned
- Take screenshots of each step
- Note any differences from instructions
- Try different browsers/devices if applicable

#### Step 4: Document Your Findings

Write a clear comment with:

- **Confirmed:** "I can reproduce this with Grafana X.X.X on Y OS"
- **Cannot reproduce:** "I followed the steps but X did not happen"
- **Additional info:** "I found this also happens when..."
- **Screenshots/videos:** Attach visual evidence
- **Environment:** Exact versions you tested with

### Common Bug Triage Scenarios

#### Scenario 1: Clear Reproduction Steps, Not Reproduced Yet

```
Reporter says: "Click button X, then Y happens"
Your task: Reproduce exactly
Your comment: "Reproduced on Grafana 11.3 on Mac OS with Prometheus datasource.
Steps: [exact steps]. Result: Button does not respond on first click."
```

#### Scenario 2: Vague Report Needing Clarification

```
Reporter says: "Something is broken"
Your task: Ask for more info
Your comment: "Thanks for reporting. To help us reproduce this, could you provide:
- What version of Grafana?
- What are the exact steps to reproduce?
- What browser and OS?
- Do you have error logs?"
```

#### Scenario 3: Likely Duplicate

```
Reporter reports issue that seems similar to #12345
Your task: Check if it's a duplicate
Your comment: "This may be a duplicate of #12345. Could you confirm if that
issue describes the same behavior you're experiencing?"
```

#### Scenario 4: Works as Intended

```
Reporter says: "Feature X should do Y"
Your research: Feature is designed to do Z
Your comment: "This is working as designed. Feature X is intended to [explain].
If you'd like it to work differently, consider opening a feature request instead."
```

### Issues Perfect for Bug Triage Practice

Look for issues marked with:

- `needs investigation` - These explicitly need manual testing
- `type/bug` AND `0 comments` - Confirmed but not yet investigated
- Recent bugs without comments - Fresh issues needing triage

**Example issues from earlier search:**

- #106197 (Traces with cycle shows JavaScript error)
- #94362 (Heatmap data link not visible)
- #106278 (DataLink double quotes issue)
- #96567 (Time picker collapses too aggressively)

### Step-by-Step: Practicing Bug Triage

Here's a practical workflow:

1. **Find a "needs investigation" issue:**

   ```
   Search: is:open label:"needs investigation" -is:assigned
   ```

2. **Read the full report carefully:**
   - Note all environment details
   - Check for reproduction steps
   - Look for screenshots/error messages
   - Read all comments

3. **Set up your environment:**
   - Use `make run` to start Grafana locally
   - Configure the mentioned datasource
   - Create test data if needed

4. **Reproduce the bug:**
   - Follow the exact steps
   - Try multiple times (could be flaky)
   - Try on different browser if mentioned
   - Check browser console for errors (F12)

5. **Comment with findings:**

   ```
   I tested this on Grafana v11.3 with [datasource] on [OS/Browser].

   Steps I followed:
   1. [step]
   2. [step]

   Result: ✓ Reproduced / ✗ Could not reproduce

   [Additional observations]

   Error logs: [paste any errors from browser console]
   ```

6. **Add labels if you have permissions:**
   - If reproduced: Add `type/bug`
   - Add relevant `area/*` label
   - Add `effort/small` if it seems like an easy fix

### Tools for Bug Investigation

**Browser Developer Tools:**

- F12 or Right-click → Inspect
- Console tab shows JavaScript errors
- Network tab shows API calls
- Application tab shows local storage

**Grafana Logs:**

- Docker: `docker logs <container>`
- Local: Check terminal where you ran `make run`
- Grafana UI: Settings → About → Logs (limited)

**Query Inspector (Data Sources):**

- Click "Inspect" in query builder
- Shows raw request/response
- Helps debug datasource issues

### Creating Test Data

For different datasources:

**Prometheus:**

```bash
# Use included testdata datasource
# Or set up local Prometheus with docker-compose
```

**Loki:**

```bash
# Docker compose in devenv/docker/
# Or use Grafana Cloud Loki
```

**Tempo (Traces):**

```bash
# Example in issue #106197 shows how to push test traces
curl -X POST -H 'Content-Type: application/json' \
  http://localhost:4318/v1/traces -d @trace.json
```

---

## 🚀 Getting Started: Your First Contribution

### Option A: First Documentation Task

1. Go to `contribute/documentation/README.md`
2. Follow the Writers' Toolkit link
3. Find a `type/docs` issue on GitHub with "help wanted"
4. Fork the writers-toolkit repo
5. Create a simple section in an existing doc
6. Submit PR with reference to the issue

**Good first doc task:** Adding an example or clarifying existing documentation

### Option B: First Triage Task

1. Search GitHub: `is:open label:"needs investigation" -is:assigned`
2. Pick an issue with clear reproduction steps
3. Set up Grafana locally with `make run`
4. Follow the reproduction steps exactly
5. Leave a comment with your findings (whether you reproduced it or not)
6. That's it! You've done triage.

**Good first triage task:** Something with a "Calculate from data" option or simple UI interaction

---

## 📋 Triage & Documentation Checklist

### Before You Start Triage

- [ ] I understand the issue is about a specific feature, not a question
- [ ] I have access to the same version of Grafana
- [ ] I have the same datasource/environment available
- [ ] I can follow the reproduction steps

### Before You Submit Triage Comment

- [ ] I followed the steps exactly as written
- [ ] I noted my Grafana version, OS, and browser
- [ ] I either reproduced or clearly explained why I couldn't
- [ ] I attached screenshots if helpful
- [ ] I checked browser console for errors
- [ ] My comment is clear and actionable

### Before You Submit Documentation

- [ ] I read the Writers' Toolkit style guide
- [ ] I used present simple tense throughout
- [ ] I used active voice
- [ ] I added relevant code examples
- [ ] I included prerequisites
- [ ] I linked to related content
- [ ] I followed Markdown best practices
- [ ] I added a clear title that explains the section

---

## 🔗 Useful Links

- **Triage Guide:** `/home/radmunix/grafana/contribute/ISSUE_TRIAGE.md`
- **Writers' Toolkit:** https://github.com/grafana/writers-toolkit
- **Style Guide:** `/home/radmunix/grafana/AGENTS.md`
- **Bug Search:** https://github.com/grafana/grafana/issues?q=label%3A%22needs+investigation%22+is%3Aopen
- **Docs Issues:** https://github.com/grafana/grafana/issues?q=label%3A%22type%2Fdocs%22+is%3Aopen

---

## 💡 Tips for Success

**For Documentation:**

- Start small - improve one section first
- Use real examples from your own experience
- Get feedback early from maintainers
- Link between related docs
- Keep tone friendly and encouraging

**For Triage:**

- Be respectful to reporters - they found a real issue
- If you can't reproduce, ask clarifying questions
- Provide all your environment info
- Link to similar issues you find
- Remember: confirming a bug is valuable even if you don't fix it

Happy contributing! 🎉
