# Editors

## Why Your Editor Matters

Writing software means solving problems. Your editor — the program you use to write code — can make solving those problems much easier.

A good editor does more than just let you type. It can:

- **Automate repetitive tasks**, so you don't waste time or mental energy on them
- **Organize your screen**, keeping your code, terminal, and database tools all within easy reach
- **Speed up your work** with keyboard shortcuts
- **Reduce eye strain**, with adjustable colors, themes, and fonts

This chapter compares the main types of editors used in software development, and what each one is good — and not so good — for.

## It's All Just Text

Here's something worth understanding before you pick an editor: when you write code, you're really just creating a text file.

The only difference between a code file and a plain text file is the **extension** — the letters after the dot in the filename. A Python file ends in `.py`, a PHP file ends in `.php`, and so on. That extension tells your computer (and your editor) what language the file is written in. Once you save the file, a program called an **interpreter** or **compiler** reads your code and runs it.

Because a code file is just text, you can technically write it in *any* text editor, as long as you save it with the right extension. That's exactly why there are so many editors to choose from — and why picking one can feel more complicated than it should be.

In this chapter, we'll use **Python** as our example language. It's popular, powerful, and one of the easier languages to learn — and like most languages, you can write it in almost any editor.

## Three Types of Editors

Editors generally fall into three groups. Each has its own strengths, and most developers only discover which one suits them through trial and error — sometimes it takes weeks or months of daily use before a small annoyance (like a slow file-opening speed, or a missing feature you need) becomes a dealbreaker.

Beyond features, don't ignore comfort. Background color, font style, font size — these affect how tired your eyes get after hours of staring at a screen. A comfortable setup is worth customizing.

The three groups are:

1. Shell-based editors
2. Text editors
3. IDEs (Integrated Development Environments)

Let's go through each one.

### 1. Shell-Based Editors

**Examples:** Vi, Vim, Nano

These editors run directly inside a command-line shell — a text-only interface — on Windows, Linux, or macOS. There's no graphical interface, no mouse-friendly menus, just text and keyboard commands.

**Why they matter:**
Shell-based editors are the tool of last resort — and sometimes the *only* option. If you're logged into a remote server with no graphical interface and something breaks, a shell-based editor may be the only way to fix it or apply a quick patch while a permanent fix is worked out.

**Things to know:**
- They have a steep learning curve.
- If your career leads toward Linux systems administration or DevOps, you *will* run into Vi or Vim eventually.
- Vim is available for Windows too, though you're unlikely to need it there.
- Vim can be extended with plugin bundles (such as SPF13) to add more modern features.

**Bottom line:** Learn the basics — they're genuinely useful in an emergency — but don't rely on a shell-based editor as your everyday, primary editor.

### 2. Text Editors

**Examples:** Notepad, Gedit, Sublime Text, Visual Studio Code (VS Code)

Text editors give you a graphical window to write code in, ranging from extremely basic to surprisingly powerful.

**The bare-minimum end:**
Windows' **Notepad** offers no coding features at all — no syntax highlighting, no shortcuts, nothing beyond plain typing. You *can* technically write code in it (just save the file with the right extension), but there's no good reason to. It's mentioned here mainly as a warning: not every editor is a good choice.

Linux's **Gedit** is a small step up — it has a few basic helpful features. It's fine for a short, disposable script, but not something to build a habit around.

**The powerful end:**
Some text editors are genuinely feature-rich, extendable with plugins, and customizable. Two well-known examples:

- **Atom** — created by GitHub, known for a modern, approachable feel. *(Note: Atom was officially discontinued in December 2022. It's mentioned here for historical context, but it's no longer a recommended choice for new projects.)*
- **Visual Studio Code (VS Code)** — made by Microsoft, known for being lightweight, fast-starting, and heavily extendable via plugins. VS Code has since become one of the most widely used code editors among developers worldwide, and it's the one we'll focus on in this chapter.

Other modern text editors worth knowing about include **Sublime Text** and **Neovim** (a more modern, plugin-friendly evolution of Vim).

**Trade-offs to expect:**
- You'll likely spend time hunting for and configuring plugins to get the features you want.
- Even with plugins, large projects can open and search more slowly than they would in a dedicated IDE.

### 3. IDEs (Integrated Development Environments)

**Examples:** PyCharm, Eclipse, Wing

An IDE bundles everything into one application:

- A code editor
- A debugger
- An interpreter or compiler
- A built-in terminal/shell
- A database browser
- Often, built-in version control integration (like Git)

**What makes IDEs stand out:**
- They automatically check your code for errors as you type, based on the specific version of the language you're using.
- They index your entire project the moment you open it, so searching and jumping between files is instant.
- You can jump from where a function is *called* to where it's *defined*, and back again, in one click.
- They come packed with keyboard shortcuts for common actions.
- Some keep a local history of your changes, so you can recover code you accidentally deleted — even before you've committed it to version control.
- Most support plugins too, just like text editors.

**Cost:** Some IDEs are paid, but they're often worth the investment. Others — including **PyCharm** — offer a free "Community" edition with a smaller feature set alongside a paid "Professional" edition.

## What You Actually Gain From a Good Setup

A well-configured editor or IDE pays off in a few concrete ways:

- **Code navigation** — jumping straight to the code you need, instead of scrolling and searching manually.
- **Debugging** — stepping through your program one line at a time while it runs, and inspecting or changing values as you go. This is one of the most powerful tools available to a developer.
- **Integrated tools** — running database queries or shell commands in a pane right next to your code, instead of switching between separate windows.

### Why Code Completion Matters More Than It Sounds

Code completion — where your editor suggests the rest of what you're typing — sounds like a minor convenience. It isn't.

Programming already asks a lot of your working memory: you're holding a problem, a plan, and the syntax of a language in your head all at once. Every time you have to stop and recall an exact keyword or function name, you break that concentration.

Code completion removes that burden. It doesn't matter if you *could* remember the exact syntax — outsourcing that small task to your editor keeps your focus on the actual problem you're solving, which is what programming is really about.

## Quick Comparison

| | Shell-based (Vi, Vim, Nano) | Text editor (VS Code, Sublime) | IDE (PyCharm, Eclipse) |
|---|---|---|---|
| **Interface** | Text-only | Graphical | Graphical |
| **Learning curve** | Steep | Gentle to moderate | Moderate |
| **Setup needed** | Minimal, but powerful once learned | Plugins needed for full features | Mostly built-in |
| **Best for** | Remote servers, quick fixes | Everyday coding, lightweight projects | Large projects, serious development |
| **Cost** | Free | Free | Often free tier, paid for full features |

## Takeaway

Don't treat your choice of editor as an afterthought. Whether you land on a plugin-loaded text editor or a full IDE, the goal is the same: let the tool handle the small, memorizable, repetitive things — so you can spend your energy where it actually matters, on solving the problem in front of you.