# 🚀 Guia de Setup - Painel Kanban Cloud com Firebase

## 📋 Passo a Passo Completo

### **1️⃣ CRIAR PROJETO NO FIREBASE**

#### A. Acessar Firebase Console
1. Ir para [https://console.firebase.google.com](https://console.firebase.google.com)
2. Clique em **"Adicionar Projeto"**
3. Nome: `painel-kanban-pedidos`
4. Google Analytics: Ativar (opcional)
5. Clique em **"Criar Projeto"** e aguarde

#### B. Configurar Autenticação
1. No Firebase Console, clique em **Autenticação** (esquerda)
2. Clique em **"Começar"**
3. Vá para **"Método de login"**
4. Ative **"Email/Senha"**
   - Ativar "Criação de conta por Email/Senha"
5. Clique em **Salvar**

#### C. Criar Firestore Database
1. Clique em **Firestore Database** (esquerda)
2. Clique em **"Criar banco de dados"**
3. Modo: **Produção** (vamos adicionar regras)
4. Localização: **Mais próxima do seu país** (ex: `southamerica-east1` para Brasil)
5. Clique em **"Criar"**

#### D. Obter Configurações Firebase
1. Vá para **Configurações do Projeto** (engrenagem, canto superior direito)
2. Clique em **"Apps"**
3. Clique no ícone `</>`(Web)
4. Nome do app: `painel-kanban-web`
5. Clique em **"Registrar app"**
6. **COPIE A CONFIGURAÇÃO** que aparece (firebaseConfig)

### **2️⃣ CONFIGURAR REGRAS DE SEGURANÇA DO FIRESTORE**

#### A. Acessar Regras
1. No Firestore, clique em **"Regras"**
2. Limpe o conteúdo padrão
3. **COPIE E COLE** o código abaixo:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Coleção: usuários
    match /usuarios/{uid} {
      allow read, write: if request.auth.uid == uid;
    }
    
    // Coleção: pedidos (Firestore)
    match /pedidos/{pedidoId} {
      // Visualizadores: apenas leitura
      allow read: if request.auth != null;
      
      // Editores e Admins: criar, ler, atualizar
      allow create, update: if request.auth != null && 
        (get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role == 'editor' ||
         get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role == 'admin');
      
      // Apenas Admin pode deletar
      allow delete: if request.auth != null &&
        get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role == 'admin';
    }
    
    // Coleção: auditoria (log de ações)
    match /auditoria/{logId} {
      // Apenas autenticados podem ler
      allow read: if request.auth != null;
      
      // Qualquer autenticado pode criar (sistema registra automaticamente)
      allow create: if request.auth != null;
      
      // Proteger de modificação/deleção
      allow update, delete: if false;
    }
  }
}
```

4. Clique em **"Publicar"**

### **3️⃣ CRIAR ESTRUTURA INICIAL NO FIRESTORE**

#### A. Criar Documento Exemplo (Admin)
1. No Firestore, clique em **"+" ao lado de "Coleção"**
2. Nome da coleção: `usuarios`
3. ID do documento: seu UID (ou deixar automático)
4. Adicionar campos:
   - `email` (string): seu@email.com
   - `role` (string): `admin`
   - `criado_em` (timestamp): agora

#### B. Criar Coleção de Pedidos
1. Clique em **"+" ao lado de "Coleção"**
2. Nome: `pedidos`
3. Deixe vazio por enquanto (vamos popular importando arquivo)

#### C. Criar Coleção de Auditoria
1. Clique em **"+" ao lado de "Coleção"**
2. Nome: `auditoria`
3. Deixe vazio

### **4️⃣ BAIXAR E CONFIGURAR O ARQUIVO HTML**

#### A. Copiar Código HTML
- Peça o arquivo `index-cloud.html` completo

#### B. Substituir Configurações Firebase
No arquivo HTML, procure por:
```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "seu-projeto.firebaseapp.com",
  projectId: "seu-projeto",
  storageBucket: "seu-projeto.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc"
};
```

**SUBSTITUA COM** as configurações que você copiou do Firebase Console

### **5️⃣ FAZER UPLOAD PARA INTERNET**

Escolha uma das opções:

#### **OPÇÃO A: GitHub Pages (Gratuito)**
1. Criar repositório GitHub `painel-kanban`
2. Upload arquivo `index-cloud.html` como `index.html`
3. Settings → Pages → Branch: main
4. URL: `https://seu-usuario.github.io/painel-kanban`

