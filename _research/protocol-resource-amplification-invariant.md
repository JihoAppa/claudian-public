---
title: "Protocol Resource Amplification Invariant"
section: "Method Cards"
summary: "A resource-amplification invariant for protocol parsers and HTTP/2 Bomb style bugs."
---

# Protocol Resource Amplification Invariant

> [!tldr]
> 프로토콜 parser는 decoded size뿐 아니라 object count, allocation lifetime, stalled stream retention을 동시에 제한해야 한다.
> HTTP/2 Bomb은 오래 알려진 compression bomb과 Slowloris hold가 결합될 때 전혀 다른 blast radius가 생긴다는 사례다.
> Swarmer는 protocol implementation audit에서 단일 limit이 아니라 limit matrix를 검증해야 한다.

## 정의

Protocol Resource Amplification Invariant는 작은 wire representation이 큰 내부 상태를 만들고, 그 상태가 오래 유지될 수 있는 protocol implementation에서 반드시 지켜야 하는 자원 경계 규칙이다.

안전한 구현은 다음 네 축을 동시에 제한해야 한다.

1. wire payload size
2. decoded payload size
3. internal object count / bookkeeping allocation
4. allocation lifetime / stalled stream retention

## 분석 질문

- 작은 compressed token이 많은 header/object/allocation으로 확장되는가?
- decoded-size limit은 있지만 object-count limit이 없는가?
- flow-control, timeout, backpressure, half-open stream이 cleanup을 지연하는가?
- spec이 split header, duplicate header, continuation frame 같은 예외를 허용하는가?
- 여러 구현체가 같은 spec 해석 때문에 같은 bug class를 공유하는가?

## Swarmer Method Card

```yaml
method_card:
  id: protocol-resource-amplification-invariant
  title: Protocol parsers must bound size, object count, and allocation lifetime together
  cve_ids: [CVE-2026-49975]
  affected_projects: [Apache httpd, nginx, Microsoft IIS, Envoy, Cloudflare Pingora]
  vulnerability_class: resource-exhaustion
  confidence: high
  analysis_reverse_engineering:
    researcher_question: "Can two known protocol resource primitives compose into a larger DoS class?"
    entrypoints: [HTTP/2 header decode, HPACK dynamic table, flow-control window, stream timeout]
    trust_boundary: "unauthenticated network client -> protocol parser and stream state"
    sink: "server memory allocation retained by stalled stream"
    missing_invariant: "object-count and allocation-lifetime limits are missing or not coupled to decoded-size limits"
    patch_delta_signal: "new max header count, split cookie accounting, stalled stream bounds"
    likely_search_queries:
      - "HPACK"
      - "SETTINGS_HEADER_TABLE_SIZE"
      - "WINDOW_UPDATE"
      - "LimitRequestFields"
      - "max_headers"
      - "cookie"
    negative_checks:
      - "hard cap on header field count including split cookies"
      - "bounded lifetime for stalled streams"
      - "per-connection and per-worker memory limits"
      - "HTTP/2 disabled or terminated by protected edge"
  reusable_pattern:
    trigger_questions:
      - "Are size limits and count limits separate?"
      - "Can the client cheaply keep server allocations live?"
      - "Can old DoS primitives compose across protocol features?"
    expected_safe_behavior:
      - "Small wire input cannot create unbounded internal objects."
      - "Stalled streams time out without retaining large allocations."
      - "Default configuration includes safe caps."
    false_positive_checks:
      - "HTTP/2 is disabled."
      - "Edge proxy normalizes and caps header count."
      - "Implementation frees request allocations before response flow-control stalls."
    proof_strategy:
      - "Use authorized lab only."
      - "Measure memory accounting under bounded, non-production HTTP/2 requests."
      - "Verify rejection/timeout behavior rather than weaponized throughput."
  swarmer_mapping:
    adapters: [protocol, web-http, oss-c, oss-rust, oss-js]
    stages: [scout, map, BCDA, BGA, P14-review]
    plugin_kind: cve-pattern
    priority: high
```

## Related

- http2-bomb-cve-2026-49975-protocol-resource-amplification-26-06
- [CVE-분석방법-증류-광범위취약점-26-06]({{ site.baseurl }}/research/broad-scope-cve-method-distillation-2026-06/)
- Swarmer

## Sources

- http2-bomb-cve-2026-49975-protocol-resource-amplification-26-06
