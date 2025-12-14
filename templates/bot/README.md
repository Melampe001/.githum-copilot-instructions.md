# Bot Template - Discord/Telegram/Slack Bots

## 🤖 Ejemplo de uso

```typescript
// PROYECTO: Bot de Discord para moderación y estadísticas
// Stack: Discord.js + TypeScript + MongoDB + Docker
```

## 📦 Stack Generado

- **Runtime**: Node.js 20.x
- **Lenguaje**: TypeScript 5.x
- **Bot Framework**: Discord.js / Telegraf / Slack Bolt
- **Database**: MongoDB / PostgreSQL
- **Cache**: Redis
- **Testing**: Jest + Supertest
- **Deployment**: Docker + Railway

## 📁 Estructura

```
bot-project/
├── src/
│   ├── commands/
│   │   ├── moderation/
│   │   ├── utility/
│   │   └── fun/
│   ├── events/
│   │   ├── ready.ts
│   │   ├── messageCreate.ts
│   │   └── interactionCreate.ts
│   ├── services/
│   │   ├── database.ts
│   │   └── logger.ts
│   ├── utils/
│   │   ├── permissions.ts
│   │   └── validators.ts
│   ├── types/
│   └── index.ts
├── tests/
│   ├── commands/
│   └── services/
├── config/
│   └── default.json
├── Dockerfile
├── docker-compose.yml
└── package.json
```

## 🎯 Características Incluidas

### Discord Bot Features
- Slash commands
- Button interactions
- Select menus
- Modal forms
- Embed messages
- Role management
- Auto-moderation
- Logging system

### Telegram Bot Features
- Command handlers
- Inline keyboards
- Callback queries
- Media handling
- Group management
- Admin commands

### Slack Bot Features
- Slash commands
- Interactive messages
- Scheduled messages
- File uploads
- Channel management

## 📝 Ejemplo: Discord Bot

### Main Bot File
```typescript
// src/index.ts
import { Client, GatewayIntentBits, Collection } from 'discord.js';
import { loadCommands } from './utils/commandLoader';
import { connectDatabase } from './services/database';
import { logger } from './services/logger';

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
  ],
});

client.commands = new Collection();

async function start() {
  try {
    await connectDatabase();
    await loadCommands(client);
    await client.login(process.env.DISCORD_TOKEN);
    logger.info('Bot started successfully');
  } catch (error) {
    logger.error('Failed to start bot:', error);
    process.exit(1);
  }
}

start();
```

### Command Example
```typescript
// src/commands/moderation/ban.ts
import { SlashCommandBuilder, PermissionFlagsBits } from 'discord.js';
import type { Command } from '@/types';

export const command: Command = {
  data: new SlashCommandBuilder()
    .setName('ban')
    .setDescription('Ban a user from the server')
    .addUserOption(option =>
      option
        .setName('user')
        .setDescription('User to ban')
        .setRequired(true)
    )
    .addStringOption(option =>
      option
        .setName('reason')
        .setDescription('Reason for ban')
    )
    .setDefaultMemberPermissions(PermissionFlagsBits.BanMembers),
  
  async execute(interaction) {
    const user = interaction.options.getUser('user', true);
    const reason = interaction.options.getString('reason') ?? 'No reason provided';
    
    try {
      await interaction.guild?.members.ban(user, { reason });
      await interaction.reply({
        content: `✅ Successfully banned ${user.tag}`,
        ephemeral: true,
      });
      
      // Log to database
      await logModeration({
        type: 'ban',
        userId: user.id,
        moderatorId: interaction.user.id,
        reason,
        timestamp: new Date(),
      });
    } catch (error) {
      await interaction.reply({
        content: '❌ Failed to ban user',
        ephemeral: true,
      });
    }
  },
};
```

