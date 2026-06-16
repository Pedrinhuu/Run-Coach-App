# Run Coach Daily

Aplicativo Progressive Web App (PWA) para acompanhamento de treinos de corrida. Planeje suas semanas, registre seus treinos, acompanhe métricas e mantenha-se motivado com frases diárias e versículos.

## Acesse o App

**[runcoachdaily.pages.dev](https://runcoachdaily.pages.dev)**

Funciona direto no navegador do celular. Pode ser instalado na tela inicial como um app nativo.

## Funcionalidades

### Treino da Semana
- Importação de plano semanal via JSON (gerado pelo seu coach ou pelo Claude)
- Visualização dos treinos do dia com detalhes de aquecimento, intervalos e desaquecimento
- Status de cada treino: Pendente, Concluído ou Não feito
- Registro de log ao concluir (distância, pace, tempo, observações)

### Modo "Em Treino"
- Overlay fullscreen com timer em tempo real
- Acompanhamento fase a fase (aquecimento → intervalos → desaquecimento)
- Anúncios por voz (Web Speech API) das fases e transições
- Beeps sonoros e vibração nos intervalos
- Tela sempre ligada durante o treino (Wake Lock API)

### Histórico
- Registro de todas as semanas importadas
- Visualização e edição de treinos passados
- Exclusão de semanas ou treinos individuais

### Métricas
- Dashboard com estatísticas de treino
- Distância total, pace médio, tempo total
- Volume semanal e frequência de treinos
- Visualização read-only dos dados acumulados

### Calendário de Corridas
- Cadastro de provas futuras (nome, data, distância, meta de pace)
- Estimativas de tempo para cada prova
- Edição e exclusão de corridas
- Importação automática de provas via JSON semanal

### Personalização
- Nome do usuário com saudação personalizada (bom dia/tarde/noite)
- Frase motivacional do dia (30 frases, rotação diária)
- Versículo bíblico do dia (30 versículos, rotação diária)
- Tema claro e escuro
- Opção de silenciar áudio/voz

## Stack Técnica

| Item | Detalhe |
|------|---------|
| **Frontend** | HTML + CSS + Vanilla JS (arquivo único) |
| **Hospedagem** | Cloudflare Pages |
| **Armazenamento** | localStorage (dados ficam no celular) |
| **PWA** | Service Worker + Manifest |
| **Deploy** | Automático via GitHub (push na main) |
| **Servidor** | Nenhum — 100% estático |

## Estrutura do Projeto

```
├── index.html      # App completo (HTML + CSS + JS)
├── manifest.json   # Configuração do PWA
├── sw.js           # Service Worker (cache offline)
├── _headers        # Headers de segurança (Cloudflare Pages)
├── .gitignore      # Arquivos ignorados pelo git
└── README.md       # Este arquivo
```

## Multiusuário

Cada celular tem seus próprios dados isolados via localStorage. Não existe conta nem login — basta abrir o app e começar a usar. Ideal para casais ou famílias onde cada pessoa usa no seu próprio celular.

## Formato do JSON Semanal

O plano semanal é gerado externamente (por um coach ou pelo Claude) e importado no app. Formato esperado:

```json
{
  "semana": 1,
  "titulo": "Semana 1 — Base aeróbica",
  "objetivo": "Construir base com corridas leves",
  "dias": [
    {
      "dia": "Segunda",
      "tipo": "Corrida leve",
      "descricao": "Corrida contínua em ritmo confortável",
      "aquecimento": "5 min caminhada",
      "detalhes": [
        { "repeticoes": 1, "acao": "Correr", "duracao": "30 min", "pace": "6'30\"/km" }
      ],
      "desaquecimento": "5 min caminhada",
      "tempo_total": "40 min"
    }
  ],
  "estimativas_provas": [
    { "nome": "Meia Maratona XYZ", "tempo_estimado": "2h05min" }
  ]
}
```

## Instalação como App (PWA)

### Android (Chrome)
1. Acesse [runcoachdaily.pages.dev](https://runcoachdaily.pages.dev)
2. Toque no menu (3 pontos) → **"Adicionar à tela inicial"**
3. Confirme o nome e toque em **"Adicionar"**

### iPhone (Safari)
1. Acesse [runcoachdaily.pages.dev](https://runcoachdaily.pages.dev)
2. Toque no botão de compartilhar (quadrado com seta)
3. Role e toque em **"Adicionar à Tela de Início"**

## Deploy

O deploy é automático. Qualquer push na branch `main` aciona o Cloudflare Pages, que publica a nova versão em ~30 segundos.

```bash
git add .
git commit -m "descrição da mudança"
git push origin main
```

## Histórico de Versões

| Versão | Destaques |
|--------|-----------|
| **2.2** | Migração para Cloudflare Pages |
| **2.1** | Correção de botões duplicados na importação |
| **2.0** | Renomeado para Run Coach Daily |
| **1.9** | Edição de nome, frases motivacionais reescritas |
| **1.8** | Saudação personalizada, frase e versículo do dia |
| **1.7** | Modo multiusuário, tela de boas-vindas |
| **1.6** | Wheel pickers estilo Zepp |
| **1.5** | Aba de métricas com dashboard completo |
| **1.4** | Importação JSON, remoção do chat com IA |
| **1.3** | Calendário de corridas com estimativas |
| **1.2** | Edição e exclusão de treinos no histórico |
| **1.1** | Modo "Em Treino" com voz, timer e wake lock |
| **1.0** | Versão inicial do app |

## Licença

Projeto pessoal. Todos os direitos reservados.

---

Feito com Claude Code
