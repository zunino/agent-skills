---
name: lang-grilling
description: Check user's knowledge of a natural language, such as English.
metadata:
  author: Andre Zunino <neyzunino@gmail.com>
  creation: 22 March 2026
---

# Lang-grilling skill

## Purpose

Help the user check his proficiency in a language of choice.

## Phases

Carry out the grilling in phases:

1. Objective: single-answer, 4-5 multiple-choice questions (use your `question` to show the question and get the user's selection instead of adding it to the regular chat context; <critical>Do not add your own comments for each choice!</critical>)
2. Fill-in the blanks: special kind of objective question where the user is shown a sentence with 1 or more gaps and then has to choose the correct sequence of particles to fill-in the gaps. The gaps could happen for prepositions, modal verbs, phrasal verbs or idioms.
3. Practical: questions that require the submission of written responses; the user will always have to type, not choose from a list.

<critical>Questions should always be asked one at a time.</critical>

## Set-up

Upon starting, use your `question` tool to ask the user about the following:

- Level
  - Beginner
  - Intermediary
  - Advanced
  - Proficient
- Mode
  - Quickie (3 objective; 1 practical)
  - Daily routine (4 objective; 2 practical)
  - Devoted pilgrim (5 objective; 5 practical)
  - Custom (any number of each phase, as specified by the user)

## Assessment

After each answer, provide an assessment, letting the user know how he did.

- For the objective phase, point out which the correct choice was and briefly explain why.
- For the practical phase, evaluate the submitted text in terms of grammatical correctness and idiomatic use of the language.

## Scoring

At the end, present the user with a final score, from 0 to 10, considering that every question, regardless of type, has the same weight.

## Early exit

The user may choose to terminate the grilling before it reaches its end. When that happens, present their score up to that point and end the session.
