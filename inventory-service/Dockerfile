# syntax=docker/dockerfile:1.10
# ==========================================
# Estágio 1: Base & Dependências
# ==========================================
FROM node:22-alpine AS base

# Definir diretório de trabalho padrão
WORKDIR /usr/src/app

# Instalar dependências apenas quando necessário (cache inteligente)
FROM base AS dependencies
COPY package*.json ./
# Em 2026, 'npm clean-install' garante builds limpos e determinísticos
RUN npm ci --only=production
# Salva uma cópia para o ambiente de desenvolvimento se necessário
RUN cp -R node_modules prod_node_modules
RUN npm ci

# ==========================================
# Estágio 2: Desenvolvimento (Local)
# ==========================================
FROM base AS development
COPY --from=dependencies /usr/src/app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "--watch", "index.js"] 
# Nota: O Node.js possui flag '--watch' nativa, dispensando nodemon.

# ==========================================
# Estágio 3: Produção (Segura e Otimizada)
# ==========================================
FROM base AS production
ENV NODE_ENV=production

# Copia apenas os módulos de produção e o código fonte limpo
COPY --from=dependencies /usr/src/app/prod_node_modules ./node_modules
COPY . .

# Boa prática crucial: Nunca rodar como Root em produção
USER node

EXPOSE 3000
CMD ["node", "index.js"]
