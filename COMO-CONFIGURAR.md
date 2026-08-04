# Como ficou organizado

- **`index.html`** → site público (visitantes/clientes). Só lê dados do Supabase, não edita nada.
- **`admin.html`** → painel administrativo, separado do site. É onde você faz login e edita tudo (textos, serviços, galeria, depoimentos, fotos).
- Os dois continuam usando o **mesmo banco Supabase** — então tudo que você salvar no admin aparece automaticamente no site, para todo mundo, sem precisar reenviar arquivo nenhum.
- O site público não mostra mais nenhum link ou ícone de admin — acesse o painel direto digitando `seusite.com/admin` (ou `admin.html`).

## ⚠️ Passo obrigatório antes de usar: criar seu login e travar a edição

Hoje a edição usava uma "senha" guardada só no seu navegador — qualquer pessoa que abrisse o código do site conseguiria editar os dados, mesmo sem essa senha. Troquei o login do admin para um **login de verdade (e-mail + senha) via Supabase Auth**, mas isso só funciona depois de 2 ajustes rápidos no painel do Supabase (gratuito, 5 minutos):

### 1. Criar seu usuário admin
Você pediu para logar só com **usuário + senha** (sem e-mail). O painel faz isso na tela, mas por baixo dos panos o Supabase Auth sempre precisa de um e-mail — então o sistema converte automaticamente o usuário digitado em um e-mail fixo, sem você precisar ver ou digitar isso.

1. Entre em [supabase.com](https://supabase.com/dashboard) → seu projeto.
2. Menu **Authentication → Users → Add user**.
3. No campo e-mail, cadastre exatamente: `admin@duya-admin.local`
4. Escolha a senha que você vai usar para logar.
5. Marque "Auto Confirm User".

No painel (`/admin`), você então loga digitando **usuário: `admin`** e a senha escolhida — o site completa o `@duya-admin.local` sozinho.

Se quiser trocar o nome de usuário (ex: `duya` em vez de `admin`), é só cadastrar o e-mail correspondente (`duya@duya-admin.local`) no Supabase e usar esse nome para logar.

### 2. Restringir quem pode editar (Row Level Security)
Isso é o que garante que só você, logado, pode alterar o site — visitantes continuam só enxergando.

No **SQL Editor** do Supabase, rode:

```sql
alter table duya_config enable row level security;
alter table duya_services enable row level security;
alter table duya_gallery enable row level security;
alter table duya_testimonials enable row level security;

-- Qualquer pessoa pode LER (o site público precisa disso)
create policy "leitura publica" on duya_config for select using (true);
create policy "leitura publica" on duya_services for select using (true);
create policy "leitura publica" on duya_gallery for select using (true);
create policy "leitura publica" on duya_testimonials for select using (true);

-- Só usuário logado pode ESCREVER (isso é o que protege o admin)
create policy "escrita autenticada" on duya_config for all using (auth.role() = 'authenticated');
create policy "escrita autenticada" on duya_services for all using (auth.role() = 'authenticated');
create policy "escrita autenticada" on duya_gallery for all using (auth.role() = 'authenticated');
create policy "escrita autenticada" on duya_testimonials for all using (auth.role() = 'authenticated');
```

E no **Storage → bucket `duya` → Policies**, crie:
- 1 policy de leitura pública (`select`, `true`) — para as imagens aparecerem no site.
- 1 policy de escrita (`insert`/`update`/`delete`) restrita a `authenticated` — para só você poder enviar fotos.

Sem esse passo 2, o painel admin.html continua funcionando normalmente, mas tecnicamente alguém com conhecimento técnico ainda poderia editar os dados direto sem passar pelo login — o passo 2 fecha essa brecha.

## No dia a dia
- Editar o site: acesse `/admin`, faça login, mude o que quiser, clique em "Salvar" em cada seção → já reflete no site.
- Enviar fotos (logo, galeria, serviços, depoimentos): use os botões de upload — vão direto para o Supabase Storage e o link é salvo automaticamente.
- Trocar sua senha: dentro do admin, botão "🔑 Alterar senha".
