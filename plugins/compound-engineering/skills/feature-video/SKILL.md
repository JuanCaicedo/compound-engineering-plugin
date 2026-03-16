---
name: feature-video
description: Record a video walkthrough of a feature and add it to the MR description
argument-hint: "[MR number or 'current'] [optional: base URL, default localhost:3000]"
---

# Feature Video Walkthrough

<command_purpose>Record a video walkthrough demonstrating a feature, upload it, and add it to the PR description.</command_purpose>

## Introduction

<role>Developer Relations Engineer creating feature demo videos</role>

This command creates professional video walkthroughs of features for PR documentation:
- Records browser interactions using agent-browser CLI
- Demonstrates the complete user flow
- Uploads the video for easy sharing
- Updates the MR description with an embedded video

## Prerequisites

<requirements>
- Local development server running (e.g., `bin/dev`, `rails server`)
- agent-browser CLI installed
- Git repository with an MR to document
- `ffmpeg` installed (for video conversion)
- `rclone` configured (optional, for cloud upload - see rclone skill)
- Public R2 base URL known (for example, `https://<public-domain>.r2.dev`)
</requirements>

## Setup

**Check installation:**
```bash
command -v agent-browser >/dev/null 2>&1 && echo "Installed" || echo "NOT INSTALLED"
```

**Install if needed:**
```bash
npm install -g agent-browser && agent-browser install
```

See the `agent-browser` skill for detailed usage.

## Main Tasks

### 1. Parse Arguments

<parse_args>

**Arguments:** $ARGUMENTS

