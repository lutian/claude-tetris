---
description: Cria um worktree git em .trees/<nome> e executa ali as instruções, isolado do código principal
argument-hint: <descrição da tarefa a executar no worktree>
allowed-tools: Bash(git worktree:*), Bash(git status:*), Bash(git branch:*), Bash(git add:*), Bash(git commit:*), Bash(git diff:*), Bash(git log:*), Read, Write, Edit, Glob, Grep
---

O requerimento do usuário é: $ARGUMENTS

Execute os passos abaixo, em ordem, sem pular nenhum.

## 1. Derivar o nome do worktree

A partir do requerimento acima, escolha um nome curto que resuma a tarefa:
- No máximo 3 palavras
- kebab-case, minúsculas, sem acentos nem caracteres especiais (ex.: "adicionar sistema de pontuação por combo" → `combo-score`)

Se o requerimento estiver vazio ou vago demais para nomear, pare e peça ao usuário uma descrição da tarefa — não continue sem isso.

## 2. Evitar colisão

Rode `git worktree list` e `git branch --list` para verificar se `.trees/<nome>` ou o branch `<nome>` já existem. Se existirem, use um sufixo `-2`, `-3`, etc. até achar um nome livre.

## 3. Criar o worktree

```
git worktree add .trees/<nome>
```

Isso cria o worktree a partir do HEAD atual e gera automaticamente o branch `<nome>`.

## 4. Trabalhar de forma isolada

A partir daqui, **todo o trabalho acontece dentro de `.trees/<nome>/`**:

- Todas as leituras/edições de arquivo usam caminhos sob `.trees/<nome>/...`.
- **Não edite nenhum arquivo fora de `.trees/<nome>/`** — isso inclui não tocar em `game.js`, `index.html`, `style.css` ou `README.md` na raiz do repositório principal.
- Para comandos git, sempre use `git -C .trees/<nome> ...` (nunca `cd` para lá, para não perder o contexto do diretório principal).

## 5. Executar o requerimento

Implemente o requerimento do usuário inteiramente dentro do worktree, seguindo as convenções descritas no `CLAUDE.md` do projeto (arquitetura de `game.js`, constantes tunáveis, etc.). Se a mudança afetar comportamento/funcionalidades documentadas no `README.md`, atualize o `README.md` **dentro do worktree** (ele é em espanhol — mantenha o idioma).

## 6. Verificar

Este projeto não tem build/lint/test automatizado. Para verificar manualmente, sirva o worktree:

```
python3 -m http.server 8000 --directory .trees/<nome>
```

e descreva o que deveria ser observado no navegador para validar a mudança.

## 7. Reportar

Ao final, resuma para o usuário:
- Nome do worktree e branch criado
- Caminho (`.trees/<nome>`)
- Arquivos alterados (`git -C .trees/<nome> status --short`)
- Como remover o worktree quando não for mais necessário: `git worktree remove .trees/<nome>` (e `git branch -D <nome>` se o branch também não for mais necessário)

Não faça commit nem push automaticamente — só se o usuário pedir explicitamente.
