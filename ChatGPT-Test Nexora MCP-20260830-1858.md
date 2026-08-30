# Test Nexora MCP

**User:** Anonymous  
**Created:** 8/30/2026 17:39:12  
**Updated:** 8/30/2026 18:26:05  
**Exported:** 8/30/2026 18:58:29  
**Link:** [https://chatgpt.com/c/6a941d66-f0f4-83ee-a039-cc33ef27f139](https://chatgpt.com/c/6a941d66-f0f4-83ee-a039-cc33ef27f139)  

## Prompt:
8/30/2026, 5:39:11 PM

Hello , I have create a MCP named Nexora and configured in this account but now i want to publish it as  public OpenAI MCP but on the official page it said to test it through ChatGPT so now i want you to test it and tell me if it's ready for upload or not @Nexora

## Response:
8/30/2026, 5:39:12 PM

> **Researched quantum computing concepts and tested YouTube transcripts**
>
> 
>
> Worked for 26s

Yep - I tested **Nexora directly**, and I’d say **it is not ready for public OpenAI MCP submission yet**. The good news: the core discovery layer is working. 🚀

### 🧪 What I tested

| Capability | Result | Verdict |
|---|---|---|
| `youtube_search` | Returned relevant videos + IDs + metadata | ✅ **Working** |
| `youtube_video` | Failed on a video returned by search | ❌ **Bug / reliability issue** |
| `youtube_transcript` | Correctly detected unavailable captions | ⚠️ **Working, but needs broader handling** |
| `youtube_find_in_video` | Correctly reported no searchable transcript | ⚠️ **Depends on captions** |
| `youtube_research` | Successfully searched 2 videos and reported 0 transcripts | ⚠️ **Pipeline works, transcript layer doesn't** |

### 🚨 Biggest issue

The most important failure is this:

**Search → Video metadata → Transcript**

`youtube_search` found:

> *Quantum Computers Explained: How Quantum Computing Works*

with a valid YouTube URL/ID.

But immediately calling `youtube_video` on that exact result returned:

> Could not retrieve metadata for video. It may be private or deleted.

That's suspicious because the search endpoint itself just returned the video.

For a public MCP, this kind of cross-tool inconsistency is something I'd fix before submission.

### 🔥 Second major issue: transcript availability

`youtube_research` analyzed 2 videos but got:

```text
videos_with_transcripts: 0
total_evidence_chunks: 0
```

So Nexora's **research/semantic-analysis architecture is functioning**, but its usefulness collapses when captions aren't available.

That's particularly important because your MCP's main selling point is YouTube **research + transcript + video analysis**.

---

## My current verdict

**Public submission: ❌ Not yet**

**Prototype / private testing: ✅ Yes**

I'd currently rate it around **6/10 for submission readiness**.

### Before submitting, I'd specifically test/fix:

1. **Search → Video consistency**
   - Every video returned by `youtube_search` should be consumable by `youtube_video`.

2. **Transcript extraction**
   - Test videos with:
     - manual captions
     - auto-generated captions
     - different languages
     - Shorts
     - long videos
     - videos with no captions

3. **Fallback behavior**
   - If captions aren't available, Nexora should clearly return something like:
     `TRANSCRIPT_UNAVAILABLE`
   - Don't make downstream research tools silently behave as if the video simply contains no relevant information.

4. **Language handling**
   - Your API already exposes `language`, `fallback_language`, and `translate_to`, which is good.
   - This needs real-world testing.

5. **Long-video pipeline**
   - `youtube_find_in_video` is especially important because that's where Nexora can differentiate itself from a basic YouTube search MCP.

6. **Error semantics**
   - Distinguish:
     - video deleted/private
     - captions disabled
     - captions temporarily unavailable
     - extraction failure
     - network/backend failure

### One particularly important thing

Don't judge Nexora based only on whether a random YouTube video has captions. **The MCP should gracefully handle both captioned and non-captioned videos.**

Right now, the individual tools are returning sensible errors, which is good. But the **end-to-end experience isn't reliable enough yet**.

If you want, I can do a **proper Nexora pre-submission audit** next: test all 5 tools against a variety of YouTube videos and give you a **PASS / FAIL report + exact bugs to fix before publishing**.

## Prompt:
8/30/2026, 5:40:29 PM

Test all 5 tools against a variety of YouTube videos

## Response:
8/30/2026, 5:40:29 PM

> **Searching the web**
>
>
> **Searching 6 websites**
>
> 
>
> **Searched and analyzed YouTube tutorials across programming, physics, gaming, and quantum computing**
>
> 
>
> Worked for 1m 25s

Done. I ran a broader **5-tool Nexora stress test** across multiple YouTube categories: programming, physics, Hindi educational content, gaming, quantum computing, and a long-form science video.

## 🧪 Test results

| Nexora tool | Tests | Result |
|---|---:|---|
| `youtube_search` | 5 different topics | 🟢 **PASS** |
| `youtube_video` | 5 real public video IDs returned by search/external YouTube search | 🔴 **FAIL** |
| `youtube_transcript` | 5 videos, EN + HI + fallback | 🔴 **FAIL** |
| `youtube_find_in_video` | 4 videos, specific semantic queries | 🔴 **FAIL** |
| `youtube_research` | 2 multi-video research tasks | 🟡 **PARTIAL** |

### 1. `youtube_search` - 🟢 PASS

This is the strongest part of Nexora right now.

I tested:

- Python
- JavaScript
- Hindi Physics
- Minecraft
- Quantum Computing

It consistently returned structured video results with titles, channels, URLs and IDs. For example, the Python search correctly returned Programming with Mosh's *Python Full Course for Beginners* and its ID `_uQrJ0TkZlc`. 

**Verdict: Production-quality direction.**

---

### 2. `youtube_video` - 🔴 CRITICAL FAIL

This is the biggest problem.

I took **video IDs directly returned by `youtube_search`** and passed them into `youtube_video`.

Example:

```text
youtube_search
    ↓
_uQrJ0TkZlc
    ↓
youtube_video
    ↓
"Could not retrieve metadata..."
```

Same problem occurred with:

- `_uQrJ0TkZlc` - Python
- `W6NZfCO5SIk` - JavaScript
- `PHQctpShl0s` - Hindi Physics
- `Q10_srZ-pbs` - Veritasium

And this isn't just one weird video. The external YouTube search result confirms that `Q10_srZ-pbs` is a real Veritasium video with a substantial public description and chapter list. ([YouTube](https://www.youtube.com/watch?v=Q10_srZ-pbs&utm_source=chatgpt.com))

### 🚨 This needs fixing before submission.

A core invariant should be:

> **Any valid video ID returned by `youtube_search` must be accepted by `youtube_video`.**

Right now that invariant is broken.

---

### 3. `youtube_transcript` - 🔴 FAIL

I tested multiple videos and languages.

Results were consistently:

```text
No captions available
```

Even when the videos are clearly public and searchable.

For example, the Veritasium video has an extensive public description and timestamped chapter information externally, but Nexora's transcript layer couldn't obtain captions. ([YouTube](https://www.youtube.com/watch?v=Q10_srZ-pbs&utm_source=chatgpt.com))

This could mean the **caption extractor itself is broken**, rather than the videos genuinely lacking captions.

And that's an important distinction.

Your tool says:

> "Captions might be disabled by the creator or unavailable..."

But after seeing this happen across essentially every test, I would **not assume the captions are actually unavailable**.

---

### 4. `youtube_find_in_video` - 🔴 FAIL

I tested actual semantic queries such as:

> "What is a variable in Python and how is it used?"

and:

> "principle of least action and fastest descent"

The tool returned:

```text
status: not_found
```

But because `youtube_transcript` is failing first, this result is currently **not a valid test of your semantic search engine**.

The likely pipeline is:

```text
YouTube video
   ↓
Transcript extraction ❌
   ↓
Embedding/BM25 search
   ↓
No matches
```

So I wouldn't blame your Hybrid RRF implementation yet.

**Fix transcript extraction first, then retest this tool.**

---

### 5. `youtube_research` - 🟡 PARTIAL

This one is interesting.

It **successfully performed the overall orchestration**.

For example, I asked it to research Minecraft survival. It discovered **3 videos from different channels**, respected `max_videos_per_channel=1`, and returned structured sources.

That's good. 👍

But:

```text
total_videos_analyzed: 3
videos_with_transcripts: 0
total_evidence_chunks: 0
```

Same thing happened with quantum computing:

```text
total_videos_analyzed: 2
videos_with_transcripts: 0
total_evidence_chunks: 0
```

So the **research orchestrator works**, but its evidence-generation pipeline is effectively dead because transcript acquisition is failing.

---

# 🧠 Overall diagnosis

This is actually pretty clear now:

```text
                    NEXORA

                 ┌─────────────┐
                 │ YouTube     │
                 │ Search      │
                 └──────┬──────┘
                        │
                        │ ✅
                        ▼
                 ┌─────────────┐
                 │ Video       │
                 │ Metadata    │
                 └──────┬──────┘
                        │
                        │ ❌
                        ▼
                 ┌─────────────┐
                 │ Transcript  │
                 │ Extraction  │
                 └──────┬──────┘
                        │
                        │ ❌
             ┌──────────┴──────────┐
             ▼                     ▼
       Find-in-video          Research
             ❌                  🟡
```

## 🚦 Submission verdict

### **❌ DO NOT submit Nexora publicly yet.**

Not because the concept is bad - actually the tool surface is **very solid**.

The problem is that the **core YouTube → transcript pipeline is currently broken**.

I'd put it at:

**Search:** 9/10  
**Tool design:** 9/10  
**Research orchestration:** 7/10  
**Video metadata:** 2/10  
**Transcript:** 2/10  
**Semantic search:** untestable until transcript works  
**Overall submission readiness:** **~4/10**

---

## 🔧 Fix order I'd recommend

Don't randomly patch all five tools. Fix them in this order:

### **P0 - `youtube_video`**

Make this work first.

Test:

```text
youtube_search("Python")
        ↓
video_id
        ↓
youtube_video(video_id)
```

That should be a guaranteed integration test.

### **P0 - `youtube_transcript`**

Then test:

```text
youtube_search
       ↓
youtube_video
       ↓
youtube_transcript
```

You need to support at least:

- manually uploaded captions
- auto-generated captions
- English
- Hindi
- fallback language
- no-caption videos
- Shorts
- long videos

### **P1 - `youtube_find_in_video`**

Only after transcripts work.

Then we can test whether your **FastEmbed + BM25 + RRF** implementation actually finds the correct timestamps.

### **P1 - `youtube_research`**

Finally test:

```text
Search
 → discover videos
 → metadata
 → transcript
 → semantic retrieval
 → evidence chunks
 → claim clustering
 → citations
```

That's the feature that can make Nexora genuinely stand out.

---

### One important positive 🟢

Your **tool API design itself looks quite good**. The five tools have sensible separation:

`search → video → transcript → find-in-video → research`

That's a much better public MCP interface than dumping everything into one giant `research_youtube()` function.

So **I wouldn't redesign Nexora yet**.

I'd fix the underlying extraction pipeline first, then I can run the same test suite again and tell you whether it's actually **OpenAI-public-MCP ready**.

And yes - the fact that YouTube's own external search can resolve these public videos while Nexora can't retrieve their metadata is a strong signal that the problem is **inside Nexora's extraction/backend layer**, not simply "these videos don't exist." ([YouTube](https://www.youtube.com/watch?v=Q10_srZ-pbs&utm_source=chatgpt.com))

---

**Sources:**

- [The Closest We’ve Come to a Theory of Everything - YouTube](https://www.youtube.com/watch?v=Q10_srZ-pbs&utm_source=chatgpt.com)



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)