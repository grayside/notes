---
title: 'Configuring Hugo Base URL in Codespaces'
date: '2024-01-27T20:07:57Z'
publishDate: '2023-12-23T20:07:57Z'
categories: ['til']

description: 'Starting the hugo dev server with the correct base URL in Codespaces.'
tags:
    - til
    - codespaces
    - hugo
    - shell
    - taskfile
draft: false
rssFullContent: true

# notes.grayside.dev
author: "grayside" 
showFullContent: false
color: "blue" # ["orange", "blue", "red", "green", "pink"]
---

Learning about [Codespaces](https://docs.github.com/en/codespaces/overview),
and more specifically, [Hugo](https://gohugo.io/) and [Taskfile](https://taskfile.dev)
in Codespaces.

My theme's starter gave me a devcontainer based on the (now archived)
[community devcontainer for Hugo](https://github.com/microsoft/vscode-dev-containers/blob/main/containers/hugo).

Running inside the codespace defined by this container, running `hugo serve`
with the Codespaces port forwarding works well. However, when run this way,
Hugo hasn't been configured with the correct base URL which could break
functionality dependent on the full URL, such as validating RSS or
single-origin scripts.

## Setting the right base URL in codespace dev

Some quick research found an
[example repository](https://github.com/shotor/hugo-codespaces-example?tab=readme-ov-file#debug-on-codespaces)
with a README that provided the tweaks needed for the hugo command:

```bash
hugo server --baseUrl "https://{githubUrl}" --appendPort=false
```

How do you get `githubUrl`?

There are a couple methods for this, such as going to the browser preview tab
and copying the base URL. Pulling that into your codespace terminal, that might look like:

```bash
githubUrl=stellar-codespace-id-sn2ggl3bl4rn3y-1313.app.github.dev
hugo server --baseUrl "https://{githubUrl}" --appendPort=false
```

However, that's a manual step that's needed almost per work session.

## Zero Config Version

Inspecting the terminal environment of the codespace, it turns out the pieces
of the codespace domain are available via built-in environment variables:

- **app.github.dev** is sourced from `GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN`
- **stellar-codespace-id-sn2ggl3bl4rn3y** is sourced from `CODESPACE_NAME`

Leaning on these variables, we can simplify the command to something with zero
additional configuration:

```bash
hugo server --baseURL "https://${CODESPACE_NAME}-1313.${GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN}" --appendPort=false
```

Note the browser preview on this command injects the port as part of the domain, it ends up looking like:

```bash
https://${CODESPACE_NAME}-1313.${GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN}
```

Where 1313 is the hugo server default port.

## Bringing this to Taskfile

I’ve started exploring the use of [taskfile.dev](http://taskfile.dev) for task management,
and an adaptation of this command might look like:

```bash
version: '3'

tasks:
  cs-serve:
    cmds:
      - |
        hugo server --appendPort=false --baseURL \
          "https://{{ .CODESPACE_NAME }}-1313.{{ .GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN }}" 
    requires:
      vars: [CODESPACE_NAME, GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN]
```

Rather than make [direct use of environment variables](https://taskfile.dev/usage/#env-files),
I’m using [taskfile variables](https://taskfile.dev/usage/#variables)
that inherit values from the environment. This way I can make these special
codespace environment variables required before execution without adding
more complex shell code.
