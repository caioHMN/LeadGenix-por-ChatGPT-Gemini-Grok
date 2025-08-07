# Documentação do Aplicativo Móvel LeadGenix (Android)

## 📝 Visão Geral

O aplicativo móvel LeadGenix para Android oferece aos vendedores B2B acesso direto à plataforma de inteligência de mercado, com foco em simplicidade, segurança e experiência fluida. O app utiliza um **WebView** para carregar a interface web (React/Next.js), mas implementa **autenticação nativa** para garantir integração transparente com o backend.

---

## 🎯 Objetivos

- **Simplicidade:** Minimizar desenvolvimento nativo, aproveitando a interface web já existente.
- **Autenticação Nativa:** Login seguro com Firebase Authentication (e-mail/senha ou provedores sociais).
- **Experiência do Usuário:** Interface minimalista, navegação tipo “abas de navegador”.
- **Escalabilidade:** Base para funcionalidades nativas futuras (notificações push, offline, navegação aprimorada).

---

## 🏗️ Estrutura do Aplicativo

### Arquitetura

- **WebView:** Carrega a URL principal (`https://app.leadgenix.com`), permitindo acesso ao painel de leads, filtros e relatórios.
- **Autenticação Nativa:** Integração com Firebase Authentication para login, armazenamento seguro de token JWT.
- **Navegação:** Atividade principal (`MainActivity`) exibe WebView, com barra de navegação opcional.
- **Cache:** Token armazenado localmente (SharedPreferences ou Android Keystore) para persistência da sessão.

### Estrutura de Projeto Sugerida

```
leadgenix-mobile-android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/leadgenix/mobile/
│   │   │   │   ├── MainActivity.java
│   │   │   │   └── utils/
│   │   │   │       └── AuthManager.java
│   │   │   ├── res/
│   │   │   │   ├── layout/
│   │   │   │   │   └── activity_main.xml
│   │   │   │   └── values/
│   │   │   │       └── strings.xml
│   │   └── test/
├── .github/
│   ├── workflows/
│   └── ISSUE_TEMPLATE/
├── build.gradle
├── README.md
├── LICENSE
└── .gitignore
```

---

## ⚙️ Requisitos Técnicos

- **Plataforma:** Android (API 21+)
- **Linguagem:** Java ou Kotlin (Kotlin recomendado)
- **Dependências:**  
  - Firebase Authentication  
  - WebView nativo  
  - OkHttp/Retrofit (opcional para integração API)
- **Ferramentas:**  
  - Android Studio  
  - Gradle  
  - GitHub Actions (CI/CD)

---

## 🔑 Fluxo de Autenticação

1. **Tela de Login**
    - Tela nativa (Firebase Auth: e-mail/senha ou Google Sign-In)
    - Validação de credenciais e obtenção de token JWT

2. **Armazenamento do Token**
    - Token salvo localmente (SharedPreferences ou Keystore)

3. **Carregamento do WebView**
    - WebView carrega a URL do painel, incluindo o token (`?token=JWT` ou header)
    - Backend valida token, autentica sessão

4. **Manutenção da Sessão**
    - App verifica validade do token e renova automaticamente se necessário
    - Token expirado? Usuário é redirecionado ao login

5. **Logout**
    - Limpa token local, recarrega tela de login

---

## 👩‍💻 Implementação Básica

### MainActivity (Kotlin)

```kotlin
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
            // Redireciona para tela de login
            startActivity(Intent(this, LoginActivity::class.java))
        }
    }

    private fun loadWebView() {
        webView.webViewClient = WebViewClient()
        webView.settings.javaScriptEnabled = true
        auth.currentUser?.getIdToken(false)?.addOnSuccessListener { result ->
            val token = result.token
            webView.loadUrl("https://app.leadgenix.com?token=$token")
        }
    }
}
```

### Layout (`activity_main.xml`)

```xml
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

---

## 🚀 Sugestões de Melhorias Futuras

- **Notificações Push:** Integrar Firebase Cloud Messaging (FCM) para alertas de novos leads.
- **Offline Support:** Armazenar dados essenciais (leads recentes) localmente, sincronizando quando possível.
- **Navegação Nativa:** Barra inferior de navegação (voltar, atualizar, logout).
- **Otimização do WebView:** Cache, tratamento de erros (offline), carregamento otimizado.
- **Conformidade LGPD:** Informar usuário sobre coleta/uso de dados; tokens em Keystore.
- **Testes Automatizados:** JUnit para autenticação, Espresso para UI, cobertura mínima.

---

## 📜 Licença

O projeto está sob a [MIT License](LICENSE).

---

## 📅 Próximos Passos

1. Criar repositório `leadgenix-mobile-android` no GitHub.
2. Configurar Firebase Authentication e integração backend.
3. Prototipar tela de login nativa e fluxo WebView.
4. Testar autenticação e navegação com ambiente de staging.
5. Planejar integração futura com FCM para notificações.

---

**LeadGenix Mobile App — Documentação oficial para MVP | 2025**
