# Run Coach Daily

Antes de alterar o app, leia `HANDOFF_CODEX.md` para conhecer a arquitetura, o estado atual e as pendências.

## Regras permanentes

1. O código do app permanece no único arquivo `index.html`.
2. Não use dois hifens seguidos em textos, código ou comentários.
3. Não exponha chaves, tokens ou outros segredos no código.
4. Preserve a compatibilidade com os dados existentes no localStorage.
5. Não adicione dependências, CDN, frameworks ou etapa de build.
6. Não apague dados do localStorage sem confirmação explícita.

## Entrega

Para qualquer mudança no `index.html`, atualize APP_VERSION, CHANGELOG e o cache em `sw.js`.
Valide a alteração, faça o commit e envie para a branch `main`.

Atualize `HANDOFF_CODEX.md` quando uma entrega relevante alterar o estado do projeto ou antes de migrar para uma nova conversa.
