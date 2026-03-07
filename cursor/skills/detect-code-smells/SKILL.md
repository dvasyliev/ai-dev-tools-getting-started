---
name: detect-code-smells
description: Find common code smells and suggest small fixes without changing behavior.
---

# Skill: Detect Code Smell

## Overview

This skill checks code for common problems and suggests simple improvements.

## Instructions

- Review the files provided (or only changed files if a diff is provided).
- Find code smells like:
  - long functions/components
  - duplicated code
  - unclear names
  - too much logic in one file
  - deep nesting and complex conditions
  - unused code
- Do not change behavior.
- Suggest small, safe fixes.

## Expected output

- Short summary
- List of smells (file + where)
- 3–7 fix suggestions
