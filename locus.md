# Locus — Multi-Vendor DVR/NVR Forensic Analysis Suite

**Project:** Locus  
**PS ID:** 26150 (SIH 2026) | **Theme:** Cybersecurity & Digital Forensics  
**Target Category:** Multi-vendor DVR/NVR forensic analysis and recovery platform  

---

## Core Differentiator & Primary Objective

> **Central Forensic Safety Principle:**  
> **Locus prioritizes forensic correctness over successful-looking output.** A system crash or explicit `UNKNOWN` state is safer than a false positive or a confidently wrong forensic result. The system strictly prefers `UNKNOWN`, `UNSUPPORTED`, `AMBIGUOUS`, `PARTIAL`, `CORRUPTED`, or `UNRECOVERABLE` over best-guess reconstruction, false confidence, or unvalidated recovery.

**Locus** is a specialized desktop forensic platform engineered for forensically defensible multi-vendor DVR/NVR acquisition analysis, proprietary layout reconstruction, stream validation, timeline normalization, full-chain provenance, and structured evidence reporting.

It addresses surveillance disk evidence (such as raw disk images `.dd`, `.raw`, `.img`) from major OEMs including **Dahua, Hikvision, CP Plus, Honeywell, TP-Link, Godrej, Uniview, and Matrix**. Rather than relying on generic carving tools or proprietary closed-source player utilities, Locus uses a modular architecture combining vendor-specific forensic adapters, layout validation engines, deterministic metadata parsing, and cryptographic verification (`SHA-256` & `MD5`).

Computer vision AI (YOLOv8 via ONNX Runtime) is strictly scoped as a **secondary analytical/triage layer** operating after read-only evidence extraction and validation to accelerate investigator review. AI is never treated as primary ground truth or proof of identity.

---

## Navigation & Documentation Index

- [[MVP/MVP|MVP Specification]] — Core MVP Architecture, Forensic Principles & Feature Specs (Features 01–08)
- [[project-overview]] — Project Overview & Executive Summary
- [[official-problem-statement]] — Complete, unedited official text of PS 26150
- [[problem-statement]] — Technical Problem Analysis, Risks of Incorrect Recovery & Objectives
- [[architecture-and-tech-stack]] — System Architecture, Technology Rationale & Security Model
- [[requirements-and-modules]] — Modular Requirements, Data Model & Vendor Support Matrix
- [[team-and-recruitment]] — Technical Team Composition & Engineering Roles
- [[deliverables-and-roadmap]] — MVP Scope, Phase 2/3 Roadmap & Hackathon Demo Workflow
- [[reverse-engineering/00-index|Reverse Engineering Index]] — Laboratory Findings on DVR Filesystems & Video Layouts