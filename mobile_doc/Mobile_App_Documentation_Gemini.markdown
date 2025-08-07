# Documentação do Aplicativo Móvel LeadGenix (Android)

## Visão Geral
O aplicativo móvel LeadGenix para Android é uma interface leve que permite aos usuários (vendedores B2B) acessar a plataforma de inteligência de mercado diretamente de seus dispositivos móveis. Para manter a simplicidade, o app utiliza um **WebView** para carregar a interface web da plataforma (hospedada em React/Next.js), mas implementa **autenticação nativa** para oferecer uma experiência fluida e segura. O foco é replicar a funcionalidade do painel web (visualização de leads, filtros, notificações) com integração nativa para login e gerenciamento de sessões.

## Objetivos
- **Simplicidade**: Redirecionar para a interface web existente, minimizando o desenvolvimento nativo.
- **Autenticação Nativa**: Usar o sistema de autenticação do Android (ex.: integração com Firebase Authentication) para login seguro.
- **Experiência de Usuário**: Interface minimalista, semelhante a "abas de navegador", com navegação intuitiva.
- **Escalabilidade**: Base para futuras funcionalidades nativas (ex.: notificações push, offline).

## Estrutura do Aplicativo

### Arquitetura
- **WebView**: Carrega a URL da plataforma LeadGenix (ex.: `https://app.leadgenix.com`) para exibir o painel de leads, filtros e relatórios.
- **Autenticação Nativa**: Integração com Firebase Authentication para login via e-mail/senha ou provedores sociais (ex.: Google).
- **Navegação**: Interface com uma única atividade principal contendo o WebView e uma barra de navegação básica (opcional: botões de voltar, atualizar).
- **Cache**: Armazenamento local de tokens de autenticação para manter a sessão do usuário.

### Estrutura do Projeto
O projeto será hospedado em um repositório GitHub chamado `leadgenix-mobile-android` com a seguinte estrutura:

```
leadgenix-mobile-android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/leadgenix/mobile/
│   │   │   │   ├── MainActivity.java       # Atividade principal (WebView + Auth)
│   │   │   │   └── utils/
│   │   │   │       └── AuthManager.java    # Gerenciamento de autenticação
│   │   │   ├── res/
│   │   │   │   ├── layout/
│   │   │   │   │   └── activity_main.xml   # Layout do WebView
│   │   │   │   └── values/
│   │   │   │       └── strings.xml         # Strings (URL, mensagens)
│   │   └── test/                           # Testes unitários
├── .github/
│   ├── workflows/                          # Pipelines CI/CD
│   └── ISSUE_TEMPLATE/                     # Modelos de issues
├── build.gradle                            # Configuração do Gradle
├── README.md                               # Documentação do repositório
├── LICENSE                                 # Licença MIT
└── .gitignore
```

## Requisitos Técnicos
- **Plataforma**: Android (API 21+ para compatibilidade ampla).
- **Linguagem**: Java ou Kotlin (Kotlin recomendado para modernidade).
- **Dependências**:
  - Firebase Authentication (autenticação nativa).
  - WebView (nativo do Android).
  - OkHttp/Retrofit (opcional, para chamadas à API de autenticação).
- **Ferramentas**:
  - Android Studio para desenvolvimento.
  - Gradle para gerenciamento de dependências.
  - GitHub Actions para CI/CD (build, testes, deploy na Play Store).

## Fluxo de Autenticação
1. **Tela de Login**:
   - O usuário abre o app e é apresentado a uma tela nativa de login (e-mail/senha ou Google Sign-In).
   - O Firebase Authentication valida as credenciais e retorna um token JWT.
2. **Armazenamento do Token**:
   - O token é salvo localmente (ex.: SharedPreferences ou Room) para manter a sessão.
3. **Carregamento do WebView**:
   - Após o login, o WebView carrega a URL da plataforma (`https://app.leadgenix.com`) com o token JWT incluído no cabeçalho da requisição ou como parâmetro (ex.: `?token=JWT`).
   - A API Gateway (`leadgenix-api`) valida o token e autoriza o acesso.
4. **Manutenção da Sessão**:
   - O app verifica periodicamente a validade do token e renova se necessário.
   - Caso o token expire, o usuário é redirecionado para a tela de login.
5. **Logout**:
   - Opção nativa para logout, que limpa o token local e recarrega a tela de login.

## Implementação Básica
### Exemplo de Código (Kotlin, MainActivity)
```kotlin
package com.leadgenix.mobile

import android.os.Bundle
import android.webkit.WebView
import android.webkit.WebViewClient
import androidx.appcompat.app.AppCompatActivity
import com.google.firebase.auth.FirebaseAuth

class MainActivity : AppCompatActivity() {
    private lateinit var webView: WebView
    private lateinit var auth: FirebaseAuth

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        auth = FirebaseAuth.getInstance()
        webView = findViewById(R.id.webview)

        // Verifica autenticação
        if (auth.currentUser != null) {
            loadWebView()
        } else {
            // Redireciona para tela de login (implementada separadamente)
            startActivity(LoginActivity::class.java)
        }
    }

    private fun loadWebView() {
        webView.webViewClient = WebViewClient()
        webView.settings.javaScriptEnabled = true
        val token = auth.currentUser?.getIdToken(false)?.result?.token
        webView.loadUrl("https://app.leadgenix.com?token=$token")
    }
}
```

### Layout (activity_main.xml)
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <WebView
        android:id="@+id/webview"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## Sugestões para Melhorias
1. **Notificações Push**:
   - Integrar Firebase Cloud Messaging (FCM) para enviar alertas de novos leads diretamente no app.
   - Exemplo: Notificar o vendedor quando uma empresa em expansão é identificada no seu nicho.
2. **Offline Support**:
   - Armazenar leads recentes localmente (usando Room) para acesso offline.
   - Sincronizar dados quando o dispositivo reconectar.
3. **Navegação Nativa**:
   - Adicionar uma barra de navegação nativa com botões (ex.: voltar, atualizar, logout) para melhorar a usabilidade.
4. **Otimização do WebView**:
   - Configurar cache para carregar a interface mais rapidamente.
   - Implementar tratamento de erros (ex.: tela de "sem conexão").
5. **Conformidade com LGPD**:
   - Garantir que o app informe o usuário sobre o uso de dados (ex.: pop-up com política de privacidade).
   - Armazenar tokens de forma segura (ex.: Android Keystore).
6. **Testes**:
   - Configurar testes unitários (JUnit) para autenticação.
   - Testes de UI (Espresso) para validar o fluxo de login e WebView.

## Licença
O aplicativo está licenciado sob a [MIT License](LICENSE), permitindo contribuições externas enquanto mantém flexibilidade para uso comercial.

## Próximos Passos
1. Criar repositório `leadgenix-mobile-android` no GitHub.
2. Configurar Firebase Authentication e integrar com a API Gateway.
3. Prototipar tela de login nativa e WebView.
4. Testar o fluxo de autenticação com um servidor de staging.
5. Planejar integração com FCM para notificações push.