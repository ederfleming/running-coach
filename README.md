# Running Coach

Aplicativo pessoal para acompanhar treinos de corrida de forma local-first, simples e usável durante a corrida.

Esta primeira versão foi implementada como um `index.html` estático para ser fácil de abrir, testar, compartilhar e públicar no GitHub Pages. Os dados ficam salvos localmente no navegador do aparelho via `localStorage`.

## Publicacao e privacidade

O projeto pode ser públicado gratuitamente no GitHub Pages usando um repositório público.

Pontos importantes:

- o código do app fica público no GitHub;
- plano, histórico e resultados ficam salvos localmente no navegador, não no repositório;
- não versionar arquivos de backup, histórico exportado ou planos pessoais sensiveis;
- ao públicar mudanças, basta enviar `index.html`, `README.md`, `manifest.webmanifest`, `icon.svg` e `service-worker.js`;
- repositório privado com GitHub Pages pode exigir plano pago, dependendo da conta/organização.

A URL pública esperada neste projeto e:

```txt
https://ederfleming.github.io/running-coach/
```

## Objetivo

Permitir acompanhar um plano de corrida sem backend, login ou sincronização.

O app deve permitir:

- visualizar o treino atual;
- executar treinos guiados por segmentos;
- registrar resultado pós-treino;
- acompanhar histórico;
- importar planos gerados pelo ChatGPT em JSON;
- exportar backup JSON.

## Escopo da versão HTML

Incluído:

- dashboard com próximo treino, progresso, concluídos, km acumulados e sequência;
- plano por semanas;
- status de treino planejado/concluído;
- tela de execução com cronômetro, etapa atual, velocidade, pace e progresso;
- alternância entre modo Esteira e modo Rua na tela de treino;
- no modo Esteira, exibição de cronômetro, velocidade e pace da etapa;
- no modo Rua, exibição de km alvo, faixa de pace e tempo estimado;
- cronômetro persistido por timestamp para recuperar tempo após bloqueio/reabertura;
- tentativa de manter a tela ligada durante o treino com Screen Wake Lock quando suportado;
- controles de iniciar, pausar, continuar, voltar etapa, avançar etapa e finalizar;
- formulário de resultado com distância, tempo, esforço, dor antes/durante/depois e observações;
- histórico local;
- importação de JSON direto na tela inicial e na tela de dados;
- exportação de backup JSON;
- estado inicial sem treinos cadastrados.

Fora desta versão:

- SQLite;
- Expo/React Native;
- instalacao via AltStore;
- HealthKit;
- Apple Watch;
- notificacoes;
- sincronização;
- autenticacao;
- graficos avançados.

## Limitacao importante

Como esta versão e um HTML estático, a persistência depende do navegador onde o arquivo for aberto.

No iPhone, abrir o arquivo diretamente pelo WhatsApp pode usar um visualizador temporario. Para não perder dados, o caminho mais confiavel e:

1. abrir o arquivo no Safari ou Chrome;
2. usar sempre o mesmo navegador;
3. exportar backup JSON com frequência.

Para persistência mais robusta, a próxima evolução recomendada e transformar em PWA hospedada em uma URL simples ou iniciar o app Expo com SQLite.

Durante a execução do treino, o app salva o estado do cronômetro usando data/hora real. Se o iPhone bloquear a tela ou suspender o JavaScript, ao voltar o app recalcula o tempo decorrido e avança para a etapa correta. O app também tenta usar Screen Wake Lock para manter a tela ligada, mas o suporte depende do navegador/iOS e não é garantido.

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
            "day": "Terça",
            "title": "Esteira leve",
            "target": "Base aeróbica",
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

## Formato de importação de plano

O app aceita um JSON contendo `plan` ou diretamente o objeto do plano.

