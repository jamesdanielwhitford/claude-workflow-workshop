# AI-Assisted Technical Writing Workshop

This workshop shows you how to use AI tools effectively for technical writing. You'll learn to control AI by providing quality context, not by hoping it generates the right content.

## AI Tools Reformat Data

If you don't understand that AI tools reformat data, you don't know how to use them as a tool. You're just letting it do whatever it thinks it should do with no structured guidance.

Without proper prompting, AI gives generic answers. Everyone gets the same output because you're using the default prompt the company built into the tool.

To really utilize AI, you need to control the context.

**Think of AI as a reformatter:**
- You provide the information it will reformat
- You provide instructions and/or examples of how to format it
- AI is extremely good at this

**AI is very bad when:**
- Zero context provided
- No structure for formatting provided

This understanding changes how you set up your workflow and workstation.

## Creating a Good CLAUDE.md File and Project Structure

Your workflow and workstation need to be set up around this understanding of AI as a reformatting tool.

### The CLAUDE.md File

This file does a few things:

1. **Tells Claude where context is stored** and where work will be done
   - Where to retrieve context
   - Where to perform work

2. **Keeps track of progress**
   - What has been done
   - What still needs to be done
   - Problems we've run into

This way, you can end the Claude session after updating the CLAUDE.md file. When you restart, Claude has less context (lighter, faster) but understands everything through this file.

### Project Directory Structure

```
/project-root/
├── CLAUDE.md              # Workflow guide and progress tracker
├── brief/                 # Project requirements
├── research/              # Verified documentation
├── code/                  # Working code examples
├── repositories/          # Cloned documentation repos
├── draft.md              # Writing workspace
└── writing-rules/         # Style guidelines
```

Look at the `CLAUDE.md` file in this workshop folder to see how it works.

---

## The Workflow

Now that you have the CLAUDE.md file and project structure set up, here's the workflow.

### Step 1: Understand the Brief and Find Correct Information

Read the brief to understand what you're building and why someone would want to build it.

Find out: **what documentation can I have directly in my workstation?**

Clone repositories with up-to-date, correct documentation into your project:

```bash
git clone https://github.com/product/docs.git repositories/product-docs
```

This gives you direct access to the most accurate context.

### Step 2: Source Documents from Cloned Repositories

Ask Claude to look at the brief and use the cloned repositories to find relevant documents.

**What Claude does:**
- Searches through `repositories/` for relevant documentation
- Copies exact files into `research/` folder
- You validate and either keep or delete from `research/`

This ensures the context Claude accesses is the best, most up-to-date, quality context that it will just be reformatting for you.

### Step 3: Research When You Don't Have Direct Documentation

If you can't have documentation directly in your work structure (maybe they don't have public docs), do research.

**You can research yourself, or ask Claude to research for you.**

When Claude researches:
- Claude generates a file in `research/` with all links it thinks are relevant
- File includes links and short summaries of information
- You go through and check the links actually have the content Claude says
- You approve or deny the links
- For approved links: copy the raw text content from the website into `research/`

### Step 4: The Coding Stage

AI is a much better coder than writer, but it still helps to provide context.

**Some languages need less context:**
- Simple frameworks like Express and Vanilla JS work very well
- AI has lots of quality training data for these

**For newer frameworks:**
- Get SDK documentation in your working directory
- Or use a tool like Context7 (an MCP tool that retrieves relevant context)

Claude uses context from `research/` and possibly Context7 to generate code in the `code/` folder.

**When code is complete and works:**
- Delete old attempts
- Make sure `code/` folder only has correct, working code
- Claude knows this is the correct final implementation
- Move on to writing

### Step 5: The Writing Stage

Write simply. Follow the ["How to Write for Developers"](https://refactoringenglish.com/chapters/write-blog-posts-developers-read/) philosophy.

**Create a heading skeleton:**
- Write headings you know will be easy for readers to understand
- Headings that will fulfill the brief
- Add code snippets in order under appropriate headings
- Show readers step-by-step what they need to create
- Read about [writing for skimmers](https://refactoringenglish.com/chapters/write-blog-posts-developers-read/#accommodate-skimmers) - your headings should tell a story on their own

Now you have a skeleton: headings + code snippets from the code stage.

**Ask Claude to add research references:**

Tell Claude: "Look at this skeleton I've created. Use the research I've approved in `research/` folder. Link to those files and provide summaries of information I can use to write each section."

Claude updates your draft with helpful information you'll use to write the article.

**When you struggle with writing:**

If you're having trouble with Markdown format or explaining something clearly:

Use the `writing-rules/` folder:
- Good example article
- Ritza writing rules
- AI kill list

Ask Claude to reformat your work based on these writing rules - making it more succinct or readable.

## Summary

**The key insight:** AI reformats data. Give it quality input (context + formatting rules) and it gives you quality output.

**Your workflow:**
1. Set up CLAUDE.md and project structure
2. Understand the brief
3. Get correct documentation into your workspace
4. Let Claude source relevant docs (you verify)
5. Research what you can't clone (you verify)
6. Build working code with Claude's help
7. Create heading skeleton + code snippets
8. Let Claude add research references
9. Write content using your research
10. Let Claude reformat rough sections per writing rules

**Remember:** You curate. AI reformats.
