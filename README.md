# Security Alert

보안 알림 전송을 위한 Python 기반 샌드박스 앱(Sandboxed Python application for delivering security alerts). `requests` / `urllib3` / `charset_normalizer` / `idna` 의존성을 번들링하여 외부 Python 환경 없이도 HTTP 요청 기반 알림을 전송할 수 있도록 구성된 경량 도구입니다.

A small, self-contained Python tool for security alert delivery. The app bundles its own HTTP stack (`requests`, `urllib3`, `charset_normalizer`, `idna`) under `lib/python3/` so it can run inside a sandboxed environment without relying on a system Python install.

## 한눈에 보기 / At a Glance

| 항목 / Item | 값 / Value |
| --- | --- |
| 제품 성격 / Product type | 샌드박스 실행형 Python 알림 앱 / Sandboxed Python notification app |
| 진입점 / Entry point | `security_alert/app.manifest` + `security_alert/lib/python3/` |
| 런타임 / Runtime | 임베디드 Python 3 (번들 의존성) / Embedded Python 3 (vendored deps) |
| 외부 의존성 / External deps | `requests 2.x`, `urllib3`, `charset_normalizer 3.4.4`, `idna 3.11` |
| 네트워크 / Network | HTTPS 기반 알림 발송 / HTTPS-based alert dispatch |
| 상태 / Status | 안정 (의존성 핀 고정) / Stable (deps pinned) |

### 운영 흐름 / Operator Flow

1. 운영자가 `security_alert/` 디렉터리에서 앱 매니페스트(`app.manifest`)를 로드합니다. / Operator loads the manifest at `security_alert/app.manifest`.
2. 런타임이 번들된 `lib/python3/` 모듈 경로를 우선 사용하도록 환경을 구성합니다. / Runtime prepends the bundled `lib/python3/` to `sys.path`.
3. 알림 페이로드를 `requests` / `urllib3` 스택으로 전송합니다. / Payload is dispatched via the bundled `requests` / `urllib3` stack.
4. 문자셋 감지는 `charset_normalizer`, IDN 도메인은 `idna` 로 처리합니다. / Charset detection via `charset_normalizer`; IDN via `idna`.

## 주요 특징 / Features

- 번들 의존성으로 시스템 Python 영향 없음 / Vendored deps isolate from host Python.
- HTTP/1.1 및 HTTP/2 지원 (`urllib3.http2`) / HTTP/1.1 + HTTP/2 ready (`urllib3.http2`).
- SOCKS / PyOpenSSL 확장 모듈 포함 (`urllib3.contrib.*`) / SOCKS / PyOpenSSL contrib modules shipped.
- IDN 호스트명 처리 / IDN hostname handling (`idna` 3.11).
- 응답 인코딩 자동 추정 / Response encoding auto-detection (`charset_normalizer` 3.4.4).

## 디렉터리 구조 / Repository Layout

| 경로 / Path | 역할 / Role |
| --- | --- |
| `security_alert/` | 앱 루트 / App root |
| `security_alert/README.md` | 상세 문서 진입점 / Detailed docs entry point |
| `security_alert/app.manifest` | 샌드박스 매니페스트 / Sandbox manifest |
| `security_alert/lib/python3/urllib3/` | HTTP 클라이언트 코어 / HTTP client core |
| `security_alert/lib/python3/requests/` | 상위 HTTP API / Higher-level HTTP API |
| `security_alert/lib/python3/charset_normalizer/` | 인코딩 감지 / Encoding detection |
| `security_alert/lib/python3/<dist-info>/` | 패키지 메타데이터 / Package metadata |

## 빠른 시작 / Quickstart

```bash
# 번들 모듈을 우선 사용하는 파이썬 호출
PYTHONPATH=security_alert/lib/python3 python3 -c "import requests, urllib3, charset_normalizer, idna; print('ok')"
```

샌드박스 런타임에서는 `security_alert/app.manifest` 의 진입점 정의에 따라 위 경로가 자동으로 `PYTHONPATH` 에 추가됩니다. / In a sandboxed runtime, the entry point declared in `app.manifest` prepends `lib/python3/` automatically.

## 설정 / Configuration

이 저장소는 설정 파일을 외부에 두지 않고 매니페스트와 코드 내부에서 관리합니다. 알림 대상 엔드포인트, 프록시, 인증 정보는 상위 호출자가 환경 변수 또는 매니페스트 인자로 전달하는 것을 가정합니다. / Configuration (endpoint, proxy, credentials) is expected to be supplied by the caller via environment variables or manifest args — this repo ships no separate config file.

## 로컬 개발 / Local Development

1. `security_alert/lib/python3/` 를 `PYTHONPATH` 에 추가합니다.
2. 프로젝트의 다른 모듈을 호출하는 진입 스크립트는 저장소에 포함되어 있지 않으므로, 운영자가 별도 래퍼를 제공해야 합니다.
3. 의존성을 수정할 경우 동일 버전(`charset_normalizer 3.4.4`, `idna 3.11`) 으로 핀을 유지합니다.

No wrapper scripts are committed in this snapshot; downstream operators provide their own entry point while reusing the vendored stack.

## 테스트 / Testing

번들 라이브러리의 자체 테스트 스위트는 포함되어 있지 않습니다. 운영 환경 검증은 매니페스트 로드 → 페이로드 발송 → 응답 코드 확인 순으로 수행합니다. / No bundled test suite ships here. Validate by loading the manifest, dispatching a payload, and asserting on the response code.

## 기여 / Contributing

기여 절차는 저장소 최상위 `CONTRIBUTING.md` 를 따릅니다. 의존성 버전 변경 시 `*.dist-info/METADATA` 와 라이브러리 디렉터리를 함께 갱신하세요. / Follow `CONTRIBUTING.md`. When bumping a dep, update both the `*.dist-info/METADATA` and the library directory.

## 라이선스 / License

`LICENSE` 파일을 참조하세요. 번들된 라이브러리(`urllib3`, `requests`, `charset_normalizer`, `idna`) 는 각자의 라이선스를 따르며 `*.dist-info/licenses/` 에 동봉되어 있습니다. / See `LICENSE`. Bundled libraries retain their own licenses shipped under `*.dist-info/licenses/`.

## 유지보수 / Maintainers

운영 및 변경 책임은 저장소 소유 조직에 있습니다. 자세한 연락처는 `CONTRIBUTING.md` 와 `security_alert/README.md` 를 확인하세요. / Ownership lives with the repository owner; see `CONTRIBUTING.md` and `security_alert/README.md` for contact details.

## 추가 문서 / Further Documentation

- 앱 상세: `security_alert/README.md`
- 매니페스트: `security_alert/app.manifest`
- 라이선스 원문: `LICENSE`, `security_alert/lib/python3/*/licenses/`