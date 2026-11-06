# Agenda Cultural Belém — Notificações Push (Flutter + FCM)

Atividade Prática 4 — Programação para Dispositivos Móveis.

Aplicativo Flutter com **duas telas** (agenda e detalhes do evento) que usa o **Firebase Cloud Messaging (FCM)** para avisar o público sobre a programação cultural de Belém. O mesmo código roda **no Android** e **no navegador (Chrome)**, publicado num link do Firebase Hosting. Os dados dos eventos ficam no próprio código (`lib/data/eventos_data.dart`). Os eventos são fictícios e foram criados só para esta atividade.

---

## 1. Finalidade e decisões

**Finalidade:** reunir num só lugar os eventos culturais da semana em Belém (música, gastronomia, cinema, artes) e avisar o usuário no momento certo, sem que ele precise abrir o app para descobrir as novidades.

**Público-alvo:** moradores e turistas de Belém que gostam de programação cultural, principalmente a gratuita.

**Quando uma notificação é útil:**

| # | Situação que motiva o aviso | Por que notificar |
|---|---|---|
| 1 | A programação da semana/mês foi publicada | O usuário não abre o app todo dia; o aviso traz ele de volta quando há novidade |
| 2 | Um evento foi escolhido como **destaque da semana** | A imagem chama atenção e vende o evento já na barra de notificações |
| 3 | Um evento está com **ingressos acabando** | É urgente: o toque precisa levar direto ao evento (data, local, preço), sem procurar na lista |

---

## 2. As três notificações (o que enviar no Firebase Console)

> Caminho no console: **Firebase Console → seu projeto → Messaging (Engajamento/Run → Messaging) → Nova campanha → Notificações do Firebase**.

### Desafio 1 — Notificação simples
| Campo | Valor |
|---|---|
| Título | `Programação de novembro no ar! 🎭` |
| Texto | `5 eventos culturais em Belém esta semana, 3 deles gratuitos. Toque para ver a agenda.` |
| Imagem | (vazio) |
| Dados personalizados | (nenhum) |

Resultado esperado: com o app em **segundo plano**, a notificação aparece na barra do Android (ou como notificação do Chrome no Windows). Ao tocar, o app abre na **tela inicial (agenda)**.

### Desafio 2 — Notificação personalizada com imagem
| Campo | Valor |
|---|---|
| Título | `Destaque da semana: Festival de Carimbó 🌺` |
| Texto | `Sábado, 18h, na Praça da República. Roda aberta, oficina de dança e entrada gratuita!` |
| Imagem da notificação | `https://raw.githubusercontent.com/SEU_USUARIO/SEU_REPOSITORIO/main/notificacoes/carimbo.jpg` |
| Dados personalizados | (nenhum, ou `evento_id = 1` se quiser que o toque abra o festival) |

Resultado esperado: a notificação mostra a **miniatura** e, ao ser expandida (arrastar para baixo), a **imagem grande** do festival.

> A imagem precisa estar numa URL **pública HTTPS** (JPG/PNG, até 1 MB). As imagens estão na pasta `notificacoes/` deste repositório — com o repositório **público** no GitHub, use o link "raw" acima. Para conferir o link, abra-o no navegador: deve aparecer só a imagem.

### Desafio 3 — Notificação com navegação
| Campo | Valor |
|---|---|
| Título | `Últimos ingressos! 🎻` |
| Texto | `Concerto Sinfônico Amazônico nesta sexta, 20h, no Theatro da Paz. Toque para ver os detalhes.` |
| Imagem | opcional: `.../notificacoes/concerto.jpg` |
| **Dados personalizados** (Opções adicionais) | chave **`evento_id`** · valor **`3`** |

Resultado esperado: ao tocar na notificação, o app abre **direto na tela de detalhes do Concerto**, com uma faixa "Aberto a partir de uma notificação (evento_id = 3)". Funciona com o app:
- **em segundo plano** → Android: `FirebaseMessaging.onMessageOpenedApp` · Web: o `firebase-messaging-sw.js` foca a aba e envia o `evento_id` para o app;
- **fechado** (iniciado pelo toque) → Android: `FirebaseMessaging.instance.getInitialMessage()` · Web: o service worker abre o app com `?evento_id=3` na URL, que o app lê ao iniciar.

