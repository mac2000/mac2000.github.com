---
layout: post
title: custom bash autocomplete
tags: [bash, autocomplete]
---

<amp-img src="/images/2020/bash_autocomplete/bash_autocomplete.gif" alt="bash autocomplete for custom function" width="643" height="431"></amp-img>

Often I do need to check which containers in a given kubernetes deployment in which status, to retrieve such info following command might be used:

```bash
kubectl get po -l app=my-awesome-app -o json | jq "[.items[0].status.containerStatuses[] | {name: .name, ready: .ready, state: .state | keys[0]}]"
```

which will return something like:

```bash
[
  {
    "name": "exporter",
    "ready": true,
    "state": "running"
  },
  {
    "name": "nginx",
    "ready": true,
    "state": "running"
  },
  {
    "name": "proxy",
    "ready": true,
    "state": "running"
  }
]
```

as you can imagine remembering such command is not possible, so we are going to create a function in `~/.bash_profile`:

```bash
po-kube() {
    kubectl get po -l app=$1 -o json | jq "[.items[0].status.containerStatuses[] | {name: .name, ready: .ready, state: .state | keys[0]}]"
}
```

and from now on we cat instead run:

```bash
po-kube my-awesome-app
```

but what if you have many deploymnets with long names?

here are pretty nice documentation of [how bash autocompletion works](https://debian-administration.org/article/316/An_introduction_to_bash_completion_part_1) and [how to create custom bash autocomplete](https://debian-administration.org/article/317/An_introduction_to_bash_completion_part_2)

In my case I added `/usr/local/etc/bash_completion.d/po-kube`:

```bash
{% raw %}
_po-kube() {
    local cur prev opts
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    opts=$(kubectl get deployment --template '{{range .items}}{{.metadata.name}}{{" "}}{{end}}')

    if [[ ${cur} == * ]] ; then
        COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
        return 0
    fi
}
complete -F _po-kube po-kube
{% endraw %}
```

and from now on I can do something like:

```bash
po-kube s[TAB]
sms-api spam-api
```