Exemplo mínimo:

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
            "day": "Terça",
            "title": "Intervalado moderado",
            "target": "Blocos fortes com recuperação leve",
            "type": "Treino Intervalado (Tiros)",
            "intensityZone": "Z3-Z4",
            "segments": [
              {
                "name": "Aquecimento",
                "minutes": 10,
                "speedKmh": 7.5,
                "zone": "Z1-Z2",
                "note": "Comece confortável"
              },
              {
                "name": "Bloco moderado 1",
                "minutes": 2,
                "speedKmh": 9.2,
                "zone": "Z3-Z4",
                "note": "Ritmo controlado"
              },
              {
                "name": "Recuperação 1",
                "minutes": 1.5,
                "speedKmh": 7.5,
                "zone": "Z1-Z2",
                "note": "Respire e recupere"
              }
            ]
          }
        ]
      }
    ]
  }
}
```


Campos opcionais por treino para corrida na rua:

- `type`: tipo do treino. Use `Rodagem Leve (Tiro Curto/Fácil)`, `Longão`, `Treino Progressivo`, `Treino Intervalado (Tiros)`, `Fartlek`, `Treino de Ritmo (Tempo Run)` ou `Treino de Recuperação ou Regenerativo`;
- `intensityZone`: zona principal do treino, como `Z1`, `Z2`, `Z3`, `Z4`, `Z5` ou faixas como `Z3-Z4`;
- `targetDistanceKm`: distância alvo total do treino em km;
- `paceRangeMinKm`: faixa de pace alvo no formato `["6:00", "6:30"]`;
- alternativamente, pode usar `paceMinKm` e `paceMaxKm`;


A zona de cada etapa deve ser informada com `zone` ou `intensityZone`. Quando ausente, o app usa a zona principal do treino como fallback.

## Zonas de pace

Na aba `Dados`, o questionário de 3 km calcula faixas iniciais de pace para Z1 a Z5. Informe o tempo no formato `min:seg` e confirme que o teste foi realizado em esforço forte e constante. Opcionalmente, informe FC máxima medida e FC de repouso para ver as mesmas zonas em BPM pela reserva de frequência cardíaca. Quando um treino ou segmento tiver `zone` ou `intensityZone`, a faixa pessoal também aparece no plano e na tela de execução. O perfil fica salvo localmente e é incluído no backup JSON completo.
- cada segmento também pode ter `zone` ou `intensityZone` para indicar a zona daquela etapa.

Quando esses campos existem, a tela de treino mostra km alvo, pace alvo e tempo estimado. Quando não existem, o app calcula distância e pace médio a partir dos segmentos com `minutes` e `speedKmh`.

Regras:

- cada semana precisa ter `workouts`;
- cada treino precisa ter `segments`;
- cada segmento precisa ter `name`, `minutes` e `speedKmh`;
- velocidades devem preferencialmente ser multiplos de `0.5 km/h`.

## Roadmap recomendado

### Fase 1 - HTML estático validavel

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
- repositórios locais;
- schemas Zod de importação/exportação;
- telas: Dashboard, Plano, Detalhe, Execução, Registro, Histórico, Dados;
- instalacao via AltStore.

### Fase 4 - Evolução de treino

Objetivo: acompanhar melhor carga e progresso.

Entregas:

- calendario;
- graficos;
- estatisticas semanais;
- carga semanal incluindo CrossFit/musculacao;
- edição manual de treino;
- notificacoes locais.

## Criterios de sucesso do MVP atual

- abrir `index.html` no navegador;
- iniciar sem treinos cadastrados;
- importar plano JSON valido pela tela inicial;
- iniciar um treino;
- avançar/voltar etapas;
- finalizar treino;
- salvar resultado;
- ver resultado no histórico;
- copiar resumo do histórico para análise no chat;
- baixar histórico JSON;
- fechar e abrir novamente mantendo os dados;
- exportar backup JSON;
- exportar backup JSON.

## Próximas decisões

1. Manter HTML estático por enquanto ou evoluir direto para Expo?
2. O acompanhamento deve ser por plano fechado ou por agenda semanal recorrente?
3. O histórico deve registrar apenas corrida ou também CrossFit/musculacao desde ja?
4. O treino concluído pode ser editado depois?
5. Ao importar novo plano, deve apagar resultados antigos ou manter histórico separado?
