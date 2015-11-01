---
layout: post
title: "Machine Learning Foundations: The Learning Problem"
comments: True
---

### The Essence of Machine Learning

Improving some performance measure with experience computed from data.

**Data --> ML --> Improved performance measure**

### When to Use Machine Learning?

1. Exists some 'underlying pattern' to be learned - so the 'performance measure' can be improved.
2. But NO programmable (easy) definition - so ML is needed.
3. Somehow there's **data** about the pattern - so ML has input to learn from.

### Some Interesting Examples

- Food:
  - data: tweets (words + location)
  - skill: tell food poisoning likeliness of restaurants

- Clothing:
  - data: sales figures + client surveys
  - skill: give fashion recommendation to clients

- Housing:
  - data: characteristics of buildings and energy loads
  - skill: predict energy loads

- Transportation:
  - data: traffic sign images and meanings
  - skill: recognize traffic signs

- Education:
  - data: students' records of quizzes
  - skill: predict whether a student can answer another quiz question correctly

- Entertainment:
  - data: how many users have rated some movies
  - skill: predict how a user would rate an unrated movie

### Components of ML

- Input feature vector **x**
- Output label y
- Target function f: **x** -> y
- Data: training set

$$
D=\\{ (\mathbf{x}\_1, y\_1),(\mathbf{x}\_2, y\_2), ... (\mathbf{x}\_N, y\_N) \\}
$$

- Hypothesis <=> skill
  - g: **x** -> y
  - 'learned' formula to be used

f: ideal formula

g: hypothesis, approximates f

- Purpose of ML: unknown f, find g that approximates f
- Hypothesis set **H**, includes many g's

$$ g\in \mathbf{H}= \\{ h_{ k } \\} $$

<br>
<br>
### Machine Learning's Relationship with Data Mining, Artificial Intelligence and Statistics

#### ML vs. DM

- Definition
  - ML: find hypothesis g ~ f. 
  - DM: use (huge) data to find 'interesting' properties. 

- For KDD Cup, ML = DM
- If 'interesting property' is g ~ f, ML and DM can help each other.
- Traditional DM also focuses on efficient computing in large databases.

#### ML vs. AI

- g ~ f is intelligent: ML can realize AI, among other routes.
- e.g. chess playing
  - traditional AI: game tree
  - ML for AI: learn from board data

*"ML is a possible route to realize AI"*

#### ML vs. Statistics

- Statistics: use data to make inference about an unknown process
- g is an inference outcome, f is unknown
  - statistics can be used to achieve ML
- Traditional statistics also focuses on provable results with math assumptions, and care less about
computation.

*"Statistics is a useful tool for ML"*





<br>
<br>
<br>
#### Side note for redcarpet with MathJax:

- doesn't respect $ or $$ (will parse characters within these, such as _), but has workarounds:
- has a useful option, no_intra_emphasis, so underscores between ascii characters are not converted to `<em>` tags, so `$x_1$` is fine
- **need to escape underscores** after non-ascii characters, for example `$\mathbf{x}\_1$`
- need to double escape curly bracket: `\\{`
- need three backslash for new line `\\\`
- `=` cannot be on its own line