### Event Handler
```typescript
// src/events/interactionCreate.ts
import { Events, Interaction } from 'discord.js';
import { logger } from '@/services/logger';

export default {
  name: Events.InteractionCreate,
  async execute(interaction: Interaction) {
    if (!interaction.isChatInputCommand()) return;

    const command = interaction.client.commands.get(interaction.commandName);

    if (!command) {
      logger.warn(`Command ${interaction.commandName} not found`);
      return;
    }

    try {
      await command.execute(interaction);
      logger.info(`Command ${interaction.commandName} executed by ${interaction.user.tag}`);
    } catch (error) {
      logger.error('Error executing command:', error);
      
      const reply = {
        content: '❌ An error occurred while executing this command.',
        ephemeral: true,
      };

      if (interaction.replied || interaction.deferred) {
        await interaction.followUp(reply);
      } else {
        await interaction.reply(reply);
      }
    }
  },
};
```

## 📦 Dependencies

### Discord Bot
```json
{
  "dependencies": {
    "discord.js": "^14.14.0",
    "typescript": "^5.0.0",
    "mongodb": "^6.2.0",
    "redis": "^4.6.0",
    "dotenv": "^16.3.0",
    "winston": "^3.11.0"
  },
  "devDependencies": {
    "@types/node": "^20.8.0",
    "jest": "^29.7.0",
    "ts-node": "^10.9.0",
    "nodemon": "^3.0.0"
  }
}
```

### Telegram Bot
```json
{
  "dependencies": {
    "telegraf": "^4.15.0",
    "typescript": "^5.0.0",
    "mongodb": "^6.2.0",
    "dotenv": "^16.3.0"
  }
}
```

## 🚀 Scripts

```json
{
  "scripts": {
    "dev": "nodemon --exec ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest",
    "lint": "eslint src/**/*.ts",
    "deploy:commands": "ts-node scripts/deploy-commands.ts"
  }
}
```

## 🔐 Environment Variables

```env
# .env.example
DISCORD_TOKEN=your_bot_token_here
DISCORD_CLIENT_ID=your_client_id_here
MONGODB_URI=mongodb://localhost:27017/bot
REDIS_URL=redis://localhost:6379
NODE_ENV=development
LOG_LEVEL=info
```

## 🧪 Tests

```typescript
// tests/commands/ping.test.ts
import { createMockInteraction } from '../mocks/interaction';
import { command as pingCommand } from '@/commands/utility/ping';

describe('Ping Command', () => {
  it('should respond with Pong!', async () => {
    const interaction = createMockInteraction();
    await pingCommand.execute(interaction);
    
    expect(interaction.reply).toHaveBeenCalledWith({
      content: expect.stringContaining('Pong!'),
      ephemeral: false,
    });
  });
});
```

## 🐳 Docker

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  bot:
    build: .
    restart: unless-stopped
    env_file: .env
    depends_on:
      - mongodb
      - redis
    networks:
      - bot-network

  mongodb:
    image: mongo:7
    volumes:
      - mongodb_data:/data/db
    networks:
      - bot-network

  redis:
    image: redis:7-alpine
    networks:
      - bot-network

volumes:
  mongodb_data:

networks:
  bot-network:
```

## 🚀 Deploy

### Railway
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login and deploy
railway login
railway init
railway up
```

### Docker
```bash
docker-compose up -d
```

## 📊 Database Schema

```typescript
// src/models/Guild.ts
interface GuildConfig {
  guildId: string;
  prefix: string;
  modLogChannel?: string;
  welcomeChannel?: string;
  autoModeration: {
    enabled: boolean;
    maxWarnings: number;
    filterWords: string[];
  };
  levels: {
    enabled: boolean;
    announceChannel?: string;
  };
}
```

## ✅ Checklist de Funcionalidades

- [x] Slash commands registrados
- [x] Event handlers configurados
- [x] Database connection
- [x] Error handling robusto
- [x] Logging system
- [x] Permission checks
- [x] Rate limiting
- [x] Tests unitarios
- [x] Docker ready
- [x] CI/CD pipeline