> Na web o plugin `firebase_messaging` não implementa `getInitialMessage` nem `onMessageOpenedApp` (retornam vazio), por isso o clique é tratado no service worker.

IDs disponíveis: `1` Carimbó · `2` Feira de Sabores · `3` Concerto · `4` Mostra de Cinema · `5` Arte Marajoara. Um id inexistente abre a tela "Evento não encontrado".

### Bônus — app aberto (primeiro plano)
Com o app aberto (aba visível, no caso da web) o sistema **não** mostra a notificação sozinho. O app trata isso em `FirebaseMessaging.onMessage` exibindo um aviso dentro do app (título, mensagem, imagem e botão **VER** quando há `evento_id`).

---

## 3. Como rodar no Android Studio (passo a passo)

> **Vai usar só a versão web (Chrome)?** Faça 3.1 (o emulador é dispensável), 3.2 marcando **Web** em Platforms, 3.3, o item 1 de 3.4 (criar o projeto no Firebase) e então pule para a **seção 4**. O Gradle (3.5) e o `google-services.json` só são necessários para o Android.

### 3.1 Pré-requisitos
- **Android Studio** com os plugins **Flutter** e **Dart** (Settings → Plugins).
- **Flutter SDK 3.38 ou mais novo** (`flutter --version`; se for mais antigo, rode `flutter upgrade`).
- Um **celular Android** com depuração USB ou um **emulador com Google Play** (Device Manager → criar dispositivo com imagem que tenha o ícone da Play Store, API 33+). Emuladores sem Google Play não recebem push.

### 3.2 Criar o projeto
1. Android Studio → **New Flutter Project** → Flutter → Next.
2. Project name: **`agenda_cultural`** (tem que ser exatamente esse nome, porque os imports usam `package:agenda_cultural/...`).
3. Organization: por exemplo `com.eduardo` → o pacote Android será `com.eduardo.agenda_cultural`.
4. Platforms: marque **Android** (as outras podem ficar desmarcadas) → Create.

### 3.3 Copiar os arquivos deste repositório
Copie para dentro do projeto criado, **substituindo** quando perguntar:
```
lib/                  (todo o conteúdo)
web/firebase-messaging-sw.js   (só se for usar a versão web — seção 4)
firebase.json, .firebaserc     (só para publicar o link — seção 4)
assets/eventos/       (imagens usadas dentro do app)
notificacoes/         (imagens usadas nas notificações)
test/widget_test.dart
pubspec.yaml
analysis_options.yaml
README.md
```
Depois clique em **Pub get** (barra amarela no topo do `pubspec.yaml`) ou rode `flutter pub get` no Terminal do Android Studio.

### 3.4 Criar o projeto no Firebase (plano Spark, gratuito)
1. Acesse <https://console.firebase.google.com> → **Criar projeto** → nome `agenda-cultural` → o Google Analytics pode ficar desativado. O plano já começa no **Spark** (não precisa de cartão).
2. Na visão geral, clique no ícone do **Android** para adicionar um app.
3. **Nome do pacote Android**: copie o `applicationId` de `android/app/build.gradle.kts` (ex.: `com.eduardo.agenda_cultural`). Tem que ser idêntico.
4. Baixe o **`google-services.json`** e coloque em **`android/app/google-services.json`**.
5. Pule a etapa "Adicionar SDK" do console — ela é feita no passo seguinte.

### 3.5 Configurar o Gradle (Kotlin DSL)
**`android/settings.gradle.kts`** — no bloco `plugins { ... }` adicione a linha do Google Services:
```kotlin
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "..." apply false   // já existe
    id("org.jetbrains.kotlin.android") version "..." apply false // já existe
    id("com.google.gms.google-services") version "4.4.2" apply false // ADICIONAR
}
```

**`android/app/build.gradle.kts`** — adicione o plugin e garanta `minSdk = 24`:
```kotlin
plugins {
    id("com.android.application")
    id("kotlin-android")
    id("dev.flutter.flutter-gradle-plugin")
    id("com.google.gms.google-services") // ADICIONAR
}

android {
    // ...
    defaultConfig {
        // ...
        minSdk = 24   // TROCAR "flutter.minSdkVersion" por 24 (exigido pelo firebase_messaging 17)
    }
}
```

