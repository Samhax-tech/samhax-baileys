# Samhax Baileys

Samhax Baileys is an enhanced version of the popular Baileys WhatsApp API library with improved stability, additional features, and integration with the Samhax libsignal-node library.

## Features

- **Enhanced Stability**: Improved connection handling with better error recovery and session management
- **Samhax libsignal-node Integration**: Uses the robust @whiskeysockets/libsignal-node for secure messaging
- **Advanced Session Management**: Better session reset and verification capabilities
- **Improved PreKey Upload**: Enhanced pre-key management with automatic retries and backoff strategies
- **Better Error Handling**: More comprehensive error handling throughout the library
- **LID Support**: Full support for Legacy ID handling

## Installation

```bash
npm install @samhax-tech/baileys @whiskeysockets/libsignal-node
```

## Usage

### Basic Usage

```javascript
const { makeWASocket, DisconnectReason } = require('@samhax-tech/baileys')
const { Boom } = require('@hapi/boom')

async function connectToWhatsApp() {
    // Load auth state if available
    const { state, saveState } = await useMultiFileAuthState('auth_info_baileys')
    
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    })
    
    sock.ev.process(async (events) => {
        // Connection updates
        if(events['connection.update']) {
            const update = events['connection.update']
            const { connection, lastDisconnect } = update
            
            if(connection === 'close') {
                if(lastDisconnect?.error?.output?.statusCode !== DisconnectReason.loggedOut) {
                    // Reconnect after a delay
                    setTimeout(() => {
                        connectToWhatsApp()
                    }, 5000)
                } else {
                    console.log('Connection logged out')
                }
            }
            
            console.log('Connection Update:', update)
        }
        
        // Authentication updates
        if(events['creds.update']) {
            await saveState()
        }
        
        // Received messages
        if(events['messages.upsert']) {
            const { messages } = events['messages.upsert']
            console.log('Messages received:', messages)
            
            // Process messages here
            for(const message of messages) {
                if(!message.key.fromMe && message.message) {
                    const text = message.message.conversation
                    
                    // Echo the message back
                    await sock.sendMessage(message.key.remoteJid, { 
                        text: `Echo: ${text}` 
                    }, { quoted: message })
                }
            }
        }
    })
}

connectToWhatsApp()
```

### Advanced Features

#### Using Connection Stability Features

The library includes enhanced connection stability features accessible through the `connectionStability` object:

```javascript
// Check if session is established
const isEstablished = await sock.connectionStability.isSessionEstablished(jid)

// Reset session for a specific JID
await sock.connectionStability.resetSession(jid)

// Verify identity
const isValid = await sock.connectionStability.verifyIdentity(jid, identity)

// Get local registration ID
const regId = await sock.connectionStability.getLocalRegistrationId()
```

#### Using Enhanced Signal Repository

The library provides an enhanced signal repository with additional methods:

```javascript
// Access the enhanced signal repository
const { signalRepository } = sock

// Verify identity
const isValid = await signalRepository.verifyIdentity({ jid, identity })

// Process pre-key bundle
const result = await signalRepository.processPreKeyBundle({ jid, bundle })

// Check session establishment
const isEstablished = await signalRepository.isSessionEstablished({ jid })
```

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `waWebSocketUrl` | string | WhatsApp Web socket URL |
| `connectTimeoutMs` | number | Connection timeout in milliseconds |
| `logger` | Logger | Pino logger instance |
| `keepAliveIntervalMs` | number | Keep alive interval in milliseconds |
| `browser` | BrowserConfig | Browser configuration |
| `auth` | AuthState | Authentication state |
| `printQRInTerminal` | boolean | Whether to print QR code in terminal |
| `defaultQueryTimeoutMs` | number | Default query timeout |

## Differences from Original Baileys

1. **Enhanced libsignal Integration**: Uses Samhax's @whiskeysockets/libsignal-node instead of the deprecated libsignal package
2. **Better Session Management**: Improved session handling with reset and verification capabilities
3. **Stability Improvements**: More robust connection handling with better error recovery
4. **Additional Methods**: Extra utility methods for connection stability and debugging
5. **Improved PreKey Handling**: Better pre-key management with automatic retries

## Contributing

Feel free to submit issues and enhancement requests via GitHub Issues. Pull requests are welcome!

## License

MIT