---
title: Some tips to beautify markdown
author: Lynn
date: 2022-11-25 20:00:00 +0800
last_modified_at: 2022-11-25 21:30:00 +0800
categories: [Markdown, Edit]
tags: [markdown]
syntax: colorful
pin: true
---

This post is to show how sites renders markdown file syntax and beatify it to beautify it to be more in line with viewing blog.

## Dividing line

You can write `---` to your md to show a dividing line and the line is as followed:

---

## Lists

### Ordered list
You can use code like this to specify an ordered list
```text
1. Firstly
2. Secondly
3. Thirdly
```
1. Firstly
2. Secondly
3. Thirdly

### Unordered list
You can use code like this to specify an unordered list
```text
- Chapter
    + Section
        * Paragraph
```
- Chapter
    + Section
        * Paragraph


### ToDo list
You can use code like this to specify a to-do list
```text
- [ ] Job
    + [x] Step 1
    + [x] Step 2
    + [ ] Step 3
```
- [ ] Job
    + [x] Step 1
    + [x] Step 2
    + [ ] Step 3

### Description list
You can use code like this to specify a to-do list
```text
Dogs
: I like samoyed~

Cats
: Prefer dogs, don't you?
```
Dogs
: I like samoyed~

Cats
: Prefer dogs, don't you?

## Block Quote
### Inline block quote
```text
The is an example of `Inline block quote`.
```
This is an example of `Inline block quote`.

### Simplest block quote
```text
> This line shows the _block quote_.
```
> This line shows the _block quote_.

### Prompts block quote
```text
> An example showing the `type` type prompt.
({: .prompt-tip }|{: .prompt-info }|{: .prompt-warning }|{: .prompt-danger })
```

> An example showing the `tip` type prompt.
{: .prompt-tip }

> An example showing the `info` type prompt.
{: .prompt-info }

> An example showing the `warning` type prompt.
{: .prompt-warning }

> An example showing the `danger` type prompt.
{: .prompt-danger }

### Tables block quote
The code may be a little complicated:
```text
| Primary Key                  | Color                     | Size  |
|:-----------------------------|:--------------------------|------:|
| Samoyed                      | White                     | Large |
| Golden retriever             | Golden Yellow             | Large |
| Border collie                | White and black combined  | Large |
```

| Primary Key                  | Color                     | Size  |
|:-----------------------------|:--------------------------|------:|
| Samoyed                      | White                     | Large |
| Golden retriever             | Golden Yellow             | Large |
| Border collie                | White and black combined  | Large |

### Common code block
Use double `"```"` to surround your codes
```
This is a common code snippet, without syntax highlight and line number.
```

### Specific Languages
Specific the language name after the first `"```"`
#### Java:
```java
  class Main {
    public static void main(String[] args) {
      System.out.println("hello world");
    }
  }
```

#### Shell:
```bash
if [ $1 -ne 0 ]; then
    echo "The command was not successful.";
fi;
```

#### SQL:
```sql
SELECT *
FROM parquet.`/PATH/TO/PARQUET_FILE.parquet`
WHERE PRIMARY_KEY="samoyed";
```

## Footnote
Use `[^fn-nt-?]` to locate a footnote, and the `?` means the serial number.
Click the hook will locate the first footnote[^footnote], and here is second footnote[^fn-nth-2].

## Links
Use `<>` to include a link
<http://127.0.0.1:80>

## Filepath
### Simple filename format
```text
Here is the `/simple/filename/file`{: .filepath}.
```
Here is the `/simple/filename/file`{: .filepath}.

### Specific filename format
```sass
@import
  "colors/light-typography",
  "colors/dark-typography"
```
{: file='_sass/main.scss'}

## Reverse Footnote

[^footnote]: The first footnote source
[^fn-nth-2]: The second footnote source