Parse the input:
- First argument: MR number or "current" (defaults to current branch's MR)
- Second argument: Base URL (defaults to `http://localhost:3000`)

```bash
# Get MR number for current branch if needed
glab mr view -F json | jq '.iid'
```

</parse_args>

### 2. Gather Feature Context

<gather_context>

**Get MR details:**
```bash
glab mr view [number] -F json
```

**Get changed files:**
```bash
glab mr diff [number] --name-only
```

**Map files to testable routes** (same as playwright-test):

| File Pattern | Route(s) |
|-------------|----------|
| `app/views/users/*` | `/users`, `/users/:id`, `/users/new` |
| `app/controllers/settings_controller.rb` | `/settings` |
| `app/javascript/controllers/*_controller.js` | Pages using that Stimulus controller |
| `app/components/*_component.rb` | Pages rendering that component |

</gather_context>

### 3. Plan the Video Flow

<plan_flow>

Before recording, create a shot list:

1. **Opening shot**: Homepage or starting point (2-3 seconds)
2. **Navigation**: How user gets to the feature
3. **Feature demonstration**: Core functionality (main focus)
4. **Edge cases**: Error states, validation, etc. (if applicable)
5. **Success state**: Completed action/result

Ask user to confirm or adjust the flow:

```markdown
**Proposed Video Flow**

Based on MR ![number]: [title]

1. Start at: /[starting-route]
2. Navigate to: /[feature-route]
3. Demonstrate:
   - [Action 1]
   - [Action 2]
   - [Action 3]
4. Show result: [success state]

Estimated duration: ~[X] seconds

Does this look right?
1. Yes, start recording
2. Modify the flow (describe changes)
3. Add specific interactions to demonstrate
```

</plan_flow>

### 4. Setup Video Recording

<setup_recording>

**Create videos directory:**
```bash
mkdir -p tmp/videos
```

**Recording approach: Use browser screenshots as frames**

agent-browser captures screenshots at key moments, then combine into video using ffmpeg:

```bash
ffmpeg -framerate 2 -pattern_type glob -i 'tmp/screenshots/*.png' -vf "scale=1280:-1" tmp/videos/feature-demo.gif
```

</setup_recording>

### 5. Record the Walkthrough

<record_walkthrough>

Execute the planned flow, capturing each step:

**Step 1: Navigate to starting point**
```bash
agent-browser open "[base-url]/[start-route]"
agent-browser wait 2000
agent-browser screenshot tmp/screenshots/01-start.png
```

**Step 2: Perform navigation/interactions**
```bash
agent-browser snapshot -i  # Get refs
agent-browser click @e1    # Click navigation element
agent-browser wait 1000
agent-browser screenshot tmp/screenshots/02-navigate.png
```

**Step 3: Demonstrate feature**
```bash
agent-browser snapshot -i  # Get refs for feature elements
agent-browser click @e2    # Click feature element
agent-browser wait 1000
agent-browser screenshot tmp/screenshots/03-feature.png
```

**Step 4: Capture result**
```bash
agent-browser wait 2000
agent-browser screenshot tmp/screenshots/04-result.png
```

**Create video/GIF from screenshots:**

```bash
# Create directories
mkdir -p tmp/videos tmp/screenshots

# Create MP4 video (RECOMMENDED - better quality, smaller size)
# -framerate 0.5 = 2 seconds per frame (slower playback)
# -framerate 1 = 1 second per frame
ffmpeg -y -framerate 0.5 -pattern_type glob -i 'tmp/screenshots/*.png' \
  -c:v libx264 -pix_fmt yuv420p -vf "scale=1280:-2" \
  tmp/videos/feature-demo.mp4

# Create low-quality GIF for preview (small file, for GitLab embed)
ffmpeg -y -framerate 0.5 -pattern_type glob -i 'tmp/screenshots/*.png' \
  -vf "scale=640:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=128[p];[s1][p]paletteuse" \
  -loop 0 tmp/videos/feature-demo-preview.gif
```

**Note:**
- The `-2` in MP4 scale ensures height is divisible by 2 (required for H.264)
- Preview GIF uses 640px width and 128 colors to keep file size small (~100-200KB)

</record_walkthrough>

### 6. Upload the Video

<upload_video>

**Upload with rclone:**

```bash
# Check rclone is configured
rclone listremotes

# Set your public base URL (NO trailing slash)
PUBLIC_BASE_URL="https://<your-public-r2-domain>.r2.dev"

# Upload video, preview GIF, and screenshots to cloud storage
# Use --s3-no-check-bucket to avoid permission errors
rclone copy tmp/videos/ r2:kieran-claude/pr-videos/pr-[number]/ --s3-no-check-bucket --progress
rclone copy tmp/screenshots/ r2:kieran-claude/pr-videos/pr-[number]/screenshots/ --s3-no-check-bucket --progress

# List uploaded files
rclone ls r2:kieran-claude/pr-videos/pr-[number]/

# Build and validate public URLs BEFORE updating PR
VIDEO_URL="$PUBLIC_BASE_URL/pr-videos/pr-[number]/feature-demo.mp4"
PREVIEW_URL="$PUBLIC_BASE_URL/pr-videos/pr-[number]/feature-demo-preview.gif"

curl -I "$VIDEO_URL"
curl -I "$PREVIEW_URL"

# Require HTTP 200 for both URLs; stop if either fails
curl -I "$VIDEO_URL" | head -n 1 | grep -q ' 200 ' || exit 1
curl -I "$PREVIEW_URL" | head -n 1 | grep -q ' 200 ' || exit 1
```

</upload_video>

### 7. Update MR Description

<update_mr>

**Get current MR description:**
```bash
glab mr view [number] -F json | jq -r '.description'
```

**Add video section to MR description:**

If the MR already has a video section, replace it. Otherwise, append:

**IMPORTANT:** GitLab cannot embed external MP4s directly. Use a clickable GIF that links to the video:

```markdown
## Demo

[![Feature Demo]([preview-gif-url])]([video-mp4-url])

*Click to view full video*
```

Example:
```markdown
[![Feature Demo](https://<your-public-r2-domain>.r2.dev/pr-videos/pr-137/feature-demo-preview.gif)](https://<your-public-r2-domain>.r2.dev/pr-videos/pr-137/feature-demo.mp4)
```

**Update the MR:**
```bash
glab mr update [number] --description "[updated description with video section]"
```

**Or add as a comment if preferred:**
```bash
glab mr note [number] --message "## Feature Demo

![Demo]([video-url])

_Automated walkthrough of the changes in this MR_"
```

</update_mr>

### 8. Cleanup

<cleanup>

```bash
# Optional: Clean up screenshots
rm -rf tmp/screenshots

# Keep videos for reference
echo "Video retained at: tmp/videos/feature-demo.gif"
```

</cleanup>

### 9. Summary

<summary>

Present completion summary:

```markdown
## Feature Video Complete

**MR:** ![number] - [title]
**Video:** [url or local path]
**Duration:** ~[X] seconds
**Format:** [GIF/MP4]

### Shots Captured
1. [Starting point] - [description]
2. [Navigation] - [description]
3. [Feature demo] - [description]
4. [Result] - [description]

### MR Updated
- [x] Video section added to MR description
- [ ] Ready for review

**Next steps:**
- Review the video to ensure it accurately demonstrates the feature
- Share with reviewers for context
```

</summary>

## Quick Usage Examples

```bash
# Record video for current branch's MR
/feature-video

# Record video for specific MR
/feature-video 847

# Record with custom base URL
/feature-video 847 http://localhost:5000

# Record for staging environment
/feature-video current https://staging.example.com
```

## Tips

- **Keep it short**: 10-30 seconds is ideal for MR demos
- **Focus on the change**: Don't include unrelated UI
- **Show before/after**: If fixing a bug, show the broken state first (if possible)
- **Annotate if needed**: Add text overlays for complex features
