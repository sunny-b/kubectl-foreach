# kubectl koptica

Run a `kubectl` command in one or more contexts (clusters) in parallel
(similar to GNU parallel/xargs).

## Usage

```text
    kubectl koptica [OPTIONS] [PATTERN]... -- [KUBECTL_ARGS...]

Patterns can be used to match context names from kubeconfig:
      (empty): matches all contexts
         NAME: matches context with exact name
    /PATTERN/: matches context with regular expression
        ^NAME: remove context with exact name from the matched results
   ^/PATTERN/: remove contexts matching the regular expression from the results
    
Options:
    -c=NUM     Limit parallel executions (default: 0, unlimited)
    -I=VAL     Replace VAL occurring in KUBECTL_ARGS with context name
    -q         Suppress output
    -y         Disable and accept confirmation prompts ($KUBECTL_KOPTICA_DISABLE_PROMPTS) 
    -l         Display kube contexts matching the query and exit
    -h/--help  Print help
```

## Demo

Query a pod by label in `minikube` and `*-prod*` contexts:

```text
$ kubectl koptica /-prod/ minikube -- get pods -n kube-system --selector compute.twitter.com/app=coredns --no-headers

     eu-prod | coredns-59bd9867bb-6rbx7   2/2     Running   0          78d
     eu-prod | coredns-59bd9867bb-9xczh   2/2     Running   0          78d
     eu-prod | coredns-59bd9867bb-fvn6t   2/2     Running   0          78d
    minikube | No resources found in kube-system namespace.
 useast-prod | coredns-6fd4bd9db4-7w9wv   2/2     Running   0          78d
 useast-prod | coredns-6fd4bd9db4-9pk8n   2/2     Running   0          78d
 useast-prod | coredns-6fd4bd9db4-xphr4   2/2     Running   0          78d
 uswest-prod | coredns-6f987df9bc-6fgc2   2/2     Running   0          78d
 uswest-prod | coredns-6f987df9bc-9gxvt   2/2     Running   0          78d
 uswest-prod | coredns-6f987df9bc-d88jk   2/2     Running   0          78d
```

## Examples

**Match to contexts by name:** Run a command ("kubectl version") on contexts `c1`, `c2`
and `c3`:

```sh
kubectl koptica c1 c2 c3 -- version
```

**Match to contexts by pattern:** Run a command on contexts starting with `gke`
(regular expression syntax):

```sh
kubectl koptica /^gke/ -- get pods
```

**Match all contexts:** empty context matches all contexts.

```sh
kubectl koptica -- version
```

**Excluding contexts:** Use the matching syntaxes with a `^` prefix to use them
for exclusion. If no matching contexts are specified.

e.g. match all contexts **except** `c1` and except those ending
with `prod` (single quotes for escaping `$` in the shell):

```shell
kubectl koptica ^c1 ^/prod'$'/ -- version
```

**Using with kubectl plugins:** Customize how context name is passed to the command
(useful for kubectl plugins as `--context` must be specified after plugin name).

In this example, `_` is replaced with the context name when calling "kubectl
my_plugin".

```shell
kubectl koptica -I _ -- my_plugin -ctx=_
```

**Limit parallelization:** Only run 3 commands at a time:

```
kubectl koptica -c 3 /^gke-/
```

## Remarks

**Do not use this tool programmatically yet:**

This tool is not intended for deploying workloads to clusters, or using
programmatically. Therefore, it does not provide a structured output format or
ordered printing that is meant to be parsed by or piped to other programs (maybe
except for `grep`).
