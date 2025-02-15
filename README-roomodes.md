# .roomodes Feature Documentation

## Overview

The `.roomodes` feature in Roo-Code allows you to define project-specific custom modes. These custom modes can override global modes or add new modes specific to a project. The `.roomodes` file is placed in the root directory of your project and contains the configuration for these custom modes.

## Creating and Managing Custom Modes

To create and manage custom modes in `.roomodes`, follow these steps:

1. **Create the `.roomodes` file**: In the root directory of your project, create a file named `.roomodes`.

2. **Define custom modes**: Add the custom modes configuration in JSON format. Each mode should have a unique `slug`, a `name`, a `roleDefinition`, and an array of `groups`. Optionally, you can add `customInstructions`.

Example `.roomodes` file:
```json
{
  "customModes": [
    {
      "slug": "designer",
      "name": "Designer",
      "roleDefinition": "You are Roo, a UI/UX expert specializing in design systems and frontend development. Your expertise includes:\n- Creating and maintaining design systems\n- Implementing responsive and accessible web interfaces\n- Working with CSS, HTML, and modern frontend frameworks\n- Ensuring consistent user experiences across platforms",
      "groups": [
        "read",
        "edit",
        "browser",
        "command",
        "mcp"
      ],
      "customInstructions": "Additional instructions for the Designer mode"
    }
  ]
}
```

3. **Save the file**: Save the `.roomodes` file in the root directory of your project.

## Precedence of .roomodes over Global Settings

When both global custom modes and project-specific custom modes are defined, the project-specific modes in `.roomodes` take precedence. This means that if a mode with the same `slug` exists in both the global settings and `.roomodes`, the configuration in `.roomodes` will be used.

## Examples of Valid .roomodes Configurations

Here are some examples of valid `.roomodes` configurations:

### Example 1: Adding a New Mode
```json
{
  "customModes": [
    {
      "slug": "data-scientist",
      "name": "Data Scientist",
      "roleDefinition": "You are Roo, a data scientist with expertise in data analysis, machine learning, and statistical modeling.",
      "groups": [
        "read",
        "edit",
        "command"
      ],
      "customInstructions": "Focus on data analysis and machine learning tasks."
    }
  ]
}
```

### Example 2: Overriding an Existing Mode
```json
{
  "customModes": [
    {
      "slug": "code",
      "name": "Code",
      "roleDefinition": "You are Roo, a highly skilled software engineer with extensive knowledge in many programming languages, frameworks, design patterns, and best practices.",
      "groups": [
        "read",
        "edit",
        "browser",
        "command",
        "mcp"
      ],
      "customInstructions": "Additional instructions for the Code mode"
    }
  ]
}
```

### Example 3: Mode with File Restrictions
```json
{
  "customModes": [
    {
      "slug": "markdown-editor",
      "name": "Markdown Editor",
      "roleDefinition": "You are Roo, a markdown editor with expertise in editing and formatting markdown files.",
      "groups": [
        "read",
        ["edit", { "fileRegex": "\\.md$", "description": "Markdown files only" }],
        "browser"
      ],
      "customInstructions": "Focus on editing and formatting markdown files."
    }
  ]
}
```

By following these instructions, you can create and manage custom modes in your project using the `.roomodes` feature in Roo-Code.
