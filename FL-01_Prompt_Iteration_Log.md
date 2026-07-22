# Prompt Iteration Log -- FL-01

## Task

**Real Task (FL-01):**

Generate engaging portfolio website copy for an AI Engineer's personal
portfolio.

The portfolio should:

-   Target recruiters
-   Highlight AI/ML projects
-   Sound professional
-   Include a strong hero section
-   Include CTAs
-   Be concise

------------------------------------------------------------------------

## Version 1 --- Naive Prompt

**Technique:** None (Baseline)

### Prompt

``` text
Write content for my AI portfolio website.
```

### Output

-   Generic introduction
-   Mentions AI and programming
-   No recruiter focus
-   No sections
-   Very short
-   Generic ending

### Observation

The model understood the topic but produced extremely generic content
with no structure or business value.

------------------------------------------------------------------------

## Version 2 --- Role Assignment

**Technique:** Role Prompting

### Prompt

``` text
You are an award-winning UX copywriter who creates portfolio websites that help AI engineers get hired.

Write portfolio website content.
```

### Output

-   Better headline
-   More confident tone
-   Recruiter-oriented language
-   Better CTA

### Observation

Assigning a role immediately improved confidence and professionalism.
Instead of sounding like a student project, it sounded like a real
product.

------------------------------------------------------------------------

## Version 3 --- Context & Motivation

**Technique:** Provide Context

### Prompt

``` text
You are an award-winning UX copywriter.

I'm a final-year BS Artificial Intelligence student applying for AI Engineer internships.

My audience is recruiters and startup founders.

The goal is to convince them to contact me within 30 seconds.

Create compelling homepage copy.
```

### Output

-   Mentions recruiter goals
-   Better storytelling
-   Strong value proposition
-   More persuasive CTA

### Observation

Adding context helped the model understand why the content was being
written. The output became significantly more relevant.

------------------------------------------------------------------------

## Version 4 --- Few-shot Examples

**Technique:** Few-shot Prompting

### Prompt

``` text
You are an award-winning UX copywriter.

Here are examples.

Example Hero:

Building AI Systems That Solve Real Problems

Example CTA:

Let's Build Something Intelligent.

Now write similar-quality copy for my AI portfolio website.
```

### Output

-   Similar premium style
-   Better headlines
-   Better transitions
-   Consistent tone

### Observation

Examples acted as style guidance. The output matched the desired writing
quality much more closely.

------------------------------------------------------------------------

## Version 5 --- Output Structure

**Technique:** Structured Output

### Prompt

``` text
You are an award-winning UX copywriter.

Create homepage copy.

Return ONLY this format.

Hero Title

Hero Subtitle

About

Skills

Projects

Experience

CTA

Footer
```

### Output

Everything appeared in clearly labeled sections and was easy to copy
directly into a website.

### Observation

The structure eliminated randomness and made the response immediately
usable.

------------------------------------------------------------------------

## Version 6 --- Step Decomposition

**Technique:** Step Decomposition

### Prompt

``` text
You are an award-winning UX copywriter.

Think through the task in stages.

Step 1:
Identify the audience.

Step 2:
Determine what recruiters care about.

Step 3:
Create a positioning statement.

Step 4:
Generate homepage copy.

Return only the final copy.
```

### Output

-   Highly targeted
-   Strong recruiter messaging
-   Better flow
-   More persuasive
-   Better CTA hierarchy

### Observation

Breaking the task into logical stages improved coherence and reduced
generic wording.

------------------------------------------------------------------------

# Cross-Model Comparison

## Claude

### Strengths

-   Better storytelling
-   More natural language
-   More creative headlines
-   Better transitions
-   More premium tone

### Weaknesses

-   Occasionally verbose
-   Sometimes assumes details not provided

------------------------------------------------------------------------

## ChatGPT

### Strengths

-   Better organization
-   Cleaner formatting
-   More actionable structure
-   Easier to edit
-   Better adherence to requested output format

### Weaknesses

-   Slightly less creative
-   Can sound more formal or templated without examples

------------------------------------------------------------------------

## Overall Comparison

Claude produced more engaging and persuasive marketing copy, especially
in storytelling and headline quality. ChatGPT consistently followed
formatting instructions more precisely and produced output that was
easier to integrate directly into a portfolio website. For this task,
Claude excelled at creativity, while ChatGPT excelled at structure and
instruction-following.

------------------------------------------------------------------------

# Final Reusable Prompt Template

``` text
You are an expert [ROLE].

Goal:
[Describe the objective.]

Audience:
[Who will read it?]

Context:
[Provide relevant background.]

Requirements:
- Requirement 1
- Requirement 2
- Requirement 3

Examples (optional):
Example 1:
...

Example 2:
...

Instructions:

1. Identify the audience.
2. Determine their priorities.
3. Plan the content.
4. Produce the final result.

Return the output using exactly this structure:

[Desired headings or format]

Do not include explanations.
Return only the final answer.
```
