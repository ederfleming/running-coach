# Running Coach

Aplicativo pessoal para acompanhar treinos de corrida de forma local-first, simples e usavel durante a corrida.

Esta primeira versao foi implementada como um `index.html` estatico para ser facil de abrir, testar e compartilhar. Os dados ficam salvos localmente no navegador do aparelho via `localStorage`.

## Objetivo

Permitir acompanhar um plano de corrida sem backend, login ou sincronizacao.

O app deve permitir:

- visualizar o treino atual;
- executar treinos guiados por segmentos;
- registrar resultado pos-treino;
- acompanhar historico;
- importar planos gerados pelo ChatGPT em JSON;
- exportar backup JSON.

## Escopo da versao HTML

Incluido:

- dashboard com proximo treino, progresso, concluidos, km acumulados e sequencia;
- plano por semanas;
- status de treino planejado/concluido;
- tela de execucao com cronometro, etapa atual, velocidade, pace e progresso;
- controles de iniciar, pausar, continuar, voltar etapa, avancar etapa e finalizar;
- formulario de resultado com distancia, tempo, esforco, dor antes/durante/depois e observacoes;
- historico local;
- importacao de JSON direto na tela inicial e na tela de dados;
- exportacao de backup JSON;
- estado inicial sem treinos cadastrados.

Fora desta versao:

- SQLite;
- Expo/React Native;
- instalacao via AltStore;
- HealthKit;
- Apple Watch;
- notificacoes;
- sincronizacao;
- autenticacao;
- graficos avancados.

## Limitacao importante

Como esta versao e um HTML estatico, a persistencia depende do navegador onde o arquivo for aberto.

No iPhone, abrir o arquivo diretamente pelo WhatsApp pode usar um visualizador temporario. Para nao perder dados, o caminho mais confiavel e:

1. abrir o arquivo no Safari ou Chrome;
2. usar sempre o mesmo navegador;
3. exportar backup JSON com frequencia.

Para persistencia mais robusta, a proxima evolucao recomendada e transformar em PWA hospedada em uma URL simples ou iniciar o app Expo com SQLite.

## Modelo de dados atual

O backup exportado possui esta estrutura:

```json
{
  "plan": {
    "id": "10k-sub60-base",
    "name": "10 km abaixo de 1h",
    "weeks": [
      {
        "id": "w1",
        "title": "Semana 1",
        "workouts": [
          {
            "id": "w1-t1",
            "day": "Terca",
            "title": "Esteira leve",
            "target": "Base aerobica",
            "segments": [
              {
                "name": "Aquecimento",
                "minutes": 5,
                "speedKmh": 7,
                "note": "Solto e confortavel"
              }
            ]
          }
        ]
      }
    ]
  },
  "results": [
    {
      "id": "result-123",
      "workoutId": "w1-t1",
      "completedAt": "2026-08-03T00:00:00.000Z",
      "distanceKm": 5,
      "durationMin": 35,
      "effort": 5,
      "painBefore": 0,
      "painDuring": 0,
      "painAfter": 0,
      "notes": "Esteira"
    }
  ]
}
```

## Formato de importacao de plano

O app aceita um JSON contendo `plan` ou diretamente o objeto do plano.

Exemplo minimo:

```json
{
  "plan": {
    "id": "meu-plano",
    "name": "10 km sub 60",
    "weeks": [
      {
        "id": "semana-1",
        "title": "Semana 1",
        "workouts": [
          {
            "id": "s1-t1",
            "day": "Terca",
            "title": "Esteira leve",
            "target": "Base",
            "segments": [
              {
                "name": "Corrida leve",
                "minutes": 30,
                "speedKmh": 8.5,
                "note": "Confortavel"
              }
            ]
          }
        ]
      }
    ]
  }
}
```

Regras:

- cada semana precisa ter `workouts`;
- cada treino precisa ter `segments`;
- cada segmento precisa ter `name`, `minutes` e `speedKmh`;
- velocidades devem preferencialmente ser multiplos de `0.5 km/h`.

## Roadmap recomendado

### Fase 1 - HTML estatico validavel

Objetivo: validar uso real durante treino sem instalar app.

Entregas:

- melhorar fluxo no iPhone;
- testar abertura via WhatsApp, Arquivos e Safari;
- ajustar tamanho de fonte e botoes em uso real;
- validar se `localStorage` e suficiente por enquanto;
- incluir confirmacao antes de sobrescrever plano importado.

### Fase 2 - PWA simples

Objetivo: manter simplicidade do HTML, mas com experiencia mais confiavel no celular.

Entregas:

- hospedar em URL propria;
- adicionar manifest;
- permitir "Adicionar a Tela de Inicio";
- usar IndexedDB em vez de `localStorage`;
- melhorar backup/restauracao.

### Fase 3 - Expo MVP

Objetivo: virar aplicativo mobile real para iPhone.

Stack:

- Expo;
- React Native;
- TypeScript;
- Expo Router;
- Expo SQLite;
- Zustand;
- React Hook Form;
- Zod;
- NativeWind ou alternativa equivalente.

Entregas:

- estrutura de pastas;
- banco SQLite com migrations;
- repositorios locais;
- schemas Zod de importacao/exportacao;
- telas: Dashboard, Plano, Detalhe, Execucao, Registro, Historico, Dados;
- instalacao via AltStore.

### Fase 4 - Evolucao de treino

Objetivo: acompanhar melhor carga e progresso.

Entregas:

- calendario;
- graficos;
- estatisticas semanais;
- carga semanal incluindo CrossFit/musculacao;
- edicao manual de treino;
- notificacoes locais.

## Criterios de sucesso do MVP atual

- abrir `index.html` no navegador;
- iniciar sem treinos cadastrados;
- importar plano JSON valido pela tela inicial;
- iniciar um treino;
- avancar/voltar etapas;
- finalizar treino;
- salvar resultado;
- ver resultado no historico;
- copiar resumo do historico para analise no chat;
- baixar historico JSON;
- fechar e abrir novamente mantendo os dados;
- exportar backup JSON;
- exportar backup JSON.

## Proximas decisoes

1. Manter HTML estatico por enquanto ou evoluir direto para Expo?
2. O acompanhamento deve ser por plano fechado ou por agenda semanal recorrente?
3. O historico deve registrar apenas corrida ou tambem CrossFit/musculacao desde ja?
4. O treino concluido pode ser editado depois?
5. Ao importar novo plano, deve apagar resultados antigos ou manter historico separado?
