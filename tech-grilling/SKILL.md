---
name: tech-grilling
description: Check the user's knowledge of a technology topic, such as programming languages (e.g. "I'd like to check my knowledge of Python"), development frameworks or libraries (e.g. "Ask me about state management in a React application"), tools (e.g. "Grill me on the Vim editor") etc. The grilling is done in different phases, from conceptual to practical questions.
metadata:
  author: Andre Zunino <neyzunino@gmail.com>
  creation: 21 March 2026
  revised: 23 March 2026
---

# Tech-grilling skill

## Purpose

Help the user further develop or test their knowledge of a technology topic.

## Phases

Carry out the grilling in phases:

1. Conceptual: questions exploring important concepts on the topic
2. Objective: single-answer, 4-5 multiple-choice questions
3. Practical: short problems that require the submission of code snippets

<critical>Questions should always be asked one at a time.</critical>

Phases 1 and 2 are mandatory, regardless of the topic. Include phase 3 only when appropriate (e.g. when the topic is a programming language, algorithms etc.)

### Phase transitions

At the beginning of each phase, show a title announcing it:
  - "CONCEPTUAL QUESTIONS" for phase 1
  - "OBJECTIVE QUESTIONS" for phase 2
  - "PRACTICAL QUESTIONS" for phase 3

## Set-up

Upon starting, use your `question` tool to allow the user to make visual selections for setting up the session.

- Topic
- Difficulty level
  - Beginner
  - Intermediate
  - Seasoned
- Mode
  - Quickie (1 conceptual; 2 objective; no practical)
  - Daily routine (2 conceptual; 3 objective; 2 practical)
  - Devoted pilgrim (4 conceptual; 5 objective; 5 practical)
  - Custom (specified by the user; if this option is chosen, prompt the user for the desired counts)

## Difficulty level vs phases vs questions

Here are some pointers you should observe when coming up with the questions for each phase, based on what difficulty level the user chose:

| Difficulty   | Phase      | Questions                    |
|--------------|------------|------------------------------|
| Beginner     | Conceptual | Surface-level only, not involving more intricate relationships with other concepts. Questions that can typically be answered very directly. |
| Beginner     | Objective  | Basic, covering the most known features; no gotchas. |
| Beginner     | Practical  | Introductory-level problems that could be solved just by reading the introductory part of the documentation. |
| Intermediate | Conceptual | Diving a little deeper; exploring the interplay among the main concepts. |
| Intermediate | Objective  | Requiring more careful thought, with some eventual gotchas. |
| Intermediate | Practical  | Problems requiring the combination of more than a single feature or concept. |
| Seasoned     | Conceptual | Checking for solid understanding of a technology's concepts and eventually exploring tradeoffs and best practices. |
| Seasoned     | Objective  | More intricate, requiring familiarity and knowledge of nuances. |
| Seasoned     | Practical  | Solutions expected to be not only correct, but also idiomatic. |

## Assessment

After each answer, evaluate the user's answer:

- For the conceptual phase, emphasize what they got right (if anything) or provide guidance to help them see what they got wrong.
- For the objective phase, point out which the correct choice was and briefly explain why.
- For the practical phase, evaluate the submitted code in terms of correctness and idiomatic use of the language.

<critical>Don't calculate or show scores per question.</critical>

## Scoring

At the end, present the user with a final score, from 0 to 10, considering that every question, regardless of type, has the same weight. E.g. for a "daily routine", there are a total of 7 questions (2 conceptual, 3 objective and 2 practical). In that case, each question should be worth ~1.43 points.

### Difficulty level vs phases vs scoring

Upon calculating the final score, consider that the difficulty level has an impact on how conceptual and practical questions are graded. Objective questions are not mentioned, since they are either right or wrong.

| Difficulty   | Phase      | Scoring                      |
|--------------|------------|------------------------------|
| Beginner     | Conceptual | Be a little lenient, based on whether the answer shows a minimal, correct understanding. |
| Beginner     | Practical  | Consider efforts to solve the problems and award partial scores, even if the result is not correct. |
| Intermediate | Conceptual | Expect correct understanding of the main concepts, even if there are gaps. Grade according to how many gaps were present. |
| Intermediate | Practical  | Consider answers that don't produce the expected results to be wrong and score 0, except if it's apparent that the user's effort was really close, in which case you may grant some score. |
| Seasoned     | Conceptual | Don't be lenient; demand correct understanding of the concepts. Only if the user shows very clear understanding of a concept may an eventual omission still result in partial scoring. |
| Seasoned     | Practical  | Make sure code snippets provided not only produce the expected results, but follow best practices and idioms. |
