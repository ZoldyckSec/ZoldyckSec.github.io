---
title: Conversor - HackTheBox WriteUp
description: This write-up for the retired machine Expressway was completed while the machine was still active on the HackTheBox platform. Publication was held until after its official retirement to comply with platform guidelines.
author: ZoldyckSec
date: 2026-01-05 00:00:00 -0800
categories: [HackTheBox]
tags: [Conversor, Linux, Path-transversal, SSTI, sqlite]
render_with_liquid: false
---
## Description:

- `Date` January
- `Machine` Conversor (Linux)
- `Difficulty` Easy
- `Prepared by` Allan Espinoza(Aka.ZoldyckSec)

## Summary

This machine is a great lesson in why developers should not only protect against a format's most well-known vulnerabilities (such as XXE in XML), but also sanitize basic user input. We will move from a black box analysis to a source code audit (White-Box), forge an attack chain combining **Path Traversal** and **Server-Side Template Injection (SSTI)**, and culminate in an escalation of privileges exploiting **CVE-2024-48990**.
