# Run Coach Daily

Progressive Web App (PWA) para acompanhamento de treinos de corrida. Você importa o plano semanal gerado pelo seu coach (humano ou de IA), executa os treinos com timer guiado por voz, registra os resultados e acompanha métricas, recordes e provas.

## Acesse o App

**[runcoachdaily.pages.dev](https://runcoachdaily.pages.dev)**

Funciona direto no navegador do celular e pode ser instalado na tela inicial como um app nativo.

## Funcionalidades

### Home
- Saudação personalizada, frase motivacional e versículo do dia
- Card "Próximo treino" com a próxima atividade pendente da semana, em ordem cronológica
- Aderência da semana (atividades concluídas sobre o total)

### Semana
- Importação do plano semanal via JSON, com preview antes de confirmar
- Cards por treino com aquecimento, bloco principal, desaquecimento, zonas e pace alvo
- Alternância entre versão de rua e de esteira
- Protocolo de fortalecimento do joelho como atividade da semana, com carrossel guiado exercício a exercício
- Registro de treino com distância, pace, tempo, dor no joelho, observações e PSE

### Modo "Em Treino"
- Overlay em tela cheia com timer em tempo real
- Acompanhamento fase a fase, incluindo treinos intervalados com ciclos
- Anúncios por voz (Web Speech API), beeps e vibração nas transições
- Tela sempre ligada durante o treino (Wake Lock API)
- Retomada de treino interrompido

### Histórico
- Aba Treinos com todos os registros, edição e exclusão
- Aba Corridas com provas futuras, concluídas e aguardando resultado
- Registro do resultado real da prova (tempo e pace) lado a lado com o estimado
- Recordes pessoais por distância, calculados dinamicamente
- PR manual, para registrar um recorde antigo sem precisar cadastrar a prova
- Animação comemorativa quando um resultado bate o recorde da distância

### Métricas
- Resumo da semana e totais gerais (distância, tempo, pace médio, melhor pace)
- Evolução de pace, streak de consistência e alertas registrados
- Gráfico de carga das últimas semanas
- Desempenho por tipo de treino

### Avaliação
- Testes de VAM e estimativas de VO2max
- Cálculo automático das zonas de treino (Z1 a Z5) a partir do seu resultado

### Integrações (opcionais)
- **Intervals.icu**: envia o treino estruturado para o relógio, com pace alvo por zona
- **GitHub Gist privado**: backup automático dos seus dados a cada mudança
- **Backup manual**: exportação e importação de um arquivo JSON completo

## Formato do JSON Semanal

O plano é gerado fora do app (por um coach ou por um chat de IA) e importado pela aba Semana.

> **Fonte de verdade:** o app tem um **"Guia para o coach"** em **Ajustes > Backup e dados**, com o modelo em branco, as perguntas que o coach deve fazer e todas as regras de formato. Ele é gerado a partir do mesmo código que valida a importação, então nunca fica desatualizado. Use o botão **Copiar guia** e cole no chat do seu coach. O exemplo abaixo serve só para dar uma ideia do formato.

```json
{
  "semana": 1,
  "fase": "Base",
  "foco_da_semana": "Adaptação aeróbica com volume baixo",
  "alerta_coach": null,
  "observacao_coach": "Mantenha o ritmo confortável nesta fase.",
  "prs_manuais": [
    { "distancia_km": 5, "tempo": "30:00" }
  ],
  "estimativas_provas": [
    {
      "nome": "Corrida do Parque 10K",
      "data": "2026-09-20",
      "distancia": "10K",
      "tempo_estimado": "55'00\"",
      "pace_estimado": "5'30\"/km",
      "nova": true
    }
  ],
  "treinos": [
    {
      "dia": "Segunda",
      "tipo": "Corrida Leve",
      "objetivo": "Volume aeróbico controlado",
      "rua": {
        "aquecimento": "5 min caminhada",
        "zona_aquecimento": "Z1",
        "principal": "20 min contínuo em pace 7'45\"/km a 8'30\"/km",
        "zona_principal": "Z2",
        "desaquecimento": "5 min caminhada",
        "zona_desaquecimento": "Z1",
        "distancia_km": 2.5,
        "duracao_total": "30 min",
        "pace_alvo_central_segundos": 488
      },
      "esteira": {
        "aquecimento": "5 min a 5,0 km/h",
        "principal": "20 min a 7,0 km/h a 7,5 km/h",
        "desaquecimento": "5 min a 5,0 km/h",
        "distancia_km": 2.5,
        "duracao_total": "30 min"
      }
    }
  ],
  "protocolo_joelho": null
}
```

### Regras que o parser exige

| Regra | Detalhe |
|-------|---------|
| **Datas** | Sempre `AAAA-MM-DD`. Ano solto (`"2027"`) ou `DD/MM/AAAA` não são aceitos como data válida |
| **Protocolo de joelho** | Só no campo `protocolo_joelho`, nunca repetido como treino na lista `treinos`. Use `null` quando não houver |
| **Semana com prova** | O treino `"tipo": "Prova"` deve cair no mesmo dia e ter a mesma distância da prova em `estimativas_provas`, para o app vincular os dois e checar recorde |
| **Treino intervalado** | Usa `intervalos` com `ciclos` e `fases` dentro do bloco de rua ou esteira |
| **Pace alvo** | `pace_alvo_central_segundos` é o pace em segundos por km (7'30" = 450) |
| **Campos opcionais** | `prs_manuais` e `estimativas_provas` aceitam `[]` quando não houver |

## Stack Técnica

| Item | Detalhe |
|------|---------|
| **Frontend** | HTML + CSS + Vanilla JS em arquivo único, sem dependências |
| **Hospedagem** | Cloudflare Pages |
| **Armazenamento** | localStorage (os dados ficam no aparelho) |
| **PWA** | Service Worker com cache versionado + Manifest |
| **Deploy** | Automático via GitHub (push na main) |
| **Servidor** | Nenhum, 100% estático |

## Estrutura do Projeto

```
├── index.html                        # App completo (HTML + CSS + JS)
├── manifest.json                     # Configuração do PWA
├── sw.js                             # Service Worker (cache offline versionado)
├── version.json                      # Versão publicada (atualizada pelo CI)
├── _headers                          # Headers de segurança (Cloudflare Pages)
├── .github/workflows/                # Action que atualiza o version.json
├── .gitignore
└── README.md
```

## Dados e Privacidade

Não existe conta, login nem servidor. Tudo fica no `localStorage` do próprio aparelho, então cada celular tem seus dados isolados. Isso torna o app naturalmente multiusuário: cada pessoa usa no seu aparelho, sem interferir na outra.

Uma instalação nova começa **vazia**: sem corridas, sem protocolo, sem avaliação e sem nome preenchido. Nada é pré-carregado a partir de dados de outra pessoa.

Chaves usadas no `localStorage`:

| Chave | Conteúdo |
|-------|----------|
| `runcoach_semana_atual` | Semana em andamento |
| `runcoach_historico_semanas` | Semanas anteriores |
| `runcoach_logs` | Treinos registrados |
| `runcoach_corridas` | Provas cadastradas e seus resultados |
| `runcoach_prs_manuais` | Recordes informados manualmente |
| `runcoach_protocolo_joelho` / `_status` / `_posicao` | Protocolo, conclusões e posição no carrossel |
| `runcoach_avaliacao` | Resultado do teste e zonas |
| `runcoach_settings` | Nome, tema e preferência de áudio |
| `runcoach_treino_ativo` | Treino em andamento (para retomar) |
| `runcoach_intervals_key` | Chave do Intervals.icu |
| `runcoach_github_token` / `_gist_id` / `_sync_meta` | Backup no GitHub |

### Credenciais

As chaves do Intervals.icu e do GitHub ficam **apenas** no aparelho, são digitadas em campos de senha e **nunca** entram no backup manual nem no backup do GitHub. Nenhuma chave de API existe no código.

## Backup

- **Manual**: em Ajustes, exporta um arquivo JSON com todos os dados e importa de volta em qualquer aparelho, sem depender de internet ou conta externa.
- **Automático (GitHub)**: com um token de escopo `gist`, o app cria um Gist privado e o atualiza sozinho a cada mudança relevante. Serve para recuperar os dados caso o cache do navegador seja limpo.

> Limpar "cookies e dados de sites" no navegador apaga o `localStorage` e, com ele, todos os dados do app. Limpar apenas o cache de arquivos não afeta os dados.

## Instalação como App (PWA)

### Android (Chrome)
1. Acesse [runcoachdaily.pages.dev](https://runcoachdaily.pages.dev)
2. Toque no menu (3 pontos) e depois em **"Adicionar à tela inicial"**
3. Confirme o nome e toque em **"Adicionar"**

### iPhone (Safari)
1. Acesse [runcoachdaily.pages.dev](https://runcoachdaily.pages.dev)
2. Toque no botão de compartilhar (quadrado com seta)
3. Role e toque em **"Adicionar à Tela de Início"**

## Deploy

O deploy é automático: qualquer push na branch `main` aciona o Cloudflare Pages, que publica em cerca de 30 segundos. Uma GitHub Action atualiza o `version.json`, usado pelo app para avisar que existe uma versão nova.

```bash
git add .
git commit -m "descrição da mudança"
git push origin main
```

Ao mexer no `index.html`, lembre de subir o `APP_VERSION`, adicionar a entrada no `CHANGELOG` e trocar o nome do cache no `sw.js`, senão o Service Worker continua servindo a versão antiga.

## Desenvolvimento

O desenvolvimento do projeto é conduzido com o Codex. Para retomar o trabalho em uma nova conversa, leia e atualize o arquivo `HANDOFF_CODEX.md`, que reúne a arquitetura, as regras do projeto, as pendências e os próximos passos.

Para alterações de código, use uma conversa por objetivo: funcionalidade, bug, refatoração ou revisão. Ao concluir a alteração, valide, faça o commit e envie para a branch `main`.

## Histórico de Versões

O changelog completo fica **dentro do app**, em Ajustes > Sobre > Histórico de versões. Marcos principais:

| Versão | Destaques |
|--------|-----------|
| **4.7** | Novo sistema visual esportivo, navegação flutuante e Home com identidade de rota |
| **4.6** | Treino de Prova vinculado à corrida cadastrada, recorde checado pelos dois caminhos, instalação nova sem dados pré-carregados |
| **4.5** | Recordes pessoais manuais, animação de novo PR, guia para o coach, blindagem contra JSON de formato antigo |
| **4.4** | Backup automático em Gist privado do GitHub, gráfico de carga semanal |
| **4.3** | Histórico de corridas com resultado real versus estimado e recordes por distância |
| **4.2** | Envio ao relógio com bloco principal e distância, normalização de tipos de treino |
| **4.1** | Protocolo de fortalecimento do joelho com carrossel guiado |
| **4.0** | Integração com Intervals.icu, abas reorganizadas, métricas com streak e carga |
| **3.x** | Avaliação física com VAM, VO2max e zonas de treino |
| **2.x** | Multiusuário, importação por JSON, calendário de corridas, migração para Cloudflare Pages |
| **1.x** | Versão inicial, modo "Em Treino" com voz e timer, histórico e métricas |

## Licença

Projeto pessoal. Todos os direitos reservados.

---

Desenvolvido com Codex