> **Alternativa automática:** em vez dos passos 3.4.2 a 3.5, instale o Firebase CLI e o FlutterFire CLI (`npm i -g firebase-tools`, `firebase login`, `dart pub global activate flutterfire_cli`) e rode `flutterfire configure` na pasta do projeto. Ele registra o app, baixa o `google-services.json` e altera o Gradle sozinho. Ainda assim confira o `minSdk = 24`.

A permissão `POST_NOTIFICATIONS` (Android 13+) já é incluída pelo plugin `firebase_messaging`, e o app a solicita na primeira abertura — não é preciso editar o `AndroidManifest.xml`.

### 3.6 Executar
1. Selecione o celular/emulador na barra superior e clique em **Run ▶**.
2. Na primeira abertura, toque em **Permitir** notificações.
3. Toque no ícone de **sino** (canto superior direito) → aparece o **Token FCM** do aparelho → **Copiar token**. O token também é impresso no console **Run** entre as linhas `==== FCM TOKEN ====`.

### 3.7 Enviar as notificações
1. Firebase Console → **Messaging** → **Nova campanha** → **Notificações do Firebase**.
2. Preencha título, texto e (se for o caso) a imagem — tabelas da seção 2.
3. Clique em **Enviar mensagem de teste** → cole o token → **+** → **Testar**.
4. Para o Desafio 3, os **dados personalizados** ficam em **Opções adicionais**. Se o toque não abrir o evento na mensagem de teste, conclua a campanha (Avançar → Segmentação: selecione o seu app → Programação: Agora → **Revisar → Publicar**). A campanha publicada sempre leva os dados personalizados.

### 3.8 Como testar cada estado do app
| Estado | Como deixar o app nesse estado | Quem trata |
|---|---|---|
| Segundo plano | Abra o app e aperte **Home** | Android exibe; toque → `onMessageOpenedApp` |
| Fechado | Abra a tela de **recentes** e **deslize o app para fora** | Android exibe; toque → `getInitialMessage()` |
| Primeiro plano | App aberto na tela | `onMessage` → aviso dentro do app |

> ⚠️ Não use **"Forçar parada"** nas configurações do Android: nesse estado o sistema bloqueia o recebimento de push até o app ser aberto de novo.

---

## 4. Versão web (Chrome) com link público

### 4.1 Habilitar a web no projeto
- Projeto novo: ao criar no Android Studio, marque **Android** e **Web** em Platforms.
- Projeto já criado só com Android: no Terminal do Android Studio rode `flutter create --platforms=web .`
- Copie **`web/firebase-messaging-sw.js`** deste repositório para a pasta `web/` do projeto (**não** substitua o `index.html` que o Flutter gerou). Copie também `firebase.json` e `.firebaserc` para a raiz.

### 4.2 Registrar o app Web no Firebase
1. Firebase Console → engrenagem → **Configurações do projeto** → **Geral** → "Seus apps" → ícone **`</>`** (Web) → apelido `Agenda Cultural Web` → **não** marque Hosting aqui → Registrar.
2. O console mostra um objeto `firebaseConfig`. Copie `apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId` e `appId` para **os dois arquivos**:
   - `lib/firebase_config_web.dart`
   - `web/firebase-messaging-sw.js`
3. **Configurações do projeto** → aba **Cloud Messaging** → **Certificados push da Web** → **Gerar par de chaves**. Copie a chave para `vapidKey` em `lib/firebase_config_web.dart`.
4. Troque `SEU-PROJETO` no `.firebaserc` pelo **ID do projeto** (aparece em Configurações do projeto → Geral).

### 4.3 Testar no seu computador
```
flutter pub get
flutter run -d chrome --web-port 5000
```
A porta fixa mantém o mesmo endereço (`http://localhost:5000`) e, portanto, o mesmo token entre execuções.

