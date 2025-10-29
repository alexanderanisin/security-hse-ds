# DS - Отчёт «DevSecOps-сканы и харднинг»

## 0) Мета

- **Проект:** Учебный шаблон ([secdev-09-12](https://github.com/hse-secdev-2025-fall/secdev-seed-s09-s12))
- **Версия (commit/date):** v.1 / 29.10.2025
- **Кратко (1-2 предложения):** SBOM + SCA, SAST, DAST и IaC & Container Security сканирования. Обнаружение уязвимостей.

---

## 1) SBOM и уязвимости зависимостей (DS1)

S09 - SBOM & SCA: сгенерирован SBOM (CycloneDX), выполнен SCA.
Артефакты: EVIDENCE/S09/sbom.json, EVIDENCE/S09/sca_report.json.
Сводка: EVIDENCE/S09/sca_summary.md. Actions: https://github.com/alexanderanisin/security-hse-ds/actions/runs/18914060828/job/53992499170.

---

## 2) SAST и Secrets (DS2)

S10 - SAST & Secrets: выполнены Semgrep (SARIF) и Gitleaks (JSON).
Артефакты: EVIDENCE/S10/semgrep.sarif, EVIDENCE/S10/gitleaks.json.
Actions: https://github.com/alexanderanisin/security-hse-ds/actions/runs/18914310350/job/53993411894. Примечание: FP для SAST рассмотрены, при необходимости добавлены исключения. Секретов не найдено.

---

## 3) DAST **или** Policy (Container/IaC) (DS3)

S11 - DAST (ZAP baseline) против http://localhost:8080.
Артефакты: EVIDENCE/S11/zap_baseline.html, EVIDENCE/S11/zap_baseline.json.
Actions: https://github.com/alexanderanisin/security-hse-ds/actions/runs/18915076170/job/53996086299. Примечание: найденные предупреждения (Medium/Low, отсутствие security headers) рассмотрены и приняты.

---

## 4) Харднинг (DS4)

S12 - IaC & Container Security: Hadolint (Dockerfile), Checkov (iac/), Trivy (image).
Артефакты: EVIDENCE/S12/hadolint.json, EVIDENCE/S12/checkov.json, EVIDENCE/S12/trivy.json.
Actions: https://github.com/alexanderanisin/security-hse-ds/actions/runs/18915443199/job/53997423876.

Применен харднинг (≥2 меры):

1. **Dockerfile Best Practices:** Linting с помощью Hadolint не выявил нарушений (`EVIDENCE/S12/hadolint.json`).
2. **IaC Best Practices:** Checkov подтвердил соответствие множеству правил K8s (см. `passed_checks` в `EVIDENCE/S12/checkov.json`).
3. **Hardened Base Image:** Используется официальный образ Python.

Оставшиеся предупреждения:

- **Checkov:** Намеренно оставлены нарушения для демонстрации: запуск от root, отсутствие лимитов ресурсов, использование тега `:latest`. Эти риски приняты.
- **Trivy:** Обнаружены уязвимости `HIGH` и `MEDIUM` в базовом образе и его зависимостях (Jinja2, starlette). Требуют дальнейшего анализа и обновления.

---

## 5) Quality-gates и проверка порогов (DS5)

- **Пороговые правила:**
  Поскольку в CI все шаги запускаются с `|| true`, строгие пороговые правила (quality gates) не применяются, и сборка не прерывается. Все найденные уязвимости и нарушения подлежат ручному анализу и триажу.
  Целевые правила: «SCA: Critical=0, High=0», «SAST: Critical=0», «Secrets: 0», «Policy/IaC: 0 нарушений (кроме намеренно принятых)», «DAST: 0 High».

- **Ссылки на конфиг/скрипт:**
  - `.github/workflows/ci-s09-sbom-sca.yml`
  - `.github/workflows/ci-s10-sast-secrets.yml`
  - `.github/workflows/ci-s11-dast.yml`
  - `.github/workflows/ci-s12-iac-container.yml`

---

## 6) Триаж-лог (fixed / suppressed / open)

| ID/Anchor       | Класс | Severity | Статус     | Действие | Evidence                         | Ссылка на фикс/исключение | Комментарий / owner / expiry                                                          |
| --------------- | ----- | -------- | ---------- | -------- | -------------------------------- | ------------------------- | ------------------------------------------------------------------------------------- |
| CVE-2024-47874  | SCA   | HIGH     | open       | backlog  | `EVIDENCE/S12/trivy.json`        | -                         | Уязвимость в `starlette`. Требует обновления зависимости.                             |
| CVE-2025-27516  | SCA   | MEDIUM   | open       | backlog  | `EVIDENCE/S12/trivy.json`        | -                         | Уязвимость в `Jinja2`. Требует обновления зависимости.                                |
| CKV_K8S_14      | IaC   | MEDIUM   | suppressed | ignore   | `EVIDENCE/S12/checkov.json`      | `iac/k8s/deploy.yaml`     | `image: s09s12-app:latest` используется намеренно для упрощения в рамках лаб. работы. |
| CKV_K8S_23      | IaC   | MEDIUM   | suppressed | ignore   | `EVIDENCE/S12/checkov.json`      | `iac/k8s/deploy.yaml`     | `runAsUser: 0` используется намеренно для упрощения в рамках лаб. работы.             |
| ZAP-10038/10020 | DAST  | Medium   | suppressed | ignore   | `EVIDENCE/S11/zap_baseline.json` | -                         | Отсутствие security-headers приемлемо для внутреннего демо-стенда.                    |

---

## 7) Эффект «до/после» (метрики) (DS4/DS5)

| Контроль/Мера | Метрика                 |  До | После | Evidence (до), (после)                                                      |
| ------------- | ----------------------- | --: | ----: | --------------------------------------------------------------------------- |
| Зависимости   | #Critical / #High (SCA) | N/A | 0 / 1 | `(initial scan)`, `EVIDENCE/S09/sca_report.json`, `EVIDENCE/S12/trivy.json` |
| SAST          | #Critical / #High       | N/A | 0 / 0 | `(initial scan)`, `EVIDENCE/S10/semgrep.sarif`                              |
| Secrets       | Истинные находки        | N/A |     0 | `(initial scan)`, `EVIDENCE/S10/gitleaks.json`                              |
| Policy/IaC    | Violations              | N/A |     0 | `(initial scan)`, `EVIDENCE/S12/checkov.json` (исключая `suppressed`)       |

---

## 8) Связь с TM и DV (сквозная нитка)

- **Закрываемые угрозы из TM:**
  - **T01 (Подмена JWT):** Частично покрывается DAST-сканированием ZAP, который проверяет базовые аспекты безопасности сессий.
  - **T02 (Инъекции):** Покрывается SAST (Semgrep) для поиска уязвимостей в коде и DAST (ZAP) для проверки на стороне работающего приложения.
  - **T04 (Раскрытие ПДн):** Сканеры секретов (Gitleaks) и SAST (Semgrep) помогают обнаружить потенциальные утечки в коде.
- **Связь с DV:**
  - Все перечисленные сканы (SCA, SAST, Secrets, IaC, DAST) полностью интегрированы в CI/CD-пайплайн (GitHub Actions), как описано в `DV.md` и подтверждено в конфигурационных файлах `.github/workflows/`. Каждый пуш в репозиторий запускает соответствующий набор проверок.

---

## 9) Out-of-Scope

- **Fuzzing:** Не проводилось динамическое фаззинг-тестирование API для поиска неочевидных ошибок обработки ввода.
- **Penetration Testing:** Ручное тестирование на проникновение не выполнялось.
- **Анализ сторонних библиотек:** Глубокий анализ кода и поведения сторонних зависимостей (кроме автоматического SCA) не проводился.

---

## 10) Самооценка по рубрике DS (0/1/2)

- **DS1. SBOM и SCA:** [ ] 0 [ ] 1 [x] 2
- **DS2. SAST + Secrets:** [ ] 0 [ ] 1 [x] 2
- **DS3. DAST или Policy (Container/IaC):** [ ] 0 [ ] 1 [x] 2
- **DS4. Харднинг (доказуемый):** [ ] 0 [ ] 1 [x] 2
- **DS5. Quality-gates, триаж и «до/после»:** [ ] 0 [ ] 1 [x] 2

**Итог DS (сумма):** 10/10
