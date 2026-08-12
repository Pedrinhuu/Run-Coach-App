# Run Coach Daily, handoff para continuidade no Codex

Leia este documento ao iniciar uma nova conversa. Ele descreve o projeto, as regras
invioláveis, a arquitetura, as armadilhas já descobertas e o fluxo de trabalho esperado.

---

## 1. Projeto

- **App:** Run Coach Daily, PWA de acompanhamento de treinos de corrida, em português.
- **Produção:** https://runcoachdaily.pages.dev
- **Repositório:** https://github.com/Pedrinhuu/Run-Coach-App
- **Stack:** HTML + CSS + JavaScript puro, tudo em `index.html` (~5.700 linhas).
- **Deploy:** push na branch `main` publica no Cloudflare Pages em cerca de 30 segundos.
- **Versão atual:** 4.7.2 (`APP_VERSION` no index.html, cache `runcoach-v472` no sw.js).
- **Desenvolvimento:** Codex. Alterações concluídas devem receber commit e push na branch `main`.
- **Usuários reais:** Pedro (dono) e a esposa dele, Jéssica, em instalação separada.
  Vários bugs vieram justamente da instalação dela. Sempre considere o usuário novo.

### Arquivos

```
index.html      # o app inteiro (HTML + CSS + JS)
sw.js           # Service Worker, cache versionado
manifest.json   # PWA
version.json    # versão publicada, atualizada por GitHub Action (não editar à mão)
_headers        # headers de segurança do Cloudflare Pages
README.md       # documentação do projeto
```

---

## 2. Restrições obrigatórias (nunca violar)

1. O app permanece **single-file**: todo o código vai em `index.html`.
2. **Nunca usar travessão duplo** (dois hifens seguidos) em nenhum texto gerado,
   nem em código, comentário, commit ou texto de interface.
3. **Nunca colocar chave de API ou token no código.** Credenciais vivem apenas no
   localStorage do aparelho, digitadas pelo usuário em campo tipo password.
4. **Retrocompatibilidade total** com o localStorage existente. Mudanças só podem
   alterar o comportamento de instalação limpa, nunca dados já salvos.
5. **Sem dependências novas.** Nada de biblioteca, CDN, framework ou build step.
6. **Nunca apagar dados do localStorage sem confirmação explícita do usuário.**

---

## 3. Arquitetura, mapa de funções

Busque pelo nome da função, não por número de linha (o arquivo muda toda hora).

### Armazenamento e helpers base
- `KEYS`, `load()`, `save()`: acesso ao localStorage. `save()` também agenda o
  backup no GitHub via `scheduleGistSync()`.
- `getWeek()` / `setWeek()`: semana atual. `getWeek` normaliza treinos malformados
  com `normalizarTreino()` sem tocar no que está salvo.
- `getLogs()`, `getHistory()`, `getRaces()`, `getPRsManuais()`, `getProtocoloJoelho()`,
  `getAvaliacao()`, `getSettings()`.
- `parseDataSegura(valor)`: normaliza datas para AAAA-MM-DD. Aceita DD/MM/AAAA e
  AAAA/MM/DD, devolve `null` para o que não der para interpretar. **Use sempre isto
  antes de qualquer `new Date(...)` sobre dado vindo de JSON.**
- `mondayISOOf(data)`, `intervaloSemanaAtual()`, `dataDoTreino(diaNome)`,
  `todayISO()`: cálculo de semana e datas, tudo em UTC.

### Telas
- `render()`: monta a aba atual. Tem try/catch: erro de render mostra um card de
  recuperação em vez de travar a navegação.
- `renderHome()`, `renderSemana()`, `renderHistorico()` (com `renderTreinosSubTab` e
  `renderCorridasSubTab`), `renderMetricas()`, `renderAvaliacao()`.
- `attachHandlers()`: religa todos os eventos após cada render.

### Regras de negócio centrais
- `proximaAtividadeSemana(w)`: fila da próxima atividade pendente (corridas com bloco
  de treino mais o protocolo de joelho), ordenada por dia da semana.
- `calcAderencia(w)`: conta exatamente as atividades visíveis na tela Semana.
- `calcularPRs(corridas)`: recordes por distância, considerando resultados reais e
  PRs manuais. Sempre derivado, nunca persistido.
- `validarSemanaImport(obj)`: validador único do JSON de importação.
- `previewSemana(back)`: preview e confirmação do import.
- `encontrarCorridaDaProva()`, `registrarResultadoNaCorrida()`,
  `sincronizarTreinoProvaComCorrida()`: vínculo entre treino tipo Prova e corrida
  cadastrada.
- `celebrarNovoPR(msg)`: animação de novo recorde, em CSS puro.

### Componentes
- `openPicker({ title, columns, separators, onConfirm })`: roleta customizada.
  Colunas com `wrap: true` rolam de forma circular.
- `openPacePicker`, `openTempoPicker`, `openTempoRaceResultPicker`, `openDistPicker`,
  `openDatePicker`, `pickerInputHtml`, `pickerInputSet`.
