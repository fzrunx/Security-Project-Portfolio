# 🛡️ API Security Monitoring & Automated Response System

**Wazuh · Shuffle · pfSense 기반 API 위협 탐지 및 대응 시스템**

> 웹 서비스에서 발생하는 비정상적인 API 요청을 탐지하고, 공격 유형에 따라 자동 차단 또는 관리자 승인 기반으로 대응하는 보안 모니터링 시스템입니다.



## 📌 Project Overview

웹 서비스가 운영되면서 발생하는 비정상적인 API 접근과 반복적인 요청을 탐지하고, 보안 이벤트에 신속하게 대응할 수 있도록 시스템을 구축했습니다.

Wazuh의 룰 기반 로그 탐지를 중심으로 Shuffle을 통한 대응 흐름을 구성했으며, pfSense 방화벽과 연동하여 IP 차단을 수행합니다.

| 구분 | 내용 |
|:---|:---|
| 프로젝트 | API Security Monitoring & Automated Response |
| 개발 형태 | 팀 프로젝트 |
| 주요 목적 | API 위협 탐지 및 대응 자동화 |
| 대상 서비스 | PHP 기반 쇼핑몰 및 API 서버 |
| 주요 기술 | Wazuh, Shuffle, pfSense, Grafana |


## 🎯 Project Goals

- **API 위협 탐지** — Wazuh 룰을 활용한 비정상적인 요청 및 공격 이벤트 탐지
- **대응 프로세스 구성** — 공격 유형에 따른 자동 차단과 관리자 승인 방식 분리
- **네트워크 보안 강화** — pfSense를 활용한 네트워크 분리 및 IP 차단
- **보안 현황 시각화** — Grafana를 통한 탐지 이벤트 및 대응 상태 확인


## 🏗️ System Architecture

```mermaid
flowchart TB
    Internet((Internet))
    subgraph Firewall["pfSense Firewall"]
        FW["Network Segmentation<br/>WAN / DMZ / LAN / SIEM"]
    end

    subgraph DMZ["DMZ - Web Service"]
        WEB["Apache / PHP 7.4<br/>Shopping Mall & API"]
        WAF["ModSecurity v2 WAF"]
    end

    subgraph LAN["LAN - Database"]
        DB[("MySQL")]
    end

    subgraph SIEM["SIEM Network"]
        WAZUH["Wazuh SIEM<br/>Log Collection & Rule Detection"]
        SHUFFLE["Shuffle<br/>Workflow Orchestration"]
        BLOCK["Custom block-api"]
        GRAFANA["Grafana<br/>Monitoring Dashboard"]
        OS["OpenSearch"]
    end

    Internet --> FW
    FW --> WEB
    WEB --> WAF
    WAF --> DB

    WEB -. "Security Logs" .-> WAZUH
    WAZUH --> SHUFFLE
    SHUFFLE --> BLOCK
    BLOCK -. "SSH Block Command" .-> FW

    WAZUH --> OS
    OS --> GRAFANA
```

> 네트워크 구역을 분리하고, 웹 서비스에서 발생한 보안 로그를 SIEM 환경으로 전달하여 탐지와 대응이 이어지도록 구성했습니다.



## ⚙️ Technology Stack

| Category | Technology | Role |
|:---|:---|:---|
| Web Server | Apache | 웹 서비스 운영 |
| Backend | PHP 7.4 | 쇼핑몰 및 API 서버 |
| Database | MySQL | 서비스 데이터 관리 |
| Firewall | pfSense | 네트워크 분리 및 IP 차단 |
| IPS | Suricata | 네트워크 침입 탐지 및 방지 |
| WAF | ModSecurity v2 | 웹 요청 필터링 |
| SIEM | Wazuh | 로그 수집 및 룰 기반 탐지 |
| Search Engine | OpenSearch | 이벤트 데이터 저장 및 조회 |
| SOAR | Shuffle | 보안 이벤트 기반 대응 워크플로우 |
| Dashboard | Grafana | 탐지 및 대응 현황 시각화 |
| Custom API | block-api | pfSense 차단 명령 전달 |



## 🔄 Detection & Response Flow

공격 유형과 위험도에 따라 대응 방식을 두 가지로 구분했습니다.

### 1. 대량 요청 공격 — 자동 차단

짧은 시간 동안 비정상적으로 많은 요청이 발생하는 상황을 대상으로 자동 차단 흐름을 구성했습니다.

```mermaid
flowchart TD
    A["비정상적인 대량 요청 발생"] --> B["Wazuh 로그 수집"]
    B --> C["탐지 룰 조건 충족"]
    C --> D["Wazuh Alert 생성"]
    D --> E["Shuffle 이벤트 전달"]
    E --> F["block-api 차단 요청"]
    F --> G["pfSense IP 차단"]
    G --> H["차단 완료 상태 로그 전달"]
    H --> I["Wazuh 수집"]
    I --> J["Grafana 차단 현황 표시"]
```

**대응 기준**

