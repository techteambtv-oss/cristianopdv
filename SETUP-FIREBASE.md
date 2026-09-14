# Configurando o acompanhamento em tempo real (Firebase)

O HTML já vem preparado para sincronizar as OS e os dados da loja com o
Firestore (banco de dados gratuito do Google), para que o link de
acompanhamento funcione no **celular do cliente**, e não só no computador
da loja. Enquanto não configurar, o app funciona normalmente, só que a
página pública mostra "Acompanhamento on-line indisponível".

## 1. Criar o projeto (gratuito)

1. Acesse https://console.firebase.google.com e faça login com uma conta Google.
2. Clique em **"Adicionar projeto"**, dê um nome (ex: `cristiano-pdv`) e conclua a criação.
3. No menu lateral, vá em **Build > Firestore Database** → **"Criar banco de dados"**.
   - Escolha o modo **produção**.
   - Escolha a região mais próxima (ex: `southamerica-east1` para o Brasil).

## 2. Pegar as credenciais do app Web

1. No menu lateral, clique na engrenagem → **"Configurações do projeto"**.
2. Na aba **Geral**, role até "Seus apps" e clique no ícone **`</>`** (Web).
3. Dê um apelido ao app (ex: `painel-web`) e clique em **"Registrar app"**.
4. Copie o objeto `firebaseConfig` mostrado na tela.
5. Abra o arquivo `Cristiano_PDV.html`, procure por `const firebaseConfig = {`
   (logo depois da definição do `DB`) e substitua os valores de exemplo
   pelos que você copiou.

## 3. Configurar as regras de segurança do Firestore

No console do Firebase, vá em **Firestore Database > Regras** e cole:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Config da loja (nome, logo, whatsapp) - qualquer um pode ler,
    // mas só grava se o formato dos dados bater com o esperado.
    match /sistema/config {
      allow read: if true;
      allow write: if request.resource.data.keys().hasOnly(
        ['nomeSistema','logoSistema','whatsapp','telefone']
      );
    }

    // Ordens de serviço para acompanhamento - qualquer um com o link/código
    // pode ler o status; só grava se o formato dos dados bater.
    match /acompanhamentos/{codigo} {
      allow read: if true;
      allow write: if request.resource.data.keys().hasOnly(
        ['numero','status','clienteNome','equipamento','tipoEquip','marca',
         'modelo','tecnico','dataAbertura','hora','valorTotal','timeline','atualizadoEm']
      );
    }
  }
}
```

Clique em **"Publicar"**.

> ⚠️ **Sobre segurança:** como o app não tem um login próprio ligado ao
> Firebase (usa apenas um login local simples), essas regras liberam a
> **leitura pública** (necessário para o cliente ver o status) e a
> **escrita sem autenticação**, só validando o formato dos dados. Isso é
> aceitável para uma loja pequena, mas qualquer pessoa que descubra a URL
> do seu projeto Firebase tecnicamente poderia escrever dados nesses dois
> caminhos. Se isso for uma preocupação, o próximo passo recomendado é
> adicionar **Firebase Authentication** (login da loja) e restringir a
> escrita a usuários autenticados — posso te ajudar a implementar isso
> depois, se quiser.

## 4. Testar

1. Abra o `Cristiano_PDV.html` no navegador, faça login e crie/edite uma OS.
2. Clique em **"Ver página"** na tela da OS — deve abrir o painel de
   acompanhamento normalmente.
3. Copie o link (botão **"Copiar link"**) e abra em outro navegador, no
   modo anônimo, ou no seu celular — o status deve aparecer normalmente.
4. Volte na OS, mude o status e salve — a página aberta no outro
   dispositivo deve **atualizar sozinha**, sem precisar recarregar
   (isso é o "tempo real": ela fica ouvindo o Firestore).

## O que é sincronizado (e o que não é)

Por privacidade, só vai para a nuvem o necessário para a tela pública:
número da OS, status, nome do cliente, equipamento, técnico, datas,
valor total e o histórico de status. Dados sensíveis como **senha do
aparelho**, CPF, endereço e demais campos internos **não são enviados**
à nuvem — continuam só no computador da loja.