### 4.4 Publicar e gerar o link (Firebase Hosting, grátis no Spark)
1. Instale o **Node.js LTS** (<https://nodejs.org>).
2. No Terminal:
```
npm install -g firebase-tools
firebase login
flutter build web --release
firebase deploy --only hosting
```
3. No fim aparece **Hosting URL: `https://SEU-PROJETO.web.app`** — esse é o link do app. A cada mudança no código, repita `flutter build web --release` e `firebase deploy --only hosting`.

### 4.5 Usar no Chrome
1. Abra o link → clique em **Ativar notificações** → **Permitir**.
2. Clique no **sino** → **Copiar token** (o token da web é diferente do token do celular).
3. **Windows:** Configurações → Sistema → Notificações → **Google Chrome ativado** e **Não incomodar desligado**. No **macOS** o Chrome não mostra a imagem grande nas notificações; para o Desafio 2 grave no Windows (ou no Android).

### 4.6 Estados do app na web
| Estado | Como deixar o app nesse estado | Quem trata |
|---|---|---|
| Segundo plano | Mude para **outra aba** (por exemplo, a do Firebase Console) ou minimize a janela | Chrome exibe; clique → service worker foca a aba e abre o evento |
| Fechado | **Feche a aba do app**, mantendo o Chrome aberto | Chrome exibe; clique → service worker abre `/?evento_id=3` |
| Primeiro plano | Aba do app **visível** na tela (deixe o console em outra janela, lado a lado) | `onMessage` → aviso dentro do app |

---

## 5. Estrutura do código
```
lib/
├── main.dart                          # Firebase.initializeApp, handler de segundo plano, MaterialApp
├── firebase_config_web.dart           # Chaves do app Web + VAPID (só para o navegador)
├── models/evento.dart                 # Modelo Evento
├── data/eventos_data.dart             # 5 eventos definidos no código + buscarEventoPorId()
├── services/notification_service.dart # Permissão, token, onMessage, onMessageOpenedApp, getInitialMessage, navegação
├── services/ponte_web.dart            # Web: recebe o clique vindo do service worker (stub no Android)
├── screens/home_screen.dart           # Agenda + painel de notificações (token e histórico)
├── screens/detalhe_evento_screen.dart # Detalhes do evento (destino do Desafio 3)
└── widgets/evento_card.dart
web/firebase-messaging-sw.js           # Service worker: exibe em segundo plano e trata o clique (Desafio 3 na web)
firebase.json / .firebaserc            # Publicação no Firebase Hosting
notificacoes/                          # Imagens públicas para o campo "Imagem da notificação"
assets/eventos/                        # As mesmas imagens, usadas dentro do app
```

## 6. Problemas comuns
| Sintoma | Solução |
|---|---|
| `Default FirebaseApp is not initialized` / erro do google-services | `google-services.json` fora de `android/app/` ou plugin `com.google.gms.google-services` não adicionado |
| `minSdkVersion 21 cannot be smaller than version 24` | Ajuste `minSdk = 24` em `android/app/build.gradle.kts` |
| Erro de versão no `pub get` | Atualize o Flutter (`flutter upgrade`) — os pacotes exigem Flutter 3.38+ |
| Notificação não chega | Emulador sem Google Play, permissão negada, app em "Forçar parada", ou token antigo (reinstalou o app → copie o novo token) |
| Imagem não aparece | URL não é pública/HTTPS, repositório privado, ou arquivo > 1 MB. Abra o link no navegador para testar. Expanda a notificação para ver a imagem grande |
| Toque não abre o evento | Chave escrita diferente de `evento_id` ou valor fora de 1–5 |
| Web: `Erro ao obter token` no painel | Chaves de `lib/firebase_config_web.dart` ou VAPID erradas, ou o `firebase-messaging-sw.js` não está na pasta `web/` |
| Web: nada aparece no Windows | Notificações do Chrome desativadas no Windows, "Não incomodar" ligado, ou a aba do app estava visível (aí aparece o aviso dentro do app) |
| Web: mudei o código e nada mudou no link | Rode de novo `flutter build web --release` e `firebase deploy --only hosting`; recarregue com Ctrl+Shift+R |
| Web: permissão bloqueada | Clique no cadeado da barra de endereço → Notificações → Permitir, e recarregue |

## 7. Tecnologias
Flutter · Dart · `firebase_core` · `firebase_messaging` · `web` (ponte com o service worker) · Firebase Cloud Messaging e Firebase Hosting (plano Spark). Sem backend, login ou banco de dados — os envios são manuais pelo Firebase Console.
