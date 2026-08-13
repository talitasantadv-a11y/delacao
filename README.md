# Delação Jurídica — Sistema de Gestão do Time

Sistema web para gestão do time: tarefas, contas a pagar/receber, cadastro de
atletas/advogadas (com OAB) e repositório de documentos (regimento interno e
de competições).

## Stack

- [Next.js](https://nextjs.org) (App Router) + TypeScript + Tailwind CSS
- [Prisma](https://prisma.io) com banco **Postgres** (compatível com Vercel Postgres/Neon, Supabase, Railway etc.)
- [Auth.js / NextAuth](https://authjs.dev) com login por e-mail e senha, com papéis (ADMIN, FINANCEIRO, ATLETA)
- Upload de documentos: [Vercel Blob](https://vercel.com/docs/storage/vercel-blob) em produção, disco local em desenvolvimento

## Publicando na nuvem (Vercel) — passo a passo, só com cliques

Esse é o caminho recomendado: depois de pronto, todo o time acessa por um link,
sem precisar instalar nada no computador de ninguém.

1. **Suba este código para o GitHub** (se ainda não estiver lá).
2. Acesse **https://vercel.com**, entre com sua conta do GitHub e clique em
   **"Add New" → "Project"**. Selecione o repositório `delacao`.
3. Antes de clicar em "Deploy", adicione as variáveis de ambiente (aba
   "Environment Variables" na tela de configuração do projeto):
   - `AUTH_SECRET`: uma chave aleatória. Pode gerar uma em https://generate-secret.vercel.app/32
4. Clique em **"Deploy"**. O primeiro deploy provavelmente vai **falhar** — é
   esperado, porque falta o banco de dados. Sem problema, o próximo passo
   resolve isso.
5. Dentro do projeto na Vercel, vá na aba **"Storage"** → **"Create Database"**
   → escolha **"Postgres"** (Neon) → siga os passos e conecte ao projeto.
   Isso cria automaticamente a variável `DATABASE_URL` (ou similar) nas
   configurações do projeto.
   - Se a variável criada tiver outro nome (ex: `POSTGRES_PRISMA_URL`), vá em
     "Settings" → "Environment Variables" e crie uma variável `DATABASE_URL`
     com o mesmo valor.
6. (Opcional, para o upload de documentos funcionar) Na mesma aba
   **"Storage"**, clique em **"Create Database"** de novo → escolha
   **"Blob"** → conecte ao projeto. Isso cria a variável
   `BLOB_READ_WRITE_TOKEN` automaticamente.
7. Vá na aba **"Deployments"** e clique em **"Redeploy"** no último deploy
   (ou faça um novo commit). Dessa vez o build faz automaticamente: criar as
   tabelas no banco e cadastrar a usuária administradora inicial e as
   pendências combinadas.
8. Quando o deploy terminar, clique no link gerado (algo como
   `delacao.vercel.app`). Pronto — esse é o link que todo o time vai usar.

**Login inicial:**
- **E-mail:** talitasant.adv@gmail.com
- **Senha provisória:** `Delacao@2026`

**Troque essa senha assim que possível** (em "Atletas" → editar seu perfil → "Redefinir senha").

## Rodando localmente (opcional, para testar antes de publicar)

Requer um banco Postgres acessível (pode ser o mesmo da nuvem, copiando a
`DATABASE_URL` da Vercel, ou um Postgres local/gratuito de algum provedor).

```bash
npm install
cp .env.example .env   # edite DATABASE_URL e AUTH_SECRET
npm run build           # cria as tabelas, popula os dados iniciais e builda
npm run start            # ou: npm run dev, para modo desenvolvimento
```

Acesse http://localhost:3000.

## Papéis de acesso

- **ADMIN**: acesso total — cria/edita/remove integrantes, gerencia tarefas, financeiro e documentos.
- **FINANCEIRO**: acesso a tarefas, financeiro, atletas e documentos (sem remover integrantes).
- **ATLETA**: acesso a tarefas, ao próprio cadastro (atualiza atestado médico, tamanho de uniforme, contato) e a documentos. Sem acesso ao módulo financeiro.

## Módulos

- **Tarefas** (`/tarefas`): quadro de pendências (Pendente / Em andamento / Concluída), com prazo, categoria e responsável.
- **Financeiro** (`/financeiro`): contas a pagar e a receber, com status (pendente, pago, atrasado, cancelado) e totais.
- **Atletas** (`/atletas`): cadastro do time — nome, OAB/UF, CPF, contato, tamanho de uniforme e controle do atestado médico de aptidão física.
- **Documentos** (`/documentos`): upload e download de regimento interno do time e regulamentos de competições.
- **Painel** (`/dashboard`): resumo com próximas tarefas, vencimentos financeiros e pendências de atestado médico.

Os dados iniciais no `prisma/seed.ts` já refletem a lista de pendências combinada (uniformes, passagens dos técnicos, parcelas de hotel/inscrição, rifas, patrocínios etc.) — **os valores em R$ foram deixados como `0` e precisam ser preenchidos** em "Financeiro" com os valores reais.

## Comandos úteis

```bash
npm run dev             # ambiente de desenvolvimento
npm run build            # gera as tabelas do banco, popula os dados iniciais e builda para produção
npm run start              # roda o build de produção
npm run lint                 # checagem de lint
npm run db:push                # só sincroniza o schema do banco, sem popular dados
npm run db:seed                  # só popula/atualiza os dados iniciais
npx prisma studio                    # interface visual para ver/editar os dados do banco
```