- 프로젝트에서 설정한 기준: 60초 이내 30회 요청
- Wazuh 룰을 통해 조건에 해당하는 이벤트 탐지
- Shuffle이 block-api에 차단 요청 전달
- block-api가 pfSense에 차단 명령 전달
- 차단 완료 상태를 Wazuh로 전달하고 Grafana에서 확인

### 2. Shadow API — 관리자 승인 후 차단

관리되지 않는 API 접근과 같이 정상적인 사용자의 요청과 구분이 필요한 상황은 자동 차단하지 않고 관리자의 판단을 거치도록 구성했습니다.

```mermaid
flowchart TD
    A["관리되지 않는 API 접근"] --> B["Wazuh 로그 수집"]
    B --> C["탐지 룰 조건 충족"]
    C --> D["Wazuh Alert 생성"]
    D --> E["Shuffle 이벤트 전달"]
    E --> F["승인 대기 상태 전달"]
    F --> G["Grafana 승인 대기 표시"]
    G --> H["관리자 승인 페이지"]
    H --> I["차단 승인"]
    I --> J["block-api 차단 요청"]
    J --> K["pfSense IP 차단"]
    K --> L["차단 완료 상태 전달"]
    L --> M["Grafana 차단 완료 표시"]
```

**대응 방식**

- Wazuh가 관련 로그를 룰 기반으로 탐지
- Shuffle이 승인 대기 상태를 전달
- Grafana에서 승인 대기 항목 확인
- 관리자가 별도 승인 페이지에서 차단 여부 결정
- 승인 시 block-api가 pfSense에 차단 명령 전달
- 차단 완료 상태를 Wazuh에 전달하고 Grafana에서 상태 확인



## 💡 Key Implementation

### 1. 자동 차단과 수동 승인 분리

모든 공격을 자동 차단하면 정상적인 사용자의 요청까지 차단할 가능성이 있습니다.

이를 고려하여 대응 방식을 구분했습니다.

| 구분 | 대량 요청 공격 | Shadow API |
|:---|:---|:---|
| 대응 방식 | 자동 차단 | 관리자 승인 |
| 판단 기준 | 설정된 요청 횟수 및 시간 조건 | 관리되지 않는 API 접근 이벤트 |
| 차단 실행 | 자동 요청 | 관리자 승인 후 실행 |
| 목적 | 신속한 대응 | 오탐으로 인한 피해 최소화 |

※ 30회/60초는 프로젝트에서 설정한 대응 기준이며, 모든 비정상 요청이나 자동화 공격을 정확하게 판별하는 기준은 아닙니다.

### 2. Custom block-api 개발

폐쇄망 환경에서 외부 연동 및 알림 방식에 제약이 있어, 차단 명령을 전달하는 별도의 block-api를 구현했습니다.

- Shuffle로부터 차단 요청 수신
- pfSense에 SSH 기반 차단 명령 전달
- 차단 요청과 실제 방화벽 동작 연결

### 3. Grafana 기반 대응 현황 시각화

Wazuh에서 처리되는 보안 이벤트와 대응 상태를 Grafana 대시보드에서 확인할 수 있도록 구성했습니다.

- 탐지 이벤트 확인
- 승인 대기 항목 확인
- 차단 완료 항목 확인
- 대응 상태 변화 모니터링

## 🎬 Project Demo
<video src="https://github.com/fzrunx/Security-Project-Portfolio/issues/1#issue-6367348078" controls width="100%"></video>


## 🔧 Project Considerations & Limitations

### 폐쇄망 환경에 따른 대응 구조 변경

초기에는 Shuffle을 활용해 공격 알림을 내부 메신저로 전달하고, 관리자가 메신저에서 차단 여부를 선택하는 형태를 구상했습니다.

하지만 폐쇄망 환경에서 외부 서비스 및 메신저 연동에 제약이 발생했습니다.

이에 따라 별도의 block-api와 관리자 승인 페이지를 개발하여 대응 구조를 변경했습니다.

### 개선 방향

- 요청 횟수뿐 아니라 사용자 및 IP별 정상 트래픽 패턴을 고려한 탐지 기준 개선
- 정상적인 서비스 요청이 차단되는 오탐 상황을 줄이기 위한 예외 처리
- 차단 시간 설정 및 차단 해제 기능 보완
- 공격 유형별 탐지 룰과 대응 정책 세분화



## 👥 Project Outcome

이번 프로젝트를 통해 웹 서비스에서 발생하는 로그를 수집하고, 탐지 룰을 기반으로 보안 이벤트를 생성한 뒤 대응 시스템과 연동하는 과정을 경험했습니다.

특히 단순히 공격을 탐지하는 데서 끝나는 것이 아니라, 공격 유형에 따라 자동 차단과 관리자 승인 방식을 구분하고 방화벽과 연동하는 대응 구조를 구현했습니다.

**주요 경험**
- Wazuh 룰 기반 로그 탐지 및 이벤트 처리
- Shuffle을 활용한 보안 대응 워크플로우 구성
- pfSense 방화벽 연동 및 IP 차단
- Custom block-api 및 관리자 승인 페이지 구현
- Grafana 기반 보안 이벤트 모니터링

---

**Information Security Project | Team Project**
