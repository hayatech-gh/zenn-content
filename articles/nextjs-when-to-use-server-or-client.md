---
title: ""
emoji: "🎏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

| 用途                                                                       | Server Component | Client Component |
| -------------------------------------------------------------------------- | ---------------- | ---------------- |
| Fetch data                                                                 | ○                | ×                |
| Access backend resources (directly)                                        | ○                | ×                |
| Keep sensitive information on the server (access tokens, API keys, etc)    | ○                | ×                |
| Keep large dependencies on the server / Reduce client-side JavaScript      | ○                | ×                |
| Add interactivity and event listeners (onClick(), onChange(), etc)         | ×                | ○                |
| UseState と Lifecycle Effects (useState(), useReducer(), useEffect(), etc) | ×                | ○                |
| Use browser-only APIsdata                                                  | ×                | ○                |
| Use custom hooks that depend on state, effects, or browser-only APIs       | ×                | ○                |
| Use React Class components                                                 | ×                | ○                |
