PS C:\Users\Jose Montalvo\Documents\GitHub\k8s-enterprise-capstone-team1> trivy image --severity HIGH,CRITICAL nginx:latest
2026-04-10T23:22:37-04:00       INFO    [vuln] Vulnerability scanning is enabled
2026-04-10T23:22:37-04:00       INFO    [secret] Secret scanning is enabled
2026-04-10T23:22:37-04:00       INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2026-04-10T23:22:37-04:00       INFO    [secret] Please see https://trivy.dev/docs/v0.69/guide/scanner/secret#recommendation for faster secret detection
2026-04-10T23:22:38-04:00       INFO    Detected OS     family="debian" version="13.4"
2026-04-10T23:22:38-04:00       INFO    [debian] Detecting vulnerabilities...   os_version="13" pkg_num=151
2026-04-10T23:22:38-04:00       INFO    Number of language-specific files       num=0
2026-04-10T23:22:38-04:00       WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/docs/v0.69/guide/scanner/vulnerability#severity-selection for details.

Report Summary

┌────────────────────────────┬────────┬─────────────────┬─────────┐
│           Target           │  Type  │ Vulnerabilities │ Secrets │
├────────────────────────────┼────────┼─────────────────┼─────────┤
│ nginx:latest (debian 13.4) │ debian │       15        │    -    │
└────────────────────────────┴────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


nginx:latest (debian 13.4)
==========================
Total: 15 (HIGH: 15, CRITICAL: 0)

┌─────────────────────────┬────────────────┬──────────┬──────────┬───────────────────┬─────────────────┬──────────────────────────────────────────────────────────────┐
│         Library         │ Vulnerability  │ Severity │  Status  │ Installed Version │  Fixed Version  │                            Title                             │
├─────────────────────────┼────────────────┼──────────┼──────────┼───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libde265-0              │ CVE-2026-33164 │ HIGH     │ affected │ 1.0.15-1+b3       │                 │ libde265 is an open source implementation of the h.265 video │
│                         │                │          │          │                   │                 │ codec. Pr...                                                 │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-33164                   │
├─────────────────────────┼────────────────┤          │          ├───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libexpat1               │ CVE-2026-25210 │          │          │ 2.7.1-2           │                 │ libexpat: libexpat: Information disclosure and data          │
│                         │                │          │          │                   │                 │ integrity issues due to integer overflow...                  │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-25210                   │
├─────────────────────────┼────────────────┤          │          ├───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libheif-plugin-dav1d    │ CVE-2025-68431 │          │          │ 1.19.8-1          │                 │ libheif is an HEIF and AVIF file format decoder and encoder. │
│                         │                │          │          │                   │                 │ Prior...                                                     │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2025-68431                   │
├─────────────────────────┤                │          │          │                   ├─────────────────┤                                                              │
│ libheif-plugin-libde265 │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
├─────────────────────────┤                │          │          │                   ├─────────────────┤                                                              │
│ libheif1                │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
├─────────────────────────┼────────────────┤          │          ├───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libnghttp2-14           │ CVE-2026-27135 │          │          │ 1.64.0-1.1        │                 │ nghttp2: nghttp2: Denial of Service via malformed HTTP/2     │
│                         │                │          │          │                   │                 │ frames after session termination...                          │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-27135                   │
├─────────────────────────┼────────────────┤          ├──────────┼───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libssl3t64              │ CVE-2026-28390 │          │ fixed    │ 3.5.5-1~deb13u1   │ 3.5.5-1~deb13u2 │ openssl: OpenSSL: Denial of Service due to NULL pointer      │
│                         │                │          │          │                   │                 │ dereference in CMS...                                        │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-28390                   │
├─────────────────────────┼────────────────┤          ├──────────┼───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libsystemd0             │ CVE-2026-29111 │          │ affected │ 257.9-1~deb13u1   │                 │ systemd: systemd: Arbitrary code execution or Denial of      │
│                         │                │          │          │                   │                 │ Service via spurious IPC...                                  │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-29111                   │
├─────────────────────────┼────────────────┤          ├──────────┼───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libtiff6                │ CVE-2026-4775  │          │ fixed    │ 4.7.0-3+deb13u1   │ 4.7.0-3+deb13u2 │ libtiff: libtiff: Arbitrary code execution or denial of      │
│                         │                │          │          │                   │                 │ service via signed integer...                                │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-4775                    │
├─────────────────────────┼────────────────┤          ├──────────┼───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libtinfo6               │ CVE-2025-69720 │          │ affected │ 6.5+20250216-2    │                 │ ncurses: ncurses: Buffer overflow vulnerability may lead to  │
│                         │                │          │          │                   │                 │ arbitrary code execution.                                    │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2025-69720                   │
├─────────────────────────┼────────────────┤          │          ├───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ libudev1                │ CVE-2026-29111 │          │          │ 257.9-1~deb13u1   │                 │ systemd: systemd: Arbitrary code execution or Denial of      │
│                         │                │          │          │                   │                 │ Service via spurious IPC...                                  │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-29111                   │
├─────────────────────────┼────────────────┤          │          ├───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ ncurses-base            │ CVE-2025-69720 │          │          │ 6.5+20250216-2    │                 │ ncurses: ncurses: Buffer overflow vulnerability may lead to  │
│                         │                │          │          │                   │                 │ arbitrary code execution.                                    │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2025-69720                   │
├─────────────────────────┤                │          │          │                   ├─────────────────┤                                                              │
│ ncurses-bin             │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
├─────────────────────────┼────────────────┤          ├──────────┼───────────────────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ openssl                 │ CVE-2026-28390 │          │ fixed    │ 3.5.5-1~deb13u1   │ 3.5.5-1~deb13u2 │ openssl: OpenSSL: Denial of Service due to NULL pointer      │
│                         │                │          │          │                   │                 │ dereference in CMS...                                        │
│                         │                │          │          │                   │                 │ https://avd.aquasec.com/nvd/cve-2026-28390                   │
├─────────────────────────┤                │          │          │                   │                 │                                                              │
│ openssl-provider-legacy │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
│                         │                │          │          │                   │                 │                                                              │
└─────────────────────────┴────────────────┴──────────┴──────────┴───────────────────┴─────────────────┴──────────────────────────────────────────────────────────────┘