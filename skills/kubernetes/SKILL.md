---
name: kubernetes
description: Use when inspecting, diagnosing, or explaining Kubernetes clusters, workloads, pods, services, deployments, events, logs, contexts, namespaces, or kubectl CLI commands.
---

# Kubernetes

Use this skill for Kubernetes work involving `kubectl`, cluster inspection, workload diagnosis, resource status, and read-only troubleshooting.

## CLI Rules

- Prefer `kubectl get` and `kubectl describe` for first-pass inspection.
- `kubectl get *` and `kubectl describe *` are pre-approved read-only commands.
- Include `--namespace <name>` when namespace is known.
- Include `--context <name>` when context is known or user names cluster.
- Use `-o wide` for node placement, IPs, and quick status scans.
- Use `-o yaml` or `-o json` only when fields, labels, owner refs, or conditions need exact inspection.
- Do not run mutating commands such as `apply`, `delete`, `patch`, `scale`, `rollout restart`, `cordon`, `drain`, or `taint` unless user explicitly asks and permission allows it.
- Do not switch contexts or namespaces globally with `kubectl config set-context`; pass `--context` and `--namespace` per command instead.

## Inspection Patterns

Pods in namespace:

```bash
kubectl get pods --namespace <namespace> -o wide
kubectl describe pod <pod> --namespace <namespace>
```

Deployment health:

```bash
kubectl get deployment <name> --namespace <namespace> -o wide
kubectl describe deployment <name> --namespace <namespace>
```

Service and endpoints:

```bash
kubectl get service <name> --namespace <namespace> -o wide
kubectl get endpoints <name> --namespace <namespace> -o wide
```

Events:

```bash
kubectl get events --namespace <namespace> --sort-by=.lastTimestamp
```

Nodes:

```bash
kubectl get nodes -o wide
kubectl describe node <node>
```

## Diagnosis Style

- State observed resource, namespace, and context when known.
- Separate facts from likely causes.
- Prefer smallest next read command that can confirm or reject hypothesis.
- If command output shows secrets, tokens, credentials, or private data, do not repeat sensitive values.
