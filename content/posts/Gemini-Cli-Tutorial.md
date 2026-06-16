+++
date = '2026-06-11T14:57:05-04:00'
draft = false
title = 'Gemini CLI Tutorial'
+++

This tutorial serves as my personal reference for using the Gemini CLI. I have documented this blog while following this [YouTube playlist](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3D1AF5pFGwRTM%26list%3DPL4cUxeGkcC9h-AKdBSCRpqjD3y6T7Xgrb).

## Setup

### Installation

To install the Gemini CLI, run the command below in a terminal window. Ensure Node.js is installed on your system.

Bash

```
npm install -g @google/gemini-cli
```

After installing the CLI tool, navigate to your project folder. If it is your first time launching the Gemini CLI, it will prompt you to authenticate. To use a different account later, you can run the following command in the CLI:



```
/auth
```

We have three options to log in:

* Using a Google account

* Using a Gemini API Key

* Using Vertex AI

Selecting a Google account will redirect you to an authentication page. Alternatively, you can use an API key, available from Google AI Studio. This document does not cover Vertex AI.

Using a Google account is straightforward, and for now, we will rely on it. We will discuss the API key shortly. Once authenticated, we can start using the Gemini CLI.

If you have already created the project and have some files in the folder, let us start with the following query to our new tool:



```
Tell me about this project
```

Given some code and a `Readme.md` file, the system reads both files and produces a project summary.

### First Use

I prefer using Visual Studio for C# development. However, since Visual Studio lacks extensions for Gemini, I will switch to Visual Studio Code (VS Code) for my development work.

Open VS Code, and within its terminal, run the command `gemini`. If it is your first time using Gemini, you will be prompted to install a companion extension. Proceed with the installation.

## The Gemini.md File

First, the goal is to create a Gemini markdown file (`Gemini.md`) that provides instructions to Gemini about project conventions, packages, libraries, project architecture, and any coding preferences. For instance, it is preferred to avoid bloating the template files.

When there is an existing project with added files and code, the Gemini CLI can be used to generate the Gemini file. The CLI searches through the codebase to identify patterns and gather information such as the framework in use, CSS libraries, application structure, overall purpose, and data handling. It then summarizes this information and writes it to the Gemini file. We use the following command to generate the `Gemini.md` file:



```
/init
```

The program reads multiple files and examines the application to generate content for a new Gemini file. However, it has not written the content yet; it only displays a diff showing the proposed changes. In the chat window, it prompts for permission to write this content to the file. By default, whenever the Gemini CLI intends to make file changes or run commands, it will seek user approval beforehand.

When prompted, users typically have three options:

1. **"Yes, allow once"**: Permits a single write action but prompts again for future changes.

2. **"Yes, always allow"**: Grants permission for the session without further prompts.

3. **Refuse/Modify**: Users can refuse, suggest modifications to the content, or cancel the operation altogether.

The user prefers to review each change before it is applied and therefore selects **"Yes, allow once."**

The above command will create a `Gemini.md` file based on the project and the `Readme.md` file. We can, and most probably have to, add additional instructions. These are the instructions I added to my `Gemini.md` file:



```
### Additional Coding Preferences

- Check whether any NuGet packages can implement the functionality before writing new code.
- If a service or utility is used in multiple projects, the method should be implemented in the AppCommon project.
```

There is one more step in that process: run the command `/memory refresh`. This command instructs the Gemini CLI to rescan for any Gemini files since its last scan. After running it, a message should appear indicating that the memory has been refreshed.

> **Note:** Additional Gemini files can be created within subdirectories to offer instructions tailored to specific files in that section of the application. For instance, if there are instructions relevant only when creating or editing page components, a new Gemini file could be placed in the `pages` folder. When working on files within that folder, the model will follow those specific instructions.

Additionally, these files should not be created and then neglected. As the application evolves, changes are made, and various services, preferences, or styles are integrated, the Gemini file should be continuously updated to mirror these developments. Otherwise, the AI model will rely on outdated information about the project.

This process is always the first step when working on an existing project. For a new project, the file should still be created, but initially, there will be no data for Gemini to generate content from. In such cases, basic preferences are typically set manually at the start, and then the file is updated progressively as the application develops.

A global `Gemini.md` file is also possible, but it will not be covered in this document.

## Code Changes

Although Gemini can handle the entire project, we will begin by working on one file at a time. To observe the changes, let us open the specific file where updates will be made.

In the Gemini CLI UI, users can enter prompts describing the changes they want to make. Typically, they prepare a markdown document listing all desired modifications, then select each feature and paste the corresponding prompt into the chat window. Editing directly within the prompt window can be a bit tricky, but having the document ready allows users to review their planned changes and ask Gemini to carry out each step individually.

The Gemini CLI can also commit our changes to Git.

## Commands and Settings

You can get a list of all commands by using the forward slash (`/`) character.

* We have already discussed the `/init` command.

* The `/chat` command is used to manage chat sessions with Gemini. It includes several subcommands:

  * `list`: Displays all previous chat commands.

  * `save`: Saves the current session with a specified tag (e.g., `/chat save "tag name for saving"`).

  * Other useful subcommands include `clear`, `compress`, `auth`, and more.

* The `/model` command allows us to select and switch the model that we want to use with the Gemini CLI.

* The `/stats model` command provides information about our API usage.

* The `/memory` command was discussed earlier; it includes subcommands such as `show`, `add`, `refresh`, and `list`.

## Context

This section discusses what context is, how it helps the AI model make decisions, and how to manage context within projects.

Context is used to inform Gemini about what is expected. For instance, if there is a need to change the font size in the file `Index.html`, one might specify:

> "Please increase the font size of paragraph text in `@Index.html`."

This instructs Gemini to limit the changes exclusively to `Index.html`.

## Adding an API Key

1. Obtain an API key from [Google AI Studio](https://www.google.com/search?q=https://aistudio.google.com/).

2. Run the `/auth` command in your terminal.

3. Enter your authentication key (API Key).

Ensure that the model in use matches the one selected in AI Studio; if not, access the settings and enable preview features.


