# 🧠 Multi-Agent AI Content Pipeline — System Architecture

## Overview

This architecture represents a multi-agent content automation workflow built in **n8n**.  
It orchestrates AI-driven research, prompt engineering, content generation, and submission for human review — all in a modular, API-first design.

---

## 🎯 Core Objectives
- Automate the discovery of **trending AI Automation topics** from live data sources (Google Trends + YouTube).
- Generate **AI-assisted prompts** for multi-format content (blogs & videos).
- Produce **SEO-ready blog posts** and **AI-generated videos** automatically.
- Submit final content to **Google Sheets** for review, tracking, and notification.

---

## 🧩 System Architecture

```flow
st=>start: Trigger (Manual / Schedule)
init=>operation: InitPayload
split=>operation: SourceSplitter
route=>condition: RouteBySource?
gt=>operation: HTTP Request\nGoogle Trends (RSS)
yt=>operation: HTTP Request\nYouTube Data API
parseGT=>operation: ParseTrendsTopics
parseYT=>operation: ParseYouTubeVideos
merge=>operation: MergeSources
dedupe=>operation: DeduplicateAndRank
expand=>operation: ExpandTopics
batch=>operation: SplitInBatches (size=1)
prompt=>operation: PromptGenerator\n(OpenAI: Message a model)
parsePrompt=>operation: ParsePrompts
blogCreate=>operation: BlogCreator\n(OpenAI)
parseBlog=>operation: ParseBlogOutput
addKey=>operation: AddMergeKey (Set merge_key)
sheetAppend=>operation: SheetAppend_ContentRow (Google Sheets)
mergeFallback=>operation: MergeManualFallback
capture=>operation: CaptureSheetRow
videoSubmit=>operation: VideoSubmit\n(HTTP Request placeholder)
poll=>operation: PollVideoStatus (Wait + GET)
fetch=>operation: FetchVideoResult
collate=>operation: CollateContent (Function)
sheetUpdate=>operation: SheetUpdateRow (Google Sheets)
notify=>operation: NotifyReviewer (Slack/Email)
end=>end: Done

st->init->split->route
route(yes)->gt->parseGT->merge
route(no)->yt->parseYT->merge
merge->dedupe->expand->batch->prompt->parsePrompt->blogCreate->parseBlog->addKey->sheetAppend
sheetAppend->mergeFallback->capture->videoSubmit->poll
poll(yes)->fetch->collate->sheetUpdate->notify->end
poll(no)->poll

```



------

## ⚙️ Component Summary

| Agent                       | Role                      | Key Nodes                                                    | APIs / Tools                        |
| --------------------------- | ------------------------- | ------------------------------------------------------------ | ----------------------------------- |
| **ContentResearch Agent**   | Fetch trending AI topics  | `HTTP Request (Google Trends)`, `HTTP Request (YouTube API)`, `ParseTrendsTopics`, `ParseYouTubeVideos`, `MergeSources` | Google Trends RSS, YouTube Data API |
| **Prompt Agent**            | Generate creative prompts | `SplitInBatches`, `OpenAI (Message a Model)`, `ParsePrompts` | OpenAI GPT-4                        |
| **ContentCreator Agent**    | Create blog & video       | `OpenAI (Blog Creator)`, `HTTP Request (Video API)`, `ParseBlogOutput`, `CollateContent` | OpenAI, Runway/Gemini               |
| **ContentSubmission Agent** | Review & tracking         | `Google Sheets Append`, `CaptureSheetRow`, `SheetUpdateRow`, `NotifyReviewer` | Google Sheets API, Slack/Gmail      |

------

## 🔄 Data Flow

1. **Research Phase:** Fetches top AI automation topics via Google Trends & YouTube.
2. **Prompt Generation:** Uses OpenAI to build detailed prompts.
3. **Content Creation:** Generates blogs & (mock) videos.
4. **Submission:** Logs entries to Google Sheets for human review.
5. **Notification:** Sends alert when new content is ready.

------

## 🧱 Extensibility

- Swap OpenAI with Gemini API (drop-in replacement).
- Add translation/localization agent.
- Extend review logic via Slack approval threads or Notion API.

------

## 🧠 Error Handling

- Each HTTP call wrapped with fallback checks.
- Dedupe ensures no duplicate topics are processed.
- Capture nodes track execution context for reliable sheet row updates.

------

## 📈 Outcome

> The system simulates a complete AI-driven content operation pipeline:
>  from data mining to creation to publishing — all orchestrated through n8n.

------

