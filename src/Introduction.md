# Introduction

First is a statically typed language for describing the shape of a system. It's trending towards a full systems-level language. This page covers status, audience, and license. The language itself is in the sections that follow.

## What First Is

First describes a system’s structure in a readable, code-like format instead of prose. It can replace Markdown or HTML for specs. Early experiments suggest that agents translating First specs into code can use significantly less capable models while producing better results.

We call First a _criticality-partitioned_ language. Today, the highest-criticality structure is written in First, while lower-criticality implementation is generated  into TypeScript, Swift, Rust, or Kotlin. Soon, both sides of that boundary will live in First. Humans will author the structure that must stay explicit and agents will expand the implementation where acceptable. The completed First program will then be mechanically translated to TypeScript, Swift, Rust, or Kotlin.

## Status

| Area | State |
| --- | --- |
| Parser | Substantial, moving fast |
| Syntax highlighting | Imminent |
| Editor | Not yet |
| Compiler | Not yet |
| Backends | Planned |

## What You Can Do Today

Write First instead of a Markdown spec. Hand it to an agent and ask for an implementation. The structure stays visible, so agents produce better-shaped output. It already beats Markdown. The Replier app is built this way.

## Who It's For

- The shape of your system has to survive.
- You want a native target but think in TypeScript.
- Controlling structure beats describing outcomes and accepting what returns.

## When First Isn't The Right Fit

- Your workflow works and doesn't hurt.
- You're optimizing for velocity and turnover.
- Changing process is overhead you don't want.

If none of those apply, there's no reason to switch.

## Supported Targets

Swift and TypeScript initially; TypeScript doubles as the analysis core. Rust and Kotlin will follow. Nothing emits today.

## How To Read These Docs

Start with Imports, Spaces and Classes, and Functions. Read Language Examples to see First cold.

Terms used throughout: `notice` (a report, not a failure), `unknowable` (compiler recovery state), `bomb` (an error that halts), glyphs (editor rendering of serialized source).

## Near-Term Work

Parser completion, syntax highlighting, symbol resolution, Swift and TypeScript backends, the editor. Intermediate step: generate more First instead of jumping to a target.

## License And Open Source

Documentation: MIT, public on GitHub. Compiler is internal while we are still in research, but we expect MIT once we release.
