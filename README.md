# git-flow observer

```yaml
# .github/workflows/git-flow-observer.yml

name: "git-flow observer"

on: pull_request

jobs:
  git-flow-observer:
    if: github.head_ref != 'main' || github.base_ref != 'develop'
    runs-on: ubuntu-latest
    steps:
      - name: "Observe"
        uses: noraworld/git-flow-observer@latest
        with:
          head: "develop"
          base: "main"
```
