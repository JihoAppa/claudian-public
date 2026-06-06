---
title: "Kernel Policy DSL Object Lifetime Invariant"
section: "Method Cards"
summary: "A reusable invariant for nf_tables-style kernel policy DSL lifetime bugs."
---

# Kernel Policy DSL Object Lifetime Invariant

> [!tldr]
> Kernel policy DSL은 rule, set, element, verdict 같은 object가 transaction/GC/release path에서 하나의 lifetime owner를 가져야 한다.
> CVE-2024-0193과 CVE-2024-1086은 `nf_tables` 계열에서 value semantics와 object lifetime이 어긋난 사례다.
> Swarmer는 kernel DSL audit에서 parser correctness보다 transaction state machine과 release path를 먼저 모델링해야 한다.

## 정의

Kernel Policy DSL Object Lifetime Invariant는 `nf_tables`, eBPF, seccomp, filesystem policy parser처럼 kernel 안에서 작은 DSL을 해석하는 subsystem이 지켜야 하는 lifetime 규칙이다.

위험한 구조:

- rule/set/element가 transaction 중 active/deactivated state를 오간다.
- commit/abort/destroy/GC path가 같은 object ownership을 다르게 해석한다.
- catchall, verdict, anonymous set 같은 special case가 normal path와 다른 lifetime을 가진다.
- validation은 통과했지만 release path에서 value 의미가 바뀐다.
- module/user namespace/policy API reachability가 local attacker에게 열려 있다.

## 사례 매핑

| CVE | Subsystem | 깨진 축 |
|---|---|---|
| CVE-2024-0193 | `nf_tables` pipapo set backend | catchall element lifetime / GC / transaction state |
| CVE-2024-1086 | `nf_tables` verdict handling | value validation / double-free / UAF path |

## 분석 질문

- object가 active, deactivated, pending destroy, GC candidate 상태를 언제 오가는가?
- 한 transaction path에서 object를 free했는데 다른 path가 pointer/reference를 유지하는가?
- special element, catchall, anonymous object가 normal lifetime rule을 우회하는가?
- release path가 validation path와 같은 value semantics를 쓰는가?
- target config에서 해당 subsystem이 reachable한가?

## Swarmer Method Card

```yaml
method_card:
  id: kernel-policy-dsl-object-lifetime-invariant
  title: Kernel policy DSL objects need one owner across transaction and GC paths
  cve_ids: [CVE-2024-0193, CVE-2024-1086]
  affected_projects: [Linux kernel nf_tables]
  vulnerability_class: local-privilege-escalation
  confidence: high
  analysis_reverse_engineering:
    researcher_question: "Can a kernel policy object be freed, deactivated, or garbage-collected while still reachable through another semantic state?"
    entrypoints: [nf_tables, pipapo, catchall, set element, verdict, transaction commit, transaction abort, GC]
    trust_boundary: "local user or container workload -> kernel policy DSL API"
    sink: "use-after-free, double-free, stale policy object reference"
    missing_invariant: "object lifetime is not unified across validation, transaction, and release paths"
    patch_delta_signal: "refcount or state validation added; GC exclusion; normalized verdict range"
  reusable_pattern:
    trigger_questions:
      - "Does a special-case object have a different lifetime than normal objects?"
      - "Can abort/commit/GC race or reorder object release?"
      - "Are values normalized before they affect release paths?"
      - "Is the subsystem reachable in the target config?"
    expected_safe_behavior:
      - "Each object has a single lifetime owner across transaction states."
      - "GC cannot free objects still reachable by active or pending paths."
      - "Invalid values cannot reach release logic."
    false_positive_checks:
      - "Affected subsystem is disabled in kernel config."
      - "Required namespace or capability is unavailable."
      - "Fixed kernel contains the state/refcount validation."
    proof_strategy:
      - "Use authorized local/kernelCTF lab only."
      - "Verify version, config, reachability, and fixed behavior."
      - "Do not copy public exploit payloads into analysis context."
  swarmer_mapping:
    adapters: [kernel, linux, oss-c]
    stages: [scout, map, BCDA, BGA, P14-review]
    plugin_kind: runtime-invariant
    priority: high
```

## Related

- google-kernelctf-cve-2024-0193-netfilter-pipapo-uaf-26-06
- linux-nftables-cve-2024-1086-public-exploit-26-06
- [Linux-Public-Exploit-Triage]({{ site.baseurl }}/research/linux-public-exploit-triage/)
- [Google-KernelCTF-Research-Lane-26-06]({{ site.baseurl }}/research/google-kernelctf-research-lane-2026-06/)
- Swarmer

## Sources

- google-kernelctf-cve-2024-0193-netfilter-pipapo-uaf-26-06
- linux-nftables-cve-2024-1086-public-exploit-26-06
