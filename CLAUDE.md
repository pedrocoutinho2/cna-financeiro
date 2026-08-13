# CLAUDE.md · cnataquara-adm

Instruções permanentes para sessões do Claude neste repo.

## O que é

Sistema administrativo do CNA Taquara. Cobre Taquara e Queimados. Equipe,
documentos, contratos e parcerias, folha de pagamento, despesas, receitas, DRE
e projeções.

Já foi só "financeiro". Não é mais. O escopo hoje é administrativo, e o
financeiro é um dos módulos.

- Produção: https://adm.cnataquara.com.br
- Supabase: `thelqaxsnuynevizhcla` (tabelas `fin_*`)
- Nome do repo segue o padrão da unidade: `cnataquara-{subdomínio}`

## Arquitetura

Monólito de arquivo único. `index.html` com ~2.280 linhas, **um único bloco
`<script>`**, HTML + CSS + JS vanilla inline. Sem build.

Arquivos:

| Arquivo | O que é |
|---|---|
| `index.html` | Aplicação administrativa completa |
| `rep.html` | Tela de registro de ponto |
| `rep-entrar.html` | Entrada do registro de ponto |
| `CNAME` | `adm.cnataquara.com.br` |

Acesso a dados via REST direto, sem `supabase-js`:

```js
const SUPA = 'https://thelqaxsnuynevizhcla.supabase.co';
async function api(...)   // helper único de request
```

Tabelas: `fin_funcionarios`, `fin_func_ficha`, `fin_func_eventos`, `fin_folha`,
`fin_vales`, `fin_lancamentos`, `fin_despesas_recorrentes`, `fin_receitas`,
`fin_contratos`, `fin_parceiros`, `fin_documentos`, `fin_documento_tipos`,
`fin_usuarios`, `fin_config`, `fin_df`, `fin_ponto_espelho`.

RPCs de usuário: `fin_usuarios_listar`, `fin_usuario_criar`,
`fin_usuario_atualizar`, `fin_usuario_senha`, `fin_usuario_excluir`,
`fin_ativar_com_convite`, `fin_perfil_salvar`, `fin_perfil_trocar_senha`.

Storage: bucket `gestao-docs`, acesso por signed URL com expiração de 3600s.

## Atenção: `rep.html` não fala com este banco

`rep.html` e `rep-entrar.html` moram neste repo mas apontam para o Supabase do
ponto eletrônico (`snipevyvfxaotjhnabmx`), não para `thelqaxsnuynevizhcla`.

Não assuma que tudo neste repo é `fin_*`. Antes de mexer nessas duas telas,
confira qual banco a página está usando. O ponto eletrônico é
**arquiteturalmente isolado** por razão fiscal e legal, e essa separação de
banco é proposital. Ver `cnataquara-ponto`.

## Regras de folha

- Marlon e Camila: Comercial, comissão de matrícula
- Sabrina: Coordenação, `coord_comercial` (50% do total de matrícula do Comercial)
- Natanny e Letícia: Administrativo, R$ 1,50 por pagamento
- Professores: sem comissão
- VT: 6% de desconto sobre salário + comissão (configurável)
- INSS: progressivo via função `fin_inss()`, recolhido à parte por guia
- Campo de adiantamento aparece na tabela e no PDF

Telas do administrativo não levam mascote.

## Deploy

Nunca entregue arquivo para upload manual. Sempre via API do GitHub.

1. Extraia o `<script>` com `re.findall`, salve em `/tmp/` e rode `node --check`
2. `GET` do arquivo para pegar o `sha`
3. `PUT` em `contents` com `sha` + conteúdo em base64
4. Poll em `pages/builds/latest` até `built` (~60 a 90s)
5. Confirme lendo via API com `Accept: application/vnd.github.raw`.
   A URL do Pages pode servir cache.

## Regras

- Busque o arquivo atual antes de editar. Nunca escreva de memória.
- Confirme com o Pedro antes de qualquer operação destrutiva no banco.
- Prefixo `fin_` nas tabelas é histórico e fica como está. Renomear tabela aqui
  significa reescrever as queries de um `index.html` de 146 KB, com risco em
  produção e ganho cosmético.
- Padrão visual compartilhado: ver `assets/PADRAO.md` no repo `cnataquara-ponto`.
