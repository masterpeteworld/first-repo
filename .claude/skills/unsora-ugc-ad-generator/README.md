# UGC Ad Generator Skill

An end-to-end UGC video ad generation skill for general AI agents, including Claude Code, Codex, Claude, OpenClaw, Hermes, and similar agent runtimes.

The skill takes a product link, product image, text description, or combination of inputs, then:

- Runs market and voice-of-customer research before ideation.
- Proposes up to 20 research-grounded UGC ad concepts.
- Writes authentic UGC first-frame prompts.
- Writes Seedance 2.0 video prompts.
- Generates 1 to 20 ad variations through the Unsora MCP.
- Chains and stitches ads longer than 15 seconds with ffmpeg.
- Optionally prepares Instagram or TikTok posting through Unsora.

## Contents

- `SKILL.md` - the main skill instructions.
- `references/market_research_rules.md` - research process, PullPush API usage, and Voice-of-Customer brief format.
- `references/first_frame_prompt_rules.md` - first-frame image prompt rules.
- `references/seedance_prompt_rules.md` - Seedance 2.0 video prompt rules.
- `references/unsora_and_ffmpeg.md` - Unsora MCP parameters plus ffmpeg extraction and stitching.

## Agent Compatibility

This repository is written as a portable skill package. It can be adapted into any agent environment that can read Markdown instructions and has access to the required tools.

Known target agent environments include:

- Claude Code
- Codex
- Claude
- OpenClaw
- Hermes

## Requirements

The agent runtime should provide access to:

- Unsora MCP tools for image/video generation and posting.
- Web search/fetch tools for product and market research.
- PullPush API access for Reddit post/comment research.
- `ffmpeg` for frame extraction and video stitching.

## Safety

The skill is designed to pause before paid generation steps and before posting to social accounts.
