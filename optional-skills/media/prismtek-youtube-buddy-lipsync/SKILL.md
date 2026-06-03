---
name: prismtek-youtube-buddy-lipsync
description: Optional Hermes-compatible Prismtek/Buddy talking-host overlay workflow.
version: 1.1.0
author: Prismtek
license: MIT
platforms: [macos, linux]
metadata:
  hermes:
    tags: [youtube, video, ffmpeg, imagemagick, pixel-art, vtuber, buddy, prismtek]
    related_skills: [youtube-content, media-rendering, buddy-appearance]
  canonical_runtime: codysumpter-cloud/buddy-agent#18
  operator_reference: codysumpter-cloud/buddy-brain#315
  app_contract: codysumpter-cloud/prismtek-apps#138
  knowledge_reference: codysumpter-cloud/knowledge-vault#4
---

# Prismtek YouTube Buddy Lip-Sync

This optional skill documents the Hermes-compatible surface for the Prismtek/Buddy talking-host lip-sync workflow.

The canonical runnable package lives in `buddy-agent`. This fork keeps a clean optional-skill landing zone so the upstream Hermes PR can be treated as an intentional optional contribution rather than the only implementation.

## What the Workflow Does

- Generates small pixel-art mouth states from a clean Buddy avatar.
- Uses FFmpeg/ffprobe to estimate narration energy.
- Renders a proof clip with a Buddy overlay.
- Produces a JSON receipt for review.

## Required Tools

```bash
command -v magick
command -v ffmpeg
command -v ffprobe
python3 --version
```

## Runtime Owner

Use `codysumpter-cloud/buddy-agent#18` as the source of truth for runnable scripts and tests.

## Related PRs

- Runtime: `codysumpter-cloud/buddy-agent#18`
- Operator: `codysumpter-cloud/buddy-brain#315`
- Omni/local device: `codysumpter-cloud/omni-buddy#11`
- App contract: `codysumpter-cloud/prismtek-apps#138`
- Knowledge vault: `codysumpter-cloud/knowledge-vault#4`
- Upstream context: `NousResearch/hermes-agent#26463`
