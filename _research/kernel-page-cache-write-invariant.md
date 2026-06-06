---
title: "Kernel Page Cache Write Invariant"
section: "Method Cards"
summary: "A shared invariant for Dirty COW, Dirty Pipe, Copy Fail, and Dirty Frag style bugs."
---

# Kernel Page Cache Write Invariant

> [!tldr]
> kernel fast path가 page-backed memory에 in-place write를 하기 전에는 page ownership, COW state, reference semantics를 증명해야 한다.
> Dirty COW, Dirty Pipe, Copy Fail, Dirty Frag는 서로 다른 subsystem에서 같은 "read-only/shared page가 attacker-writable이 되는가" 질문으로 묶인다.
> Swarmer는 kernel audit에서 reachability와 operational mitigation blast radius를 함께 다뤄야 한다.

## 정의

Kernel Page Cache Write Invariant는 Linux kernel의 network, crypto, splice, filesystem, page-cache 경로에서 shared page를 private mutable buffer처럼 쓰면 안 된다는 원칙이다.

취약한 구조는 다음과 같다.

- local user가 kernel path에 attacker-controlled bytes를 넣을 수 있다.
- kernel fast path가 packet/fragment/crypto output을 in-place로 쓴다.
- target page가 page-cache-backed 또는 shared/COW-sensitive memory다.
- write primitive가 read-only file cache나 executable mapping에 영향을 준다.
- user namespace, module autoloading, socket reachability로 trigger path가 열린다.

## 대표 사례

| 사례 | CVE | 깨진 축 |
|---|---|---|
| Dirty COW | CVE-2016-5195 | COW race가 read-only private mapping write로 전환 |
| Dirty Pipe | CVE-2022-0847 | stale pipe buffer flag가 read-only file page cache write로 전파 |
| Copy Fail | CVE-2026-31431 | `AF_ALG`/`splice`/crypto in-place optimization이 page-cache write로 전환 |
| Dirty Frag | CVE-2026-43284, CVE-2026-43500, CVE-2026-46300 | network/crypto/fragment path가 shared page ownership을 증명하지 못함 |

## 분석 질문

- 이 write path는 page ownership을 어떻게 증명하는가?
- shared fragment와 page-cache page가 같은 buffer로 취급되는가?
- `splice`, crypto decrypt, network fragment assembly가 page cache와 만나는가?
- local account나 container workload가 trigger 가능한가?
- mitigation이 운영 기능을 깨뜨리는가?

## Swarmer Method Card

```yaml
method_card:
  id: kernel-page-cache-write-invariant
  title: Kernel in-place writes must prove page ownership and COW state
  cve_ids: [CVE-2016-5195, CVE-2022-0847, CVE-2026-31431, CVE-2026-43284, CVE-2026-43500, CVE-2026-46300]
  affected_projects: [Linux kernel]
  vulnerability_class: local-privilege-escalation
  confidence: high
  analysis_reverse_engineering:
    researcher_question: "Which kernel fast paths can write attacker-controlled bytes into page-cache-backed memory?"
    entrypoints: [get_user_pages, madvise, pipe_buffer, splice, AF_ALG, algif_aead, xfrm ESP, esp4, esp6, rxrpc, skb frag, crypto decrypt]
    trust_boundary: "local unprivileged user -> kernel networking/crypto page path"
    sink: "in-place write to shared or page-cache-backed memory"
    missing_invariant: "page ownership and COW state not proven before write"
    patch_delta_signal: "copy-before-write, ownership/refcount validation, module/namespace hardening"
    likely_search_queries:
      - "copy-on-write"
      - "pipe_buffer.flags"
      - "copy_page_to_iter_pipe"
      - "AF_ALG"
      - "algif_aead"
      - "splice"
      - "skb_frag"
      - "MSG_SPLICE_PAGES"
      - "xfrm"
      - "esp4"
      - "esp6"
      - "rxrpc"
      - "page cache"
    negative_checks:
      - "affected modules unavailable or blocked"
      - "unprivileged user namespaces disabled where relevant"
      - "copy-on-write enforced before mutation"
      - "local account/container trigger path absent"
  reusable_pattern:
    trigger_questions:
      - "Can local users reach network/crypto paths that mutate shared pages?"
      - "Does the write target originate from page cache or a shared fragment?"
      - "Can the primitive affect executable/read-only file cache?"
    expected_safe_behavior:
      - "Writable private copy is made before mutation."
      - "Module availability and namespace policies limit reachability."
      - "Mitigation does not silently break required workloads without review."
    false_positive_checks:
      - "Kernel version contains fixed copy-before-write semantics."
      - "Protocol modules are not loaded and cannot autoload."
      - "No local unprivileged trigger context exists."
    proof_strategy:
      - "Use authorized local lab only."
      - "Verify kernel version, module state, namespace policy, and mitigation status."
      - "Avoid running public LPE payloads on production hosts."
  swarmer_mapping:
    adapters: [kernel, linux, oss-c]
    stages: [scout, map, BCDA, BGA, P14-review]
    plugin_kind: runtime-invariant
    priority: high
```

## Related

- linux-dirty-frag-cve-2026-43284-43500-46300-page-cache-lpe-26-06
- linux-copy-fail-cve-2026-31431-public-exploit-26-06
- linux-dirty-pipe-cve-2022-0847-public-exploit-26-06
- linux-dirty-cow-cve-2016-5195-public-exploit-26-06
- [Linux-Public-Exploit-Triage]({{ site.baseurl }}/research/linux-public-exploit-triage/)
- [CVE-분석방법-증류-광범위취약점-26-06]({{ site.baseurl }}/research/broad-scope-cve-method-distillation-2026-06/)
- [Linux-Public-Exploit-CVE-Collection-26-06]({{ site.baseurl }}/research/linux-public-exploit-cve-collection-2026-06/)
- Swarmer

## Sources

- linux-dirty-frag-cve-2026-43284-43500-46300-page-cache-lpe-26-06
- linux-copy-fail-cve-2026-31431-public-exploit-26-06
- linux-dirty-pipe-cve-2022-0847-public-exploit-26-06
- linux-dirty-cow-cve-2016-5195-public-exploit-26-06
