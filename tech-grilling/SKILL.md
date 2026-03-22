---
name: tech-grilling
description: Check knowledge of the user on some technical topic, such as programming languages, development frameworks, tools etc. The grilling is done in different phases, starting with a set of conceptual questions and then moving on to more practical ones.
metadata:
  author: Andre Zunino <neyzunino@gmail.com>
  creation: 21 March 2026
---

# Tech-grilling skill

## Purpose

This skill is used when the user wishes to brush up, further develop or test their knowledge of some technical topic.

## Phases

Carry out the grilling in phases:

1. Conceptual: questions exploring important concepts on the topic
2. Objective: single-answer, multiple-choice questions
3. Practical: short problems that require the submission of code snippets

<critical>Questions should always be asked one at a time.</critical>

Phases 1 and 2 are mandatory, regardless of the topic. Include phase 3 only when appropriate (e.g. when the topic is a programming language, algorithms etc.)

## Set-up

Upon starting, ask the user about the following parameters, using a nice TUI, if you have it available:

- Topic
- Depth (difficulty level)
  - Beginner
  - Intermediary
  - Seasoned
- Mode
  - Quickie (1 conceptual; 3 objective; no practical)
  - Daily routine (2 conceptual; 3 objective; 2 practical)
  - Devoted pilgrim (4 conceptual; 5 objective; 5 practical)
  - Custom (any number of each phase, as specified by the user)

## Assessment

After each answer, provide an assessment, letting the user know how he did.

- For the conceptual phase, emphasize what they got right (if anything) or provide guidance to help them see what they got wrong.
- For the objective phase, point out which the correct choice was and briefly explain why.
- For the practical phase, evaluate the submitted code in terms of correctness and idiomatic use of the language.

## Scoring

At the end, present the user with a final score, from 0 to 10, considering that every question, regardless of type, has the same weight.

## Early exit

The user may choose to terminate the grilling before it reaches its end. When that happens, present their score up to that point and end the session.