- `openLogModal(idx, existingLog)`: registro de treino.
- `openImportarSemanaModal()`, `openSettingsModal()`, `openSchemaModal()`.

### Backup e integrações
- `buildBackupObject()` e `restoreFromBackupObject()`: fonte única de serialização,
  usada tanto pelo export manual quanto pelo backup no GitHub.
- `backupKeys()` e `BACKUP_EXCLUDE_KEYS`: o que entra e o que nunca entra no backup.
- `scheduleGistSync()`, `doGistSync()`, `githubCreateGist()`, `githubUpdateGist()`,
  `githubFetchGist()`: backup automático em Gist privado.
- `enviarParaIntervals()`, `buildIntervalsDescription()`: envio do treino ao relógio.
- `guiaCoachTexto()` e `templateSemanaImport()`: guia em branco para o coach, fonte
  única usada tanto em Ajustes quanto no modal do estado vazio.

---

## 4. Chaves do localStorage

| Chave | Conteúdo |
|---|---|
| `runcoach_semana_atual` | Semana em andamento |
| `runcoach_historico_semanas` | Semanas anteriores |
| `runcoach_logs` | Treinos registrados |
| `runcoach_corridas` | Provas e seus resultados reais |
| `runcoach_prs_manuais` | Recordes informados manualmente |
| `runcoach_protocolo_joelho` | Protocolo de fortalecimento (conteúdo) |
| `runcoach_protocolo_status` | Conclusões do protocolo |
| `runcoach_protocolo_posicao` | Posição no carrossel do protocolo |
| `runcoach_avaliacao` | Teste físico e zonas |
| `runcoach_settings` | Nome, tema, áudio |
| `runcoach_treino_ativo` | Treino em andamento, para retomar |
| `runcoach_versao` | Versão vista pelo usuário |
| `runcoach_intervals_key` | Chave do Intervals.icu |
| `runcoach_github_token` | Token do GitHub |
| `runcoach_github_gist_id` | Id do gist de backup |
| `runcoach_github_sync_meta` | Status da última sincronização |

**Nunca entram no backup:** `runcoach_intervals_key`, `runcoach_github_token`,
`runcoach_github_gist_id`, `runcoach_github_sync_meta`.

---

## 5. Formato do JSON de importação

O plano semanal é gerado por um coach de IA externo e colado no app. Campos aceitos:

```json
{
  "semana": 1,
  "fase": "Base",
  "foco_da_semana": "",
  "alerta_coach": null,
  "observacao_coach": "",
  "prs_manuais": [{ "distancia_km": 5, "tempo": "30:00" }],
  "estimativas_provas": [{
    "nome": "", "data": "AAAA-MM-DD", "distancia": "",
    "tempo_estimado": "", "pace_estimado": "", "nova": true
  }],
  "treinos": [{
    "dia": "Segunda", "tipo": "Corrida Leve", "objetivo": "",
    "rua": {
      "aquecimento": "", "zona_aquecimento": "Z1",
      "principal": "", "zona_principal": "Z2",
      "desaquecimento": "", "zona_desaquecimento": "Z1",
      "distancia_km": 0, "duracao_total": "", "pace_alvo_central_segundos": 0,
      "intervalos": { "ciclos": 7, "fases": [{ "nome": "", "duracao_segundos": 120, "referencia": "" }] }
    },
    "esteira": { }
  }],
  "protocolo_joelho": null
}
```

Regras que o parser exige:
- Datas sempre AAAA-MM-DD.
- `protocolo_joelho` apenas no campo de nível superior, nunca repetido como treino
  na lista. `null` quando não houver.
- Treino tipo `"Prova"` deve cair no mesmo dia e com a mesma distância da prova
  correspondente em `estimativas_provas`, para o app vincular os dois.
- `pace_alvo_central_segundos` é pace em segundos por km.

A fonte de verdade viva desse formato é `guiaCoachTexto()` dentro do app, exibido em
Ajustes. Se mudar o schema, **atualize o guia junto**, senão ele mente para o usuário.

---

## 6. Armadilhas já descobertas (leia antes de mexer)

Estes bugs já aconteceram em produção. Não reintroduza.

### Fuso horário
`new Date('AAAA-MM-DD')` cria meia-noite **UTC**. Misturar isso com métodos locais
(`getDay`, `setDate`, `toISOString`) gera erro de mais ou menos um dia no Brasil.
Todo cálculo de semana e de dia usa UTC (`getUTCDay`, `setUTCDate`). Já causou:
aderência contando o protocolo da semana anterior, e data errada no envio ao relógio.

### Invalid time value
`toISOString()` sobre uma data inválida lança `RangeError` e derruba o fluxo inteiro.
Já quebrou a importação de semana por causa de um treino sem `dia` válido.
Sempre valide antes de converter, e use `parseDataSegura`.

### Dados de schema antigo derrubam a navegação
Um treino `null` no array fazia `renderSemana` lançar, o `render()` não atribuía o
HTML e a aba simplesmente não abria, sem erro visível. Por isso existem hoje:
`normalizarTreino()` em `getWeek()` e o try/catch em `render()`. Mantenha os dois.

