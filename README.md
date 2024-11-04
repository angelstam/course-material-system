# Course Material System
A system for handling course material as markdown in GitHub repos.

## Course Material
Course material should be split into small reuseable parts, topics. Each topic
is categorized and stored in GitHub repos together with other related topics.

```
repo/<subject>/<subarea>/<topic>.md
```

- A subject could be a programming language or a framework.

- A subarea could be a concept in a language or some part of a framework.

- A topic could be a specific keyword or construct in a language of a specific
  API in a framework.

```mermaid
classDiagram
    Topic "*" *-- "1" Subarea
    Subarea "*" *-- "1" Subject
    class Topic{
        path
        sha/tag
        repo
        preview
        description
    }
    class Subarea{
        name
        path
        repo
        index
    }
    class Subject{
        name
        index
    }
```

## Use Case: Public End Users
In this use case users are unauthenticated and could be for example course
participants or previous employees.

```mermaid
C4Context
    title System context diagram Course Material for public end users

    Boundary(b0, "End users (public)", "internet") {
        Person_Ext(userA, "Course Participant", "")
        Person_Ext(userB, "Previous Employee")

        System(SystemAPI, "Public API", "Allows access to Markdown and Themes for existing presentations using static URL.")

        %% Static content from public API
        Rel(SystemAPI, userA, "Sends static content", "HTTP")
        Rel(SystemAPI, userB, "Sends static content", "HTTP")

        %% Inter system communication
        Rel(SystemGithubRAW, SystemAPI, "", "HTTP")
        BiRel(SystemGithubAPI, SystemService, "Uses", "HTTP")
        BiRel(SystemService, SystemAPI, "Uses", "HTTP")

        System_Boundary(b1, "Internal") {
            System(SystemService, "Backend", "Handle communication with the GitHub API.")
        }

        System_Boundary(bGithub, "GitHub") {
            System_Ext(SystemGithubRAW, "GitHub Raw", "Direct links to raw markdown, theme or other resource.")
            System_Ext(SystemGithubAPI, "GitHub API", "If we need it to get links to raw resources")
        }
    }

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="2")
```

### Public API
This API does not require authenticatio. It acts like a proxy and possibly a
cache for resources that would otherwise require authentication. Is is used to
give public access to markdown, themes and resources used in presentations.

Markdown and other resources is fetched from GitHub using raw files from the
main GitHub site or using the GitHub API and
[octokit.js](https://github.com/octokit/octokit.js).

Access to resources using raw files could possibly just redirect to GitHub,
as access to to each raw files is using a unique token per file.


## Use Case: Employees
In this use case users are authenticated and could be for example employees.

```mermaid
C4Context
    title System context diagram Course Material for employees

    Boundary(b0, "(public)", "internet") {
        System(SystemAPI, "Public API", "Allows access to Markdown and Themes for existing presentations using static URL.")

        Rel(SystemGithubRAW, SystemAPI, "Uses raw static content", "HTTP")
        UpdateRelStyle(SystemGithubRAW, SystemAPI, $offsetY="130", $offsetX="-140")

        Enterprise_Boundary(b1, "Employees") {
            Person(employeeA, "Employee", "Employee using the couse material.")
            Person(employeeB, "Employee Moderator", "Employee moderating the couse material.")

            System(SystemUI, "UI", "Enable employees to Search, Preview and Contribute Markdown.")

            %% Static content from public API
            Rel(SystemAPI, employeeA, "Uses", "HTTP")
            UpdateRelStyle(SystemAPI, employeeA, $offsetY="-40")
            Rel(SystemAPI, employeeB, "Uses", "HTTP")
            UpdateRelStyle(SystemAPI, employeeB, $offsetY="-40")

            %% Interaction with UI
            BiRel(SystemUI, employeeA, "Uses", "HTTP")
            BiRel(SystemUI, employeeB, "Uses", "HTTP")

            %% Interaction with the GitHub UI
            BiRel(SystemGithub, employeeB, "Uses", "HTTP")

            %% Inter system communication
            BiRel(SystemService, SystemUI, "Uses", "HTTP")
            BiRel(SystemDB, SystemService, "Uses", "SQL")
            BiRel(SystemGithubAPI, SystemService, "Uses", "HTTP")
        }

        System_Boundary(bGithub, "GitHub") {
            System_Ext(SystemGithubAPI, "GitHub API", "Handle issues for contributions, list repo content, etc.")
            System_Ext(SystemGithubRAW, "GitHub Raw", "Direct links to raw markdown, theme or other resource.")
            System_Ext(SystemGithub, "GitHub UI", "Handle issues for contributions in the GitHub UI.")
        }

        System_Boundary(b2, "Backend") {
            SystemDb(SystemDB, "DB", "Database that stores indexes and references to Markdown.")
            System(SystemService, "Backend Service", "Handle issues for contributions, list repo content, etc.")
        }
    }

    UpdateLayoutConfig($c4ShapeInRow="2", $c4BoundaryInRow="1")
```

### Employee
Regular employees use the [Public API](#public-api) and an internal UI that
require authentication.

### Employee Moderator
Employees that moderate course material use the Public API and the internal UI.
To moderate contributions from regualar employees they use GitHubs UI for issues.

### Backend Service
- Handles access to the GitHub API.
- Indexing and preview of markdown and themes.
- Create issues thru the GitHub API for contributions to markdown and themes.
- Provides search for markdown.
- Manage presentations in local DB or possibly on GitHub.


## The presentation data structure
```mermaid
classDiagram
    Presentation *-- Theme
    Presentation *-- CustomSlide
    Presentation *-- ContentAtomSlide
    Presentation : name
    Presentation : template
    Presentation : slides[]
    class Theme{
      name
      template
    }
    class CustomSlide{
      content
    }
    class ContentAtomSlide{
      link
    }
```

### Theme
This is either the name of a predefined theme in
[reveal.js](https://revealjs.com/) or a custom theme.

### Slides
This is the content of the presentation.
Slides can be created either specific for this presentation or be a reference
to a moderated slide from existing course material.
