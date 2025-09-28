# Концептуальная схема деплоя через GitHub Actions на локальную ВМ (self-hosted runner)

## 1. Действия в GitHub Actions

Пример Workflow для деплоя проекта на ветку `dev`:

```yaml
name: Deploy Dev Stand

on:
  push:
    branches: ["dev"]

jobs:
  build-and-deploy:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v3

      # Кэш pnpm
      - name: Cache pnpm store
        uses: actions/cache@v3
        with:
          path: ~/.pnpm-store
          key: pnpm-store-${{ runner.os }}-${{ hashFiles('pnpm-lock.yaml') }}
          restore-keys: |
            pnpm-store-${{ runner.os }}-

      # Установка зависимостей
      - name: Install deps
        run: pnpm install --frozen-lockfile

      # Сборка проекта
      - name: Build
        run: pnpm build

      # Деплой на Nginx
      - name: Deploy to Nginx
        run: cp -r dist/* /var/www/html/

      # Лог успешного деплоя
      - name: Log deploy
        run: echo "✅ Deploy finished at $(date)"

2. Настройка ВМ (Ubuntu) для self-hosted runner
2.1 Установка Ubuntu
Установить сервер Ubuntu.

Во время установки отметить чекбокс Дополнения гостевой ОС (VirtualBox Guest Additions).

Настройка сети:

Адаптер 1 → Сетевой мост

В VirtualBox → Общие → Дополнительно:

Shared Clipboard → Bidirectional

Drag’n’Drop → Bidirectional

2.2 Установка графического интерфейса и буфера обмена
sudo apt update
sudo apt install xfce4 -y
startx  # запуск графической оболочки
2.3 Установка Guest Additions
В меню VirtualBox: Devices → Insert Guest Additions CD Image

В Ubuntu выполнить:

sudo apt update
sudo apt install build-essential dkms linux-headers-$(uname -r) -y
sudo mkdir -p /media/VBoxGuestAdditions
sudo mount /dev/cdrom /media/VBoxGuestAdditions
sudo /media/VBoxGuestAdditions/VBoxLinuxAdditions.run
sudo reboot

3. Настройка self-hosted runner
3.1 Создание и скачивание runner
На GitHub: Settings → Actions → Runners → New self-hosted runner → Linux → x64

На ВМ выполнить:

mkdir ~/actions-runner && cd ~/actions-runner
curl -o actions-runner-linux-x64-<version>.tar.gz -L https://github.com/actions/runner/releases/download/v<version>/actions-runner-linux-x64-<version>.tar.gz
tar xzf ./actions-runner-linux-x64-<version>.tar.gz
3.2 Настройка runner

./config.sh --url https://github.com/<user>/<repo> --token <TOKEN>
# Runner group → Enter
# Имя runner → Enter (по умолчанию имя ВМ)
# Labels → Enter
# Запуск как сервис → yes
3.3 Запуск runner как сервиса
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh status
4. Установка зависимостей на сервере
Node.js + pnpm
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pnpm
pnpm -v  # проверка установки
Git

sudo apt install git -y
Nginx

sudo apt install nginx -y
Права на директорию для деплоя

sudo chown -R <user>:<user> /var/www/html
sudo chmod -R 755 /var/www/html
Разрешение перезагрузки Nginx без пароля

sudo visudo
# Добавить строку:
<user> ALL=(ALL) NOPASSWD: /bin/systemctl reload nginx
5. Проверка IP ВМ
ip a
# Найти интерфейс, например enp0s3:
# inet 192.168.1.45/24 brd 192.168.1.255 scope global dynamic enp0s3
# В примере IP = 192.168.1.45
После настройки GitHub Actions, self-hosted runner и сервера, любые пуши в ветку dev будут автоматически:

Билдить проект

Деплоить его на Nginx

Логировать успешный деплой

# React + TypeScript + Vite
- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Babel](https://babeljs.io/) for Fast Refresh
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/) for Fast Refresh

## React Compiler
The React Compiler is enabled on this template. See [this documentation](https://react.dev/learn/react-compiler) for more information.
Note: This will impact Vite dev & build performances.
## Expanding the ESLint configuration
If you are developing a production application, we recommend updating the configuration to enable type-aware lint rules:
```js
export default defineConfig([
  globalIgnores(['dist']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [
      // Other configs...

      // Remove tseslint.configs.recommended and replace with this
      tseslint.configs.recommendedTypeChecked,
      // Alternatively, use this for stricter rules
      tseslint.configs.strictTypeChecked,
      // Optionally, add this for stylistic rules
      tseslint.configs.stylisticTypeChecked,

      // Other configs...
    ],
    languageOptions: {
      parserOptions: {
        project: ['./tsconfig.node.json', './tsconfig.app.json'],
        tsconfigRootDir: import.meta.dirname,
      },
      // other options...
    },
  },
])
```

You can also install [eslint-plugin-react-x](https://github.com/Rel1cx/eslint-react/tree/main/packages/plugins/eslint-plugin-react-x) and [eslint-plugin-react-dom](https://github.com/Rel1cx/eslint-react/tree/main/packages/plugins/eslint-plugin-react-dom) for React-specific lint rules:

```js
// eslint.config.js
import reactX from 'eslint-plugin-react-x'
import reactDom from 'eslint-plugin-react-dom'

export default defineConfig([
  globalIgnores(['dist']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [
      // Other configs...
      // Enable lint rules for React
      reactX.configs['recommended-typescript'],
      // Enable lint rules for React DOM
      reactDom.configs.recommended,
    ],
    languageOptions: {
      parserOptions: {
        project: ['./tsconfig.node.json', './tsconfig.app.json'],
        tsconfigRootDir: import.meta.dirname,
      },
      // other options...
    },
  },
])
```
