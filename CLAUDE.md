# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository is a study workspace for AWS Certified Solutions Architect Associate (SAA-C03) exam preparation.

## Structure

```
aws certif/
├── 01-secure-architectures/     # Domain 1 (30%) — IAM, KMS, network security
├── 02-resilient-architectures/  # Domain 2 (26%) — HA, Auto Scaling, SQS, Route 53
├── 03-high-performing/          # Domain 3 (24%) — compute, caching, networking
├── 04-cost-optimized/           # Domain 4 (20%) — pricing models, storage tiers
├── cheatsheets/                 # Quick reference sheets per service/topic
├── practice-questions/          # Exam patterns, traps, and mock questions
├── progress.md                  # Checklist tracker per topic
└── README.md                    # Exam overview and resources
```

## Notes Conventions

- Language: mix of English (AWS terminology) and Bahasa Indonesia (explanations)
- Format: Notion-style structured Markdown — use headers, tables, code blocks
- Exam tips are highlighted with `> **Exam tip:**` blockquotes
- Common traps are documented in `practice-questions/patterns-and-traps.md`

## When Adding Content

- Add domain notes to the relevant `0X-*/notes.md` file
- Add service one-liners to `cheatsheets/services-reference.md`
- Add exam patterns and tricky questions to `practice-questions/patterns-and-traps.md`
- Update checkboxes in `progress.md` when a topic is complete
