# DV - Мини-проект «DevOps-конвейер»

## 0) Мета

- **Проект:** [Заметки](https://github.com/alexanderanisin/security-app-hse)
- **Версия (commit/date):** v.1 / 2025-10-21
- **Кратко (1-2 предложения):** запускаем FastAPI приложение с SQLite и JWT-авторизацией для управления заметками

---

## 1) Воспроизводимость локальной сборки и тестов (DV1)

- **Одна команда для сборки/тестов:**

1. Выдайте разрешение на запуск файла, например, для Linux:

    ```bash
    chmod +x oneliner.sh
    ```

2. Запустите приложение

Запуск на Linux/macOS

```bash
./oneliners/oneliner.sh
```

Запуск на Windows (через Powershell)

```bash
bash ./oneliners/winliner.sh
```

Запуск с помощью Docker Compose:

```bash
docker compose up --build
```

- **Версии инструментов (фиксация):**

  ```bash
  python --version # 3.14
  pip freeze > EVIDENCE/pip-freeze.txt
  ```

- **Описание шагов (кратко):**

1. curl -LsSf https://astral.sh/uv/install.sh | sh
2. uv sync --all-extras
3. source .venv/bin/activate
4. pytest -q
5. uvicorn src.main:app --reload

---

## 2) Контейнеризация (DV2)

- **Dockerfile:** ./Dockerfile
- **Сборка/запуск локально:**

  ```bash
  docker build -t app:local .
  docker run --rm -p 8080:8080 app:local
  ```

  Healthcheck есть: `/status/check_availability`

- **docker-compose:** `./docker-compose.yml` - 1 сервис, back-end приложения.

---

## 3) CI: базовый pipeline и стабильный прогон (DV3)

- **Платформа CI:** GitHub Actions
- **Файл конфига CI:** `.github/workflows/ci.yml`
- [ ] Стадии (минимум):** checkout → deps → **build** → **test** → (package)
- **Фрагмент конфигурации (ключевые шаги):**

  ```yaml
    name: CI Pipeline

    on:
      push:
        branches: [ "main" ]
      pull_request:
        branches: [ "main" ]

    jobs:
      build-and-test:
        runs-on: ubuntu-latest
        strategy:
          matrix:
            python-version: ["3.13", "3.14"]

        steps:
          - name: Checkout repository
            uses: actions/checkout@v5

          - name: Install uv and set Python ${{ matrix.python-version }}
            uses: astral-sh/setup-uv@v6
            with:
              python-version: ${{ matrix.python-version }}
              enable-cache: true

          - name: Install the project
            run: uv sync --locked --all-extras --dev

          - name: Run tests
            env:
              DB_NAME: ${{ secrets.DB_NAME }}
              JWT_SECRET_KEY: ${{ secrets.JWT_SECRET_KEY }}
            run: uv run pytest

          - name: Build Docker image
            run: docker build . --tag my-app:${{ github.sha }}
  ```

- **Стабильность:** Да
- **Ссылка/копия лога прогона:** `EVIDENCE/ci-2025-10-21-build-python-3-13.txt` и `EVIDENCE/ci-2025-10-21-build-python-3-14.txt`

---

## 4) Артефакты и логи конвейера (DV4)

| Артефакт/лог                    | Путь в `EVIDENCE/`                                                                       | Комментарий                       |
| ------------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------- |
| Лог успешной сборки/тестов (CI) | `ci-2025-10-21-build-python-3-13.txt`   и `EVIDENCE/ci-2025-10-21-build-python-3-14.txt` | ключевые шаги/время               |
| Локальный лог сборки (опц.)     | `build.log`                                                             | для сверки                        |
| Freeze/версии инструментов      | `pip-freeze.txt`                                                           | воспроизводимость окружения       |

---

## 5) Секреты и переменные окружения (DV5 - гигиена, без сканеров)

- **Шаблон окружения:** добавлен файл `/.env.example` со списком переменных
- **Хранение и передача в CI:**  
  - секреты лежат в настройках репозитория/организации (**masked**),  
  - в pipeline они **не печатаются** в явном виде.
- **Памятка по ротации:** Секреты хранятся в GitHub Secrets. При утечке или компрометации, новый секрет должен быть сгенерирован и обновлен в настройках репозитория.

---

## 6) Индекс артефактов DV

| Тип     | Файл в `EVIDENCE/`                                                | Дата/время         | Коммит/версия | Runner/OS       |
| ------- | ----------------------------------------------------------------- | ------------------ | ------------- | --------------- |
| CI-лог  | `ci-YYYY-MM-DD-build.txt`                                         | `2025-10-21 23:38` | `8232ac2`     | `ubuntu-latest` |
| Лок.лог | `local-build-YYYY-MM-DD.txt`                                      | `2025-10-21 23:38` | `8232ac2`     | `local`         |
| Freeze  | `pip-freeze.txt`                                                  | `2025-10-21 23:38` | `8232ac2`     | -               |
| Секреты | `secret.txt` или CI-лог сборки (секреты не утекают в логи сборки) | `2025-10-21 23:38` | `8232ac2`     | -               |

---

## 7) Самооценка по рубрике DV (0/1/2)

- **DV1. Воспроизводимость локальной сборки и тестов:** [ ] 0 [ ] 1 [x] 2  
- **DV2. Контейнеризация (Docker/Compose):** [ ] 0 [ ] 1 [x] 2  
- **DV3. CI: базовый pipeline и стабильный прогон:** [ ] 0 [ ] 1 [x] 2  
- **DV4. Артефакты и логи конвейера:** [ ] 0 [ ] 1 [x] 2  
- **DV5. Секреты и конфигурация окружения (гигиена):** [ ] 0 [ ] 1 [x] 2  

**Итог DV (сумма):** 10/10
