---
title: "Google kernelCTF Submission Contract"
section: "Method Cards"
summary: "The kernelCTF submission contract as a reproducibility and proof-quality checklist."
---

# Google kernelCTF Submission Contract

> [!tldr]
> kernelCTF는 exploit 성공 여부뿐 아니라 target config, stability, disclosure timing, documentation structure를 함께 요구하는 계약이다.
> 이 계약은 Swarmer의 BGA/P14 레인에 "proof quality"와 "portability" 체크리스트로 들어가야 한다.
> 핵심은 payload 생성이 아니라 reproducibility, target constraint, vulnerability/exploit documentation separation이다.

## 정의

Google kernelCTF Submission Contract는 Linux kernel vulnerability exploit 연구를 공개 가능한 구조로 제출하기 위한 규칙 묶음이다.

중요한 contract field:

- target kernel version and config
- disabled features: user namespace, `io_uring`, `nftables` 등
- reliability requirement and stability bonus
- 0-day vs 1-day timing and disclosure protection window
- vulnerability documentation vs exploit documentation separation
- public PR structure and metadata schema
- kernelXDK portability requirement

## Swarmer 적용

Swarmer는 kernelCTF 문서를 다음 방식으로 써야 한다.

| Lane | 적용 |
|---|---|
| scout | target config와 disabled feature를 candidate reachability에 반영 |
| map | CVE root cause와 exploit technique을 별도 graph로 분리 |
| BCDA | duplicate, feature-disabled, version mismatch를 false-positive check로 처리 |
| BGA | owned lab/kernelCTF environment만 proof target으로 허용 |
| P14 review | reliability, runtime limit, kernelXDK, disclosure deadline을 checklist화 |

## 분석 질문

- 이 CVE는 current LTS target의 config에서 reachable한가?
- exploit이 target-specific offset에 의존하는가, kernelXDK로 추상화 가능한가?
- vulnerability docs와 exploit docs가 분리되어 있는가?
- stability가 10%/90% threshold를 충족하는 evidence가 있는가?
- 0-day/1-day disclosure timing이 reward eligibility와 충돌하지 않는가?

## Related

- google-kernelctf-rules-26-06
- [Google-KernelCTF-Research-Lane-26-06]({{ site.baseurl }}/research/google-kernelctf-research-lane-2026-06/)
- [Linux-Public-Exploit-Triage]({{ site.baseurl }}/research/linux-public-exploit-triage/)
- Swarmer

## Sources

- google-kernelctf-rules-26-06
