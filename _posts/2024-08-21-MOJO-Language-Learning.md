---
title: MOJO Language Learning
date: 2024-08-21 17:10:00 -0500
categories: [development, mojo]
tags: [mojo, python]
---

I’ve recently started experimenting with a newer programming language called Mojo. As a Python developer, I was curious about what Mojo could offer and how it might solve some of the issues we commonly face in Python. This post documents my journey as I learn Mojo, and my first project to get started a simple Debt Payoff Calculator.

## Initial Thoughts and Challenges
Mojo is an interesting language that shows promise in addressing some of the limitations of Python, especially its slowness. However, since it is still in its early development, documentation is limited and articles on SourceForge are scarce.

### Issues I Encountered:
- Difficulty implementing user input similar to Python’s `input()` function.
- Syntax and function declaration errors that required adjustments from what I’m used to in Python.
- Understanding the current limitations of Mojo, especially its reliance on Python for certain tasks.

### Ongoing Challenges:
I’m still working on a reliable solution for capturing user input in Mojo. Once I figure it out, I’ll be sure to document it in detail.

## First Project: Debt Payoff Calculator

As part of my learning process, I decided to create a simple Debt Payoff Calculator. 

### Example of the Debt Payoff Calculator in Action:

```bash
$ mojo main.mojo 
Debt Payoff Timeline Calculator
It will take 57 months to pay off the debt.
```

```bash
$ mojo main.mojo 
Debt Payoff Timeline Calculator
Monthly payment is too low to cover the interest. The debt will never be paid off.
Debt cannot be paid off with the provided payment amount.
```

### Project Details and Code:
I’ve been logging issues and challenges during development, and I’ve documented each step as I troubleshoot problems. Here’s the link to the repository where you can follow my progress and see the full code:

**GitHub Repository:** [MojoLangLearning](https://github.com/CalebFIN/MojoLangLearning)

### Log of Issues:

```zsh
(max-venv) host:~/DebtCalculator$ mojo main.mojo
Debt Payoff Timeline Calculator
It will take 57 months to pay off the debt.

(max-venv) host:~/DebtCalculator$ mojo main.mojo
Debt Payoff Timeline Calculator
Monthly payment is too low to cover the interest. The debt will never be paid off.
Debt cannot be paid off with the provided payment amount.

(max-venv) host:~/DebtCalculator$ mojo main.mojo
~/DebtCalculator/main.mojo:31:5: error: use of unknown declaration 'let', 'fn' declarations require explicit variable declarations
    let input = Python.import_module("builtins").input
    ^~~
```
Learning Mojo is interesting. The language is still very new but shows potential. Integrating Python modules comes with challenges.
