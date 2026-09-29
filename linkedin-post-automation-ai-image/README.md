# LinkedIn Post Automation with AI Image

n8n workflow that generates and publishes **3 LinkedIn posts per week** (Monday, Wednesday, Friday), each paired with an AI-generated image. Topics are pulled from a Google Sheets content calendar and marked as published after each post goes live.

- **Workflow name:** Abdullah Mustafa (Linkedin Post Automations with AI Image)
- **File:** `Abdullah Mustafa(Linkedin Post Automations with AI Image).json`
- **Status:** Active
- **Execution order:** v1

---

## How It Works

1. A **schedule trigger** fires three times a week.
2. The workflow reads unused topics from a Google Sheet.
3. It picks up to 3 topics and assigns each to a posting slot.
4. For every topic it:
   - Waits until the slot time.
   - Generates the post copy with an LLM agent.
   - Formats the text (Unicode bold, markdown cleanup).
   - Generates a matching image prompt and then the image.
   - Publishes the post to LinkedIn with the image.
   - Marks the topic as **Published** in the sheet.
5. If fewer than 3 topics are available, it emails an alert.

## Schedule

| Slot | Day | Time |
|------|-----|------|
| 1 | Monday | 10:00 AM |
| 2 | Wednesday | 3:00 PM |
| 3 | Friday | 3:00 PM |

The trigger (`shudelue`) runs the pipeline on Monday 08:00, Wednesday 15:00, and Friday 15:00.

---

## Node Flow

```
Schedule Trigger (shudelue)
  -> Read Topics (Google Sheets)
  -> Prepare Topics (Code)
       ├─ Has Topic 1? ─> Wait until Mon 10AM ─> Generate Post 1 ─> Format Post 1
       │     ─> Message a model ─> Code in JavaScript ─> Generate an image1
       │     ─> Publish Post 1 ─> Mark Topic 1 Published
       ├─ Has Topic 2? ─> Wait until Wed 3PM ─> Generate Post 2 ─> Format Post 2
       │     ─> Message a model1 ─> Code in JavaScript1 ─> Generate an image
       │     ─> Publish Post 2 ─> Mark Topic 2 Published
       ├─ Has Topic 3? ─> Wait until Fri 3PM ─> Generate Post 3 ─> Format Post 3
       │     ─> Message a model2 ─> Code in JavaScript2 ─> Generate an image2
       │     ─> Publish Post 3 ─> Mark Topic 3 Published
       └─ Fewer Than 3 Topics? ─> Notify Missing Topics (Gmail)
```

Each of the three **Generate Post** agents is backed by a Google Gemini chat model.

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| `shudelue` | Schedule Trigger | Runs Mon/Wed/Fri |
| `Read Topics` | Google Sheets | Reads the Content Calendar sheet |
| `Prepare Topics` | Code | Filters unused topics, selects 3, assigns slots |
| `Has Topic 1?` / `2?` / `3?` | IF | Routes each available topic down its branch |
| `Fewer Than 3 Topics?` | IF | Detects when fewer than 3 topics exist |
| `Wait until Mon 10AM` / `Wed 3PM` / `Fri 3PM` | Wait | Delays until the slot time |
| `Generate Post 1/2/3` | LangChain Agent | Writes the LinkedIn post copy |
| `Google Gemini Chat Model` / `1` / `2` | Gemini Chat Model | LLM backing each agent |
| `Format Post 1/2/3` | Code | Converts `**bold**` to Unicode bold, strips markdown |
| `Message a model` / `1` / `2` | Google Gemini | Builds an image prompt from the post |
| `Code in JavaScript` / `1` / `2` | Code | Parses the returned JSON into `final_image_prompt` |
| `Generate an image1` / `Generate an image` / `Generate an image2` | Alibaba Cloud (Qwen) | Generates the image (`wan2.7-image-pro`) |
| `Publish Post 1/2/3` | LinkedIn | Publishes the post as PUBLIC with the image |
| `Mark Topic 1/2/3 Published` | Google Sheets | Sets `Status = Published` for the row |
| `Notify Missing Topics` | Gmail | Emails when fewer than 3 topics are available |

---

## Google Sheet

- **Document:** `content_calendar_500_topics_mixed`
- **Sheet / tab:** `Content Calendar`

Expected columns:

| Column | Description |
|--------|-------------|
| `Topic` | Post topic/title (required) |
| `Status` | `Published` is treated as used; anything else is available |
| `Slot` | Reserved for slot label |
| `GeneratedPost` | Reserved for generated copy |
| `ScheduledFor` | Reserved for scheduled timestamp |
| `PublishedAt` | Reserved for publish timestamp |
| `LinkedInPostId` | Reserved for LinkedIn post id |
| `urn` | Reserved for LinkedIn URN |
| `row_number` | Internal row index used for updates |

`Prepare Topics` selects the first 3 rows where `Topic` is non-empty and `Status != "published"`.

---

## Credentials Required

| Credential | Used By |
|------------|---------|
| Google Sheets OAuth2 | Read Topics, Mark Topic * Published |
| LinkedIn OAuth2 | Publish Post 1/2/3 |
| Google Gemini (PaLM) API | Gemini Chat Models, Message a model * |
| Alibaba Cloud / Qwen | Generate an image * |
| Gmail OAuth2 | Notify Missing Topics |

## Models Used

- **Text:** Google Gemini (`gemini-3-flash-preview`, `gemini-3.6-flash`)
- **Image:** Alibaba Cloud Qwen `wan2.7-image-pro`

---

## Alerts

If fewer than 3 topics are available, `Notify Missing Topics` sends an email to `mustafaabdullah22002@gmail.com` stating how many posts will be skipped and asking for more topics.

---

## Notes / Known Quirks

- `Mark Topic 2 Published` and `Mark Topic 3 Published` reference `row1` from `Prepare Topics` instead of `row2` / `row3`. They may update the wrong row.
- Those same nodes set `Topic` to a literal `"="`, which can blank out the topic cell.
- `Wait until Mon 10AM` uses a hardcoded date (`2026-08-04T01:26:00`), while the Wednesday/Friday waits compute their time dynamically from `$now`.
- The LinkedIn publishing account is `kkeQ9hSCxp` (`person` field).

## Setup

1. Import the JSON into n8n.
2. Attach valid credentials for Google Sheets, LinkedIn, Google Gemini, Alibaba Cloud (Qwen), and Gmail.
3. Point `Read Topics` at your own content calendar spreadsheet and tab.
4. Update the `person` field in the Publish nodes to your LinkedIn profile.
5. Add topics with `Status != Published` and activate the workflow.
