# instagram-web.js

Cliente não oficial do Instagram Web para Node.js, inspirado na experiência de uso do
[`whatsapp-web.js`](https://github.com/wwebjs/whatsapp-web.js). Permite gerir mensagens,
publicar Posts e Reels e usar o agendamento nativo do Instagram através de um browser controlado
por Puppeteer — sem usar a API da Meta.

> [!WARNING]
> Este projeto controla o Instagram Web e não é oficial nem afiliado à Meta ou ao Instagram.
> Alterações no site podem quebrar funcionalidades, e a automação pode causar limitações na
> conta. Usa apenas contas que controlas e não uses para spam.

## Principais funcionalidades

- Login manual na página oficial do Instagram, incluindo 2FA e verificações.
- Sessão persistente com `LocalAuth`; nunca guarda a palavra-passe no código.
- Envio e receção de mensagens de texto.
- API semelhante ao `whatsapp-web.js`, com `Client`, `Chat`, `Message` e eventos.
- Publicação imediata de imagens, vídeos, carrosséis e Reels.
- Agendamento nativo de Posts e Reels em contas profissionais.
- API HTTP local pronta a usar com `curl`, PowerShell ou qualquer aplicação.

## Instalação

Requer [Node.js](https://nodejs.org/) 22.12 ou superior.

### Clonar e executar o servidor

```bash
git clone https://github.com/atlanticsupport/instagram-web.js.git
cd instagram-web.js
npm install
npm start
```

O servidor fica disponível em `http://127.0.0.1:3000`.

### Instalar como dependência Git

```bash
npm install github:atlanticsupport/instagram-web.js
```

Depois importa o cliente com:

```js
const { Client, LocalAuth } = require('instagram-web.js');
```

## Quick start — API HTTP

Mantém `npm start` aberto e executa os exemplos seguintes noutro terminal.

### 1. Fazer login

```bash
curl -X POST http://127.0.0.1:3000/auth/login
```

Na primeira utilização abre uma janela com o login oficial. Depois de autenticar, a janela fecha
e o Chromium continua invisível. Nas execuções seguintes, a sessão guardada é reutilizada.

Confirmar o estado:

```bash
curl http://127.0.0.1:3000/auth/status
```

Resposta:

```json
{
  "status": "authenticated",
  "accountId": "123456789"
}
```

### 2. Enviar uma mensagem

`target` pode ser um nome de utilizador ou o ID numérico de uma conversa.

```bash
curl -X POST http://127.0.0.1:3000/messages \
  -H "Content-Type: application/json" \
  -d '{"target":"nome_do_utilizador","content":"Olá!"}'
```

### 3. Publicar um Post agora

O caminho em `media` é local ao computador onde o servidor está a correr.

```bash
curl -X POST http://127.0.0.1:3000/posts \
  -H "Content-Type: application/json" \
  -d '{"media":"C:/media/foto.jpg","caption":"A minha publicação"}'
```

Para publicar um carrossel, envia um array:

```json
{
  "media": ["C:/media/1.jpg", "C:/media/2.jpg"],
  "caption": "Carrossel de exemplo"
}
```

### 4. Agendar um Reel

`publishAt` deve ser uma data ISO 8601 futura com fuso horário explícito. O agendamento nativo
requer uma conta profissional do tipo Criador ou Empresa.

```bash
curl -X POST http://127.0.0.1:3000/reels \
  -H "Content-Type: application/json" \
  -d '{"media":"C:/media/reel.mp4","caption":"Novo Reel","publishAt":"2026-08-18T17:00:00+01:00"}'
```

Sem `publishAt`, o Post ou Reel é publicado imediatamente.

### 5. Terminar sessão

```bash
curl -X POST http://127.0.0.1:3000/auth/logout
```

O logout encerra o browser e apaga o perfil local autenticado. O login seguinte volta a pedir as
credenciais na página oficial.

## Example usage — biblioteca Node.js

```js
const { Client, LocalAuth } = require('instagram-web.js');

async function main() {
    const client = new Client({
        authStrategy: new LocalAuth({ clientId: 'conta-principal' }),
    });

    client.on('login', () => console.log('Conclui o login na janela do Instagram'));
    client.on('ready', () => console.log('Cliente pronto'));
    client.on('message', async (message) => {
        console.log(message.from, message.body);
        if (message.body === '!ping') await message.reply('pong');
    });

    await client.initialize();
}

main().catch(console.error);
```

### Conversas e mensagens

```js
const chats = await client.getChats({ limit: 20 });
const messages = await chats[0].fetchMessages({ limit: 20 });

await chats[0].sendMessage('Olá pela conversa');
await client.sendMessage('nome_do_utilizador', 'Olá pelo username');
await client.sendMessage('ID_NUMERICO_DA_CONVERSA', 'Olá pelo ID');
await messages[0].reply('Resposta à mensagem');
```

### Publicar conteúdo

```js
await client.publishPost('./media/foto.jpg', {
    caption: 'Legenda da publicação',
});

await client.publishPost('./media/reel.mp4', {
    caption: 'Reel publicado agora',
});

await client.schedulePost('./media/reel.mp4', {
    caption: 'Reel agendado',
    publishAt: '2026-08-18T17:00:00+01:00',
});
```

Formatos aceites: AVIF, JPG/JPEG, PNG, HEIC, HEIF, MP4 e MOV. A legenda deve ter no máximo
2200 caracteres. O agendamento deve estar no futuro e pode ter no máximo 75 dias de antecedência.

## Supported features

| Funcionalidade | Estado | Notas |
| --- | :---: | --- |
| Login oficial, 2FA e verificações | ✅ | Feito diretamente na janela do Instagram |
| Sessão persistente (`LocalAuth`) | ✅ | Um perfil local por `clientId` |
| Login e logout por HTTP | ✅ | `/auth/login` e `/auth/logout` |
| Listar conversas | ✅ | `getChats()` |
| Ler mensagens de texto | ✅ | Polling configurável |
| Enviar mensagens de texto | ✅ | Por username ou ID da conversa |
| Responder a mensagens | ✅ | `Message.reply()` |
| Publicar imagem no feed | ✅ | AVIF, JPG, PNG, HEIC e HEIF |
| Publicar vídeo/Reel | ✅ | MP4 ou MOV |
| Publicar carrossel | ⚠️ | Depende do seletor múltiplo exposto pelo Instagram Web |
| Agendar Post nativamente | ✅ | Requer conta profissional |
| Agendar Reel nativamente | ✅ | Requer conta profissional |
| Várias contas | ⚠️ | Usa um `Client` e `clientId` diferente por conta |
| Receber anexos de mensagens | ❌ | Ainda não implementado |
| Enviar anexos por mensagem | ❌ | Ainda não implementado |
| Stories | ❌ | Ainda não implementado |
| Comentários, gostos e seguidores | ❌ | Ainda não implementado |
| Reações e chamadas | ❌ | Ainda não implementado |

Legenda: ✅ suportado · ⚠️ suporte parcial ou dependente da interface · ❌ não suportado.

## API HTTP

| Método | Endpoint | Corpo | Descrição |
| --- | --- | --- | --- |
| `GET` | `/health` | — | Estado geral, autenticação e ID da conta |
| `GET` | `/auth/status` | — | `unauthenticated`, `authenticating` ou `authenticated` |
| `POST` | `/auth/login` | — | Inicia ou reutiliza a sessão do Instagram |
| `POST` | `/auth/logout` | — | Termina e apaga a sessão local |
| `POST` | `/messages` | `{ target, content }` | Envia uma mensagem de texto |
| `POST` | `/posts` | `{ media, caption, publishAt? }` | Publica ou agenda um Post |
| `POST` | `/reels` | `{ media, caption, publishAt? }` | Publica ou agenda um Reel MP4/MOV |

A API aceita JSON até 100 kB e responde com erros em JSON:

```json
{
  "error": "Instagram is not authenticated"
}
```

### Configuração do servidor

| Variável | Predefinição | Descrição |
| --- | --- | --- |
| `INSTAGRAM_API_PORT` | `3000` | Porta HTTP entre 1 e 65535 |
| `INSTAGRAM_API_KEY` | vazio | Protege todos os endpoints com Bearer token |

Com uma chave configurada, envia o cabeçalho em todos os pedidos:

```bash
curl http://127.0.0.1:3000/health \
  -H "Authorization: Bearer A_TUA_CHAVE"
```

Por segurança, o servidor escuta apenas em `127.0.0.1`.

## Eventos

| Evento | Quando é emitido |
| --- | --- |
| `login` | É necessário concluir o login na janela |
| `authenticated` | A sessão foi reconhecida |
| `auth_failure` | O login falhou ou expirou |
| `ready` | Inbox carregada e cliente pronto |
| `message` | Nova mensagem recebida |
| `message_create` | Mensagem recebida ou enviada |
| `post_published` | Publicação confirmada pelo Instagram |
| `post_scheduled` | Agendamento confirmado pelo Instagram |
| `post_error` | Publicação ou agendamento falhou |
| `poll_error` | A consulta periódica de mensagens falhou |
| `disconnected` | Browser fechado ou logout efetuado |

## Correspondência com whatsapp-web.js

| whatsapp-web.js | instagram-web.js |
| --- | --- |
| `Client` | `Client` |
| `LocalAuth` | `LocalAuth` |
| `client.initialize()` | `client.initialize()` |
| Evento `ready` | Evento `ready` |
| Evento `message` | Evento `message` |
| `client.sendMessage()` | `client.sendMessage()` |
| `Chat.sendMessage()` | `Chat.sendMessage()` |
| `Message.reply()` | `Message.reply()` |

## Como funciona

1. Puppeteer inicia um perfil Chromium persistente.
2. O utilizador autentica-se na página oficial do Instagram.
3. O browser visível fecha e reinicia em modo invisível.
4. Mensagens são consultadas pela sessão carregada no Instagram Web.
5. Envios e publicações usam a própria interface Web.
6. Agendamentos ficam guardados no Instagram e não dependem do processo Node continuar aberto.

Não partilhes, comprimas ou publiques `.instagram-web-auth/`: essa pasta contém a sessão do
browser e permite acesso à conta enquanto for válida. A pasta já está incluída no `.gitignore`.

## Testes

```bash
npm test
```

## Contribuir

Issues e pull requests são bem-vindos. Antes de implementar uma alteração grande, abre uma issue
para alinhar o comportamento esperado e inclui um teste pequeno para qualquer lógica nova.

## Disclaimer

Este projeto não é afiliado, associado, autorizado, aprovado ou oficialmente ligado à Meta ou ao
Instagram. “Instagram” e marcas relacionadas pertencem aos respetivos proprietários. Não existe
garantia de que a utilização desta integração não resulte em limitações, verificações ou bloqueio
da conta.

## Licença

[MIT](LICENSE)
