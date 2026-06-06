---
title: "Google kernelCTF research lane — 2026-06"
section: "Security Research"
summary: "Google kernelCTF rules and CVE-2024-0193 distilled into a reusable kernel research lane."
---

# Google kernelCTF research lane — 2026-06

> [!tldr]
> kernelCTF는 public exploit corpus이자 reproducibility contract다.
> CVE-2024-0193은 `nf_tables` policy DSL object lifetime을 역설계하는 좋은 seed이며, rules 문서는 target constraint와 proof quality를 정의한다.
> Claudian은 kernelCTF 문서를 method card로 증류하고, Swarmer는 이를 kernel lane의 scout/map/BCDA/BGA/P14 context로 소비해야 한다.

## 결론

이번 inbox 2건은 단순 clipping이 아니라 swarmer kernel lane 설계 자료다.

| 문서 | 역할 | 내재화 결과 |
|---|---|---|
| kernelCTF rules | target/reward/submission/reliability contract | [Google-KernelCTF-Submission-Contract]({{ site.baseurl }}/research/google-kernelctf-submission-contract/) |
| CVE-2024-0193 kernelCTF docs | public exploit-backed Linux kernel UAF case | [Kernel-Policy-DSL-Object-Lifetime-Invariant]({{ site.baseurl }}/research/kernel-policy-dsl-object-lifetime-invariant/) |

## CVE skill 판단

`CVE-2024-0193_lts` 문서는 Google security-research의 kernelCTF PoC 경로이므로 CVE-분석방법-역설계 대상이다. 다만 실제 exploit payload를 위키에 보존하지 않고, 다음 지식만 추출한다.

- affected subsystem: `nf_tables` / pipapo set backend
- root class: object lifetime / UAF
- key special case: catchall element and transaction/GC boundary
- proof context: kernelCTF or authorized local lab only
- false-positive checks: target config, feature availability, fixed kernel, namespace/capability requirement

## Swarmer lane injection

| Stage | 주입할 지식 |
|---|---|
| scout | `nf_tables`, `pipapo`, catchall, set element, transaction, GC keyword |
| map | object lifetime state machine and release path graph |
| BCDA | target config reachability, disabled feature, duplicate/known CVE checks |
| BGA | kernelCTF/owned lab proof boundary and fixed-behavior verification |
| P14 review | kernelCTF rules, public PoC signal, reliability/runtime/documentation checklist |

## 확장 조사 방향

다음 kernelCTF corpus 처리부터는 CVE별로 세 가지를 수집한다.

1. `docs/vulnerability.md`: root cause, affected subsystem, patch/versions.
2. `docs/exploit.md`: technique taxonomy only, no payload copy.
3. `metadata.json`: target version, reliability, kernelXDK usage, public proof metadata.

이 구조를 유지하면 swarmer가 exploit code를 직접 학습하지 않아도 "어떻게 분석했을까?"를 충분히 재사용할 수 있다.

## Related

- [Google-KernelCTF-Submission-Contract]({{ site.baseurl }}/research/google-kernelctf-submission-contract/)
- [Kernel-Policy-DSL-Object-Lifetime-Invariant]({{ site.baseurl }}/research/kernel-policy-dsl-object-lifetime-invariant/)
- [Linux-Public-Exploit-Triage]({{ site.baseurl }}/research/linux-public-exploit-triage/)
- CVE-분석방법-역설계
- [Linux-Public-Exploit-CVE-Collection-26-06]({{ site.baseurl }}/research/linux-public-exploit-cve-collection-2026-06/)
- Claudian-Swarmer-Knowledge-Bridge-26-06
- Swarmer

## Sources

- google-kernelctf-rules-26-06
- google-kernelctf-cve-2024-0193-netfilter-pipapo-uaf-26-06
