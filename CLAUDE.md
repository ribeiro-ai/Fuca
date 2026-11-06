# App Fuca — Contexto técnico

## Produto
- PWA (web app instalável) de controle de função em fraternidade sem fins lucrativos. Sem loja de apps.
- Custo alvo: R$ 0. Segurança obrigatória.
- Adicionar à tela inicial é necessário para push (obrigatório no iPhone, iOS 16.4+) → prever tela de orientação "como adicionar à tela inicial".

## Requisitos
- Roles: admin e membro. Máximo 120 usuários. Grupos: G1 e G2.
- Calendário de eventos pré-definido, com datas por grupo; só o admin altera a agenda.
- Aviso ao usuário no dia do evento.
- Mensalidade: o app só registra status; pagamento ocorre fora.
  - Fluxo: pendente → informado (membro) → validado (só admin); atrasado por vencimento.
- Primeira tela: login; opções exibidas conforme a permissão.
- Avisos com prioridade (ex.: pagamento ausente primeiro, depois função na data). Regras exatas: etapa de modelagem.
- Notificações: in-app + push.
- Acessos toda terça e quinta (mitigação da pausa do Supabase Free).

## Decisões
- Backend: Supabase Free (Auth, Postgres, RLS, Realtime, Edge Functions, pg_cron + pg_net).
- Front: Cloudflare Pages (Vercel Hobby descartado: uso não comercial). Workers da Cloudflare: não necessário.
- Sem gateway de pagamento; v1 sem comprovante.
- Autorização via RLS no banco (vale mesmo se alguém abrir tela indevida).
- Jobs agendados rodam no Supabase em UTC (BRT = UTC-3):
  - pg_cron diário marca "atrasado";
  - pg_cron + Edge Function geram avisos do dia do evento por grupo.
- Avisos in-app: Realtime. Push: Web Push (VAPID) via Edge Function. Feed in-app como redundância.
- Prioridade dos avisos: campo no aviso; o app só exibe na ordem.

## Segurança
- Chave secreta (service role) do Supabase nunca no front nem no repositório.
- Free não inclui: proteção contra senha vazada, timeout de sessão, sessão única por usuário.

## Limites (Supabase Free)
- 500 MB banco, 50.000 MAU, 1 GB storage, 2 projetos ativos.
- Edge Functions: 500.000 invocações/mês.
- Realtime: 200 conexões simultâneas, 2M mensagens/mês.
- Pausa após 7 dias de baixa atividade (uso regular normalmente evita, sem garantia).
- Sem backups automáticos (Pro: diário, 7 dias, $25/mês).
- Sem preview branches (só Pro): o que for enviado vai direto ao app real.

## Limites (Cloudflare)
- Pages Free: requisições estáticas e banda ilimitadas, 500 builds/mês. Continua suportado; novidades só em Workers (migração possível por configuração).
- Pages: integração GitHub ou upload direto; a escolha não pode ser trocada depois no mesmo projeto.
- Workers Free (se usado): 100.000 req/dia, 10 ms CPU/req.

## Push
- iOS: exige iOS 16.4+ e app na tela inicial; há relatos de inscrições que deixam de funcionar sem aviso.

## Pendências de validação
Responder "Preciso de documentação ou validação externa para confirmar essa viabilidade." até validar:
- RLS impor sozinha que o membro só altere o status para "informado" (alternativa: Edge Function conferindo o papel).
- Job pg_cron interno contar como atividade contra a pausa do Supabase.
- Comportamento do Workers Free ao exceder o limite.
- Cron Triggers no Workers Free.
- Regras de acesso por usuário no Storage Free (só se comprovante entrar em versão futura).
- Custos e requisitos de publicação em loja (fora do plano atual).

## Em aberto (etapas futuras, não são dúvidas técnicas)
- Publicação: GitHub automático (alterar → GitHub → Cloudflare publica telas; Supabase aplica migrações e Edge Functions) ou upload manual na v1. GitHub Free: repos privados ilimitados, 2.000 min/mês de automação.
- Ferramenta/framework do front (define se SSR/Workers passam a ser necessários).
- Regras de prioridade dos avisos.
- Uso no portfólio (objetivo principal ou secundário; anonimizar contexto, dados fictícios, autorização da organização).

## Próxima etapa
Modelagem do banco: tabelas, roles e políticas RLS. Aguarda confirmação do usuário; ele pediu compreensão antes de execução.
