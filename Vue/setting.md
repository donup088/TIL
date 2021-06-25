### Vue 프로젝트 초기 세팅
- eslint, prettier 적용
    - overlay 되는 부분 해제
        - vue.config.js 파일에 다음 코드 추가
        ```
        module.exports = {
            devServer: {
                overlay: false,
            },
        };
        ```
    - settings.js 에 다음 코드 추가
    ```
     "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    "eslint.workingDirectories": [
        {
            "mode": "auto"
        }
    ],
    "eslint.validate": [
        {
            "language": "vue",
            "autoFix": true
        },
        {
            "language": "javascript",
            "autoFix": true
        },
        {
            "language": "javascriptreact",
            "autoFix": true
        },
        {
            "language": "typescript",
            "autoFix": true
        },
        {
            "language": "typescriptreact",
            "autoFix": true
        }
    ]
    ```
    - .eslintrc.js 파일에 다음 코드 추가
    ```
      rules: {
        "no-console": "off",
        // "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
        // "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off",
        "prettier/prettier" : ['error', {
        singleQuote: true,
        semi: true,
        useTabs: true,
        tabWidth: 2,
        trailingComma: 'all',
        printWidth: 80,
        bracketSpacing: true,
        arrowParens: 'avoid',
        endOfLine: 'auto',
        }]
    },
    ```
    - format on save prettier 설정 체크 해제(ctrl , 누르고 검색후 해제)