#### **OPÇÃO B: Vercel (Recomendado - Gratuito)**
1. Ir para [https://vercel.com](https://vercel.com)
2. Sign up com GitHub
3. "New Project" → seu repositório
4. Deploy automático

#### **OPÇÃO C: Firebase Hosting (Integrado)**
1. Terminal: `npm install -g firebase-tools`
2. `firebase login`
3. `firebase init hosting`
4. Upload arquivo para pasta `public/`
5. `firebase deploy`

### **6️⃣ CRIAR PRIMEIRO USUÁRIO**

1. Abrir o painel em seu computador/internet
2. Clique em **"Criar Conta"**
3. Email: seu@email.com
4. Senha: senha_forte_123
5. Tipo de acesso: **Admin** (para você)
6. Clique em **"Criar"**

**OBS:** A primeira conta será "editor". Para tornar **admin**:
- No Firebase Console → Firestore
- Collection `usuarios` → seu documento
- Editar campo `role` para `admin`

### **7️⃣ CONVIDAR OUTROS USUÁRIOS**

Cada pessoa que quer usar deve:
1. Acessar o link do painel
2. Clicar em **"Criar Conta"**
3. Email e senha
4. Escolher acesso:
   - **Editor**: pode mover pedidos, importar arquivos
   - **Visualizador**: apenas lê os dados (sem editar)

**Você (admin)** depois pode editar a role de cada um no Firebase

### **8️⃣ FORMATO DO ARQUIVO PARA IMPORTAR**

CSV ou XLSX com colunas:
```
pedido,nome,rep,tipo,emissao,valor
001,Empresa A,João,Venda Direta,2024-06-01,5000
002,Empresa B,Maria,Consultoria,2024-06-05,8000
003,Empresa C,Pedro,Manutenção,2024-06-10,3000
```

**Mapeamento automático:**
- `pedido` ← numero, nº
- `nome` ← cliente
- `rep` ← vendedor
- `tipo` ← desc, descTipo
- `emissao` ← data
- `valor` ← total

### **9️⃣ CHECKLIST FINAL**

- [ ] Projeto Firebase criado
- [ ] Autenticação Email/Senha ativada
- [ ] Firestore Database criado
- [ ] Regras de segurança publicadas
- [ ] Coleções criadas (usuarios, pedidos, auditoria)
- [ ] Configurações Firebase copiadas no HTML
- [ ] Arquivo publicado online
- [ ] Primeiro usuário (admin) criado
- [ ] Arquivo de pedidos preparado (CSV/XLSX)
- [ ] Importação testada
- [ ] Outros usuários convidados

---

## 🔑 Papéis e Permissões

| Ação | Admin | Editor | Visualizador |
|------|-------|--------|--------------|
| Ver pedidos | ✅ | ✅ | ✅ |
| Mover pedidos | ✅ | ✅ | ❌ |
| Importar arquivo | ✅ | ✅ | ❌ |
| Deletar pedidos | ✅ | ❌ | ❌ |
| Limpar banco | ✅ | ❌ | ❌ |
| Ver auditoria | ✅ | ✅ | ✅ |

---

## ⚠️ TROUBLESHOOTING

### "Erro de autenticação"
- Verificar se Email/Senha está ativado no Firebase
- Limpar cache do navegador

### "Não consegue salvar dados"
- Verificar regras de segurança do Firestore
- Verificar se usuário tem `role` correto

### "Dados não sincronizam"
- Verificar conexão de internet
- Verificar console do navegador (F12) para erros
- Limpar localStorage: `localStorage.clear()`

### "Arquivo não importa"
- Verificar formato CSV/XLSX
- Nomes de colunas exatos (pedido, nome, rep, tipo, emissao, valor)
- Verificar se usuário é Editor ou Admin

---

## 📞 SUPORTE

**Erro recorrente?**
1. Abra o Console do Navegador (F12)
2. Aba **Console**
3. Copie a mensagem de erro
4. Verifique se:
   - Firebase Config está correto
   - Firestore Rules estão publicadas
   - Coleções existem no Firebase

**Documentação oficial:**
- [Firebase Console](https://console.firebase.google.com)
- [Firestore Rules](https://firebase.google.com/docs/firestore/security/start)
- [Firebase Auth](https://firebase.google.com/docs/auth/web)

---

**Pronto! Seu painel compartilhado está online!** 🎉