### Defaults com dados pessoais
Já houve três casos de valor padrão hardcoded vazando dados do dono do app para a
instalação de outra pessoa (corridas, protocolo de joelho, perfil no prompt).
**Regra:** dado que só faz sentido preenchido pelo usuário começa vazio ou `null` em
instalação limpa. Nada de semear exemplo com dado real de alguém.

### Duas fontes de verdade apodrecem
Já aconteceu com o changelog (popup mostrava versão diferente do histórico) e com o
formato do JSON (README documentava schema que o parser rejeitava). Sempre que dois
lugares mostrarem a mesma informação, derive os dois da **mesma função**.

### PR e contagens são derivados
Recorde pessoal nunca é persistido como campo fixo: é sempre calculado por
`calcularPRs()`. A checagem de novo recorde é feita **antes** de gravar o resultado,
com comparação estritamente menor, o que garante idempotência.

### requestAnimationFrame não roda em aba oculta
O posicionamento inicial do picker dependia só de rAF e abria no topo quando a aba
não estava visível. Hoje posiciona de imediato e reforça no frame seguinte.

### Manifesto do PWA fica preso em cache
O ícone da Home mudou na versão 4.7.1, mas o instalador do Android ainda mostrava a
marca anterior. A página já era nova; o `manifest.json` antigo vinha do cache do
Service Worker e do cache HTTP. Desde a 4.7.2, o manifesto usa URL versionada no
`index.html`, não entra no pré-cache e é buscado com `no-store`. O header exige
revalidação. Ao mudar ícones ou metadados de instalação, atualize também a versão da
URL do manifesto.

---

## 7. Fluxo de trabalho esperado

Para **qualquer** mudança em `index.html`:

1. **Investigue antes de editar.** Leia a função real, não presuma. Vários prompts
   traziam hipóteses de causa que estavam erradas; a causa verdadeira estava em outro
   lugar. Reproduza o bug antes de corrigir.
2. **Suba a versão:** `APP_VERSION` no index.html, entrada nova no topo do `CHANGELOG`
   e nome novo do cache em `sw.js`. Sem isso o Service Worker serve a versão velha.
3. **Valide de verdade no navegador**, não por inspeção visual:
   - servir o app localmente (havia um dev server na porta 3001);
   - limpar Service Worker e caches antes de testar, senão você testa código antigo;
   - injetar o estado no localStorage, chamar as funções reais e conferir o resultado;
   - **sempre restaurar o localStorage original ao final do teste**;
   - conferir que não há erro no console.
4. **Checagem de sintaxe** antes de commitar: extrair o maior bloco `<script>` do
   index.html para um arquivo e rodar `node --check`.
5. **Commit e push:**
   ```
   git stash && git pull --rebase && git stash pop
   git add index.html sw.js
   git commit -m "vX.Y.Z: descrição"
   git push
   ```
   A mensagem de commit deve explicar a **causa raiz**, não só o sintoma.
6. **Relate honestamente:** o que foi testado, o que passou, o que ficou de fora.
   Se algo não foi validado, diga.

---

## 8. Estado atual (v4.7.2)

Funcionalidades ativas: importação semanal por JSON com preview, cards de treino com
rua e esteira, modo Em Treino com voz e timer, protocolo de joelho com carrossel,
registro de treino com PSE, histórico de treinos e corridas, resultado real de prova
versus estimado, recordes por distância com PR manual e animação, métricas com carga e
streak, avaliação física com VAM e zonas, envio ao Intervals.icu, backup manual e
backup automático em Gist privado do GitHub, guia para o coach.

O sistema visual 4.7 usa uma linguagem de painel esportivo: superfícies em carvão e grafite,
menta para desempenho, amarelo e coral para estados, cartões amplos, navegação inferior flutuante
e uma identidade de rota abstrata na Home. O modo escuro é o padrão apenas para instalações novas;
preferências já salvas continuam preservadas.

A marca 4.7.1 usa uma rota geométrica em forma de R, com menta como caminho principal e trechos
amarelo e coral representando variações de intensidade. O mesmo símbolo aparece na Home, no favicon
e nos ícones 192 e 512 do manifesto, sem arquivo de imagem externo.

A versão 4.7.2 corrige o manifesto antigo exibido na instalação do Android: a URL do manifesto é
versionada, sua identidade é fixada por `id: "/"` e o cache deixa de reter metadados anteriores.

Pontos conhecidos, não resolvidos:
- `PLAN_START` e `PLAN_END` são fixos e definem a barra de progresso para todos os
  usuários. Para um usuário novo isso é arbitrário. Decisão de produto pendente.
- `runcoach_backup.json` na raiz é um artefato local do dono, já no `.gitignore`.

---

## 9. Como atualizar este handoff

Antes de migrar para uma conversa nova, atualize este arquivo com a versão atual, as alterações feitas,
as decisões tomadas, os arquivos importantes, o que foi validado, as pendências e o próximo passo.
O objetivo é permitir que a conversa seguinte continue o trabalho sem depender do histórico anterior.
